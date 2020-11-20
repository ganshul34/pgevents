# fyle-pgevents

Utility to generate events from PG using logical replication and push it to Rabbitmq.

# Build

Easiest way is to use docker.

```
docker build -t fyle_pgevents .
```

# Pre-requisites

To get the logical decoding working correctly, you'll need to run set replica identity to full for the tables in question. Example:

```
alter table transactions replica identity full;
```

If you don't do this, the old value will not be sent during updates and deletes and the diffs may be incorrect.

# Usage

Set the following environment variables to connect to PostgreSQL >= 10 and to RabbitMQ as broker

```
export PGHOST=xxx
export PGPORT=5432
export PGDATABASE=test
export PGUSER=postgres
export PGPASSWORD=xxx
export PGSLOT=test_slot
export PGTABLES=public.transactions,public.reports

export RABBITMQ_URL=yyy
export RABBITMQ_EXCHANGE=table_exchange
export RABBITMQ_QUEUE_NAME=audit

```

Note that if you're running on Docker on Mac and want to connect to host machine, then set PGHOST to host.docker.internal and not localhost or 127.0.0.1.


To read from PostgreSQL and send data to rabbitmq
```
docker run -i -e PGHOST -e PGPORT -e PGDATABASE -e PGUSER -e PGPASSWORD -e PGSLOT -e RABBITMQ_URL -e RABBITMQ_EXCHANGE --rm fyle_pgevents pgevent_producer
```

To read data from rabbitmq exchange and print it to stdout
```
docker run -i -e RABBITMQ_URL -e RABBITMQ_EXCHANGE -e RABBITMQ_QUEUE_NAME --rm fyle_pgevents pgevent_consumer_debug
```

For detailed information, use the help flag

```
$ docker run -i -e PGHOST -e PGPORT -e PGDATABASE -e PGUSER -e PGPASSWORD -e PGSLOT --rm fyle_pgevents pgevent_producer --help
Usage: pgevent_producer [OPTIONS]

Options:
  --pghost TEXT             Postgresql Host ($PGHOST)  [required]
  --pgport TEXT             Postgresql Host ($PGPORT)  [required]
  --pgdatabase TEXT         Postgresql Database ($PGDATABASE)  [required]
  --pguser TEXT             Postgresql User ($PGUSER)  [required]
  --pgpassword TEXT         Postgresql Password ($PGPASSWORD)  [required]
  --pgslot TEXT             Postgresql Replication Slot Name ($PGSLOT)
                            [required]
  --pgtables TEXT           Restrict to specific tables e.g.
                            public.transactions,public.reports
  --rabbitmq-url TEXT       RabbitMQ url ($RABBITMQ_URL)  [required]
  --rabbitmq-exchange TEXT  RabbitMQ exchange ($RABBITMQ_EXCHANGE)  [required]
  --help                    Show this message and exit.

```

# Development

Map the volume to the docker container and run the utility from within the container while you're making changes in the editor:

```
docker run -it -e PGHOST -e PGPORT -e PGDATABASE -e PGUSER -e PGPASSWORD -e PGSLOT -e PGTABLES -e RABBITMQ_URL -e RABBITMQ_EXCHANGE -e RABBITMQ_QUEUE_NAME --rm -v $(pwd):/fyle_pgevents --entrypoint=/bin/bash fyle_pgevents
```

Now make changes to the python files. Then run the command from shell:

```
python pgevent_producer.py
```

