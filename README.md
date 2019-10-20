# Schools and Bike Trails in Albuquerque, New Mexico
##### MAP 675, October 2019
##### Authored by Ritchie Katko and Lauren Oldham

## Data
Albuquerque school locations and bike path data was provided by the [City of Albuquerque GIS Data portal](https://www.cabq.gov/gis/geographic-information-systems-data).

Albuquerque city limits data was provided by the [City of Albuquerque ABQ Data portal](http://www.cabq.gov/abq-data).

Bernalillo County boundaries data provided by the [Bernalillo County Planning and Development Services](https://www.bernco.gov/planning/download-gis-data.aspx).

## Scenario
The City of Albuquerque, New Mexico is a sprawling city that wants to cut down its reliance on automobiles. Their main goal is to become a bike-friendly town, allowing students to safely and quickly bike to and from public schools within the city limits. Their first step to analysis is to plot bike trails and school locations on a map. The entirety of Bernalillo County eventually wants to follow suit.

## File Structure
The layout and original directory names are as follows:
1. abq-trails-schools
    1. index.html
    2. README.md
    3. data/
        1. city-limits
        2. biketrails
        3. CountyBoundaries
        4. apsschools // (schoolLocations.shp is the file with all school types combined)

## Geoprocessing
### Reprojecting
We must first check each shapefile's Coordinate Reference System and projection. First navigate to each shapefile directory with the command `$ cd directoryName` in terminal, then issue the following command:

```
// generic example
$ ogrinfo -so shapefileName.shp shapefileName

// actual example for schoolLocations shapefile
$ ogrinfo -so schoolLocations.shp schoolLocations
```

The `ogrinfo -so` command reveals the following projection info:
1. Bernalillo County boundary: NAD83
2. Albuquerque city limits boundary: WG84
3. School locations: NAD83
4. Bike trails: NAD83

Because the city limits shapefile is already in WGS84 web marcator, it does not need to be reprojected. The other three shapefiles, however, need to be reprojected with the following command:

```
// generic example
$ ogr2ogr new-shp-name.shp -t_srs "EPSG:4326" original-shp-name.shp

// actual example for schoolLocations shapefile
$ ogr2ogr abq-schools-prj.shp -t_srs "EPSG:4326" schoolLocation.shp
```

Just as we originally checked the projection info of each shapefile, we should verify that each polygon has been properly reprojected by issuing the `ogrinfo -so shapefileName.shp shapefileName` command again - we should see WGS84 as the projection for each file.

```
// schoolLocations example
$ ogrinfo -so abq-schools-prj.shp abq-schools-prj

// output
INFO: Open of `abq-schools-prj.shp'
      using driver `ESRI Shapefile' successful.

Layer name: abq-schools-prj
Metadata:
  DBF_DATE_LAST_UPDATE=2019-10-16
Geometry: Point
Feature Count: 188
Extent: (-106.860379, 34.958278) - (-106.341575, 35.227325)
Layer SRS WKT:
GEOGCS["WGS 84",
    DATUM["WGS_1984",
        SPHEROID["WGS 84",6378137,298.257223563,
            AUTHORITY["EPSG","7030"]],
        AUTHORITY["EPSG","6326"]],
    PRIMEM["Greenwich",0,
        AUTHORITY["EPSG","8901"]],
    UNIT["degree",0.0174532925199433,
        AUTHORITY["EPSG","9122"]],
    AUTHORITY["EPSG","4326"]]
OBJECTID: Integer64 (10.0)
NAME: String (175.0)
TYPE: String (254.0)
GRADELEVEL: String (254.0)
AUTHORIZER: String (254.0)
SCHOOLOFCH: String (50.0)
LASTUPDATE: Date (10.0)
created_us: String (254.0)
created_da: Date (10.0)
last_edite: String (254.0)
last_edi_1: Date (10.0)
```

Now that our shapefiles are in the correct projection, we can rewrite them as json files to be easily fed into web maps. In this example, we write up a level in our directory with `../` before the new json name. 

```
// generic example
$ ogr2ogr -f "GeoJSON" ../abq-schools.json abq-schools-prj.shp

// school locations example
$ ogr2ogr -f "GeoJSON" ../abq-schools.json abq-schools-prj.shp
```

After inspecting the bike trails shapefile in QGIS, we already know that we don't want county-wide trails; rather, we'd like to see what the City of Albuquerque has in place. We can filter for this data with the command:

```
ogr2ogr -f "GeoJSON" -where "PhysicalJu='City of Albuquerque'" bike-trails-filtered-2.json bike-trails-filtered.json
```

In this case, `PhysicalJu` is the column name that defines the location/governing body of each bike trail.