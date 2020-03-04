# Aquarium Multi-Deployment

Configuration for running multiple [Aquaria](https://github.com/klavinslab/aquarium) for multiple labs on a single machine.

Can also be used to bolt `https` onto the front of a single instance of Aquarium.

Inspired by the configuration for deploying Aquarium at [TACC](https://www.tacc.utexas.edu/), built by @mwvaughn and @eriksf who modified the Aquarium docker-compose files for use with Ansible, and made tweaks to use [nginx-proxy](https://github.com/jwilder/nginx-proxy).
These tweaks make it possible to control traffic for multiple instances, and enable `https`.

## Resource Considerations

We don't have direct experience with this configuration, but can extrapolate from the configuration we use for the UW BIOFAB.
The BIOFAB is a moderately-sized cloud lab with at most 10 protocol batches being run simultaneously at any given time.
Based on the hardware used by the BIOFAB, we expect that a 2.5 GHz processor with 8 cores and 16-32 GB RAM, with a fast 1 TB SSD would be sufficient for a core service lab or multiple small labs with comparable bandwidth.

Note that we do not store NGS files in our object store.

## Before you start

1. Decide on a naming scheme for host names.

   Each Aquarium instance will need two cnames: one for Aquarium itself and the other for the minio object store service.

   The convention we have used with TACC is to use the lab name with the service "name" as prefix.
   For example, the cname for the Aquarium instance for our lab would be `aq-klavins`, while the cname for the minio service would be `data-upload-klavins` or perhaps `s3-klavins`.
   In the typical configuration, users won't use the web interface for the `minio` service directly, so choosing a mnemonic name is not necessary except for your own sanity.

   *Use a domain that you or your organization own.*

2. For `https`, set up your [certificate authority](https://letsencrypt.org/getting-started/).

   The configuration uses [Let's Encrypt](https://letsencrypt.org) by default, but it should be possible to use a different CA (see the [nginx-proxy](https://github.com/jwilder/nginx-proxy) readme).
   This will require changes to the configuration in the `nginx-proxy` directory.

   Note that it is possible to start the `nginx-proxy` service without the `letsencrypt-nginx-proxy-companion` service (e.g., without https).

3. Select a port numbering scheme for the Aquarium and minio services.

   For example, use successive numbers starting at 81 for Aquarium instances, and 9001 for minio.

   You won't use these ports directly, but they need to be set to unique values for each service and for all labs.

## Setting up Aquarium for a lab

1. Choose the CNAMEs for the lab using the chosen scheme.

   The instructions below refer to the CNAME for Aquarium as `AQ_CNAME` and the one for the minio service as `S3_CNAME`.

2. Choose the ports for the services according to the chosen scheme.

   The instructions below use `AQ_PORT` for the Aquarium service port, and `S3_PORT` for the minio service port.

3. Make a copy of the aquarium-instance directory for the lab

   ```bash
   LAB_INSTANCE=lab-name
   cp -r aquarium-instance $LAB_INSTANCE
   ```

4. Set `LETSENCRYPT_ADMIN_EMAIL` to the admin email for your Let's Encrypt account, unless you are not using `https`.

5. Run the `setup.sh` script with the CNAMEs set to those from the first step.  

   ```bash
   cd $LAB_INSTANCE
   bash ./setup.sh $AQ_CNAME $AQ_PORT $S3_CNAME $S3_PORT $LETSENCRYPT_ADMIN_EMAIL
   ```


   Inspect the `.env` file to ensure that the following are set as expected:

   | Variable | Description |
   |----------|-------------|
   | APP_CNAME | CNAME for lab Aquarium service |
   | APP_PUBLIC_PORT | port for lab Aquarium service |
   | S3_CNAME | CNAME for lab minio service |
   | S3_PUBLIC_PORT | port for lab minio service |
   | LETSENCRYPT_ADMIN_EMAIL | email address for Let's Encrypt |

## Lab Customizations

1. Set instance details in `$LAB_INSTANCE/config/instance.yml`.
   
   You should set the value for `instance_name` to the name of the lab (e.g., `Klavins Lab`).
   If the lab will send email to users, set `lab_email_address` to an address for the lab.

   You can change `image_uri` to point at an image server, though minio is used by default.

2. Set the EULA for the lab in `$LAB_INSTANCE/config/lab-eula.yml`.

   This is not the user agreement for using Aquarium, but for how people use the lab.
   So, the content should be determined by the lab to which the Aquarium instance belongs.

## Starting the proxy server

1. Create a Docker network named `dockernet` that will connect each Aquarium instance to `nginx-proxy`.

   ```bash
   docker network create dockernet
   ```

2. To start the `nginx-proxy` service with the `letsencrypt-nginx-proxy-companion` service (e.g., using https), run the command

   ```bash
   cd nginx-proxy
   bash ./deploy.sh up -d
   ```

   To start `nginx-proxy` without https run the command

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

To shut down the container

```bash
docker-compose -p $LAB_INSTANCE down -v --remove-orphans
```
