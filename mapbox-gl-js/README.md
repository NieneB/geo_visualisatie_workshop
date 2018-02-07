## Interactive Web Map with [MapboxGL.js](https://www.mapbox.com/mapbox-gl-js/api/)

- [ ] Load basic Mapbox Map
- [ ] Load Custom Vector Tiles
- [ ] Adding GeoJSON
- [ ] Fonts
- [ ] Extrusion

## 1. Basic Vector Tile Map
[01_basemap.html](01_basemap.html)

Vector tiles do not contain any information about how to visualize them. Therefore we will always need 2 sources when displaying a web map based on vector tiles:

- Data : The vector tiles
- Styling: JSON formatted style definition. 

The style definition is most commonly supplied in a `JSON` format. Mapbox provides good documentation on the [style specifications] ](https://www.mapbox.com/mapbox-gl-js/style-spec/)

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


## 2. Making a custom style

[02_custom_map.html](02_custom_map.html) & [02_mystyle.json](02_mystyle.json)


We are also free to create our custom style. For this we need to create a separte style JSON file. A Mapbox style spec has the following Root properties:

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

The 2 most important for now are the `sources` and the `layers`. The sources tell us where our data is comming from. Vector Tiles or GeoJSON data for example. By setting `layers` we can style every seperate layer availble in the vector tiles and assigning it colors etc. 

For now we start with 1 source, namely our vector tiles that are hosted by PDOK:

```
"pdok":{
    "type": "vector",
    "tiles":  ["http://geodata.nationaalgeoregister.nl/beta/topotiles/{z}/{x}/{y}.pbf"]
}
```


Next we can create layers, accordingly on what layers are available in the vector data. The first layer is a background layer with the background fill color white. Then we call a admin layer which is available in the vector tiles. We apply a filter to the layer in order to get only the province boundaries. 

```
"layers":[ 
        { 
            "id":  "background",
            "type": "background",
            "paint": {
                "background-color":"#FFFFFF"
                }
        },
        {
            "id": "admin",
            "type": "fill",
            "source": "pdok",
            "source-layer": "admin",
            "maxzoom": 22,
            "minzoom": 0,
            "filter": ["==", "lod1", "province"],
            "paint": {
                "fill-color" :"#54D8CC",
                "fill-outline-color": "#ffffff"
            }
        }
    ]
```

We can go on and on with adding layers. Have a look at http://geodata.nationaalgeoregister.nl/beta/topotiles-viewer/styles/achtergrond.json to see a complete styling json for getting a map of the Netherlands. 

The only thing we have to do is change the `style` reference in our JavaScript. (and we can delete the Mapbox token)

```js
var map = new mapboxgl.Map({
        container: 'map-container',
        style: '02_mystyle.json',
        hash: true,
        zoom: 11,
        pitch: 60,
        bearing: 62.4,
        center: [ 4.8, 52.4]
    });
```
## 3. Adding a GeoJSON source

[03_custom_map_geojson.html](03_custom_map_geojson.html) & [03_mystyle.json](03_mystyle.json)

We can add mutiple sources to our style spec. Let's add our points. For this we need to run a local serve again! 


```json
"aardbeving_points":{
    "type": "geojson",
    "data": "http://localhost:8000/data/aardbevingen_NL.geojson"
}
```

And define a point layer:

```
{
    "id": "aardbeving",
    "type": "circle",
    "source": "aardbeving_points",
    "maxzoom": 22,
    "minzoom": 0,
    "paint":{
        "circle-radius": 5,
        "circle-color":{
            "type": "categorical",
            "property": "TYPE",
            "stops":[
                ["ind", "#f3ec9c"],
                ["tec", "#F5CB5B"]
            ]
        }
    }
}
```


## 4. Fonts Glyps

[04_custom_map_fonts.html](04_custom_map_fonts.html) & [04_mystyle.json](04_mystyle.json)

In order to add labels we need a font. Fonts have also to be converted to pbf, in order to render them with WebGL. Mapbox provides some fonts but then you need the Mapbox token again.
We add the `glyphs` reference at the top of our style specification. See [[04_mystyle.json]]

Using Mapbox fonts is possible by adding the glyphs to the style.json:

    "glyphs": "mapbox://fonts/openmaptiles/{fontstack}/{range}.pbf",

If you do not want to depend on the Mapbox token we can use free providers or make our own. For exapmle using fonts from openmaptiles:

    "glyphs": "http://fonts.openmaptiles.org/{fontstack}/{range}.pbf",

Making a font pbf range ourselfs is also possible: https://github.com/mapbox/fontmachine

