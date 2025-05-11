# Creating Interactive POI Screens with Tileserver-GL and Maplibre React Native

When you're using Tileserver-GL as your map server and want to make the existing POIs interactive with detailed information screens, here's a comprehensive solution:

## 1. Understanding Your Tileserver-GL POIs

First, verify that your vector tiles from Tileserver-GL contain POI data by:
1. Checking your style.json to see which layers contain POIs
2. Visiting `http://your-tileserver-url/data/your-source.json` to inspect available layers

## 2. Querying POIs from Vector Tiles

```jsx
import React, { useState, useRef } from 'react';
import { View, Text, StyleSheet, Modal, TouchableOpacity, Image } from 'react-native';
import MapLibreGL from '@maplibre/maplibre-react-native';

const InteractivePOIScreen = () => {
  const mapRef = useRef(null);
  const [selectedPOI, setSelectedPOI] = useState(null);
  const [poiDetailsVisible, setPoiDetailsVisible] = useState(false);

  // Your tileserver endpoint
  const tileserverUrl = 'http://your-tileserver-address/data/your-source/{z}/{x}/{y}.pbf';
  const sourceId = 'your-source-id'; // Should match source in your style

  const handleMapPress = async (e) => {
    const features = await mapRef.current?.queryRenderedFeaturesAtPoint(
      e.properties.coordinate,
      ['==', '$type', 'Point'], // Filter for point features
      ['poi-layer'] // Your POI layer name
    );

    if (features && features.length > 0) {
      setSelectedPOI(features[0]);
      setPoiDetailsVisible(true);
    }
  };

  return (
    <View style={styles.container}>
      <MapLibreGL.MapView
        ref={mapRef}
        style={styles.map}
        styleURL="http://your-tileserver-address/styles/your-style/style.json"
        onPress={handleMapPress}
      >
        <MapLibreGL.Camera
          zoomLevel={12}
          centerCoordinate={[your-lng, your-lat]}
        />

        {/* Add your vector tile source */}
        <MapLibreGL.VectorSource
          id={sourceId}
          url={tileserverUrl}
        >
          {/* POI layer - should match your style layer */}
          <MapLibreGL.SymbolLayer
            id="poi-layer"
            sourceLayerID="pois" // Layer name in your vector tiles
            style={{
              iconImage: ['get', 'icon'],
              iconSize: 0.5,
              iconAllowOverlap: true,
            }}
          />
        </MapLibreGL.VectorSource>
      </MapLibreGL.MapView>

      {/* POI Detail Modal */}
      <Modal
        visible={poiDetailsVisible}
        animationType="slide"
        transparent={true}
        onRequestClose={() => setPoiDetailsVisible(false)}
      >
        <View style={styles.modalContainer}>
          {selectedPOI && (
            <View style={styles.detailCard}>
              <TouchableOpacity 
                style={styles.closeButton}
                onPress={() => setPoiDetailsVisible(false)}
              >
                <Text style={styles.closeButtonText}>×</Text>
              </TouchableOpacity>
              
              <Text style={styles.poiTitle}>{selectedPOI.properties?.name || 'POI'}</Text>
              
              {selectedPOI.properties?.image && (
                <Image 
                  source={{ uri: selectedPOI.properties.image }} 
                  style={styles.poiImage}
                />
              )}
              
              <View style={styles.detailRow}>
                <Text style={styles.detailLabel}>Category:</Text>
                <Text style={styles.detailValue}>{selectedPOI.properties?.category || 'N/A'}</Text>
              </View>
              
              <View style={styles.detailRow}>
                <Text style={styles.detailLabel}>Address:</Text>
                <Text style={styles.detailValue}>{selectedPOI.properties?.address || 'N/A'}</Text>
              </View>
              
              {selectedPOI.properties?.description && (
                <Text style={styles.poiDescription}>
                  {selectedPOI.properties.description}
                </Text>
              )}
              
              <TouchableOpacity style={styles.directionsButton}>
                <Text style={styles.directionsButtonText}>Get Directions</Text>
              </TouchableOpacity>
            </View>
          )}
        </View>
      </Modal>
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
  modalContainer: {
    flex: 1,
    justifyContent: 'flex-end',
    backgroundColor: 'rgba(0,0,0,0.5)',
  },
  detailCard: {
    backgroundColor: 'white',
    borderTopLeftRadius: 20,
    borderTopRightRadius: 20,
    padding: 20,
    maxHeight: '70%',
  },
  closeButton: {
    alignSelf: 'flex-end',
    padding: 10,
  },
  closeButtonText: {
    fontSize: 24,
    color: '#333',
  },
  poiTitle: {
    fontSize: 22,
    fontWeight: 'bold',
    marginBottom: 15,
  },
  poiImage: {
    width: '100%',
    height: 150,
    borderRadius: 10,
    marginBottom: 15,
  },
  detailRow: {
    flexDirection: 'row',
    marginBottom: 10,
  },
  detailLabel: {
    fontWeight: 'bold',
    width: 100,
  },
  detailValue: {
    flex: 1,
  },
  poiDescription: {
    marginTop: 15,
    color: '#666',
    lineHeight: 20,
  },
  directionsButton: {
    backgroundColor: '#4285F4',
    padding: 15,
    borderRadius: 8,
    marginTop: 20,
    alignItems: 'center',
  },
  directionsButtonText: {
    color: 'white',
    fontWeight: 'bold',
  },
});

export default InteractivePOIScreen;
```

