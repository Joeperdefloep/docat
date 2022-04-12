![docat](doc/assets/docat-teaser.png)

**Host your docs. Simple. Versioned. Fancy.**

[![build](https://github.com/docat-org/docat/workflows/docat%20ci/badge.svg)](https://github.com/docat-org/docat/actions)
[![Gitter](https://badges.gitter.im/docat-docs-hosting/community.svg)](https://gitter.im/docat-docs-hosting/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

## Getting started

The simplest way is to build and run the docker container,
you can optionally use volumes to save state:

```sh
# run container in background and persist data (docs, nginx configs)
# use 'ghcr.io/docat-org/docat:unstable' to get the latest changes
mkdir -p docat-run/db && touch docat-run/db/db.json
docker run \
  --detach \
  --volume $PWD/docat-run/doc:/var/docat/doc/ \
  --volume $PWD/docat-run/locations:/etc/nginx/locations.d/ \
  --volume $PWD/docat-run/db/db.json:/app/docat/db.json \
  --publish 8000:80 \
  ghcr.io/docat-org/docat
```

Go to [localhost:8000](http://localhost:8000) to view your docat instance:

![docat screenshot](doc/assets/docat-screenshot.png)

If you want to run the application different than in a docker container, look at the
[backend](docat/README.md) and [web](web/README.md) docs.

### Push documentation to docat

The preferred way to push documentation to a docat server is using the [docatl](https://github.com/docat-org/docatl)
command line application:

```sh
docatl push --host http://localhost:8000 /path/to/your/docs PROJECT VERSION
```

There are also docker images available for CI systems.

#### Using standard UNIX command line tools

If you have static html documentation or use something like
[mkdocs](https://www.mkdocs.org/), [sphinx](http://www.sphinx-doc.org/en/master/), ...
to generate your documentation, you can push it to docat:

```sh
# create a zip of your docs
zip -r docs.zip /path/to/your-docs
# upload them to the docat server (replace PROJECT/VERSION with your projectname and the version of the docs)
curl -X POST -F "file=@docs.zip" http://localhost:8000/api/PROJECT/VERSION
```

When you have multiple versions you may want to tag some version as **latest**:

```sh
# tag the version VERSION of project PROJECT as latest
curl -X PUT http://localhost:8000/api/PROJECT/VERSION/tags/latest
```


## Advanced config.json

It is possible to configure some things after the fact.

1. Create a `config.json` file
2. Mount it inside your docker container `--volume /path/to/config.json:/var/www/html/config.json`

Supported config options:

* headerHTML

## Securing your API endpoints

For securing your API, so that only authenticated users can manage projects, add
the following lines to `/etc/nginx/default.conf`:
```
    location /api {
        client_max_body_size 100M;
        proxy_pass http://python_backend;
        auth_basic "restricted";
        auth_basic_user_file /app/docat/.htpasswd; 
    }
```

Then, copy your file you created with `htpasswd` and users to
/app/docat/.htpasswd: 

```
docker cp /path/to/local/.htpasswd /app/docat/.htpasswd
```
For more information, see [the Nginx documentation](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/)