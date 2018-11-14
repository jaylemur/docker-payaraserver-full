Updated repository for Payara Dockerfiles. This repository is for the **Full Profile** of [Payara Server](http://www.payara.fish).

# Supported tags and respective `Dockerfile` links

-	[`latest`](https://github.com/payara/docker-payaraserver-full/blob/master/Dockerfile)
  - contains latest released version of Payara Server Full Profile
-	[`prerelease`](https://github.com/payara/docker-payaraserver-full/blob/prerelease/Dockerfile)
  - contains nightly build of Payara Server Full Profile from the master branch (updated daily)
-	[other tags](https://hub.docker.com/r/payara/server-full/tags/) correspond to past releases of Payara Server Full Profile matched by short version number

# Usage

## Quick start

To boot the default domain with HTTP listener exported on port 8080:

```
docker run -p 8080:8080 payara/server-full
```

The Docker container specifies the default entry point, which starts a custom domain `docker-domain` in foreground so that Payara Server becomes the main process.

## Open ports

Most common default open ports that can be exposed outside of the container:

 - 8080 - HTTP listener
 - 8181 - HTTPS listener
 - 4848 - HTTPS admin listener
 - 9009 - Debug port

## Administration

To boot and export admin interface on port 4848 (and also the default HTTP listener on port 8080):

```
docker run -p 4848:4848 -p 8080:8080 payara/server-full
```

Because Payara Server doesn't allow insecure remote admin connections (outside of a Docker container), the admin interface is secured by default, accessible using HTTPS on the host machine: [https://localhost:4848](https://localhost:4848) The default user and password is `admin`.

## Application deployment

### Remote deployment

Once admin port is exposed, it is possible to deploy applications remotely, outside of the docker container, by means of admin console and asadmin tool as usual.

### Deployment on startup

The docker image supports 2 ways of deploying applications on server startup.

#### Deployment on startup using a startup script

Since version 172, Payara Server supports running asadmin commands automatically after the domain is started, including the deploy command to deploy applications. This is the preferred way to deploy applications on Docker container startup.

The default Docker entry point will scan the folder `$DEPLOY_DIR` for files and folders and deploy them automatically after the domain is started.

In order to deploy applications, you can mount the `$DEPLOY_DIR` (`/opt/payara/deployments`) folder as a docker volume to a directory, which contains your applications. The following will run Payara Server in the docker and will start applications that exist in the directory `~/payara/apps` on the local file-system:

```
docker run -p 8080:8080 -v ~/payara/apps:/opt/payara/deployments payara/server-full
```

In order to build a Docker image that contains your applications and starts them automatically, you can copy the applications into the `$DEPLOY_DIR` directory. and run the resulting docker image instead of the original one.

The following example Dockerfile will build an image that starts Payara Server and deploys `myapplication.war` when the Docker container is started:

```
FROM payara/server-full

COPY myapplication.war $DEPLOY_DIR
```

You can now build the Docker image and run the application `myapplication.war` with the following commands:

```
docker build -t mycompany/myapplication .
```

```
docker run -p 8080:8080 mycompany/myapplication
```

## The default Docker entry point

The default entry point is `/bin/bash` as per the docker default. The default `CMD` command runs the following scripts:

- `${SCRIPTS_DIR}/generate_deploy_commands.sh`. This script outputs deploy commands to the post boot command file located at `$POSTBOOT_COMMANDS` (default `$CONFIG_DIR/post-boot-commands.asadmin`). If the deploy commands are already found in that file, this script does nothing.
- `${SCRIPTS_DIR}/startInForeground.sh`. This script starts the server in the foreground, in a manner that allows the Payara instance to be controlled by the docker host. The server will run the pre boot commands found in the file at `$PREBOOT_COMMANDS`, as well as the post boot commands found in the file at `$POSTBOOT_COMMANDS`.

Because of the default entrypoint being unchanged, you can override the default instruction set to run any number of preparatory commands for testing. For example, the following command will start the container at a bash prompt, allowing you to browse the image and configure the Payara Server instance as you like:

```
docker run -p 8080:8080 -it payara/server-full bash
```

It's possible to run a custom set of asadmin commands either by specifying the `POSTBOOT_COMMANDS` environment variable to point to the absolute path of the custom post boot command file, or by providing a custom file located at `$POSTBOOT_COMMANDS` (default `$CONFIG_DIR/post-boot-commands.asadmin`).

## Environment Variables

The following environment variables are available to be used. When edited either in a `Dockerfile` or before the `startInForeground.sh` script is ran, they will change the behaviour of the Payara Server instance.

- `JVM_ARGS` - Specifies a list of JVM arguments which will be passed to Payara in the `startInForeground.sh` script.
- `DEPLOY_PROPS` - Specifies a list of properties to be passed with the deploy commands generated in the `generate_deploy_commands.sh` script, For example `'--properties=implicitCdiEnabled=false'`.
- `POSTBOOT_COMMANDS` - The name of the file containing post boot commands for the Payara Server instance. This is the file written to in the `generate_deploy_commands.sh` script.
- `PREBOOT_COMMANDS` - The name of the file containing pre boot commands for the Payara Server instance.
- `AS_ADMIN_MASTERPASSWORD` - The master password to pass to Payara Server. This is overriden if one is specified in the `$PASSWORD_FILE`.

The following environment variables shouldn't be changed, but may be helpful in your Dockerfile.

|  Variable name  |           Value            | Description |
| --------------- | -------------------------- | ----------- |
| `HOME_DIR`      | `/opt/payara`              | The home directory for the `payara` user |
| `PAYARA_DIR`    | `/opt/payara/appserver`    | The root directory of the Payara installation |
| `SCRIPT_DIR`    | `/opt/payara/scripts`      | The directory where the `generate_deploy_commands.sh` and `startInForeground.sh` scripts can be found. |
| `CONFIG_DIR`    | `/opt/payara/config`       | The directory where the post and pre boot files are generated to by default. |
| `DEPLOY_DIR`    | `/opt/payara/deployments`  | The directory where applications are searched for in `generate_deploy_commands.sh` script. |
| `PASSWORD_FILE` | `/opt/payara/passwordFile` | The location of the password file for asadmin. This can be passed to asadmin using the `--passwordfile` parameter. |

# Details

Payara Server installation is located in the `/opt/payara` directory. This directory is the default working directory of the docker image. The directory name is deliberately free of any versioning so that any scripts written to work with one version can be seamlessly migrated to the latest docker image.

- Full and Web editions are derived from the OpenJDK 8 images with a Debian Jessie base
- Micro editions are built on OpenJDK 8 images with an Alpine Linux base to keep image size as small as possible.

Payara Server is a patched, enhanced and supported application server derived from GlassFish Server Open Source Edition 4.x. Visit [www.payara.fish](http://www.payara.fish) for full 24/7 support and lots of free resources.

Full Payara Server and Payara Micro documentation: [https://docs.payara.fish/](https://docs.payara.fish/)
