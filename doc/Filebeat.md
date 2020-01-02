
# Filebeat

Turns out it has a docker module thingy.  I'm going to run that in a container.

Following this:  [https://www.elastic.co/guide/en/beats/filebeat/current/running-on-docker.html](https://www.elastic.co/guide/en/beats/filebeat/current/running-on-docker.html)

## Setup
Had to add authentication bits to the setup command.  I also had to make sure these ip addresses were right.
```
[ave@pez bb]$ docker run \
  --network=elastic-net \
  docker.elastic.co/beats/filebeat:7.4.0 setup \
  -E setup.kibana.host=172.27.0.6:5601 \
  -E output.elasticsearch.hosts=["http://172.27.0.2:9200"] \
  -E output.elasticsearch.username="elastic" \
  -E output.elasticsearch.password="OnNm6sI1inAi9h8mpx3A"
```

## Monitoring Docker
Filebeat has a module that can watch docker for things like container starts, stops, creations, etc.  I want to get that rolling.

### filebeat.autodiscover
I turned this on.  Because, sure.  But when I ran it, the logs show it going out to google cloud.  There's no google cloud in the config.  Ok, nevermind, those logs are old.  `/var/log/filebeat` is read-only anyway.  Where are the logs for my filebeat?

They are going to `/usr/share/filebeat/logs`.

## Location
I moved docker to `/home/docker` from `/var/lib` so adjustments need to be made in the configuration.

## Protocol
Use the `http` protocol, not `https`.

And events!

Ok, those weren't useful.  I told docker to watch everything, and it did, but it didn't differentiate between containers.

It looks like we can apply labels, like Traefik, and push filebeat's docker configuration around on a per container basis.  This is nice, because the per container configs are with that container, and the base filebeat container only needs to be told to listen.


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0MjE1NTYwODddfQ==
-->