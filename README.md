# airflow-pipeline [![Build Status](https://travis-ci.org/datagovsg/airflow-pipeline.svg?branch=)](https://travis-ci.org/datagovsg/airflow-pipeline)

An [Airflow](https://airflow.incubator.apache.org/) setup that aims to work well with Hadoop and Spark


## What this gives you

This docker image is based off [`datagovsg/python-spark-airflow:1.7`](https://hub.docker.com/r/datagovsg/python-spark-airflow/) image which is based off [`datagovsg/python-spark:2.7-2.0`](https://hub.docker.com/r/datagovsg/python-spark/). See their respective docker files to know where they are installed.

- Python 2.7
- Spark 2.0
- Hadoop 2.7
- Sqoop 1.4.6 (with JDBC connectors for PostgreSQL, MySQL and SQL Server)
- Airflow 1.7 (with PostgreSQL 9.5)


## Authentication

Password authentication is enabled as a security mechanism for administering Airflow via its admin UI.

Set `AIRFLOW_USER`, `AIRFLOW_EMAIL` and `AIRFLOW_PASSWORD` under `webserver` service in the docker compose file before starting the container.

Every time the airflow web server starts, it will create the user if it does not exist.


## Hadoop configuration

#### Hadoop user and group

Since the docker user is `afpuser` and group is `hadoop`, see [Dockerfile](Dockerfile). Therefore, your Hadoop admin should also add the same user and group to your hadoop cluster. Also grant HDFS permissions on `PIPELINE_DATA_PATH` e.g. /datasets/hadoop

#### Hadoop client configuration files

To write to HDFS and connect to the YARN ResourceManager, the (client side) configuration files for the Hadoop cluster must be added.

Obtain from your Hadoop administrator and place in `./hadoop*` directory. Note the environment variables that might be overwritten. e.g. Overriding `HADOOP_MAPRED_HOME` in `hadoop-env.sh`

The configuration contained in this directory will be distributed to the YARN cluster so that all containers used by the application use the same configuration.

See also http://spark.apache.org/docs/latest/running-on-yarn.html


## Docker image

You might have to `docker login` first before you can build any images.

To change the environment variables, edit `docker-compose.yml` instead of `Dockerfile` without the need to rebuild the docker image.

To start, use only the `docker-compose.yml` file i.e. `docker-compose -p afp -f docker-compose.yml up --build -d` 

To bring the containers up for development, use also the `docker-compose.override.yml`. This will additionally create a volume at `./dags` and mounted in the container at `/airflow/dags`, allowing you to do edit the DAG files directly on your development machine and having them updated with the container immediately.


## Deploying into production

Since credentials are managed as environment variables, it is recommended that your env file or `docker-compose.production.yml` be stored securely. Do not commit them to any source code repositories.

In the given `docker-compose.yml`, the environment variables used to store credentials are placeholders only.


## Tests

To run tests, use `docker-compose -f docker-compose.test.yml up --build`


## Logs

To follow docker logs, use `docker-compose -f docker-compose.yml logs --tail=10 -f`


## Accessing the docker container
1. Ensure that container is deployed on your server
2. SSH into server
3. Access the container's bash shell: `docker exec -ti afp_airflow_1 bash`
4. Change user to afpuser: `gosu afpuser bash`

#### To backfill DAGs
- Run the airflow backfill: `airflow backfill -s <YYYY-MM-DD> -e <YYYY-MM-DD> <dag_name>`

#### Manually execute a DAG
- Run the airflow DAG: `airflow trigger_dag <dag_name>`
