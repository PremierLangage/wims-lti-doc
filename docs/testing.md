# Testing

## Continuous testing

Tests are executed weekly on *Github Actions* against Python 3.6, 3.7, 3.8 and 3.9 and the latest
version of **WIMS**. You can find these
tests [here](https://github.com/PremierLangage/wims-lti/actions).

## Running the tests locally

### A running **WIMS** instance

To run the tests of **WIMS-LTI**, you will need a running **WIMS** instance. Since classes will be
created with fixed ID, it is recommended to use a newly-created instance for each run of the test
suite.

> ***/!\ Never run the tests against a **WIMS** production server as the tests can mess-up with its database.***

To facilitate the installation (and reload between each run) of a new **WIMS** instance, a *docker*
image with a minimal installation has been
created : [qcoumes/wims-minimal](https://hub.docker.com/repository/docker/qcoumes/wims-minimal).

You can install it and run a docker instance on `localhost:7777` using the following commands :

```shell
docker run -itd --cpuset-cpus=$(($(cat /proc/cpuinfo | grep -e "processor\s*:\s*\d*" | wc -l) - 1)) -p 7777:80 --name wims-minimal qcoumes/wims-minimal
docker exec -i wims-minimal ./bin/apache-config
docker exec -i wims-minimal service apache2 restart
```

To reset the **WIMS** instance (assuming you didn't change the `--name` argument), use the following
commands :

```shell
docker container rm wims-minimal -f
docker run -itd --cpuset-cpus=$(($(cat /proc/cpuinfo | grep -e "processor\s*:\s*\d*" | wc -l) - 1)) -p 7777:80 --name wims-minimal qcoumes/wims-minimal
docker exec -it wims-minimal ./bin/apache-config
docker exec -it wims-minimal service apache2 restart
```

> ***/!\ Do not forget to reset your WIMS instance between each run of the test suite.***

### Using `django manage.py`

With a **WIMS** instance running, can now run the tests in two ways, the first one being
using `django manage.py`.

This will execute the tests against **your current environment**, so you need to have installed the
dependencies beforehand.

To run the tests, simply use the command `django manage.py test` at the root of the project.

### Using `tox`

`tox` create a python environment in which the dependencies from `requirements.txt` will be
automatically installed before running the tests inside said environment.

`tox` is not listed as a dependency for the project, so you will need to install it using your
package manager or `pip`: `pip3 install tox`.

To run the tests using `tox` use the commands `python3 -m tox` at the root of the project.

In the unlikely case you have multiple versions of python 3 installed, you may need to tell tox
which one to use to avoid the tests running on both version (and thus failing due to the **WIMS**
instance not being restarted in between). For that, use the option `-e py[version]` where version is
the concatenation of the major and minor number of the version you want to use,
e.g. `python3 -m tox -e py39`
for python 3.9.


## Additional information

### WIMS_URL

The default URL used by the tests to communicate with **WIMS**
is `http://localhost:7777/wims/wims.cgi`, but you can change it by setting the *environment
variable* `WIMS_URL` :

```shell
WIMS_URL=http://my_wims.server/wims/wims.cgi python3 manage.py test
```

or

```shell
WIMS_URL=http://my_wims.server/wims/wims.cgi python3 -m tox
```


### Test failing unexpectedly

For some unknown reason, the communication with the **WIMS** server can fail sometime
