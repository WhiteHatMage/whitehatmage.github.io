---
title: A White Mage’s Guide to Web3 Bug Hunting
description: >-
  This post is about working as a professional web3 bug bounty hunter and some things to consider before choosing a program.
categories: [Hunting]
tags: [hunting]
pin: true
image:
  path: /assets/img/char-select.png
---

## Life of a Hunter

Hunting comes with a lot of uncertainty. If this is your main source of income, you can go months without earning anything. That pressure adds up, both mentally and financially. If you hunt part time, you are competing with people who can spend all day on it.

There is no formula for predicting bounty income. But, if there were, it would have to include how likely a real exploit exists, how much damage it could cause, and how likely the project is to treat you fairly after disclosure.

---

## The Odds of Finding a Real Exploit

To find a live exploit, one has to exist first. After that, you still need to be the one who finds it before anyone else.

### What Makes Vulnerabilities More Likely

We never know which projects are vulnerable ahead of time, so all we can do is risk assessment and decide where our time has the best expected return.

#### Complexity

**Bugs hide in complex systems.**

The bugs themselves are usually simple. What makes them hard to spot is the attack path. Layers of logic and assumptions stack until a small mistake becomes exploitable. If you look at real fixes, many involve just a single missing check.

Most serious issues I have found were simple mistakes that only mattered because they existed inside large and complicated systems.

Simple codebases rarely have meaningful exploit paths. I usually avoid them or spend minimal time on them. Any real issues there are likely already caught by developers, auditors, or users.

Good targets tend to be blockchains, large DeFi protocols, math-heavy systems, projects with many components, and systems with external or cross-chain integrations. Bridges and diamond proxy setups are classic examples.

#### Innovation

**Complexity hides bugs, but innovation creates space for them.**

Innovation can appear at multiple layers. Sometimes it is conceptual, with new designs or specifications. Other times it is an implementation of an existing idea in a new way.

When a project takes a new approach, there is a good chance that some attack path was not fully considered. New attack surfaces will keep appearing around new industries like RWAs, new blockchain designs, zero-knowledge systems, privacy tech, and AI integrations.

Implementation-level innovation matters too. Teams experiment with new yield mechanics, consensus tweaks, execution optimizations, or porting contracts to new languages. These changes often introduce subtle bugs, especially when language or VM behavior is misunderstood.

Ecosystem maturity matters. Large ecosystems tend to harden over time. New chains, uncommon languages, or unusual execution environments tend to have more basic issues simply because fewer people have checked them yet.

##### Optimization

**Many bugs originate from optimization.**

Gas optimizations in contracts, heavy use of assembly, manual memory management, rewritten math expressions, or performance-focused changes in blockchains and large systems all add extra layers of complexity. Even refactoring code for clarity or efficiency can introduce entirely new classes of bugs. Watch out for these patterns, especially in updates.

These changes often hide edge cases the developer did not anticipate, or introduce subtle oversights related to obscure behaviors of virtual machines, languages, or execution environments.

#### Code Quality and Audits

**Code quality tells a story.**

Bugs are more likely when developers rush releases or lack perfectionism. Poor code is also harder to secure. Even non-functional signals like sloppy comments or inconsistent naming often correlate with deeper issues.

Ignoring best practices, making beginner mistakes, or misunderstanding basic security patterns like checks-effects-interactions are all warning signs. Any project that still falls below a basic standard is a strong vulnerability candidate.

Audit reports are useful context. If an audit found many basic issues, that signals weak implementation. Multiple Critical findings are a serious red flag.

Fixing many issues requires many code changes, which increases the chance of introducing new bugs. Auditors miss these for many reasons: limited time, correlated changes, ongoing development, fatigue. No audit makes a system safe enough.

Based on how solid a project feels, I adjust my focus:

- **Good codebase with good audits:** Novel or very complex exploit paths, upgrades, operational or configuration issues, assets indirectly affected.
- **Good codebase with average audits:** Complex paths and known security pitfalls auditors catch more often than developers.
- **Average codebase with good audits:** Audit fixes and leftover weak design.
- **Average codebase with average audits:** Missed but not extremely complex exploit paths.

