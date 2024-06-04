
# 1. The starting point
The fundamental difference between pip and Conda packaging is what they put in packages.
-   Pip packages are Python libraries like NumPy or `matplotlib`.
-   Conda packages include Python libraries (NumPy or `matplotlib`), C libraries (`libjpeg`), and executables (like C compilers, and even the Python interpreter itself).

## Pip: Python libraries only
For example, let’s say you want to install Python 3.9 with NumPy, Pandas, and the gnuplot rendering tool, a tool that is unrelated to Python. Here’s what the pip `requirements.txt` would look like:
```text
numpy
pandas
```

Installing Python and gnuplot is _out of scope_ for pip. You as a user must deal with this yourself. You might, for example, do so with a Docker image:
```dockerFile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y gnuplot python3.9
COPY requirements.txt .
RUN pip install -r requirements.txt
```
Both the Python interpreter and gnuplot need to come from system packages, in this case Ubuntu’s packages.


## Conda: Any dependency can be a Conda package (almost)
With Conda, Python and gnuplot are just more Conda packages, no different than NumPy or Pandas. The `environment.yml` that corresponds (somewhat) to the `requirements.txt` we saw above will include all of these packages:
```yml
name: myenv
channels:
  - conda-forge
dependencies:
  - python=3.9
  - numpy
  - pandas
  - gnuplot
```
Conda only relies on the operating system for basic facilities, like the standard C library. Everything above that is Conda packages, not system packages.
We can see the difference if the corresponding `Dockerfile`; there is no need to install any system packages:
```dockerFile
FROM continuumio/miniconda3
COPY environment.yml .
RUN conda env create
```
This base image ships with Conda pre-installed, but we’re not relying on any existing Python install, we’re installing a new one in the new environment.


## Why Conda packages everything?
Why did Conda make the decision to package everything, Python interpreter included? How does this benefit you? In part it’s about portability and reproducibility.
1.  **Portability across operating systems:** Instead of installing Python in three different ways on Linux, macOS, and Windows, you can use the same `environment.yml` on all three.
2.  **Reproducibility:** It’s possible to pin almost the whole stack, from the Python interpreter upwards.
3.  **Consistent configuration:** You don’t need to install system packages and Python packages in two different ways; (almost) everything can go in one file, the `environment.yml`.

But it also addresses another problem: how to deal with Python libraries that require compiled code.

# 2. Beyond pure Python: Packaging compiled extensions
In the early days of Python packaging, a package included just the source code that needed to be installed. For pure Python packages, this worked fine, and still does. But what happens when you need to compile some Rust or C or C++ or Fortran code as part of building the package?

## Solution #1: Compile it yourself
The original solution was to have each user compile the code themselves at install time. This can be quite slow, wastes resources, is often painful to configure, and still doesn’t solve a big part of the problem: shared library dependencies.
The Pillow image graphics library, for example, relies on third party shared libraries like `libpng` and `libjpeg`. In order to compile Pillow yourself, you have to install all of them, plus their development headers. On Linux or macOS you can install the system packages or the Homebrew packages; for Windows this can be more difficult. But you’re going to have to write different configuration for every single OS and even Linux distribution.

## Solution #2: Pip wheels
The way pip solves this problem is with packages called “wheels” that can include compiled code. In order to deal with shared library dependencies like `libpng`, any shared library external dependencies get bundled inside the wheel itself.
For example, let’s look at a Pillow wheel for Linux; a wheel is just a ZIP file so we can use standard ZIP tools:
```shell
$ zipinfo Pillow.whl
...
Pillow.libs/libpng16-213e245f.so.16.37.0
Pillow.libs/libjpeg-183418da.so.9.4.0
...
PIL/FpxImagePlugin.py
PIL/PalmImagePlugin.py
...
PIL/_imagingcms.cpython-39-x86_64-linux-gnu.so
...
```
The wheel includes both Python code, a compiled Python extension, and third-party shared libraries like `libpng` and `libjpeg`. This can sometimes make packages larger, as multiple copies of third-party shared libraries may be installed, one per wheel.

## Solution #3: Conda packages
Conda packages take a different approach to third-party shared libraries. `libjpeg` and `libpng` are packaged as additional Conda packages:
```shell
$ conda install -c conda-forge pillow
...
The following NEW packages will be INSTALLED:

...
  jpeg               conda-forge/linux-64::jpeg-9d-h36c2ea0_0
...
  libpng             conda-forge/linux-64::libpng-1.6.37-h21135ba_2
...
  pillow             conda-forge/linux-64::pillow-7.2.0-py38h9776b28_2
  zstd               conda-forge/linux-64::zstd-1.5.0-ha95c52a_0
...
```
Those installed `libjpeg` and `libpng` can then be depended on by other installed packages. They’re not wheel-specific, they’re available to any package in the Conda environment.
Conda can do this because it’s not a packaging system only for Python code; it can just as easily package shared libraries or executables.

