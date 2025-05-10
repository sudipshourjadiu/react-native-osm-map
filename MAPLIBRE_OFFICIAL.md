Got it! Here's the **minimal working example** that **only loads the map** from the **MapLibre demo server** in a React Native app using `@maplibre/maplibre-react-native`.

---

### ✅ Step-by-Step Code: `MapDemo.js`

```jsx
import React from 'react';
import { View, StyleSheet } from 'react-native';
import MapView from '@maplibre/maplibre-react-native';

const MapDemo = () => {
  return (
    <View style={styles.page}>
      <MapView
        style={styles.map}
        styleURL="https://demotiles.maplibre.org/style.json"
        compassEnabled={true}
        zoomLevel={10}
        centerCoordinate={[13.388860, 52.517037]} // Berlin
      />
    </View>
  );
};

const styles = StyleSheet.create({
  page: { flex: 1 },
  map: { flex: 1 },
});

export default MapDemo;
```

---

### 🧩 Usage

Make sure you installed the package:

```bash
npm install @maplibre/maplibre-react-native
npx pod-install
```

And that you added **internet permissions** on Android:

```xml
<!-- AndroidManifest.xml -->
<uses-permission android:name="android.permission.INTERNET" />
```

---

Let me know if you'd like to add zoom controls, markers, or switch to offline tiles next.
