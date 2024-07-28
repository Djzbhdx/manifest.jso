Describe your question here.

Use code blocks to share the code snippets which explains the issue.

```javascript
console.log("Hello Dcoder")

```


or add a Dcoder file/project by clicking file link icon in quick bar to link the project or file code where users can reproduce the issue.
example:

[Dcoder Typing Effect](https://code.dcoder.tech/feed/code/5d98add6bdde8b7601995352/Dcoder_typing_effect)

Click above link to try the code and suggest fixes or changes.
self.addEventListener('install', function(event) {
    event.waitUntil(
        caches.open('crypto-trading-cache').then(function(cache) {
            return cache.addAll([
                '/',
                '/index.html',
                '/manifest.json',
                '/service-worker.js',
                'https://cdn.jsdelivr.net/npm/chart.js',
                'https://cdn.jsdelivr.net/npm/@walletconnect/web3-provider@1.6.6/dist/umd/index.min.js',
                'https://cdn.jsdelivr.net/npm/web3@1.3.6/dist/web3.min.js'
            ]);
        })
    );
});

self.addEventListener('fetch', function(event) {
    event.respondWith(
        caches.match(event.request).then(function(response) {
            return response || fetch(event.request);
        })
    );
});