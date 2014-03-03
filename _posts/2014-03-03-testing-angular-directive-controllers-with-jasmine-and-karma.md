---
layout: post
author: dagingaa
title: Testing AngularJS directive controllers with Jasmine and Karma
category: Technology
---

This blog post is a follow up to the popular
[Testing AngularJS directive templates with Jasmine and
Karma](http://daginge.com/technology/2013/12/14/testing-angular-templates-with-jasmine-and-karma/)
blog post I did earlier.

As with that post, we are going to focus on testing directives in AngularJS.
This time, we will focus on testing controllers, which can pose a bit of a
challenge because everyone writes directives like the one below, with an
anonymous function for the controller.

{% highlight javascript linenos %}
angular.module('myApp').directive('test', function() {
    return  {
        templateUrl: '/some/template/url.html',
        restrict: 'E',
        replace: true,
        controller: function ($scope) {
            $scope.open = false;
            $scope.toggle = function () {
                $scope.open = !$scope.open;
            };
        }
   };
});
{% endhighlight %}

Now, the problem is that the usual approach to controller testing is to
instantiate the controller using
[$controller](http://docs.angularjs.org/api/ng/service/$controller), but this
fails since we do not have a named controller registered on our module.

However, there is a solution. First of all, you could simply register a
controller on your module, and then use that as you normally would. Or, if you
have, like me, written code like the one above for a lot of directives without
really testing them before, there is another approach that doesn't leave you
refactoring.

By compiling the element like we did in directive template testing, we can
access the scope and start testing the publicly accessible methods right away.

{% highlight javascript linenos %}
$scope = $rootScope.$new();
var element = angular.element("<test></test>");
template = $compile(element)($scope);
$scope.$digest();
{% endhighlight %}

You now have access to the `$scope`, and can test that as you usually would.
Below is a minimal example testing the `toggle()` method above with Jasmine.

{% highlight javascript linenos %}
describe("Directive", function () {

    var $scope;

    beforeEach(inject(function($rootScope, $compile) {
        $scope = $rootScope.$new();
        var element = angular.element("<test></test>");
        template = $compile(element)($scope);
        $scope.$digest();
        controller = element.controller;
    }));

    it("should toogle open when toggle() is called", inject(function() {
        $scope.open = false;
        $scope.toggle();
        expect($scope.open).toBeTruthy();
        $scope.toggle();
        expect($scope.open).toBeFalsy();
    }));

});
{% endhighlight %}

So with that, you can now access everything on the `$scope` and start testing!
