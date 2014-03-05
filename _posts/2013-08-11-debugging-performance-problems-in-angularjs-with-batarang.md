---
layout: post
author: dagingaa
title: Debugging performance problems in AngularJS with Batarang and Chrome DevTools
category: Technology
---

Currently I am creating a medium-scale application in AngularJS for my
employer, Comoyo. We are transitioning away from an outdated frontend stack,
and replacing it with an application mostly built using AngularJS for the
frontend. This has been en extremely smooth process, and I am greatly enjoying
working with AngularJS.

A couple of days ago we wanted to test how my application worked on a
Playstation 3. Applications developed for the Playstation 3 are developed as
regular web applications, and run under a limited, but functional browser. From
before, we had an application built using SpineJS, which was very fast and
snappy. However, when we fired up my AngularJS application, it was dead slow.
Navigation was horrible, and even simple clicks to scroll a horizontal list,
took about 5 seconds to execute. What was going on?

First, some introduction to how the application is built. Our front page
consists of a set of scrollable horizontal lists. These lists consist of
several movie covers. We have 34 lists on the front page, and each list consist
of a maximum of 15 movie covers. Some quick math tells us that we have 510
items showing on a naive implementation of the front page. The horizontal lists
are scrollable however, so each list only show a maximum of 6 covers at a time.

![Comoyo on AngularJS](/assets/img/posts/batarang/comoyoangular.png)

If we follow the developer guide for how AngularJS applications should be
built, a layout like this requires the use of ng-repeat to iterate over the
array of lists, and another ng-repeat to iterate over the covers in each list.
Below is some simplified code to illustrate the example. The real code features
two directives, one for horizontal lists, and one for covers.

{% highlight html linenos %}
<div class="lists" ng-repeat="list in lists">
  <h1>{ { list.title } }</h1>
  <div class="previousPage" ng-click="previousPage()"></div>
  <ol class="scrollable list">
    <li class="cover" ng-repeat="movie in list.items">
      <img ng-src="movie.coverImages.160x225">
      <h2 class="title">{ { movie.title } }</h2>
      <p class="runningTime">{ { movie.runningTime } } minutes</p>
    </li>
  </ol>
  <div class="nextPage" ng-click="nextPage()"></div>
</div>
{% endhighlight %}

On my desktop machine, this code was running without any issues. I easily made
60 fps while scrolling, although the page itself took about 400 ms to render.
Clicking next page was also very fast, and the items scrolled nicely with CSS
transitions. What was causing the slowdown on the PS3 when clicking next page?
Let's take a look at the next page function.

{% highlight javascript linenos %}
$scope.nextPage = function () {
  var totalLength = $scope.list.items.length;

  // Make sure the rightmost cover becomes the leftmost
  var numberOfCoversToScroll = $scope.numberOfCoversDisplayed() - 2;
  var nextFirstVisibleElementIndex = 
    indexOfFirstVisibleElement + numberOfCoversToScroll;

  // Logic for end-of-list
  if (nextFirstVisibleElementIndex < (totalLength - numberOfCoversToScroll)) {
    indexOfFirstVisibleElement = nextFirstVisibleElementIndex;
    var wScroller = angular.element($scope.element.children[2]);
    wScroller.removeClass("atFirstPage");
  } else {
    // The end of the list becomes visible, set first visible element
    // so that the last visible element is pushed right
    indexOfFirstVisibleElement = totalLength - numberOfCoversToScroll;
    $scope.atLastPage();
  }

  // Set up CSS transition
  var properties = {
    left: "-"
      + ($scope.coverWidth + $scope.coverMargin) * indexOfFirstVisibleElement
      + "px"
  };
  var scrollableElement = $scope.element.children[2].children[1].children[0];
  angular.element(scrollableElement).css(properties);
}
{% endhighlight %}

So, there are some dirty bits with this code, but it works. I couldn't see
anything that would account for the 5 second delay we saw on the Playstation.

##Batarang and Chrome DevTools to the rescue!##
Enter [Batarang](https://github.com/angular/angularjs-batarang). Batarang adds
debugging and profiling tools for AngularJS application to Chrome DevTools. It
features an interactive model/scope explorer, dependency graph, and performance
monitoring. We are going to look more closely into the latter.

Going into my application and firing up the performance tab I was shocked to
see that something called ngRepeatWatch was taking up 85% of the run time for
my application. Watch functions were responsible for 95% of the run time for my
application in total.

Doing some quick googling, it seemed as if this is a known problem in the
angular community. NgRepeat has serious performance problems when faced with
large data sets, and I was even nesting it. This lead down a patch of reading
about [how AngularJS data bindings worked under the
hood](http://angular-tips.com/blog/2013/08/watch-how-the-apply-runs-a-digest/)
and [how to remove unnecessary
watches](http://angular-tips.com/blog/2013/08/removing-the-unneeded-watches/).
I was spawning 34 * 15 = 510 ngRepeatWatches and 510 * 3 = 1530 regular watches
on the front page! Every time that ng-click (with a subsequent $apply) was
called, it needed to do 1530 + 510 = 2040 DOM reads to check if a value has
been updated. This is not a big problem on the modern browser in my laptop, but
turned out to be a huge performance bottleneck on the legacy browser running on
the PS3.

##Fixing the problem (...ish)##
To fix the problem, most people suggested redoing the application layout to not
feature such a large dataset, citing that the issue was not with ng-repeat, but
with the way the application was designed. While this may be true, I wanted to
see if I could solve the data-binding problem in an elegant way.
[Bindonce](https://github.com/Pasvaz/bindonce) is a nice little set of angular
directives that enable you to, with small modifications in your templates,
automatically unregister watchers when an attribute is set the first time.
Bindonce even waits for the object to be populated before unregistering the
watcher. My HTML template now looked like this:

{% highlight html linenos %}
<div class="lists" bindonce ng-repeat="list in lists">
  <h1 bo-text="list.title"></h2>
  <div class="previousPage" ng-click="previousPage()"></div>
  <ol class="scrollable list">
    <li class="cover" bindonce ng-repeat="movie in list.items">
      <img bo-src="movie.coverImages.160x225">
      <h2 class="title" bo-text="movie.title"></h2>
      <p class="runningTime" bo-text="movie.runningTime + ' minutes'"></p>
    </li>
  </ol>
  <div class="nextPage" ng-click="nextPage()"></div>
</div>
{% endhighlight %}

This reduced the number of watchers to about 510. However, the biggest
performance bottleneck was still not fixed. ngRepeatWatch now stood for 95% of
the application run time (but time spent was unchanged from before). I "solved"
this by implementing infinite scroll with
[ngInginiteScroll](http://binarymuse.github.io/ngInfiniteScroll/) using
in-memory data which drastically sped up the initial rendering time of the
page, but made frame rates tank while hitting the bottom of the page. A more
permanent fix would be to write a custom ng-repeat directive that didn't
register watchers, but that is for another day.

![Comoyo on AngularJS](/assets/img/posts/batarang/performance.png)


##Takeaways##
While I greatly enjoy working with AngularJS, the framework has some major
gotchas when you rely on it too much. Data-binding, especially on older
browsers, is slow when done the way Angular does it for large datasets. When
ECMAScript 6 starts getting implemented with Object.observe, this will
hopefully become much faster.

- Avoid using ng-repeat for large datasets. Design your application around it
  if possible
- Batarang is essential when developing AngularJS applications
- Bindonce is a great way to minimize the number of watches, but requires you
  to change the structure of your template
- Avoid letting AngularJS do magic for you when you don't really need it
