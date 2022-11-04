# Omniperf Deployment

```eval_rst
.. toctree::
   :glob:
   :maxdepth: 4
```

Omniperf is broken into two installation components:

1. **Omniperf Client-side (_Required_)**
   - Provides core application profiling capability
   - Allows collection of performance counters, filtering by IP block, dispatch, kernel, etc
   - CLI based analysis mode
   - Stand alone web interface for importing analysis metrics
2. **Omniperf Server-side (_Optional_)**
   - Mongo DB backend + Grafana instance

---

## Client-side Installation

Omniperf requires the following basic software dependencies prior to usage:

* Python (>=3.7)
* CMake (>= 3.19)
* ROCm (>= 5.1)

In addition, Omniperf leverages a number of Python packages that are
documented in the top-level `requirements.txt` file.  These must be
installed prior to Omniperf configuration.  

The recommended procedure for Omniperf usage is to install into a shared file system so that multiple users can access the final installation.  The following steps illustrate how to install the necessary python dependencies using [pip](https://packaging.python.org/en/latest/) and Omniperf into a shared location controlled by the `INSTALL_DIR` environment variable.

```{admonition} Configuration variables
The following installation example leverages several
[CMake](https://cmake.org/cmake/help/latest/) project variables
defined as follows:
| Variable             | Description                                                          |
| -------------------- | -------------------------------------------------------------------- |
| CMAKE_INSTALL_PREFIX | controls install path for Omniperf files                             |
| PYTHON_DEPS          | provides optional path to resolve Python package dependencies        |
| MOD_INSTALL_PATH     | provides optional path for separate Omniperf modulefile installation |
```

```shell
# define top-level install path
$ export INSTALL_DIR=/opt/apps/omniperf

# install python deps
$ python3 -m pip install -t ${INSTALL_DIR}/python-libs -r requirements.txt

# configure Omniperf for shared install
$ mkdir build
$ cd build
$ cmake -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}/{__VERSION__} \
        -DPYTHON_DEPS=${INSTALL_DIR}/python-libs \
        -DMOD_INSTALL_PATH=${INSTALL_DIR}/modulefiles ..

# install
$ make install
```



After completing these steps, a successful top-level installation directory looks as follows:
```shell
$ ls $INSTALL_DIR
modulefiles  {__VERSION__}  python-libs
```

The installation process includes creation of an environment
modulefile for use with [Lmod](https://lmod.readthedocs.io). On
systems that support Lmod, a user can register the Omniperf modulefile
directory and setup their environment for execution of Omniperf as
follows:



```shell
$ module use $INSTALL_DIR/modulefiles
$ module load omniperf
$ which omniperf
/opt/apps/omniperf/{__VERSION__}/bin/omniperf

$ omniperf --version
ROC Profiler:   /opt/rocm-5.1.0/bin/rocprof

omniperf (v{__VERSION__})
```

```{tip} Sites relying on an Lmod Python module locally may wish to
customize the resulting Omniperf modulefile post-installation to
include additional module dependencies.
```

NOTE: The optional `ROCPROF` environment variable can be used to define a
custom rocprof release. If not set, the default rocprof tool detected will be
used to do profiling.




%%% ### Generate Packaging
%%% ```console
%%% cd build
%%% cpack -G STGZ
%%% cpack -G DEB -D CPACK_PACKAGING_INSTALL_PREFIX=/opt/omniperf
%%% cpack -G RPM -D CPACK_PACKAGING_INSTALL_PREFIX=/opt/omniperf
%%% ```

---

## Omniperf Server Setup

Note: Server-side setup is not required to profile or analyze
performance data from the CLI. It is provided as an additional mechanism to import performance
data for examination within a detailed [Grafana GUI](https://github.com/grafana/grafana).

The recommended process for enabling the server-side of Omniperf is to
use the provided Docker file to build the Grafana and MongoDB
instance.

### Persist Storage
```bash
$ sudo mkdir -p /usr/local/persist && cd /usr/local/persist/
$ sudo mkdir -p grafana-storage mongodb
$ sudo mkdir -p grafana-storage mongodb
```

### Build and Launch
```bash
$ sudo docker-compose build
$ sudo docker-compose up
```