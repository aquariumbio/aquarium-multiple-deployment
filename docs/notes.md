# deployment notes

This setup is roughly based on the configuration for deploying Aquarium at TACC (UT Austin).
Though they use Ansible for building lab-specific configurations, and portainer for running and managing docker containers.

The directory `aquarium-instance` contains a generic configuration for a single lab instance.
This is basically the same as the [aquarium-local](https://github.com/klavinslab/aquarium-local) configuration with tweaks that allow use of [nginx-proxy](https://github.com/jwilder/nginx-proxy) to control traffic for multiple instances.
Though this repo could also be used to bolt `https` onto the front of a single instance of Aquarium.

## Before you start

1. Decide on a naming scheme for host names.
   Each Aquarium instance will need two cnames: one for Aquarium itself and the other for the minio service for uploading data.
   The convention used by the deployments at TACC that this repo is based on use the lab name with a prefix that indicates the service, for instance our lab would be `aq-klavins` and `data-upload-klavins`.
   In the typical configuration, users wont use the web interface for the `minio` service directly, so choosing a mnemonic name is not necessary except for your own sanity.

   Also, be sure to use a domain that you or your organization own.

2. For `https`, setup your [certificate authority](https://letsencrypt.org/getting-started/).

   The default configuration uses Let's Encrypt, but it should be possible to use a different CA (see the [nginx-proxy](https://github.com/jwilder/nginx-proxy) readme).
   This will likely require changes to the configuration in the `nginx-proxy` directory.

   Alternatively, you can disable `https` by commenting out the letsencrypt-nginx-proxy-companion service in `nginx-proxy/docker-compose.yml`.

3. Select a port numbering scheme for the Aquarium and minio services so that both services for all labs will use unique ports.
   For instance, use successive numbers starting at 81 for Aquarium instances, and 9001 for minio.

   You wont use these ports directly, but they need to be set to unique values, and at this point nothing is setup to manage these automatically.

## Setting up Aquarium for a lab

1. Choose the CNAMEs for the lab using the chosen scheme. 
   The instructions below refer to the CNAME for Aquarium as `APP_NAME` and the one for the minio service as `S3_NAME`.

2. Choose the ports for the services according to the chosen scheme.
   The instructions use `APP_PORT` for the Aquarium service port, and `S3_PORT` for the minio service port.

3. Make a copy of the aquarium-instance directory for the lab

   ```bash
   LAB_INSTANCE=lab-name
   cp -r aquarium-instance $LAB_INSTANCE
   ```

4. Set `LETSENCRYPT_ADMIN_EMAIL` to the admin email for your Let's Encrypt account, unless you've disabled `https`.

5. run the `setup.sh` script

   ```bash
   cd $LAB_INSTANCE
   bash ./setup.sh $APP_CNAME $APP_PORT $S3_CNAME $S3_PORT $LETSENCRYPT_ADMIN_EMAIL
   ```

   where the CNAMEs are those from the first step.  

   Inspect the `.env` file to ensure that the following are set as expected:

   | Variable | Description |
   |----------|-------------|
   | APP_CNAME | CNAME for lab Aquarium service |
   | APP_PUBLIC_PORT | port for lab Aquarium service |
   | S3_CNAME | CNAME for lab minio service |
   | S3_PUBLIC_PORT | port for lab minio service |
   | LETSENCRYPT_ADMIN_EMAIL | email address for Let's Encrypt |





6. set instance details in `$LAB_INSTANCE/config/instance.yml`

7. set the EULA for the lab (this is not for using Aquarium, but for how people use the lab) in `$LAB_INSTANCE/config/lab-eula.yml`

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

## Running a lab instance

From within the lab Aquarium directory, start Aquarium with

```bash
docker-compose -p $LAB_INSTANCE up --build -d
```

The option `-d` runs the container in detached mode, and the option `-p` gives the service a different name than the default.

To shutdown the container

```bash
docker-compose -p $LAB_INSTANCE down -v --remove-orphans
```





