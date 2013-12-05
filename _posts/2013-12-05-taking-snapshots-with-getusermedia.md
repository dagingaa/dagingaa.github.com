---
layout: post
author: dagingaa
title: Taking snapshots of your webcam with getUserMedia and canvas
category: Technology
---
Recently I was tasked with creating a chat for the new service I'm working on
at work, [appear.in](https://appear.in). We are a small service doing
[WebRTC](http://www.webrtc.org/)-based video chat, without registration,
logins, installs or anything. We wanted users in the chat to have profile
pictures, but seeing as we have no login, and we don't want to ask users for
their email just to get the Gravatar, we had to be a bit creative. We already
had access to the webcam, so why don't we grab a screenshot from that, and have
that appear (yes, pun) next to the chat message?

So, how do you generate a still image from the webcam of the user? In this
post, I will tell you how, with live examples!

## First, we need to access the webcam through `getUserMedia()` ##
This first part should be easy enough. We start off with some HTML.

{% highlight html linenos %}
<video id="selfView"></video>
<canvas id="drawCanvas" class="hide" height="480" width="640"></canvas>
{% endhighlight %}

As you can see, we need a video element to show the video in, and a canvas
element. Notice that the canvas element has a CSS class `hide`, let's just
assume that has the property `display: none`. We could also choose to hide the
video element, but where's the fun in that? You will also see that the canvas
has a defined height and width property. This is needed for the draw to work
correctly.

Now let's move over to some JavaScript. We need to get access to the webcam
first, and put that in the video element.

{% highlight javascript linenos %}
navigator.getUserMedia({video: true, audio: false}, function (localStream) {
    var video = document.getElementById("selfView");
    video.src = URL.createObjectURL(localStream);
    video.play();
});
{% endhighlight %}

This would prompt the user to give us access to his or her webcam. What's going
on here is that we are using something called
[`navigator.getUserMedia()`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator.getUserMedia)
to gain access to the webcam (and possibly audio input) from the device. This
works on both mobile and desktop, provided that the browser is of a high enough
version and type (Chrome, Firefox and Opera all do well). The `getUserMedia()`
function takes three arguments, constraints (object), successCallback
(function) and errorCallback (function).

Constraints are given in the form of a JavaScript object. In the example above
we request video, but not audio. There are also a bunch of other contraints
such as preferred height and width you could add.

The second argument in the example above is the successCallback. This function
takes one argument, the requested stream as an object. To show that to the user
we must first fetch the video element, then create a URL referencing the stream
by using
[`URL.createObjectURL`](https://developer.mozilla.org/en-US/docs/Web/API/URL.createObjectURL).
When all this is said and done, the video will be shown in the video element.
Remember to call `play()`!

Try it yourself below!

<button onclick="enableWebcam('selfView')" class="btn" style="margin: 10px auto; display: block">
Enable Webcam
</button>
<video height="480" width="640" id="selfView" style="margin: 0 auto; background-color: black; display: block;">
</video>

<script>
navigator.getUserMedia = ( navigator.getUserMedia ||
        navigator.webkitGetUserMedia ||
        navigator.mozGetUserMedia ||
        navigator.msGetUserMedia);
var enableWebcam = function (id) {
    navigator.getUserMedia({video: true, audio: false}, function (localStream) {
        var video = document.getElementById(id);
        video.src = URL.createObjectURL(localStream);
        video.play();
    });
}
</script>

## Taking a snapshot from a video element ##
This next part is where it gets cool. By taking a snapshot of the video data
and drawing that into a canvas, we can extract image data. Remember that
`<canvas>` tag we added earlier? We are going to use that to draw to.

{% highlight javascript linenos %}
var video = document.getElementById("selfView");
var canvas = document.getElementById("drawCanvas");
var ctx = canvas.getContext("2d");
ctx.drawImage(video, 0, 0, 640, 480);
{% endhighlight %}

What this does is take the canvas, extract the 2D context from it, and use the
[`drawImage()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement.drawImage)
function to take the data from the `HTMLVideoElement` and turning that into raw imagedata.

All you need to do now is to e.g. use
[`canvas.toDataURL()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement)
to turn that raw ImageData into something we can show in for example a
`<img>`-tag.

Try it yourself below!

<canvas id="drawCanvas" class="hide" height="480" width="640">
</canvas>

<div style="margin: 0 auto; width: 640px; display: flex;">
<button onclick="enableWebcam('selfView2')" class="btn" style="margin: 10px auto; display: block">
1: Start Webcam
</button>

<button onclick="takeSnapshot()" class="btn" style="margin: 10px auto; display: block">
2: Take Snapshot
</button>
</div>

<div style="margin: 10px auto; width: 640px; display: flex;">
<video height="240" width="320" id="selfView2" style="margin: 0 auto; background-color: black;">
</video>
<img id="snapshot" onclick="takeSnapshot()" height="240" width="320" style="padding: 0">
</div>

<script>
var takeSnapshot = function () {
    var video = document.getElementById("selfView2");
    var canvas = document.getElementById("drawCanvas");
    var img = document.getElementById("snapshot");
    var ctx = canvas.getContext("2d");
    ctx.drawImage(video, 0, 0, 640, 480);
    img.src = canvas.toDataURL();
};
</script>

So, now you know how to take snapshots with your webcam! [Tweet me](https://www.twitter.com/daginge) any questions you might have.
