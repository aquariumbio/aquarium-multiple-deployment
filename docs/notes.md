# deployment notes

This setup is roughly based on the configuration for deploying Aquarium at TACC (UT Austin).
Though they use Ansible for building lab-specific configurations, and portainer for running and managing docker containers.

The directory aquarium-instance contains a generic configuration for a single lab instance.
This is basically the same as the [aquarium-local](https://github.com/klavinslab/aquarium-local) configuration with minor (and  incomplete) tweaks related to nginx configuration.

Anything that is broken is probably related to [nginx-proxy](https://github.com/jwilder/nginx-proxy)

**changes are necessary to `setup.sh` and `docker-compose.yml` before this will work**

## (Intended) Steps for setting up a lab

1. Decide on naming scheme for host names.
   Each Aquarium instance will need two cnames: one for Aquarium itself and the other for the minio service for uploading data.
   The convention we have used is to use the lab name with a prefix that indicates the service.
   For example, the UW BIOFAB might use `aq-biofab` and `data-upload-biofab` as the hostnames for these two services.

   While you are at it, select a port numbering scheme for the Aquarium and minio services.
   The nginx-proxy service publishes ports 80 and 443

2. For each lab, make a copy of the aquarium-instance directory for the lab

   ```bash
   LAB_INSTANCE=lab-name
   cp -r aquarium-instance $LAB_INSTANCE
   ```

3. run the `setup.sh` script

   ```bash
   cd $LAB_INSTANCE
   bash ./setup.sh $APP_CNAME $APP_PORT $S3_CNAME $S3_PORT $LETSENCRYPT_ADMIN_EMAIL
   ```

   where the CNAMEs are those from the first step.  

   Inspect the `.env` file to make sure that I didn't mess something up

4. set instance details in `$LAB_INSTANCE/config/instance.yml`

5. set the EULA for the lab (this is not for using Aquarium, but for how people use the lab) in `$LAB_INSTANCE/config/lab-eula.yml`

## Starting the proxy server

1. Create a Docker network named `dockernet` that will connect each Aquarium instance to `nginx-proxy`.

   ```bash
   docker network create dockernet
   ```

2. Start the `nginx-proxy` service

   ```bash
   cd nginx-proxy
   docker-compose up -d
   ```

## Running an instance

From within the instance directory, start Aquarium with

```bash
docker-compose -p aquarium-haase up --build -d
```

The `-d` runs the container in detached mode, and the `-p` gives the service a different name than the default.

To shutdown the container

```bash
docker-compose -p aquarium-haase down -v --remove-orphans
```





