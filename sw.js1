// Service Worker – Ready for dynamic caching
const CACHE_NAME = 'my-site-cache-v1';
const DYNAMIC_CACHE = 'dynamic-cache-v1';
const OFFLINE_URL = '/offline.html';

// List of files to cache initially
const urlsToCache = [
  '/',
  '/index.html',
  '/style.css',
  '/main.js',
  OFFLINE_URL,
];

// Install – cache static assets
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(urlsToCache))
  );
  self.skipWaiting();
});

// Activate – clean old caches
self.addEventListener('activate', event => {
  const cacheWhitelist = [CACHE_NAME, DYNAMIC_CACHE];
  event.waitUntil(
    caches.keys().then(keys =>
      Promise.all(
        keys.map(key => {
          if (!cacheWhitelist.includes(key)) {
            return caches.delete(key);
          }
        })
      )
    )
  );
  self.clients.claim();
});

// Fetch – serve cached, fallback to network, dynamic caching
self.addEventListener('fetch', event => {
  const request = event.request;
  if (request.method !== 'GET') return;

  event.respondWith(
    caches.match(request).then(cachedResponse => {
      if (cachedResponse) return cachedResponse;

      return fetch(request).then(networkResponse => {
        return caches.open(DYNAMIC_CACHE).then(cache => {
          if (networkResponse.type === 'basic' || networkResponse.type === 'cors') {
            cache.put(request, networkResponse.clone());
          }
          return networkResponse;
        });
      }).catch(() => {
        if (request.mode === 'navigate') {
          return caches.match(OFFLINE_URL);
        }
      });
    })
  );
});
