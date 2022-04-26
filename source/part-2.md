# Part 2 - mwdblib

## Setup

Create a virtualenv and activate it

```
$ python3 -m venv venv
$ . venv/bin/activate
```

Install mwdblib with CLI extras + ipython shell for convenience

```
(venv) $ pip install mwdblib[cli] ipython
```

If you have problems during installation with ``cryptography`` package, try to upgrade pip and retry the previous step.

```
(venv) $ pip install -U pip
...
(venv) $ pip install mwdblib[cli] ipython
```

## **Exercise #2.1**: Get information about 10 recent files using mwdblib

**Goal**: learn how the `recent_files` method works

The main interface is MWDB object that provides various methods to interact with MWDB. Let's start with log in to mwdb.cert.pl service.

```python
In [1]: from mwdblib import MWDB

In [2]: mwdb = MWDB()

In [3]: mwdb.login()
Username: demologin
Password:
```

After successful login, let's begin with `recent_files` to get recently uploaded file from API.

```python
In [4]: mwdb.recent_files()
Out[4]: <generator object MWDB._recent at ...>
```

`recent_files` function returns generator which does the same job as scrolling down the Samples view to view the older entries. Let's use the `next` function to get the most recent file:

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

## **Exercise #2.2**: Check properties of `780e8fb254e0b8c299f834f61dc80809`

**Objectives**:
- Check file's name, tags and children
- Get the first 16 bytes of the file
- Get the linked configuration of this file.
- Check names of the other files that are parents of that configuration

Let's start with getting a file by hash. Use `mwdb.query_file` method to get an object.

```python
In [11]: file = mwdb.query_file("780e8fb254e0b8c299f834f61dc80809")

In [12]: file
Out[12]: <mwdblib.file.MWDBFile at ...>
```

Using the retrieved `MWDBFile` object we can get some details about the file e.g. name, tags, child objects.

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

As you can see in `[15]`, there is a configuration attached to the file. We can get it by index operator or use config attribute to get the latest configuration object. Let's see what has been ripped:

```python
In [17]: file.children[0].config
Out[17]: {'c2': [{'host': '172.111.210.207'}], 'type': 'avemaria'}

In [18]: file.config
Out[18]: <mwdblib.file.MWDBConfig at ...>

In [19]: file.config.config
Out[19]: {'c2': [{'host': '172.111.210.207'}], 'type': 'avemaria'}
```

Many malware samples can share the same configuration. Let's explore them:

```python
In [20]: avemaria = file.config

In [21]: avemaria.parents
Out[21]: 
[<mwdblib.file.MWDBFile at 0x7faed2383f10>,
 <mwdblib.file.MWDBFile at 0x7faed2383070>,
 <mwdblib.file.MWDBFile at 0x7faed2335b20>,
 <mwdblib.file.MWDBFile at 0x7faed2335a00>]

In [22]: [parent.name for parent in avemaria.parents]
In [22]: 
['400000_2236f1a1cacde1dc',
 '400000_1973838fc27536e6',
 '400000_2bf452f7796153ef',
 '400000_3539b9d228df73c6']
```

## **Exercise #2.3**: Using mwdblib CLI

**Objectives**:
  - Download 10 files that were tagged as `ripped:lokibot` using mwdblib CLI

1. First exit ipython using `exit()` or CTRL+D 

2. Then type `mwdb` command in terminal and press ENTER

   mwdblib library installs CLI tool along with the Python binding which can be used in fancy oneliners and Bash scripts

3. Type `mwdb list` to see the list of recent files

4. Copy one of the hashes and paste as a `mwdb get <hash>` argument

5. Download file contents using `mwdb fetch <hash>`

6. Search for other files (e.g. ripped:lokibot) using `mwdb search 'tag:"ripped:lokibot"'`

7. You can also get only file hashes for further processing by adding `-o short -n 10`

   (`-o short` means hash-only output and `-n 10` fetches only 10 first files)

8. Make an oneliner that will download first 10 samples tagged as `ripped:lokibot`

Answer:

```
mwdb search 'tag:"ripped:lokibot"' -o short -n 10 | xargs -n 1 mwdb fetch
```

or

```
for f in $(mwdb search 'tag:"ripped:lokibot"' -o short -n 10); do mwdb fetch $f; done
```

## **Exercise #2.4**: Joining CLI with other tools

**Objectives**:
   - Get 10 most recent Mutexes from ``nanocore`` configs

````{dropdown} Click to see the intended solution
```bash
for f in $(mwdb search configs 'family:nanocore' -n 10 -o short )
        mwdb fetch $f /tmp/$f
        cat /tmp/$f | jq '.Mutex'
end
```

or

```bash
for f in $(mwdb search configs 'family:nanocore' -n 10 -o short )
        mwdb fetch $f - | jq '.Mutex'
end
```
````
