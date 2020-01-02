
# Elastic
Setting up the Elastic stack for instrumentation.

## Neat
The elasticsearch docker containers don't have curl or wget, but you can use netcat to do things.  You have to hit enter twice after you type out the GET line.
```
[root@964abfe77bd3 elasticsearch]# nc es-02 9200
GET /_cat/nodes HTTP/1.0

HTTP/1.0 200 OK
content-type: text/plain; charset=UTF-8
content-length: 141

172.26.0.2 10 99 1 0.18 0.35 0.32 dilm - es-01
172.26.0.4 13 99 2 0.18 0.35 0.32 dilm * es-03
172.26.0.3 33 99 2 0.18 0.35 0.32 dilm - es-02

^C
```
## Networks
In the first compose for elastic, I included the proxy-net, but then es-01 tried looking for es-02 on that network.  I couldn't figure out how to tell elastic which docker network to publish on.

I decided to make `elastic-net` an external network and attache traefik to it.  This let's me disable the link to elasticsearch api just by dropping that network connection.  And elastic can't get confused about which port to publish on.

## Kibana
Ok, pretty straight forward setup based on the other services and I'm getting a `Bad Gateway` response.  The kibana logs look OK, they are connecting to elasticsearch anyway.  I'm sure this is a kibana.yml thing, and setting a hostname properly or something like that.
Using firefox on pez, I can connect directly to the container's ip on 5601.  Working good there.
Ok, it was a copy paste error.  I "borrowed" the loadbalancer line from elasticsearch but didn't change 9200 to 5601.  Now it's working.

## Security
Doing this before finishing kibana...
[http://codingfundas.com/setting-up-elasticsearch-6-8-with-kibana-and-x-pack-security-enabled/index.html](http://codingfundas.com/setting-up-elasticsearch-6-8-with-kibana-and-x-pack-security-enabled/index.html)

Don't forget to `chown ave:users *.p12` on those.

And it's up!

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4ODE5NTI2NDVdfQ==
-->