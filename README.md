# Oracle Database 11g xe container images

## How to build and run

### 1. Download Installation Binaries

**IMPORTANT:** You will have to provide the installation binaries of Oracle Database (except for Oracle Database 18c XE) and put them into the `dockerfiles/<version>` folder. You only need to provide the binaries for the edition you are going to install. The binaries can be downloaded from the [Oracle Technology Network](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html), make sure you use the linux link: *Linux x86-64*. The needed file is named *linuxx64_\<version\>_database.zip*. You also have to make sure to have internet connectivity for yum. Note that you must not uncompress the binaries. The script will handle that for you and fail if you uncompress them manually!

Before you build the image make sure that you have provided the installation binaries and put them into the root directory.

### 2. Build a image

```bash
docker build --force-rm=true --no-cache=true --build-arg DB_EDITION=xe -t oracle/database:11.2.0.2-xe -f Dockerfile.xe .
```

**IMPORTANT:** The resulting images will be an image with the Oracle binaries installed. On first startup of the container a new database will be created, the following lines highlight when the database is ready to be used:

    #########################
    DATABASE IS READY TO USE!
    #########################

You may extend the image with your own Dockerfile and create the users and tablespaces that you may need.

The character set for the database is set during creating of the database. 11gR2 Express Edition supports only UTF-8. You can set the character set for the Standard Edition 2 and Enterprise Edition during the first run of your container and may keep separate folders containing different tablespaces with different character sets.

#### Running Oracle Database 11gR2 Express Edition in a container

To run your Oracle Database Express Edition container image use the `docker run` command as follows:

    docker run --name <container name> \
    --shm-size=1g \
    -p 1521:1521 -p 8080:8080 \
    -e ORACLE_PWD=<your database passwords> \
    -v [<host mount point>:]/u01/app/oracle/oradata \
    oracle/database:11.2.0.2-xe
    
    Parameters:
       --name:        The name of the container (default: auto generated)
       --shm-size:    Amount of Linux shared memory
       -p:            The port mapping of the host port to the container port.
                      Two ports are exposed: 1521 (Oracle Listener), 8080 (APEX)
       -e ORACLE_PWD: The Oracle Database SYS, SYSTEM and PDB_ADMIN password (default: auto generated)

       -v /u01/app/oracle/oradata
                      The data volume to use for the database.
                      **Has to be writable by the Unix "oracle" (uid: 1000) user inside the container!**
                      If omitted the database will not be persisted over container recreation.
       -v /u01/app/oracle/scripts/startup | /docker-entrypoint-initdb.d/startup
                      Optional: A volume with custom scripts to be run after database startup.
                      For further details see the "Running scripts after setup and on startup" section below.
       -v /u01/app/oracle/scripts/setup | /docker-entrypoint-initdb.d/setup
                      Optional: A volume with custom scripts to be run after database setup.
                      For further details see the "Running scripts after setup and on startup" section below.

There are two ports that are exposed in this image:

* 1521 which is the port to connect to the Oracle Database.
* 8080 which is the port of Oracle Application Express (APEX).

On the first startup of the container a random password will be generated for the database if not provided. You can find this password in the output line:

    ORACLE PASSWORD FOR SYS AND SYSTEM:

**Note:** The ORACLE_SID for Express Edition is always `XE` and cannot be changed, hence there is no ORACLE_SID parameter provided for the XE build.

The password for those accounts can be changed via the `docker exec` command. **Note**, the container has to be running:

    docker exec <container name> /u01/app/oracle/setPassword.sh <your password>

Once the container has been started you can connect to it just like to any other database:

    sqlplus sys/<your password>@//localhost:1521/XE as sysdba
    sqlplus system/<your password>@//localhost:1521/XE

### Deploying Oracle Database on Kubernetes

Helm is a package manager which uses a packaging format called charts. [helm-charts](helm-charts/) directory contains all the relevant files needed to deploy Oracle Database on Kubernetes. For more information on default configuration, installing/uninstalling the Oracle Database chart on Kubernetes, please refer [helm-charts/oracle-db/README.md](helm-charts/oracle-db/README.md).

### Running SQL*Plus in a container

You may use the same container image you used to start the database, to run `sqlplus` to connect to it, for example:

    docker run --rm -ti oracle/database:19.3.0-ee sqlplus pdbadmin/<yourpassword>@//<db-container-ip>:1521/ORCLPDB1

Another option is to use `docker exec` and run `sqlplus` from within the same container already running the database:

    docker exec -ti <container name> sqlplus pdbadmin@ORCLPDB1

### Running scripts after setup and on startup

The container images can be configured to run scripts after setup and on startup. Currently `sh` and `sql` extensions are supported.
For post-setup scripts just mount the volume `/opt/oracle/scripts/setup` or extend the image to include scripts in this directory.
For post-startup scripts just mount the volume `/opt/oracle/scripts/startup` or extend the image to include scripts in this directory.
Both of those locations are also represented under the symbolic link `/docker-entrypoint-initdb.d`. This is done to provide
synergy with other database container images. The user is free to decide whether to put the setup and startup scripts
under `/opt/oracle/scripts` or `/docker-entrypoint-initdb.d`.

After the database is setup and/or started the scripts in those folders will be executed against the database in the container.
SQL scripts will be executed as sysdba, shell scripts will be executed as the current user. To ensure proper order it is
recommended to prefix your scripts with a number. For example `01_users.sql`, `02_permissions.sql`, etc.

**Note:** The startup scripts will also be executed after the first time database setup is complete.  
**Note:** For 11gR2 Express Edition only, use `/u01/app/oracle/scripts/` instead of `/opt/oracle/scripts/`.

The example below mounts the local directory myScripts to `/opt/oracle/myScripts` which is then searched for custom startup scripts:

    docker run --name oracle-ee -p 1521:1521 -v /home/oracle/myScripts:/opt/oracle/scripts/startup -v /home/oracle/oradata:/opt/oracle/oradata oracle/database:19.3.0-ee

## License

To download and run Oracle Database, regardless whether inside or outside a container, you must download the binaries from the Oracle website and accept the license indicated at that page.

All scripts and files hosted in this project and GitHub [docker-images/OracleDatabase](./) repository required to build the container images are, unless otherwise noted, released under [UPL 1.0](https://oss.oracle.com/licenses/upl/) license.

## Copyright

Copyright (c) 2014,2021 Oracle and/or its affiliates.