---

## Being First

Even if a vulnerability exists, you still need to find it before someone else.

### Time Since Launch

**Right after launch, basic bugs surface quickly.**

Auth mistakes, common attack vectors, operational errors, and cheap exploits are often found within hours or days. Black hats monitor new deployments closely. If I hear about a launch early, I speedrun these checks.

In the first weeks or months, more complex attack paths missed by audits are often discovered. This is a good window for deeper reviews, especially after upgrades.

The start of a bounty program matters too. Early on, competition is intense. Many strong hunters focus there. I try to check early, then come back later for a deeper pass.

Over time, ROI drops. Only the first report gets paid.

### How Many Eyes Have Looked at the Code

**More eyes means the remaining exploit paths are harder.**

This includes auditors, bounty hunters, developers integrating the code, and teams maintaining forks. Popular projects harden over time. Some ecosystems even share fixes privately.

Projects with few users, obscure chains, or no active bounties are often easier targets.

High TVL projects attract sophisticated attackers willing to invest heavily. If they are already active, remaining bugs are likely very complex.

The key is not doing what everyone else is doing. Look at neglected parts, uncommon chains, older programs, or code paths people avoid.

---

## How I Actually Find Bugs

**Finding real exploits is not the same as flagging suspicious code.**

Anyone can point at an anti pattern. Static analyzers and AI tools do that constantly. Developers often already know about these issues, or they were downgraded during audits.

Finding real Critical attack paths requires deep technical skill and experience.

I go deep, but not like a full audit. I focus on critical paths and constantly ask what assumptions must hold for this to be safe.

Most systems make no sense at first. I research unfamiliar concepts, read docs, and build understanding slowly. I might work for a few days, then come back weeks or months later. I do not need full understanding to find an exploit. I just need enough.

I have found severe vulnerabilities in systems I did not fully understand. Not simple bugs, but complex paths. I ignored most of the code and obsessed over one execution path, checking every assumption and branch.

This builds intuition over time.

Coming back later helps. I see the code differently. I may have learned new techniques, or the system changed. Sometimes I walk through attack paths away from the screen.

