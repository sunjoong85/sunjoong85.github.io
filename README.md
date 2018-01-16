# sunjoong85.github.io
Sunjoong Kevin Kim Blog

## PWA Introduction
PWA is a concept which provides user experiences that have the reach of the web.

1. Reliable - Work on any network conditions. service worker acting like a proxy which has full control of caches of key resources.
  Even work offline.
  
2. Fast - Load instantly
  -> more then 3 seconds for loading -> 53% of users abandon a site
  -> No janky scroll, response immediately.
  -> More visiting, More staying
  
3. Engaging - Much more feel like a native. Home Screen. Immersive full screen experience. Push notification.


## PWA Checklist
https://developers.google.com/web/progressive-web-apps/checklist

### Baseline
1. Site is served over HTTPs
2. Responsive on tables & mobile devices
  -> responsive design. viewport-friendly site
3. Work offline
  -> Service worker
4. Add to Home screen
  -> App menifesto file
5. First load fast even on 3G
  -> script async, don't block rendering time, PRPL pattern
6. Site works cross-browser
  -> Edge, Chrome, Firefox and Safari
7. Page transitions don't feel like they block on the network
  -> Show skeleton screen on transition and move to next page immediately
8. Each page has a URL
  -> SPA. use router
