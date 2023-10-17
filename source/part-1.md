# Part 1 - MWDB

## **Exercise #1.0**: Getting familiar with the interface

MWDB welcomes us with a list of recently uploaded samples.

**Recent views** allow to see basic information about latest objects in the repository and interactively explore the dataset using Lucene-based queries and clickable fields. If sample was uploaded within last 72 hours, it is additionally marked with yellowish background. Yellow color is a bit more intense if file was uploaded at least 24 hours ago.

If you click on sample hash, you will navigate to the detailed sample view. Here you can see all the details about file. Left side contains a few tabs. The first one called Details presents basic file information like original file name, size, file type and hash values.

On the right side of view you can see tags, relations with other objects and the comments section. This information are added to MWDB mainly by our analysis backend, where sample is sent on the first upload. If analysis was successful, all interesting analysis artifacts are uploaded back to the MWDB.

During our tour we will go through all of these elements, starting from tags.

## **Exercise #1.1**: Filtering samples by tags

**Goal**: get familiar with the interface, play around with the search query

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

   As you can see, we have various tags identifying the malware based on various criteria. If we want to find everything that is recognized as specific family regardless of the source of classification, we can just use the wildcards (like in `ripped:*` case):

   ```
   tag:*formbook*
   ```

   You probably also noticed that tags are colored and the color is not completely random. Explanation is on the slides.

4. Now, add an additional condition to the query:

   ```
   tag:*formbook* AND size:[10000 TO 15000]
   ```

5. Lucene query language also supports ranges so we can search for samples of given size or uploaded on given date

   ```
   tag:*formbook* AND size:[10000 TO 15000] AND upload_time:<=2021-01-01
   ```

## **Exercise #1.2**: Exploring sample view and hierarchy

**Goal**: explore the sample view, understand the object hierarchy

1. Copy hash to the query field in Samples view: `5762523a60685aafa8a681672403fd19`

   Click on the hash in row to navigate to the sample details.

   Here you can see all the details about file. Left side contains three tabs: Details, Relations and Preview. The first one called Details presents basic file information like original file name, size, file type and hash values.

   Blue-colored fields are clickable, you can use them to quickly search for other samples matching the given criteria.

   Just beneath the file details tab, attributes are displayed. This section contains all sorts of miscellaneous information about the sample and/or analysis.

   On the right side of view you can see tags, relations with other objects and the comments section.

   As you deduce from tags, sample is a rar archive (`archive:rar`) and comes from Malwarebazaar (`feed:malwarebazaar`). Link to the sample on MalwareBazaar platform can be found in the Attributes section.

   In this case, related samples are the file that were unpacked from that archive.

2. See the `Related samples` box on the right. Go to the child sample tagged `ripped:formbook`

   This is the actual executable contained in the malicious archive. Based on tags we can say that:

   - `runnable:win32:exe` it is Windows 32-bit executable
   - `yara:win_formbook`  One of our Yara rules matched this sample as Formbook
   - `et:formbook`        ET Pro traffic rules matched this sample as Formbook (more info in comments)
   - `ripped:formbook`    We have successfully ripped Formbook configuration from this sample

3. Navigate to the next child tagged `dump:win32:exe`

   This is the memory dump that contains the unpacked formbook payload. We got it by running the sample on our sandbox and then performing memory dumps when specific *interesting* prerequisites are matched. It's worth nothing that while the analysis produces a bunch of memory dumps, we upload the best one that allowed us to get the complete malware configuration.

4. Check `Static config` tab.

   See the extracted static configuration.

   Configuration is the second data type in MWDB. Malware configurations are meant to parametrize the malware behavior and they usually contain useful IoCs.

   The format of configuration depends on malware family, usually deriving from the structure "proposed" by the malware author.

5. Go back to the `Details` tab and make sure you're on MD5 `8e56eee9cf853d2ec4c695282c01fe0a`

   Go to the `Relations` tab. It presents the parents and children of current object. Notice how two distinct samples have been unpacked into the same malicious core.

6. Click on the Config box in the Relations graph to expand it. Zoom out the graph to see the whole graph.

