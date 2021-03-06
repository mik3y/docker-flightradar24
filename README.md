# docker-flightradar24
Docker container running flightradar24 fr24feed. Designed to work in tandem with mikenye/piaware. Builds and runs on x86_64, arm32v7 (see below).

fr24feed pulls ModeS/BEAST information from the mikenye/piaware (or another host providing ModeS/BEAST data), and sends data to flightradar24.

For more information on what fr24feed is, see here: https://www.flightradar24.com/share-your-data

## Supported tags and respective Dockerfiles
* `latest`, `1.0.24`
  * `latest-amd64`, `1.0.24-amd64` (`1.0.24` branch, `Dockerfile.amd64`)
  * `latest-arm32v7`, `1.0.24-arm32v7` (`1.0.24` branch, `Dockerfile.arm32v7`)
* `development` (`master` branch, `Dockerfile.amd64`, `amd64` architecture only, not recommended for production)

## Changelog

### v1.0.24
 * Original image

## Multi Architecture Support
Currently, this image should pull and run on the following architectures:
 * ```amd64```: Linux x86-64
 * ```arm32v7```, ```armv7l```: ARMv7 32-bit (Odroid HC1/HC2/XU4, RPi 2/3)

## Obtaining a Flightradar24 Sharing Key

First-time users should obtain a Flightradar24 sharing key.

In order to obtain a Flightradar24 sharing key, initially run the container with the following command:

`docker run --rm -it --entrypoint fr24feed mikenye/fr24feed --signup`

This will take you through the signup process. At the end of the signup process, you'll be presented with:
```
Congratulations! You are now registered and ready to share ADS-B data with Flightradar24.
+ Your sharing key (xxxxxxxxxxxx) has been configured and emailed to you for backup purposes.
+ Your radar id is X-XXXXXXX, please include it in all email communication with us.
```

Take a note of the sharing key, as you'll need it when launching the container.

## Configuring `mikenye/piaware` Container
If you're using this container with the `mikenye/piaware` container to provide ModeS/BEAST data, you'll need to ensure you've opened port 30005 into the `mikenye/piaware` container, so this container can connect to it.

The IP address or hostname of the docker host running the `mikenye/piaware` container should be passed to the `mikenye/fr24feed` container via the `BEASTHOST` environment variable shown below. The port can be changed from the default of 30005 with the optional `BEASTPORT` environment variable.

## Up-and-Running with `docker run`

```
docker run \
 -d \
 --rm \
 --name fr24feed \
 -e TZ="YOUR_TIMEZONE" \
 -e BEASTHOST=beasthost
 -p 8754:8754 \
 mikenye/fr24feed
```

## Up-and-Running with Docker Compose

```
version: '2.0'

services:
  fr24feed:
    image: mikenye/fr24feed:latest
    tty: true
    container_name: fr24feed
    restart: always
    ports:
      - 8754:8754
    environment:
      - TZ="Australia/Perth"
      - BEASTHOST=beasthost
```

## Up-and-Running with Docker Compose, including `mikenye/piaware`

```
version: '2.0'

services:

  piaware:
    image: mikenye/piaware:latest
    tty: true
    container_name: piaware
    mac_address: de:ad:be:ef:13:37
    restart: always
    devices:
      - /dev/bus/usb/001/004:/dev/bus/usb/001/004
    ports:
      - 8080:8080
      - 30005:30005
    environment:
      - TZ="Australia/Perth"
      - LAT=-32.463873
      - LONG=113.458482
    volumes:
      - /var/cache/piaware:/var/cache/piaware

  fr24feed:
    image: mikenye/fr24feed:latest
    tty: true
    container_name: fr24feed
    restart: always
    ports:
      - 8754:8754
    environment:
      - BEASTHOST=dockerhost
      - FR24KEY=xxxxxxxxxxx
```

For an explanation of the `mikenye/piaware` image's configuration, see that image's readme.


## Runtime Environment Variables

There are a series of available environment variables:

| Environment Variable | Purpose                         | Default |
| -------------------- | ------------------------------- | ------- |
| `BEASTHOST`          | Required. IP/Hostname of a Mode-S/BEAST provider (dump1090) | |
| `BEASTPORT`          | Optional. TCP port number of Mode-S/BEAST provider (dump1090) | 30005 |
| `FR24KEY`            | Required. Flightradar24 Sharing Key | |
| `TZ`                 | Your local timezone (optional)  | GMT     |


## Ports

The following ports are used by this container:

* `8754` - fr24feed web interface - optional but recommended 
* `30003` - fr24feed TCP BaseStation output listen port - optional, recommended to leave unmapped unless explicitly needed
* `30334` - fr24feed TCP Raw output listen port - optional, recommended to leave unmapped unless explicitly needed

## Logging
* The `fr24feed` process is logged to the container's stdout, and can be viewed with `docker logs [-f] container`.
* `fr24feed` log file exists at `/var/log/fr24feed.log`, with automatic log rotation.

