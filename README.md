# Game Download Cache Docker Container

## Introduction

Steamcache is a caching proxy server for game download content. Any game downloads made on the network will be cached to disk and reused for future downloads. This can dramatically reduce internet usage while also increasing subsequent download speeds. The only limit is the size/speed of the cache on disk and speed of the local network.

The primary use case is gaming events, such as LAN parties, which need to be able to cope with hundreds or thousands of computers receiving an unannounced patch - without spending a fortune on internet connectivity. Other uses include smaller networks, such as Internet Cafes and home networks, where the new games are regularly installed on multiple computers; or multiple independent operating systems on the same computer.

This container is designed to support any game that uses HTTP. This makes it suitable for:

 - Steam (Valve)
 - Battle.net (Blizzard)
 - Origin (EA Games)
 - League of Legends (Riot Games)
 - Frontier Launchpad
 - Uplay (Ubisoft)
 - Windows Updates

## Usage

Start by installing [docker](https://docs.docker.com/engine/installation/) and [docker-compose](https://docs.docker.com/compose/install/).

Clone this repository and the submodules.

    git clone --recursive https://github.com/kixelated/steamcache.git

Open up `docker-compose.override.yml`. You will need to change `LANCACHE_IP` from the default `10.0.0.3` to the host's IP address.

Run the following command to build all of the containers and start them:

    docker-compose up --build

You can also use the `-d` argument to run these in the background, in which case you use `docker-compose logs` to view the logs and `docker-compose down` to stop everything.

Finally, grab a computer with Steam and change the network DNS server to the IP address of the host running steamcache. Flush your DNS and try downloading some games; you should see downloads in the steamcache logs!

## Quick Explanation

Steamcache is broken into three parts:

* A DNS proxy (steamcache-dns)
* A HTTP game cache and proxy (steamcache-http)
* A HTTPS proxy (steamcache-sni)

Steam and most other game clients use HTTP servers to download files. Steamcache works by running a custom DNS server that returns normal results, except for a curated list of domains. For these downloads we return the IP address of the steamcache host instead which will be inside the network.

The steamcache host runs its own HTTP server that will handle these download requests normally intended for the game content servers. The file is served from disk if it's stored in the cache. Otherwise, the file is downloaded and stored in the cache for future requests.

The final component is a optional HTTPS proxy that fixes some edge cases. Unfortunately, it's only possible to cache HTTP requests and some services mix HTTP and HTTPS on the same domain. The SNI proxy forwards any HTTPS requests that were incorrectly sent to steamcache via DNS.

## Full Setup

You will want a dedicated steamcache host for a full setup. Keep in mind that steamcache will serve as the primary DNS server, so any outages will cause issues connecting to the internet.

Change the network's DHCP DNS server to the steamcache host's IP and any computers on the network will automatically use steamcache. Just make sure that you define an external DNS server (such as Google's 8.8.8.8) for steamcache otherwise it will try to resolve in an infinite loop.

Steamcache can run on commodity hardware and has fairly low resource usage. You will want a large hard drive to cache downloads otherwise the least recently used files will be deleted. Your download speed will be limited to the speed of the disk so throw in a SSD or RAID if you want better performance. Otherwise the speed of the network will be the limit which is usually 1Gb/s these days.

## Configuration

Docker Compose is not required but makes it very easy to run all of the services on the same host. Check out the [documentation](https://docs.docker.com/compose/compose-file) and add any overrides to `docker-compose.override.yml` (or just `docker-compose.yml`).

Steamcache uses Google DNS (8.8.8.8 and 8.8.4.4) by default but you can specify your own or remove the override altogether. Just make sure that steamcache doesn't try to use itself as the DNS server. Repeat this configuration for all services (http,dns,sni):

    services:
      http:
        dns:
          - 8.8.8.8
          - 8.8.4.4


The cache is stored as a Docker data volume in the docker installation directory. If you want to use another drive, mount it, delete the existing volume with `docker-compose down -v`, and add these driver options:

    volumes:
      data:
        driver_opts:
          type: none
          device: /path/is/writable/and/exists/
          o: bind


Docker doesn't rotate logs by default, so you might want to rotate them automatically or risk running out of disk space. Repeat this configuration for all services (http,dns,sni):

    services:
      http:
        logging:
          driver: "json-file"
          options:
            max-size: "10m"
            max-file: "3"


For some bonus network performance, you can have docker use the host network instead directly. It's not the recommened mode so experiment at your own risk. Repeat this configuration for all services (http,dns,sni):

    services:
      http:
        network_mode: "host"


## License

The MIT License (MIT)

Copyright (c) 2016 Jessica Smith, Robin Lewis, Brian Wojtczak, Jason Rivers

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
