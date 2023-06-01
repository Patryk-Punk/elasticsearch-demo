# ElasticSearch Using cURL
Hello, this is a simple Elasticsearch cluster project for playing around with that I'll keep up to date with the latest version of Elasticsearch and Kibana. I have included a json file with some account data for a simple bank account and below are a list of simple commands to get you started.

Please let me know if there's something you think would be neat you want included.

<b>CURRENT VERSION 7.10</b>

## UPLOAD ONE DOC
```
curl -X POST "localhost:9200/bank/_doc/?pretty" -H 'Content-Type: application/json' -d'
{
  "account_number": 123,
  "balance": 50000,
  "firstname": "Joe",
  "lastname": "Smoe",
  "age": 25,
  "gender": "M",
  "address": "60 State Street",
  "employer": "Pluralsight",
  "email": "joesmoe@pluralsight.com",
  "city": "Boston",
  "state": "MA"
}'
```

<br/>
<br/>

## UPLOAD BULK DOCS WITH PRETTY FORMAT
```curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_bulk?pretty&refresh" --data-binary "@accounts.json"```

### UPLOAD BULK DOCS WITHOUT PRETTY FORMAT
```curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_bulk?refresh" --data-binary "@accounts.json"```

refresh tells all shards that are affected to refresh and make data available for search.
data-binary flag used for when you're bulk uploading text

<br/>
<br/>

## MAPPINGS FOR BANK INDEX
```
curl -X PUT "localhost:9200/bank/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "account_number": {
        "type": "long"
      },
      "balance": {
        "type": "double"
      },
      "firstname": {
        "type": "keyword"
      },
      "lastname": {
        "type": "keyword"
      },
      "age": {
        "type": "integer"
      },
      "gender": {
        "type": "keyword"
      },
      "address": {
        "type": "text"
      },
      "employer": {
        "type": "keyword"
      },
      "email": {
        "type": "keyword"
      },
      "city": {
        "type": "keyword"
      },
      "state": {
        "type": "keyword"
      }
    }
  }  
}'
```
<br/>
<br/>

## Fetch Data

```curl -X GET "localhost:9200/bank/_doc/0?pretty"```

<br/>
<br/>

## Search Data

### Search by query params (NOT RECOMMENDED)
```curl -X GET "localhost:9200/bank/_search?pretty&q=Holt"```

<br/>
<br/>

### Search by body
```
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "firstname": "Holt"
    }
  }
}
'
```
You can specify whether is whether the operation done for the above command is AND or OR by the following:
```
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "address": {
        "query": "State Street",
        "operator": "AND"
      }
    }
  }
}
'
```

<br/>
<br/>

### Search by phrase prefixing
```
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_phrase_prefix": {
      "firstname": "Hol"
    }
  }
}
'
```

<br/>
<br/>

## Snapshot

You can create snapshots of your Elasticsearch instance and restore the entire isntance or just indices that you specify.

### Create a snapshot repository
First we need to create a repository for our Elasticsearch cluster. I have already handled setting up the path to the repo and setting it up in the elasticsearch.yml, so all we need to do is run this command.

```
curl -X PUT "localhost:9200/_snapshot/elastic_backup?pretty" -H 'Content-Type: application/json' -d'
{
  "type": "fs",
  "settings": {
    "location": "elastic_backup"
  }
}'
```

Now let's get that we have the snapshot repository created.

```curl -X GET "localhost:9200/_snapshot/elastic_backup?pretty"```

To delete the snapshot run this:

```curl -X DELETE "localhost:9200/_snapshot/elastic_backup?pretty"```

<br/>

### Create a snapshot
Now we can create a snapshot of our Elasticsearch isntance.

```curl -X PUT "localhost:9200/_snapshot/elastic_backup/snapshot_1?wait_for_completion=true&pretty"```

The wait_for_completion parameter specifies whether or not the request should return immediately after snapshot initialization (default) or wait for snapshot completion.

You can also specify what indices, you want to snapshot

```
curl -X PUT "localhost:9200/_snapshot/elastic_backup/snapshot_2?wait_for_completion=true&pretty" -H 'Content-Type: application/json' -d'
{
  "indices": "bank",
  "ignore_unavailable": false,
  "include_global_state": false,
  "metadata": {
    "taken_by": "patryk",
    "taken_because": "showing off just backing up bank index"
  }
}
'
```

The parameters above are chosen to be included on what is useful. 
`ignore_unavailable` defaults to `false`, but I wanted to include it because we can choose whether to error or not if an index is unavailable.
`include_global_state` defaults to `true` and when `true` includes the cluster state.

>The cluster state includes:
> - Persistent cluster settings
> - Index templates
> - Legacy index templates
> - Ingest pipelines
> - ILM lifecycle policies

`metadata` is an object that is attached to the snapshot.

<br/>

### Restore Snapshots
To restore a snapshot, you need to know the name of the the snapshot and the repo it's under.

```curl -X POST "localhost:9200/_snapshot/elastic_backup/snapshot_1/_restore?pretty"```

> Note: That you will get conflicts with restoring if any of the indices in the snapshot match with existing ones. This includes indices made by elasticsearch, so it's best to use the command below and specify the index.

Similarly, to restore only certain indices, we can specify it in the body of the request like when we create the snapshot.

```
curl -X POST "localhost:9200/_snapshot/elastic_backup/snapshot_1/_restore?pretty" -H 'Content-Type: application/json' -d'
{
  "indices": "bank",
  "ignore_unavailable": false,
  "include_global_state": false,
  "rename_pattern": "bank",
  "rename_replacement": "restored_bank",
  "include_aliases": false          
}
'
```

`rename_pattern` is find what indices need to be renamed. This will fail if multiple indices are caught and renamed to the same name.
`rename_replacement` is the string that will be used.
`include_aliases` are restored from the snapshot as well.


