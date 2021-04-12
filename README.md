# MWDB Training

**Agenda**:

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

## Cheat-sheet

**Exercise #2**: Sample hash to find `5762523a60685aafa8a681672403fd19`

**Exercise #3**: Configuration hash is `f2e216695d4ce7233f5feb846bc81b8fffe9507988c7f5caaca680c0861e5e02` (https://mwdb.cert.pl/config/f2e216695d4ce7233f5feb846bc81b8fffe9507988c7f5caaca680c0861e5e02)

## Exercises

**Exercise #1**: Filtering samples by tags

1. Go to the main view and click on any tag starting with `runnable:` to include only runnable samples
2. Click on the 🛇 character of tag starting with `feed:` to exclude that feed from the results
3. Then click quick query `Only ripped:*` to include only the original, ripped samples.

**Exercise #2**: Exploring sample view and hierarchy

1. Copy hash to the query field in Samples view: `5762523a60685aafa8a681672403fd19`

   Click on the hash in row to navigate to the sample details.

   Here you can see all the details about file. Left side contains three tabs: Details, Relations and Preview. The first one called Details presents basic file information like original file name, size, file type and hash values.

   BLue-colored fields are clickable, so you can search for other samples with the same name or size.

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

    Memory dump contains the unpacked code part as a result of dynamic analysis in sandbox. We're performing multiple dumps based on many heuristics but we upload to the MWDB only the best candidate
    that contains the most complete malware configuration.

4. Check `Static config` tab.

	See the extracted static configuration that parametrize the malware behavior and usually contains useful IoCs.

	Configuration is the second data type in MWDB. It is used to 

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

	 Let's look if `/suod/` path was used in other configs as well.

4. Modify query to look for other configs with `/suod/` path replacing the domain with wildcard `*`.

	```
	cfg.urls*.url:"*/suod/"
	```

	There is older configuration that contain `/suod` path. What URL was used in older configuration? 

5. Check if `/suod/` occurs in other configurations regardless of the configuration structure. For that query we can use full-text search in JSON.

	```
	cfg:"*/suod/*"
	```

6. Search for configurations that contain '.land' TLD

	```
	cfg:"*.land*"
	```


