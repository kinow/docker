# FDB

Use it until ECMWF publishes an official image. Feel free
to send pull requests or to create issues.

## Building

See the args and labels. Build could be performed with the
following command, for example:

```bash
IMAGE_VERSION="5.11.17-jammy"; docker build \
  --no-cache=true \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:SZ') \
  --build-arg AUTHOR="$(getent passwd $USER | cut -d ':' -f 5 | cut -d ',' -f 1)" \
  --build-arg IMAGE_VERSION="${IMAGE_VERSION}" \
  --tag "kinow/fdb:${IMAGE_VERSION}" --tag "kinow/fdb:latest" .

```

The image size is about 288MB, where the base image (debian)
has 74.8 MB, ecbuild 1.42MB, eccodes 55.4 MB, eckit 48.9 MB,
metkit 14MB, and FDB occupies 59.5 MB. The remaining space
is used by test data.
