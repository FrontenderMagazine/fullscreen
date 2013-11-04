<section>
We have the ability to easily make immersive fullscreen web sites and
applications, but like anything on the web there are a couple of ways to do it. 
This is especially important now that more browsers are supporting an "installed
web app" experience which launch fullscreen.<figure> <figcaption>Example of a
site going fullscreen in Chrome for Android</figcaption></figure>

## Getting your app or site fullscreen {#toc-getting}

There are several ways that a user or developer can get an web app fullscreen
.

*   Fake it: auto-hide the address bar
*   Request the browser to go fullscreen in response to a user gesture.
*   Install the app to the home screen

### Fake it: auto-hide the address bar {#toc-faking}

You can "fake fullscreen" by auto-hiding the address bar as follows:

    window.scrollTo(0,1);

This is a pretty simple method, the page loads and the browser bar is told to
get out of the way. Unfortunately it is not standardised and not well supported.
You also have to work around a bunch of quirks. For example browsers often 
restore the position on the page when the user navigates back to it. Using

    window.scrollTo overrides this, which annoys the user. To work around this
you have to store the last position in localStorage, and deal with the edge 
cases (for example, if the user has the page open in multiple windows
). 

### Request the browser to go fullscreen in response to a user gesture {#toc-
request
}

[Not all platforms are equal][1]. iOS Safari doesn't have a fullscreen API, but
we do on Chrome on Android and Firefox.

Most applications you build will use a combination of the JS API and the CSS
selectors provided by the fullscreen specification.

The main JS API's that you need to care about when building a fullscreen
experience are:

*       *element*.requestFullscreen() (currently prefixed for both Chrome and
    Firefox) displays the element in fullscreen mode.
   

*       document.cancelFullscreen() (currently prefixed for both Chrome and
    Firefox) cancels fullscreen mode.
   

*       document.fullscreenElement (currently prefixed for both Chrome and
    Firefox) returns true if any of the elements are in fullscreen mode.
   

When your app is fullscreen you no longer have the browser's UI controls chrome
available to you. This changes the way that users interact with your experience.
They don't have the standard navigation controls such as Forwards and Backwards;
they have their escape hatch that is the Refresh button. It's important to cater
for this scenario. You can use some CSS selectors to help you change the style 
and presentation of your site when the browser enters fullscreen mode.

    <button id="goFS">Go fullscreen<button> 
    <script>
       var goFS = document.getElementById("goFS");
       goFS.addEventListener("click", function() {
          document.body.requestFullScreen();
       }, false);
    </script>

The above example is a little contrived; I've hidden all the complexity around
the use of vendor prefixes.

The actual code is a lot more complex. [Mozilla has created][2] a very useful
script that you can use to toggle fullscreen. As you can see, due to the vendor 
prefix situation it is rather complex and cumbersome compared to the specified 
API. Even with the slightly simplified code below, it is still complex.

    function toggleFullScreen() {
      var doc = window.document;
      var docEl = doc.documentElement;
          
      var requestFullScreen = docEl.requestFullscreen || docEl.mozRequestFullScreen || docEl.webkitRequestFullScreen;
      var cancelFullScreen = doc.exitFullscreen || doc.mozCancelFullScreen || doc.webkitExitFullscreen;
          
      if(!doc.fullscreenElement && !doc.mozFullScreenElement && !doc.webkitFullscreenElement) { 
        requestFullScreen.call(docEl);
      }
      else {
        cancelFullScreen.call(doc);  
      }
    }

We web developers hate complexity. A nice high-level abstract API you can use
is[][3]Sindre Sorhus' [Screenfull.js][4] module which unifies the two slightly
different JS API's and vendor prefixes into one consistent API.

### Launching a page fullscreen {#toc-launch}

Launching a fullscreen web page when the user navigates to it is not possible.
Browser vendors are very aware that a fullscreen experience on every page load 
is a huge annoyance, therefore a user gesture is required to enter fullscreen. 
Vendors do allow users to "install" apps though, and the act of installing is a 
signal to the operating system that the user wants to launch as an app on the 
platform.

Across the major Mobile platforms it is pretty easy to implement using either
meta tags, or manifest files as follows.

