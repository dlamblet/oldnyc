# How to do various things for Old NYC.

(Suggest opening this file in TextMate or another editor that understands Markdown.)

## Bring up a demo
Set up the Python environment:

```bash
virtualenv env
source env/bin/activate
pip install -r requirements.txt
```

Then start the appengine app locally via:

    ./develop.py

Note: this doesn't actually run AppEngine, just a hacked up local server that emulates it.

## Push a new version to App Engine

Change `version` in viewer/app.yaml to match today's date. Then, from the `viewer` directory run:

    appcfg.py --email danvdk@gmail.com update .

Visit appengine.google.com and make the new version the default.


## Update the data

Data (lat/lon→image lists, image metadata), is hosted on GitHub pages. To
update this, run something like this sequence:

    cd ..
    git clone https://github.com/oldnyc/oldnyc.github.io.git
    cd ../oldnyc
    ./generate_static_site.py
    cd ../oldnyc.github.io
    git add .
    git commit -a -m 'Update site'
    git push

## Update the JavaScript bundle

    ./update-js-bundle.sh

## Iterate on geocoding
It's easiest to do this by iterating on the records.pickle file, which has one
entry per milstein card, rather than one entry per photo.

    ./generate-geocodes.py --coders milstein --pickle_path nyc/records.pickle --output_format records.js --geocode > /tmp/records.json

This will print out lots of information about incorrect geocodes and eventually print something like:
25574 milstein
25574 (total)
This is out of 43363 total records in the Milstein collection.

By default, this only uses the local "geocache"--it doesn't fetch any geocodes
from Google Maps. If you want to do that, add --use_network:

    ./generate-geocodes.py --coders milstein,nyc-parks --pickle_path nyc/records.pickle --output_format records.js --geocode --use_network > /tmp/records.json

Geocoding is done based on the "Full Address" column of milstein.csv. You can see this by running:

    csvcut -c7,15 nyc/milstein.csv

If you want to determine per-borough geocoding coverage, run

    ./nyc/coverage-by-borough /tmp/records.json


## Regenerate geocodes for the viewer (nyc-lat-lons-ny.js)
To get new geocodes into the frontend, you need to geocode photos.pickle. Do so
with:

    ./generate-geocodes.py --coders milstein,nyc-parks --pickle_path nyc/photos.pickle --lat_lon_map lat-lon-map.txt --output_format lat-lons-ny.js --geocode > viewer/static/js/nyc-lat-lons-ny.js

The lat-lon-map.txt file can be generated via:

    ./generate-geocodes.py --coders milstein,nyc-parks --pickle_path nyc/records.pickle --output_format locations.txt --geocode > locations.txt
    ./cluster-locations.py locations.txt > lat-lon-map.txt

## Update the static site (oldnyc.github.io)

OldNYC issues XHRs to GitHub pages using XDomain. This allows a site consisting of completely static content. To update the static site, first clone it into a sibling directory to this repo:

    cd ..
    git clone https://github.com/oldnyc/oldnyc.github.io

And then run the update script:

    cd oldnyc
    ./generate_static_site.py

## Generate photos.pickle

photos.pickle is like records.pickle, but it duplicates each record across all its photos.
(There are potentially several photos on the Milstein card for each record.)

```bash
cd nyc
./crops-to-json.py crops.txt > /tmp/photos.json
./expand-pickle.py records.pickle /tmp/photos.json photos.pickle
```

## Generate crops.txt
...


## Generate records.pickle from CSV
The original source for the data is a CSV file that Matt K gave me, milstein.csv.

To convert this to a records.pickle file, run:

    cd nyc
    ./csv-to-record.py

This will read "mistein.csv" and create a "records.pickle" file in the nyc directory.
