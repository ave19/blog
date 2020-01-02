# Traefik

I keep typing `traefix`.  Can't stop myself!

I want to be able to have multiple backends inside docker and route traffic to each through the same open ports to the internet.

I found a link to Traefik while reading issues with the `nginx-proxy` docker github project.  People were having problems.

Traefik has good documentation, but I also found a tutorial

[https://www.digitalocean.com/community/tutorials/how-to-use-traefik-as-a-reverse-proxy-for-docker-containers-on-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-use-traefik-as-a-reverse-proxy-for-docker-containers-on-ubuntu-18-04)

## Pez

```
htpasswd -nb admin FooBear88!
admin:$apr1$sjdrL0sg$ERIs7Dz4EFXS6lBCU3x6u/
```
Following that tutorial, I got traefik up.  Not mentioned in the tutorial is you have to go to your domain and add a CNAME for `monitor.jaccard.us`.
Also, so far, the https has a default cert on it, nothing from letsencrypt yet.

At this point, though, the wordpress container doesn't seem to be working.  I'm going to nuke the site and db, remove the images, and re-up it to see if it builds everything where I expect it to go.

The tutorial created a network called `web`.  Maybe that's a thing.

Turns out, `docker-compose` takes the name of it's parent directory as the `stack`
I have moved `docker-compose.yml` to `/home/blog/candle` and changed the locations for wordpress and the database to be under `/home/blog/candle/mysql` and `/home/blog/candle/site`.  

Pointing a firefox browser that's running out of a pez xterm with ssh -X and aiming at the docker container ip:80 shows the setup page.

`candle:bmxhOTN0%kMF(NPc82`

Ok, so, `https://candle.jaccard.us/` seems to be... almost... happening...

Wordpress will have detected the docker IP for it's set up.  You need to change that to https://candle.jaccard.us/ in order for it to redirect properly.

Candle is up!

## SSL

Ok, so now I'm going to break it.  First, I'm going to move Traefik out of `traefik_setup` and into `traefik` a sibling of `./candle`.

Next, I'm looking at this guide:

[https://medium.com/@joshuaavalon/setup-traefik-step-by-step-406792afe9b2](https://medium.com/@joshuaavalon/setup-traefik-step-by-step-406792afe9b2)

Traefik isn't working quite like I expected.  There's a v1 and a v2 configuration, and depending on which blog you read, you might see options for one or the other.  I'm attempting to get a v2 version of the container up and running.  The basic authentication is working now, but there's no dashboard yet.  Still working on it.


# OK.  Breaking was a lot of breaking.

I have learned the following thing.
1.  Traefik has versions.  Version 1 configs are not compatible with version 2 configs.  People's blogs and guides do not often clarify which version they are using.  The documentation page for Traefik has a version picker, so you can make sure you're reading the right stuff, but it's at the bottom of the left hand nav menu.  If you follow a link from google, it might link into a v1.7 flavor when you need the v2.  I'm using v2.
2. Traefik has static and dynamic configurations.  Most of the dynamic configs can be applied as labels inside the docker-compose for a service.  
```
services:
  traefik:
    labels:
      - "traefik.http.routers.api.rule=PathPrefix(`/api`)"
```
And that works pretty well.
Anything you see in this part of a compose:
```
services:
  traefik:
    command:
      - "providers.docker=true"
```
Can be moved to a configuration file `traefik.yml`  like so...
```
providers:
  docker: {}
```
... if you mount that configuration into `/etc/traefik` in the container, like so:
```
services:
  traefik:
    volumes:
      - /home/blog/traefik/etc:/etc/traefik
```

## EntryPoints
If you say the entry point for a service is `web` then you have to get to it on port 80.  You can also connect to exposed ports on the host machine, or from the host machine to internal docker networks by IP and port until Traefik is running, then it sort of takes over.  

## Git
I checked in the working basic configuration so I can reset to it later.  It actually made a milestone of sorts.

## Basic HTTP Auth
Authentication is provided via middleware in Traefik.  `basicauth` is http basic username and password.  You can embed the users in the compose, but it's probably more handy to put them in an external file.  I did that and it stopped working.  The location of that file will be read by the running container, which means, the location is from *inside the container*.  Therefore, I put that auth file in the `/home/blog/traefik/etc` directory and referenced it as `/etc/traefik/http_basic_auth_users` inside the setup.  So, beware of the trap, that's the same file.

I got basic auth working on the `whoami` service by referencing the middleware from the treafik service's configuration.  The `http_basic_auth` bit is an arbitrary string.  I picked one that named the auth type being used so when you're looking at a service you have some hope.
```
services:
  traefik:
    volumes:
      - /home/blog/traefik/etc:/etc/traefik
    labels:
    - "traefik.http.routers.api.middlewares=http_basic_auth"
    - "traefik.http.middlewares.http_basic_auth.basicauth.usersfile=/etc/traefik/http_basic_auth_users"
  whoami:
    labels:
      - "traefik.http.routers.whoami.middlewares=http_basic_auth"
```


> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTI3NDcwMDY2NF19
-->