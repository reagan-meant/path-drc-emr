# PATH DRC EMR

This project holds the build configuration for the PATH DRC EMR MVP application, found on https://om.rs/drc, built on the community's reference application EMR "OpenMRS 3.0".

## Quick start

### Package the distribution and prepare the run

```
docker compose build
```

### Run the app

```
docker compose up
```

The new OpenMRS UI is accessible at http://localhost/openmrs/spa

OpenMRS Legacy UI is accessible at http://localhost/openmrs

## Overview

This distribution consists of four images:

* db - This is just the standard MariaDB image supplied to use as a database
* backend - This image is the OpenMRS backend. It is built from the main Dockerfile included in the root of the project and
  based on the core OpenMRS Docker file. Additional contents for this image are drawn from the `distro` sub-directory which
  includes a full Initializer configuration for the reference application intended as a starting point.
* frontend - This image is a simple nginx container that embeds the 3.x frontend, including the modules described in  the
  `frontend/spa-build-config.json` file.
* gateway - This image is an even simpler nginx reverse proxy that sits in front of the `backend` and `frontend` containers
  and provides a common interface to both. This helps mitigate CORS issues.

## Contributing to the configuration

This project uses the [Initializer](https://github.com/mekomsolutions/openmrs-module-initializer) module
to configure metadata for this project. The Initializer configuration can be found in the configuration
subfolder of the distro folder. Any files added to this will be automatically included as part of the
metadata for the RefApp.

Eventually, we would like to split this metadata into two packages:

* `openmrs-core`, which will contain all the metadata necessary to run OpenMRS
* `openmrs-demo`, which will include all of the sample data we use to run the RefApp

The `openmrs-core` package will eventually be a standard part of the distribution, with the `openmrs-demo`
provided as an optional add-on. Most data in this configuration _should_ be regarded as demo data. We
anticipate that implementation-specific metadata will replace data in the `openmrs-demo` package,
though they may use that metadata as a starting point for that customization.

To help us keep track of things, we ask that you suffix any files you add with either
`-core_demo` for files that should be part of the demo package and `-core_data` for
those that should be part of the core package. For example, a form named `test_form.json` would become
`test_core-core_demo.json`.

Frontend configuration can be found in `frontend/config-core_demo.json`.

Thanks!

# Backup and Restore

This project provides out-of-the-box backup and restore functionality for your OpenMRS deployment, powered by the robust [Restic](https://restic.readthedocs.io/en/stable/) tool. This enables you to back up and restore your application data using a wide variety of storage backends supported by Restic, such as local disk, S3, Azure, Google Cloud Storage, and more.

## How It Works

- **Backup** and **restore** are managed by dedicated Docker Compose services using the `mekomsolutions/restic-compose-backup` and `mekomsolutions/restic-compose-backup-restore` images.
- The backup service periodically snapshots your data volumes and database, storing them in your configured Restic repository.
- The restore service can be used to recover your data from a specific snapshot.

## Configuration

### Environment Variables

You can configure backup and restore behavior using the following environment variables in your `.env` file or directly in your Compose files:

| Variable                  | Description                                                                                  |
|---------------------------|----------------------------------------------------------------------------------------------|
| `RESTIC_REPOSITORY`       | The Restic repository URL (e.g., local path, s3, etc.)                                      |
| `RESTIC_PASSWORD`         | Password for the Restic repository                                                          |
| `RESTIC_KEEP_DAILY`       | Number of daily snapshots to keep                                                           |
| `RESTIC_KEEP_WEEKLY`      | Number of weekly snapshots to keep                                                          |
| `RESTIC_KEEP_MONTHLY`     | Number of monthly snapshots to keep                                                         |
| `RESTIC_KEEP_YEARLY`      | Number of yearly snapshots to keep                                                          |
| `LOG_LEVEL`               | Log verbosity (e.g., `info`, `debug`)                                                       |
| `CRON_SCHEDULE`           | Cron schedule for automatic backups (e.g., `0 2 * * *` for daily at 2am)                    |
| `RESTIC_RESTORE_SNAPSHOT` | (Restore only) The snapshot ID or tag to restore                                           |
| `BACKUP_PATH`             | Local directory for storing backup repository (default: `./backup`)                          |

### Volumes and Labels

- The `backend` and `db` services are labeled and configured to include their data volumes in the backup.
- Additional volumes (e.g., for configuration checksums, person images, complex obs) are also included and labeled for backup.

## Usage

### Restore

To restore from a backup:

1. Set the `BACKUP_PATH`, `RESTIC_PASSWORD`, and `RESTIC_RESTORE_SNAPSHOT` environment variables.
2. Start the restore service:

```sh
docker compose -f docker-compose.yml -f docker-compose-restore.yml up -d
```

This will restore the specified snapshot to the appropriate volumes. The `backend` and `db` services are configured to wait for the restore to complete before starting.

***IMPORTANT*** 

The restore process will leave a restore container that will block the backup process. To clean up the restore container, run the following command: 

```sh
docker compose -f docker-compose.yml -f docker-compose-restore.yml rm restore
docker compose -f docker-compose.yml -f docker-compose-restore.yml exec backup restic unlock -v
```

## References

- [Restic Documentation](https://restic.readthedocs.io/en/stable/)
- [mekomsolutions/restic-compose-backup](https://hub.docker.com/r/mekomsolutions/restic-compose-backup)

# Offline or Air-Gapped Installations

In situations where the instance running the project is not connected to the internet we provide pre-packaged images which can be loaded on the instance. To obtain the image check under the [Releases](https://github.com/path-drc/path-drc-emr/releases) section and download the `path-drc-emr-images-bundle.tgz` file.

To load the images on the instance, extract the `path-drc-emr-images-bundle.tgz` file and run the following script:

```sh
./load-images.sh
```
