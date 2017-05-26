# ElasticSearch API

```bash
# node info
curl -XGET 'http://localhost:9200/_nodes?filter_path=xx.xx'

# system config
curl -XGET 'localhost:9200/_nodes/process?pretty'

# cluster stats
curl -XGET 'http://localhost:9200/_cluster/health'

# all index setting
curl -XGET 'http://localhost:9200/_settings'

# index status

curl -XGET 'http://localhost:9200/_status'

# index stats(include statics)
curl -XGET 'localhost:9200/_stats'

# analyze
curl -XGET 'localhost:9200/_analyze?analyzer=standard' -d 'this is a test'

```



