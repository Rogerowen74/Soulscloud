# Solis Cloud Scraper 

The Solis Cloud scraper does the following
* Scrapes data from your plant data at soliscloud.com on demand
* Exposes the data via service end points
* Optionally automatically refreshes the data via separate script

The Solis Cloud Scraper requires node.js

## Data scraped
The following data is made available

* Current yield
* Current battery charge
* Current draw from battery
* Current export to grid
* Current import from grid
* Todays total yield
* Todays total charging
* Todays total discharging
* Todays import from grid
* Todays export to grid
* Todays house consumption
* Station capacity

## Installation
* Download or clone the repository
* Run `npm install`
* Setup your scraper.properties file

## Scraping Data
To scrape data from the Solis Cloud website and expose via the endpoints, run the following command once all installation steps are complete
* `node scraper.js`

This will start the scraper on port 5561, perform an initial scrape and make the `/data` endpoint available.  The port can be changed from the configuration.

## Refreshing Scraped Data
To instruct the scraper to refresh the data (scrape again), call the `/refresh` endpoint

 This endpoint is secured with basic authentication, with configuration in the scraper.properties file

## Refreshing automatically
An optional refresh script is also provided.  By default it will request a data refresh every 6 minutes between 6am and 11pm.  This can be changed in the configuration.

To start the refresher run the command
* `node refresher.js`

## Retrieving the scraped data
This data is made available at the following endpoints
 * `http://[your.ip.addr]:[port]/data`
 * `http://[your.ip.addr]:[port]/v1/data`
 * `http://[your.ip.addr]:[port]/v2/data`

 * e.g `http://127.0.0.1:5561/v2/data`

**Note** `/data` and `/v1/data` are identical.

The endpoints are secured with basic authentication, with configuration in the scraper.properties file

### V1 Endpoint(s)

The V1 (or just `/data`) endpoint returns all values as strings without any form of parsing.  These are the values as they appear on the Solis website.

The response to this call will look like this:
```
{
   "totalYield":"5.4kWh",
   "currentGen":"0kW",
   "batteryCharge":"21%",
   "drawFromBattery":"0.079kW",
   "todaysCharging":"8kWh",
   "todaysDischarging":"10kWh",
   "todayFromGrid":"3.22kWh",
   "todayToGrid":"0.53kWh",
   "currentGridInOut":"0.249kW",
   "currentHouseDraw":"0.328kW",
   "totalHouseConsumption":"13.22kWh",
   "scrapeStartDurationMs":1668713753046,
   "scrapeEndTimeMs":1668713766326
}
```

### V2 Endpoint

The V2 endpoint returns values appropriately typed.  It will also return a negative value for currentBatteryUsage to indicate battery discharge and a negative value for currentGridUsage to indicate grid import.

The response to this call will look like this:
```
{
  "currentYield": 1.124,
  "currentYieldUnit": "kW",
  "currentBatteryCharge": 16,
  "currentBatteryChargeUnit": "%",
  "currentBatteryUsage": 0.021,
  "currentBatteryUsageUnit": "kW",
  "currentGridUsage": 0,
  "currentGridUsageUnit": "kW",
  "currentHouseConsumption": 1.103,
  "currentHouseConsumptionUnit": "kW",
  "todayYield": 1.3,
  "todayYieldUnit": "kWh",
  "todaysCharging": 3.4,
  "todaysChargingUnit": "kWh",
  "todaysDischarging": 3.4,
  "todaysDischargingUnit": "kWh",
  "todayGridImport": 2.2,
  "todayGridImportUnit": "kWh",
  "todayGridExport": 0.1,
  "todayGridExportUnit": "kWh",
  "stationCapacity": 5.175,
  "stationCapacityUnit": "kWp",
  "scrapeStartDurationMs": 1676370170785,
  "scrapeEndTimeMs": 1676370185058
}
```

You can use curl to check it's working properly using the command

* `curl -u username:password http://[your.ip.addr]:[port]/v2/data`
 *Where username and password match the values in scraper.properties*
 e.g
 * `curl -u rich:rich-password http://127.0.0.1:5561/v2/data`
 
## Configuration
Configuration is held in the file scraper.properties which should be in the same folder as the scraper.js file.

*Note that some properties are shared if running both index.js and refresher.js from the same location.*

The following configuration values are required by the scraper.js script
* solis.url = [url of soliscloud.com]
* solis.username = [your username for soliscloud.com]
* solis.password = [your password for soliscloud.com]
* solis.maxSelectorRetries = [maximum number of retries is page elements are not rendered]
* service.port = [the port to expose scraped data and call refresh on]
* service.username = [the username to access the /data and /refresh endpoints]
* service.password = [the password to access the /data and /refresh endpoints]

The following configuration values are required by the refresher.js script
* service.port = [the port to expose scraped data and call refresh on]
* service.url = [the URL where the scraper is running
* service.username = [the username to access the /refresh endpoints]
* service.password = [the password to access the /refresh endpoints]
* refresh.interval-mins = [the interval in minutes to call the /refresh endpoint]
* refresh.start-hour = [the hour to start refreshing]
* refresh.end-hour = [the hour to stop refreshing]

An example config looks like:

    # Details for accessing solis cloud used by scraper.js
    solis.url = https://soliscloud.com
    solis.username = richs-email@gmail.com
    solis.password = crazyPassw0rd!
    solis.maxSelectorRetries = 5
    service.port = 5561
    
    # Authentication details shared by scraper.js and refresher.js
    service.username = scrape-master
    service.password = gimme-some-data
    
    # Details to refresh the scraped data used by refresher.js
    service.refresh.url = http://scraper.domain.com:5561/refresh
    refresh.interval-mins = 6
    refresh.start-hour = 6
    refresh.end-hour = 23

## Restrictions/Limitations
* The scraper choses the first plant on the list presented at soliscloud.com
* When a notification is posted to the soliscloud website, e.g upcoming maintenance announcement, this will need to be manually cleared by logging in to the website and clicking the OK button (or similar).
