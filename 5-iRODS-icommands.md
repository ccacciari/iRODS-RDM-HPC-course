<img align="right" src="images/surf.jpg" width="100px">
<br><br>


# iRODS basic data handling 

**Authors**
- Arthur Newton (SURF)
- Claudio Cacciari (SURF)
- Maithili Kalamkar Stam (SURF)

**License**
Copyright (c) 2020, SURFsara. All rights reserved.
This project is licensed under the GPLv3 license.
The full license text can be found in [LICENSE](LICENSE).


## 1. Goal

Learning about the CLI tool icommands of iRODS and do basic data handling of data stored in an iRODS instance:

- uploading/downloading data
- adding metadata to data objects/collections
- querying based on metadata

### What we expect you to do
You can comfortably go through this exercise on your own and more importantly, at your own pace. We will take stops in intervals to see if we are all more or less on the same page. In the end we can discuss together the 'Foof for Brain' questions. You can of course always ask questions or help if you cannot proceed.

## 2. Prerequisites

- an account on Lisa or Snellius system at SURF


## 3. Login to Lisa

In this course we will use the Lisa login node as our user interface. If you don't have access to Lisa or Cartesius system and you want to only use the basic data handling of iRODS without the SLURM data processing, you have to install the icommands on your local machine. 

Login to the Lisa compute cluster with your appropriate credentials:

```sh
ssh lcur#@lisa.surfsara.nl
```

Since 2020, the icommands are installed on Lisa (and Spider) system wide. 


## 4. Clone course repository

If you have not done it yet, clone this repository to your home folder:

```sh
git clone https://github.com/ccacciari/iRODS-RDM-HPC-course.git
cd iRODS-RDM-HPC-course
```

Otherwise skipt this step.

## 5. Connecting to iRODS

Please if you have loaded the irods module in a previous tutorial/exercise, unload it:

```sh
module unload iRODS-iCommands/4.3.0
```

If you now use the command `ienv`, you should see somethiong like this: 

```sh
ienv
irods_version - 4.2.8
...
```

To connect to iRODS you need to create the environment file within your homefolder of Lisa. 
The environment file in the git repository can be used:

```sh
mkdir -p ~/.irods
cp irods_environment.json ~/.irods/irods_environment.json
```

Use your favourite text editor (nano or vi) to change your username.

```sh
    "irods_user_name": "YOUR DEMO ACCOUNT NAME",
```

Now initialize the connection:

```sh
iinit
```

Enter your password and you should be logged into iRODS.
A scrambled password file, `~/.irods/.irodsA`, will be created which will be used for subsequent connections.

To test your connection issue a simple command:

```sh
ils
```

You should see the listing of the collection defined as the home folder in your environment file, `irods_home`.


## 6. General help icommands

You can find help for the icommands with:

```sh
ihelp
```

which will show you all the possible commands. 

Additionally, the following command gives you more information about the iRODS environment (typically not needed):

```sh
ienv
```

## 7. Basic data/collection handling

Note that in iRODS files are called data objects, and folders are called collections. 

The icommands have basic file handling functionality that have their (almost) equivalent in Linux terminal programs but with an `i` in front of the name, *e.g.* `ls` and `ils`, `cp` and `icp`, `mv` and `imv`, `pwd` and `ipwd`, `mkdir` and `imkdir`, `cd` and `icd`,  `rm` and `irm`. 
If you want to know more about the available icommands use for example `ils -h`.
However, please do realize iRODS is not a filesystem like you have on your personal computer.
Data handling is done differently.


### 7.1 Create a collection and navigate

Use `ils` to see what collections are available to you and navigate to that collection with `icd`:

```sh
ils

icd <collection name>
```

Note that you can't use bash tab completion with paths in iRODS.

You can create a collection with `imkdir`.
For this course it is convenient to create a collection for yourself:

```sh
imkdir <favorite color>_<username>

icd <favorite color>_<username>
```

If you get an error that you are not allowed to create a collection, be sure to have navigated to a collection where you do have the permission to write.
You can use:

```sh
ils -A 
```

to check the permissions (`-A` stands for Access Control Lists) of data objects and collections.

> **_Food for brain:_**
>
> * If you run a command 'ils /' would you see all the collections?
> * Can different collections be at different physical locations/storage backends?
> * Within a collection can you have objects stored at different physical locations/storage backends?
> * How do you know what permissions you have on a collection?


### 7.2 Uploading a file or folder

In order to put a file in iRODS:

```sh
iput source_file destination_dataobject
```

when no destination data object is given, the local filename will be used.
iRODS will issue a warning if the file already exists (which you can force to do with the `-f` option).

In order to put a folder in iRODS use the recursive option:

```sh
iput -r source_folder destination_collection
```

You need to add the `-r` option for recursively putting data in iRODS.


Depending on the policy of the iRODS server, you can also immediately calculate the checksum while uploading a file:

```sh
iput -K source_file
```

Note that his can also be done afterwards with the `ichksum` command. 

There are many more options for `iput` which could (or could not) optimize data transfer, *e.g.* `-b` for bulk upload to overcome network overhead and `-N` for number of threads.
However, the default behaviour is already high performant in most cases if the iRODS server is configured correctly.

You could check with `ils` what has happened. 
What does `ils -l` or `ils -L` show? 


#### 7.2.1 Exercise

- Upload `alice.txt` to your home directory in iRODS
- Create a new collection `aliceInWonderland` within your home directory
- Move `alice.txt` into this new collection.

> **_Food for brain:_**
>
> * Should/can you use iput and icp commands interchangeably, when, why and why not?

