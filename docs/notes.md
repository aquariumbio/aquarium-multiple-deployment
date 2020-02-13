# deployment notes

This setup is roughly based on the configuration for deploying Aquarium at TACC (UT Austin).
Though they use Ansible for building lab-specific configurations, and portainer for running and managing docker containers.

The directory aquarium-instance contains a generic configuration for a single lab instance.
This is basically the same as the [aquarium-local](https://github.com/klavinslab/aquarium-local) configuration with minor (and  incomplete) tweaks related to nginx configuration.

**changes are necessary to `setup.sh` and `docker-compose.yml` before this will work**

## (Intended) Steps for setting up a lab

1. make a copy of the aquarium-instance directory for the lab

   ```bash
   cp aquarium-instance aquarium-haase
   ```

2. run the `setup.sh` script

   ```bash
   cd aquarium-haase
   bash ./setup.sh
   ```

   (some details are incomplete in this script)

3. set instance details in `aquarium-haase/config/instance.yml`

4. set the EULA for the lab (this is not for using Aquarium, but for how people use the lab) in `aquarium-haase/config/lab-eula.yml`


## Running an instance

```bash
docker-compose -p aquarium-haase up --build -d
```

The `-d` runs the container in detached mode.

To shutdown the container

```bash
docker-compose -p aquarium-haase down -v --remove-orphans
```