#### iOS

Ever since the launch of the iPhone, users have been able to install Web Apps
to the home screen and have them launch as full-screen web apps.

    <meta name="apple-mobile-web-app-capable" content="yes">

[From Apple][5]:

> If content is set to yes, the web application runs in full-screen mode;
> otherwise, it does not. The default behavior is to use Safari to display web 
> content. You can determine whether a webpage is displayed in full-screen mode 
> using the window.navigator.standalone read-only Boolean JavaScript property.
>
#### Chrome for Android

The Chrome team has recently implemented a feature that tells the browser to
launch the page fullscreen when the user has added it to the home screen. It is 
similar to the iOS Safari model.

    <meta name="mobile-web-app-capable" content="yes">

[From Chrome][6]:

> You can set up your web app to have an application shortcut icon added to a
> device's homescreen, and have the app launch in full-screen "app mode" using 
> Chrome for Android's "Add to homescreen" menu item.
>
#### Firefox OS

Firefox OS has implemented a installable web app and packaged app model for
developers to create apps for the user's phone. This model is a little more 
complex, but it fits in to a wider strategy of installable apps. The developer 
has to define a manifest for the app that Firefox OS will parse at install time.

[From Mozilla][7]:

    {
      "name": "My App",
      "description": "My elevator pitch goes here",
      "launch_path": "/",
      "icons": {
        "128": "/img/icon-128.png"
      },
      **"fullscreen": true**
    }

## API Tips {#toc-apitips}

### Making the document fullscreen {#toc-documentfullscreen}

It is natural to think that you take the body element fullscreen, but if you
are on a WebKit or Blink based rendering engine you will see it has an odd 
effect of shrinking the body width to the smallest possible size that will 
contain all the content (Mozilla Gecko is fine
).<figure>

![][8]<figcaption>Figure 1: Fullscreen on the body element.</figcaption></
figure
>
To fix this, use the document element instead of the body element:

    document.documentElement.requestFullScreen();<figure>

![][8]<figcaption>Figure 2: Fullscreen on the document element.</figcaption></
figure
>
### Making a video element fullscreen {#toc-videofullscreen}

To make a video element fullscreen it is exactly the same as making any other
element fullscreen. You call the

    requestFullScreen method on the video element. 

    <video id=videoElement></video>
    <button id="goFS">Go Fullscreen</button> 
    <script>
       var goFS = document.getElementById("goFS");
       goFS.addEventListener("click", function() {
          var videoElement = document.getElementById("videoElement");
          videoElement.requestFullScreen();
       }, false);
    </script>

If your <video> element doesn't have the controls attribute defined, there's no
way for the user to control the video once they are fullscreen. The recommended 
way to do this is to have a basic container that wraps the video and the 
controls that you want the user to see.

    <div id="container">
      <video><video>
      <div>
        <button>Play<button>
        <button>Stop<button>
        <button id="goFS">Go fullscreen<button> 
      <div>
    <div>
    <script>
       var goFS = document.getElementById("goFS");
       goFS.addEventListener("click", function() {
          var container = document.getElementById("container");
          container.requestFullScreen();
       }, false);
    <script>

This gives you a lot more flexibility because you can combine the container
object with the CSS pseudo selector (for example to hide the "goFS" button.
)

    <style>
    #goFS:full-screen #goFS {
      display: none;
    } 
    #goFS:-webkit-full-screen #goFS {
      display: none;
    } 
    #goFS:-moz-full-screen #goFS {
      display: none;
    } 
    <style>

Using these patterns you can detect when full-screen is running and adapt your
user interface appropriately:

*   Provide a link to get back to the start page
*   Provide a mechanism to close dialogs or travel backwards

## UX Guidelines {#toc-uxguidelines}

### Don't rely on navigation controls {#toc-dontrely}

iOS and Firefox OS don't have a hardware back button or refresh gesture,
therefore you must ensure that users can navigate throughout the app without 
getting locked in.

You can detect if you are running in a fullscreen mode or an installed mode
easily on all the major platforms.

#### iOS

    if(window.navigator.standalone == true) {
      // My app is installed and therefore fullscreen
    }

#### Chrome for Android