## **Exercise #1.3**: Looking for similar configurations

**Goal**: Find configurations that are similar to Formbook config 

1. Click on the config hash (`f2e216695d4ce7233f5feb846bc81b8fffe9507988c7f5caaca680c0861e5e02`) in `Related configs` tab.

2. Go to the Preview tab

   Configurations are just a simple JSON objects. The only special thing is hashing algorithm e.g. lists are hashed non-orderwise, so if domains were ripped in different order, configuration hash will be still the same.

3. Go back to the Details tab. Expand `urls` and click on `www.discorddeno.land/suod/`

   ```
   cfg.urls*.url:"www.discorddeno.land/suod/"
   ```

   The resulting query looks for all `url` keys in `urls` lists that have `www.discorddeno.land/suod/`.

4. Let's check if `/suod/` path was used in other configs as well.

   Modify query to look for other configs with `/suod/` path replacing the domain with wildcard `*`.

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

7. Then click on `agenttesla` config (`e031b192d40f6d234756f8508f7d384db315983b57d8fc3216d20567056bd88b`) - you might have to scroll down a bit.

   ```{eval-rst}
   .. note::

      **Hint:** Instead of scrolling, you can help yourself by adding `AND family:agenttesla` to the query
   ```
   
   Ok, there is no .land TLD. but .landa e-mail address. To ilustrate how full-text search works, go to Preview, press CTRL-F and type ".land" to see what parts of JSON were matched

   How we can improve our query? Let's add `"` character at the end to match the end string.

   ```
   cfg:"*.land\"*"
   ```

   Go to the Gandcrab configuration and check in Preview what was matched.

## **Exercise #1.4**: Blobs and dynamic configurations

**Goal**: Familiarize yourself with the blob object type

The third object type in MWDB is blob. While config represents structured (JSON) data, blob is an unstructured one. Blobs are just simple text files, usually containing some raw, but human-readable content.

Let's take a look at some examples.

