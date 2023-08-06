## Build

```bash
$ docker build \
  -t kinow/autosubmit:4.0.84-bullseye-slim \
  -t kinow/autosubmit:latest \
  .
```

## Run

This image is useful for testing, or getting started with Autosubmit.
For a production installation, please consult the Autosubmit documentation.

The examples in this section show different ways to use this image.
For running experiments, SSH keys will have to be configured so that
the container running this image can SSH locally, and also to a remote
server (if there are remote jobs).

There is also a `docker-compose.yml` file that builds a complete
client and server cluster for Autosubmit. The `autosubmit` node
can be used to submit jobs to the other nodes (as it is normally
configured with VM's submitting jobs to remote Slurm, PBS, etc.).

```bash
$ docker run --rm kinow/autosubmit:latest \
  autosubmit --version
```

### Use volumes

Create the directories to hold DB and experiments somewhere.

```bash
$ mkdir -pv /tmp/autosubmit/{database,experiments}
```

Create an external DB, for example:

```bash
$ docker run --rm \
  -v /tmp/autosubmit/database:/app/autosubmit/database \
  -v /tmp/autosubmit/experiments:/app/autosubmit/experiments \
  kinow/autosubmit:latest \
  autosubmit install
```

Create a dummy experiment:

```bash
$ docker run --rm \
  -v /tmp/autosubmit/database:/app/autosubmit/database \
  -v /tmp/autosubmit/experiments:/app/autosubmit/experiments \
  kinow/autosubmit:latest \
  autosubmit expid -H local -d test --dummy
```

Confirm any container created with the image can access the experiments:

```bash
$ docker run --rm \
  -v /tmp/autosubmit/database:/app/autosubmit/database \
  -v /tmp/autosubmit/experiments:/app/autosubmit/experiments \
  kinow/autosubmit:latest \
  autosubmit describe
```

To delete an experiment (-ti if you do not pass `-f`):

```bash
$ docker run --rm \
  -ti \
  -v /tmp/autosubmit/database:/app/autosubmit/database \
  -v /tmp/autosubmit/experiments:/app/autosubmit/experiments \
  kinow/autosubmit:latest \
  autosubmit delete -f a000
```

### Run experiments

In order to run experiments, Autosubmit will need valid
SSH configuration so that it can submit the jobs to local
or remote hosts (always via `ssh`.)

To start with SSH:

```bash
$ docker run --rm \
  -ti \
  -p 2222:22 \
  -v $(pwd -P)/authorized_keys:/home/autosubmit/.ssh/authorized_keys \
  kinow/autosubmit:latest /bin/bash
```

### Docker Compose Example

First you will need valid SSH keys to use within the Docker Compose
cluster. The linuxserver.io image, used for the computing nodes,
provides a keygen utility.

```bash
$ docker run --rm -it --entrypoint /keygen.sh linuxserver/openssh-server
```

Copy the generated private key into `id_rsa` and the public key
into `id_rsa.pub`. Also copy the public key into `authorized_keys`.

You **must** have these three files created before running `docker compose`,
`id_rsa.pub`, `id_rsa`, and `authorized_keys`.

Remember to set the correct permissions too, e.g.

```bash
$ chmod 600 id_rsa*
$ chmod 600 authorized_keys
```

Start the cluster:

```bash
$ docker compose up
```

You can also increase the number of computing nodes, as in this example
where two are created.

```bash
$ docker compose up --force-recreate --build --scale computing_node=2
```

List the ports:

```bash
$ docker compose ps
NAME                           IMAGE                                       COMMAND                  SERVICE             CREATED             STATUS              PORTS
autosubmit                     dockerfiles-autosubmit                      "sudo /usr/sbin/sshd…"   autosubmit          33 minutes ago      Up 32 minutes       0.0.0.0:32783->22/tcp, :::32783->22/tcp
dockerfiles-computing_node-1   lscr.io/linuxserver/openssh-server:latest   "/init"                  computing_node      33 minutes ago      Up 32 minutes       2222/tcp, 0.0.0.0:32782->22/tcp, :::32782->22/tcp
dockerfiles-computing_node-2   lscr.io/linuxserver/openssh-server:latest   "/init"                  computing_node      33 minutes ago      Up 32 minutes       2222/tcp, 0.0.0.0:32781->22/tcp, :::32781->22/tcp
```

You can connect to the Autosubmit VM via Docker or SSH. The example below
uses Docker. Autosubmit will be loaded automatically via the Micromamba
Conda environment:

```bash
$ docker exec -ti dockerfiles-autosubmit-1 /bin/bash
```

For SSH, you have to use the randomly generated local TCP port that
is bound to the port `22` inside the container. Looking at the example above,
you could use:

```bash
$ ssh -i id_rsa -p 32791 autosubmit@localhost
Warning: Permanently added '[localhost]:32791' (ED25519) to the list of known hosts.
X11 forwarding request failed on channel 0
Linux b35e552f22bc 5.15.0-78-generic #85-Ubuntu SMP Fri Jul 7 15:25:09 UTC 2023 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Aug  6 09:45:28 2023 from 172.21.0.1
```

From the Autosubmit container, you should be able to connect to the
computing nodes.

```bash
$ ssh autosubmit@dockerfiles-computing_node-1
Welcome to OpenSSH Server
5a29566b9914:~$ 
logout
Connection to dockerfiles-computing_node-1 closed.
$ ssh autosubmit@dockerfiles-computing_node-2
Welcome to OpenSSH Server
cff117cd9a67:~$ 
logout
Connection to dockerfiles-computing_node-2 closed.
```
