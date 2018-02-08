## Data preprocessing

## create GeoJSON and TopoJSON of Earthquakes
The input data is a `.txt` file with point coordinates. We need to convert this to a `GeoJSON` file in order to plot it over our maps. We can do this with `Qgis` or  `ogr2ogr`


    ogr2ogr -f geojson data/aardbevingen_NL.geojson CSV:data/2017_06_Aardbeving_NL.txt -oo X_POSSIBLE_NAMES=LON -oo Y_POSSIBLE_NAMES=LAT -oo KEEP_GEOM_COLUMNS=NO    

In Qgis:

Add a Delimeted Text File into QGIS. Set point coordinates for the geometry definition. Set comma as seperator, or change all comma's to points. Set Layer CRS to WGS84, EPSG:4326.

Save as GeoJSON. COORDINATE PRECISION: 6

## Create TopoJSON of Municipalities
Install the following software with ndjson:

    npm install --global topojson-client ndjson-cli topojson-simplify topojson-server

1. download the gemeenten from https://geodata.nationaalgeoregister.nl/cbsgebiedsindelingen/ows?request=GetFeature&Service=WFS&version=2.0.0&typename=cbsgebiedsindelingen:cbs_gemeente_2017_gegeneraliseerd&outputFormat=application/json&srsName=EPSG:4326&propertyName=statcode,statnaam,geom and save it as `gemeenten2017.geojson`.

2.  create intermediate ndjson, removing and renaming some attributes.
    
	ndjson-cat gemeenten2017.geojson | ndjson-split 'd.features' \
	| ndjson-map 'd.id = +d.properties["statcode"].substring(2,6), d.properties["name"] = d.properties["statnaam"], delete d.properties["statcode"], delete d.properties["statnaam"], delete d.geometry_name, d' > gemeenten2017.ndjson

3. create topojson from the geojson, simplifying geometry.

    geo2topo --quantization 1e5 --newline-delimited municipalities=gemeenten2017.ndjson \
	| toposimplify -s 0.000000001 > gemeenten2017.topojson

