# weather-display

Weather display for our home.


## Get started

* `npm i`
* `npm start`
* Open http://127.0.0.1:8080/



## Rendering

```sh
LAT="60.222"
LON="24.83"
LOCATION="Espoo"
BATTERY="100"
TIMEZONE="Europe/Helsinki"

curl -vv -o weather.png \
  -H "x-api-key: $API_KEY" \
  "https://europe-west3-weather-display-367406.cloudfunctions.net/weather-display?lat=$LAT&lon=$LON&locationName=$LOCATION&batteryLevel=$BATTERY&timezone=$TIMEZONE"
```


## How it works

The project has two separate parts:

* [render](render/) - Generates a PNG containing the weather forecast at render time. Runs in Google Cloud Function (mostly for convenience).
  * Weather data is fetched from API by Finnish Meteorological Institute
  * HTML, CSS, and Headless Chrome are utilised to generate the PNG file. This part could be done with a lower-level approach, but using CSS for layouting is super convenient.
  * The view is a purposely dumb single HTML file, which has mock data to make development easy. The mock data will be replaced with real data using DOM ids. Not having a build tool removes a lot of unnecessary complexity.

* [rasp](rasp/) - Fetches the PNG from `render`, updates the ePaper display, and goes back to idle. Runs on Raspberry Pi Zero.


## Random notes

* All dates within the system are UTC, they are converted to local times on render. "Start of day" and "End of day" concepts are tricky.


```sh
jq -c -Rs '{ html: .}' src/templates/weather.html > .temp-body.json

curl -vv -o render.png \
  -X POST \
  -H "content-type: application/json" \
  -d@.temp-body.json \
  https://europe-west3-kimmo-b.cloudfunctions.net/chromium-render

rm -f .temp-body.json
```


* https://api.open-meteo.com/v1/forecast?latitude=62.22&longitude=24.8&hourly=temperature_2m,precipitation,weathercode&daily=weathercode&timezone=Europe%2FHelsinki&start_date=2022-10-31&end_date=2022-11-05


### All fields for `fmi::forecast::harmonie::surface::point::simple`

The model can return data up to 50h from now.

```json
{
  "Pressure": 1015.7,
  "GeopHeight": 26.3,
  "Temperature": 6.4,
  "DewPoint": 4.9,
  "Humidity": 92.8,
  "WindDirection": 127,
  "WindSpeedMS": 1.97,
  "WindUMS": -1.37,
  "WindVMS": 1.37,
  "PrecipitationAmount": 0.38,
  "TotalCloudCover": 100,
  "LowCloudCover": 100,
  "MediumCloudCover": 0,
  "HighCloudCover": 58.9,
  "RadiationGlobal": 4.4,
  "RadiationGlobalAccumulation": 682913.3,
  "RadiationNetSurfaceLWAccumulation": -1537350,
  "RadiationNetSurfaceSWAccumulation": 613723.9,
  "RadiationSWAccumulation": 14.2,
  "Visibility": 7441.7,
  "WindGust": 3.6,
  "time": "2022-11-02T07:00:00.000Z",
  "location": {
    "lat": 60.222,
    "lon": 24.83
  }
}
```

### All fields for `ecmwf::forecast::surface::point::simple`

The model can return data up to 10 days from now.

```json
{
  "GeopHeight": 37.6,
  "Temperature": 5.8,
  "Pressure": 1016,
  "Humidity": 95.7,
  "WindDirection": null,
  "WindSpeedMS": null,
  "WindUMS": -1.8,
  "WindVMS": -0.1,
  "MaximumWind": null,
  "WindGust": null,
  "DewPoint": null,
  "TotalCloudCover": null,
  "WeatherSymbol3": null,
  "LowCloudCover": null,
  "MediumCloudCover": null,
  "HighCloudCover": null,
  "Precipitation1h": 0,
  "PrecipitationAmount": null,
  "RadiationGlobalAccumulation": null,
  "RadiationLWAccumulation": null,
  "RadiationNetSurfaceLWAccumulation": null,
  "RadiationNetSurfaceSWAccumulation": null,
  "RadiationDiffuseAccumulation": null,
  "LandSeaMask": null,
  "time": "2022-11-02T07:00:00.000Z",
  "location": {
    "lat": 2764063,
    "lon": 8449330.5
  }
}
```

### All fields for `ecmwf::forecast::surface::point::simple`

```json
{
  "GeopHeight": 37.6,
  "Temperature": 5.8,
  "Pressure": 1016,
  "Humidity": 95.7,
  "WindDirection": null,
  "WindSpeedMS": null,
  "WindUMS": -1.8,
  "WindVMS": -0.1,
  "MaximumWind": null,
  "WindGust": null,
  "DewPoint": null,
  "TotalCloudCover": null,
  "WeatherSymbol3": null,
  "LowCloudCover": null,
  "MediumCloudCover": null,
  "HighCloudCover": null,
  "Precipitation1h": 0,
  "PrecipitationAmount": null,
  "RadiationGlobalAccumulation": null,
  "RadiationLWAccumulation": null,
  "RadiationNetSurfaceLWAccumulation": null,
  "RadiationNetSurfaceSWAccumulation": null,
  "RadiationDiffuseAccumulation": null,
  "LandSeaMask": null,
  "time": "2022-11-02T07:00:00.000Z",
  "location": {
    "lat": 2764063,
    "lon": 8449330.5
  }
}
```

## Credits

* Refresh icon: Created by andriwidodo from The Noun Project
* Severi Salminen for inspiration and assets https://github.com/sevesalm/eInk-weather-display