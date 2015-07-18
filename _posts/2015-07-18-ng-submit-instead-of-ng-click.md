---
layout: post
title: Clever Hack to Respect Keyboard Access in User Forms
---

Be honest. Do the input fields in your app have broken submit-on-Enter support?

Chances are, if you are working on an AngularJS codebase, you are posting data via AJAX requests. And chances are, some of that data comes from user forms. What do we mean by forms? Some examples:

* login form
* signup page for new users
* single-input in-place property editor widget
* anything with an `<input>` element, basically!

If those form inputs do not submit form data on when the Enter key is pressed, that's a cardinal sin that will frustrate every user you'll have! It's only slightly less grating than broken back-button support.

Folks type, then they hit Enter. If the Enter key does not do anything, they are forced to reach for their mouse or to tab over to the submit button: an interruption in their flow, and a blemish on the impression of subtle quality that you might be striving for. Not every form will have this expectation, for sure, but settling for anything but consistent support across the board might confuse and annoy users even further!

Now, if your input elements *do* support submit-on-Enter, is it implemented the simplistic way, or the clever way that will save your time and sanity over the project's lifetime?

Let's look at the typical markup offered by the average AngularJS form tutorial out there:

```html
<label>
    First Name
    <input type="text" ng-model="data.firstName" />
</label>
<label>
    Last Name
    <input type="text" ng-model="data.lastName" />
</label>
<label>
    Email
    <input type="text" ng-model="data.email" />
</label>

<button ng-click="signup(data)">Sign Up</button>
```

Our example assumes a scope with a `data` object that stores the collected input, and a `signup` function that submits the AJAX request.

The obvious flaw so far is the one we discussed: lack of support for submit-on-Enter. How do we fix that?

Here's our naïve fix:

```html
<label>
    First Name
    <input type="text" ng-model="data.firstName" ng-keypress="$event.which === 13 &amp;&amp; signup(data)" />
</label>
<label>
    Last Name
    <input type="text" ng-model="data.lastName" ng-keypress="$event.which === 13 &amp;&amp; signup(data)" />
</label>
<label>
    Email
    <input type="text" ng-model="data.email" ng-keypress="$event.which === 13 &amp;&amp; signup(data)" />
</label>

<button ng-click="signup(data)">Sign Up</button>
```

We simply added a key-press handler on each field. In every one of them, we check for the Enter key-code (`13`) before calling the AJAX submit function. Yes, it works, but the code is unwieldy (even if we wrap it in a custom directive like `ng-enter`), and we are liable to forget to add it to future form inputs. Plus, there is always a chance that we are stepping on some browser- or input-specific behaviour - such as Shift-Enter hooks in certain text fields that should *not* submit the form.

Let's try again. Can we keep things [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself), improve interaction support and rescue the universe from ancient evil?

Our final attempt:

```html
<form ng-submit="signup(data)">
    <label>
        First Name
        <input type="text" ng-model="data.firstName" />
    </label>
    <label>
        Last Name
        <input type="text" ng-model="data.lastName" />
    </label>
    <label>
        Email
        <input type="text" ng-model="data.email" />
    </label>

    <button type="submit">Sign Up</button>
</form>
```

First, we wrapped the entire thing in a `<form>` element. Then we moved the button's `ng-click` handler to be the form element's `ng-submit` handler. No other handlers needed. Yep, it's that obvious.

Every browser already has built-in handling of submit-on-Enter. It is automatically present on any input element (including special ones like `textarea`) that is inside a `<form>` element. We just need to intercept the latter's `submit` event to perform our custom AJAX data submission.

The `<form>` element has originally been used to implement traditional HTTP POST (or GET) data submission - that is why it's not obvious to consider it in an app that relies on AJAX instead. But the extra richness of behaviour that it brings is a clear vote for keeping it around in modern web apps, too. In fact, AngularJS creators foresaw this and made it so that [`ng-submit` prevents legacy HTTP submit](https://docs.angularjs.org/api/ng/directive/ngSubmit) for the typical AJAX form (which would otherwise clobber and reload the entire page). So this is actually the recommended way to implement AJAX form submission behaviour.

In fact, this does not even have to be used for AJAX. Any situation where the interface involves textual input with an action trigger can be implemented using the above solution.

Some caveats:

* there should be some kind of submit button (`<button type="submit">` or `<input type="submit">`) in the form element, otherwise certain browsers ignore Enter-key
* make sure that the form has no `action` attribute - if it does exist the `ng-submit` directive will let a legacy HTTP submit happen, refreshing the entire page

An extra benefit that we haven't mentioned so far is that adding this `<form>` wrapper will also improve mobile behaviour! Playing nicely towards browser expectations pays off in many ways.

Next challenge: form behaviour while waiting for AJAX results (animated loading spinner, disabling double-submit). Stay tuned.
