# PostgreSQL client image

The [Dockerfile](Dockerfile) installs PostgreSQL client tools on Alpine 3.20.2. Use `psql` from the image to connect to an existing PostgreSQL server.

From the repository root, build the image and check the installed client version.

```sh
docker build -t borneo-psql:local ./containers/psql
docker run --rm borneo-psql:local psql --version
```
