# Aispace coordinates #
These scripts allows to get a [TopoJSON](http://en.wikipedia.org/wiki/GeoJSON#TopoJSON) file with the coordinates of an airspace.

Say you want to get the coordinates of the Belgian Upper FIR, EBBUFIR, for AIRAC 396, then you run the following command:

```bash
$ airspace_coordinates 396 EBBUFIR
```

and you will get a file `EBBUFIR.json`.

## Description ##
The scripts depend on the javascript packages as described in `package.json`. They can be installed issuing the following command ([`npm`](https://www.npmjs.com/) is the [JavaScript](http://en.wikipedia.org/wiki/JavaScript) package manager, of course):
```bash
$ npm install
```

### Database connectivity ###
You could need DB connectivity to retrieve the coordinates of the airblocks composing the airspace. Should that be the case, remember to set the following environment variables for the user, password and DB name respectively: `DBUSR`, `DBPWD` and `DBNAME`.

### ... or local files ###
For the case above of `EBBUFIR`, you can find the relevant coordinate files for the airblocks in the `data` directory.
For example the CSV file  'data/999EB' lists the lon/lat coordinates of the airblock `999EB`, while `999EB.json` id the GeoJSON version.

The list of airblocks composing `EBBUFIR` in AIRAC 396 is in file `data/EBBUFIR.csv`.


### `airspace_coordinates` ###
This scripts produces a TopoJSON file for an airspace.

It optionally produces the CSV and GeoJSON files for the airblocks composing the airspace if they are not present in the current directory. The naming convention for the files is:

* `<airblock id>` for the CSV file with the coordinates of `<airblock id>` airblock.
  For example `035EB` contains

  ```
  lon,lat
  0043016E,512800N
  0043105E,512752N
  0043016E,512732N
  ```

  These are obtained via the execution of:

  ```bash
  $  ./airblock_coordinates 396 035EB
  ```

  which behind the scenes, executes the following SQL statement:

  ```sql
  SELECT  VTX_LON_FROM, VTX_LAT_FROM
      FROM ENV4.BOUNDARY_SEGMENT
      WHERE  TWO_D_AREA_ID = '035EB'  AND AC_ID=396
      ORDER BY SEQUENCE_NUMBER;
  ```


* `<airblock id>.json` for the GeoJSON file for the polygon of `<airblock id>` airblock


An example of its usage is:

```bash
$ airspace_coordinates 396 EBBUFIR
```


### `airblocks_for_airspace` ###
TODO

### `airblocks_coordinates` ###
TODO

### `toGeoJSONPolygon` ###
TODO


### Generation of TopoJSON ###
TODO
In order to merge all the polygons we can use `mapshaper`:

```bash
# eventually copy the coordinate files if no DB connection is available...
# $ cp data/[0-9][0-9]*EB.json .
$ mapshaper [0-9][0-9]*EB.json combine-files -merge-layers name=EBBUFIR -dissolve -o EBBUFIR.json format=topojson
```

You can visualize the result http://www.mapshaper.org/

## Data ##
The `data` directory contains all airblock files (CSV and GeoJSON) for EBBUFIR for AIRAC 396.
The file `EBBUFIR.csv` contains the list of airblock for `EBBUFIR`.

## Issues ##
This results in a polygon with a small inlet: why? Is this an artifact of quantization/simplifications/...?
Or an error in the data?