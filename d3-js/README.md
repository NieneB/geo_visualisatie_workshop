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

## 1. Use TopoJSON instead

[02_topojson.html](02_topojson.html)

## 1. Add multiple sources

[03_queue.html](03_queue.html)

## 1. Visualise your data

[04_visualisation.html](04_visualisation.html)

## 1. Add a base map for context

[06_basemap.html](06_basemap.html)
