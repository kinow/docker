# FDB

Use it until ECMWF publishes an official image. Feel free
to send pull requests or to create issues.

## Building

See the args and labels. Build could be performed with the
following command, for example:

```bash
IMAGE_VERSION="5.11.10-bookworm-slim"; docker build \
  --no-cache=true \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:SZ') \
  --build-arg AUTHOR="$(getent passwd $USER | cut -d ':' -f 5 | cut -d ',' -f 1)" \
  --build-arg IMAGE_VERSION="${IMAGE_VERSION}" \
  --tag "ces/tests-eric:${IMAGE_VERSION}" .

```

