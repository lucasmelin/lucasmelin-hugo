---
date: "2022-04-20"
title: "Selecting elements matching one of the conditions in Playwright"
tags: ["playwright", "til"]
summary: "In working with Playwright, sometimes you need to search for an element like a button or dropdown that might have one of several attributes."
slug: playwright-select-conditions
---

In working with [Playwright](https://playwright.dev/), sometimes you need to search for an element like a button or dropdown that might have one of several attributes.

The `:is()` pseudo-class is an [experimental CSS pseudo-class](https://developer.mozilla.org/en-US/docs/Web/CSS/:is). It is a function that takes a selector
list as its argument, and selects any element that can be selected by one of the selectors in that list. This is useful for writing large selectors in a more
compact form.

For example:

```python
@when('I select any reason from the dropdown')
def step_impl(context):
    page = context.browser.page
    reason_one = "Reason One"
    reason_two = "Reason Two"
    page.click(f":is([data-assoc-field={reason_one}], [data-assoc-field={reason_two}]) > a")
```