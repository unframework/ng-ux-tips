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
            angular.module('app', []).controller('Main', function ($scope) {
                $scope.dropdownIsActive = false;
                $scope.toggleDropdown = function () {
                    $scope.dropdownIsActive = !$scope.dropdownIsActive;
                };
            });
        </script>

        <button ng-click="toggleDropdown()">Choose Action</button>
        <div ng-if="dropdownIsActive" class="dropdown-body">
            <a href="#">Action Alpha</a>
            <a href="#">Action Bravo</a>
            <a href="#">Action Charlie</a>
        </div>
    </body>
</html>
```

The user opens the dropdown by clicking (or hitting `Enter`) on the "Choose Action" button. Then they click the dropdown element, and so on.

But what if they choose not to do anything? The user might scan the action list and decide to click on something else on the screen instead. The dropdown menu will be "stuck" there, which is not consistent with how most native implementations would behave in that case.
