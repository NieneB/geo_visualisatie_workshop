## Geo Visualisaties voor het Web
#### Door: Niene Boeijen, Webmapper

Workshop voor DataViz Festival 8-2-2017

https://www.vizualism.com/festival/

# Presentation

https://nieneb.github.io/geo_visualisatie_workshop/

# Content

* Why geo?
* What is a map? 
* Geo Data Exploration
* Getting my map to the web
	- Leaflet.js
	- D3.js
	- MapboxGL.js

# Leaflet.js

see [leaflet-js/README.md](leaflet-js/README.md)

- Webmap / slippy maps
- adding WMS
- adding GeoJSON/TopoJSON
- Time animation

# D3.js

- Projections
- GeoJSON
- Time


# MapboxGL.js

- Adding GeoJSON
- Making own style
- Making/Using custom VT
- Animations
- extrusions


# Data voorbereiden
Input tsv zit in `data/`. Converteer naar geojson met:

    ogr2ogr -f geojson data/aardbevingen_NL.geojson CSV:data/2017_06_Aardbeving_NL.txt -oo X_POSSIBLE_NAMES=LON -oo Y_POSSIBLE_NAMES=LAT -oo KEEP_GEOM_COLUMNS=NO    

Converteer naar topojson met:

    npm install -g topojson
    geo2topo data/aardbevingen_NL.geojson > data/aardbevingen_NL.topojson