## Summary: conda vs pip
| parameter                  | pip              | conda           |
| -------------------------- | ---------------- | --------------- |
| Install python             | No               | Yes, as package |
| 3rd party shared libraries | Inside the wheel | Yes, as package |
| Executable and tools       | No               | Yes, as package |
| Python source code         | Yes, as package  | Yes, as package |

# 3. PyPI vs Conda-Forge
Another fundamental difference between pip and Conda is less about the tools themselves, and more about the package repositories they rely on and how they work. In particular, most Python programs will rely on open source libraries, and these need to be downloaded from somewhere. For these, pip relies on [PyPI](https://pypi.org/), whereas Conda supports multiple different “==channels==” hosted on [Anaconda](https://anaconda.org/).

The default Conda channel is maintained by Anaconda Inc, the company that created Conda. It tends to have limited package selection and be somewhat less up-to-date, with some potential benefits regarding stability and GPU support. Beyond that I don’t know that much about it.

But there’s also the [Conda-Forge](https://conda-forge.org/) community channel, which packages far more packages, tends to be up-to-date, and is where you probably want to get your Conda packages most of the time. You can mix packages from the default channel and Conda-Forge, if you want the default channel’s GPU packages.
Let’s compare PyPI with Conda-Forge.

##  PyPI
Packages on PyPI are typically uploaded by the author of the Python package. For example, I am the author of the [Fil memory profiler](https://pythonspeed.com/fil/), and I also created the [PyPI package](https://pypi.org/project/filprofiler/).
Each package maintainer might compile or build their packages in their own idiosyncratic way, maintaining their own build infrastructure, choosing their own compilation options, and so on.
For example, NumPy can rely on multiple different BLAS libraries for fast linear algebra operations. The maintainers have chosen to build their PyPI packages with OpenBLAS; if you want another option, like Intel’s (maybe?) faster MKL, you’re out of luck unless you’re willing to compile the code yourself.

## Conda-Forge
Conda-Forge is a community project where package maintainers can be different than the original author of the package. For example, I have commit access to the [`typeguard` Conda-Forge recipe](https://github.com/conda-forge/typeguard-feedstock/) even though I am not a maintainer of the `typeguard` library.
Instead of custom builds done differently by each package maintainer, Conda-Forge has centralized build systems that recompile libraries, update recipe repositories, and in general automate everything massively. When a new version of Python 3 comes out, for example, a centralized update will happen, all the individual package maintainers will get PRs adding new packages; on PyPI this is up to individual maintainers to figure out.
Because of packaging infrastructure is centralized, Conda-Forge is able to [let you choose which BLAS to use](https://conda-forge.org/docs/maintainer/knowledge_base.html?highlight=mesa#switching-blas-implementation), and it will be used for NumPy and SciPy and whatever other packages you use that rely on BLAS.

# 4. Dealing with PyPI-only packages in Conda
While Conda-Forge has many packages, it doesn’t have all of them; many Python packages can only be found on PyPI. You can deal with lack of these packages in a number of ways.

## Install pip packages in a Conda environment
Conda environments are wrappers around virtualenvs; as such you can just call `pip install` yourself. If you’re using an `environment.yml` to install your Conda packages, you can also add pip packages:
```yml
name: myenv
channels:
  - conda-forge
dependencies:
  - python=3.9
  - numpy
  - pandas
  - gnuplot
  - pip:
      # Package that is only on PyPI
      - sandu
```

## Package it for Conda-Forge yourself
Because Conda-Forge does not require maintainers of the code to do the packaging, anyone can volunteer to add a package to Conda-Forge. That includes you!
For many Python packages it’s surprisingly easy process, and it’s quite automated, so handling new releases is often as easy as approving an automatically-created PR.

## Summary: PyPI vs. Conda-Forge

| parameter                    | PyPI                         | Conda-Forge   |
| ---------------------------- | ---------------------------- | ------------- |
| who creates package?         | Author of code               | Anyone        |
| Build infrastructure         | Maintained by author         | Centralized   |
| Open source python libraries | Essentially all              | Many          |
| Other open source tools      | None                         | Many          |
| Windows/MacOs/Linux packages | Usually, but upto maintainer | Almost always |

# 5. Additional tooling for Pip and Conda
Here’s a quick summary of some of the additional tooling you might want to use with either one:

| parameter            | pip                            | conda                                     |
| -------------------- | ------------------------------ | ----------------------------------------- |
| Reproducible builds  | pip-tools, pipenv, Poetry      | conda-lock                                |
| Virtual environments | `python -m venv`, `virtualenv` | Built-in                                  |
| Security scanning    | Most security scanners         | Jake                                      |
| Alternatives         | Poetry, Pipenv                 | Mamba, much faster and highly recommended |

To reiterate: if you do use Conda, I highly recommend [using Mamba as a replacement](https://pythonspeed.com/articles/faster-conda-install/). It supports the same command-line options and is much faster.


# 6. Understanding Conda and Pip
- Pip basic
	- [Pip](https://pip.pypa.io/en/stable/) is the Python Packaging Authority’s recommended tool for installing packages from the [Python Package Index](https://pypi.org/), PyPI.
	- Pip installs Python software packaged as wheels or source distributions. The latter may require that the system have compatible compilers, and possibly libraries, installed before invoking pip to succeed.
- Conda basic
	- [Conda](https://conda.io/docs/) is a cross platform package and environment manager that installs and manages conda packages from the [Anaconda repository](https://repo.anaconda.com/) as well as from the [Anaconda Cloud](https://anaconda.org/). 
	- Conda packages are binaries. There is never a need to have compilers available to install them. 
	- Additionally conda packages are not limited to Python software. They may also contain C or C++ libraries, R packages or any other software.
- ∇'encs:
	- Pip installs Python packages whereas conda installs packages which may contain software written in any language. 
		- For example, before using pip, a Python interpreter must be installed via a system package manager or by downloading and running an installer. Conda on the other hand can install Python packages as well as the Python interpreter directly.
	- conda has the ability to create isolated environments that can contain different versions of Python and/or the packages installed in them. This can be extremely useful when working with data science tools as different tools may contain conflicting requirements which could prevent them all being installed into a single environment. Pip has no built in support for environments but rather depends on other tools like [virtualenv](https://virtualenv.pypa.io/en/latest/) or [venv](https://docs.python.org/3/library/venv.html) to create isolated environments. 
		- Tools such as [pipenv](https://pipenv.readthedocs.io/en/latest/), [poetry](https://poetry.eustace.io/), and [hatch](https://github.com/ofek/hatch) wrap pip and virtualenv to provide a unified method for working with these environments.
	- Pip and conda also differ in how dependency relationships within an environment are fulfilled. When installing packages, pip installs dependencies in a recursive, serial loop. No effort is made to ensure that the dependencies of all packages are fulfilled simultaneously. This can lead to environments that are broken in subtle ways, if packages installed earlier in the order have incompatible dependency versions relative to packages installed later in the order. In contrast, conda uses a satisfiability (SAT) solver to verify that all requirements of all packages installed in an environment are met. This check can take extra time but helps prevent the creation of broken environments. As long as package metadata about dependencies is correct, conda will predictably produce working environments.
- Caveat with Conda
	- Given the similarities between conda and pip, it is not surprising that some try to combine these tools to create data science environments. A major reason for combining pip with conda is when one or more packages are only available to install via pip. Over 1,500 packages are available in the Anaconda repository, including the most popular data science, machine learning, and AI frameworks. These, along with thousands of additional packages available on Anaconda cloud from channeling including [conda-forge](https://conda-forge.org/) and [bioconda](https://bioconda.github.io/), can be installed using conda. Despite this large collection of packages, it is still small compared to the over 150,000 packages available on PyPI. Occasionally a package is needed which is not available as a conda package but is available on PyPI and can be installed with pip. In these cases, it makes sense to try to use both conda and pip.

## Comparision of conda and pip
|                       | conda                   | pip                             |
| --------------------- | ----------------------- | ------------------------------- |
| manages               | binaries                | wheel or source                 |
| can require compilers | no                      | yes                             |
| package types         | any                     | Python-only                     |
| create environment    | yes, built-in           | no, requires virtualenv or venv |
| dependency checks     | yes                     | no                              |
| package sources       | Anaconda repo and cloud | PyPI                                |


# References:
1. [Pip vs Conda: an in-depth comparison of Python’s two packaging systems](https://pythonspeed.com/articles/conda-vs-pip/)
2. [Understanding Conda and Pip](https://www.anaconda.com/blog/understanding-conda-and-pip)
3. [Conda vs Pip: Choosing your python package manager](https://www.askpython.com/python/conda-vs-pip)


# Tips
- Like `pip freeze`, getting the environment.yml file for conda environment: 
	- `conda env export -f environment.yml -n gpu-test` (where gpu-test is the name of conda env. Dumps the file in the same directory from where this command is run)
	- to create a new conda env using environment.yml file: `conda env create --name <conda-env-name> --file=environment.yml` 
		- for verbosity, use `-v` flag
		- for more details on the options, run `$conda env create --help`

