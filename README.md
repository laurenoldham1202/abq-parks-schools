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
$ ogr2ogr -f "GeoJSON" ../new-json-name.json shapefile-name.shp

// school locations example
$ ogr2ogr -f "GeoJSON" ../abq-schools.json abq-schools-prj.shp
```

Before performing anymore geoprocessing, we want to simplify the `PathType` column in the bike trails json. Our goal is to create data-driven styles based on this field, so the original highly-detailed category descriptions are too cumbersome. We can use the QGIS Field Calculator (side note: tried to do this with ogr, no luck...would like to try again!) to create a new simplified `path_type` column, comprised of the following `PathType`s:


1. Proposed
    1. (Proposed) Bike Blvd.
    2. (Proposed) Bike Lane
    3. (Proposed) Bike Route
    4. (Proposed) Trail Paved
    5. (Proposed) Trail Unpaved
2. Shared Roadway (Bikes and Automobiles)
    1. BikeBlvd - A shared roadway optimized by bicycle traffic.
    2. BikeLane - A portion of the street with a designated lane for bicycles.
    3. BikeRoute - Cars and bicycles share the street.
    4. Buffered Lane - Conventional bike lanes paired with a designated buffer space.
3. Bikes and Pedestrians Only
    1. Crossing- Bicycle or pedestrian under/over crossings.
    2. Hiking trail - An unpaved trail open to foot traffic only.
    3. Paved Multiple Use Trail - A paved trail closed to automotive traffic.
    4. Unpaved Multiple Use Trail  An unpaved trail closed to automotive traffic.
4. Other
    1. NMDOT - A Bicycle facility Owned and Maintained by NMDOT with different design standards than CABQ.
    
With the new simplified `path_type` column, we also want to filter our data for locations in City of Albuquerque and Bernalillo County. We accomplish this by the ogr command:

    
```
ogr2ogr -f "GeoJSON" -where "PhysicalJu='City of Albuquerque' OR PhysicalJu='Bernalillo County'" bike-trails-filtered.json bike-trails.json
```

We've created a new `bike-trails-filtered.json` file from the original `bike-trails.json`. We can now check the size of our files by navigating to the `/data` directory and issuing the command `$ ls -l`. We see that our filtered bike trails data is quite large:

```
total 67240
drwx------@ 15 Lauren  staff       480 Oct 16 08:44 CountyBoundary
-rw-r--r--   1 Lauren  staff     77241 Oct 16 08:57 abq-schools.json
drwx------@ 13 Lauren  staff       416 Oct 16 08:47 apsschools
-rw-r--r--   1 Lauren  staff     68117 Oct 16 08:55 bernalillo-county.json
-rw-r--r--   1 Lauren  staff  11864866 Oct 20 17:50 bike-trails-filtered.json
-rw-r--r--   1 Lauren  staff  13898781 Oct 20 07:59 bike-trails.json
drwx------@ 12 Lauren  staff       384 Oct 16 08:48 biketrails
drwxr-xr-x   8 Lauren  staff       256 Oct 16 09:06 city-limits
-rw-r--r--   1 Lauren  staff    868616 Oct 16 09:07 city-limits.json
```

To rectify this, we import the json into [MapShaper](https://mapshaper.org/) to simplify it to 1%. We save fix any line breaks and export as a GeoJSON named `bike-trails-simp.json`. We check our file sizes again with `$ ls -l` and see that the file size has effectively been cut in half:

```
-rw-r--r--   1 Lauren  staff  11864866 Oct 20 17:50 bike-trails-filtered.json
-rw-r--r--@  1 Lauren  staff   5925293 Oct 21 10:40 bike-trails-simp.json
```

We're now ready to start importing our data into an interactive Leaflet map!

## Interactive Map

This webmap will utilize Leaflet libraries to allow for cartographic visualization and interactivity within the browser.

Other libraries used in this map
 - jquery to request json data from the remote repository

### Establish a figure/ground perspective and a visual hierarchy.

First its important analyze the data and consider what this information is intended to represent.

1. County Shape (a polygon encompassing nearly all the other data)
2. City Limit Shape (a polygon mostly within the county data above)
3. Trail Lines (different types of lines constrained within city limits)
    1. Proposed
    2. Shared
    3. Bikes and Pedestrians Only
    4. Other
4. School Points (points scattered within City Limits)

County and City Limits function in the background atop a dark baselayer, with trails and schools in the foreground.  The background layers should be a few shades lighter than the baselayer.  Foreground layers will have more contrast than these layers.  

Trails is the most interesting layer, as it has subtypes which allow for graphical variation,and will define most viewer's knowledge of the geography of Albuquerque, due to concentrations and distribution of trail data. The majority of which is along or parallel to existing transportation corridors.  

Schools sits atop all other layers and stands out, scattered across the map page, as it are the only point data.

The map is bounded to keep the viewer from slipping too far away from the map data, while zoom limits are also in place to prevent a user from zooming too far away from the data, while also conveying the extent of the dataset.  

#### Layering & Styling
To achieve the desired figure/ground and related hierarchy, the following layers and colors were established. 

0. Baselayer (achieved in CSS as parameter for `map`  `background: rgba(0, 0, 0, 0.75`);
1. County (#263238 - very dark grayish blue)
2. City Limits (#616161 - very dark gray )
3. Trails
    1. Proposed (orange with half opacity)
    3. Shared (#FDD835 - yellow)
    4. Bikes and Pedestrians Only (#e0e0e0 - very light gray)
    5. Other (#7cb342 - moderate green)
    <!-- 6. Hiking Only ( ) -->
<!-- would be interesting here to consider subtypes for proposed (shared or bike/ped only which could be half opacity of the existing trail color) -->
 -->
4. Schools (##0097a7 - dark cyan)

<!-- proposed colors -->
<!-- 
#ffa000 - pure orange
#FDD835 - yellow
#e0e0e0 - very light gray
#616161 - very dark gray 
#263238 - very dark grayish blue
#0097a7 - dark cyan
#7cb342 - moderate green
-->

### Interactivity
#### map constraints
Constraining the browser to the area of interest is achieved via limiting Zoom levels and bounding the map to a geographic area.  

Initial zoom was set to fill the frame with data.  

```javascript
    const options = {
        center: [35.086905, -106.645317],
        zoom: 12, 
        zoomLevel: 0,
        minZoom: 11,
        maxZoom: 16,
        maxBounds: [[35.269168, -106.383361], [34.91915, -106.84547]]
    };
```

#### Tooltip

### Rollover response on Trails Layer

Upon rollover a tooltip will follow the mouse cursor showing showing concatenated string with most relevant data (name, type, length, etc.).

 