### 7.3 Downloading a data object or collection

To download a data object, you can use the `iget` command:

```sh
iget source_dataobject destination_file
```

Again, if the local destination is not specified, the name of the data object will be used. 
If the filename already exists, `iget` will result in an error.

To download a collection you have to again specify the `-r` option:

```sh
iget -r source_collection destination_folder
```


#### 7.3.1 Exercise

- download the data object `alice.txt` as `aliceRestore.txt`
- download the collection `aliceInWonderland`

> **_Food for brain:_**
>
> * With the iget command, can you put data to another remote location instead of the Lisa login node?

## 8. Removing files and the trashbin

You can remove files using `irm`:

```sh
irm dataobject
```

It depends on the configured policy of the iRODS instance whether there is a trashbin. 
Note that removing a file is just a rename.
If you really want to delete a file either use `irmtrash` after removing or `irm -f` upon removing a data object.

#### 8.1 Optional Exercise

- how would you be able find the removed file back?

> **_Food for brain:_**
>
> * If you run the command 'ils /surfZone1', what do you see?
> * Can you find your deleted files here?
> * Can you see another user's deleted data as well?

## 9. Adding metadata and querying for data

In iRODS a data object is not only the bitstream and the filename, but user defined metadata is part of the data object. 
Metadata is essential to give a data object more context.
You can add all kinds of metadata to a data object: administrative metadata (about the management, processing of the data), descriptive metadata (about the content, author, etc) or provenance metadata (about the history of the data object).
This makes it very powerful, as metadata and data can not be out of sync. 
You can manually add metadata, which can also be scripted or let an iRODS rule add metadata. 

In iRODS, per metadata item you can store three strings: key, value, unit. You can dismiss the unit string, but the key/value pair needs to be unique.


### 9.1 Metadata handling

Manual metadata handling can be done via the `imeta` command. 
For each command, -d, -C, -R, or -u is used to specify which type of object to work with: data objects, collections, resources, or users, respectively.
To add metadata to a data object or a collection:

```sh
imeta add -d dataobject Key Val Unit

imeta add -C collection Key Val Unit
```

Again, it is possible to leave out the unit, and you can set multiple key value pairs with the same key. 
There is no logical limit to the number of metadata items per data object or collection. 

You can remove metadata as such:

```sh
imeta rm -d dataobject Key Val Unit

imeta rm -C collection Key Val Unit
```

where you do have to explicitly specify the whole set of \{key, value, unit\} to remove it.
You can use wildcards (`%`) with `imeta rmw`.
You can also modify, copy, show metadata by the `imeta mod`, `imeta cp`, `imeta ls` commands respectively:

```sh
imeta mod -d dataobject key1 val1 unit1 n:key2 v:diffval2 u:unit2

imeta cp -C -C collection1 collection2

imeta ls -C collection
```

You can also immediately add metadata to the data object or collection upon upload:

```sh
iput source_file --metadata "key1;val1;unit1;key2;val2;unit2"
```


```sh
iput source_file --metadata "key1;val1;;key2;val2;unit2"
```

#### 9.1.1 Exercise 

- add metadata to alice.txt
- add metadata to the `aliceInWonderland` collection
- create a new file locally (`echo "testing metadata" > lorem.txt`), upload file and add metadata in one go


### 9.2 Querying based on metadata

Metadata attached to data objects and collections becomes very useful when you want to query for data.
The `iquest` command is used to query data objects/collections.
It uses a SQL like syntax.
Do note that not all SQL options are available by default, **e.g.** a join is not possible.

You can look up the available keys you can query for via:

```sh
iquest attrs
```

Note that there are a lot.

General `iquest` queries look like:

```sh
iquest "select COLL_NAME, DATA_NAME, META_DATA_ATTR_VALUE where META_DATA_ATTR_NAME like 'key1'" 
```

In above command we wanted to look for everything which has the metadata attribute name (metadata key) that resembles 'key1' and retrieve the collection name, data object name and metadata attribute value (metadata value). 
iRODS will respond something like:

```sh
COLL_NAME = /surfZone1/home/irods-user1
DATA_NAME = testfile
META_DATA_ATTR_VALUE = val1
------------------------------------------------------------
```

You can change the format of the iRODS response by using a C like format string after `iquest` like so:

```sh
iquest "User %-6.6s has %-5.5s access to file %s" "SELECT USER_NAME,  DATA_ACCESS_NAME, DATA_NAME WHERE COLL_NAME = '/surfZone1/home/irods-user1'"
```

where after `iquest` there is the format string and the second string is again the SQL like query.
This can be convenient for making reports but also for retrieving absolute filepaths:

```sh
iquest "%s/%s" "select COLL_NAME, DATA_NAME where META_DATA_ATTR_NAME like 'TEMPERATURE' and META_DATA_ATTR_VALUE > '300'" 
```

This could be useful in concatenating commands in HPC data staging which we will be using in the following section:

```sh
iquest "%s/%s" "select COLL_NAME, DATA_NAME where META_DATA_ATTR_NAME like 'key1' and META_DATA_ATTR_VALUE = 'val1'" | xargs iget
```

#### 9.2.1 Exercise
- try to find the files you have added above by searching for the associated metadata you added
- search for files with `META_DATA_ATTR_NAME` is `author` and `META_DATA_ATTR_VALUE` is `Lewis Carroll`. Do you know these files?

## 10. What next?
Now that you know the basic data handling in iRODS, you can follow the next section which is about setting up a data processing pipeline while still retaining data provenance with the data handling tool discussed in this section [iRODS in HPC](7-iRODS-in-HPC.md).

