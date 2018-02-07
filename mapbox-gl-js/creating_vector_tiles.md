# Making my own Vector Tiles

## Run Tippecanoe

Tippecanoe can transform geosjon data to Vector tiles. Either install it or run with docker. 

https://github.com/mapbox/tippecanoe
https://hub.docker.com/r/niene/tippecanoe/

	docker run -v current/working/dir:/data --rm niene/tippecanoe --no-tile-compression --output-to-directory=/data/tiles --maximum-zoom=20 --minimum-zoom=5 --named-layer=aardbevingen:/data/2017_06_Aardbeving.geojson

Mount a local volume to your docker container by using the `-v local/working/dir:docker/directory` 

`--rm` Remove docker and volumes on finish.

`--output-to-directory=` is the directory in the container. But becuase it is mounted your results will be also in your `local/working/dir`

Now you have a directory with all your tiles! If you put this on a server (or on localhost server) you can acces them by specifying the location path to the folder and adding `x/y/z`

	localhost:8000/my/tile/directory/{x}/{y}/{z}.pbf 