When launching as an installed app, Chrome is not running in true fullscreen
experience so

    document.fullscreenElement returns null and the CSS selectors don't work.
Chrome also does not an API comparable to iOS's navigator.standalone property.

When the user requests fullscreen via a gesture on your site, the standard
fullscreen API's are available including the CSS pseudo selector that lets you 
adapt your UI to react to the fullscreen state like the following

    selector:-webkit-full-screen {
      display: block; // displays the element only when in fullscreen
    }
    
    selector {
      display: none; // hides the element when not in fullscreen mode
    }

#### Firefox

When the user requests fullscreen via your site or the user launches the app in
fullscreen mode all the standard fullscreen API's are available including the 
CSS pseudo selector that lets you adapt your UI to react to the fullscreen state
like the following:

    selector:-moz-full-screen {
      display: block; // hides the element when not in fullscreen mode
    }
    
    selector {
      display: none; // hides the element when not in fullscreen mode
    }

#### Specification

The spelling in the specification differs slightly from the implementations in
Chrome and Firefox, but the functionality is the same.

    selector:fullscreen {
      display: block;
    }
    
    selector {
      display: none; // hides the element when not in fullscreen mode
    }

### Keep the user in the fullscreen experience {#toc-keepuserin}

The fullscreen API can be a little finicky sometimes. Browser vendors don't
want to lock users in a fullscreen page so they have developed mechanisms to 
break out of fullscreen as soon as they possibly can. This means you can't build
a fullscreen website that spans multiple pages because:

*   Changing the URL programmatically by using window.location = "http://
    example.com" breaks out of fullscreen
   
*   A user clicking on an external link inside your page will exit fullscreen
*   Changing the URL via the 
        navigator.pushState API will also break out of the fullscreen
    experience.
   

You have two options if you want to keep the user in a fullscreen experience:

1.  Use the installable web app mechanisms to go fullscreen
2.  Manage your UI and app state using the # fragment.

By using the #syntax to update the url (window.location = "#somestate"), and
listening to the window.onhashchange event you can use the browser's own history
stack to manage changes in the application state, allow the user to use their 
hardware back buttons, or offer a simple programmatic back button experience by 
using the history API as follows:

    window.history.go(-1)

### Let the user choose when to go fullscreen {#toc-letuserchoose}

There is nothing more annoying to the user than a web site doing something
unexpected. When a user navigates to your site don't try and trick them into 
fullscreen.

Don't intercept the first touch event and issue a requestFullScreen.

1.  it is annoying
2.  Browsers may decided to prompt the user at some point in the future about
    allowing the app to take up the fullscreen.
   <figure>

![][8]<figcaption>Figure 3: FirefoxOS prompting for access to fullscreen.</
figcaption
></figure>
If you want to launch apps fullscreen do think about using the install
experiences for each platform.

### Don't spam the user to install your app to a homescreen {#toc-nospam}

If you plan on offering a fullscreen experience via the installed app
mechanisms be considerate to the user:

*   Be discreet. Use a banner or footer to let them know they can install the
    app.
   
*   If they dismiss the prompt, don't show it again.
*   On a users first visit they are unlikely to want to install the app unless
    they are happy with your service. Consider prompting them to install after a 
    positive interaction on your site.
   
*   If a user visit your site regularly and they don't install the app, they
    are unlikely to install your app in the future. Don't keep spamming them.
   

## Conclusions {#conslusions}

While we don't have a fully standardised and implemented API, using some of the
guidance presented in this article you can easily build experiences that take 
advantage of the user's entire screen, irrespective of the client.</section>

 [1]: http://caniuse.com/#feat=fullscreen

 [2]: https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Using_full_screen_mode
 [3]: http://sindresorhus.com/screenfull.js
 [4]: https://github.com/sindresorhus/screenfull.js

 [5]: https://developer.apple.com/library/safari/documentation/AppleApplications/Reference/SafariHTMLRef/Articles/MetaTags.html
 [6]: https://developers.google.com/chrome/mobile/docs/installtohomescreen
 [7]: https://developer.mozilla.org/en-US/Apps/Developing/Manifest#fullscreen
 [8]: img/1.gif