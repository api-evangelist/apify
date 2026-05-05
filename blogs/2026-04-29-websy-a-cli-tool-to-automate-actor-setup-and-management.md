---
title: "Websy: a CLI tool to automate Actor setup and management"
url: "https://blog.apify.com/websy-cli-tool-actor-setup-management/"
date: "Wed, 29 Apr 2026 08:30:39 GMT"
author: "BowTiedRaccoon"
feed_url: "https://blog.apify.com/rss/"
---
<div class="kg-card kg-callout-card kg-callout-card-blue"><div class="kg-callout-emoji">&#x1f449;</div><div class="kg-callout-text">This article was produced as part of <a href="https://apify.com/write-for-apify">Write for Apify</a> - a program for developers sharing original articles about what they&apos;ve built with Apify.</div></div><img alt="Websy: a CLI tool to automate Actor setup and management" src="https://storage.ghost.io/c/f2/6e/f26ec999-9a90-4aee-a0d4-9b3ca2bb668f/content/images/2026/04/Websy_-a-CLI-tool-to-automate-Actor-setup-and-management.png" /><p>Maintaining a growing portfolio of <a href="https://apify.com/jungle_synthesizer">Apify Actors</a> comes with overhead that has nothing to do with scraping logic. Update a description here, fix a category there, regenerate schema files, check quality scores. At a certain scale - somewhere around the fifteenth Actor - the same clicks through <a href="https://console.apify.com/">Apify Console</a> start adding up.</p><p>Websy is a CLI tool we built to manage Actors from the terminal using a single YAML spec file as the source of truth. No more tabbing through the UI for routine maintenance. No more hand-editing four separate JSON schema files every time a field changes.</p><p>We spoke with several other Actor builders while working on this. Nobody had built shared tooling for this workflow. Everyone was either doing it manually or relying on scattered personal scripts. That was surprising - it seemed like a significant gap.</p><h2 id="why-websy-and-who-are-we">Why &quot;Websy&quot; and who are we?</h2><p>We are the team at <a href="https://orbtop.com/" rel="noreferrer">OrbTop</a> - led by your friendly neighborhood Raccoon. We build tools that give clients a top-down view of markets, customers, and opportunities. Market intelligence for your online business. Part of that means maintaining a growing portfolio of Actors. This is where Websy comes in.</p><p>We started early on with a spider theme for various tools for our Actors - they&apos;re crawling websites, after all. When we needed a name for the helper script that manages them all, Websy seemed right. An affectionate name for a small utility that handles the tedious work.</p><h2 id="two-problems-nobody-was-solving">Two problems nobody was solving</h2><h3 id="when-clicking-through-the-ui-adds-up">When clicking through the UI adds up</h3><p>Apify Console handles Actor metadata well. Click into settings, update the description, pick categories, set run options, save. It works.</p><p>At scale, the repetition adds up. Each Actor has a title, description, categories, default run options, etc. - all managed through the console UI. Steps get forgotten. Fields get missed.</p><h3 id="schema-files-by-hand">Schema files by hand</h3><p>This is a different problem, but they share a solution, so it seems appropriate to present both of them in one place. Every Apify Actor requires a set of JSON schema files inside the <code>.actor/</code> directory:</p><ul><li><code>actor.json</code> - basic Actor metadata (name, version, build tag)</li><li><code>input_schema.json</code> - defines the input fields users see in the console</li><li><code>dataset_schema.json</code> - describes the output data structure and display views</li><li><code>output_schema.json</code> - defines how results are presented</li></ul><p>Currently, these files are written and maintained by hand. They are verbose JSON, repetitive across Actors, and they all need to stay consistent with each other. Adding a new field to a dataset means editing <code>dataset_schema.json</code> to define the field, potentially updating views in the same file, and making sure the field names match what the scraper actually outputs.</p><p>Renaming a field requires the same kind of multi-file coordination. This is mechanical work that lends itself to automation.</p><h2 id="one-yaml-spec-to-rule-them-all">One YAML spec to rule them all</h2><p>The core idea behind Websy is straightforward: define everything in one <code>websy-spec.yml</code> file and generate or push everything from there.</p><p>Here is what a sample spec file looks like:</p><figure class="kg-card kg-image-card"><img alt="Websy: a CLI tool to automate Actor setup and management" class="kg-image" height="1998" src="https://storage.ghost.io/c/f2/6e/f26ec999-9a90-4aee-a0d4-9b3ca2bb668f/content/images/2026/04/data-src-image-f2d05879-84be-4058-8715-e08bd447b28b.png" width="1800" /></figure><p>One file. Everything in one place. The <code>actor_details</code> section maps directly to what would otherwise be set through Apify Console. The <code>schemas</code> section defines all four JSON files. When adding a new Actor, we copy a spec, adjust the fields, and run two commands.</p><p>The spec lives in version control alongside the Actor code. Changes to descriptions, schemas, or categories are tracked in git history the same way code changes are.</p><h2 id="building-the-cli">Building the CLI</h2><p>Websy is a Node.js CLI built with Commander.js. The Websy class is a thin wrapper around the <a href="https://docs.apify.com/api/v2">Apify REST API</a> using <code>got</code> for HTTP requests. The value is in the workflow, not the tech stack.</p><figure class="kg-card kg-image-card"><img alt="Websy: a CLI tool to automate Actor setup and management" class="kg-image" height="958" src="https://storage.ghost.io/c/f2/6e/f26ec999-9a90-4aee-a0d4-9b3ca2bb668f/content/images/2026/04/data-src-image-b15f20d0-c5e3-4969-9b1f-efbdc6b0c212.png" width="1800" /></figure><pre><code class="language-javascript">class Websy {
  constructor(config) {
    this.apiToken = process.env.API_TOKEN;
    this.baseUrl = &apos;https://api.apify.com/v2&apos;;
    this.client = got.extend({
      prefixUrl: this.baseUrl,
      headers: {
        &apos;Authorization&apos;: `Bearer ${this.apiToken}`,
        &apos;Content-Type&apos;: &apos;application/json&apos;
      },
      responseType: &apos;json&apos;
    });
  }
}</code></pre><p>One practical detail that eliminates a lot of repetition: auto-resolving the Actor ID. Most of the time, Websy runs from inside an Actor&apos;s project directory, so it reads <code>.actor/actor.json</code> and derives the ID automatically. No need to pass <code>--id</code> on every command.</p><pre><code class="language-javascript">static resolveActorId(providedId, prefix) {
  if (providedId) return providedId;
  try {
    const actorJsonPath = join(process.cwd(), &apos;.actor&apos;, &apos;actor.json&apos;);
    if (fs.existsSync(actorJsonPath)) {
      const actorJson = JSON.parse(fs.readFileSync(actorJsonPath, &apos;utf8&apos;));
      const actorName = actorJson.name;
      if (actorName) {
        const derivedId = `${prefix}~${actorName}`;
        console.log(`Using actor ID: ${derivedId} (derived from .actor/actor.json)`);
        return derivedId;
      }
    }
  } catch (error) {
    console.error(&apos;Error reading actor.json:&apos;, error.message);
  }
  return null;
}</code></pre><p>The main commands:</p><ul><li><code>websy update</code> - pushes Actor metadata (title, description, categories, run options) from the spec to the Apify API</li><li><code>websy info</code> - pulls the current state of an Actor and displays it alongside quality scores, metrics, and a diff against the local spec</li><li><code>websy gen-schemas</code> - generates all four <code>.actor/*.json</code> files from the schemas section of the spec</li><li><code>websy gen-input</code> - generates an <code>INPUT.json</code> with sensible defaults for local testing</li></ul><h2 id="schema-generationthe-biggest-time-saver">Schema generation - the biggest time saver</h2><p>This is where Websy provides the most value. The <code>ActorSchemaManager</code> class takes the schemas section of the spec and generates all four JSON files in one operation.</p><p>Running <code>websy gen-schemas</code> produces:</p><ul><li><code>.actor/actor.json</code> with proper structure and version info</li><li><code>.actor/input_schema.json</code> with all input fields, editors, types, and validation rules</li><li><code>.actor/dataset_schema.json</code> with field definitions and display views</li><li><code>.actor/output_schema.json</code> with the output template</li></ul><p>The YAML-to-JSON mapping handles the repetitive parts automatically. Field names are converted to title case for display labels. Editor types are inferred from field types - integers get number editors, booleans get checkboxes, arrays get string list editors. Any of it can be overridden.</p><pre><code class="language-javascript">static inferEditor(fieldConfig) {
  if (fieldConfig.editor) return fieldConfig.editor;
  const type = fieldConfig.type || &apos;string&apos;;
  switch (type) {
    case &apos;integer&apos;:
    case &apos;number&apos;:
      return &apos;number&apos;;
    case &apos;boolean&apos;:
      return &apos;checkbox&apos;;
    case &apos;array&apos;:
      return &apos;stringList&apos;;
    default:
      return &apos;textfield&apos;;
  }
}</code></pre><figure class="kg-card kg-image-card"><img alt="Websy: a CLI tool to automate Actor setup and management" class="kg-image" height="1176" src="https://storage.ghost.io/c/f2/6e/f26ec999-9a90-4aee-a0d4-9b3ca2bb668f/content/images/2026/04/data-src-image-7bb945b1-a2ea-40b4-b687-a40183cfc8fe.png" width="1017" /></figure><p>Before running for real, we always preview first:</p><pre><code class="language-bash">websy gen-schemas --dry-run</code></pre><p>This prints all four generated JSON files to the terminal without writing anything to disk. Review the output, verify the fields look correct, then run again without the flag.</p><p>The same workflow applies to input file generation:</p><pre><code class="language-bash">websy gen-input --dry-run</code></pre><p>This creates an <code>INPUT.json</code> populated with default values from the spec - prefills, sensible zeros for numbers, empty arrays for lists. Useful for quickly testing an Actor locally without assembling the input by hand every time. We made it a separate command from <code>gen-schemas</code> so that one can rerun it many times without having all the schema files generated every time.</p><h2 id="keeping-local-and-remote-in-sync">Keeping local and remote in sync</h2><p>This feature exists because of early mistakes. Before building the comparison logic, there was no reliable way to tell whether the latest spec had been pushed or not. Running <code>websy update</code> was a bit tricky - is this an intentional sync, or is it about to overwrite something that was set through the UI and never captured in the spec?</p><p>The <code>websy info</code> command now pulls the Actor&apos;s current state from the API and compares it field by field against the local spec. If anything differs, it shows exactly what:</p><pre><code>=== Local vs Online Diffs ===
&#x26a0;&#xfe0f;  The following fields differ between local spec and online:

  description:
    Local:  &quot;Scrapes product data from example.com with full pagination support&quot;
    Online: &quot;Scrapes product data from example.com&quot;

  categories:
    Local:  [&quot;ECOMMERCE&quot;,&quot;LEAD_GENERATION&quot;]
    Online: [&quot;ECOMMERCE&quot;]

&#x1f4a1; Run &quot;websy update&quot; to sync local spec to online.</code></pre><p>If everything matches:</p><pre><code>&#x2705; Local spec is in sync with online actor.</code></pre><p>The info command also pulls quality scores, recommendations, public metrics (users, ratings, bookmarks as shown on <a href="https://apify.com/store">Apify Store</a>), run success rates, and issue response times - all in one terminal view. Before Websy, checking an Actor&apos;s health meant opening several screens every time.</p><pre><code>=== Actor Information ===
ID:          abc123def456
Name:        my-website-scraper
Title:       My Website Scraper
Version:     0.1
Categories:  ECOMMERCE, LEAD_GENERATION
IsPublic:    true

&#x2705; Pricing: PAY_PER_EVENT (1 events: result: $0.005)
&#x2705; Icon: Set

=== Actor Metrics ===
Total Users:          1,234
Monthly Active Users: 456
Star Rating:          &#x2b50; 4.8 (23)
Bookmarks:            89
Runs Success Rate:    &gt;99% succeeded (2,340 runs in 30d)
Issue Response Time:  2h 15m

=== Actor Quality ===
Quality Score:      87.50%
Quality Percentile: 92.00%</code></pre><figure class="kg-card kg-image-card"><img alt="Websy: a CLI tool to automate Actor setup and management" class="kg-image" height="1666" src="https://storage.ghost.io/c/f2/6e/f26ec999-9a90-4aee-a0d4-9b3ca2bb668f/content/images/2026/04/data-src-image-3f7a777a-ee8c-4788-94b9-e0398c7c8e43.png" width="1800" /></figure><h2 id="what-didnt-work">What didn&apos;t work</h2><p>Not everything went as planned.</p><p>We originally intended to automate Actor icon uploads - read an image file, push it through the API. The image handling libraries turned out to be more complex to integrate than expected, and the API&apos;s icon upload is not as straightforward as setting a text field. We shelved it. Instead, Websy checks whether an icon is set and flags it in the info output if it is missing. A simpler solution that still catches the oversight.</p><p>The diffing feature was born out of accidentally overwriting live settings. Without a comparison view, there was no reliable way to tell whether a spec had already been pushed. The diff solved that problem, but it should have been built from the start rather than after losing changes.</p><p>The list of valid Actor categories - values like AI, ECOMMERCE, SEO_TOOLS - appears to change over time on the Apify side. We maintain it as a hardcoded array in the source. When Apify adds a new category, the validation rejects it until the list is updated manually. A public API endpoint for valid categories would make this easier to maintain, and we&apos;d happily switch to it.</p><h2 id="getting-started">Getting started</h2><p>The full source code is available on GitHub: <a href="https://github.com/OrbTop/websy">https://github.com/OrbTop/websy</a></p><p>Setup is straightforward. Clone the repo, configure the Apify API token in the config file, and the tool is ready. The commands used most frequently:</p><pre><code class="language-bash"># Generate all .actor/*.json schema files from the spec
websy gen-schemas -s ./websy-spec.yml

# Preview before writing (recommended as a first step)
websy gen-schemas -s ./websy-spec.yml --dry-run

# Push Actor metadata to the API
websy update -s ./websy-spec.yml

# Check Actor status, quality, and diff against local spec
websy info

# Generate INPUT.json for local testing
websy gen-input</code></pre><p>Apify Console handles a lot that Websy doesn&apos;t touch. But for repetitive setup and upkeep tasks, having everything in a spec file and a terminal command has eliminated a significant amount of friction and a significant number of mistakes.</p><p>The repository is open. Feedback and suggestions from other builders dealing with similar challenges are welcome.</p><p>Stay tuned for future articles where we expand Websy with support for managing pricing info, generating Actor README files, and more.</p><div class="kg-card kg-callout-card kg-callout-card-blue"><div class="kg-callout-emoji">&#x1f449;</div><div class="kg-callout-text">Want to share what you&apos;ve built with Apify or Crawlee? Join <a href="https://apify.com/resources/write-for-apify" rel="noreferrer">Write for Apify</a> - a program for developers sharing original articles about their projects, with $500 for accepted submissions.</div></div>
