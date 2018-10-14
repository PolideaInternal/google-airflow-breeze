# Composer Airflow Breeze Container

It's a breeze to have Airflow running for GCP-related development.

## About

The Composer Airflow Breeze Container allows you to easily create development
environment to work on apache/incubator-airflow repository and test your changes
interfacing with GCP without going through the overhead of manually setting 
up an Airflow environment.

It allows you to have multiple contribution workspaces simultaneously, storing
them in subdirectories of its base directory.

It also allows you to share common configuration 

## Intended Usage

-   Make updates within the incubator-airflow folder (preferably outside of the
    container).
-   Test your updates within the container (by running a bash session within the
    container or using the `-t` flag).


## Running

To run the container, use run_environment.sh. This will create a workspace with
the name "default".

`./run_environment.sh -a PROJECT_ID`

If you want to use a different workspace, use the -w flag:

`./run_environment.sh -a PROJECT_ID -w dataproc_workspace`

If you want to forward a port for using the webserver, use the -p flag:

`./run_environment.sh -a PROJECT_ID -w another_workspace -p 8080`

For a full list of commands supported, use -h:

`./run_environment.sh -h`

### Run an Example Dag

-   Use run_environment.sh to run a container with the port forwarded to 8080.
-   (optional) Setup a tmux session so that you can have multiple terminals
    within your container.
-   Start the airflow db: `airflow initdb`
-   Start the webserver: `airflow webserver`
-   View the Airflow webapp at `http://localhost:8080/`
-   Start the scheduler in a separate terminal (make sure the separate terminal
    is still in the container; tmux will help here): `airflow scheduler`
-   Copy an example dag into the DAGs folder: `cp
    /home/airflow/incubator-airflow/airflow/example_dags/tutorial.py
    /home/airflow/dags`
-   It may take up to 5 minutes for the scheduler to notice the new DAG. Restart
    the scheduler manually to speed this up.
-   Enable the DAG once it appears in the webapp.

## Run Upstream Unit Tests

Ensure that you have set up TravisCI on your fork of the Airflow GitHub repo
before beginning your contribution. Before making any local changes, determine
the tests relevant to your change and run them to ensure they pass. Due to the
complexity of managing a full Airflow environment, some test cases are not
currently supported.
Note that some cases
will be particularly difficult to support such as any test cases requiring an
SSH connection.

After making your changes, use the contribution environment to test them. Any
changes you make to the Airflow source files should be immediately available in
the environment. When you finish making changes, ensure that the TravisCI build
passes before making a pull request, even if the tests pass within the
environment. The environment currently only supports Python 2.7, but tests are
also run on Python 3.4.

In the examples below, note that testing arguments mirror the directory of the
tests folder in the upstream Airflow repo. Tests are gathered recursively if you
specify a directory, so if you wish to run all tests, you could specify "tests"
as the argument.

### Run with run_environment.sh

You can use run_environment.sh to start a container, run a test, and delete the
container using the -t flag.

#### Core Tests

`./run_environment.sh -t tests.core:CoreTest`

#### GCP Dataproc Operator Tests

`./run_environment.sh -t tests.contrib.operators.test_dataproc_operator`

### Run within a bash session

If you are working within the container, you may use the following commands to
run tests.

#### Core Tests

`./run_unit_tests.sh tests.core:CoreTest -s --logging-level=DEBUG`

#### GCP Dataproc Operator Tests

`./run_unit_tests.sh tests.contrib.operators.test_dataproc_operator -s
--logging-level=DEBUG`

## Using a Service Account

If you would like to interact with a GCP project, you may authenticate with
service account credentials using the json secret key generated with the
service account. To do this, you must:
1. Create a service account in the project you wish to use with all permissions
   needed.
1. Download the service account's key and place it in the directory "keys"
   directly under run_environment.sh as the file "key.json".

If you followed the steps correctly, then your directory structure may include:
- upstream/run_environment.sh
- upstream/key/key.json

Once you have completed these steps, your key will automatically be put into
your environment when the environment is run, and the service account will
be logged in for usage.

## Cleanup

If you are done using this tool, remember to delete the image it generated (it
takes up about 2 gigabytes!) Ensure you do not have any open workspaces, then
use `./run_environment.sh -c` to delete the image. You may also delete the
workspace folders generated by this tool.
