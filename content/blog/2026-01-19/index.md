---
date: "2026-01-19"
title: "A social filesystem"
tags: ["at-protocol"]
summary: "Dan Abramov wrote an excellent piece on how the AT Protocol works."
slug: a-social-filesystem
---

[A social filesystem](https://overreacted.io/a-social-filesystem/): [Dan Abramov](https://danabra.mov/) wrote an excellent piece on how the [AT Protocol](https://atproto.com/) works. It walks you through step-by-step, showing how an application's data is stored in files and folders, which then apps such as [BlueSky](https://bsky.app/) or [Tangled](https://tangled.org/) can aggregate and render. I'd been curious about the AT Protocol for a bit, but Dan's article really made the idea click for me.

Since the apps and data are separate, this means is that if an app were to shut down, anyone could put the app (or at least a similar app) back online with the existing content. It also allows developers to create apps that combine information from multiple sources. This approach also made me think of the "[file over app](https://stephango.com/file-over-app)" philosophy from [Steph Ango](https://stephango.com/about), the CEO of [Obsidian](https://stephango.com/obsidian).