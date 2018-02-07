# **Geo** Visualisaties voor het Web
#### By: Niene Boeijen, Webmapper

Workshop for DataViz Festival 8-2-2017 > https://www.vizualism.com/festival/

# Presentation

The presentation can be found here: https://nieneb.github.io/geo_visualisatie_workshop/

# Workshop

### How to run

Clone this repository from Github: `git clone https://github.com/NieneB/geo_visualisatie_workshop`

Some of these examples can be viewed by simply opening the file in your web browser. But for some a proper web server is required. The two simplest options are:

1. if you have Python installed, you can run `python -m SimpleHTTPServer` from the root of the repository. Then browse to the address it provides and add the name of the file you want to see, e.g. `http://localhost:8000/leaflet-js/01_basemap.html`
2. if you have NodeJS installed, you can run the `live-server` module which also has live-reload capabilities: it will refresh your browser when you make changes. Install it with `npm install -g live-server` and run it with `live-server` in the project root. Then you can open `http://localhost:8080/leaflet-js/01_basemap.html` in your browser, for example.

### Data

The geoJSON and topoJSON files used are available in `/data`. See [data/README.md](data/README.md) for the methods used to create them. The data preparation has already be done for you.

# D3.js

See [d3-js/README.md](d3-js/README.md)

What will you do: 

Topics covered:

- Projections
- GeoJSON
- Time

# Leaflet.js

See [leaflet-js/README.md](leaflet-js/README.md)

What will you do: 

- [ ] Basic Base map
- [ ] Projections EPSG:3857 and EPSG:28992
- [ ] WMS overlay
- [ ] Adding a GeoJSON
- [ ] TopoJSON
- [ ] Interactivity

Topics covered:

- Interactive map
- Webmap / slippy maps
- adding WMS
- adding GeoJSON/TopoJSON
- Projections
- Using Dutch Geo Data


# MapboxGL.js

see [mapbox-gl-js/README.md](mapbox-gl-js/README.md)

What will you do:

- [ ] Load basic Mapbox Map
- [ ] Load Custom Vector Tiles
- [ ] Adding GeoJSON
- [ ] Fonts
- [ ] Extrusion

Topics covered:

- Vector Tiles
- PDOK data
- Adding GeoJSON
- Styling options

