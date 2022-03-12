# Tor Relay Server on Docker

**Fork from [https://github.com/chriswayg/tor-server/issues/1](https://github.com/chriswayg/tor-server/issues/1)**

**A complete, efficient and secure Tor relay server Docker image based on Debian Bullseye for arm64v7**
*This docker image will update automatically each time the Debian Bullseye base image is updated and build & install the latest current stable version of Tor server. It will run Tor as an unprivileged regular user, as recommended by torproject.org.*

The Tor network relies on volunteers to donate bandwidth. The more people who run relays, the faster the Tor network will be. If you have at least 2 megabits/s for both upload and download, please help out Tor by configuring your Tor to be a relay too.

![Tor](https://camo.githubusercontent.com/b52d76f0851c402c220ced3198f18e53e196fc7a2ac7544474fdafaec46257c4/68747470733a2f2f6d656469612e746f7270726f6a6563742e6f72672f696d6167652f6f6666696369616c2d696d616765732f323031312d746f722d6c6f676f2d666c61742e737667 "Tor logo")

[`Tor`][1] is free software and an open network that helps you defend against
traffic analysis, a form of network surveillance that threatens personal
freedom and privacy, confidential business activities and relationships, and
state security.

- Tor prevents people from learning your location or browsing habits.
- Tor is for web browsers, instant messaging clients, and more.
- Tor is free and open source for Windows, Mac, Linux/Unix, and Android

## Quickstart

This will run a Tor relay server with defaults and a randomized Nickname:

`docker run -d --name=tor_relay_1 -p 9001:9001 --restart=always dnikms/rpi-tor-server`

You can set a Nickname (only letters and numbers) and a Contact Email using environment variables:

```bash
docker run -d --name=tor_relay_1 -p 9001:9001 \
-e TOR_NICKNAME=Tor4docker -e CONTACT_EMAIL=tor4@example.org \
--restart=always dnikms/rpi-tor-server
```

## Tor configuration

For more detailed customisation edit `./torrc` on the host to the intended settings:

```env
### /etc/torrc ###
# see /etc/torrc/torrc.default and https://www.torproject.org/docs/tor-manual.html.en

# Server's public IP Address (usually automatic)
#Address 10.10.10.10

# Port to advertise for incoming Tor connections.
# common ports are 9001, 443
ORPort 9001 IPv4Only - Or if you have IPv6Only specify or if you have both leave the port only

# Mirror directory information for others (optional)
# common ports are 9030, 80
#DirPort 9030

# Run as a relay only (not as an exit node)
ExitPolicy reject *:*         # no exits allowed

# Set limits
#RelayBandwidthRate 1024 KB   # Throttle traffic to
#RelayBandwidthBurst 2048 KB  # But allow bursts up to
#MaxMemInQueues 512 MB        # Limit Memory usage to

# Run Tor as obfuscated bridge
#ServerTransportPlugin obfs3 exec /usr/bin/obfsproxy managed
#ServerTransportListenAddr obfs3  0.0.0.0:54444
#BridgeRelay 1

# Run Tor only as a server (no local applications)
SocksPort 0

# Run Tor as a regular user (do not change this)
User debian-tor
DataDirectory /var/lib/tor

# If no Nickname or ContactInfo is set, docker-entrypoint will use
# the environment variables to add Nickname/ContactInfo here
#Nickname Tor4                 # only use letters and numbers
#ContactInfo email@example.org
```

## Run Tor with mounted `torrc`

Mount your customized `torrc` into the container. You can reuse the `secret_id_key` from a previous Tor server installation (`docker cp tor_relay:/var/lib/tor/keys/secret_id_key ./`) by mounting it, too, to continue with the same Fingerprint and ID.

```bash
docker run -d --name=tor_relay_1 -p 9001:9001 \
-v $PWD/torrc:/etc/tor/torrc \
-v $PWD/keys:/var/lib/tor/keys/ \
--restart=always dnikms/rpi-tor-server
```

Check with ```docker logs tor_relay_1```. If you see the message ```[notice] Self-testing indicates your ORPort is reachable from the outside. Excellent. Publishing server descriptor.``` at the bottom after quite a while, your server started successfully.

## Run Tor using docker-compose.yml

Adapt this example `docker-compose.yml` with your settings:

```yaml
relay:
  image: dnikms/rpi-tor-server
  restart: always
  ports:
    - "9001:9001"
    - "9030:9030"
  environment:
    ## set your Nickname here (only use letters and numbers)
    TOR_NICKNAME: Tor4
    ## an email address to contact you
    CONTACT_EMAIL: email@example.org
    ## Set throttle rate
    Relay_BR: 1MB
    ## Set brust throttle rate
    Relay_BB: 1.5MB
    ## If you want to change default port
    ORPort: 443
    ## If you want to change default dir port
    DirPort: 80
```

### Start the Tor server

Start a new instance of the Tor relay server, show the current fingerprint and display the logs:

```bash
docker-compose up -d
docker exec tor_relay_1 cat /var/lib/tor/fingerprint
docker-compose logs
```

## References

- [https://www.torproject.org/docs/tor-relay-debian.html.en](https://www.torproject.org/docs/tor-relay-debian.html.en)
- [https://www.torproject.org/projects/obfsproxy-debian-instructions.html.en](https://www.torproject.org/projects/obfsproxy-debian-instructions.html.en)

[1]: https://www.torproject.org/
