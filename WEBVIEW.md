To use **OpenStreetMap (OSM)** with **Leaflet.js** inside a **React Native WebView**, you can follow these steps to embed an interactive, fully functional OSM map using HTML+JavaScript inside a WebView component.

---

### ✅ 1. Install WebView

```bash
npm install react-native-webview
```

For iOS:

```bash
cd ios && pod install
```

---

### ✅ 2. Create the HTML with Leaflet

Here's a simple Leaflet map loading OpenStreetMap tiles:

```html
<!-- leaflet.html -->
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Leaflet Map</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link
    rel="stylesheet"
    href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
  />
  <style>
    #map { height: 100vh; width: 100vw; margin: 0; padding: 0; }
    body, html { margin: 0; padding: 0; }
  </style>
</head>
<body>
  <div id="map"></div>

  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script>
    const map = L.map('map').setView([51.505, -0.09], 13);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '© OpenStreetMap contributors'
    }).addTo(map);
  </script>
</body>
</html>
```

---

### ✅ 3. Load HTML in WebView

You can include that HTML as a string or as a local asset in your React Native project.

#### 📄 Option A: As an inline HTML string

```jsx
import React from 'react';
import { WebView } from 'react-native-webview';

const leafletHtml = `
  <!DOCTYPE html>
  <html>
  <head>
    <meta charset="utf-8" />
    <title>Leaflet Map</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <style>
      #map { height: 100vh; width: 100vw; margin: 0; padding: 0; }
      body, html { margin: 0; padding: 0; }
    </style>
  </head>
  <body>
    <div id="map"></div>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script>
      const map = L.map('map').setView([51.505, -0.09], 13);
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '© OpenStreetMap contributors'
      }).addTo(map);
    </script>
  </body>
  </html>
`;

export default function MapScreen() {
  return (
    <WebView
      originWhitelist={['*']}
      source={{ html: leafletHtml }}
      style={{ flex: 1 }}
    />
  );
}
```

#### 📄 Option B: From a local HTML file (better for large or complex setups)

* Place your `leaflet.html` in `assets` folder (e.g., `./assets/leaflet.html`)
* Load using:

```jsx
<WebView
  originWhitelist={['*']}
  source={require('./assets/leaflet.html')}
  style={{ flex: 1 }}
/>
```

For iOS, make sure the asset is copied to the bundle in Xcode. For Android, place the file in `android/app/src/main/assets`.

---

### 🌐 Interactivity & Customization

You can use Leaflet’s plugins and APIs to:

* Add markers or popups
* Draw shapes
* Handle user clicks and route paths
* Communicate with React Native via `window.ReactNativeWebView.postMessage(...)`

Let me know if you want help with two-way communication or map interactivity from the React Native side.
