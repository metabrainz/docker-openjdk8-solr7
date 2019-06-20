## Updating the docker-solr repository

First, we need to modify our https://github.com/metabrainz/docker-openjdk8-solr7 repository for the new version.
To do that, you need a Linux host that runs Docker, and has `git`, `wget` and `gpg` installed.

First, get the repository:

```bash
git clone git@github.com:metabrainz/docker-openjdk8-solr7.git

cd docker-solr
```

If you're in Europe, you can override the download file locations for much faster downloads:

```bash
export SOLR_DOWNLOAD_SERVER="http://www-eu.apache.org/dist/lucene/solr"
export archiveUrl="https://www-eu.apache.org/dist/lucene/solr"
```

Run the script that creates a directory for the new version, downloads solr to checksum, and creates a Dockerfile:

```bash
tools/update.sh
git status
```

## Test the new Dockerfile

To test the Dockerfile locally:

```bash
tools/build_all.sh
```

Keep an eye out for "This key is not certified with a trusted signature!"; it would be good to verify the fingerprints with ones you have in your PGP keyring.

There are two scripts directories: `./scripts`, used by `solr:8` which uses the Solr installer, and `./scripts-before8` for older versions which were installed by simply untarring the distribution.
When changing scripts in one of these, remember to review the other directory to see if the same changes apply there.

To run simple automated tests against the images:

```bash
tools/test_all.sh
```

To manually test a container:

```bash
docker container run --name solr-test -d -p 98983:8983 metabrainz/solr:latest solr-demo
```

Check the logs for startup messages:

```bash
docker container logs solr-test
```

Get the URL of the Solr running on it:

```bash
echo "http://localhost:$(docker port solr-test 8983/tcp| sed 's/^.*://')/"
```

and check that URL in your browser, paying particular attention to the `solr-impl` in the administration interface, which lists the Solr version.
Check for errors under "Logging".

If that looks in order, then clean up the container:

```bash
docker container kill solr-test
docker container rm solr-test
```

and remove our local images:

```bash
docker image list metabrainz/solr | awk '{print $1":"$2}' | xargs -n 1 docker image rm

```

## Commit changes to our local repository

Now we can commit the changes to our repository.

Check in the changes:

```bash
git status
git add 6.6
git add -A
git commit -m "Add Solr 6.6.0"
git push
git rev-parse HEAD
```

Make note of that git SHA.

## Update our README

Our repository has a README https://github.com/metabrainz/docker-openjdk8-solr7/blob/master/README.md which shows
supported tags. This is there for the convenience of
our users to update this section, run:

```
tools/update_readme.sh
git diff README.md
```

Then commit and push that change:

```bash
git commit -m "Update README for Solr 6.6.0" README.md
git push
```

That is our repository updated.

## Check the automated build

The check-in will trigger an automated build on https://travis-ci.org/metabrainz/docker-openjdk8-solr7.
Verify that that succeeds.

That's it!
