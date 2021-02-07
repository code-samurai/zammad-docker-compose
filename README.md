# Welcome to Zammad

Zammad is a web based open source helpdesk/ticket system with many features
to manage customer communication via several channels like telephone, facebook,
twitter, chat and e-mails. It is distributed under the GNU AFFERO General Public
 License (AGPL). Do you receive many e-mails and want to answer them with a team of agents?
You're going to love Zammad!

## Use case for this repo

This repo is meant to be the starting point for somebody who likes to use dockerized multi-container Zammad in production.

## Getting started with zammad-docker-compose

<https://docs.zammad.org/en/latest/install-docker-compose.html>

## CI Status

[![CI Status](https://github.com/zammad/zammad-docker-compose/workflows/ci/badge.svg)](https://github.com/zammad/zammad-docker-compose/actions)

## Using a reverse proxy

In environments with more then one web applications it is necessary to use a reverse proxy to route connections to port 80 and 443 to the right application.
To run Zammad behind a revers proxy, we provide `docker-compose.proxy-example.yml` as a starting point.

1. Copy `./.examples/proxy/docker-compose.proxy-example.yml` to your own configuration, e.g. `./docker-compose.prod.yml`  
    `cp ./.examples/proxy/docker-compose.proxy-example.yml ./docker-compose.prod.yml`
2. Modify the environment variable `VIRTUAL_HOST` and the name of the external network in `./docker-compose.prod.yml` to fit your environment.
3. Run docker-composer commands with the default and your configuration, e.g. `docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d`  

See `.examples/proxy/docker-compose.yml` for an example proxy project.

Like this, you can add your `docker-compose.prod.yml` to a branch of your Git repository and stay up to date by merging changes to your branch.

## Using Rancher

```console
RANCHER_URL=http://RANCHER_HOST:8080 rancher-compose --env-file=.env up
```

## Running without Elasticsearch

Elasticsearch is an optional, but strongly recommended dependency for Zammad. More details can be found in the [documentation](https://docs.zammad.org/en/latest/prerequisites/software.html#elasticsearch-optional). There are however certain scenarios when running without Elasticsearch may be desired, e.g. for very small teams, for teams with limited budget or as a temporary solution for an unplanned Elasticsearch downtime or planned cluster upgrade.

Elasticsearch is enabled by default in the example `docker-compose.yml` file. It is also by default required to run the "zammad-init" command. Disabling Elasticsearch is possible by setting a special environment variable: `ELASTICSEARCH_ENABLED=false` for the `zammad-init` container and removing all references to Elasticsearch everywhere else: the `zammad-elasticsearch` container, it's volume and links to it.

## Upgrading

### From =< 3.3.0-12

We've updated the Elasticsearch image from 5.6 to 7.6.
As there is no direct upgrade path we have to delete all Elasticsearch indices and rebuild them.
Do the following to empty the ES docker volume:

```console
docker-compose stop
set -o pipefail DOCKER_VOLUME="$(docker volume inspect zammaddockercompose_elasticsearch-data | grep Mountpoint | sed -e 's#.*": "##g' -e 's#",##')/*"
echo "${DOCKER_VOLUME}" #check this is a valid docker volume path! if not do not proceed or you might lose data!
rm -r $(docker volume inspect zammaddockercompose_elasticsearch-data | grep Mountpoint | sed -e 's#.*": "##g' -e 's#",##')/*
docker-compose start
```

To workaround the [changes in the PostgreSQL 9.6 container](https://github.com/docker-library/postgres/commit/f1bc8782e7e57cc403d0b32c0e24599535859f76) do the following:

```console
docker-compose start
docker exec -it zammaddockercompose_zammad-postgresql_1 bash
psql --username postgres --dbname zammad_production
CREATE USER zammad;
ALTER USER zammad WITH PASSWORD 'zammad';
ALTER USER zammad WITH SUPERUSER CREATEDB;
```
