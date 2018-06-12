## Aerospike Server Dockerfile

This repository contains the Dockerfile for [Aerospike](http://aerospike.com).

### Dependencies

- [ubuntu:14.04](https://registry.hub.docker.com/_/ubuntu/)

### Installation

1. Install [Docker](https://www.docker.io/).

### Building

1. Install [Docker](https://www.docker.io/).

2. Build the image:

		sudo ./build -u EE_username -p EE_password -v <EE server Version>
	
	This will build the image using an `aerospike-server.deb` package in the local directory. If the package is not found, then it will require you to specify additional parameters to assist in downloading the package.
	
	This will build an image called `aerospike/aerospike-server-enterprise`.
	
	For details on using the build script and additional options, run:
	
		sudo ./build --help


### Usage

The following will run `asd` with all the exposed ports forward to the host machine.

	sudo docker run -tid --name aerospike -p 3000:3000 -p 3001:3001 -p 3002:3002 -p 3003:3003 aerospike/aerospike-server-enterprise
	
**NOTE** Although this is the simplest method to getting Aerospike up and running, but it is not the prefered method. To properly run the container, please specify an **custom configuration** with the **access-address** defined.

### Advanced Usage 


#### Custom Configuration

## Custom Configuration

There are two ways to configure Aerospike.

**Environment Variables**

You can provide environment variables to the running container via the `-e` flag. To set my default Namespace name to "aerospike-demo":

    docker run -e "NAMESPACE=aerospike-demo" aerospike/aerospike-server-enterprise ...

List of Environment Variables:

  * SERVICE_THREADS - Default: Number of vCPUs
  * TRANSACTION_QUEUES - Default: Number of vCPUs
  * TRANSACTION_THREADS_PER_QUEUE - Default: 4
  * LOGFILE - Default: /dev/null, do not log to file, log to stdout
  * SERVICE_ADDRESS - Default: any
  * SERVICE_PORT - Default: 3000
  * HB_ADDRESS - Default: any
  * HB_PORT - Default: 3002
  * FABRIC_ADDRESS - Default: any
  * FABRIC_PORT - Default: 3001
  * INFO_ADDRESS - Default: any
  * INFO_PORT - Default: 3003
  * NAMESPACE - Default: test
  * REPL_FACTOR - Default: 2
  * MEM_GB - Default: 1, the unit is always `G` (GB)
  * DEFAULT_TTL - Default: 30d
  * STORAGE_GB - Default: 4, the unit is always `G` (GB)

See the [configuration reference](https://www.aerospike.com/docs/reference/configuration/index.html) for what each controls.

This is not compatible with using custom configuration files.

**Custom Conf File**


By default, `asd` will use the configuration file in `/etc/aerospike/aerospike.conf`, which is generated by the entrypoint script. Environment variables will have no effect on your custom configuration file. To provide a custom configuration, you should first mount a directory containing the file using the `-v` option for `docker`:

	-v <DIRECTORY>:/opt/aerospike/etc

Where `<DIRECTORY>` is the path to a directory containing your custom configuration file. Next, you will want to tell `asd` to use a configuration file from `/opt/aerospike/etc`, by using the `--config-file` option for `aerospike/aerospike-server`:
 
	--config-file /opt/aerospike/etc/aerospike.conf

This will use tell `asd` to use the file in `/opt/aerospike/etc/aerospike.conf`, which is mapped to `<DIRECTORY>/aerospike.conf`.

A full example:

	docker run -tid -v <DIRECTORY>:/opt/aerospike/etc --name aerospike -p 3000:3000 -p 3001:3001 -p 3002:3002 -p 3003:3003 aerospike/aerospike-server-enterprise /usr/bin/asd --foreground --config-file /opt/aerospike/etc/aerospike.conf

#### access-address Configuration

In order for Aerospike to properly broadcast its address to the cluster or applications, the **access-address** needs to be set in the configuration file. If it is not set, then the IP address within the container will be used, which is not accessible to other nodes.

To specify **access-address** in aerospike.conf:

	network {
		service {
			address any                  # Listening IP Address
			port 3000                    # Listening Port
			access-address 192.168.1.100 # IP Address to be used by applications
																	 # and other nodes in the cluster.
		}
		...


#### Persistent Data Directory

With Docker, the files within the container are not persisted. To persist the data, you will want to mount a directory from the host to the guest's `/opt/aerospike/data` using the `-v` option:

	-v <DIRECTORY>:/opt/aerospike/data

Where `<DIRECTORY>` is the path to a directory containing your data files.

A full example:

	docker run -tid -v <DIRECTORY>:/opt/aerospike/data --name aerospike -p 3000:3000 -p 3001:3001 -p 3002:3002 -p 3003:3003 aerospike/aerospike-server-enterprise


#### Clustering

##### Mesh Clustering

Mesh networking requires setting up links between each node in the cluster. This can be achieved in two ways:

1. Define a configuration for each node in the cluster, as defined in [Network Heartbeat Configuration](http://www.aerospike.com/docs/operations/configure/network/heartbeat/#mesh-unicast-heartbeat).

2. Use `asinfo` to send the `tip` command, to make the node aware of another node, as defined in [tip command in asinfo](http://www.aerospike.com/docs/tools/asinfo/#tip).





