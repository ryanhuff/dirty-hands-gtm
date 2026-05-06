# You Can Vibe Code a CPQ. You Probably Shouldn't.

Every few years, the "build vs. buy" debate gets a new costume. Spreadsheets. No-code tools. Now it's vibe coding: describe what you want in plain English, watch an AI spit out a working app, and ship it before lunch.

For quoting? The instinct makes sense. If your team has been managing pricing in a spreadsheet saved to someone's desktop, a vibe-coded app is an upgrade. It can enforce some pricing rules. It can look like a real tool. It can do things a spreadsheet never could.

But the question was never "can you build it?" The question is "who maintains it at 2 AM when a rep can't send a quote?"

## The Spreadsheet Lineage

Vibe-coded CPQ tools are the direct descendants of the spreadsheet workaround. Same impulse: the official tool doesn't do what we need, so we'll build our own.

Companies have been doing this for years. A RevOps manager builds a pricing spreadsheet with VLOOKUP formulas and conditional formatting. It works. Then the product catalog changes. Then a new pricing tier gets added. Then someone needs approval workflows. Then someone saves over the master copy.

The spreadsheet didn't fail because it was a spreadsheet. It failed because internal tools accumulate requirements faster than one team can maintain them.

Vibe coding accelerates the first part of that cycle. You get to "working prototype" faster than ever. But the maintenance curve hasn't changed.

## What Maintenance Actually Looks Like

Building a quoting tool is not the hard part. Maintaining one is.

**Week 1:** Your vibe-coded app generates quotes from a price list. Reps love it.

**Month 2:** Finance wants discount controls. You add approval logic. The AI generates code you don't fully understand, but it works.

**Month 4:** A product bundle changes. The pricing logic breaks for one specific configuration. The rep discovers this while on a call with the prospect. You go back to the AI and describe the problem. But the AI doesn't remember the original requirements, the pricing edge cases you described three months ago, or why the code works the way it does. It generates a fix that breaks something else.

**Month 6:** Sales wants multi-currency support. Then the VP asks for quote versioning so they can track what changed between revisions. The person who prompted the original build moved to a new role or is working on a new project. The new RevOps hire opens the codebase and realizes they're not looking at something they can extend. They're looking at something they need to rebuild from scratch, or hire a developer to untangle.

**Month 9:** You need audit trails for finance. Discount reporting for leadership. Role-based access controls for compliance. Each feature is "just one more prompt," but the codebase is now a patchwork of AI-generated modules that were never architected to work together.

This is the trajectory. Not because vibe coding is bad. Because internal tools follow this pattern regardless of how they're built.

## The Maintenance Tax Nobody Budgets For

When you buy a purpose-built tool, maintenance is included. Updates ship. Bugs get fixed. New HubSpot API versions get supported. Security patches happen without a Slack message to your RevOps team.

When you build internally (whether with code, no-code, or vibe code), you're signing up for:

- **Every bug is your bug.** No support ticket. No SLA. Just your team, the codebase, and a rep who needs a quote out now.
- **Every integration update is your problem.** HubSpot changes their API. Your ERP updates. Your payment processor modifies their webhook format. Each one is a maintenance event.
- **Every new requirement is a development project.** Approval workflows, discount escalation, branded templates, multi-currency support. Each one seemed small in isolation. Together, they're a product roadmap.
- **Knowledge concentration risk.** The person who prompted the AI to build it becomes a single point of failure. And unlike a developer leaving behind code their peers can read, a non-technical builder leaves behind a codebase nobody on the team can maintain. When they leave or get promoted, you don't lose institutional knowledge. You lose the only person who could talk to the AI about what it built.

None of these costs show up in the "look what I built this afternoon" demo.

## Where Vibe Coding Actually Works

This isn't an argument against vibe coding. It's an argument against vibe coding the wrong things.

Vibe coding is excellent for:

- **One-off internal tools** that solve a specific, bounded problem and can be rebuilt if they break.
- **Prototypes** that validate whether a workflow makes sense before committing to a purchase.
- **Glue scripts** that connect two systems with a simple data transformation.

The pattern: low complexity, low maintenance surface, and disposable. If the tool breaks and you can rebuild it in an afternoon, vibe code it. The person maintaining it doesn't need to be a developer, because there's nothing to debug.

Quoting doesn't fit that pattern. Quoting is a high-stakes, high-complexity process that touches sales, finance, and operations. It accumulates requirements over time. It needs to be reliable every single day, because a broken quote is a delayed deal.

## The Real Comparison

The honest comparison isn't "vibe-coded CPQ vs. buying Quotivity." It's "vibe-coded CPQ vs. the spreadsheet you're already using."

Both are internal tools. Both depend on your team for maintenance. Both accumulate complexity that eventually exceeds what one person can manage.

The vibe-coded version just gets to that point faster, because AI lets you build more complexity before you feel the weight of maintaining it.

A purpose-built CPQ inside HubSpot handles this differently. Price books, product configuration, discount controls, approval workflows, branded templates: these exist as maintained features, not as prompts someone ran six months ago. When HubSpot updates their API, the integration updates. When you need audit trails, they're already there. When the RevOps lead changes roles, the next person inherits a product with documentation, not a codebase with commit messages like "fixed the thing."

## The Decision Framework

If you're evaluating whether to vibe code your CPQ, ask three questions:

**1. How many people depend on this tool, and what happens when it's down?**
Quoting sits at the center of your revenue cycle. A broken quote isn't a minor inconvenience. It's a deal that stalls, a rep who can't close, a prospect who goes quiet. If every deal in your pipeline runs through this tool, the cost of an outage isn't "someone files a bug." It's lost revenue. Internal tools with that kind of blast radius need reliability guarantees that a single maintainer can't provide.

**2. What happens when the builder leaves?**
If the answer is "we rebuild it," that's the cost of building. Factor in rebuilding every 12 to 18 months as team members rotate.

**3. Does this tool need to get more complex over time?**
Quoting always gets more complex. New products, new pricing models, new compliance requirements, new reporting needs. Purpose-built tools absorb that complexity. Internal tools buckle under it.

## Build the Things Only You Can Build

The vibe coding movement is genuinely exciting. It puts real capability in the hands of teams who never had access to custom software before.

Use that power for the things that are unique to your business. The internal dashboard that shows exactly the metrics your CEO cares about. The workflow automation that maps to your specific approval chain. The data transformation that connects your proprietary system to your CRM.

Don't use it to rebuild something that already exists as a maintained product. Quoting is a solved problem. The question is whether you want to solve it yourself, every day, or whether you want to sell while someone else maintains the infrastructure.

Your RevOps team has better things to build.

<!-- Editorial notes:
- Fixed duplicate "quote versioning" reference (appeared in both Month 6 and Month 9). Changed Month 9 to "role-based access controls for compliance."
- No prohibited terms found.
- No em-dashes or semicolons found.
- No unsubstantiated product claims. Article is opinion/thought leadership, not product marketing. Product mention in "The Real Comparison" section is contextual, not a pitch.
- Heading hierarchy clean: H1 > H2 throughout.
- Internal consistency verified: "three questions" promised, three delivered.
- Sentence rhythm varies well throughout. No runs of 3+ same-structure sentences outside of intentional list patterns.
- Terminology consistent: "vibe coding" (gerund), "vibe code" (verb), "vibe-coded" (adjective) used correctly throughout.
-->