1. Navigate to [https://mwdb.cert.pl/blob/60c9ad80cde64e7cae9eec0c11dd98175860243aa40a3d8439bbf142d2a0e068](https://mwdb.cert.pl/blob/60c9ad80cde64e7cae9eec0c11dd98175860243aa40a3d8439bbf142d2a0e068)

   What we see is bunch of decrypted strings from AgentTesla that were ripped from the malware sample.

   They're not structured because we don't semantically analyze every string, but it's still nice to have them in repository.

2. Jump to [https://mwdb.cert.pl/blob/48914f0a6b9f4499da31d2217a7ee2e8c8f35f93ab5c992333f5c1aa947d9009](https://mwdb.cert.pl/blob/48914f0a6b9f4499da31d2217a7ee2e8c8f35f93ab5c992333f5c1aa947d9009)

   We're now looking at decrypted strings from the Remcos family. Even if data is unstructured, it can be considered a part of static configuration and used in searching for malware similarities.

   Let's take a look at the parent of this blob: the static configuration object.

   [https://mwdb.cert.pl/config/29c1f3c14a446b2a77ce58cbc59619fbfe7459c56fe1c8408597538384aa56ac](https://mwdb.cert.pl/config/29c1f3c14a446b2a77ce58cbc59619fbfe7459c56fe1c8408597538384aa56ac)

   Not much, just C2 host/port and password. 

3. Let's take a look for another configuration with this host by expanding `c2` key and clicking at the `host` address.

   Oh, there is another one.

   The resulting query is `cfg.c2*.host:"ongod4life.ddns.net:4344"`

   You should be looking at: [https://mwdb.cert.pl/config/9afac348443a7aa9ca5d33cffcc984751cebf15f065cb90b48911943fb10e1f6](https://mwdb.cert.pl/config/9afac348443a7aa9ca5d33cffcc984751cebf15f065cb90b48911943fb10e1f6)

   They're pretty much the same and only the `raw_cfg` differs. How to easily compare them?

4. Go to the blob (`da2055f0e90355bfaf3cc932f7fdb2f82bfd79c26f95b61b23b9cd77f9b0e32d`). In blob view, find the `Diff with` button on the right side of tabs. Click that button.

   Now we can choose another blob to compare.

   The simplest way is to copy to clipboard the previous blob id and paste it into query bar.

   `48914f0a6b9f4499da31d2217a7ee2e8c8f35f93ab5c992333f5c1aa947d9009`

   Then press ENTER and choose the searched blob. What's the difference between these blobs?

   ```{eval-rst}
   .. warning::

      This feature has known bug in v2.10.1, so go directly into this link: ` diff/48914f0a6b9f4499da31d2217a7ee2e8c8f35f93ab5c992333f5c1aa947d9009/da2055f0e90355bfaf3cc932f7fdb2f82bfd79c26f95b61b23b9cd77f9b0e32d <https://mwdb.cert.pl/diff/48914f0a6b9f4499da31d2217a7ee2e8c8f35f93ab5c992333f5c1aa947d9009/da2055f0e90355bfaf3cc932f7fdb2f82bfd79c26f95b61b23b9cd77f9b0e32d>`_
   ```
   But blobs are not only the strings and unstructured static things.

5. Go to the `Blobs` list and click on `dyn_cfg` in `Blob type` column or type manually `type:dyn_cfg`.
   Then filter out `dynamic:mirai` tag (there are lots of them but they're not that interesting).

6. Check out the `hancitor` dynamic configuration.

   [https://mwdb.cert.pl/blob/3b032876cc2d77d28625b9dfee0686663e60385cda7f9031afac6cf2b0c6d6e4](https://mwdb.cert.pl/blob/3b032876cc2d77d28625b9dfee0686663e60385cda7f9031afac6cf2b0c6d6e4)	

   Dynamic configurations also parametrize the malware behavior, but they're fetched from external source. In that case we have set of commands to run a second stage malware.

   Fetched second stage is linked as a blob child.

   Other possible types of dynamic configuration are injects, mail templates for spam botnets or malware updates.

   Example of more verbose Kronos dynamic configuration:

   [https://mwdb.cert.pl/blob/2e4d109edb8b2fa7c1f1d7592a284bbf15e3e51d24d1d9cdda91c9ae582cf05c/config](https://mwdb.cert.pl/blob/2e4d109edb8b2fa7c1f1d7592a284bbf15e3e51d24d1d9cdda91c9ae582cf05c/config)

   Blob children are files dropped from C&C and structured (parsed) fragments of dynamic configuration

## **Exercise #1.5**: Let's upload something!

**Goal**: Learn how object sharing and access inheritance work.

All objects you've seen so far are shared with all MWDB accounts: the 'public' group.

If you use a query:

```
NOT shared:public
```

You should not have any results, because all samples you see are public. So how to gather some 'private' samples? You need to upload them!

1. Fetch an example sample from GitHub `ex5malware.zip`. **Don't unpack it**, just download to some temporary location.

   ```
   $ wget https://github.com/CERT-Polska/training-mwdb/raw/main/ex5malware.zip
   ```

2. Click on `Upload` in the navbar ([https://mwdb.cert.pl/upload](https://mwdb.cert.pl/upload))

   Select the sample you have just downloaded.

3. Take a look at `Share with` options. There are four options:

   - `All my groups` sample will be shared with all your **private** groups, so it will be visible to the general public. E.g. it will be shared only within your organization
   - `Single group` in case you belong to multiple user groups, you can select a specific one
   - `Everybody`, everyone will see the sample
   - `Only me`, not event your colleagues from the same organization will be able to see the sample

4. Upload a sample with a default `All my groups` option.

5. Take a look at the Shares box. Who has access to your sample?

6. Use the Relations tab to traverse to the Config. Who has access to the configuration?

In MWDB sharing model, if you upload a private sample you get immediate access to all its descendants. So you always get all the data related with ripped configuration, but not necessarily all of its parents.

If you want, you can always change your mind and share the sample with somebody else. But you can't reverse the action, so if something was shared by mistake, contact the administrators.

7. Go back to the original sample and share it with `public` using `Share with group` input field.

