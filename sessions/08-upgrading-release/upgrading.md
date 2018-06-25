layout: true
class: inverse, middle, large

---
class: special
# Upgrading Galaxy
Tracking releases

slides by @martenson, @afgane, @nsoranzo

.footnote[\#usegalaxy / @galaxyproject]

---
# Release Cycle

* Galaxy aims to release each 4 months
* Releases are tagged with year and month, for example 17.09, 18.01
* Per the [security policy](https://github.com/galaxyproject/galaxy/blob/dev/SECURITY_POLICY.md), releases within the past 12 months are supported

---
# Releases

* Every release has [release notes](https://docs.galaxyproject.org/en/master/releases)
  * We put substantial effort in making them as useful as we can
  * Highlights, security announcements, deprecation notices, enhancements grouped by impact, etc...
* Every release has its own branch in the `galaxy` GitHub repository
  * Named as `release_YY_MM`
  * Kept up to date (especially for recent releases)

---
# Mantain local Galaxy modifications

Two options:
1. `git stash && git ... && git stash pop`
2. Commit your changes to a local branch, merge/rebase upstream Galaxy

The latter is probably better than the former for large changes

---
# Another possibility

If you want to keep your configurations and other local changes under version control, you can move these files/folders outside of Galaxy and into their own repository. You can then use symbolic links to place the files where Galaxy expects them.

---
# Keeping a release up to date

To keep your `release_*` branch up to date you can:

```console
$ git stash  # optional
$ git pull --ff-only
$ git stash pop  # optional
```

and restart Galaxy

Works well if no local commits exist

---
# Major release upgrade

When a new release is out:
* Plan a service downtime for the upgrade and inform your users
* Configure your reverse proxy to serve a custom error page, e.g. for nginx inside the `server` section add:
  ```ini
  error_page 502 /static/custom_502.html;
  ```
* When it's time, stop the Galaxy server processes (not the database server!)

---
# Housekeeping

Not usually necessary, but might help:

```console
$ find . -name '*.pyc' -delete
$ rm -rf database/compiled_templates/*
```

---
# Upgrading the galaxy repo

```console
$ git fetch
$ git stash  # optional
$ git checkout release_YY.MM
$ git pull --ff-only
$ git stash pop  # optional
```

---
# Diff samples

```console
$ diff -u galaxy.yml galaxy.yml.sample
$ diff -u datatypes_conf.xml datatypes_xml.conf.sample
```

Merge changes as desired/necessary.

---
# Diff samples

Alternatively:

```console
$ git diff release_17.09..release_18.01 -- config/galaxy.yml.sample
```

---
# Upgrade virtualenv

```console
$ export GALAXY_CONFIG_FILE=/srv/galaxy/config/galaxy.yml
$ export GALAXY_VIRTUAL_ENV=/srv/galaxy/venv
$ . $GALAXY_VIRTUAL_ENV/bin/activate
$ pip install --upgrade pip setuptools
$ ./scripts/common_startup.sh
$ deactivate
```

---
# make client on `dev`

Since January 2018, the `dev` Galaxy branch does not contain updated client
build artifacts (e.g. JavaScript bundles in `static/`).

- run `make client`

---
# Tool migrations?

Galaxy source tools -> Tool Shed

We haven't done these in a long time, but may do more

Galaxy will notify you on first startup after upgrade including migration

---
# Database migrations

1. **Backup** your database
   - [PostgreSQL backup docs](https://www.postgresql.org/docs/current/static/backup.html)
2. Run
   ```console
   $ sh manage_db.sh upgrade -c /srv/galaxy/config/galaxy.ini
   ```

---
# Start Galaxy

* Monitor the log files
* Check `/api/version`
* Check that everything still works

---
# Distribute Galaxy

If you're using a compute cluster and not running from shared file system

---
# Downgrading

If for some reason you need to move back to an older release, the process is similar, but the
order is a bit different:

```console
$ export GALAXY_CONFIG_FILE=/srv/galaxy/config/galaxy.yml
$ export GALAXY_VIRTUAL_ENV=/srv/galaxy/venv
$ sh manage_db.sh -c /srv/galaxy/config/galaxy.yml downgrade 135
$ find . -name '*.pyc' -delete
$ git fetch
$ git stash  # optional
$ git checkout release_17.09
$ git pull --ff-only
$ git stash pop  # optional
$ ./scripts/common_startup.sh
```

.footnote[Here we assume you know the database version of the Galaxy release you want to
downgrade to (135 in the example above). This simplifies the procedure because
the database downgrade needs to be done while being on the newer branch.]
