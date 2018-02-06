## Interactive Web Maps with [Leaflet.js](https://leafletjs.com)

We're going to make a map of earthquakes in the Netherlands using LeafletJS as a mapping library.

- load a background map (tiles)
- plot earthquakes over top
- add another overlay layer (municipalities)
- do this in several projections: EPSG:3857 and EPSG:28992

### How to run

Clone this repository from Github: `git clone https://github.com/NieneB/geo_visualisatie_workshop`

Some of these examples can be viewed by simply opening the file in your web browser. But for some a proper web server is required. The two simplest options are:

1. if you have Python installed, you can run `python -m SimpleHTTPServer` from the root of the repository. Then browse to the address it provides and add the name of the file you want to see, e.g. `http://localhost:8000/leaflet-js/01_basemap.html`
2. if you have NodeJS installed, you can run the `live-server` module which also has live-reload capabilities: it will refresh your browser when you make changes. Install it with `npm install -g live-server` and run it with `live-server` in the project root. Then you can open `http://localhost:8080/leaflet-js/01_basemap.html` in your browser, for example.


The geojson and topojson files  used are available in `../data`. See the end of this file for the method used to create them.

### Create a base map
The first thing we will practice is to create a basic map with Leaflet that can be zoomed and panned. It will use _tiles_, server-generated square bitmap images that get sent over the network and reassembled by Leaflet into a seamless map image. We will do this in two projections: Web Mercator and Amersfoort/RD.

