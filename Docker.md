Synthea instances can be run in parallel using [Docker](https://www.docker.com/), an open-source software containerization platform. Docker wraps synthea in a complete, lightweight filesystem that contains everything needed to run Synthea: code, system tools, libraries, etc. - everything that we install in our own production Synthea environments.

## Getting Started

* Install and run the [Docker Engine](https://docs.docker.com/engine/getstarted/step_one/). Variants exist for OS X, Linux, and Windows.
* If you're new to Docker, check out Docker's helpful [Getting Started Guide](https://docs.docker.com/engine/getstarted/).
* Pull the latest Synthea images from [The Docker Hub](https://hub.docker.com/r/synthetichealth/synthea/).

## Synthea Variants

There are several variants of Synthea available on the Docker Hub:

| Variant | When to use it |
|:--------|:---------------|
| `synthetichealth/synthea:latest` | Use this variant if you're new to Synthea, just trying things out, or only need a few patient records to play with. This is suitable for single-threaded Synthea runs. |
| `synthetichealth/synthea:jruby` | **(coming soon!)** When multithreaded, Synthea runs 30% faster by leveraging the concurrent power of the JVM. Use this variant if you need to generate _lots of records, fast_ and you plan to run Synthea multithreaded. |

To pull a Synthea image from the Docker Hub:

```
$ docker pull <synthea_variant>
```

For example:

```
$ docker pull synthetichealth/synthea:latest
```

## Configuration

**This is important:** Before running Synthea with Docker check the following settings in `synthea.yml`:

```yml
  ...
  docker:
    dockerized: true
    location: '/mnt/synthea'
```

This configures Synthea to run in a Docker environment. Specifically, this does the following:

1. Instructs Mongoid (the mongodb driver used with the `health-data-standards` gem) to connect to a mongo database running at `mongo:27017` instead of using a database at `localhost:27017`.

2. Instructs the exporter to export all records (ccda, fhir, html) to a persistent storage container with a mounted volume at `/mnt/synthea`.

## Running Synthea

Synthea runs in a multi-container environment managed using [`docker-compose`](https://docs.docker.com/compose/reference/overview/). Synthea uses the following containers:

| Container | Purpose |
|:----------|:--------|
| `synthea` | The main container that contains Synthea's source, configuration, and executes a Synthea run. |
| `mongo` | A standard mongodb container used by the CCDA exporter and `health-data-standards` gem. |
| `synthea_output` | A persistent storage volume mounted to a standard Ubuntu container. Synthea writes its output to this persistent volume at `/mnt/synthea`. The `synthea` container can connect to the database(s) in this container at `mongo:27017`. |
| `inspector` | A standard ubuntu instance that can be used to view and manipulate the output in `/mnt/synthea`. The persistent storage volume can be attached to this container using `--volumes-from`. |

### 0. Check Your Configuration in `synthea.yml`

See [Configuration](#configuration). There are important settings that must be checked before attempting to run Synthea in Docker.

### 1. Create a Persistent Storage Volume

```
$ docker create -v /mnt/synthea --name synthea_output ubuntu 
```

This creates a [data volume container](https://docs.docker.com/engine/tutorials/dockervolumes/#/creating-and-mounting-a-data-volume-container) using a standard Ubuntu image and names it "synthea_output" for easy reference. The path to the volume (`/mnt/synthea/`) is _the same_ in both the `synthea_output` container and the `synthea` container. This location should also match `docker:location` in `synthea.yml`.

### 2. Run Synthea

```
$ docker-compose -f docker/ruby/docker-compose.yml run synthea bundle exec rake synthea:sequential
```

This tells Docker to:

1. Create the `mongo` and `synthea` services specified in the `docker-compose.yml` file
2. Link them together in an isolated network
3. Then pass the command `bundle exec rake synthea:sequential` to the running `synthea` container

The first time you call `docker-compose run` Docker will pull the required images from the Docker Hub and build them locally. The setup required for this first run will take a little while to execute, so sit tight. Subsequent runs will reuse these images and will be fast.

After building, Synthea will automatically run and write all output to the persistent storage volume at `/mnt/synthea`.

### 3. Inspect the Output

Create and run the standalone `inspector` container to mount and view the `synthea_output` volume:

```
$ docker run -it --name inspector --volumes-from synthea_output ubuntu
```

This creates a regular ubuntu instance from the base `ubuntu` image and mounts the volume. The `-it` arguments run the Docker container in interactive mode (`-i`) using a pseudo-TTY terminal (`-t`).

You should then be able to view the output from the latest Synthea run:

```
$ cd /mnt/synthea && ls
  CCDA  fhir  html
```

### 4. Inspecting the Output of Subsequent Runs

Since you only need to create the `inspector` container once, you can reuse it to inspect the output volume:

```
$ docker start -i inspector
$ cd /mnt/synthea && ls
```

Docker already knows to connect the volume and use TTY. The `-i` tells Docker to use interactive mode.

## Building Synthea

Docker images are tagged with `:tags` to indicate the specific characteristics and configuration of the image. The `:latest` build of Synthea on the Docker Hub uses the latest source code and the defaults in `synthea.yml`. However, `docker:dockerized` should always be set to `true` since we're using Synthea in a Docker environment.

After making changes to Synthea's source code or configuration, use `docker-compose build` to rebuild the Synthea image:

```
$ docker-compose -f docker/ruby/docker-compose.yml build synthea
```

After successfully building the new image, you can optionally push it to the Docker Hub:

```
$ docker push synthetichealth/synthea:latest
```
This process should look familar to `git` users. You will need an account on the Docker Hub and permission to push images to the [SyntheticHealth](https://hub.docker.com/u/synthetichealth/) organization.

## Performance

Synthea is only as fast as the Docker environment it runs in and that environment's host. However, within it's container Synthea will take advantage of all of the resources that are available. By default Docker typically allocates 2 CPUs and 2GB of RAM to running containers. If you need faster performance try adjusting your Docker preferences to allocate more CPUs and additional memory to running containers.







