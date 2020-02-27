# Open Oscar Service Provider

A collection of tools for deploying and running [OSCAR EMR/EHR](https://oscar-emr.com/), an open-source Electronic Medical Record (EMR) for the Canadian family physicians.

This repo was originally based on [scoophealth (UVIC)](https://github.com/scoophealth/oscar-latest-docker)'s fork of [Bell Eapen's](http://nuchange.ca) [Oscar in a Box](https://github.com/dermatologist/oscar-latest-docker).

## Quickstart
  * Install Docker and docker-compose
  * `git clone https://github.com/open-osp/open-osp.git`
  * `cd open-osp`
  * `./deploy-release.sh`
  * Browse to Oscar on localhost!

## Purpose
The goal of this repo is to provide a hosting-agnostic toolkit for automated Oscar EMR deployment. We want to centralize Oscar configurations for modern DevOps tools and share [best practices](https://12factor.net/) for modern web application deployment for Oscar. This may help Service Providers who need to automate deployments, Oscar integrators/vendors/developers who need to do testing, and self-hosted users. At this time, we recommend using OscarEMR DevOps to run Oscar quckly and easily for:

* Training (use ./deploy-release.sh)
* Continuous integration of Oscar integrations (use ./deploy-release.sh or ./deploy-source.sh)
* Testing (use ./deploy-release.sh or ./deploy-source.sh)
* Oscar develpment environments with high [dev/prod parity](https://12factor.net/dev-prod-parity)

## Scope
What does this repo do?

* Builds Oscar from source, for usage locally or to published to DockerHub for use by others.
* Bootstraps a MariaDB database from the same source code as you built from.
* Runs a new containerized Oscar environment including database, in one command (from source or from a tested image).
* Runs drugref locally.

## Usage

./deploy-source.sh will download the latest official develop branch, or a branch specified by `OSCAR_REPO=<url>` and `OSCAR_BRANCH=<branchname>`.

./build-release.sh will build a Docker image to be used with any war file you download to your directory, or one specified in `OSCAR_WAR=<url>`. If neither is found, the latest stable oscar release (currently 15) will be used.

./deploy-release will create a newly installed Oscar environment in the current folder.

./start.sh will resume a previously installed Oscar in this folder.

./purge.sh will completely delete your oscar installation *and database*. Your data will be lost.

We're not aware of anyone using this system for production usage at this time, and don't recommend it until it's more fully tested and supports some features geared towards production usage.

## Prerequisites
* GIT
* Docker
* docker-compose

## To start a new oscar deployment.

```
git clone git@github.com:countable/oscaremr-devops.git
cd oscaremr-devops
```

This process is only for new deployments. It will not work if you have a run it before in the same folder, because you may have EMR data we don't want to overwrite. For a 2nd deployment, just copy the folder again with a new name. If you want to DELETE the database and start from scratch, do `./purge.sh` first.

```
./deploy-release.sh
```

In the future, to bring up Oscar you can just do
```
./start.sh
```

Visit `http://localhost/oscar` in your browser.

## Options

Options may be passed in via environment variables on the host.

* Deploy an Oscar fork - `OSCAR_REPO=https://bitbucket.org/dennis_warren/release-ubcpc-15.10.git ./deploy-source.sh` (when you change this variable, delete the `oscar` directory created in the repo root.
* Deploy a specific commit - `OSCAR_TREEISH=<commit, branch or tag id>`
* Deploy a specific WAR (build) you have downloaded in the same folder - `OSCAR_WARFILE`

## Steps
* Builds the lastest Oscar from source (no development environment setup needed).
* Create Docker containers for Tomcat and MariaDB.
* Bootstraps the database using default BC fixtures (easy to change to ON).

To update
```
docker cp oscar-14.0.0-SNAPSHOT.war tomcat_oscar:/usr/local/tomcat/webapps
```

To shell into the tomcat container
```
docker-compose exec tomcat_oscar bash
```

## Docker Image

A Docker image built from the Dockerfile in this repo is published [here](https://hub.docker.com/repository/docker/openosp/open-osp).

## Customize

If you want to customize some pages, you might want to do these before starting your Oscar instance, or if you have already started an instance you can do
```
docker-compose up --build -d nginx
```
to restart both tomcat_oscar and nginx.

### Customizing Login Page
You can customize the login page by adding environment variables to nginx. You can add/edit a file named 'oscar-login.env' and add variables;
```
LOGIN_TEXT=Html text that can <br> be added in the front page
LOGIN_TITLE=Title you want
BUILD_DATE=JAN-10-20
BUILD_NAME=Oscar-build-v12
BUILD_NUMBER=12
```
### Adding Login page Logo
1. You can add a logo by adding a file in the ./static/images directory named OSCAR-LOGO.png (Recommended)
2. You can change CSS for 'login-logo' id

### Custom CSS
We have provided a sample CSS in ./static/css/oscar-custom.css. Feel free to play with this.

## Adding SSL
After deploying, there will be auto-generated ssl keys that are provided but if you have one for that generated you can simply copy them to the `conf` folder and rename them as `ssl.cert` and `ssl.key`.

You can now restart your OpenOsp by doing `./start.sh`

## Enabling Backups
The backups use AWS to store your database dumps and OscarDocuments. For that you would need to:
1. Create a `~/.aws/credentials` directory containing your credentials OR install awscli `apt install awscli` and run `aws configure`.
2. Create a `local.env` file containing the following.
```
BACKUP_BUCKET=`your directory in aws e.g oscar/backups`
BACKUP_FREQ=`seconds of your backup, defaults to 86400`
```
3. Finally run ./bin/backups.sh to start your backups container.

For more information regarding the backup image, you can go to `https://github.com/countable-web/s3-backup-jobs`

## TODO

For backlog, see the [GitHub issues tab](https://github.com/open-osp/open-osp/issues).

## Thanks (References)
* [Bell Eapen (McMaster U)](http://nuchange.ca) for [Oscar in a Box](https://github.com/dermatologist/oscar-latest-docker)

