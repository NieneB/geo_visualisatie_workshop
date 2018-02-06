## Interactive Web Map with [MapboxGL.js](https://www.mapbox.com/mapbox-gl-js/api/)

- Load basic Mapbox Map
- Load Custom Vector Tiles
- Style Custom Vector Tiles
- Adding GeoJSON
- Extrusions

## Vector Tile Map

Vector tiles do not contain any information about how to visualize them. Therefore we will always need 2 sources when displaying a web map based on vector tiles:

- Data : The vector tiles
- Styling: JSON formatted style definition. 

The style definition is most commonly supplied in a `JSON` format. Mapbox provides good documentation on the [style specifications] ](https://www.mapbox.com/mapbox-gl-js/style-spec/)


## Making a basic map

We will start with the most basic form of using MapboxGL.js using their already provided styles. In order to use the default styles and data we need a Mapbox token. Later on we will use MapboxGL.js without this token, because we will use our own data and styles! 

A default Mapbox style contains all the elements we need for a map:

- Vector Tiles
- Styling information
- Fonts and Glyps

This is how to initialize a basic map:

```js
var map = new mapboxgl.Map({
        container: 'map-container',
        style: 'mapbox://styles/mapbox/streets-v9',
        hash: true,
        zoom: 11,
        pitch: 60,
        bearing: 62.4,
        center: [ 4.8, 52.4]
    });
```

`new mapboxgl.Map({...});` creates a mapbox GL map! 

`container: "map-container"` places the map in our `<div id="map-container">` object from the HTML file. 

`style: 'mapbox://styles/mapbox/streets-v9'` is the mapbox url for the tiles and style. 

`hash: true` puts the #location in the URL of our map. With `/zoom/lat/long/angle`. See what happens to the URL of your map when you pan around. 

`zoom`,`pitch` , `bearing` and `center` set the starting position of our map when opening it the first time. Now pointing to Amsterdam. 


## Making a custom style
We are also free to create our custom style.
 
```js

{
    "version": 8,
    "name": "Mapbox Streets",
    "sprite": "mapbox://sprites/mapbox/streets-v8",
    "glyphs": "mapbox://fonts/mapbox/{fontstack}/{range}.pbf",
    "sources": {...},
    "layers": [...]
}
```