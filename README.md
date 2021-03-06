# logstash-filter-geojson
[GeoJSON](http://geojson.org/) provides an elegant solution for encoding domain agnostic geographic data. 
However, when inserting documents into ElasticSearch, the GeoJSON format is not the best fit. There are several problems
* The GeoJSON format nests domain specific attributes under the `properties` element. ElasticSearch/kibana work best with flatter data structures.
* The ElasticSearch [geo_point](https://www.elastic.co/guide/en/elasticsearch/reference/1.4/mapping-geo-point-type.html) is not compatiable with GeoJSON's point type.
* The kibana [Tile Map](https://www.elastic.co/guide/en/kibana/current/tilemap.html) visualization utilizies the [Geohash](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-geohashgrid-aggregation.html) aggregration. The Geohash aggregration only supports geo_point fields.

This Logstash filter transforms GeoJSON into a structure more suitable for the ElasticStack by
* Flattening the GeoJSON data structure by unnesting all keys in the `properties` element.
* Storing point geometries in a format that is supported by ElasticSearch's geo_point type.
* Converting all non-point GeoJSON geometries into a center point to support Geohash aggregations.

## Example
Use this example logstash configuration...
```
filter {
  geojson {  
  }
}
```

...to convert GeoJSON documents like this..

```
{
  "type": "Feature",
  "geometry": {
    "type": "Point",
    "coordinates": [125.6, 10.1]
  },
  "properties": {
    "name": "Dinagat Islands"
  }
}
```

...into documents like this...

```
{
  "centroid": {
    "lat" : 10.1,
    "lon" : 125.6
  },
  "name": "Dinagat Islands"
}
```

## Configuration Options
| Configuration | Type | Default | Description |
| ------------- |:----:|:-------:|:-----------:|
| geometry_centroid_key | string | "centroid" | Specify event key containing geometry centroid  |
| properties_dig_level | integer | 1 | Specify number of levels to flatten `properties`. Specify 0 to not flatten properties, -1 to flatten all levels |
| properties_ignore_list | array of strings | [] | List of `properties` keys to ignore. Key will be ignored at all nested levels |

## Build instructions

### Set up env
* Set up [development envirnoment](https://github.com/EagerELK/logstash-development-environment)
* clone code base `git clone git@github.com:nreese/logstash-filter-geojson.git`
* `cd logstash-filter-geojson`
* `bundle install`
 
### Run tests
* `bundle exec rspec`

### Build plugin
* `gem build logstash-filter-geojson.gemspec`

### Install plugin
* `$LOGSTASH_HOME/bin/plugin install logstash-filter-geojson-0.1.0.gem`
