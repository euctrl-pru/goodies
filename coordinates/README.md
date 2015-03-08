# Aispace coordinates #
These scripts produce a [TopoJSON](http://en.wikipedia.org/wiki/GeoJSON#TopoJSON) file with the coordinates of an airspace (for a certain [AIRAC](http://en.wikipedia.org/wiki/Aeronautical_Information_Publication)).

Say you want to get the coordinates of `EBBUFIR`, the [Upper](http://en.wikipedia.org/wiki/Upper_Information_Region) [Flight Information Region](http://en.wikipedia.org/wiki/Flight_information_region) of Belgium for AIRAC 396, then you run the following command:

```bash
$ airspace_coordinates 396 EBBUFIR
```

and you will get a file `EBBUFIR.json`.

You can visualize it with [`mapshaper`](http://www.mapshaper.org/).

## Description ##
The scripts depend on the javascript packages as described in `package.json`. They can be installed issuing the following command ([`npm`](https://www.npmjs.com/) is the [JavaScript](http://en.wikipedia.org/wiki/JavaScript) package manager, of course):
```bash
$ npm install
```

### Database connectivity ###
You could need DB connectivity to retrieve the coordinates of the airblocks composing the airspace. Should that be the case, remember to set the following environment variables for the user, password and DB name respectively: `DBUSR`, `DBPWD` and `DBNAME`.

### ... or local files ###
For the case above of `EBBUFIR`, you can find the relevant coordinate files for the airblocks in the `data` directory.
For example the CSV file  `data/999EB` lists the lon/lat coordinates of the airblock `999EB`, while `999EB.json` id the [GeoJSON](http://geojson.org/) version.

The list of airblocks composing `EBBUFIR` in AIRAC 396 is in file `data/EBBUFIR.csv`.


### `airspace_coordinates` ###
This scripts produces a TopoJSON file for an airspace.

It optionally produces the CSV and GeoJSON files for the airblocks composing the airspace if they are not present in the current directory. The naming convention for the files is:

* `<airblock id>`, the CSV file with the coordinates of `<airblock id>` airblock.
  For example `035EB` contains

  ```
  lon,lat
  0043016E,512800N
  0043105E,512752N
  0043016E,512732N
  ```

* `<airblock id>.json`, the GeoJSON file for the polygon of `<airblock id>` airblock


An example of its usage is:

```bash
$ airspace_coordinates 396 EBBUFIR
```


### `airblocks_for_airspace` ###
`airblocks_for_airspace` returns the list of airblocks composing an airspace.

  ```bash
  $  ./airblocks_for_airspace 396 EBBUFIR
  ```

which returns 44 ABs (some of them below):

   ```txt
   035EB
   039EB
   049EB
   ...
   812EB
   853EB
   949EB
   999EB
   ```

The underlying SQL query (from `PRUTEST` schema) is:

   ```sql
   SELECT TWO_D_AREA_ID
      FROM ENV4.AV_BLOCK
      WHERE AC_ID = 396 AND AV_AIRSPACE_ID = 'EBBUFIR'; -- AC= AIRAC CYCLE
   ```




### `airblocks_coordinates` ###
`airblocks_coordinates` return the coordinates of the points of the airblock polygon.
The winding is clockwise and the polygon is not _closed_, i.e. the first and last coordinates do not coincide.

Executing 

```bash
  $  /airblock_coordinates 396 035EB
```

we get 3 point:

```
lon,lat
0043016E,512800N
0043105E,512752N
0043016E,512732N
```

behind the scenes executes the following SQL statement:

  ```sql
  SELECT  VTX_LON_FROM, VTX_LAT_FROM
      FROM ENV4.BOUNDARY_SEGMENT
      WHERE  TWO_D_AREA_ID = '035EB'  AND AC_ID=396
      ORDER BY SEQUENCE_NUMBER;
  ```


### `toGeoJSONPolygon` ###
`toGeoJSONPolygon` generate a GeoJSON file for the airblock polygon.
(The last point is equal to the first one.)

For example for airblock `999EB` we get:
   ```bash
   $ /toGeoJSONPolygon 999EB
   {
       "type": "FeatureCollection",
       "features": [
           {
               "type": "Feature",
               "geometry": {
                   "type": "Polygon",
                   "coordinates": [
                       [
                           [
                               [
                                   5.7877777777777775,
                                   49.7925
                               ],
                               [
                                   5.833333333333333,
                                   49.725
                               ],
                               [
                                   5.833333333333333,
                                   49.67638888888889
                               ],
                               [
                                   5.733333333333333,
                                   49.59444444444445
                               ],
                               [
                                   5.75,
                                   49.541666666666664
                               ],
                               [
                                   5.575277777777777,
                                   49.527499999999996
                               ],
                               [
                                   5.4375,
                                   49.51888888888889
                               ],
                               [
                                   5.452777777777778,
                                   49.54694444444444
                               ],
                               [
                                   5.442222222222222,
                                   49.56777777777778
                               ],
                               [
                                   5.420555555555556,
                                   49.60111111111111
                               ],
                               [
                                   5.383333333333334,
                                   49.61666666666667
                               ],
                               [
                                   5.371666666666666,
                                   49.625277777777775
                               ],
                               [
                                   5.315833333333333,
                                   49.64833333333333
                               ],
                               [
                                   5.291111111111111,
                                   49.67722222222222
                               ],
                               [
                                   5.521111111111111,
                                   49.68472222222222
                               ],
                               [
                                   5.7877777777777775,
                                   49.7925
                               ]
                           ]
                       ]
                   ]
               },
               "properties": { "id": "999EB" }
           }
       ]
   ```




### Generation of TopoJSON ###
In order to merge all the airblocks we can use [`mapshaper`](http://www.mapshaper.org/):

```bash
# eventually copy the coordinate files if no DB connection is available...
# $ cp data/[0-9][0-9]*EB.json .
$ mapshaper [0-9][0-9]*EB.json combine-files -merge-layers name=EBBUFIR -dissolve -o EBBUFIR.json format=topojson
```

You can visualize the result at http://www.mapshaper.org/

## Data ##
The `data` directory contains all airblock files (CSV and GeoJSON) for EBBUFIR for AIRAC 396.

The file `EBBUFIR.csv` contains the list of airblock for `EBBUFIR`.

## Issues ##
This results in a polygon with a small inlet: why? Is this an artifact of quantization/simplifications/...?
Or an error in the data?