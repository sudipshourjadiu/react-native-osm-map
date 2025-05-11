# Using Server-Side POIs with Maplibre React Native

When your Points of Interest (POIs) come from a server, you'll need to fetch them and properly integrate them with your map. Here's a comprehensive approach:

## 1. Fetching POIs from a Server

```jsx
import React, { useState, useEffect } from 'react';
import { ActivityIndicator } from 'react-native';
import MapLibreGL from '@maplibre/maplibre-react-native';

const ServerPOIExample = () => {
  const [pois, setPois] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [selectedPOI, setSelectedPOI] = useState(null);

  useEffect(() => {
    const fetchPOIs = async () => {
      try {
        const response = await fetch('https://your-api-endpoint.com/pois');
        const data = await response.json();
        setPois(data.features || data); // Adapt based on your API response format
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchPOIs();
  }, []);

  if (loading) return <ActivityIndicator size="large" />;
  if (error) return <Text>Error: {error}</Text>;

  return (
    <MapLibreGL.MapView style={{ flex: 1 }}>
      <MapLibreGL.Camera
        zoomLevel={12}
        centerCoordinate={[-74.0060, 40.7128]}
      />

      {/* Render POIs from server */}
      <MapLibreGL.ShapeSource
        id="poi-source"
        shape={{
          type: 'FeatureCollection',
          features: pois.map(poi => ({
            ...poi,
            properties: {
              ...poi.properties,
              // Ensure each feature has a unique id
              id: poi.properties?.id || `${poi.geometry.coordinates.join('-')}`,
            }
          }))
        }}
        onPress={(e) => {
          const feature = e.features[0];
          setSelectedPOI(feature);
        }}
      >
        <MapLibreGL.SymbolLayer
          id="poi-layer"
          style={{
            iconImage: ['get', 'icon'] || 'marker', // Use custom icon if specified
            iconSize: 0.5,
            iconAllowOverlap: true,
          }}
        />
      </MapLibreGL.ShapeSource>

      {/* Selected POI callout */}
      {selectedPOI && (
        <MapLibreGL.Marker
          coordinate={selectedPOI.geometry.coordinates}
          anchor={{ x: 0.5, y: 0 }}
        >
          <View style={styles.callout}>
            <Text>{selectedPOI.properties?.name || 'POI'}</Text>
            {selectedPOI.properties?.description && (
              <Text>{selectedPOI.properties.description}</Text>
            )}
          </View>
        </MapLibreGL.Marker>
      )}
    </MapLibreGL.MapView>
  );
};
```

## 2. Handling Different API Response Formats

### Option A: GeoJSON Format (Recommended)

If your server returns proper GeoJSON:

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {
        "id": "1",
        "name": "Central Park",
        "icon": "park"
      },
      "geometry": {
        "type": "Point",
        "coordinates": [-73.9688, 40.7851]
      }
    }
  ]
}
```

Use it directly:

```jsx
<MapLibreGL.ShapeSource
  id="poi-source"
  shape={serverData} // Your API response
>
  {/* Layers */}
</MapLibreGL.ShapeSource>
```

### Option B: Custom API Format

If your API returns a custom format:

```json
[
  {
    "id": "1",
    "name": "Central Park",
    "lat": 40.7851,
    "lng": -73.9688,
    "category": "park"
  }
]
```

Convert it to GeoJSON:

```jsx
const toGeoJSON = (items) => ({
  type: 'FeatureCollection',
  features: items.map(item => ({
    type: 'Feature',
    properties: {
      ...item,
      // Move coordinates out of properties
    },
    geometry: {
      type: 'Point',
      coordinates: [item.lng, item.lat]
    }
  }))
});

// Usage
<MapLibreGL.ShapeSource
  id="poi-source"
  shape={toGeoJSON(serverData)}
>
  {/* Layers */}
</MapLibreGL.ShapeSource>
```

## 3. Advanced Features with Server POIs

### Dynamic Filtering

```jsx
const [activeFilter, setActiveFilter] = useState('all');

<MapLibreGL.ShapeSource
  id="poi-source"
  shape={poiData}
>
  <MapLibreGL.SymbolLayer
    id="poi-layer"
    filter={activeFilter === 'all' 
      ? ['!=', 'id', ''] 
      : ['==', 'category', activeFilter]}
    style={/* ... */}
  />
</MapLibreGL.ShapeSource>
```

### Real-time Updates with WebSockets

```jsx
useEffect(() => {
  const ws = new WebSocket('wss://your-api-endpoint.com/pois/updates');
  
  ws.onmessage = (e) => {
    const newPOI = JSON.parse(e.data);
    setPois(prev => [...prev, newPOI]);
  };

  return () => ws.close();
}, []);
```

### Pagination for Large Datasets

```jsx
const [page, setPage] = useState(1);
const [allPOIs, setAllPOIs] = useState([]);

const loadMorePOIs = async () => {
  const response = await fetch(`https://your-api.com/pois?page=${page}`);
  const newPOIs = await response.json();
  setAllPOIs(prev => [...prev, ...newPOIs]);
  setPage(prev => prev + 1);
};

// Use in onRegionDidChange to load when map moves
<MapLibreGL.MapView
  onRegionDidChange={loadMorePOIs}
>
  {/* ... */}
</MapLibreGL.MapView>
```

## 4. Performance Optimization

1. **Use clustering** for large datasets:
   ```jsx
   <MapLibreGL.ShapeSource
     id="poi-source"
     shape={poiData}
     cluster={true}
     clusterRadius={50}
   >
     {/* Cluster layers */}
   </MapLibreGL.ShapeSource>
   ```

2. **Implement viewport filtering** - only request POIs visible in current map bounds:
   ```jsx
   const [bounds, setBounds] = useState(null);
   
   useEffect(() => {
     if (bounds) {
       fetch(`https://api.com/pois?bbox=${bounds.join(',')}`);
     }
   }, [bounds]);
   
   <MapLibreGL.MapView
     onRegionDidChange={async (region) => {
       const visibleBounds = await mapRef.current?.getVisibleBounds();
       setBounds(visibleBounds);
     }}
   >
     {/* ... */}
   </MapLibreGL.MapView>
   ```

3. **Cache responses** to avoid refetching the same areas.

## 5. Error Handling and Loading States

```jsx
const renderPOIs = () => {
  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error loading POIs</Text>;
  if (pois.length === 0) return <Text>No POIs found</Text>;

  return (
    <MapLibreGL.ShapeSource
      id="poi-source"
      shape={{ type: 'FeatureCollection', features: pois }}
    >
      {/* Layers */}
    </MapLibreGL.ShapeSource>
  );
};

return (
  <MapLibreGL.MapView>
    {renderPOIs()}
  </MapLibreGL.MapView>
);
```

This approach gives you a robust solution for working with server-side POIs that includes:
- Data fetching and transformation
- Proper error handling
- Performance optimizations
- Support for real-time updates
- Interactive features like filtering and selection