In [01_basemap.html](01_basemap.html) is the most basic Leaflet example, showing the addition of a tiled map layer in the Web Mercator projection (Leaflet's only natively supported projection). We just need to initialize a map and add the tile layer. The steps are:
    
      
1. initialize the map with our coordinate reference system and then set the initial view (lat/lon and zoom level)

    let map = L.map('map-container');
    map.setView([52.268, 4.998], 3)  ;   
      
2. create a tilelayer from a URL


    let bglayer_Positron = L.tileLayer('https://cartodb-basemaps-{s}.global.ssl.fastly.net/light_all/{z}/{x}/{y}.png', {
      attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a> &copy; <a href="http://cartodb.com/attributions">CartoDB</a>',
      subdomains: 'abcd',
      maxZoom: 19
    });
      
3. add it to the map


    bglayer_Positron.addTo(map);

If we want to add a basemap in a different projection, we need to let Leaflet know how to handle the different coordinate space. Luckily there is a library for that. In [02_basemap_RD.html](basemap_RD.html) we see how to do this. We define a new coordinate system for Leaflet to know where to put the map tiles and features, and initialize the map with this. The steps become:

1. define our custom coordinate reference system/tilegrid



    var RD = new L.Proj.CRS( 'EPSG:28992','+proj=sterea +lat_0=52.15616055555555 +lon_0=5.38763888888889 +k=0.9999079 +x_0=155000 +y_0=463000 +ellps=bessel +units=m +towgs84=565.2369,50.0087,465.658,-0.406857330322398,0.350732676542563,-1.8703473836068,4.0812 +no_defs', {
      resolutions: [3440.640, 1720.320, 860.160, 430.080, 215.040, 107.520, 53.760, 26.880, 13.440, 6.720, 3.360, 1.680, 0.840, 0.420, 0.210],
      bounds: L.bounds([-285401.92, 22598.08], [595401.9199999999, 903401.9199999999]),
      origin: [-285401.92, 22598.08]
    }
    );
    
2. initialize the map using this CRS


    let map = L.map('map-container', {
      crs: RD
    });

3. create a tile layer (pointing to a tileset in RD coordinates)


    let bglayer_BRTAchtergrondkaart = new L.TileLayer('http://geodata.nationaalgeoregister.nl/tiles/service/tms/1.0.0/brtachtergrondkaartgrijs/EPSG:28992/{z}/{x}/{y}.png', {
      minZoom: 0,
      maxZoom: 13,
      tms: true,
      attribution: 'Map data: <a href="http://www.kadaster.nl">Kadaster</a>'
    });
    
4. add it to the map.


    bglayer_BRTAchtergrondkaart.addTo(map) 

### Add another overlay from a WMS
The tiled layer sources we added above are an efficient way to serve raster maps. Another way is to have the client send the bounding box it wants to see, and have that image be generated on the fly server-side, and sent back. This is called a WMS (Web Map Service). Every time the map moves, a new image needs to be generated and sent. It's a tradeoff between rendering speed, server storage size, and other performance factors.

The Dutch government offers many datasets as WMS services via https://nationaalgeoregister.nl. Most of these are in the RD projection. Now we have a Leaflet map in RD, we can easily add these layers.

1. we need a Leaflet plugin to handle WMS sources:


    <script src="https://cdn.rawgit.com/heigeo/leaflet.wms/gh-pages/dist/leaflet.wms.js"></script>

2. Then to add a new layer we do the following:


      var cbs_cars = L.WMS.overlay('http://geodata.nationaalgeoregister.nl/wijkenbuurten2014/wms', {
          'layers': 'cbs_buurten_2014',
          'styles': 'wijkenbuurten_thema_buurten_gemeentewijkbuurt_gemiddeld_aantal_autos_per_huishouden',
          'srs': 'EPSG:28992',
          'format': 'image/png'
      }).addTo(map);

See the result in [03_wms_RD.html](03_wms_RD.html)

### Add earthquake locations as GeoJSON
Map tiles are 'static': they are pre-generated and stored on the server. Their advantage is that they are fast to serve, but they are not so suitable for quickly changing data or if we want to add a lot of interaction to our data. For this we can use vector data. The most prevalent format on the Web is GeoJSON. Let's add a dataset in GeoJSON to our map.

For this example, you will need to run a local web server if you are not already doing so: see above.

In [04_geojson.html](04_geojson.html) we have the same background map, but now we fetch a GeoJSON file and load it on top. 
      
1. define how our point data will be visualized


    var geojsonMarkerOptions = {
      radius: 3,
      fillColor: "#ff7800",
      color: "#000",
      weight: 1,
      opacity: 1,
      fillOpacity: 0.8
    };

2. create an empty geojson layer. We will add data to it later.


    var geojson = L.geoJson(null,{
      pointToLayer: function (feature, latlng) {
        return L.circleMarker(latlng, geojsonMarkerOptions);
      }
    }).addTo(map);

3. fetch the data and add it to the map.


    //define a function to get and parse geojson from URL
    async function getGeoData(url) {
      let response = await fetch(url);
      let data = await response.json();
      return data;
    }
    
    //fetch the geojson and add it to our geojson layer
    getGeoData('/data/aardbevingen_NL.geojson').then(data => geojson.addData(data));

We can also show this dataset on our map in RD. GeoJSON is _always_ in latitude/longitude, never in a projected coordinate system. Leaflet reprojects features from latitude/longitude to the coordinate reference system of the map. Since we have told Leaflet to use RD, we don't need to make any other changes to our code as you can see in [05_geojson_RD.html](05_geojson_RD.html)

### Add municipalities of the Netherlands as TopoJSON
GeoJSON can be used for points, lines and polygons. So we can add any other dataset if we want. But GeoJSON is quite 'verbose', so file sizes increase quickly. Especially on the web, this causes performance problems. Luckily, someone (Mike Bostock, the creator of D3.js) developed an extension to GeoJSON which can reduce filesize greatly. It works by storing the topology of the data, so that duplicate points and lines are only stored once. If we take, for example, the municipalities of the Netherlands, this means that we can cut down the size a lot since all the shared boundaries can be de-duplicated! If we add a bit of simplification, we can make spectacular gains: the GeoJSON of the Dutch municipalities is 38MB; as topojson it's initially 25MB, and when we simplify it's 488 KB (!) and still of an acceptable precision for most (web) maps.

Leaflet can't read TopoJSON, so once the much smaller file has been transferred over the network we need to convert it  back to GeoJSON to add it to the map. What we need to do:
     
     
1. extend Leaflet to be able to create a GeoJSON layer from a TopoJSON data object

 
    //copied from https://gist.github.com/brendanvinson/0e3c3c86d96863f1c33f55454705bca7. Also available as a Leaflet plugin.
    L.TopoJSON = L.GeoJSON.extend({
      addData: function (data) {
        var geojson, key;
        if (data.type === "Topology") {
          for (key in data.objects) {
            if (data.objects.hasOwnProperty(key)) {
              geojson = topojson.feature(data, data.objects[key]);
              L.GeoJSON.prototype.addData.call(this, geojson);
            }
          }

          return this;
        }

        L.GeoJSON.prototype.addData.call(this, data);

        return this;
      }
    });

    L.topoJson = function (data, options) {
      return new L.TopoJSON(data, options);
    };
      
2. create an empty geojson layer, using the new TopoJson function instead of GeoJSON


      var geojson = L.topoJson(null, {
        style: function(feature){ //define how to style the features
          return {
            color: "#000",
            opacity: 1,
            weight: 1,
            fillColor: '#ff7800',
            fillOpacity: 0.8
          }
        }
      }).addTo(map);

3. fetch the geojson and add it to our geojson layer


    getGeoData('/data/gemeenten_2017.topojson').then(data => geojson.addData(data));

see the result in [06_topojson.html](06_topojson.html)

### Adding interactivity and more
Once you have data on your map, you can add all kinds of interactivity on events like clicks and mouseovers. You can also control the zoom and location of the map. For information, see the excellent [tutorials](http://leafletjs.com/examples.html) on Leaflet's official website.



### Creating the GeoJSON and TopoJSON files
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


