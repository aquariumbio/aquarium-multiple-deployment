# deployment notes

This setup is roughly based on the configuration for deploying Aquarium at TACC (UT Austin).
Though they use Ansible for building lab-specific configurations, and portainer for running and managing docker containers.

The directory aquarium-instance contains a generic configuration for a single lab instance.
This is basically the same as the [aquarium-local](https://github.com/klavinslab/aquarium-local) configuration with minor (and  incomplete) tweaks related to nginx configuration.

**changes are necessary to `setup.sh` and `docker-compose.yml` before this will work**

## (Intended) Steps for setting up a lab

1. Decide on naming scheme for host names.
   Each Aquarium instance will need two cnames: one for Aquarium itself and the other for the minio service for uploading data.
   The convention we have used is to use the lab name with a prefix that indicates the service.
   For example, the UW BIOFAB might use `aq-biofab` and `data-upload-biofab` as the hostnames for these two services.

2. For each lab, make a copy of the aquarium-instance directory for the lab

   ```bash
   LAB_INSTANCE=lab-name
   cp aquarium-instance $LAB_INSTANCE
   ```

3. run the `setup.sh` script

   ```bash
   cd $LAB_INSTANCE
   bash ./setup.sh
   ```

   (some details are incomplete in this script)

4. set instance details in `$LAB_INSTANCE/config/instance.yml`

5. set the EULA for the lab (this is not for using Aquarium, but for how people use the lab) in `$LAB_INSTANCE/config/lab-eula.yml`


## Running an instance

```bash
docker-compose -p aquarium-haase up --build -d
```

The `-d` runs the container in detached mode.

To shutdown the container

```bash
docker-compose -p aquarium-haase down -v --remove-orphans
```





