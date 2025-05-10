Here’s a working **example of communication between Leaflet (inside WebView) and React Native** using `window.ReactNativeWebView.postMessage()` and `onMessage`:

---

### ✅ What You’ll Learn:

* How Leaflet sends messages to React Native when user clicks the map.
* How React Native listens and responds to those messages.

---

### 🔁 1. Modified HTML with Leaflet + PostMessage

```js
const leafletHtml = `
  <!DOCTYPE html>
  <html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link
      rel="stylesheet"
      href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
    />
    <style>
      #map { height: 100vh; width: 100vw; margin: 0; padding: 0; }
      html, body { margin: 0; padding: 0; }
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

      // Send clicked coordinates to React Native
      map.on('click', function(e) {
        const message = {
          type: 'map_click',
          lat: e.latlng.lat,
          lng: e.latlng.lng
        };
        window.ReactNativeWebView?.postMessage(JSON.stringify(message));
      });

      // Handle message from React Native (optional)
      window.addEventListener('message', (event) => {
        const data = JSON.parse(event.data);
        if (data.type === 'set_marker') {
          L.marker([data.lat, data.lng]).addTo(map).bindPopup('Marker set by React Native');
        }
      });
    </script>
  </body>
  </html>
`;
```

---

### 📱 2. React Native Component with WebView Communication

```jsx
import React, { useRef } from 'react';
import { Alert, View } from 'react-native';
import { WebView } from 'react-native-webview';

export default function MapScreen() {
  const webviewRef = useRef();

  const handleMapMessage = (event) => {
    const data = JSON.parse(event.nativeEvent.data);
    if (data.type === 'map_click') {
      Alert.alert('Map Clicked!', `Lat: ${data.lat}\nLng: ${data.lng}`);

      // Send a command back to WebView to place a marker
      const jsCode = `
        window.dispatchEvent(new MessageEvent('message', {
          data: JSON.stringify({
            type: 'set_marker',
            lat: ${data.lat},
            lng: ${data.lng}
          })
        }));
      `;
      webviewRef.current.injectJavaScript(jsCode);
    }
  };

  return (
    <View style={{ flex: 1 }}>
      <WebView
        ref={webviewRef}
        originWhitelist={['*']}
        source={{ html: leafletHtml }}
        onMessage={handleMapMessage}
        javaScriptEnabled={true}
        style={{ flex: 1 }}
      />
    </View>
  );
}
```

---

### 🔄 Result:

* When the user clicks on the map, React Native shows an alert with the coordinates.
* Then it sends a message back to Leaflet to place a marker at the same spot.

Would you like to include other features like real-time tracking, drawing shapes, or offline maps too?
