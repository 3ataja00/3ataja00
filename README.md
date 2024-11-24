<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>موقعي</title>
    
    <!-- إضافة manifest للـ PWA -->
    <link rel="manifest" href="#manifest">
    
    <!-- أيقونات التطبيق -->
    <link rel="icon" type="image/png" sizes="192x192" href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNk+A8AAQUBAScY42YAAAAASUVORK5CYII=">
    <link rel="apple-touch-icon" sizes="180x180" href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNk+A8AAQUBAScY42YAAAAASUVORK5CYII=">
    
    <!-- Meta tags للـ PWA -->
    <meta name="theme-color" content="#ffffff">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black">
    <meta name="apple-mobile-web-app-title" content="موقعي">
    <meta name="description" content="وصف الموقع هنا">

    <style>
        /* Reset CSS */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            min-height: 100vh;
            background: #f0f2f5;
        }

        /* تصميم متجاوب */
        .container {
            width: 100%;
            max-width: 1200px;
            margin: 0 auto;
            padding: 15px;
        }

        @media (max-width: 768px) {
            .container {
                padding: 10px;
            }
        }

        /* تنسيق عام */
        .install-prompt {
            position: fixed;
            bottom: 20px;
            left: 20px;
            right: 20px;
            background: white;
            padding: 15px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            display: none;
            z-index: 1000;
        }

        .btn {
            background: #007bff;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            margin: 5px;
        }

        .btn:hover {
            background: #0056b3;
        }

        /* Offline notification */
        .offline-status {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            background: #ff4444;
            color: white;
            text-align: center;
            padding: 10px;
            display: none;
        }
    </style>
</head>
<body>
    <div class="offline-status">
        أنت غير متصل بالإنترنت حالياً
    </div>

    <div class="container">
        <!-- محتوى موقعك الأصلي هنا -->
    </div>

    <div class="install-prompt">
        <p>هل تريد إضافة هذا الموقع إلى شاشتك الرئيسية؟</p>
        <button class="btn" id="installBtn">تثبيت</button>
        <button class="btn" id="dismissBtn">لاحقاً</button>
    </div>

    <script>
        // Service Worker Registration
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                navigator.serviceWorker.register('/service-worker.js')
                    .then(registration => {
                        console.log('ServiceWorker registered');
                    })
                    .catch(err => {
                        console.log('ServiceWorker registration failed: ', err);
                    });
            });
        }

        // PWA Install Prompt
        let deferredPrompt;
        const installPrompt = document.querySelector('.install-prompt');
        const installBtn = document.querySelector('#installBtn');
        const dismissBtn = document.querySelector('#dismissBtn');

        window.addEventListener('beforeinstallprompt', (e) => {
            e.preventDefault();
            deferredPrompt = e;
            installPrompt.style.display = 'block';
        });

        installBtn.addEventListener('click', async () => {
            if (deferredPrompt) {
                deferredPrompt.prompt();
                const { outcome } = await deferredPrompt.userChoice;
                if (outcome === 'accepted') {
                    console.log('User accepted the install prompt');
                }
                deferredPrompt = null;
                installPrompt.style.display = 'none';
            }
        });

        dismissBtn.addEventListener('click', () => {
            installPrompt.style.display = 'none';
        });

        // Offline/Online Status
        const offlineStatus = document.querySelector('.offline-status');

        window.addEventListener('online', () => {
            offlineStatus.style.display = 'none';
        });

        window.addEventListener('offline', () => {
            offlineStatus.style.display = 'block';
        });
    </script>

    <!-- Inline manifest.json -->
    <script type="application/manifest+json" id="manifest">
    {
        "name": "اسم موقعك",
        "short_name": "موقعي",
        "start_url": "/",
        "display": "standalone",
        "background_color": "#ffffff",
        "theme_color": "#ffffff",
        "orientation": "portrait",
        "icons": [
            {
                "src": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNk+A8AAQUBAScY42YAAAAASUVORK5CYII=",
                "sizes": "192x192",
                "type": "image/png"
            },
            {
                "src": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNk+A8AAQUBAScY42YAAAAASUVORK5CYII=",
                "sizes": "512x512",
                "type": "image/png"
            }
        ]
    }
    </script>

    <!-- Inline service-worker.js -->
    <script id="service-worker" type="text/javascript">
    // Service Worker Code
    const CACHE_NAME = 'my-site-cache-v1';
    const urlsToCache = [
        '/',
        '/index.html',
        '/styles.css',
        '/script.js'
    ];

    self.addEventListener('install', event => {
        event.waitUntil(
            caches.open(CACHE_NAME)
                .then(cache => {
                    return cache.addAll(urlsToCache);
                })
        );
    });

    self.addEventListener('fetch', event => {
        event.respondWith(
            caches.match(event.request)
                .then(response => {
                    if (response) {
                        return response;
                    }
                    return fetch(event.request);
                })
        );
    });
    </script>
</body>
</html>

