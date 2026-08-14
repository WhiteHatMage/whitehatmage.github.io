---
title: Web3 Bug Hunting Guidelines
categories: [Hunting]
tags: [hunting]
pin: true
image:
  path: /assets/img/mage-forest-white.jpg
  alt: Teaching white magic - by Grok Imagine
---


## The Hunter's Spirit

Hunting requires you to choose your targets wisely, be patient throughout the process, and resilient in the face of unfair outcomes.

The process is slow. It can be months before you find your first vulnerability, report it, and get paid. You have to be mentally prepared for it, and also have sufficient savings before going hunting full-time.

The most important thing is that you shouldn't give up early.

## First Steps

If it's your first time hunting, don't overcomplicate it. Pick any project you like, and try to find live vulnerabilities in it. You'll likely fail miserably, but that's okay. Projects are tightly secured, pass many security audits, and do all they can to prevent exploits. Your first goal should be to get the gist of how it feels to hunt live projects.

You should get familiar with contract scanners, transaction flows, asset flows, and actual live risks -- understand the difference between [bugs, vulnerabilities, and exploits](https://x.com/Montyly/status/1986777119234351457).

Projects will only pay good rewards for real-life exploits you can prevent.

## Hunter Strategies

Set a hunting goal. Let it be money, learning, or anything you want. That will define the strategy you should choose. You shouldn't be hunting on low-capped programs if your goal is getting $1 Million in rewards. You shouldn't pick things that are just trendy if your goal is learning.

Bounty hunting rewards the first person who reports an exploit. You’ll have to develop some unique abilities to be that person. Just don’t do what everyone else is doing, and come up with new ideas.

### Archetypes

I split my work into different persona archetypes like [zhero](https://zhero-web-sec.github.io/thoughts/bugbounty-feedback-strategy-and-alchemy) does.

- **The Miner**: Goes deep into one project at a time. I only spend time in paths that can lead to critical events, and I know them by heart. I analyze all branches, dependencies, language gotchas, even some compiler outputs. I focus on one idea at a time, but I always make sure I have full context of the graph.
- **The Differ**: Compares one mechanism across many projects. Instead of checking one project in depth, I compare how different projects solve the same problem. You might find some interesting ideas that other teams missed.
- **The Speedrunner**: Reviews new programs immediately. I used to do this because I’m fast at onboarding. I think this is now best suited for hunters working with automated setups.
- **The Watchman**: Monitors deployments and upgrades. This can be well suited for both human hunters and AI scanners. If you’re manually checking updates, you should pick the projects you’re most familiar with, as you’ll have an advantage over any tools and other hunters. If you’re trying to automate it with AI, you’ll likely catch the low-hanging fruit, since you can’t spend too much money on long runs for every single change of every project. So, find a wise balance.
- **The One-Day Hunter**: Develops ideas around less-known vulnerability types. I may get inspired by obscure writeups, little-known exploits, or CVEs of languages or libraries used by projects. For example, a vulnerability affecting a Go version might lead to an unknown bug in a library used by a blockchain client.

A note on LLMs:

Due to their small context windows, LLMs fail at analyzing complex ideas and paths accurately. We sometimes call them hallucinations, but it’s just their inability to reason. Nevertheless, many hunters have had great success using LLMs to their advantage. Tokens are cheap and subsidized at the moment, so it’s a great time to spend as many of them as possible to produce as many leads as you can -- if you choose to go for the *security-triager-hunter* path. 

If I were to use them in my workflows, I’d consider them as a non-deterministic version of static analyzers. 

### The Biomes of Bugs

Live bugs are usually simple. They hide in complex paths. If you look at fixes, many involve just a single missing check. Simple codebases shouldn't have any exploit paths. You're most likely to find good vulnerabilities in blockchains, large DeFi protocols, math-heavy systems, and projects with many integrations.

You should also look for innovation. Whenever a project takes a new approach, there is a good chance that some attack path was not fully explored.

Many bugs also originate from optimizations. Some changes often hide edge cases the developers did not anticipate.

Check code quality and code maturity. Bugs are more likely when there are signs of sloppy code or developers ignoring best practices.

Audit reports also contain useful data. You can get an idea of the risk the project faces by the number of critical and high-severity issues. Multiple simple issues indicate a lack of attention. Those are indicators of other unknown vulnerabilities that could still exist in the codebase.

Review the fixes. It often happens that projects introduce new issues when fixing old ones.

Depending on your evaluation of the codebase, you should apply different tactics. Look for low-hanging fruit in sloppy codebases. Focus on the most complex paths on clean projects.

### Competition

Projects are more likely to have vulnerabilities right after launch, or when they first start hosting a bounty program. As time goes on, you should focus on looking for the most complex attack paths and novel ideas. If you decide to look for vulnerabilities in new programs, do it as soon as possible. There's a lot of competition, and people are using automation more and more these days.

You should also consider your *competitors* -- anybody who may discover the issue before you. This includes other hunters, developers, LLMs, other projects, and attackers. The more popular a project is, the less likely you'll find simple exploits. Unpopular projects are often easier targets.

Check neglected chains, contracts, old programs, or anything competitors might avoid, like complex or unknown paths in popular projects.

### What Not to Hunt

You should avoid deprecated or unused code, as well as in-development code that doesn't match the live version. That often leads to no rewards since there's no live risk.

I also do not actively hunt for temporary DoS or griefing, since projects don't care much about those impacts. I'd also avoid other low-severity issues.

Beware of special clauses regarding third-party libraries and out-of-scope components. Also be mindful of issues that require trusted roles. They are usually not eligible for rewards.

Don't report issues you cannot prove. That doesn't work.

## Getting Paid

Some hunters assume that they will be paid fairly whenever they report a real exploit. That is often wrong.

You lose any leverage once you disclose an issue. It's best to choose your targets wisely. Before hunting, evaluate the project reputation, any bounty stories, the project treasury, and the bounty program rules. Some red flags include vague rules, prior disputes, low caps, or lack of response.

After disclosure, you should be prepared for delays and pushback. Stay calm and continue your hunting session. Address each report with professionalism.

---

## Acknowledgements

Thanks [0xdeadf4ce](https://x.com/0xdface), [lonelysloth](https://x.com/lonelysloth_sec), [aviggiano](https://x.com/aviggiano) for your suggestions.

This post was also inspired by [joran's blog](https://joranhonig.nl/blog/2022-02-13-4-bug-hunting-target-strategies).

Happy hunting.
