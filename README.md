# Purpose

The purpose of this app was to take a list of cities from the US Census API
and fetch the geospatial location from the Yahoo API and combine them into
a single record. That output is already exported at:

1. `mongodb\dump\locations.json`
1. `mongodb\dump\zipcodes.json`

    1. This project doesn't do anything with the zipcodes, but I figured I'd
    include this here, as it could be useful to query locations nearest a given
    zipcode
    1. The zipcodes also have geospatial data attached to them
    1. The zipcode data was sourced from: https://catalog.data.gov/dataset/zip-codes-80134/resource/e329ecef-5f29-4f69-93aa-70aff716ef23    

You can download that directly and use it for your own purposes.

If for whatever reason you want to actually run this project yourself,
feel free to follow the documentation mentioned below for instructions on
developing on this project.

## Documentation

See additional technical documentation in `/docs`.