This depth helped me catch the [Scroll](https://forum.scroll.io/t/report-scroll-mainnet-emergency-upgrade-on-2025-04-25/666) issue. A small change stood out because I already had a strong mental model of the system. Without that context, it would have been easy to miss.

Another public finding from [Story](https://www.story.foundation/blog/story-network-postmortem) followed the same pattern. I knew the codebase after an audit competition. A change enabled an attack path I had already considered. Testing it was straightforward.

Depth also means researching ideas across many systems. How does liquidation work here versus elsewhere? Was this copied from another project? Context often gets lost during copying.

Time is limited. Research is unpaid. Every time decision matters. I focus on systems that are complex enough to hide real bugs and pay meaningful rewards. Even when I find nothing, I want the work to carry forward.

### Hunter Archetypes

If I had to put my work into some archetypes, it would fit into:

- **The Digger:** Goes deep on a single program.
- **The Differ:** Compares one mechanism across many projects.
- **The Speedrunner:** Reviews new programs immediately.
- **The Watchman:** Monitors deployments and upgrades.
- **The Lead Hunter:** Develops ideas around less known vulnerability types.
  - **The Scavenger:** Gets inspired by obscure writeups or little known incidents.
- **The Scientist:** Builds tools for monitoring and analysis. Possibly AI in the future.

### Mindset

Your mental state matters as much as your technical skill.

I work best in short, intense bursts. A few hard days, then rest. Hunting is not a normal job, but it should not consume your life either. Stress kills clarity. Some pressure motivates. Feeling cornered destroys performance.

Pick projects that genuinely interest you. Motivation makes deep work possible.

---

## Impact and Rewards

Professionally, it makes sense to focus on Critical issues that lead to real loss of funds.

Bounties follow a [power law](https://en.wikipedia.org/wiki/Power_law). One Critical can be worth dozens of Highs. I still report lower severity issues when I find them, but I do not hunt them deliberately.

Always quantify impact. Identify assets at risk and show realistic attack paths. A Critical that only loses trivial amounts will not be treated as meaningful.

Reward size mostly reflects what projects think will attract hunters. Attackers do not care. Draining a low cap protocol is not much harder than draining a high cap one. So, time is better spent on programs with higher caps.

---

## Getting Paid Fairly

Many new hunters assume that if they report a real exploit, they will be paid fairly. That is often wrong.

Once you disclose, you lose leverage. The best defense is reducing risk before you invest time.

### Project Risk Assessment

Before hunting seriously, evaluate:

- **Reputation:** How established is the project? Do people know it? Any past bounty stories?
- **Treasury:** Can they actually afford to pay?
- **Bounty rules:** Clear rewards, clear scope, fair caps.
- **Security history:** Prior payouts, public advisories, dedicated security staff, audit depth.

If I find a bug, I report it and see how the process goes. Only after trust is established do I look for more.

Red flags include vague rules, very low caps, prior disputes, or lack of response.

Always archive the rules before submitting.

### Negotiation and Lowballing

After disclosure, delays, pushback, and misjudged severity are common. Stay calm, reference the rules, and provide proof. Mediation can help, but only if the project cooperates.

Bad signs include rule changes, long silence, or claims of prior knowledge without proof.

Getting scammed happens. It is draining. The only real fix is avoiding those projects in the first place.

### What Not to Hunt

You should avoid:

- Deprecated or unused code.
- Old contract implementations behind proxies.
- Repository code that does not match deployed code.
- Non live or incompatible chain versions.
- In development code with no proof it will ship.

I also do not actively hunt for:

- Temporary DOS or griefing.
- Issues gated on third party conditions.
- Bugs possibly mitigated by offchain systems.
- Low severity issues that lead to endless debates.

Also, some things you should not report:

- Issues you cannot prove.
- Pure informational or architectural suggestions.
- Issues requiring trusted roles.
- Spam or beg bounties.

I focus on Critical issues affecting real assets. Fair projects reward them even if scope definitions are imperfect.

### Forks and Shared Libraries

Rules around shared code are often hostile to hunters. Some projects refuse to pay for bugs in dependencies that directly affect them.

Always read scope and rules carefully. Sometimes the original library offers lower or no bounties. Bureaucracy at its finest.

### Platforms

**No platform solves fairness completely.**

I evaluate platforms by payout history, dispute handling, neutrality, and how they respond to project misconduct. Platforms exist to speed up contact and provide mediation. If a platform adds no value, direct disclosure is sometimes better.

My personal picks are Immunefi, HackenProof, and Cantina.

#### Self Hosted and No Program Cases

Self hosted programs rely entirely on project honesty. Clear rules matter more than hosting. Projects without explicit programs vary widely. Some pay generously. Others never will.

If you engage, be professional, realistic, and fair. Do not enforce rewards that were never promised. A quick vibe check on Discord or public channels can save weeks of wasted effort.

---

## Acknowledgements

Thanks [0xdeadf4ce](https://x.com/0xdface), [lonelysloth](https://x.com/lonelysloth_sec) for your suggestions. I updated the post with some of your ideas.

Also check my favorite related articles, and sources of inspiration:

- [4 Strategies for picking the perfect bounty hunting targets](https://joranhonig.nl/blog/2022-02-13-4-bug-hunting-target-strategies) - by [joran](https://x.com/joranhonig)
- [Bug bounty, feedback, strategy and alchemy](https://zhero-web-sec.github.io/thoughts/bugbounty-feedback-strategy-and-alchemy) - by [zhero](https://x.com/zhero___)

---

## Final Thoughts

Hunting lets you explore systems you care about and sharpen how you think. Just be deliberate with risk, time, and expectations.

Happy hunting.
