# Google Earth Engine Batch Asset Manager with Addons

[![PyPI version](https://badge.fury.io/py/geeadd.svg)](https://badge.fury.io/py/geeadd)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.1194308.svg)](https://doi.org/10.5281/zenodo.1194308)
[![Say Thanks!](https://img.shields.io/badge/Say%20Thanks-!-1EAEDB.svg)](https://saythanks.io/to/samapriya)

Google Earth Engine Batch Asset Manager with Addons is an extension of the one developed by Lukasz [here](https://github.com/tracek/gee_asset_manager) and additional tools were added to include functionality for moving assets, conversion of objects to fusion table, cleaning folders, querying tasks. The ambition is apart from helping user with batch actions on assets along with interacting and extending capabilities of existing GEE CLI. It is developed case by case basis to include more features in the future as it becomes available or as need arises.

![CLI](https://i.imgur.com/04iZQlK.gif)

## Table of contents
* [Installation](#installation)
* [Getting started](#getting-started)
    * [Batch uploader](#batch-uploader)
    * [Parsing metadata](#parsing-metadata)
* [Usage examples](#usage-examples)
	* [EE User](#ee-user)
	* [Create](#create)
    * [Upload a directory with images and associate properties with each image:](#upload-a-directory-with-images-and-associate-properties-with-each-image)
	* [Upload a directory with images with specific NoData value to a selected destination:](#upload-a-directory-with-images-with-specific-nodata-value-to-a-selected-destination)
	* [Asset List](#asset-list)
	* [Asset Size](#asset-size)
    * [Earth Engine Asset Report](#earth-engine-asset-report)
	* [Task Query](#task-query)
	* [Task Report](#task-report)
    * [Delete a collection with content:](#delete-a-collection-with-content)
	* [Assets Move](#assets-move)
	* [Assets Copy](#assets-copy)
	* [Assets Access](#assets-access)
	* [Set Collection Property](#set-collection-property)
	* [Cancel all tasks](#cancel-all-tasks)

## Installation
We assume Earth Engine Python API is installed and EE authorised as desribed [here](https://developers.google.com/earth-engine/python_install). To install:
```
git clone https://github.com/samapriya/gee_asset_manager_addon
cd gee_asset_manager_addon && pip install -r requirements.txt
python setup.py install
```

Installation is an optional step; the application can be also run
directly by executing geeadd.py script. The advantage of having it
installed is being able to execute geeadd as any command line tool. I
recommend installation within virtual environment. To install run
```
python setup.py develop or python setup.py install

In a linux distribution
sudo python setup.py develop or sudo python setup.py install
```


## Getting started

As usual, to print help:
```
usage: geeadd.py [-h]
                 {ee_user,create,upload,lst,ee_report,collsize,tasks,taskquery,report,delete,mover,copy,access,collprop,cleanout,cancel}
                 ...

Google Earth Engine Batch Asset Manager with Addons

positional arguments:
  {ee_user,create,upload,lst,ee_report,assetsize,tasks,taskreport,delete,mover,copy,access,collprop,cancel}
    ee_user             Allows you to associate/change GEE account to system
    create              Allows the user to create an asset collection or
                        folder in Google Earth Engine
    upload              Batch Asset Uploader.
    lst                 List assets in a folder/collection or write as text
                        file
    ee_report           Prints a detailed report of all Earth Engine Assets
                        includes Asset Type, Path,Number of
                        Assets,size(MB),unit,owner,readers,writers
    assetsize           Prints collection size in Human Readable form & Number
                        of assets
    tasks               Queries current task status
                        [completed,running,ready,failed,cancelled]
    taskreport          Create a report of all tasks and exports to a CSV file
    delete              Deletes collection and all items inside. Supports
                        Unix-like wildcards.
    mover               Moves all assets from one collection to another
    copy                Copies all assets from one collection to another:
                        Including copying from other users if you have read
                        permission to their assets
    access              Sets Permissions for Images, Collection or all assets
                        in EE Folder Example: python ee_permissions.py --mode
                        "folder" --asset "users/john/doe" --user
                        "jimmy@doe.com:R"
    collprop            Sets Overall Properties for Image Collection
    cancel              Cancel all running tasks

optional arguments:
  -h, --help            show this help message and exit

```

To obtain help for a specific functionality, simply call it with _help_
switch, e.g.: `geeadd upload -h`. If you didn't install geeadd, then you
can run it just by going to _geeadd_ directory and running `python
geeadd.py [arguments go here]`

## Batch uploader
The script creates an Image Collection from GeoTIFFs in your local
directory. By default, the collection name is the same as the local
directory name; with optional parameter you can provide a different
name. Another optional parameter is a path to a CSV file with metadata
for images, which is covered in the next section:
[Parsing metadata](#parsing-metadata).



```
usage: geeadd.py upload [-h] --source SOURCE --dest DEST [-m METADATA]                    
                        [--large] [--nodata NODATA] [--bands BANDS] [-u USER]             
                        [-b BUCKET]                                                       
                                                                                          
optional arguments:                                                                       
  -h, --help            show this help message and exit                                   
                                                                                          
Required named arguments.:                                                                
  --source SOURCE       Path to the directory with images for upload.                     
  --dest DEST           Destination. Full path for upload to Google Earth                 
                        Engine, e.g. users/pinkiepie/myponycollection                     
  -u USER, --user USER  Google account name (gmail address).                              
                                                                                          
Optional named arguments:                                                                 
  -m METADATA, --metadata METADATA                                                        
                        Path to CSV with metadata.                                        
  --large               (Advanced) Use multipart upload. Might help if upload             
                        of large files is failing on some systems. Might cause            
                        other issues.                                                     
  --nodata NODATA       The value to burn into the raster as NoData (missing              
                        data)                                                             
  --bands BANDS         Comma-separated list of names to use for the image                
                        bands. Spacesor other special characters are not                  
                        allowed.                                                          
  -b BUCKET, --bucket BUCKET                                                              
                        Google Cloud Storage bucket name.                                 
                                                                                          
```

### Parsing metadata
By metadata we understand here the properties associated with each image. Thanks to these, GEE user can easily filter collection based on specified criteria. The file with metadata should be organised as follows:

| filename (without extension) | property1 header | property2 header |
|------------------------------|------------------|------------------|
| file1                        | value1           | value2           |
| file2                        | value3           | value4           |

Note that header can contain only letters, digits and underscores.

Example:

| id_no     | class      | category | binomial             |system:time_start|
|-----------|------------|----------|----------------------|-----------------|
| my_file_1 | GASTROPODA | EN       | Aaadonta constricta  |1478943081000    |
| my_file_2 | GASTROPODA | CR       | Aaadonta irregularis |1478943081000    |

The corresponding files are my_file_1.tif and my_file_2.tif. With each of the files five properties are associated: id_no, class, category, binomial and system:time_start. The latter is time in Unix epoch format, in milliseconds, as documented in GEE glosary. The program will match the file names from the upload directory with ones provided in the CSV and pass the metadata in JSON format:

```
{ id_no: my_file_1, class: GASTROPODA, category: EN, binomial: Aaadonta constricta, system:time_start: 1478943081000}
```

The program will report any illegal fields, it will also complain if not all of the images passed for upload have metadata associated. User can opt to ignore it, in which case some assets will have no properties.

Having metadata helps in organising your asstets, but is not mandatory - you can skip it.


## Usage examples

### EE User
This tool is designed to allow different users to change earth engine authentication credentials. The tool invokes the authentication call and copies the authentication key verification website to the clipboard which can then be pasted onto a browser and the generated key can be pasted back

### Create
This tool allows you to create a collection or folder in your earth engine root directory. The tool uses the system cli to achieve this and this has been included so as to reduce the need to switch between multiple tools and CLI.
```
usage: geeadd.py create [-h] --typ TYP --path PATH

optional arguments:
  -h, --help   show this help message and exit
  --typ TYP    Specify type: collection or folder
  --path PATH  This is the path for the earth engine asset to be created full
               path is needsed eg: users/johndoe/collection
```
### Upload a directory with images to your myfolder/mycollection and associate properties with each image:
```
geeadd upload -u johndoe@gmail.com --source path_to_directory_with_tif -m path_to_metadata.csv --dest users/johndoe/myfolder/myponycollection
```
The script will prompt the user for Google account password. The program will also check that all properties in path_to_metadata.csv do not contain any illegal characters for GEE. Don't need metadata? Simply skip this option.

### Upload a directory with images with specific NoData value to a selected destination
```
geeadd upload -u johndoe@gmail.com --source path_to_directory_with_tif --dest users/johndoe/myfolder/myponycollection --nodata 222
```
In this case we need to supply full path to the destination, which is helpful when we upload to a shared folder. In the provided example we also burn value 222 into all rasters for missing data (NoData).

### Asset List
This tool is designed to either print or output asset lists within folders or collections using earthengine ls tool functions.
```
usage: geeadd.py lst [-h] --location LOCATION --typ TYP [--items ITEMS]
                     [--output OUTPUT]

optional arguments:
  -h, --help           show this help message and exit

Required named arguments.:
  --location LOCATION  This it the location of your folder/collection
  --typ TYP            Whether you want the list to be printed or output as
                       text[print/report]

Optional named arguments:
  --items ITEMS        Number of items to list
  --output OUTPUT      Folder location for report to be exported
```

### Asset Size
This tool allows you to query the size of any Earth Engine asset[Images, Image Collections, Tables and Folders] and prints out the number of assets and total asset size in non-byte encoding meaning KB, MB, GB, TB depending on size.

```
usage: geeadd assetsize [-h] --asset ASSET

optional arguments:
  -h, --help     show this help message and exit
  --asset ASSET  Earth Engine Asset for which to get size properties
```

### Earth Engine Asset Report
This tool recursively goes through all your assets(Includes Images, ImageCollection,Table,) and generates a report containing the following fields
[Type,Asset Type, Path,Number of Assets,size(MB),unit,owner,readers,writers].

```
usage: geeadd.py ee_report [-h] --outfile OUTFILE

optional arguments:
  -h, --help         show this help message and exit
  --outfile OUTFILE  This it the location of your report csv file
```
A simple setup is the following
``` geeadd --outfile "C:\johndoe\report.csv"```

### Task Query
This script counts all currently running and ready tasks along with failed tasks.
```
usage: geeadd.py tasks [-h]

optional arguments:
  -h, --help  show this help message and exit

geeadd.py tasks
```

### Task Report
Sometimes it is important to generate a report based on all tasks that is running or has finished. Generated report includes taskId, data time, task status and type
```
usage: geeadd taskreport [-h] [--r R]

optional arguments:
  -h, --help  show this help message and exit
  --r R       Folder Path where the reports will be saved
```

### Delete a collection with content:

The delete is recursive, meaning it will delete also all children assets: images, collections and folders. Use with caution!
```
geeadd delete users/johndoe/test
```

Console output:
```
2016-07-17 16:14:09,212 :: oauth2client.client :: INFO :: Attempting refresh to obtain initial access_token
2016-07-17 16:14:09,213 :: oauth2client.client :: INFO :: Refreshing access_token
2016-07-17 16:14:10,842 :: root :: INFO :: Attempting to delete collection test
2016-07-17 16:14:16,898 :: root :: INFO :: Collection users/johndoe/test removed
```

### Delete all directories / collections based on a Unix-like pattern

```
geeadd delete users/johndoe/*weird[0-9]?name*
```

### Assets Move
This script allows us to recursively move assets from one collection to the other.
```
usage: geeadd.py mover [-h] [--assetpath ASSETPATH] [--finalpath FINALPATH]

optional arguments:
  -h, --help            show this help message and exit
  --assetpath ASSETPATH
                        Existing path of assets
  --finalpath FINALPATH
                        New path for assets
geeadd.py mover --assetpath "users/johndoe/myfolder/myponycollection" --destination "users/johndoe/myfolder/myotherponycollection"
```

### Assets Copy
This script allows us to recursively copy assets from one collection to the other. If you have read acess to assets from another user this will also allow you to copy assets from their collections.
```
usage: geeadd.py copy [-h] [--initial INITIAL] [--final FINAL]

optional arguments:
  -h, --help         show this help message and exit
  --initial INITIAL  Existing path of assets
  --final FINAL      New path for assets
geeadd.py mover --initial "users/johndoe/myfolder/myponycollection" --final "users/johndoe/myfolder/myotherponycollection"
```

### Assets Access
This tool allows you to set asset acess for either folder , collection or image recursively meaning you can add collection access properties for multiple assets at the same time.
```
usage: geeadd access [-h] --mode MODE --asset ASSET --user USER

optional arguments:
  -h, --help     show this help message and exit
  --mode MODE    This lets you select if you want to change permission or
                 folder/collection/image
  --asset ASSET  This is the path to the earth engine asset whose permission
                 you are changing folder/collection/image
  --user USER    This is the email address to whom you want to give read or
                 write permission Usage: "john@doe.com:R" or "john@doe.com:W"
                 R/W refers to read or write permission
geeadd.py access --mode folder --asset "folder/collection/image" --user "john@doe.com:R"
```

### Set Collection Property
This script is derived from the ee tool to set collection properties and will set overall properties for collection.
```
usage: geeadd.py collprop [-h] [--coll COLL] [--p P]

optional arguments:
  -h, --help   show this help message and exit
  --coll COLL  Path of Image Collection
  --p P        "system:description=Description"/"system:provider_url=url"/"sys
               tem:tags=tags"/"system:title=title
```

### Cancel all tasks
This is a simpler tool, can be called directly from the earthengine cli as well
```
earthengine cli command
earthengine task cancel all

usage: geeadd.py cancel [-h]

optional arguments:
  -h, --help  show this help message and exit
```

### Changelog

#### v0.2.5
- Handles bandnames during upload thanks to Lukasz for original upload code
- Removed manifest option, that can be handled by seperate tool (ppipe)
#### v0.2.3
- Removing the initialization loop error

#### v0.2.2
- Added improvement to earthengine authorization

#### v0.2.1
- Added capability to handle PlanetScope 4Band Surface Reflectance Metadata Type
- General Improvements

#### v0.2.0
- Tool improvements and enhancements

#### v0.1.9
- New tool EE_Report was added

#### v0.1.8
- Fixed issues with install
- Dependencies now part of setup.py
- Updated Parser and general improvements
