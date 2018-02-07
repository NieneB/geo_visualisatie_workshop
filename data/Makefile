SHELL := /bin/bash

.PHONY: init download clean help

init:
	npm install --global topojson-client ndjson-cli topojson-simplify topojson-server

gemeenten2017.geojson :
	curl -Lf  --progress-bar "https://geodata.nationaalgeoregister.nl/cbsgebiedsindelingen/ows?request=GetFeature&Service=WFS&version=2.0.0&typename=cbsgebiedsindelingen:cbs_gemeente_2017_gegeneraliseerd&outputFormat=application/json&srsName=EPSG:4326&propertyName=statcode,statnaam,geom" -o $@

download: gemeenten2017.geojson

gemeenten2017.ndjson : gemeenten2017.geojson
	ndjson-cat $< | ndjson-split 'd.features' \
	| ndjson-map 'd.id = +d.properties["statcode"].substring(2,6), d.properties["name"] = d.properties["statnaam"], delete d.properties["statcode"], delete d.properties["statnaam"], delete d.geometry_name, d' > $@

gemeenten2017.topojson : gemeenten2017.ndjson
	geo2topo --quantization 1e5 --newline-delimited municipalities=$< \
	| toposimplify -s 0.000000001 > $@

process : gemeenten2017.topojson

clean : ## Remove all artefacts from file system
	rm *.geojson *.json *.ndjson *.topojson

help : ## Show this help dialog
	@IFS=$$'\n' ; \
	help_lines=(`fgrep -h "##" $(MAKEFILE_LIST) | fgrep -v fgrep | sed -e 's/\\$$//' | sed -e 's/##/:/'`); \
	printf "usage: make [target]\n\n"; \
	for help_line in $${help_lines[@]}; do \
		IFS=$$':' ; \
		help_split=($$help_line) ; \
		help_command=`echo $${help_split[0]} | sed -e 's/^ *//' -e 's/ *$$//'` ; \
		help_info=`echo $${help_split[2]} | sed -e 's/^ *//' -e 's/ *$$//'` ; \
		printf '\033[36m'; \
		printf "%-30s %s" $$help_command ; \
		printf '\033[0m'; \
		printf "%s\n" $$help_info; \
	done
