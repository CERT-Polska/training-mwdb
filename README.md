# MWDB Training

## Agenda

- **mwdb.cert.pl**
  - What the heck is MWDB?
  - Tour de mwdb.cert.pl
  - Scripting and automation with mwdblib
- **mwdb-core** and **karton**
  - Run a self-hosted mwdb-core and karton instances
  - Experiment with karton-playground
  - Distributed collaboration with mwdb remotes
  - Advanced programming techniques with malduck

## Prerequisites

Open a terminal and check if these tools are installed:

- **Python 3** (recommended 3.6 or newer) 
  ```
  $ python3 -m pip
  ```
- **Git**
  ```
  $ git
  ```
- **Docker engine with Docker Compose**
  ```
  $ docker-compose
  ```
  https://docs.docker.com/engine/install/ubuntu/

  https://docs.docker.com/compose/install/

Recommended environment is Ubuntu 18.04/20.04. Software used in training was not tested on other platforms (e.g. Mac OS, Windows), so prepare a fresh Ubuntu VM for the best workshop experience, especially if your host environment is unusual.

## Part 1a: mwdb.cert.pl tour

### Cheat-sheet for exercises

Useful materials:

- [Advanced search based on Lucene queries - mwdb-core User Guide](https://mwdb.readthedocs.io/en/latest/user-guide/7-Lucene-search.html)

Starting points:

**Exercise #2**: Sample hash to find `5762523a60685aafa8a681672403fd19`

**Exercise #3**: Configuration hash is `f2e216695d4ce7233f5feb846bc81b8fffe9507988c7f5caaca680c0861e5e02` (https://mwdb.cert.pl/config/f2e216695d4ce7233f5feb846bc81b8fffe9507988c7f5caaca680c0861e5e02)

**Exercise #4**:

1. https://mwdb.cert.pl/blob/60c9ad80cde64e7cae9eec0c11dd98175860243aa40a3d8439bbf142d2a0e068
2. https://mwdb.cert.pl/blob/48914f0a6b9f4499da31d2217a7ee2e8c8f35f93ab5c992333f5c1aa947d9009

### Exercises

**Exercise #0**: Getting familiar with the interface

MWDB welcomes us with a list of recently uploaded samples.

**Recent views** allow to see basic information about latest objects in the repository and interactively explore the dataset using Lucene-based queries and clickable fields. If sample was uploaded within last 72 hours, it is additionally marked with yellowish background. Yellow color is a bit more intense if file was uploaded at least 24 hours ago.

If you click on sample hash, you will navigate to the detailed sample view. Here you can see all the details about file. Left side contains four tabs. The first one called Details presents basic file information like original file name, size, file type and hash values. 

On the right side of view you can see tags, relations with other objects, attributes containing some extra information and the comments section. These information are added to MWDB mainly by our analysis backend, where sample is sent on the first upload. If analysis was successful, all interesting analysis artifacts are uploaded back to the MWDB.

During our tour we will go through all of these elements, starting from tags.

**Exercise #1**: Filtering samples by tags

1. Go to the main view and click on any tag starting with `runnable:` to include only runnable samples
2. Click on the 🛇 character of tag starting with `feed:` to exclude that feed from the results
3. Then click quick query `Only ripped:*` to include only the original, ripped samples.

   Let's take a look at the resulting query:

   ```
   tag:"runnable:win32:exe" AND NOT tag:"feed:malwarebazaar" AND tag:"ripped:*"
   ```	

   Query language is based on Lucene syntax subset and consists of two basic elements:

   - field conditions `tag:"runnable:win32:exe"` that support wildcards `*`, `?`
   - operators: `OR`, `AND`, `NOT`

   As you can see, we have various tags identifying the malware as 'formbook' based of various criterion. If we want to find everything that is recognized as formbook regardless of the source of classification, we can just use the wildcards (like in `ripped:*` case):

   ```
   tag:*formbook*
   ```

   You probably also noticed that tags are colored and the color is not completely random. Explanation is on the slides.

4. Now, add to the query an additional condition:

   ```
   AND size:[10000 TO 15000]
   ```

   Lucene query language also supports ranges so we can search for samples of given size or uploaded on given date

   ```
   upload_time:<=2020-01-01
   ```

**Exercise #2**: Exploring sample view and hierarchy

1. Copy hash to the query field in Samples view: `5762523a60685aafa8a681672403fd19`

   Click on the hash in row to navigate to the sample details.

   Here you can see all the details about file. Left side contains three tabs: Details, Relations and Preview. The first one called Details presents basic file information like original file name, size, file type and hash values.

   Blue-colored fields are clickable, so you can search for other samples with the same name or size.

   On the right side of view you can see tags, relations with other objects, attributes and the comments section.

   As you can see in tags, sample is `archive:rar` and comes from Malwarebazaar. Link to the MalwareBazaar can be found in the Attributes section.

   Related samples in that case are the files that were unpacked from that archive.

2. See the `Related samples` box on the left. Go to the child sample tagged `ripped:formbook`

   This is the actual executable contained in the malicious archive. Based on tags we may say that:

   - `runnable:win32:exe` it is Windows 32-bit executable
   - `yara:win_formbook`  One of our Yara rules matched this sample as Formbook
   - `et:formbook`        ET Pro traffic rules matched this sample as Formbook (more info in comments)
   - `ripped:formbook`    We have successfully ripped Formbook configuration from this sample

3. Then go to the next child tagged `dump:win32:exe`

    Memory dump contains the unpacked code part as a result of dynamic analysis in sandbox. We're performing multiple dumps based on many heuristics but we upload to the MWDB only the best candidate that contains the most complete malware configuration.

4. Check `Static config` tab.

   See the extracted static configuration.

   Configuration is the second data type in MWDB. Malware configurations are meant to parametrize the malware behavior and usually contains useful IoCs.

   The format of configuration depends on malware family, usually deriving from the structure “proposed” by the malware author.

5. Come back to the `Details` and make sure you're on MD5 `8e56eee9cf853d2ec4c695282c01fe0a`

   Go to the `Relations` tab. It shows the parents and children of current object. Notice that two samples have the same unpacked core.

6. Click on the Config box in the Relations graph to expand it. Scroll down to zoom out the graph.

**Exercise #3**: Looking for similar configurations

1. Click on the config hash (`f2e216695d4ce7233f5feb846bc81b8fffe9507988c7f5caaca680c0861e5e02`) in `Related configs` box.

2. Go to the Preview box

   Configurations are just a simple JSON objects. The only special thing is hashing algorithm e.g. lists are hashed non-orderwise, so if domains were ripped in different order, configuration hash will be still the same.

3. Go back to the Details box. Expand `urls` and click on `www.discorddeno.land/suod/`

   ```
   cfg.urls*.url:"www.discorddeno.land/suod/"
   ```

   The resulting query looks for all `url` keys in `urls` lists that have `www.discorddeno.land/suod/`.

   Let's check if `/suod/` path was used in other configs as well.

4. Modify query to look for other configs with `/suod/` path replacing the domain with wildcard `*`.

   ```
   cfg.urls*.url:"*/suod/"
   ```

   There are two configurations. What URL was used in the second configuration? 

5. Now let's check if `/suod/` occurs in other configurations regardless of the configuration structure. For that query we can use full-text search in JSON.

   ```
   cfg:"*/suod/*"
   ```

   Are there more configurations like that?

6. If not, let's search for configurations with .land TLD

   ```
   cfg:"*.land*"
   ```

7. Then click on `agenttesla` config (`e031b192d40f6d234756f8508f7d384db315983b57d8fc3216d20567056bd88b`)

   Ok, there is no .land TLD. but .landa e-mail address. To ilustrate how full-text search works, go to Preview, press CTRL-F and type ".land" to see what parts of JSON were matched

   How we can improve our query? Let's add `"` character at the end to match the end string.

   ```
   cfg:"*.land\"*"
   ```

   Go to the Gandcrab configuration and check in Preview what was matched.

**Exercise #4**: Blobs and dynamic configurations

The third object type in MWDB is blob. While config represents structured (JSON) data, blob is an unstructured one. Blobs are just simple text files, usually containing some raw, but human-readable content.

Let's take a look at some examples.

1. Navigate to the https://mwdb.cert.pl/blob/60c9ad80cde64e7cae9eec0c11dd98175860243aa40a3d8439bbf142d2a0e068

   What we see is bunch of decrypted strings from the Agent Tesla that were ripped from the malware sample.

   They're not structured because we don't semantically analyze every string, but it's still nice to have them in repository.

2. Navigate to the https://mwdb.cert.pl/blob/48914f0a6b9f4499da31d2217a7ee2e8c8f35f93ab5c992333f5c1aa947d9009

   Now we have decrypted strings from Remcos. Even if data are unstructured, they can be considered a part of static configuration and used for looking for malware similarities.

   Let's take a look at the parent of this blob: the static configuration object.

   (https://mwdb.cert.pl/config/29c1f3c14a446b2a77ce58cbc59619fbfe7459c56fe1c8408597538384aa56ac)

   Not much, just a C2 host/port and credentials. 
   
3. Let's take a look for another configuration with this host by expanding `c2` key and clicking at the `host` address.

   Oh, there is another.

   The resulting query is `cfg.c2*.host:"ongod4life.ddns.net:4344"`

   And the other configuration should be: https://mwdb.cert.pl/config/9afac348443a7aa9ca5d33cffcc984751cebf15f065cb90b48911943fb10e1f6

   They're pretty much the same and only the `raw_cfg` differs. How to easily compare them?

4. Go to the blob (`da2055f0e90355bfaf3cc932f7fdb2f82bfd79c26f95b61b23b9cd77f9b0e32d`). In blob view, find the `Diff with` button on the right side of tabs. Click on that button.

   Now we can choose another blob to compare.

   The most simple way is to copy to clipboard the previous blob id and paste it into query bar.

   `48914f0a6b9f4499da31d2217a7ee2e8c8f35f93ab5c992333f5c1aa947d9009`

   Then press ENTER and choose the searched blob. What's the difference between these blobs?

   But blobs are not only the strings and unstructured static things.

5. Go to the `Blobs` list and click on `dyn_cfg` in `Blob type` column or type manually `type:dyn_cfg`.
   Then filter out `dynamic:mirai` tag, there are lots of them but they're not so interesting.

6. Check out the `hancitor` dynamic configuration.

   https://mwdb.cert.pl/blob/3b032876cc2d77d28625b9dfee0686663e60385cda7f9031afac6cf2b0c6d6e4	

   Dynamic configurations also parametrize the malware behavior, but they're fetched from external source. In that case we have set of commands to run a second stage malware.

   Fetched second stage is linked as a blob child.

   Other possible types of dynamic configuration are injects, mail templates for spam botnets or malware updates.

   Example of more verbose Kronos dynamic configuration:

   https://mwdb.cert.pl/blob/2e4d109edb8b2fa7c1f1d7592a284bbf15e3e51d24d1d9cdda91c9ae582cf05c/config

   Blob children are files dropped from C&C and structured (parsed) fragments of dynamic configuration

**Exercise #5**: Let's upload something!

All of things you see are shared with all MWDB accounts: the 'public' group. If you use a query:

```
NOT shared:public
```

You should not have any results, because all samples you see are public. So how to gather some 'private' samples? You need to upload them!

1. Fetch an example sample from Github `ex5malware.zip`. **Don't unpack it**, just download to some temporary location.

2. Click on `Upload` in the navbar (https://mwdb.cert.pl/upload)

   Select the sample you have just downloaded.

3. Take a look at `Share with` options. There are four options:

   - `All my groups` which means that sample will be shared with all your **private** groups, so it will be visible e.g. only within your organization
   - `Single group` so you can select some specific user or group
   - `Everybody`, so all newcomers will see your sample
   - `Only me`, so even your colleagues from the same organization won't see the sample.

4. Upload a sample with a default `All my groups` option.

5. Take a look at the Shares box. Who has the access to your sample?

6. Then use the Relations tab to traverse to the Config. Who has access to the configuration?

In MWDB sharing model, if you upload an private sample you got immediate access to all its descendants. So you always get all the data related with ripped configuration, but not necessarily all of its parents (the other samples).

If you want, you can always change your mind and share the sample with somebody else. But you can't reverse the action, so if something was shared by mistake, contact the administrators.

7. Come back to the original sample and share it with `public` using `Share with group` input field.

## Part 1b: Scripting and automation with mwdblib

### Setup

Create an virtualenv and activate

```
$ python3 -m venv venv
$ . venv/bin/activate
```

Install mwdblib with CLI extras + ipython shell

```
(venv) $ pip install mwdblib[cli] ipython
```

### Exercises

**Exercise #6**: Get information about 10 recent files using mwdblib

The main interface is MWDB object that provides various methods to interact with MWDB. Let’s start with log in to mwdb.cert.pl service.

```python
In [1]: from mwdblib import MWDB

In [2]: mwdb = MWDB()

In [3]: mwdb.login()
Username: demologin
Password:
```

After successful login, let’s begin with `recent_files` to get recently uploaded file from API.

```python
In [4]: mwdb.recent_files()
Out[4]: <generator object MWDB._recent at ...>
```

`recent_files` function returns generator which does the same job as scrolling down the Samples view to view the older entries. Let’s use next function to get the most recent file:

```python
In [5]: files = mwdb.recent_files()

In [6]: file = next(files)

In [7]: file
Out[7]: <mwdblib.file.MWDBFile at ...>
```

... and we got the file! 

To get the next 10 files, we can use [itertools.islice](https://docs.python.org/3/library/itertools.html#itertools.islice) method:

```python
In [8]: import itertools

In [9]: recent_10 = list(itertools.islice(files, 10))

In [10]: recent_10
Out[10]:
[<mwdblib.file.MWDBFile at ...>,
 <mwdblib.file.MWDBFile at ...>,
 <mwdblib.file.MWDBFile at ...>,
 <mwdblib.file.MWDBFile at ...>,
 <mwdblib.file.MWDBFile at ...>,
 <mwdblib.file.MWDBFile at ...>,
 <mwdblib.file.MWDBFile at ...>,
 <mwdblib.file.MWDBFile at ...>,
 <mwdblib.file.MWDBFile at ...>,
 <mwdblib.file.MWDBFile at ...>]
```

But what we can do with these file objects?

**Exercise #7**: Check properties of `780e8fb254e0b8c299f834f61dc80809`

**Objectives**:
- Check file's name, tags and children
- Get the first 16 bytes of the file
- Get the configuration of this file.
- Check names of the other files that are parents of this configuration

Let's start with getting a file by hash. Use `mwdb.query_file` method to get an object.

```python
In [11]: file = mwdb.query_file("780e8fb254e0b8c299f834f61dc80809")

In [12]: file
Out[12]: <mwdblib.file.MWDBFile at ...>
```

Using retrieved `MWDBFile` object we can get some details about the file e.g. name, tags, child objects.

```python
In [13]: file.name
Out[13]: '400000_1973838fc27536e6'

In [14]: file.tags
Out[14]: ['dump:win32:exe', 'avemaria']

In [15]: file.children
Out[15]: [<mwdblib.file.MWDBConfig at ...>]
```

We can also download its contents

```python
In [16]: file.download()[:16]
Out[16]: b'MZ\x90\x00\x03\x00\x00\x00\x04\x00\x00\x00\xff\xff\x00\x00'
```

As you can see in `[15]`, there is a configuration attached to the file. We can get it by index operator or use config attribute to get the latest configuration object. Let’s see what has been ripped:

```python
In [17]: file.children[0].config
Out[17]: {'c2': [{'host': '172.111.210.207'}], 'type': 'avemaria'}

In [18]: file.config
Out[18]: <mwdblib.file.MWDBConfig at ...>

In [19]: file.config.config
Out[19]: {'c2': [{'host': '172.111.210.207'}], 'type': 'avemaria'}
```

Few malware samples can share the same configuration. Let’s explore them:

```python
In [20]: avemaria = file.config

In [21]: avemaria.parents
Out[21]: 
[<mwdblib.file.MWDBFile at ...>,
 <mwdblib.file.MWDBFile at ...>,
 <mwdblib.file.MWDBFile at ...>]

In [22]: [parent.name for parent in avemaria.parents]
In [22]: 
['400000_1973838fc27536e6',
 '400000_2bf452f7796153ef',
 '400000_3539b9d228df73c6']
```

**Exercise #8**: Using mwdblib CLI

**Objectives**:
- Download 10 files that were tagged as `ripped:lokibot` using mwdblib CLI

1. First exit ipython using `exit()` or CTRL+D 

2. Then type `mwdb` command in terminal and press ENTER

   mwdblib library installs CLI tool along with the Python binding which can be used in fancy oneliners and Bash scripts

3. Type `mwdb list` to see the list of recent files

4. Copy one of the hashes and paste as a `mwdb get <hash>` argument

5. Download file contents using `mwdb fetch <hash>`

6. Search for other files (e.g. ripped:lokibot) using `mwdb search 'tag:"ripped:lokibot"'`

7. You can also get only file hashes for further processing adding `-o short -n 10` parameters

   (`-o short` means hash-only output and `-n 10` fetches only 10 first files)

8. Make a oneliner that will download first 10 samples tagged as `ripped:lokibot`

Answer:

```
mwdb search 'tag:"ripped:lokibot"' -o short -n 10 | xargs -n 1 mwdb fetch
```

or

```
for f in $(mwdb search 'tag:"ripped:lokibot"' -o short -n 10); do mwdb fetch $f; done
```

## Part 2: mwdb-core and karton

### Setup

This part requires [Karton Playground](https://github.com/CERT-Polska/karton-playground) to be set up.

```
git clone https://github.com/CERT-Polska/karton-playground.git
cd karton-playground
sudo docker-compose up  # this may take a while
```

Also take a look at the [Karton documentation](https://karton-core.readthedocs.io/en/latest/)

**Available services**

- `127.0.0.1:8030` karton-dashboard
- `127.0.0.1:8080` mwdb-core (user: admin, password: admin)
- `127.0.0.1:8090` minio (user: mwdb, password: mwdbmwdb)

## Exercises

### Exercise 1: Adding new service to the Karton pipeline

1. Integrate an existing karton service into your pipeline: karton-autoit-ripper

https://github.com/CERT-Polska/karton-autoit-ripper

```
$ python3 -m venv venv
$ source ./venv/bin/activate
$ pip install karton-autoit-ripper

$ # playground-specific: copy local config to cwd
$ cp config/karton.local.ini karton.ini
$ karton-autoit-ripper
[2021-04-11 17:19:57,867][INFO] Service karton.autoit-ripper started
```

2. Download a sample, and verify its hash

```
$ wget https://github.com/CERT-Polska/training-mwdb/blob/main/autoit-malware.bin
$ sha256sum autoit-malware.bin
a4816d4fecd6d2806d5b105c3aab55f4a1eb5deb3b126f317093a4dc4aab88a1 autoit-malware.bin
```

3. Finally, upload it to your local mwdb (http://127.0.0.1:8080, admin:admin)

### Exercise 2: Write your own service

1. Download a template:

https://github.com/CERT-Polska/training-mwdb/blob/main/karton-template.py

2. Edit the template, and:

- Run the strings utility on every incoming sample
- Save the result in a variable (use subprocess.check_output)
- Upload the result to mwdb (already handled in the template
