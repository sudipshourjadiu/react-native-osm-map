# Adding Interactive Points of Interest (POIs) in Maplibre React Native

Here's a complete example of how to add interactive POIs (markers) that respond to user taps with custom callouts:

## 1. Basic Interactive Marker Example

```jsx
import React, { useState, useRef } from 'react';
import { View, Text, StyleSheet } from 'react-native';
import MapLibreGL from '@maplibre/maplibre-react-native';

const InteractivePOIExample = () => {
  const [selectedPOI, setSelectedPOI] = useState(null);
  const mapRef = useRef(null);

  // Sample POI data
  const pois = [
    {
      id: '1',
      name: 'Central Park',
      description: 'Iconic urban park in Manhattan',
      coordinates: [-73.9688, 40.7851],
    },
    {
      id: '2',
      name: 'Empire State Building',
      description: 'Famous 102-story skyscraper',
      coordinates: [-73.9857, 40.7484],
    },
    {
      id: '3',
      name: 'Statue of Liberty',
      description: 'Symbol of freedom and democracy',
      coordinates: [-74.0445, 40.6892],
    },
  ];

  const handleMarkerPress = (poi) => {
    setSelectedPOI(poi);
    
    // Center map on the selected POI
    mapRef.current?.flyTo(poi.coordinates, 5000);
  };

  return (
    <View style={styles.container}>
      <MapLibreGL.MapView
        ref={mapRef}
        style={styles.map}
        styleURL="https://demotiles.maplibre.org/style.json"
      >
        <MapLibreGL.Camera
          zoomLevel={12}
          centerCoordinate={[-74.0060, 40.7128]}
        />
        
        {/* Render POI markers */}
        {pois.map((poi) => (
          <MapLibreGL.Marker
            key={poi.id}
            coordinate={poi.coordinates}
            onPress={() => handleMarkerPress(poi)}
          >
            <View style={styles.marker}>
              <View style={styles.markerDot} />
            </View>
          </MapLibreGL.Marker>
        ))}
        
        {/* Show callout for selected POI */}
        {selectedPOI && (
          <MapLibreGL.Marker
            coordinate={selectedPOI.coordinates}
            anchor={{ x: 0.5, y: 0 }}
          >
            <View style={styles.callout}>
              <Text style={styles.calloutTitle}>{selectedPOI.name}</Text>
              <Text style={styles.calloutDescription}>
                {selectedPOI.description}
              </Text>
            </View>
          </MapLibreGL.Marker>
        )}
      </MapLibreGL.MapView>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  map: {
    flex: 1,
  },
  marker: {
    alignItems: 'center',
  },
  markerDot: {
    width: 20,
    height: 20,
    borderRadius: 10,
    backgroundColor: '#3498db',
    borderWidth: 3,
    borderColor: 'white',
  },
  callout: {
    backgroundColor: 'white',
    padding: 10,
    borderRadius: 5,
    borderWidth: 1,
    borderColor: '#ddd',
    maxWidth: 200,
  },
  calloutTitle: {
    fontWeight: 'bold',
    marginBottom: 5,
  },
  calloutDescription: {
    color: '#666',
  },
});

export default InteractivePOIExample;
```

## 2. Advanced Features to Enhance Interactivity

### Adding Custom Images as Markers

```jsx
// First, add the image to the map (do this once)
MapLibreGL.images.add('custom-pin', require('./assets/custom-pin.png'));

// Then use it in your marker
<MapLibreGL.Marker coordinate={poi.coordinates}>
  <MapLibreGL.SymbolLayer
    id={`poi-${poi.id}`}
    style={{
      iconImage: 'custom-pin',
      iconSize: 0.5,
    }}
  />
</MapLibreGL.Marker>
```

### Adding Touchable POI Clusters

```jsx
// First add your GeoJSON source
<MapLibreGL.ShapeSource
  id="poi-source"
  shape={{
    type: 'FeatureCollection',
    features: pois.map(poi => ({
      type: 'Feature',
      properties: {
        id: poi.id,
        name: poi.name,
        cluster: false,
      },
      geometry: {
        type: 'Point',
        coordinates: poi.coordinates,
      },
    })),
  }}
  cluster={true}
  clusterRadius={50}
  onPress={(feature) => {
    const coordinates = feature.geometry.coordinates;
    const properties = feature.properties;
    
    if (properties.cluster) {
      // Handle cluster press - zoom in
      mapRef.current?.getZoom().then(zoom => {
        mapRef.current?.flyTo(coordinates, zoom + 2);
      });
    } else {
      // Handle single POI press
      handleMarkerPress(pois.find(p => p.id === properties.id));
    }
  }}
>
  {/* Cluster circles */}
  <MapLibreGL.CircleLayer
    id="clusters"
    filter={['has', 'point_count']}
    style={{
      circleColor: '#3498db',
      circleRadius: 20,
    }}
  />
  
  {/* Cluster count text */}
  <MapLibreGL.SymbolLayer
    id="cluster-count"
    filter={['has', 'point_count']}
    style={{
      textField: '{point_count}',
      textSize: 12,
      textColor: 'white',
    }}
  />
  
  {/* Single POI markers */}
  <MapLibreGL.SymbolLayer
    id="single-poi"
    filter={['!', ['has', 'point_count']]}
    style={{
      iconImage: 'custom-pin',
      iconSize: 0.5,
    }}
  />
</MapLibreGL.ShapeSource>
```

### Adding POI Selection Highlight

```jsx
// Add this layer to your ShapeSource
<MapLibreGL.CircleLayer
  id="selected-poi"
  filter={['==', 'id', selectedPOI?.id || '']}
  style={{
    circleRadius: 20,
    circleColor: 'rgba(52, 152, 219, 0.3)',
    circleStrokeWidth: 2,
    circleStrokeColor: '#3498db',
  }}
/>
```

## 3. Performance Considerations

1. **For many POIs** (100+), use `ShapeSource` with `SymbolLayer` instead of individual `Marker` components
2. **Enable clustering** when dealing with hundreds or thousands of POIs
3. **Use feature state** for interactive styling instead of re-rendering components
4. **Optimize images** used for markers (appropriate size, WebP format)

This implementation gives you fully interactive POIs with:
- Custom marker designs
- Informational callouts on tap
- Smooth map transitions to selected POIs
- Support for clustering when zoomed out
- Visual feedback for selected POIs

You can further customize the appearance and behavior by modifying the styles and event handlers.
