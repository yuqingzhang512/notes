
- [elements](#elements)
    - [html](#html)
        - [manifest file](#manifest-file)
        - [cache status](#cache-status)
        - [update cache](#update-cache)


# elements

## html

```
<html manifest="www.example.com/cache.appcache">
````
* Define the application cache, an absolute or relative URL
    * absolute URL must be under the <b>same origin</b>
* should be inclueded on <b>every</b> page that you want cached(unless it's explicitly listed in the manifest file)
* see urls controlled by application cache: __chrome://appcache-internals/__
### manifest file

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
* App application’s cache is only updated when its manifest file changes. __you must modify file itself to inform the browser to refresh cached files__
* The manifest is checked <b>twice</b> during the update, one at the start and one after all cached files have been updated
* the HTML file that references your manifest file is automatically cached (though encouraged to include it in the manifest)

### cache status
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

### update cache
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
