## Collection of scripts to be used with SATestBuild.py

This repository contains the scripts to download and to analyze several projects using the clang static analyzer (CSA). 

### 1. Projects

| Project          | Version    | Description |
| ----------       | ---------- |----------|
| tmux             | 2.7        | a terminal multiplexer |
| redis            | 4.0.9      | an in-memory database |
| openSSL          | 1.1.1-pre6 | a software library for secure communication |
| twin             | 0.8.0      | a windowing environment |
| git              | 2.17.0     | a version control system |
| postgreSQL       | 10.4       | an object-relational database management system |
| SQLite3          | 3230100    | a relational database management system |
| curl             | 7.61.0     | command-line tool for transferring data |
| libWebM          | 1.0.0.27   | a WebM container library |
| memcached        | 1.5.9      | a general-purpose distributed memory caching system |
| xerces-c++       | 3.2.1      | a validating XML parser |
| XNU (MacOS only) | 4570.41.2  | an operating system kernel used in Apple products |

The projects are configured to be built in debug mode + assertions whenever possible, to improve the CSA reports.

### 2. Usage

Clang provides a number of scripts in `clang/utils/analyzer` to analyze projects using the CSA. The main scripts are:

* `SATestAdd.py`: analyses a project, creates reference results and adds the project to the list of projects to be analyzed.
* `SATestBuild.py`: generates comparative results or regenerates reference results.
* `CmpRuns.py`: given two results, shows comparative statistics.

In order to use the scripts in this repository, you must first add the projects to the list of projects to be analyzed, using `SATestAdd.py <project-directory>`, e.g.:

`$ ./SATestAdd.py tmux`

The script will download, analyze the project, generate the reference reports and add the project path to a file called `projectMap.csv` (or will first create the file if it does not exist).

Once `projectMap.csv` exists, `SATestBuild.py` can be used to generate reference reports:

`$ ./SATestBuild.py -r`

or to generate the comparative results:

`$ ./SATestBuild.py`

It will analyze all the projects in the projectMap.csv and, if generating comparative results, it will print the number of bugs added/removed.

### 2.1. Changing Project Versions

If you want to change the version of the project analyzed by the scripts, you should:

* Update the `download_project.sh` script to download the new version of the source code and rename the new source directory to `CachedSource`.
* Delete the old directory `CachedSource`, so `SATestBuild.py` can download the new source code.

### 3. The scripts

Each project directory contains the following scripts:

* `cleanup_run_static_analyzer.sh`: executed before analysis to remove any compilation object from previous compilations.
* `download_project.sh`: downloads the project and unpack it to a folder called CachedSource.
* `run_static_analyzer.cmd`: commands to configure and build the project.

In addition to these scripts, an optional file called `changes_for_analyzer.patch` can be placed in the project directory and it is applied to the project's source code before `run_static_analyzer.cmd` is executed.

### 4. Patches to projects

The repository contains patches to two projects, otherwise the CSA is not able to analyze these projects.

#### 4.1. LibWebM

The patch adds the `cstring` header to `webm2pes.cc` to fix an error that `std::memcpy` could not be found, and adds some `static_cast`s to the same file to fix some integer narrowing errors.

#### 4.2. XNU

The patch changes two makefiles do disable compilation with `-Werror`, adds the `stddef` header to fix a missing declaration of `ptrdiff_t` and unconstify a member that causes the CSA to complain about an assignment to const member.

### 5. Patch to SATBuild.py

This repo also contains a patch to `SATBuild.py` that changes the following:

* Removes `deadcode` and `security` checkers (I was only interested in path sensitive analysis when analysing these projects).
* Enables verbose mode.
* Disables html report generation.
* Adds the flag `--keep-going` to the `scan-build.py` call, so the CSA does not stop when it finds an internal error.
* Adds a new flag to make bug refutation with z3 (simply change it to `true` to enable it). Requires clang to be compiled with Z3.
* Adds `--show-stats` flag to the `CmpRuns.py` script call, to print statistics (you need clang built with assertions enabled or no statistic will be generated).
* Removes cleanup when generating reference results, as it removes html reports (if html generation is enabled).

To apply the patch, you should run:

`$ patch < SATestBuild.patch`
