# xtwsd 
> Json Web Service for XTide
=======================

---
xtwsd stands for "Xtide Web Service Daemon", and provides a json based RESTful API to David Flater's famous [XTide tide prediction software.](https://flaterco.com/xtide/index.html)

Features
-------

* Retrieve tide and current predictions in json and svg format 
* Search for stations closest to a specific lat/long 
* Add new reference and subordinate stations via a Json PUT 


Requirements
------------
Compiling xtwsd requires the development environments of following open source projects to exist on your machine. This usually means
downloading and building them from source.

| Project |  Description | Source code |
|---------|----------|-------------|
| XTide   | The primary workhorse of this project, XTide's libxtide library provides the actual prediction calculations and data. | https://flaterco.com/xtide/files.html |
| libtcd  | libtcd is a library that gives access to XTide's low level data (harmonics database) | https://flaterco.com/xtide/files.html |
| nlohmann::json | Excellent json library for C++ | (
    https://github.com/nlohmann/json |
| restbed | RESTful framework for C++ | https://github.com/Corvusoft/restbed |



Building
----------
After installing and building the development builds of the above mentioned projects, you build xtwsd using CMake. See the file
building.txt for help building the various source projects.


Running
-----------
Be sure you have also downloaded a harmonics data file from the [FlaterCo web site](https://flaterco.com/xtide/files.html). Per the XTide
specifications, you must then specify the path where your harmonics file is stored in either the environment variable *HFILE_PATH*, or the file */etc/xtide.conf*.  Finally, run xtwsd with the command:  *xtwsd &amp;lt;port&amp;gt;*

Example:

```
xtwsd 8080
```


Resource paths
----------------

The following resource paths are available from xtwsd:



### GET /locations&lt;/[tide|current]&gt;&lt;?referenceOnly=[0|1]&gt;

Retrieves a list of every tide or current station in the database. Requesting "/locations" by itself returns every tide and current
station in the database. By specifying the *referenceOnly* parameter, you can limit
the returned list to reference stations only.  Because this list can be very long, you may be more interestd in the */nearest* path described below.

Example:
```
http://127.0.0.1:8080/locations/tide?referenceOnly=1
```


### GET /nearest&lt;/[tide|current]&gt;?lat=*n*&amp;lng=*n*&lt;&amp;count=*n*&gt;&lt;&amp;referenceOnly=[0|1]&gt;

Retrieves a list of the tide or current stations that are closted to the specified latitude and longitude parameters. The parameters
*lat* and *lng* should be specified in *decimal degrees* format (e.g. 28.1234). The five closest stations will be returned unless
the *count* parameter is used to specify a different number. By specifying the *referenceOnly* parameter, you can limit
the returned list to reference stations only.  

Example:
```
http://127.0.0.1:8080/nearest/tide?lat=26.2567&amp;lng=-80.08&amp;count=10&amp;referenceOnly=1
```



### GET /location/{*stationId*}&lt;?start=YYYY-MM-DD HH:MM ZZZ&gt;&lt;&amp;days=*n*&gt;&lt;&amp;local=[1|0]&gt;&lt;&amp;detailed=[1|0]&gt;

Retrieves the tide or current predictions for the specified station. If specified, *start* is the date the predictions will start. If not specified, today's date is used.  If specified, *days* is the number of days beyond *start* to predict (default if not specified is 1). *local* determines if the times returned should be in the same time zone the station is (*local=1*) or GMT (*local=0* or not specified).
If *detailed=1*, predictions will include events such as sunrise, and moonrise, provided they occur within the specified time range. *detailed=0* (or not specified) returns only high and low tide events.

Example
```
http://127.0.0.1:8080/location/NOS:8722862?start=2019-08-15 12:00 am EDT&amp;local=1&amp;detailed=1
```

### GET /graph/{*stationId*}&lt;?start=YYYY-MM-DD HH:MM ZZZ&gt;&lt;&amp;width=*n*&gt;&lt;&amp;height=*n*&gt;

Returns an SVG graph of the tide or current predictions for the specified station, starting at the specified start date. The optional *width* and *height* can be used to specify the size (in pixels) of the returned SVG image.

Example
```
http://127.0.0.1:8080/graph/NOS:8722862?start=2019-08-15 12:00 am EDT&amp;width=1200&amp;height=600
```


### GET /harmonics/{*stationId*}

Retrieves the harmonic constituents for the prediction data for the specified tide or current station. If the station is a subordinate station, the harmonic offsets will be returned instead.

Example
```
http://127.0.0.1:8080/harmonics/NOS:8722862
```


### GET /harmonics/schema

Returns a Json Schema that specifies the required and optional data required to add or update new prediction data. This schema
also covers the data returned by *GET /harmonics{*stationId*}*.

Example
```
http://127.0.0.1:8080/harmonics/schema
```


### PUT /harmonics

Allows new prediction data to be added to the database. The Json object sent with the PUT request should match the Json Schema
returned by *GET /harmonics/schema*.  If you are updating an existing record, be sure the *index* property is populated. For
adding new records, *index* should be omitted, or set to "-1".  The response to the put will be a Json object with the following
structure:

```
{
    "statusCode": nn // The HTTP status code (200 = OK, 400 = bad request, 500 = internal error, etc.)
    "index": nnnn // The index of the record that was updated or added
    "message": "Details of any failures"
}
```

Example
```
$ curl -vX PUT http://127.0.0.1:8080/harmonics -d @stationdata.json \
--header "Content-Type: application/json"
```

contents of stationdata.json:
```
{
    "comments": "",
    "country": "Bahamas",
    "harmonics": {
        "confidence": 10,
        "constituents": [
            {   
                "name": "M2",
                "amp": 1.329,
                "epoch": 10.6 
            },
            ... see tests/ directory for complete data ...
        ],
        "datum": "Mean Lower Low Water",
        "datumOffset": 1.47,
        "zoneOffset": 0
    },
    "levelUnits": "feet",
    "name": "Settlement Point Bahamas",
    "notes": "",
    "position": {
        "lat": 26.710,
        "long": -78.99666667
    },
    "referenceStation": true,
    "source": {
        "context": "NOS",
        "name": "tidesandcurrents.noaa.gov",
        "stationId": "9710441"
    },
    "timezone": ":America/New_York",
    "type": "tide"
}
```



### GET /tcd

Retrieves basic information about the harmonics data library currently in use.

Example
```
http://127.0.0.1:8080/tcd
```
