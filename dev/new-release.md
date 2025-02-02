# Make a new release

## Update documents

- `README.md` in `build/distribution`. This file will be uploaded to SourceForge and Github Releases
- `README.html` in `protege/plugin`. This file will be displayed in the Protégé Plugin Update window.
- `update.properties` file in `protege/plugin`. This is the configuration file for Protégé plugin Auto Update

## Create a git branch for release

```console
$ git checkout <main development branch>
$ git checkout -b releasing/<version-major.minor>
```

## Update the version numbers

Update pom.xml files by Maven plugin to a SNAPSHOT version:

```console
$ ./mvnw versions:set -DnewVersion=5.0.0-SNAPSHOT
```

## Build Maven packages

```console
$ ./mvnw release:clean

# Preparing the release will sign artifacts (GPG key needed), create the new tag in git and automatically push to github.
# When 100% sure, you can skip the test by `-Darguments="-DskipTests"`
$ ./mvnw  --batch-mode release:prepare

# stage the release
$ ./mvnw release:perform

# Or stage from a Git tag
# ./mvnw release:perform  -DconnectionUrl=scm:git:git@github.com:ontop/ontop.git -Dtag=ontop-5.0.0
```

### Stage the Maven artifacts

- URL: [https://oss.sonatype.org](https://oss.sonatype.org)
- Navigate to "Staging Repositories" from the left panel
- Close the repository by clicking the `Close` button
- Release the artifacts by clicking the `Release` button

## Build and distribute Ontop bundles

```console
# we can start from the `checkout` directory generated during the Maven release
$ cd target/checkout
$ ./build-release.sh
```

Further info: [Build Instruction](/dev/build)

- Make a release in [Github Releases](https://github.com/ontop/ontop/releases)

- Deploy the packages to [SourceForge](https://sourceforge.net/projects/ontop4obda/files/) by manual upload or sftp.

```console
$ sftp user,ontop4obda@frs.sourceforge.net
$ cd /home/pfs/project/o/on/ontop4obda
# sftp://ghxiao,ontop4obda@frs.sourceforge.net/home/pfs/project/o/on/ontop4obda
```

## Update Docker Hub

- Build and push the docker image of [ontop/ontop](https://hub.docker.com/repository/docker/ontop/ontop) to Docker Hub using the script `client/docker/build-image.sh`
  - by default, the script tag the images as `ontop/ontop:[version]` and `ontop/ontop:latest` - use option `-t` (one or more times) to tag differently (e.g., `-t ontop/ontop:xyz -t ontop/ontop:latest`)
  - run `client/docker/build-image.sh --help` for available options

```console
# log in to Docker Hub
# docker login
$ cd ontop/client/docker/
# `-C`: multi-stage build. slow, but safe
# `-x`: compile for both amd64 and arm64, which entails pushing to Docker hub
$ ./build-image.sh -C -x
```

## Update the Ontop Website

- Update [the guide homepage](/guide)
- Update [the release notes](/guide/releases)

## Update the API example repository

- Update the Ontop version in <https://github.com/ontop/ontop-api-examples>

## Update branches for next development iteration

- Merge the tag for the release to master and the release branch to develop

```console
$ git checkout develop
$ git merge release/v-number #branch

# this step is also for triggering the AutoUpdate of Protege plugin
$ git checkout master
$ git merge v-number # tag
```

## Time for celebration!

- Send release emails to several mail lists and social media
