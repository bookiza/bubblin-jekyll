---
layout: post
title:  "Going Fullscreen On iPad Safari 🤩"
date:   2018-11-04 1:30:00 -0400
categories: iPad, web, tablet, fullscreen api, kids, superbooks
author: Marvin Danig
published: true

original:
  link: https://bubblin.io/blog/fullscreen-api-ipad
  date: 2018-11-04
  site: Bubblin Superbooks

---


Great news! 

Apple has rolled out support for the <a rel="nofollow" href="https://www.w3.org/TR/fullscreen/">Fullscreen API</a>
on iPad Safari with its iOS 12 update. This means developers can now create fully immersive web applications for users on the iPad. Remove every distraction on screen reliably and help the user focus on the task at hand—just like a native app!—and yet allowing them to get away if they wanted to, ala, better accessibility features of the web. 

We think that this is a huge win! 

It is a particularly useful for those of us who use iPad as their "go to" device for surfing at night. 🙂

While there are some issues in the current implementation of fullscreen api on iPad Safari, that we'll get to shortly, this update from Apple has been nothing short of bonus for us. We built fullscreen right into [Bubblin Superbooks](https://bubblin.io) for book lovers on our site. The post below explains how we implemented it, but before that let's take a look at current level of <a rel="nofollow" href="https://caniuse.com/#feat=fullscreen" target="_blank">vendor support</a>: 


<img src="https://raw.githubusercontent.com/marvindanig/assets/master/fullscreen.jpg" width="100%" alt="fullscreen-api-ipad" title="support fullscreen api ipad safari" />

UPDATE: As of 1<sup>st</sup> March, 2019 most bugs around escaping fullscreen mode have been resolved by Apple.

As you can see from Caniuse nearly all major browsers on the desktop and Android devices have had some level of support for the fullscreen API, but iOS Safari doesn't show the latest information. This feature has been released on iOS Safari (iPad only) barely a few days ago so as of ~~November 4th, 2018~~ March 1<sup>st</sup>, 2019 Caniuse doesn't (still) reflect it yet. Hopefully it is updated soon!

Aside from familiar vendor prefixes, varying method names and other few minor inconsistencies across implementations, there is not much to be able to start using the fullscreen api on the iPad Safari. Here's how we implemented using vanilla JavaScript.


First, define a named function `_toggleFullScreen` that lets you _toggle_ between fullscreen and url mode:

```javascript
const _toggleFullScreen = function _toggleFullScreen() {
	if (document.fullscreenElement || document.mozFullScreenElement || document.webkitFullscreenElement) {
		if (document.cancelFullScreen) {
			document.cancelFullScreen();
		} else {
			if (document.mozCancelFullScreen) {
				document.mozCancelFullScreen();
			} else {
				if (document.webkitCancelFullScreen) {
					document.webkitCancelFullScreen();
				}
			}
		}
	} else {
		const _element = document.documentElement;
		if (_element.requestFullscreen) {
			_element.requestFullscreen();
		} else {
			if (_element.mozRequestFullScreen) {
				_element.mozRequestFullScreen();
			} else {
				if (_element.webkitRequestFullscreen) {
					_element.webkitRequestFullscreen(Element.ALLOW_KEYBOARD_INPUT);
				}
			}
		}
	}
};
```

This `_toggleFullScreen` function takes care of all the vendor prefixes and browser quirks across the spectrum and now we just have to find out which device, browser and iOS version the user is on to enable the fullscreen functionality for them. We have to determine if the user is (1) on the iPad and (2) using the Safari browser, and (3) if the browser is on iOS 12 or higher already. 

Fortunately (or unfortunately) we have the vendor sniffing technique to rescue:

```javascript
	const userAgent = window.navigator.userAgent;

	const iPadSafari =
		!!userAgent.match(/iPad/i) &&  		// Detect iPad first.
		!!userAgent.match(/WebKit/i) && 	// Filter browsers with webkit engine only
		!userAgent.match(/CriOS/i) &&		// Eliminate Chrome & Brave
		!userAgent.match(/OPiOS/i) &&		// Rule out Opera
		!userAgent.match(/FxiOS/i) &&		// Rule out Firefox
		!userAgent.match(/FocusiOS/i);		// Eliminate Firefox Focus as well!

	const element = document.getElementById('fullScreenButton');

	function iOS() {
		if (userAgent.match(/ipad|iphone|ipod/i)) {
			const iOS = {};
			iOS.majorReleaseNumber = +userAgent.match(/OS (\d)?\d_\d(_\d)?/i)[0].split('_')[0].replace('OS ', '');
			return iOS;
		}
	}

	if (element !== null) {
		if (userAgent.match(/iPhone/i) || userAgent.match(/iPod/i)) {
			element.className += ' hidden';
		} else if (userAgent.match(/iPad/i) && iOS().majorReleaseNumber < 12) {
			element.className += ' hidden';
		} else if (userAgent.match(/iPad/i) && !iPadSafari) {
			element.className += ' hidden';
		} else {
			element.addEventListener('click', _toggleFullScreen, false);
		}
	}


```

We tested the code above on quite a few browsers on the iPad, but vendor sniffing, as you might already know is an inefficent-cumbersome-avoid-as-much-as-you-can solution. Going forward we expect more browsers on the iPad to support fullscreen api (Google Chrome next?) or be ruled out using vendor sniffing. Given that support for fullscreen api on tablets will only improve over time, we can revisit the sniffer code above until this progressive enhancement melts into broader support. 

> To see the above code working open a [Superbook on your iPad](https://bubblin.io/book/bookiza-documentation-by-marvin-danig/1), turn a few pages and go fullscreen (menu on bottom right under three dots). 

Here are a few sample `userAgent` strings that we used to identify browsers on the iPad:
#### iPad Safari:
```bash
Mozilla/5.0 (iPad; CPU OS 12_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/12.0 Mobile/15E148 Safari/604.1
```
#### Firefox Original:
```bash
Mozilla/5.0 (iPad; CPU OS 12_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) FxiOS/14.0b12646 Mobile/16B92 Safari/605.1.15
```
#### Firefox Focus:
```bash
Mozilla/5.0 (iPad; CPU OS 12_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) FocusiOS/7.0.3 Mobile/16B92 Safari/605.1.15
```

… and so on.



On the CSS side you may not have to make any changes, but here are few options to consider depending on your situation:

```css

:-webkit-full-screen body,
:-moz-full-screen body,
:-ms-fullscreen body {
	/* properties */
	width: 100vw;
	height: 100vh;
}

:full-screen body {
	/*pre-spec */
	/* properties */
	width: 100vw;
	height: 100vh;
}

:fullscreen body {
	/* spec */
	/* properties */
	width: 100vw;
	height: 100vh;
}

/* deeper elements */

:-webkit-full-screen body {
	width: 100vw;
	height: 100vh;
}

/* styling the backdrop*/

::backdrop,
::-ms-backdrop {
	/* Custom styles */
}


```


That's it. You now have a web app that can go fullscreen on iPad Safari and people can enjoy using it like a standalone/native app without needing to _add to the homescreen_ anymore. Again, we think this is a huge win for web and you might want to take advantage of it. 

---

Written by: Marvin Danig, I'm the CEO of Bubblin Superbooks but I also love to code. You can follow me on [Twitter](https://twitter.com/marvindanig).

Edited by: Sonica Arora, CTO of Bubblin Superbooks. 



**P.S.:** It's likely that some of you viewed the article and the demo book on your desktop. If you did that we recommend revisiting the demo on your iPad. We're talking about post-PC era here, it's serious! 🙈







