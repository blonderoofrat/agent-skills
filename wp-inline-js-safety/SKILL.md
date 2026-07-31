---
name: wp-inline-js-safety
description: "Stop WordPress silently corrupting inline JavaScript. wptexturize converts an ampersand caught between a < and a > into an HTML entity, which kills the entire <script> block with no PHP error and no server-side warning. Use whenever emitting or authoring inline JavaScript that reaches WordPress content filters, and whenever inline JS 'just stops working' on one page while an enqueued file is fine."
---

# WordPress silently breaking your inline JavaScript

A page that looks completely normal until something on it does not respond, with nothing wrong on the
server side at all.

## The failure

### The symptom

An inline `<script>` stops executing. The browser console shows a syntax error at a line that looks
fine in your source. No PHP error, no warning, no notice. View source and you find:

```js
if ( a < b &#038;&#038; c > d ) {
```

Your `&&` became `&#038;&#038;`, and the whole script block died.

### Why it happens

`wptexturize` is WordPress's typographic filter. It runs over post content and encodes stray
ampersands into `&#038;`: sensible for prose. It deliberately skips the *contents* of `<script>` and
`<style>` elements, and on ordinary script content it does exactly that: a lone `&` inside a
`<script>` is left alone.

The trap is that the skip is driven by splitting the content on angle-bracket spans. When your
JavaScript contains a `<` and a later `>` as **comparison operators**, the split misreads the text
between them, and an `&&` caught inside that span is encoded like any other stray ampersand.

> **Verified, not assumed.** Run against the real `wptexturize` from WordPress 6.4:
> `<script>if ( a < b && c > d ) { x(); }</script>` comes back with `&#038;&#038;`, while
> `<script>if ( a && b ) { x(); }</script>` and `<script>var s = "a & b";</script>` are returned
> unchanged. So it is specifically the `&&`-between-`<`-and-`>` shape, not ampersands in scripts
> generally. **Check it on your own version before relying on the workaround**: this is filter
> behaviour, it has been rewritten before, and one two-line script settles it for you.

So the ingredients are specific and easy to hit by accident:

```js
if ( i < len && total > 0 ) { ... }
//     ↑           ↑↑          ↑
//     <  ...      &&  ...     >     -> the && sits between a < and a >
```

One `&&` in one comparison, and the entire block is gone. `||` is safe: there is no ampersand.

### The fixes, in order of preference

**1. Do not put behavior JS in filtered content at all.** JavaScript typed into the editor, and
anything else that passes through the content filters, is texturized. Enqueue a real file, or emit
from a hook outside the content pipeline (`wp_footer`, `wp_head`, `admin_footer`), gated to the pages
that need it. This removes the class, not the instance.

*(Shortcode **output** is comparatively safe: with default priorities `do_shortcode` runs at 11, after
`wptexturize` at 10, so what a shortcode returns is not texturized. Do not let that reassure you about
JS sitting in the post body around it, which is.)*

**2. Write comparisons so the pattern cannot form.** If the JS must be inline, avoid a `<` before a
later `>` on the same span:

```js
if ( len > i && total > 0 )      // flip a < b  ->  b > a
```

Consistently using `>` means no `&&` is ever caught between a `<` and a `>`.

**3. Avoid the literal characters.** `&amp;&amp;` is not equivalent inside a script. Prefer
restructuring: nested `if`s, an early `return`, a boolean variable, over escaping games.

### How to actually verify it

**Checking the PHP source proves nothing**: the corruption happens after your code runs. Check the
*delivered* page.

- Fetch the rendered HTML and search for `&#038;` inside `<script>` blocks. Any hit is a break.
- Better: run the same filter over your emitted markup in a test, and assert the output is unchanged.
  That catches it before deploy rather than after.
- Best: load the page in a real browser in CI and fail on any `pageerror`. A texturized script throws
  a syntax error the moment it parses, so this catches the whole class including variants nobody has
  characterized yet.

---

## The rule underneath it

**A rendered page is the only evidence that a page works.** This failure is invisible to source
inspection, to PHP linting, to unit tests, and to a `200 OK`. The response is delivered, the status
is fine, and the JavaScript is dead.

If your verification never loads a real page in a real browser, it cannot see this, and will report
success while a control on that page does nothing at all.
