---
title: Docker (5 seconds)
---

This document guides you to run Bytebase in docker, which takes less than 5 seconds.

## Prerequisites
Before starting, make sure you have installed [Docker](https://www.docker.com/get-started/). 

## Run on localhost:8080 or localhost:xxxx
Run the following command to start Bytebase locally on localhost:8080.
<pre>
docker run --init \
  --name bytebase \
  --restart always \
  --add-host host.docker.internal:host-gateway \
  --publish 8080:8080 \
  --volume ~/.bytebase/data:/var/opt/bytebase \
  bytebase/bytebase:<version></version> \
  --data /var/opt/bytebase \
  --host http://localhost \
  --port 8080
</pre>



You can change `8080` to `5678` or something else, however, make sure these three ports are the same:
- --publish {{hostport}}
- :{{containerport}} 
- --port {{port}}}

<pre>
docker run --init \
  --name bytebase \
  --restart always \
  --add-host host.docker.internal:host-gateway \
  --publish 5678:5678 \
  --volume ~/.bytebase/data:/var/opt/bytebase \
  bytebase/bytebase:<version></version> \
  --data /var/opt/bytebase \
  --host http://localhost \
  --port 5678
</pre>

Bytebase will store its data under `~/.bytebase/data` , you can reset all data by running command:
  
```
rm -rf ~/.bytebase/data
```

Check [Server Startup Options](./reference/command-line) for other startup options.

## Run on [https://bytebase.example.com](https://bytebase.example.com/)

Run the following command to start Bytebase on [https://bytebase.example.com](https://bytebase.example.com/)

<pre>
docker run --init \
  --name bytebase \
  --restart always \
  --add-host host.docker.internal:host-gateway \
  --publish 80:80 \
  --volume ~/.bytebase/data:/var/opt/bytebase \
  bytebase/bytebase:<version></version> \
  --data /var/opt/bytebase \
  --host https://bytebase.example.com \
  --port 80
</pre>

<hint-block type="info">

For production setup, you need to make sure [--host](/docs/reference/command-line#--host-string), [--port](/docs/reference/command-line#--port-number) match exactly to the host:port address where Bytebase supposed to be visited. Please check [Production Setup](/docs/operating/production-setup) for more advice.

</hint-block>

## Troubleshoot

Run the following if something goes wrong.
```
docker logs bytebase
```

Normally you should see this:

<pre>
-----Config BEGIN-----
mode=release
host=http://example.com
port=8080
dsn=file:/var/opt/bytebase/bytebase.db
seedDir=seed/release
readonly=false
demo=false
debug=false
-----Config END-------
2021-07-07T16:56:02.812Z        INFO    store/sqlite.go:213     Apply database migration if needed...
2021-07-07T16:56:02.821Z        INFO    store/sqlite.go:220     Current schema version before migration: 1.1
2021-07-07T16:56:02.821Z        INFO    store/sqlite.go:247     Skip this migration file: migration/10001__init_schema.sql. The corresponding migration version 1.1 has already been applied.
2021-07-07T16:56:02.828Z        INFO    store/sqlite.go:255     Current schema version after migration: 1.1
2021-07-07T16:56:02.828Z        INFO    store/sqlite.go:263     Completed database migration.

██████╗ ██╗   ██╗████████╗███████╗██████╗  █████╗ ███████╗███████╗
██╔══██╗╚██╗ ██╔╝╚══██╔══╝██╔════╝██╔══██╗██╔══██╗██╔════╝██╔════╝
██████╔╝ ╚████╔╝    ██║   █████╗  ██████╔╝███████║███████╗█████╗
██╔══██╗  ╚██╔╝     ██║   ██╔══╝  ██╔══██╗██╔══██║╚════██║██╔══╝
██████╔╝   ██║      ██║   ███████╗██████╔╝██║  ██║███████║███████╗
╚═════╝    ╚═╝      ╚═╝   ╚══════╝╚═════╝ ╚═╝  ╚═╝╚══════╝╚══════╝

Version <version></version> has started at http://example.com:8080
</pre>

### Unable to start Bytebase with Docker

#### Using [Colima](https://github.com/abiosoft/colima)

Due to the vm mechanism of colima, try to use the `--mount` option when starting colima as shown below:

```bash
mkdir ~/volumes
colima start --mount ~/volumes:w
docker run --init --name bytebase --restart always --add-host host.docker.internal:host-gateway --publish 8080:8080 --volume ~/.bytebase/data:/var/opt/bytebase bytebase/bytebase:1.1.0 --data /var/opt/bytebase --host http://localhost --port 8080
```