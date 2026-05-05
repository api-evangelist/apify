---
title: "How I built optimized Apify Actors in Go"
url: "https://blog.apify.com/how-i-built-optimized-apify-actors-in-go/"
date: "Tue, 28 Apr 2026 10:48:08 GMT"
author: "Iñigo Garcia Olaizola"
feed_url: "https://blog.apify.com/rss/"
---
<div class="kg-card kg-callout-card kg-callout-card-blue"><div class="kg-callout-emoji">&#x1f449;</div><div class="kg-callout-text">This article was written by&#xa0;<a href="https://apify.com/igolaizola" rel="noreferrer">I&#xf1;igo Garcia Olaizola</a>&#xa0;as part of Write for Apify - a program for developers sharing original articles about what they&apos;ve built with Apify.</div></div><img alt="How I built optimized Apify Actors in Go" src="https://storage.ghost.io/c/f2/6e/f26ec999-9a90-4aee-a0d4-9b3ca2bb668f/content/images/2026/04/How-I-built-optimized-Apify-Actors-in-Go-4.png" /><p>I published my first <a href="https://apify.com/igolaizola"><u>Apify Actor</u></a> at the end of 2024 after several years of building scrapers and automation tools in Go. Go is my most productive language, and most of my side projects are already written in it.</p><figure class="kg-card kg-image-card"><a href="https://apify.com/igolaizola"><img alt="How I built optimized Apify Actors in Go" class="kg-image" height="481" src="https://storage.ghost.io/c/f2/6e/f26ec999-9a90-4aee-a0d4-9b3ca2bb668f/content/images/2026/04/data-src-image-951f6edb-187d-4b7b-aff3-f94bad5e0e99.png" width="583" /></a></figure><p>I knew <a href="https://docs.apify.com/sdk/js/docs/overview"><u>Apify&#x2019;s JavaScript</u></a> and <a href="https://docs.apify.com/sdk/python/docs/overview"><u>Python SDKs</u></a> were the standard path. But my starting point was simple: I really wanted to build the Actor in Go. The side effects of that choice were lower memory usage, smaller Docker images, and full control over every API call. To support that workflow, I built a thin Go client on top of the Apify API, and this approach now underpins all the Actors in the private GitHub repository, where I manage them. The repo is private, but I&#x2019;ll share the core patterns, trade-offs, and a few self-contained snippets you can reuse. I&#x2019;ll include the snippets inline, and you can also browse them in this <a href="https://gist.github.com/igolaizola/4887f50d24f0075ed46957f2b6157f37"><u>GitHub Gist</u></a>.</p><figure class="kg-card kg-image-card"><img alt="How I built optimized Apify Actors in Go" class="kg-image" height="1024" src="https://storage.ghost.io/c/f2/6e/f26ec999-9a90-4aee-a0d4-9b3ca2bb668f/content/images/2026/04/data-src-image-809108ab-10aa-4040-b0aa-b04a683034d5.png" width="1536" /></figure><p>This article is a practical walkthrough of that setup: how I structure Go-based Apify Actors, how I interact with the Apify API directly, and how I package small Docker images that reliably fit into the 128 MB Apify tier. I&#x2019;ll use my <a href="https://apify.com/igolaizola/facebook-ad-library-scraper"><u>Facebook (Meta) Ad Library Scraper</u></a> Actor as a concrete example and reference real code from one of my Actors.</p><p>If you already know Apify and want a Go-first workflow with tight resource budgets, this is the approach that has worked well for me.</p><h2 id="project-context">Project context</h2><p>I keep all my Actors in a single monorepo. Each Actor lives under <code>actors/&lt;name&gt;</code>, while shared Apify-related logic sits in <code>pkg/apify</code>. This lets me reuse the same API client, logging conventions, and output logic across dozens of Actors without copy-pasting.</p><p>I chose to work directly with the Apify API for two main reasons:</p><ol><li><strong>I wanted full control over runtime cost and memory usage.</strong> A thin client means no hidden buffers, no background workers, and no extra dependencies.</li><li><strong>Go is my default language.</strong> I iterate faster in a language I already know well than by learning a new SDK surface.</li></ol><p>The downside is obvious: I own the edge cases and any API changes. I&#x2019;ll come back to those trade-offs later.</p><h2 id="architecture-overview">Architecture overview</h2><h3 id="repository-layout">Repository layout</h3><p>This is a simplified version of the core structure:</p><pre><code>actors/
  fbads/
    cmd/fbads/
    fbads.go
    .actor/
