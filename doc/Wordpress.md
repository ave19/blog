
# Wordpress
I have spun up Traefik and learned enough about it to have some positive control.

Dear Lord, I have just discovered that `/etc/hosts` on pez still had the `192.168.1.0/24` network address.  That would have screwed me up.

## Finding Candle
Resolving `candle.jaccard.us` from HOST resolves to IP.
| HOST | IP | 
| --- | --- |
| pez | 192.168.0.88 |
| deltav | 73.14.121.28 |

Ok, firing the compose.

# Troubles

## Unable to establish database connection.

First launch, the candle wp container did not connect to the database.
```
WARNING: unable to establish a database connection to 'db:3306'
```
Both containers are on `192.168.64.0/24`
The command line on the wp doesn't even have ping!

The scoop here is that I did `rm -rf mysql` and nuked the old database.  The wordpress container starts trying to connect to the db as soon as it comes up, but the db takes some time to set itself up.  If you're starting from a clean directory, up the db first.

## Page not found.
Looking at my config, which obviously was copied from a guide, I have `traefik.frontend` and `traefik.backend` in the wordpress services configuration.  Those are v1 config options.

Looking at the dashboard, it doesn't look right.  I don't see the rule for `candle` in the routers.  Feels wrong.

Trying these now:
```
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.wordpress.rule=Host(`candle.jaccard.us`)"
  - "traefik.http.routers.wordpress.entrypoints=web"
```
The `enable` is default true, but I like the idea of having the reminder for setting to false to turn it off.

After a bounce, the `candle.jaccard.us` showed up in the routers section of the dashboard.  I feel better already.

## Gateway Timeout.
That's all it says.
I found the IP that wordpress is running on from the Traefik dashboard page and connected to it from firefox running directly on pez.  That brought up the installation page.  So wordpress is running and the IP is there.

Connection Results:
| URL | From | Result |
| --- | --- | --- |
| `http://192.168.96.3/wp-admin/install.php` | firefox on pez | worked |
| `192.168.0.88` | firefox on pez | did not work, it doesn't say `candle.jaccard.us` in it. |
| `candle.jaccard.us` | firefox on pez | `Gateway Timeout` |
| `candle.jaccard.us` | chrome on pez | `Gateway Timeout` |

I see this in the traefik logs:
```
'504 Gateway Timeout' caused by: dial tcp 192.168.96.3:80: i/o timeout
```
Looks like, if you don't specify which docker network a service is supposed to connect to, traefik picks randomly, and sometimes, incorrectly.  From a comment:
```
You are right it's better (and more readable) to specify `traefik.docker.network` label
```
Now I'm going to go figure out if that's a v1 or v2 option!  Looks like it's good for v2.

I created `inside` and `outside` networks.  I added a label for `inside` in the wordpress section.  The db is only on candle-internal, wordpress is on both.

Still getting gateway timeouts.  Can still connect to wp on the container's ip from pez. though.  Even with curl:
```
root@pez:/home/blog/candle# curl -H Host:candle.jaccard.us http://192.168.0.88/
Gateway Timeout
```
Ok, now I'm making the `outside` network reference the *external* traefik-default network.

```
'504 Gateway Timeout' caused by: dial tcp 192.168.208.3:80: i/o timeout
```
Lemme look at this again.  What's that IP pointing to?  That's wordpress's container.  Pez's firefox can go directly to that IP and I get the wp installation page.
I can tell by the logs in traefik that it's correctly picking where to send the request that comes in for candle.

Read this:
```
Fixed it, the containers didn't share the same external network. After adding that everything was fine
```
What?  :-p
In the traefik compose, I had:
```
networks:
  proxy-net:
```
And in candle I had:
```
networks:
  outside:
    external:
      name: traefik_default
```
Now when I look at the `traefik_proxy-net` network in portainer, I see that wordpress is actually attached, there.

That didn't do it.

I'm going to add that line to traefik, too.  Nope.

Ok, this might be a thing.  traefik is not on the candle network.  Candle's IP on the `traefik_proxy-net` is `192.168.48.4`  I need candle to listen on that IP.

Saw a bug that said sometimes the order of the networks matters.  

Um, yeah, that fixed it.  I don't like that.

I'm guessing that the `outside` in 
```
- "traefik.docker.network=outside"
```
is just passed to traefik without any kind of expansion. 
I specified the `traefik_proxy-net` and put the inside before the outside and it still works.  Unless I redo the proxy net, this should be fine.

Ok, candle's online!

## Lets Encrypt
Or... not.  Following the documentation on traefik's site for v2.  Don't have real certs yet.

### Trouble:  No such provider *letsencrypt*
Solution:  `chmod 600 acme.json`
Saw the error in the logs on traefik's container.

Logs review:  
* The configuration is getting picked up.
* Found this in a debug line:
```
time="2019-10-10T20:30:01Z" 
  level=debug 
  msg="No domain parsed in provider ACME" 
  routerName=whoami 
  rule="PathPrefix(`/whoami`)" 
  providerName=letsencrypt.acme
```
Ok, so that makes sense.  It can't issue a cert without a domain in it.
I'll change that path oriented rule to a host based one, and use `monitor.jaccard.us` for now.

And that worked within seconds!  
Adding the basic http auth worked as well.

Going to add the lines for candle.jaccard.us.
Got the default cert from traefik on `https://candle.jaccard.us:443`
I did a copy and paste and didn't change `whoami` to `wordpress`.
Now it's working fine.

## Redirect 80 into 443.
This was easy in Traefik v1, but v2 doesn't seem to have a thing for this.

There's a "redirect scheme" in the middleware section that does this.
You have to set up the 80 and 443 endpoints, then redirect 80 to 443 with the middleware.  Checked into github.

## Permission Denied!
I did some `chown -R ...` and suddenly lost the db.  
The uid:gid for those files is 999.  I set /etc/passwd and /etc/group to show that.

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTI4NTAyNzc0LC0yMDI1MDYyMjI0XX0=
-->