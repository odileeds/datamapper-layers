# Sharing geographic data layers

Defining formats for sharing GeoJSON data layers with applications such as the ODI Leeds [Data Mapper](https://mapper.odileeds.org/).

## Introduction

At [ODI Leeds](https://odileeds.org/) (Open Data Institute Node in Leeds) we work on ways to open and share data sets from local businesses, local authorities, combined authorities, and government agencies. These datasets often have a geographic element to them and one of the first ways to investigate the data is to create a map. We created the [Data Mapper](https://mapper.odileeds.org/) tool to automatically scrape geographic datasets (published as GeoJSON) from [Calderdale Data Works](https://dataworks.calderdale.gov.uk/), [Data Mill North](https://datamillnorth.org/), [Northern Data Hub](https://datahub.bradford.gov.uk/), and [York Open Data](https://www.yorkopendata.org/) then make them available to be layered on a slippy map.

The primary aim of the Data Mapper project was to make it possible to show multiple geographic data layers, from multiple sources, on the same map. This work grew out of the Urban Commons/Common Ground project (with the [University of Leeds](https://lssi.leeds.ac.uk), [Leeds Love It Share It](http://baumanlyons.co.uk/news/updateleeds-love-it-share-it), and [Leeds City Lab](https://leedscitylab.wordpress.com/)) and the ethos that emerged from those project was to avoid creating or adding new gatekeepers of data. We wanted the data to be as open as possible but we also wanted to make it easy for others to build their own tools by re-using some of the building blocks. So the Data Mapper is really a two-part project: one side is the display of geographic data on a slippy map, the other is a way of sharing data layers in a decentralised way.

## Sharing layers

### A layer

We start from the concept of a data layer. This is a distinct theme of geographic data e.g. [post office locations in Leeds](https://mapper.odileeds.org/?13/53.79660/-1.53385/osm-leeds-postoffice), [polling districts in Calderdale](https://mapper.odileeds.org/?11/53.72069/-1.95007/dataworks-14937500-83dd-43ea-8b24-89766099ea7d), the [locations of York-Council-owned trees](https://mapper.odileeds.org/?11/53.96163/-1.06430/york-9065d71f-ce65-419e-a454-cdeb286a5612), or [food hygiene ratings in Bradford](https://mapper.odileeds.org/?11/53.83916/-1.84776/bradford-food-premises). We have defined a JSON format for sharing the metadata about an individual layer. JSON is a good choice because it can be read easily by many programming languages and in a web browser with Javascript. That makes it flexible.

We wanted the actual data behind a layer to be stored in a GeoJSON file hosted on the web. This means that the end-user should be able to get the most up-to-date version of the data, reduces the number of network hops the data has to go through to get to the user, and lets the publisher disable/remove a layer instantly if they need to.

### Many layers

Once you have lots of layers you quickly realise that you need an index of them. We could have done this with a database but opted to store the index in a *JSON* file so that it would be very easy to share with others. This doesn't stop people from using a *JSON* index file to populate their own database for their own purposes. As we were working with several different data stores we decided to have the concept of multiple index files, in multiple locations, that could be independently curated (by publishers) and brought together. This has a couple of benefits:
  - Publishers could keep the metadata and data up-to-date without needing to ask permission from a central service;
  - Those creating tools that would use the layers could mix-and-match which publishers/platforms they wanted to get data from.
By going down this route we distribute the storage of the data, metadata, and indexes of data layers. This decentralised design mimicks some of the advantages of the web and means that others can build their own tools on top of it without needing permission.
