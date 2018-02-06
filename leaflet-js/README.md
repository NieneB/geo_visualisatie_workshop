## Leaflet.js

We're going to make a map of earthquakes in the Netherlands using LeafletJS as a mapping library.

- [ ] load a background map (tiles)
- [ ] plot earthquakes over top
- [ ] add another overlay layer (municipalities)
- [ ] do this in several projections: EPSG:3857 and EPSG:28992


### Create a base map

Loading data we need for this visualisation

The geojson and topojson files are available in `../data`. The method to create them was:

You will need the following software:

* either gdal or qgis, which provides a GUI frontend to gdal.
* topojson (npm install -g topojson)
* topojson-simplify (npm install -g topojson-simplify)

1. download Wijken en Buurten (en Gemeenten) kaart from CBS:
2. unzip
3. convert the shapefile to GeoJSON. Either use ogr2ogr:

    ogr2ogr -f geojson -t_srs EPSG:4326 -s_srs EPSG:28992 -sql "select GM_CODE, GM_NAAM, STED, AANT_INW, BEV_DICHTH from gem_2017 where WATER='NEE'" -lco COORDINATE_PRECISION=6 gemeenten_2017.geojson gem_2017.shp

This transforms geometry to lat/lon (EPSG:4326) and selects only a few of the many attributes present in the original dataset, in order to reduce the size of the geojson file somewhat.

Or use Qgis: load the shapefile, then right-click it in the layer list and select Save As. Make sure you select the right CRS to export: EPSG:4326 (WGS84)

4. convert the geojson to topojson to drastically reduce network transfer size. Also simplify the topojson a little.

    geo2topo  gemeenten_2017.geojson -o ~/wm/prj/geo_visualisatie_workshop/data/gemeenten_2017.topojson.temp
    toposimplify gemeenten_20172.topojson.temp -P 0.01 -o gemeenten_2017.topojson


