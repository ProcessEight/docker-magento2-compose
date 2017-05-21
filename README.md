# Docker Magento 2

A proof-of-concept repo for dockerising a Magento 2 development environment.

## TODO

* ~~Add ability to specify custom admin URI~~
* Refactor mounting of `~/.composer/` into `COMPOSER_HOME` env var to make it platform agnostic and in order to get passwordless-install when installing via composer and make installing quicker
* Merge the copying of code from `/var/www/html` on the appdata to `html` on the host into the mage-setup-raw script 
* Update service versions (e.g. Upgrade nginx to 1.13)
* Add XDebug support
* Install frontend tools (node, yarn, etc)
* Update the environment variables
* Add Windows support
* Optimise the service configs for M2
* Add Redis support
* Create a version for updating projects
* Lint Dockerfiles using [FROM:latest](https://www.fromlatest.io/)
* Make it possible to run multiple magento2-docker-compose installs without ports clashing

## Docker development workflow
 
 * Checkout this repo in a new folder, named after the feature you're implementing
 * Modify the `docker-compose.yml` and `env` files accordingly to support your new feature (if necessary)

If modifying the images:

 * Modify the images accordingly and tag them with a new version
 * Update the `docker-compose.yml` to use the new version of the image
 * Once the feature has been completed, re-build the image with the `latest` tag

## Usage

### First-time

This is a two-step process:

* Build the images
    * This creates M2-optimised versions of PHP, Nginx, MySQL images
* Run the `setup` container to setup the environment
    * This uses the images to create the environment and installs Magento 2 in a volume (data container)

### Every other time

* Run the `app` container to run the environment:

```bash
$ docker-compose up -d app
```

### Windows

It's best to run this on Windows 10 with `Docker for Windows`, otherwise the performance of the container may be unacceptably slow.

* Install `Docker for Windows`
* If the installer asks you to enable `Hyper-V` then select Yes.

## In detail

### Composer Setup

#### Authentication

Uncomment the composer line from `appdata` to mount a `.composer` directory to the `www-data` user home directory. Please first setup Magento Marketplace authentication (details at <a href="http://devdocs.magento.com/guides/v2.0/install-gde/prereq/connect-auth.html" target="_blank">http://devdocs.magento.com/guides/v2.0/install-gde/prereq/connect-auth.html</a>).

Place your auth token at `~/.composer/auth.json` with the following contents, like so:

```
{
    "http-basic": {
        "repo.magento.com": {
            "username": "MAGENTO_PUBLIC_KEY",
            "password": "MAGENTO_PRIVATE_KEY"
        }
    }
}
```

Then, just set `M2SETUP_USE_ARCHIVE` to `false` in your docker-compose.yml file. 

## First-time Setup

Using the above `docker-compose.yml` file, all you need to do is run one line to install Magento 2:

```
docker-compose run --rm setup
```

This will 
You may modify any environment variables depending on your requirements.

## Data Volumes

Your Magento source data is persistently stored within Docker data volumes. For local development, we advise copying the entire contents of the `appdata` data volume to your local machine (after setup is complete of course). Since you shouldn't be modifying any of these files, this is just to bring the fully copy of the site back to your host:

```
docker cp CONTAINERID:/var/www/html ./
```

Then, just uncomment the `./html/app/code:/var/www/html/app/code` and `./html/app/design:/var/www/html/app/design` lines within your docker-compose.override.yml file (appdata > volumes). This mounts your local `app/code` and `app/design` directories to the Docker data volume. Then, just restart your containers:

```
docker-compose up -d app
```

Any edits to these directories will correctly sync with your Docker volume.

## Running Magento CLI tool

We've setup scripts to aid in the running of Magento CLI tool with the correct permissions. To run the command line tool, you would connect as any other Docker Compose application would:

```
docker-compose exec phpfpm ./bin/magento
```

or with straight Docker command:
```
docker exec NAME_OF_PHPFPM_CONTAINER ./bin/magento
```

You can easily set these up as aliases inside your `~/.bash_profile` file (or a similar script) as so:

```
alias magento='docker-compose exec phpfpm ./bin/magento'
```
This will allow you to clear the cache by running the following command right in terminal:

```
magento cache:flush
```

## Docker Compose Override

You can copy `docker-compose.override.yml.dist` to `docker-compose.override.yml` and adjust environment variables, volume mounts etc in the `docker-compose.override.yml` file to avoid losing local configuration changes when you pull changes to this repository. 

Docker Compose will automatically read any of the values you define in the file. See [this link](https://docs.docker.com/compose/extends/#/understanding-multiple-compose-files) for more information about the override file. 
