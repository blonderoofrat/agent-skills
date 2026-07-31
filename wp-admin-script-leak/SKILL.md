---
name: wp-admin-script-leak
description: "Find WordPress JavaScript errors that appear ONLY for logged-in users, caused by admin-context scripts loading on front-end pages. Use whenever a public page throws a console error that vanishes when you log out, and whenever registering script enqueues -- a logged-out check cannot see this class of failure at all."
---

# Admin scripts leaking onto front-end pages

## The symptom

A JavaScript error on a public page, visible **only when you are logged in**. Log out, and the page is
clean. Typically a `TypeError` complaining that some property of `undefined` cannot be read, from a
script you never intended to load there.

That asymmetry is the whole tell, and it is why this survives: **the person most likely to see it is
the developer, and the person least likely to see it is a visitor**, so it gets dismissed as
"something odd in my browser" rather than filed as a bug.

## Why it happens

An admin-context script gets registered somewhere that runs on the front end. Admin scripts assume
admin globals, admin DOM structure, and dependencies that a public page does not load. Without them,
they throw on initialisation.

The usual cause is enqueuing from a hook that fires everywhere: `init`, or a shared setup function,
rather than from the context-specific hook. It can also come from a plugin doing this on your behalf,
which is worth checking before assuming the code is yours.

## The fix

Use the hook that matches the context, and do not rely on a runtime check to sort it out:

```php
add_action( 'admin_enqueue_scripts', function () {
    // admin screens only -- this hook does not fire on the front end
} );

add_action( 'wp_enqueue_scripts', function () {
    // front end only -- and gate further by page/template as needed
} );
```

**A common non-fix:** calling `is_admin()` inside `wp_enqueue_scripts`. That hook does not fire on
admin screens at all, so the check can never be true and the guard is dead code. It looks like
protection in review and provides none. (`is_admin()` also means "an admin *screen* is being
rendered", not "the current user is an administrator": confusing those two is its own bug, and a
security-relevant one.)

## How to verify

**Load a front-end page as a logged-in user with elevated capabilities, and assert the console is
clean.** A logged-out check will pass and prove nothing, because the offending script is not sent to
logged-out visitors.

This is the entire reason the class persists: the standard verification, fetch the page, check the
status, maybe check the markup: cannot see it. Only a real browser session as a privileged user can.

Worth knowing when you read the error: an uncaught exception kills the rest of *that* script block.
Other `<script>` elements still run. The damage downstream is indirect: the crashed script never
finished setting up, so anything depending on it silently does nothing.