pkg/
  apify/
    client.go
    apify.go
    saver.go
Dockerfile
Makefile
</code></pre><p>Each Actor has its own <code>cmd/&lt;actor&gt;</code> entrypoint and a package under <code>actors/&lt;actor&gt;</code> containing the business logic. Shared Apify interaction code lives in <code>pkg/apify</code>, which includes:</p><ul><ul><li>a small HTTP client for Apify API calls (<code>client.go</code>, <code>apify.go</code>)</li></ul><li>an output saver that writes to the Apify dataset or to a local JSON file (<code>saver.go</code>)</li><li>helpers for input parsing and key-value store access</li></ul><p>This keeps Actor-specific code small and makes it easy to roll out improvements across all Actors.</p><h3 id="the-dual-mode-runtime">The dual-mode runtime</h3><p>Every Actor runs in two modes:</p><ul><li><strong>Local mode</strong>, for development and debugging</li><li><strong>Apify mode</strong>, when running on the platform</li></ul><p>The Actor decides which mode it&#x2019;s in by checking environment variables. I use ACTOR_RUN_ID as the switch:</p><pre><code class="language-go">var apifyClient *apify.Client
if os.Getenv(&quot;ACTOR_RUN_ID&quot;) == &quot;&quot; {
    // Local mode: read input from file or CLI, write JSON to a file
} else {
    // Apify mode: read input from Apify KV store, write to dataset
    apifyClient, err = apify.NewActor(cfg.Debug)
}
</code></pre><p>This single check keeps both execution paths aligned and avoids the common problem where &#x201c;local mode&#x201d; slowly diverges from how the Actor behaves on Apify.</p><h2 id="building-a-go-actor-with-the-apify-api">Building a Go Actor with the Apify API</h2><h3 id="input-handling-and-schema-alignment">Input handling and schema alignment</h3><p>When running on Apify, I fetch the input directly from the key-value store. The client performs a simple GET request:</p><pre><code class="language-go">func (c *Client) GetInput(ctx context.Context, v any) error {
    u := fmt.Sprintf(&quot;v2/key-value-stores/%s/records/INPUT&quot;, c.key)
    if _, err := c.do(ctx, &quot;GET&quot;, u, nil, v); err != nil {
        return fmt.Errorf(&quot;apify: couldn&apos;t get input: %w&quot;, err)
    }
    return nil
}</code></pre><p>The input schema maps directly to a Go struct. For facebook-ad-library-scraper, it looks like this:</p><pre><code class="language-go">type Input struct {
    ProxyConfiguration apify.ProxyConfiguration `json:&quot;proxyConfiguration&quot;`
    MaxItems           int                      `json:&quot;maxItems&quot;`
    Query              string                   `json:&quot;query&quot;`
    Advertisers        []string                 `json:&quot;advertisers&quot;`
    Country            string                   `json:&quot;country&quot;`
    Category           string                   `json:&quot;category&quot;`
    MediaType          string                   `json:&quot;mediaType&quot;`
    MinDate            string                   `json:&quot;minDate&quot;`
    MaxDate            string                   `json:&quot;maxDate&quot;`
    ActiveStatus       string                   `json:&quot;activeStatus&quot;`
}</code></pre><p>In local mode, I reuse the same struct but load it from a JSON file or a raw JSON string. This keeps local tests tightly aligned with real Apify input and avoids surprises when deploying.</p><h3 id="output-handling-and-dataset-writes">Output handling and dataset writes</h3><p>For output, I push items to the default dataset when running on Apify and write to a local JSON file in local mode. This logic is centralized in <code>pkg/apify/saver.go</code>:</p><pre><code class="language-go">func (s *Saver[T]) Save(ap *Client, output string, current []T) error {
    ctx, cancel := context.WithTimeout(context.Background(), time.Minute)
    defer cancel()

    if ap != nil {
        if ap.MaxChargeReached() {
            return errors.New(&quot;apify: max charge reached&quot;)
        }
        if err := ap.SaveItems(ctx, current); err != nil {
            return err
        }
        if ap.isPPE {
            if err := ap.AddCharge(ap.resultEvent, len(current)); err != nil {
                return err
            }
        }
        return nil
    }
    // Otherwise append to a local JSON file
}
</code></pre><p>Some of the logic here is specific to <a href="https://docs.apify.com/platform/actors/publishing/monetize/pay-per-event"><u>pay-per-event (PPE) Actors</u></a>, which I&#x2019;ll cover later. Under the hood, <code>SaveItems</code> is just a direct API call to the dataset endpoint:</p><pre><code class="language-go">func (c *Client) SaveItems(ctx context.Context, v any) error {
    u := fmt.Sprintf(&quot;v2/datasets/%s/items&quot;, c.dataset)
    if _, err := c.do(ctx, &quot;POST&quot;, u, v, nil); err != nil {
        return fmt.Errorf(&quot;apify: couldn&apos;t put items: %w&quot;, err)
    }
    return nil
}</code></pre><p>Centralizing this logic keeps behavior consistent across all Actors, both locally and on the platform.</p><h3 id="proxies">Proxies</h3><p>Proxy configuration comes from input. I support both <a href="https://apify.com/proxy" rel="noreferrer">Apify Proxy</a> and external proxies.</p><p>When Apify Proxy is requested, I resolve proxy groups and country settings into a concrete proxy URL. The part I care about most is how the username is constructed from proxy groups and country:</p><pre><code class="language-go">username := &quot;auto&quot;
var parts []string
if len(cfg.ApifyProxyGroups) &gt; 0 {
    parts = append(parts, fmt.Sprintf(&quot;groups-%s&quot;, strings.Join(cfg.ApifyProxyGroups, &quot;+&quot;)))
}
if cfg.ApifyProxyCountry != &quot;&quot; {
    parts = append(parts, fmt.Sprintf(&quot;country-%s&quot;, strings.ToUpper(cfg.ApifyProxyCountry)))
}
if len(parts) &gt; 0 {
    username = strings.Join(parts, &quot;,&quot;)
}
proxyURL, err := url.Parse(fmt.Sprintf(&quot;http://%s:%s@%s:%s&quot;, username, password, host, port))
if err != nil {
    return nil, fmt.Errorf(&quot;couldn&apos;t parse proxy URL: %w&quot;, err)
}</code></pre><p>This gives me predictable proxy behavior without relying on SDK abstractions.</p><h2 id="pay-per-event-ppe-actors">Pay-per-event (PPE) Actors</h2><p>Some of my Actors run in PPE mode. My <a href="https://apify.com/igolaizola/zillow-scraper-ppe"><u>Zillow Actor</u></a> is a good example of this.</p><p>PPE is toggled by the <code>APIFY_PPE</code> environment variable. On startup, the client detects PPE mode:</p><pre><code class="language-go">if os.Getenv(&quot;APIFY_PPE&quot;) == &quot;1&quot; {
    slog.Info(&quot;This actor is running in pay-per-event (PPE) mode&quot;)
    c.isPPE = true
}</code></pre><p>When PPE is enabled, <code>Saver.Save</code> charges per result and I also guard the main loop against exceeding the configured cap. In <code>actors/zillow</code>, the Actor stops early when the cap is reached like this:</p><pre><code class="language-go">if apify != nil &amp;&amp; apify.MaxChargeReached() {
    slog.Warn(&quot;&#x26a0;&#xfe0f; Reached the max charge limit, stopping the actor&quot;)
    break
}</code></pre><p>The cap check itself is a small helper:</p><pre><code class="language-go">func (c *Client) MaxChargeReached() bool {
    return c.isPPE &amp;&amp; c.maxCharge &gt; 0 &amp;&amp; c.charged &gt;= c.maxCharge
}</code></pre><p>This keeps PPE behavior explicit: the Actor can keep streaming results, but it won&#x2019;t exceed the configured budget. The cap logic is intentionally simple: keep a running <code>charged</code> total in USD and stop when it reaches <code>ACTOR_MAX_TOTAL_CHARGE_USD</code> (if that env var is set).</p><ul><li><code>c.isPPE</code> is enabled when <code>APIFY_PPE=1</code></li><li><code>c.maxCharge</code> comes from <code>ACTOR_MAX_TOTAL_CHARGE_USD</code> (optional)</li><li><code>c.charged</code> starts with the automatic &quot;actor start&quot; charge from the Actor pricing config and increases as I call <code>AddCharge(...)</code> for result events</li></ul><p>The <code>AddCharge</code> method that gets triggered on each save is just a thin wrapper around the Apify charge endpoint, plus local accounting to track the total charged amount. Default dataset item events are charged automatically by Apify, so I only call the charge endpoint for custom events.</p><pre><code class="language-go">func (c *Client) AddCharge(event string, count int) error {
    // Default dataset item events are charged automatically by Apify
    if event != DatasetItemEvent {
        u := fmt.Sprintf(&quot;v2/actor-runs/%s/charge&quot;, c.runID)
        req := &amp;chargeRequest{
            EventName: event,
            Count:     count,
        }
        if _, err := c.do(context.Background(), &quot;POST&quot;, u, req, nil); err != nil {
            return fmt.Errorf(&quot;apify: couldn&apos;t add charge: %w&quot;, err)
        }
    }

    // Update the charged amount
    if price, ok := c.prices[event]; ok {
        c.charged += price * float64(count)
    }
    return nil
}</code></pre><h2 id="building-tiny-fast-docker-images">Building tiny, fast Docker images</h2><h3 id="multi-stage-build">Multi-stage build</h3><p>The root <code>Dockerfile</code> uses a multi-stage build: one Go builder stage and one minimal Alpine runtime stage. The final image contains only a single binary.</p><pre><code class="language-go"># builder image
FROM golang:alpine as builder
COPY . /src
WORKDIR /src
RUN apk add --no-cache make bash git

ARG ACTOR=&quot;&quot;
RUN set -eux; \
    if [ -z &quot;$ACTOR&quot; ]; then \
      if [ -n &quot;$ACTOR_PATH_IN_DOCKER_CONTEXT&quot; ]; then \
        ACTOR=&quot;$(basename &quot;$ACTOR_PATH_IN_DOCKER_CONTEXT&quot;)&quot;; \
      else \
        echo &quot;Set --build-arg ACTOR=&lt;name&gt; (or ACTOR_PATH_IN_DOCKER_CONTEXT)&quot;; exit 1; \
      fi; \
    fi; \
    make build ACTOR=&quot;$ACTOR&quot;

# running image
FROM alpine
WORKDIR /home
COPY --from=builder /src/bin/app /bin/app

ENTRYPOINT [ &quot;/bin/app&quot; ]
</code></pre><p>The final image doesn&#x2019;t include the Go toolchain, source code, or build caches. It&#x2019;s just the binary.</p><figure class="kg-card kg-image-card"><img alt="How I built optimized Apify Actors in Go" class="kg-image" height="362" src="https://storage.ghost.io/c/f2/6e/f26ec999-9a90-4aee-a0d4-9b3ca2bb668f/content/images/2026/04/data-src-image-54e3ecd0-9eea-4cfe-b10e-74d6a7a00292.png" width="937" /></figure><h3 id="makefile-and-reproducible-builds">Makefile and reproducible builds</h3><p>I use a Makefile to produce static, reproducible binaries with CGO disabled:</p><pre><code class="language-make">GOOS=$$os GOARCH=$$arch GOARM=$$arm CGO_ENABLED=0 \
go build \
  -a -x -tags netgo,timetzdata -installsuffix cgo -installsuffix netgo \
  -ldflags &quot; \
    -X main.version=$(VERSION) \
    -X main.commit=$(COMMIT_SHORT) \
    -X main.date=$(shell date -u +&apos;%Y-%m-%dT%H:%M:%SZ&apos;) \
  &quot; \
  -o &quot;$$out&quot; \
  ./actors/$(ACTOR)/cmd/$(ACTOR)</code></pre><p>The <code>netgo</code> tag and <code>CGO_ENABLED=0</code> produce a fully static binary. The <code>timetzdata</code> tag embeds timezone data so I don&#x2019;t need to install <code>tzdata</code> in the runtime image.</p><h2 id="memory-footprint-and-runtime-cost">Memory footprint and runtime cost</h2><p>My target is the 128 MB Apify tier. I design these Actors to stay comfortably under that limit:</p><ul><li>I avoid headless browsers unless unavoidable</li><li>I stream results in small batches instead of buffering everything</li><li>The runtime image is minimal and contains no extra libraries</li></ul><p>The first version of this setup routinely crossed 128 MB and was killed mid-run. The culprit was a slice accumulating parsed results before flushing to the dataset. Switching to smaller batches and clearing slices after each save brought peak memory usage back under 50 MB.</p><figure class="kg-card kg-image-card"><img alt="How I built optimized Apify Actors in Go" class="kg-image" height="471" src="https://storage.ghost.io/c/f2/6e/f26ec999-9a90-4aee-a0d4-9b3ca2bb668f/content/images/2026/04/data-src-image-620470df-96ee-4ad2-9808-bc6d673b6fac.png" width="1078" /></figure><p>I use the memory graph in the Apify run stats as my feedback loop. If I see a spike, it&#x2019;s almost always caused by buffering too much data or holding large strings longer than intended.</p><h2 id="debugging-and-local-workflow">Debugging and local workflow</h2><p>I run every Actor locally with a JSON input file before deploying. My usual flow is:</p><ol><li>Prepare an input JSON file matching the Apify input schema</li><li>Run the binary directly (no Docker) for fast iteration</li><li>Write output to a local JSON file and inspect it</li></ol><p>In <a href="https://apify.com/igolaizola/facebook-ad-library-scraper"><u>Facebook (Meta) Ad Library Scraper</u></a>, I store debug artifacts under a logs directory and validate input early. The dual-mode runtime is the biggest time saver: one codebase, two environments, and no debug-only branches that later drift.</p><h2 id="trade-offs-and-lessons-learned">Trade-offs and lessons learned</h2><p>This approach isn&#x2019;t perfect:</p><ul><li><strong>No SDK means I own the API surface.</strong> When Apify changes an endpoint, I update my client.</li><li><strong>No SDK helpers.</strong> I re-implemented dataset writes, input parsing, retries, and PPE logic.</li><li><strong>Docs matter more.</strong> I keep the Apify API docs open when adding features.</li></ul><p>For quick one-off scrapers, the official SDKs are still the better tool. For long-running, cost-sensitive Actors, this setup has paid off for me.</p><p>If I were starting today, I&#x2019;d still build the same way, but I&#x2019;d invest earlier in tests around the API client and add a small CI check against a mocked Apify API.</p><h2 id="conclusion">Conclusion</h2><p>Using Go and the Apify API directly has been the best setup for my workload. It keeps resource usage predictable, makes local debugging fast, and lets me run Actors in the smallest Apify tier without surprises.</p><p>If you&#x2019;re a Go developer and want fine-grained control over how your Actors behave, I&#x2019;d recommend trying this approach. Start with a single Actor, keep the client minimal, and only add abstractions when they earn their place.</p>
