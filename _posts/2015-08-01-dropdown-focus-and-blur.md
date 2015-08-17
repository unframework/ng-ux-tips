---
layout: post
title: Dropdown Focus and Blur
---

Everyone has to implement a dropdown at some point. But there is a surprising amount of complexity needed to make it feel not broken. It's trivial to add a simple menu toggle, but will it respond correctly when the user "clicks away"?

<!--more-->

Let's start with a simple menu:

```html
<html ng-app="app">
    <head>
        <script src="//ajax.googleapis.com/ajax/libs/angularjs/1.2.25/angular.min.js"></script>
        <style>
            .dropdown-body {
                height: 0; /* overlay surrounding content */
                overflow: visible;
            }

            .dropdown-body a {
                border: 1px solid #444;
                display: block;
            }
        </style>
    </head>

    <body ng-controller="Main">
        <script>
            angular.module('app', [])
            .controller('Main', function ($scope) {
                $scope.dropdownIsOpen = false;

                $scope.toggleDropdown = function () {
                    $scope.dropdownIsOpen =
                        !$scope.dropdownIsOpen;
                };
            });
        </script>

        <div>
            <button ng-click="toggleDropdown()">Choose Action</button>
            <div ng-if="dropdownIsOpen" class="dropdown-body">
                <a href="#">Action Alpha</a>
                <a href="#">Action Bravo</a>
                <a href="#">Action Charlie</a>
            </div>
        </div>
    </body>
</html>
```

The user opens the dropdown by clicking (or hitting `Enter`) on the "Choose Action" button. Then they click the dropdown element, and so on.

But what if they choose not to do anything? The user might scan the action list and decide to click on something else on the screen instead. The dropdown menu will be "stuck" there, which is not consistent with how most native implementations would behave in that case. The user can, of course, click the toggle button again, but we can do better than that!

Right off the bat, let's not even think of simply attaching a `click` handler to the `body`. There are many ways to lose focus, and some do not even involve user input (e.g. an app alert popup designed to "steal focus"). We need to use a deeper, more comprehensive hook.

That hook, of course, is the `blur` event that the browser generates on any focus change:

```html
<html ng-app="app">
    <head>
        <!-- ... -->
    </head>

    <body ng-controller="Main">
        <script>
            angular.module('app', [])
            .controller('Main', function ($scope) {
                $scope.dropdownIsOpen = false;

                $scope.toggleDropdown = function () {
                    $scope.dropdownIsOpen =
                        !$scope.dropdownIsOpen;
                };

                $scope.leaveDropdown = function () {
                    $scope.dropdownIsOpen = false;
                };
            });
        </script>

        <div>
            <button ng-click="toggleDropdown()" ng-blur="leaveDropdown()">Choose Action</button>
            <div ng-if="dropdownIsOpen" class="dropdown-body">
                <a href="#">Action Alpha</a>
                <a href="#">Action Bravo</a>
                <a href="#">Action Charlie</a>
            </div>
        </div>
    </body>
</html>
```

We added an `ng-blur` on the trigger button that hides the dropdown when that element loses focus. It makes sense: the trigger already has focus, so the browser will always send a `blur` event when the user clicks on something else or is interrupted by an external activity.

But this doesn't actually work! When the user clicks on a dropdown menu item, or uses the keyboard to select it, the browser notifies our trigger `blur` handler first, which hides the menu and in turn prevents any further action. Oops!

What we want to track is a sort of a "compound focus". In other words, we consider the dropdown component to be still active if *either* the trigger, or any of its menu items hold user focus. We only want to process a `blur` event when the focus truly leaves all of those elements.

Writing that behaviour "by hand" is actually quite onerous, especially due to browser compatibility issues interfering with the nitpicky and subtle nature of the user interaction.

This is where the [angular-deep-blur](http://myplanet.github.io/angular-deep-blur/) directive comes in very handy (installable via Bower or NPM):

```html
<html ng-app="app">
    <head>
        <!-- ... -->
    </head>

    <body ng-controller="Main">
        <script>
            angular.module('app', [ 'mp.deepBlur' ])
            .controller('Main', function ($scope) {
                $scope.dropdownIsOpen = false;

                $scope.toggleDropdown = function () {
                    $scope.dropdownIsOpen =
                        !$scope.dropdownIsOpen;
                };

                $scope.leaveDropdown = function () {
                    $scope.dropdownIsOpen = false;
                };
            });
        </script>

        <div deep-blur="leaveDropdown()">
            <button ng-click="toggleDropdown()">Choose Action</button>
            <div ng-if="dropdownIsOpen" class="dropdown-body">
                <a href="#">Action Alpha</a>
                <a href="#">Action Bravo</a>
                <a href="#">Action Charlie</a>
            </div>
        </div>
    </body>
</html>
```

We put the `deep-blur` handler on the parent element of the entire dropdown and removed the `ng-blur` on the trigger element.

The `deep-blur` directive only fires the blur callback when the user focus completely leaves the parent and all of its children; if the user switches focus between elements "under" the directive's element, no blur callback is triggered. This is why it is called "deep" blur - this directive is aware of the entire depth of the HTML elements under it. The [angular-deep-blur project homepage](http://myplanet.github.io/angular-deep-blur/) has a live demo that uses a dropdown menu as an example (not unexpectedly!).

The above example is now very close to the expected "native-style" behaviour of dropdowns with regard to clicks and keyboard focus.

Responding when the user clicks away from a dropdown menu is a subtle but important part of expected UX; not implementing that behaviour is a kind of omission that stumps the user flow and lowers the perceived quality of the entire interface. Implementing it is also surprisingly non-trivial, but the [angular-deep-blur](http://myplanet.github.io/angular-deep-blur/) removes a lot of the complexity of the task.
