---
date: "2026-01-18"
title: "Adding CSS classes to the body tag of a Svelte project"
tags: ["svelte", "css", "til"]
summary: "How to add CSS classes to the body tag of a Svelte project."
slug: svelte-body-tailwind
---

When working on a [Svelte](https://svelte.dev/) or [Sveltekit](https://svelte.dev/docs/kit/introduction) project, there's no way to directly add classes to the `<body>` or `<svelte:body>` tags. There's an [open issue on GitHub](https://github.com/sveltejs/svelte/issues/3105) discussing this further, which includes a few workarounds from community members.

I found that for me, the easiest way to add custom classes was by using a [global style selector](https://svelte.dev/docs/svelte/global-styles).

For example, here's an example showing how you could modify the background color of a page using [Tailwind](https://tailwindcss.com/docs/installation/framework-guides/sveltekit) classes.

```svelte
<style lang="postcss">
	@reference "tailwindcss";
	:global(body) {
		@apply bg-neutral dark:bg-neutral-800;
	}
</style>
```