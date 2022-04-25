# Part 3 - Karton

## Setup

This part requires [Karton Playground](https://github.com/CERT-Polska/karton-playground) to be set up.

```shell
git clone https://github.com/CERT-Polska/karton-playground.git
cd karton-playground
sudo docker-compose up  # this may take a while
```

Also take a look at the [Karton documentation](https://karton-core.readthedocs.io/en/latest/)

**Available services**

- `127.0.0.1:8030` karton-dashboard
- `127.0.0.1:8080` mwdb-core (user: admin, password: admin)
- `127.0.0.1:8090` minio (user: mwdb, password: mwdbmwdb)

## **Exercise #3.1**: Adding new service to the Karton pipeline

**Goal**: Learn how to connect new karton systems to your network

1. Integrate an existing karton service into your pipeline: karton-autoit-ripper

https://github.com/CERT-Polska/karton-autoit-ripper

```shell
$ python3 -m venv venv
$ source ./venv/bin/activate
$ pip install karton-autoit-ripper

$ # playground-specific: copy local config to cwd
$ cp config/karton.local.ini karton.ini
$ karton-autoit-ripper
[2021-04-11 17:19:57,867][INFO] Service karton.autoit-ripper started
```

2. Download a sample, and verify its hash

```shell
$ wget https://github.com/CERT-Polska/training-mwdb/blob/main/autoit-malware.bin
$ sha256sum autoit-malware.bin
a4816d4fecd6d2806d5b105c3aab55f4a1eb5deb3b126f317093a4dc4aab88a1 autoit-malware.bin
```

3. Finally, upload it to your local mwdb (http://127.0.0.1:8080, admin:admin)

## **Exercise #3.2**: Setup karton-yaramatcher

**Goal**: Learn how to connect new karton systems to your network

## **Exercise #3.3**: Write your own service

**Goal**: Learn how to create a new karton service from ground up

1. Download a template:

https://github.com/CERT-Polska/training-mwdb/blob/main/karton-template.py

2. Edit the template, and:

- Run the `strings` utility on every incoming sample
- Save the result in a variable (use [subprocess.check_output](https://docs.python.org/3/library/subprocess.html#subprocess.check_output))
- Upload the result to mwdb (already handled in the template)

