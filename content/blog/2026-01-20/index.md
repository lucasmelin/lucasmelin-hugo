---
date: "2026-01-20"
title: "Using Shiki for syntax highlighting with mdsvex"
tags: ["svelte", "css"]
summary: "How to integrate `Shiki` with `mdsvex`."
slug: shiki-mdsvex
---

By default, [`mdsvex`](https://mdsvex.pngwn.io) uses [`Prism`](https://prismjs.com/) for syntax highlighting which works well for basic use cases. However, in some cases you might find that you need something more powerful. The [mdsvex docs](https://mdsvex.pngwn.io/docs#with-shiki) provide a basic description of how to use [`Shiki`](https://shiki.style/) for highlighting instead, but there's no documentation showing how to enable `Shiki`'s more advanced features such as [transformers](https://shiki.style/packages/transformers). This quick walkthrough will show you how.

To start, you're going to want to the `shiki` and `@shikijs/transformers` packages to your project:

```sh
npm i -D shiki @shikijs/transformers
```

Next, open up your `svelte.config.js` file to modify the `mdsvex` config:

```js
import { createHighlighter } from 'shiki';
import { transformerMetaHighlight } from '@shikijs/transformers';

// You can use any theme found here: https://shiki.style/themes
const theme = 'catppuccin-mocha';
const highlighter = await createHighlighter({
	themes: [theme],
    // Enable any language grammars you want to use.
    // The full list of available grammars can be found here:
    // https://shiki.style/languages
	langs: [
		'css',
		'gdscript',
		'go',
		'hcl',
		'html',
		'java',
		'javascript',
		'json',
		'python',
		'shellscript',
		'toml',
		'typescript',
		'yaml'
	]
});

const config = {
	preprocess: [
		vitePreprocess(),
		mdsvex({
			extensions: ['.md'],
			highlight: {
                // It's crucial that we pass the meta parameter to the shiki
                // highlighter if we want transformers such as line highlighting
                // to work, since they use the metadata found in the codeblock.
				highlighter: async (code, lang, meta) => {
					const html = escapeSvelte(
						highlighter.codeToHtml(code, {
							lang,
							theme,
                            // This transformer provides support for highlighting
                            // specific lines in code blocks.
                            // For a list of common transformers, see:
                            // https://shiki.style/packages/transformers
							transformers: [transformerMetaHighlight()],
							meta: { __raw: meta }
						})
					);
					return `{@html \`${html}\` }`;
				}
			}
		})
	],

	kit: {
		adapter: adapter()
	},

	extensions: ['.svelte', '.svx', '.md'],
};
```

In the above example, we're using `transformerMetaHighlight` so that we can highlight specific lines in our code blocks.
This will cause `Shiki` to emit lines that look like `<span class="line highlighted">`, but we still need to handle the CSS styling ourselves.

> [!Note]
> Depending on how your CSS is loaded, you may need to use `!important` to ensure that your CSS styles are not overridden by the default `Shiki` styles after the page loads. This happens because `Shiki` applies its styles dynamically via JavaScript, which can overwrite preexisting page styles.

If you're using [Tailwind](https://tailwindcss.com/), one way to do this is by opening your `tailwind.config.js` file and adding a new style for the `pre code span.highlighted` selector. As a reference, here's the relevant snippet from my config:

```js
module.exports = {
	darkMode: 'class',
	theme: {
		extend: {
			typography: ({ theme }) => ({
				DEFAULT: {
                    'pre code span.highlighted': {
                        backgroundColor: `${theme('colors.secondary.300 / 0.2')} !important`,
                        transition: 'background-color 0.5s',
                        margin: '0 -24px',
                        padding: '0 24px',
                        width: 'calc(100% + 48px)',
                        display: 'inline-block !important'
					}
				}
            })
        }
    }
}
```
