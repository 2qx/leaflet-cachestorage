# leaflet-cachestorage
Leaflet plugin to cache tiles in cacheStorage

**Experimental** but working

Based on https://github.com/MazeMap/Leaflet.TileLayer.PouchDBCached


Stores png tiles in a cacheStorage called `offline-map-tiles` and serves them from the service with good latency (<10ms). This puts roughly 50MB of pngs in app storage under casual use. 

Import the plugin with:

    <script src="js/leaflet/leaflet.cachestorage.js"></script>
   
   
When configuring your map later pass the following parameters. The `useCache` parameter should probably test if the ServiceWorker endpoint (`/offline-map-tiles`) is available, but this configuration works well for debugging. 

        var layer = L.tileLayer(mapTileProvider, 
            {
                useCache: (location.protocol === 'https:' || location.hostname === 'localhost'),
                useOnlyCache: false,
                crossOrigin: true
            });

Uses the service worker to serve 

    self.addEventListener('fetch', function (event) {

      if (event.request.url.includes('offline-map-tiles')) {
          event.respondWith(
            caches.match(event.request).then(cachedResponse => {
              //console.log("[ServiceWorker] Request "+event.request.url)
                if (cachedResponse) {
                    return cachedResponse;
                }else{
                  console.log("[ServiceWorker] Error: 404 "+ event.request.url)
                }
              }));
        }
    }
    
    
It should also be quite possible to refactor this to use a WebWorker or the main thread.      

There is no cache cleaning or expiration.
