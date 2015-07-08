# Neutron Docker network driver

This repository contains the Neutron Docker driver for the Docker networking
[remote driver](https://github.com/docker/libnetwork/blob/master/docs/remote.md).

Using the driver will result in the Docker containers joining one or more
existing or newly created Neutron networks. Each Neutron port type will have to
add support to this repository for plugging the ports it owns.

For starters, we are targeting the MidoNet port type.

## How to run it

This driver is designed to run as a Docker container. Its command line
arguments are:

* `--neutron=<IP address>`: This will point the driver to a neutron server to
  talk to when requesting networks, ports, etc.
* `--uplink`: Device that will be used for the overlay.
* `--enabled-type=<allowed port type>: The Port types that will be served by
  the remote driver container.

Vendor specific:

* `--midonet-zk=<IP address>,<IP address2>...`: All the Zookeeper Network
  distributed database servers to connect to separated by commas.
* `--midonet-cassandra=<IP address>,<IP address2>...`: All the Cassandra
  Network distributed database servers to connect to separated by commas.
* `--midonet-api=<IP address>`: IP of the MidoNet API server. Used for
  registration to the tunnel zone.

## How it works

Each host in the docker cluster will be running the neutron networking
container. The container will be running in privileged mode and it will be
sharing the host networking namespace. By putting the remote driver on
a container but with the networking namespace we manage to get easy access to
the interface we need for the overlay without installing dependencies in the
base OS.

When a container is launched by the Docker daemon, if the user tells it to use
a network backed by the plugin, for example:

```
docker run -itd --publish-service myapp.mynet.neutron mariadb
```

Docker will request its neutron remote driver to provide a `mynet` network and
a `myapp` endpoint. Once the networking remote driver container does it by
performing Neutron operations and binding veths according to the port type, it
will return the flow of execution to the Docker daemon. Once there, the
container will be started in the new networking namespace.

All the containers in mynet.neutron will be able to connect to each other
without restrictions.