## 3. Key Implementation Details

### Querying POI Data

1. **Identify your POI layer**: Check your Tileserver-GL style.json to find the layer name containing POIs
2. **Query features**: Use `queryRenderedFeaturesAtPoint` to get POI data at tapped location
3. **Filter results**: Focus on point features in your POI layer

### Enhancing POI Data

Since vector tiles typically contain minimal properties, you can:

1. **Fetch additional details** when a POI is selected:
```jsx
const fetchPOIDetails = async (poiId) => {
  const response = await fetch(`http://your-api/pois/${poiId}`);
  return await response.json();
};

// In your handler:
const features = await mapRef.current?.queryRenderedFeaturesAtPoint(...);
if (features.length > 0) {
  const details = await fetchPOIDetails(features[0].properties.id);
  setSelectedPOI({ ...features[0], details });
}
```

2. **Add custom images** by extending your POI properties:
```jsx
// In your detail card:
{selectedPOI.details?.images?.map((img, index) => (
  <Image key={index} source={{ uri: img }} style={styles.poiImage} />
))}
```

### Performance Optimization

1. **Use feature state** for highlighting selected POIs:
```jsx
// Add to your SymbolLayer:
style={{
  iconSize: [
    'case',
    ['==', ['get', 'id'], selectedPOI?.properties?.id || ''],
    0.8, // Selected size
    0.5  // Default size
  ],
}}
```

2. **Implement viewport-aware loading** if you have many POIs:
```jsx
const [visibleBounds, setVisibleBounds] = useState(null);

useEffect(() => {
  const updateBounds = async () => {
    const bounds = await mapRef.current?.getVisibleBounds();
    setVisibleBounds(bounds);
    // Use bounds to filter or load POIs
  };
  
  // Debounce this call
  const timeout = setTimeout(updateBounds, 500);
  return () => clearTimeout(timeout);
}, [mapRegion]);
```

## 4. Advanced Features

### 1. Interactive Clustering

```jsx
<MapLibreGL.VectorSource
  id="poi-source"
  url={tileserverUrl}
  cluster={true}
  clusterRadius={50}
  clusterMaxZoom={14}
>
  <MapLibreGL.SymbolLayer
    id="clusters"
    filter={['has', 'point_count']}
    style={{
      iconSize: [
        'step',
        ['get', 'point_count'],
        0.5,  // Default size
        10, 0.6,
        50, 0.7
      ],
      iconColor: '#4285F4'
    }}
  />
  
  <MapLibreGL.SymbolLayer
    id="cluster-count"
    filter={['has', 'point_count']}
    style={{
      textField: '{point_count}',
      textSize: 12,
      textColor: '#fff'
    }}
  />
  
  <MapLibreGL.SymbolLayer
    id="single-poi"
    filter={['!', ['has', 'point_count']]}
    style={{
      iconImage: ['get', 'icon'],
      iconSize: 0.5
    }}
  />
</MapLibreGL.VectorSource>
```

### 2. POI Filtering

```jsx
const [activeCategory, setActiveCategory] = useState(null);

// In your SymbolLayer:
filter={[
  'all',
  ['!', ['has', 'point_count']], // Exclude clusters
  activeCategory 
    ? ['==', ['get', 'category'], activeCategory]
    : ['!=', 'id', '']
]}
```

### 3. Directions Integration

```jsx
const handleDirections = () => {
  const { coordinates } = selectedPOI.geometry;
  const url = Platform.select({
    ios: `http://maps.apple.com/?daddr=${coordinates[1]},${coordinates[0]}`,
    android: `https://www.google.com/maps/dir/?api=1&destination=${coordinates[1]},${coordinates[0]}`
  });
  Linking.openURL(url);
};
```

## 5. Troubleshooting Tips

1. **If POIs aren't queryable**:
   - Ensure your layer is marked as "interactive" in Tileserver-GL config
   - Verify the layer is included in your vector tiles (check the tileserver's `tilejson.json`)

2. **If properties are missing**:
   - Check your Tileserver-GL config to ensure all needed properties are included
   - You may need to modify your `config.json` to include more properties:
   ```json
   "vector_layers": [
     {
       "layer_name": "pois",
       "minzoom": 12,
       "fields": {
         "name": "String",
         "category": "String",
         "address": "String"
       }
     }
   ]
   ```

3. **For better performance**:
   - Use `queryRenderedFeaturesInRect` when zoomed out instead of point queries
   - Implement server-side filtering if you have thousands of POIs

This implementation gives you a complete solution for making Tileserver-GL POIs interactive with detailed information screens, including:
- POI selection and detail display
- Customizable information cards
- Performance optimizations
- Advanced features like clustering and filtering
- Directions integration
