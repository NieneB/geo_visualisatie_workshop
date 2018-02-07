## Interactive Web Map with [D3.js](https://d3js.org/)

We're going to make a map of earthquakes in the Netherlands using D3JS as a mapping library.

- [ ] Basic GeoJSON map
- [ ] From GeoJSON to TopoJSON
- [ ] Add multiple sources
- [ ] Visualise your data
- [ ] Add basemap

## 1. Create a basic map

[01_geojson.html](01_geojson.html)

The first thing we will practice is to create a basic map with D3. As D3 is all about drawing vectors in the web browser, we will use GeoJSON, a vector-based file that contains the polygons representing the municipalities of the Netherlands. We will do this in the Web Mercator projection.

1. Set up the SVG element on the page

```javascript
let width = 450,
  height = 600,
  svg = d3.select("body")
    .append("svg")
    .attr('width', width)
    .attr('height', height);
```

2. Pull in the geo data

```javascript
  d3.json("../data/gemeenten2017.geojson", function(error, gemeenten) {
    if (error) throw error;
```

3. Set up the projection and view

```javascript
  let projection = d3.geoMercator()
    .fitSize([width, height], gemeenten);
```

4. Draw the geo data on the map

```javascript
  let path = d3.geoPath()
    .projection(projection);

  svg.append("g")
    .selectAll("path")
    .data(gemeenten.features)
    .enter()
    .append("path")
    .attr("class","gemeente")
    .attr("d", path)
    .append("title")
    .text(function(d) { return d.properties.statnaam});
```

## 2. Use TopoJSON instead

[02_topojson.html](02_topojson.html)

GeoJSON can be used for points, lines and polygons. So we can add any other dataset if we want. But GeoJSON is quite 'verbose', so file sizes increase quickly. Especially on the web, this causes performance problems. Luckily, someone (Mike Bostock, the creator of D3.js) developed an extension to GeoJSON which can reduce filesize greatly. It works by storing the topology of the data, so that duplicate points and lines are only stored once. If we take, for example, the municipalities of the Netherlands, this means that we can cut down the size a lot since all the shared boundaries can be de-duplicated! If we add a bit of simplification, we can make spectacular gains: the GeoJSON of the Dutch municipalities is almost 1MB; as TopoJSON, we can reduce this to 188 KB (!) and still have an acceptable precision for most (web) maps.

D3JS can't read TopoJSON, so once the much smaller file has been transferred over the network we need to convert it back to GeoJSON to add it to the map. What we need to do:

1. Include a TopoJSON reader

```html
  <script src="https://d3js.org/topojson.v1.min.js"></script>
```

2. Pull in the geo data

```javascript
  d3.json("../data/gemeenten2017.topojson", function(error, municipalities) {
    if (error) throw error;

    let gemeenten = topojson.feature(municipalities, municipalities.objects.municipalities);
```

3. Reference different property name to label the municipalities

```javascript
  .text(function(d) { return d.properties.name});
```

In the [process](../data/Makefile) of converting GeoJSON to TopoJSON we have:

* upgraded and sanitised the property `statcode` (string) to the feature `id` (integer) and;
* renamed the property `statnaam` to the property `name`;
* dropped the property `geometry_name`.

## 3. Add multiple sources

[03_queue.html](03_queue.html)

Once we have drawn the Dutch municipalities, we'd like to show the earthquakes as well. This means we have to pull in 2 data sets. We don't want the page drawing to be delayed by these downloads. However, we can only run our scripts once both data sets are available in the web browser. This is what [d3.queue](https://github.com/d3/d3-queue) takes care of. It's already part of the minified D3.js library.

*Note, the earthquake data is stored as GeoJSON as there is little to gain from turning this into TopoJSON*

1. Pull in the geo data

```javascript
  d3.queue()
    .defer(d3.json, "../data/gemeenten2017.topojson")
    .defer(d3.json, "../data/aardbevingen_NL.geojson")
    .await(ready);

  function ready(error, municipalities,earthquakes){
    if (error) throw error;
```

2. Draw the earthquake data on the map

```javascript
  svg.selectAll(".quakes")
    .data(earthquakes.features)
    .enter()
    .append("path")
    .attr("d", path.pointRadius(3))
    .attr("class", "quakes");
```

## 4. Visualise your data

[04_visualisation.html](04_visualisation.html)

Although we have plotted the data, we cannot yet make much sense of it. Visualising the data allows us to group observations and reveal spatial patterns on the map.

1. Categorize the data: tectonic or induced earthquakes (nominal)

```javascript
  .attr("class",function(d) {
    return d.properties.TYPE == "ind" ? "quakes induced" : "quakes tectonic";
  });
```

2. Show the categories using CSS

```css
  .induced {
    fill: #EA9657;
  }
  .tectonic{
    fill: #35495D;
  }
```

3. Show the magnitude of the earthquakes (ratio)

```javascript
  .attr("d", path.pointRadius(function(d) {
    return 2.5*Math.sqrt((Math.exp(parseFloat(d.properties.MAG))));
  }))
```

*Note, the magnitude of earthquakes follows an exponential scale.*

4. Larger bubbles to the back, smaller bubbles to the fore.

```javascript
  earthquakes.features.sort(function(a, b) { return b.properties.MAG - a.properties.MAG; });
```

## 5. Add a base map for context

[06_basemap.html](06_basemap.html)

Did we just tell you D3.js is all about vector data and there are no map tiles involved? Actually, you can pull in raster tiles. They're great to give your map some geographic context! It's not part of D3.js minified library.

1. Include the tile reader

```html
  <script src="https://d3js.org/d3-tile.v0.0.min.js"></script>
```

2. Project the tiles

```javascript
  let tau = 2 * Math.PI;
  let tiles = d3.tile()
    .size([width, height])
    .scale(projection.scale() * tau)
    .translate(projection([0, 0]))
    ();
```

3. Add the tiles to the map

```javascript
  svg.selectAll("image")
    .data(tiles)
    .enter().append("image")
    .attr("xlink:href", function(d) { return "https://geodata.nationaalgeoregister.nl/tiles/service/wmts/brtachtergrondkaartgrijs/EPSG:3857/" + d[2] + "/" + d[0] + "/" + d[1] + ".png"; })
    .attr("x", function(d) { return (d[0] + tiles.translate[0]) * tiles.scale; })
    .attr("y", function(d) { return (d[1] + tiles.translate[1]) * tiles.scale; })
    .attr("width", tiles.scale)
    .attr("height", tiles.scale);
```

*Note: we're using tiles from the Dutch [PDOK](http://www.pdok.nl) service.*
