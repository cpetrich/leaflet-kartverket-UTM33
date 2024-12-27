# Leaflet Map with UTM33 CRS
Here is a working example for getting [leaflet](https://leafletjs.com/) to produce a map with open WMTS tiles from [Kartverket](https://www.kartverket.no/), similar to https://norgeskart.no/ (The open WMTS tiles differ slightly in layout and shading and are distributed as paletted png files, which makes them smaller than the ones on norgeskart.no.) Norgeskart uses the default projection of Norway, [EPSG:32633](https://epsg.io/32633) (UMT33), which differs from the leaflet default CRS [EPSG:3857](https://epsg.io/3857) (google projection). The code uses [proj4leaflet](https://github.com/kartena/Proj4Leaflet) (which depends on [proj4js](https://github.com/proj4js/proj4js)) to define the CRS.

Here we go, also see ``index.html`` for a working example:
```html
    <div style="width: 100vw; height: 100vh" id="map"></div>
    <!-- The Javascript code is licensed under the MIT license (c) Chris Petrich, 2021-2024. SPDX short identifier: MIT -->
    <script type="text/javascript">
      const crsStr = 'EPSG:25833';
      const crsProj4 = '+proj=utm +zone=33 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs';
      // The origin is the <TopLeftCorner> in the response to a GetCapabilities call to
      // https://cache.kartverket.no/v1/wmts/1.0.0/WMTSCapabilities.xml
      const crsTileOrg = [-2500000, 9045984];
      // crsBaseResolution is the <ScaleDenominator> at zoom level 0 from the GetCapabilities call times 0.00028
      const crsBaseResolution = 21664;
      // according to the GetCapabilities info, zoom levels 0 through 18 are defined for EPSG:25833
      const crsMaxZoom = 18;
      const crs = new L.Proj.CRS(crsStr, crsProj4,
                        { origin: crsTileOrg, 
                          resolutions: Array.from(Array(crsMaxZoom+2),(e,zoomLevel) => crsBaseResolution/Math.pow(2,zoomLevel))
                        });
      const map = L.map('map', {
          center: [68.44, 17.4], // Narvik, Norway
          zoom: 13,
          crs: crs,
      });
      // ResourceURL for the selected ows:Tile ("topo") and TileMatrixSet ("utm33n")
      const url = 'https://cache{s}.kartverket.no/v1/wmts/1.0.0/topo/default/utm33n/{z}/{y}/{x}.png';
      const kartverketTiles = L.tileLayer(url, {
          subdomains: ['', '2', '3', '4'],
          maxZoom: crsMaxZoom,
          attribution: '<a href="https://www.kartverket.no/">Kartverket</a>',
      });
      map.addLayer(kartverketTiles);
    </script>
```

## Resources used
- overview given by Tor Arve Stangeland at https://github.com/torarve/leaflet-kartverket (no license specified)
- code snipplets for the UTM32 projection and the non-open Kartverket tile server by TomazicM at https://gis.stackexchange.com/a/307651 (CC-BY-SA 3.0 license)
- GetCapabilities file of the WMTS server https://cache.kartverket.no/v1/wmts/1.0.0/WMTSCapabilities.xml
