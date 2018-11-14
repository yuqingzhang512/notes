
- [html element](#html-element)
    - [manifest file](#manifest-file)
    - [cache status](#cache-status)
    - [update cache](#update-cache)
- [head element](#head-element)
    - [title element](#title-element)
    - [meta element](#meta-element)
    - [script element](#script-element)
    - [link element](#link-element)
    - [style element](#style-element)
        - [media attribute](#media-attribute)
    - [base element](#base-element)
- [event](#event)
    - [capturing & bubbling](#capturing--bubbling)
    - [turn off](#turn-off)
    - [current target](#current-target)
    - [problems of the Microsoft model](#problems-of-the-microsoft-model)


# html element

```
<html manifest="www.example.com/cache.appcache">
````
* Define the application cache, an absolute or relative URL
    * absolute URL must be under the <b>same origin</b>
* should be inclueded on <b>every</b> page that you want cached(unless it's explicitly listed in the manifest file)
* see urls controlled by application cache: <u>chrome://appcache-internals/</u>
## manifest file

```
CACHE MANIFEST
index.html
stylesheet.css
images/logo.png
scripts/main.js
http://cdn.example.com/scripts/main.js
```

```
CACHE MANIFEST
# 21018-06-18:v2

# Explicitly cached ‘master entries’.
CACHE:
/favicon.ico
index.html
Stylesheet.css
images/logo.png
scripts/main.js

# Resources that require the user to be online.
NETWORK:
*

# static.html will be served if main.py is inaccessible
# offline.jpg will be served in place of all images in images/large/
# offline.html will be served in place of all other .html files
FALLBACK:
/main.py /static.html
images/large/ images/offline.jpg
```
* The CACHE MANIFEST is the <b>first line and required</b>
* Files can be from another domain
* Some browsers place restrictions on the amount of storage quota available to app
* If the manifest itself returns a 404 or 410, the cache is deleted
* If the manifest or s resource specified in it fails to download, the entire cache update process  fails. The browser will keep using the old cache
* App application’s cache is only updated when its manifest file changes. <u>you must modify file itself to inform the browser to refresh cached files</u>
* The manifest is checked <b>twice</b> during the update, one at the start and one after all cached files have been updated
* the HTML file that references your manifest file is automatically cached (though encouraged to include it in the manifest)

## cache status
use ```window.applicationCache``` to access the browser's app cache
```
var appCache = window.applicationCache;

switch (appCache.status) {
    case appCache.UNCACHED:    // UNCACHED == 0
        return ‘UNCACHED’;
        break;
    case appCache.IDLE:    // IDLE == 1
        return ‘IDLE’;
        break;
    case appCache.CHECKING:    // CHECKING == 2
        return ‘CHECKING’;
        break;
    case appCache.DOWNLOADING:    // DOWNLOADING == 3
        return ‘DOWNLOADING’;
        break;
    case appCache.UPDATEREADY:    // UPDATEREADY == 4
        return ‘UPDATEREADY’;
        break;
    case appCache.OBSOLETE:    // OBSOLETE == 5
        return ‘OBSOLETE’;
        break;
    default:
        return ‘UNKNOWN CACHE STATUS’;
        break;
}
```

## update cache
* user clears browser data
* the manifest file is modified
    * updating a file listed in the manifest doesn’t mean the browser will re-cache the resource. The manifest file itself must be altered.
```
var appCache = window.applicationCache;
appCache.update();    // Attempt to update the user’s cache.

…

if (appCache.status == window.applicationCache.UPDATEREADY) {
    appCache.swapCache();    // The fetch was successful, swap in the new cache
}
```
but still need to refresh the browser to use the latest version, with automation like below:
```
// check if new cache is available on page load.
window.appEventListener('load', function(e) {
    window.applicationCache.addEventListener('updateready', function(e) {
        if (window.applicationCache.status == window.applicationCache.UPDATEREADY) {
            // browser downloaded a new app cache
            if (confirm('a new version of this site is available. load it?')) {
                window.location.reload();
            } else {
                // manifest didn't cached. nothing new to server.
            }
        }
    }, false);
}, false);
```

# head element

## title element
show up:
* browser title bar or tab
* search engine will include this in the search results
* used as the name when adding to the favourite or bookmark
* when multiple exist, browser will display the first title, ignore the rest

## meta element
```
<meta name="author" content="Mark"/>
<meta http-equiv="refresh" content="45"/>
```
* <d>name</d>: for metadata that describes the content of the HTML document, most common values:
    * application-name
    * author
    * description
    * keywords: the content attributes will contain a comma-seperated list of keywords
* <d>http-equiv</d>: used to simulate an http response header, common values:
    * content-type: For example, content = "text/html; charset=UTF-8"
    * default-style: specify the default style sheet
    * refresh: force the page to automatically refresh after a certain inteval(in seconds)

## script element
type attribute is optional and will default to ```text/javascript```, required in HTML4
* as the browser is parsing the HTML document, when a ,<i>script</i> element is encountered, the script is loaded and executed before continuing to parse the rest of the document.For external files, use <b>async</b> or <b>defer</b> attributes to change this behavior.
    * async: the file is loaded and executed in parallel, while the parsing process continues.
    * defer: the script will be executed only after the pages has been fully parsed. <i>Use defer if the scripts has code that executes immediately, which references any of the HTML elements. If it executes before the document is parsed, the script might fail because the elements are not yet available.</i>

## link element
two categories:
* load resources that are used to render the source document, like style sheets
* other related documents, reader may choose to navigate to these documents but they are not needed to render the current page.
  ```
  <link rel="stylesheet" type="text/css" href="Sample.css"/>
  <link rel="icon" type="image/x-icon" href="HTMLBadge.ico"/>
  <link rel="alternate" type="text/plain" href="TextPage.txt"/>
  ```
* For style sheets, <i>type</i> is optional since <i>text/css</i> is the assumed type.
* The icon file is displayed in the browser tab, also be used in bookmarks or favourites or history list.
* resource-type relationships
  rel   |   description
  ---   |   ---
  icon  |   loads an icon, <i>type</i> is expected ti be an image file type such as "image/x-icon" or "image/png"
  prefetch  |   external resources may be needed later and should be loaded when possible
  preload   |   resources will be needed by current document and should be loaded as soon as possible
  stylesheet    |   the type is optional ans assumed to be <i>text/css</i>
* reference-type relationships
  rel   |   description
  ---   |   ---
  alternate |   
  author    |   often a mail such as <i>href="mailto://a@b.com</i>, could also be a link to a web page about author
  help  |   help page
  next  |   refernce the next document is a series
  license   |   
  pingback  |   link to a pingback service, used to notify a page when a link has been made to it from another page
  prev  |   link to previous document is a series
  search    |   the referenced document can be used for searching within the current document or web site


## style element

### media attribute
A <i>media query</i> is a <b>Boolean</b> expression that can conditionally apply a set of styles


## base element
```
<base href="www.thecreativepeople.com/html5" target="_self" />
```
used to define the base URL that should be used for all other references in the doc.
support two specific attributes:
* href
* target: specify the default behavior when a link is selected
    * supported values
        value   |   desc
        ---     |   ---
        _blank  |   open in a new window or tab
        _self   |   open in current window or tab(default value)
        _parent |   open in parent frame
        _top    |   open in topmost frame
<b>The frameset and frame elements are not supported in HTML5 so the _parent and _top values are not applicable unless using iframe</b>
<b>The base element shoud be the first child of the head element, or at least come before any link elements so the base address will be applied to subsequent link element</b>







# event

## capturing & bubbling
* Netscape - capturing (down)
* Microsoft - bubbling (up)
* any event taking place in the w3c event model is first captured until it reaches the target element and then bubbles up again
```
element.addEventListener(event, function, useCapture);
```

## turn off
* Microsoft
```
window.event.cancelBubble = true;
```
* w3c model
```
e.stopPropagation();
```

## current target
during the capturing and bubbling phases, the target does not change, it always remains a reference to the source

## problems of the Microsoft model
<i>this</i> keyword doesn't refer to the HTML element, this means if you do
```
element1.attachEvent('onclick', doSomething);
element2.attachEvent('onclick', doSomething);
```
you <b>cannot</b> know which HTML element currently handles the event