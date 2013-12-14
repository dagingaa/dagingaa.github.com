---
layout: post
author: dagingaa
title: Testing AngularJS directive templates with Jasmine and Karma
category: Technology
---

In my day job, I work on [appear.in](https.//appear.in),
a [video chat service built with AngularJS and WebRTC](http://comoyo.github.io/blog/2013/08/05/video-meetings-in-the-browser-using-webrtc-and-angularjs/).
Recently, our application has become more complex, and our HTML templates ended
up having a lot of state in them. We could no longer just use unit testing to
verify our application logic, we also needed to test the state in our
templates.

For our unit testing, we landed on using
[Jasmine](http://pivotal.github.io/jasmine/) and
[Karma](http://karma-runner.github.io/), and this has worked fairly well for
us. So when we wanted to test our templates, we wanted to use the same tools.

## Setting up Karma to serve templates
Luckily, testing anything in Angular is pretty straight forward. However,
integrating HTML templates from `templateUrl`s was not that easy. For those
familiar with Karma from before, you can use the Karma server to serve static
files needed to be used in your testing. This means that you do not need to
fire up your own test server and keep that running.

Configuring Karma to serve files is as simple as declaring which files you want
it to serve. This works great for `*.js` files, but what about HTML and other
static files? It turns out that Karma only really likes serving JavaScript
files, so we have to do some clever workarounds to make Angular templates,
which are just regular HTML files work.

To make Karma serve HTML templates, we have to use a preprocessor that turns
HTML templates into JavaScript strings and registers them with Angular's
[$templateCache](http://docs.angularjs.org/api/ng.$templateCache). This means
that Angular can access the templates without having to make separate HTTP
requests. All we need to do is serve the processed template JavaScript.

After some research, I found
[karma-ng-html2js-preprocessor](https://github.com/karma-runner/karma-ng-html2js-preprocessor),
a npm package that does the above, all from the karma configuration. Run `npm
install karma-ng-html2js-preprocessor --save-dev` in the root path of the
project you want to test. Now all you have to do is to set it up in your `karma.conf.js` file.

{% highlight javascript linenos %}
module.exports = function (config) {
    config.set({
        preprocessors: {
            'path/to/html/templates/**/*.html': ['ng-html2js']
        },

        ngHtml2JsPreprocessor: {
            // setting this option will create only a single module that contains templates
            // from all the files, so you can load them all with module('foo')
            moduleName: 'templates'
        },

        files: [
            // ...
            'path/to/html/templates/**/*.html'
        ],

        // ...
    });
};
{% endhighlight %}

## Writing tests with Jasmine
Now that we have gotten the file serving out of the way, let's take a look how
to write tests for our templates. Say we have the following HTML template we want to test:

{% highlight html linenos %}
<h1 ng-bind="header"></h1>
<p ng-bind="text"></p>
{% endhighlight %}

Now we want to verify that our `$scope` variables are not hardcoded in, and are
rendered as expected into our template. To do that, we would write the
following test.

{% highlight javascript linenos %}
describe("Directive:", function () {

    beforeEach(angular.mock.module("yourAppModule"));

    describe("template", function () {
        var $compile;
        var $scope;
        var $httpBackend;

        // Load the templates module
        beforeEach(module('templates'));

        // Angular strips the underscores when injecting
        beforeEach(inject(function(_$compile_, _$rootScope_) {
            $compile = _$compile_;
            $scope = _$rootScope_.$new();
        }));

        it("should render the header and text as passed in by $scope",
        inject(function() {
            // $compile the template, and pass in the $scope.
            // This will find your directive and run everything
            var template = $compile("<div your-directive></div>")($scope);

            // Set some values on your $scope
            $scope.header = "This is a header";
            $scope.text = "Lorem Ipsum";

            // Now run a $digest cycle to update your template with new data
            $scope.$digest();

            // Render the template as a string
            var templateAsHtml = template.html();

            // Verify that the $scope variables are in the template
            expect(templateAsHtml).toContain($scope.header);
            expect(templateAsHtml).toContain($scope.text);

            // Do it again with new values
            var previousHeader = $scope.header;
            var previousText = $scope.text;
            $scope.header = "A completely different header";
            $scope.text = "Something completely different";

            // Run the $digest cycle again
            $scope.$digest();

            templateAsHtml = template.html();

            expect(templateAsHtml).toContain($scope.header);
            expect(templateAsHtml).toContain($scope.text);
            expect(templateAsHtml).toNotContain(previousHeader);
            expect(templateAsHtml).toNotContain(previousText);

        }));

    });
});
{% endhighlight %}

## Some caveats
So, this all sounds very easy now doesn't it? And yes, it is easy, but there
are also some caveats you need to know about should you choose to test your
templates this way.

The first one is non-html content. If you, like us, use `ng-include` to insert
svg data into the DOM (so you can style it with CSS), you are going to have a
bad time. PhantomJS will throw an error and exit, so you have to find another
way. What we did was mock the file path in `$httpBackend` so that it responded
with an empty string for all instances of `.svg`.


{% highlight javascript %}
$httpBackend.whenGET('/path/to/a.svg').respond("");
{% endhighlight %}

I was not able to find a general solution for ignoring all `.svg` files based
on wildcard, but I suspect that it's possible.

The second one is that the tests can be brittle. When you want to test if an
ng-if is inserted or not, you must rely on checking for other things other than
$scope variables, such as CSS classes. This means that a well-meaning developer
can break the build by changing a CSS class, which is not very friendly.
Proceed with caution.

Hopefully the guide was useful for you! And remember, always be test.in™!
