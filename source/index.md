# MWDB Training - Home

## Workshop slides

Slides from the Botconf workshop can be found [here](https://github.com/CERT-Polska/training-mwdb/raw/main/Botconf%202022%20-%20Build%20Your%20Own%20Malware%20Analysis%20Pipeline%20Using%20New%20Open%20Source%20Tools.pdf)


```{toctree}
---
caption: Exercises
maxdepth: 2
---

part-1.md
part-2.md
part-3.md
part-4.md
```

## Prerequisites

Open a terminal and check if these tools are installed:

- **Python 3** (recommended 3.7 or newer) 
  ```shell
  $ python3 -m pip
  ```
- **Git**
  ```shell
  $ git
  ```
- **Docker engine with Docker Compose**
  ```shell
  $ docker-compose
  ```
  [Docker Engine installation instructions for Ubuntu (docs.docker.com)](https://docs.docker.com/engine/install/ubuntu/)

  [Docker Compose installation instructions (docs.docker.com)](https://docs.docker.com/compose/install/)

Recommended environment is Ubuntu 18.04/20.04. Software used in training was not tested on other platforms (e.g. Mac OS, Windows), so prepare a fresh Ubuntu VM for the best workshop experience, especially if your host environment is unusual.

## Guides for exercises

- [mwdb-core - Advanced search based on Lucene queries](https://mwdb.readthedocs.io/en/latest/user-guide/7-Lucene-search.html)

- [mwdblib - Guide for REST API and mwdblib usage](https://mwdb.readthedocs.io/en/latest/user-guide/8-REST-and-mwdblib.html)

- [mwdblib - API reference](https://mwdblib.readthedocs.io/en/latest/)

- [karton - documentation](https://karton-core.readthedocs.io/en/latest/)

- [malduck - API reference](https://malduck.readthedocs.io/en/latest/)
