# Sharing geographic data layers

Defining formats for sharing GeoJSON data layers with applications such as the ODI Leeds [Data Mapper](https://mapper.odileeds.org/).

## Introduction

At [ODI Leeds](https://odileeds.org/) (Open Data Institute Node in Leeds) we work on ways to open and share data sets from local businesses, local authorities, combined authorities, and government agencies. These datasets often have a geographic element to them and one of the first ways to investigate the data is to create a map. We created the [Data Mapper](https://mapper.odileeds.org/) tool to automatically scrape geographic datasets (published as GeoJSON) from [Calderdale Data Works](https://dataworks.calderdale.gov.uk/), [Data Mill North](https://datamillnorth.org/), [Northern Data Hub](https://datahub.bradford.gov.uk/), and [York Open Data](https://www.yorkopendata.org/) then make them available to be layered on a slippy map.

The primary aim of the Data Mapper project was to make it possible to show multiple geographic data layers, from multiple sources, on the same map. This work grew out of the Urban Commons/Common Ground project (with the [University of Leeds](https://lssi.leeds.ac.uk), [Leeds Love It Share It](http://baumanlyons.co.uk/news/updateleeds-love-it-share-it), and [Leeds City Lab](https://leedscitylab.wordpress.com/)) and the ethos that emerged from those project was to avoid creating or adding new gatekeepers of data. We wanted the data to be as open as possible but we also wanted to make it easy for others to build their own tools by re-using some of the building blocks. So the Data Mapper is really a two-part project: one side is the display of geographic data on a slippy map, the other is a way of sharing data layers in a decentralised way.

## Philosophy of sharing layers

### A layer

We start from the concept of a data layer. This is a distinct theme of geographic data e.g. [post office locations in Leeds](https://mapper.odileeds.org/?13/53.79660/-1.53385/osm-leeds-postoffice), [polling districts in Calderdale](https://mapper.odileeds.org/?11/53.72069/-1.95007/dataworks-14937500-83dd-43ea-8b24-89766099ea7d), the [locations of York-Council-owned trees](https://mapper.odileeds.org/?11/53.96163/-1.06430/york-9065d71f-ce65-419e-a454-cdeb286a5612), or [food hygiene ratings in Bradford](https://mapper.odileeds.org/?11/53.83916/-1.84776/bradford-food-premises). We have defined a JSON format for sharing the metadata about an individual layer. JSON is a good choice because it can be read easily by many programming languages and in a web browser with Javascript. That makes it flexible.

We wanted the actual data behind a layer to be stored in a GeoJSON file hosted on the web. This means that the end-user should be able to get the most up-to-date version of the data, reduces the number of network hops the data has to go through to get to the user, and lets the publisher disable/remove a layer instantly if they need to.

### Many layers

Once you have lots of layers you quickly realise that you need an index of them. We could have done this with a database but opted to store the index in a *JSON* file so that it would be very easy to share with others. This doesn't stop people from using a *JSON* index file to populate their own database for their own purposes. As we were working with several different data stores we decided to have the concept of multiple index files, in multiple locations, that could be independently curated (by publishers) and brought together. This has a couple of benefits:
  - Publishers could keep the metadata and data up-to-date without needing to ask permission from a central service;
  - Those creating tools that would use the layers could mix-and-match which publishers/platforms they wanted to get data from.

By going down this route we distribute the storage of the data, metadata, and indexes of data layers. This decentralised design mimicks some of the advantages of the web and means that others can build their own tools on top of it without needing permission from us; we don't become a gatekeeper.

## Defining formats for sharing

### A layer

A layer is defined by some standard metadata in a *JSON* file. Below is an example:

```JSON
{
  "geojson": "layers/brownfield-bradford.geojson",
  "name": "Bradford Council brownfield land",
  "desc": "A visualisation of the <a href='https://datamillnorth.org/dataset/bradford-brownfield-register'>Bradford Metropolitan Borough Council brownfield land register</a> by <a href='https://odileeds.org/'>ODI Leeds</a>.","url":"https://datamillnorth.org/dataset/bradford-brownfield-register",
  "url": "https://www.bradford.gov.uk/planning-and-building-control/planning-policy/the-brownfield-register/",
  "owner": "bradford",
  "credit": {
    "text": "&copy; Bradford Metropolitan Borough Council",
    "src":"Bradford Council"
  },
  "license": {
    "text": "OGL v3",
    "url":"https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/",
    "derivatives": false
  },
  "size": 810294,
  "centre": [53.79620,-1.53370],
  "colour": "#CE906F",
  "popup":"function(f){ var history = ''; var hist = f.properties.PlanningHistory.split(/\\|/); for (var h = 0; h < hist.length; h++){ if(h > 0){ history += ', '; } history += '[<a href=\"'+hist[h]+'\">'+h+'</a>]'; }; return '<h3>%SiteNameAddress%</h3><p>' + (f.properties.DevelopmentDescription ? f.properties.DevelopmentDescription + '<br />' : '') + '<strong>Ownership:</strong> %OwnershipStatus%<br /><strong>Planning permission:</strong> %PlanningStatus%<br /><strong>Permission date:</strong> %PermissionDate%<br /><strong>Minimum net dwellings:</strong> %MinNetDwellings%<br /><strong>Hazardous substances:</strong> ' + (f.properties.HazardousSubstances ? f.properties.HazardousSubstances : 'none listed') + '<br /><strong>Area:</strong> %Hectares% ha' + (f.properties.DateUpdate ? '<br /><strong>Last updated:</strong> %DateUpdate%.' : '') + (f.properties.PlanningHistory ? '<br /><strong>Planning history</strong>: ' + (history) + '' : '') + '</p><p class=\"edit\"><a href=\"%SiteplanURL%\">View on the %OrganisationLabel% site plan map</a></p>';}"
}
```

where:
  - `geojson` - the URL of the GeoJSON file containing the data
  - `name` - the title of the dataset
  - `desc` - a description of the dataset (can contain HTML)
  - `owner` - a key for the owner of the dataset. At the moment the only use of this has been to add an "Edit" link to the bottom of feature popups when the owner is set to `osm`.
  - `url` - an optional URL for an explanation of the dataset
  - `credit` - either a string showing the copyright statement or an object containing:
    - `text` - the string showing the copyright statement
    - `src` - a short version of the publisher's name
  - `license` - an object containing:
    - `text` - the display text of the license e.g. `"OGL v3"`
    - `url` - a link to the license terms
    - `derivatives` - an optional boolean that says if this layer has a license that allows derivative data. Defaults to `false`.
  - `size` - the size of the GeoJSON data in bytes
  - `colour` - a hex/rgb colour to use for the layer (layers have one colour)
  - `centre` - the latitude and longitude of the data layer centre (allows basic location-sensitive searching through layers without having loaded the layers)
  - `popup` - either a string with replacement keys (that will be available within the `properties` object of a feature in the GeoJSON, or a Javascript function that builds a string. If nothing is provided, a popup will contain a table of key/value pairs from the `properties` object of the feature.
  
It is also possible to create a choropleth map e.g.:

```JSON
{
	"geojson": "https://mapper.odileeds.org/layers/imd-leeds.geojson",
	"name": "Leeds indices of multiple deprivation",
	"desc": "Indices of Multiple Deprivation (IMDs) by LSOA for 2015",
	"url": "http://opendatacommunities.org/resource?uri=http%3A%2F%2Fopendatacommunities.org%2Fdata%2Fsocietal-wellbeing%2Fimd%2Findices",
	"owner": "dclg",
	"format": { "type": "choropleth", "inverse": true, "key":"2015 decile" },
	"credit":{
		"text": "&copy; Department for Communities and Local Government",
		"src": "DCLG"
	},
	"licence": {
		"text": "OGL v3",
		"url": "http://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/"
	},
	"size": 537667,
	"colour": "#D73058"
}
```

where:
  - `format` - an object:
    - `type` - should be "choropleth"
    - `key` - the GeoJSON-feature property to use for the opacity scaling. This should be a number based property.
    - `inverse` - a boolean flag to change which way the opacity should be applied. When set to `true`, smaller numbers have less opacity. When `false`, larger values have less opacity.

### Indexes of layers

An index *JSON* file is formatted as follows:

```JSON
{
  "odileeds-bmdc-brownfield-land-register": {
    "name": "Bradford Council brownfield land",
    "metadata": "https://mapper.odileeds.org/layers/meta/bmdc-brownfield-land-register.json",
    "credit":"Bradford Council"
  },
  "odileeds-bradford-food-premises": {
    "name": "Bradford food premises public register",
    "metadata": "https://mapper.odileeds.org/layers/meta/fsa-bradford.json",
    "credit": {
      "src": "Food Standards Agency"
    }
  },
  "odileeds-brownfield-barnsley": {
    "name": "Barnsley Council brownfield land",
    "metadata": "https://mapper.odileeds.org/layers/meta/brownfield-barnsley.json",
    "credit":"Barnsley Council"
  },
  "dataworks-62fad7cc-3d21-4a0f-9402-e3e65ae15ef5": {
    "name":"Local Wildlife Sites: Local Wildlife Sites.json",
    "size": 9107636,
    "centre": [53.75524,-1.94559],
    "credit": {
      "src": "Calderdale Council"
    },
    "metadata": "https://mapper.odileeds.org/layers/meta/dataworks-62fad7cc-3d21-4a0f-9402-e3e65ae15ef5.json"
  }
}
```

Each layer is given a unique key e.g. `odileeds-brownfield-barnsley` in the example above. This should be unique across all layers (not just your own layers) so it is a good idea to use namespace-like prefixes to these keys. In Data Mapper these unique keys are used to build sharable links to specific views with specific layers.

Each layer can then be defined as an object using the same format as an individual layer above, or you can add a `metadata` key which points to a URL where the rest of the metadata can be found. Any keys/values found in the `metadata` URL will over-ride any keys you set in the index file as it is considered the definitive source.
