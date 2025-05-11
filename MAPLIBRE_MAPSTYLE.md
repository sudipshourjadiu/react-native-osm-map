## Where to Put Your Map Style in Maplibre React Native

In the `MapView` component of `maplibre-react-native`, you have two main ways to specify your map style:

### 1. Using the `mapStyle` Prop (Recommended)

This is the primary way to specify your map style. You can provide either:
- A URL to a style JSON file
- A style object directly

```jsx
<MapLibreGL.MapView
  style={{flex: 1}}
  mapStyle="https://demotiles.maplibre.org/style.json" // URL approach
  // OR
  mapStyle={{ 
    version: 8,
    sources: {...},
    layers: [...]
  }} // Object approach
>
  {/* ... */}
</MapLibreGL.MapView>
```

### 2. Using Default Styles

Maplibre provides some default style URLs you can use:

```jsx
<MapLibreGL.MapView
  style={{flex: 1}}
  mapStyle={MapLibreGL.StyleURL.Default} // Light style
  // OR
  mapStyle={MapLibreGL.StyleURL.Dark} // Dark style
  // OR
  mapStyle={MapLibreGL.StyleURL.Street} // Street style
>
  {/* ... */}
</MapLibreGL.MapView>
```

### Common Style Sources

Here are some popular style sources you can use:

1. **Maplibre Demo Tiles** (good for testing):
   ```jsx
   mapStyle="https://demotiles.maplibre.org/style.json"
   ```

2. **OpenMapTiles** (requires hosting your own server or using a provider):
   ```jsx
   mapStyle="https://your-server/styles/osm-bright/style.json"
   ```

3. **Your own custom style**:
   - Create using [Maputnik](https://maputnik.github.io/)
   - Host the JSON file
   - Reference it via URL

### Important Notes

1. **Performance**: For production apps, host your style files on a CDN or your own server rather than using demo URLs.

2. **CORS**: Ensure your style JSON and any associated resources (sprites, glyphs, tiles) have proper CORS headers if hosted on a different domain.

3. **Offline Use**: For offline maps, you'll need to:
   - Bundle the style JSON with your app
   - Include all required assets (fonts, sprites, etc.)
   - Use offline tile packages

4. **Dynamic Updates**: You can change the style dynamically by updating the `mapStyle` prop.

### Example with Complete Setup

```jsx
import React from 'react';
import MapLibreGL from '@maplibre/maplibre-react-native';

function MyMap() {
  return (
    <MapLibreGL.MapView
      style={{flex: 1}}
      mapStyle={{
        version: 8,
        sources: {
          osm: {
            type: 'raster',
            tiles: ['https://a.tile.openstreetmap.org/{z}/{x}/{y}.png'],
            tileSize: 256,
            attribution: '© OpenStreetMap Contributors'
          }
        },
        layers: [{
          id: 'osm',
          type: 'raster',
          source: 'osm'
        }]
      }}
    >
      <MapLibreGL.Camera
        zoomLevel={10}
        centerCoordinate={[-74.00597, 40.71427]}
      />
    </MapLibreGL.MapView>
  );
}

export default MyMap;
```

Remember that the style JSON must conform to the [MapLibre Style Specification](https://maplibre.org/maplibre-style-spec/).
