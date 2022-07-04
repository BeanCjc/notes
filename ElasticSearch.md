# ElasticSearch

## 1. 简单docker安装ES和Kibana

```shell
grep vm.max_map_count /etc/sysctl.conf
sysctl -w vm.max_map_count=262144
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.2.3
docker network create elastic
docker run --name es01 --net elastic -p 9200:9200 -p 9300:9300 -it docker.elastic.co/elasticsearch/elasticsearch:8.2.3
docker cp es01:/usr/share/elasticsearch/config/certs/http_ca.crt .
curl --cacert http_ca.crt -u elastic https://localhost:9200
# password yVwQyQrkPonRPoGp4b3+
# Kibana enrollment token eyJ2ZXIiOiI4LjIuMyIsImFkciI6WyIxNzIuMTguMC4yOjkyMDAiXSwiZmdyIjoiNDE0ZThjZTQ2YzNhZDdiMWY0YzVmZGM0ODRkNTdlNDYwODNlMTA5YTk1MzY1YzU0MjQwZGM5ZmVjM2FkZjI5NyIsImtleSI6IjVPS2h5WUVCdnR5WWtuY0VaemFTOlZBNE1rRFhqVDNHYmdlWm1vODVzaEEifQ==
docker pull docker.elastic.co/kibana/kibana:8.2.3
docker run --name kibana --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.2.3
Ctrl C
ctrl C
docker start es01
docker update es01 --restart always
docker start kibana
docker update kibana --restart always
# elastic账号的密码更新为qwer123
```

