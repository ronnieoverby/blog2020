---
title: "Blazor Preview PWA Diffs"
date: 2020-03-21T10:42:22-04:00
draft: true
tags: ["blazor","pwa"]
---

The latest Blazor webassembly template preview ([3.2.0-preview2.20160.5](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Templates/3.2.0-preview2.20160.5)) can scaffold a new app with PWA configuration.

I was curious what exactly this would add to my app. Here's what I found:


{{< figure src="/images/blazor-preview-pwa-diffs/2020-03-21-10-56-43.png" title="Folder Diff" >}}

Looking at the folder diff, there are some new files and some changed files.

{{< figure src="/images/blazor-preview-pwa-diffs/2020-03-21-11-40-15.png" title="Index.html Diff" >}}

Unsurprisingly `Index.html` is linking to the newly added [web manifest](https://developer.mozilla.org/en-US/docs/Web/Manifest) file, `manifest.json` and  registering the service worker.

Now look at the client csproj file:

{{< figure src="/images/blazor-preview-pwa-diffs/2020-03-21-11-13-37.png" title="Client csproj Diff" >}}

Two things:

 1. The `ServiceWorkerAssetsManifest` element is interesting. What's this doing? 
    - This is defining a compile-time generated .js resource, `service-worker-assets.js`, that will be referenced by the service worker.
    - This file contains an associative array of stuff that your PWA should download and cache for offline use.
 2. Release builds are substituting `service-worker.js`.
    - The substituted file, `service-worker.published.js`:
      1.  Imports `service-worker-assets.js`
      2.  Sets up event handlers for service worker installation/activation and fetch proxying.
      3.  On install, creates fetch requests and caches the responses for each asset that is needed (there's a list of file extension regexes).
      4.  On fetch, attempts to serve GET requests from the cache.
    - By default, non-release builds have no offline capability.

It seems that we application developers are responsible for setting up fetch proxying for any additional server communication; Microsoft doesn't establish any pattern for us to follow.
They have provided [some guidance for developing offline PWAs with blazor](https://docs.microsoft.com/en-us/aspnet/core/blazor/progressive-web-app?view=aspnetcore-3.1&tabs=visual-studio#offline-support), including some explanations of what you'll find in `service-worker.published.js` and some general gotchas to watch out for.


