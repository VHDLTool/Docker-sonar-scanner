# Test documentation

The `tests/` folder contains both the tests scripts and some dummy projects to analyze.

## List of scripted integration tests

1. shell1
  * file: shell1.bash
  * purpose: Check that ShellCheck can be launched from within the container to analyze scripts in the project.
1. shell2
  * file: shell2.bash
  * purpose: Check that the sonar-scanner can analyze a shell project and sends its results to the server.
1. java
  * file: java.bash
  * purpose: Check that the java language is supported with plugins expected.

### How to run all the tests

Before testing the image, it must be built (see above).

To run the tests, the following tools are required:

* `curl`
* `jq`

To run all the tests, use the test script like this:

```sh
# from the root of the project
$ ./tests/run_tests.bash
```

## How to run a specific test

1. Create a docker bridge
  * ```sh
    $ docker network create sonarbridge
    ```
1. Export the environment variables the test needs
  * ```sh
    $ export SONARQUBE_CONTAINER_NAME=lequalsonarqube
    $ export SONARQUBE_ADMIN_PASSWORD=pass
    $ export SONARQUBE_NETWORK=sonarbridge
    $ export SONARQUBE_LOCAL_URL=http://localhost:9000
    $ export SONARQUBE_URL=http://${SONARQUBE_CONTAINER_NAME}:9000
    $ export SONARQUBE_TAG=latest
    ```
1. Run a container of the SonarQube server
  * ```sh
    docker run --name "$SONARQUBE_CONTAINER_NAME" \
            -d --rm \
            -p 9000:9000 \
            -e SONARQUBE_ADMIN_PASSWORD="$SONARQUBE_ADMIN_PASSWORD" \
            --net sonarbridge \
            lequal/sonarqube:latest
    ```
* Wait until it is configured
  * The message `[INFO] CNES SonarQube: ready!` is logged.
  * To see the logs of a container running in background
    ```sh
    $ docker container logs -f "$SONARQUBE_CONTAINER_NAME"
    Ctrl-C # once the container is ready
    ```
* Run a test script with 
  * ```sh
    $ ./tests/shell1.bash
    ```
* Test the exit status of the script with `echo $?`
  * zero => success
  * non-zero => failure

## List of options and environment variables used by the tests

Options:
* `--no-server-run`: if this option is specified, the script will not run a `lequal/sonarqube` container or create a bridge network. It will only launch the tests. In this case, make sur to set necessary environment variables.

Environment variables:
* `SONARQUBE_CONTAINER_NAME`: the name to give to the container running the `lequal/sonarqube` image.
* `SONARQUBE_ADMIN_PASSWORD`: the password of the admin account on the server.
* `SONARQUBE_URL`: URL of `lequal/sonarqube` container if already running without trailing `/` from the scanner container. e.g. http://mycontainer:9000 Use it only if no container name was given.
* `SONARQUBE_LOCAL_URL`: URL of `lequal/sonarqube` container if already running without trailing `/` from the host. e.g. http://localhost:9000
* `SONARQUBE_TAG`: the tag of the `lequal/sonarqube` image to use. e.g. latest
* `SONARQUBE_NETWORK`: the name of the docker bridge used.

## How to add a new test

Tests are just scripts.

To add a test:

1. Create a file under the `tests/` folder
1. Make it executable (with `chmod u+x tests/my_test.bash` for instance)
1. Edit the script.
1. To indicate wether the test has failed or succeed, use the exit status
    * zero => success
    * non-zero => failure
1. Add the test to the [list](#list-of-scripted-integration-tests)

Note that when using `./tests/run_tests.bash` to run the new test alongside the others, only messages on STDERR will by displayed if any.