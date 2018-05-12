# Docker Magento 2

A proof-of-concept repo for dockerising a Magento 2 development environment.

## TODO

* ~~Add ability to specify custom admin URI~~
* ~~Refactor mounting of `~/.composer/` into `COMPOSER_HOME` env var to make it platform agnostic and in order to get passwordless-install when installing via composer and make installing quicker~~
* Merge the copying of code from `/var/www/html` on the appdata to `html` on the host into the mage-setup-raw script 
* Update service versions (e.g. Upgrade nginx to 1.13)
* Add XDebug support
* Install frontend tools (yarn, snowdog frontools, etc)
* Update the environment variables
* Optimise the service configs for M2
    * Nginx
    * ~~PHP~~
    * MySQL
* Add Redis support
* Create a version for updating projects
* Lint Dockerfiles using [FROM:latest](https://www.fromlatest.io/)
* Make it possible to run multiple magento2-docker-compose installs without ports clashing

## Docker development workflow
 
 * Checkout this repo in a new folder, named after the feature you're implementing
 * Modify the `docker-compose.yml` and `env` files accordingly to support your new feature (if necessary)

If modifying the base images:

 * Modify the images accordingly and tag them with a new version
 * Update the `docker-compose.yml` to use the new version of the image
 * Once the feature has been completed, re-build the image with the `latest` tag

## Usage

### First-time

* Clone the repo:
```bash
git clone git@github.com:ProjectEight/docker-magento2-compose.git magento2-project
```
* Run the `setup` container to setup the environment. This downloads the images if you don't already have them locally and then creates all the containers.

```bash
$ docker-compose up -d setup
```

The `-d` flag tells Docker Compose to run all the containers in the background.

Now modify the `env/setup.env` file to suit the project requirements:

```bash
MAGENTO2_DB_HOST=db
MAGENTO2_DB_NAME=magento2
MAGENTO2_DB_USER=magento2
MAGENTO2_DB_PASSWORD=magento2
MAGENTO2_BASE_URL=http://magento2-project.dev:8000/
MAGENTO2_ADMIN_FIRSTNAME=Admin
MAGENTO2_ADMIN_LASTNAME=User
MAGENTO2_ADMIN_EMAIL=admin@example.com
MAGENTO2_ADMIN_USER=admin
MAGENTO2_ADMIN_PASSWORD=password123
MAGENTO2_ADMIN_URI=admin
MAGENTO2_CURRENCY=GBP
MAGENTO2_LANGUAGE=en_GB
MAGENTO2_TIMEZONE=Europe/London
MAGENTO2_VERSION=2.2.4
```

Do the same for `env/mysql.env`.

Now we're ready to install Magento 2:

```bash
$ docker-compose run --rm setup
```

This ties all the containers we just created together and also installs Magento 2 in a volume (data container).

The `--rm` flag tells Docker to remove the `setup` container after it has finished executing the commands defined in the `RUN` section of the Dockerfile.

Finally, modify the hosts file on your host (not inside the container):

```bash
$ sudo echo "127.0.0.1       magento2-project.dev" >> /etc/hosts
```

### Every other time

* Run the `app` container to start the environment:

```bash
$ docker-compose up -d app
```

### Switching projects

* First, stop `docker-compose` using:

```bash
$ docker-compose stop
```

* Second, switch to the other project folder and start that environment:

```bash
$ docker-compose up -d app
```

## In detail

### Composer Setup

#### Authentication

Uncomment the composer line from `appdata` to mount a `.composer` directory to the `www-data` user home directory. Please first setup Magento Marketplace authentication (details at <a href="http://devdocs.magento.com/guides/v2.0/install-gde/prereq/connect-auth.html" target="_blank">http://devdocs.magento.com/guides/v2.0/install-gde/prereq/connect-auth.html</a>).

Place your auth token at `~/.composer/auth.json` with the following contents, like so:

```json
{
    "http-basic": {
        "repo.magento.com": {
            "username": "MAGENTO2_PUBLIC_KEY",
            "password": "MAGENTO2_PRIVATE_KEY"
        }
    }
}
```

## Data Volumes

Your Magento source data is persistently stored within Docker data volumes. For local development, we advise copying the entire contents of the `appdata` data volume to your local machine (after setup is complete of course). Since you shouldn't be modifying any of these files, this is just to bring the fully copy of the site back to your host:

```bash
docker cp CONTAINERID:/var/www/html ./
```

Then, just uncomment the `./html/app/code:/var/www/html/app/code` and `./html/app/design:/var/www/html/app/design` lines within your docker-compose.override.yml file (appdata > volumes). This mounts your local `app/code` and `app/design` directories to the Docker data volume. Then, just restart your containers:

```bash
docker-compose up -d app
```

Any edits to these directories will correctly sync with your Docker volume.

## Running Magento CLI tool

We've setup scripts to aid in the running of Magento CLI tool with the correct permissions. To run the command line tool, you would connect as any other Docker Compose application would:

```bash
docker-compose exec phpfpm ./bin/magento
```

or with straight Docker command:
```bash
docker exec NAME_OF_PHPFPM_CONTAINER ./bin/magento
```

You can easily set these up as aliases inside your `~/.bash_profile` file (or a similar script) as so:

```bash
alias magento='docker-compose exec phpfpm ./bin/magento'
```
This will allow you to clear the cache by running the following command right in terminal:

```bash
magento cache:flush
```

## Docker Compose Override

You can copy `docker-compose.override.yml.dist` to `docker-compose.override.yml` and adjust environment variables, volume mounts etc in the `docker-compose.override.yml` file to avoid losing local configuration changes when you pull changes to this repository. 

Docker Compose will automatically read any of the values you define in the file. See [this link](https://docs.docker.com/compose/extends/#/understanding-multiple-compose-files) for more information about the override file. 
