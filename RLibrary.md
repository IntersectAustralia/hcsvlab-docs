Using the Alveo R Package
=======

The HCS vLab R package can be found at [this location](https://github.com/IntersectAustralia/hcsvlab-r)

It is a library which provides an interface in R to interact with the HCS vLab API.


## Installation

Download the appropriate binary from the list below to your machine.

| Package | Mac | Linux (Centos) | Windows |
| ------  | --- | -------------- | ------- |
| alveo   | [alveo_1.0.1.tgz](https://github.com/IntersectAustralia/hcsvlab-docs/blob/master/RPackages/alveo_1.0.1.tgz) | [alveo_1.0.1_R_x86_64-redhat-linux-gnu.tar.gz](https://github.com/IntersectAustralia/hcsvlab-docs/blob/master/RPackages/alveo_1.0.1_R_x86_64-redhat-linux-gnu.tar.gz) | [alveo_1.0.1.zip](https://github.com/IntersectAustralia/hcsvlab-docs/blob/master/RPackages/alveo_1.0.1.zip) |

Older versions of the binaries can be found in dated directories at [this page](https://github.com/IntersectAustralia/hcsvlab-docs/tree/master/RPackages)

Then inside R run the following commands:

    # install dependencies
    install.packages("rjson")
    install.packages("RCurl")
    
    # install package
    install.packages("<path to binary>")
    library(alveo)
    
where `<path to binary>` is the path to the binary downloaded above

To make use of this package you also need to download the alveo.config file from the HCS vLab server you are using and place it in the home directory of your machine which are as follows:

    Mac: /Users/<user>
    Linux: /home/<user>
    Windows: C:/Users/<user>/Documents


## Using the Library

To get started we first create a client to interact with the appropriate server. This can be done as follows:

    client <- RestClient(server_uri="app.hscvlab.org.au")   # where app.alveo.org.au is the server you are using
    
### Client

To get the API version:

    client$get_api_version()
    
Example Response:

    "V2.3"
***

To get the context of the annotations:

    client$get_annotation_context()
    
Example Response:

    $`@context`
    $`@context`$`@base`
    [1] "http://purl.org/dada/schema/0.2/"
    
    $`@context`$annotations
    $`@context`$annotations$`@id`
    [1] "http://purl.org/dada/schema/0.2/annotations"
    ...
***

To get item lists:

    client$get_item_lists()
    
Example Response:

    [[1]]
    [[1]]$name
    [1] "md2"
    
    [[1]]$item_list_url
    [1] "http://localhost:3000/item_lists/15"
    
    [[1]]$num_items
    [1] 2    
    ...
***
    
To get an item list by URI (this returns an ItemList object):

    client$get_item_list(uri)

where uri is the full URI of the item list e.g. "http://app.alveo.org.au/item_lists/1"

Example Response:

    Name: 
    [1] "MATT"
    URI: 
    [1] "http://localhost:3000/item_lists/20"
    Items: 
    [1] "http://localhost:3000/catalog/mitcheldelbridge:S1228s1"
    [2] "http://localhost:3000/catalog/mitcheldelbridge:S1228n"
***    
    
To get an item list by ID (this returns an ItemList object):

    client$get_item_list_by_id(id)
    
where id is the ID of the item list e.g. 1. This returns the same as above.
***

To get a specific item by URI:

    client$get_item(uri)

where uri is the full URI of the item e.g. "http://app.alveo.org.au/catalog/mitcheldelbridge:S1228s1"

Example Response:

    ID: 
    [1] "mitcheldelbridge:S1219s1"
    URI: 
    [1] "http://localhost:3000/catalog/mitcheldelbridge:S1219s1"
***

To search items by metadata:

    client$search_metadata(query)
    
where query is a query string e.g. "HCSvLab_collection:cooee"
Example Response:

    $num_results
    [1] 1362
    
    $items
       [1] "http://localhost:3000/catalog/mitcheldelbridge:S1228s1" 
       [2] "http://localhost:3000/catalog/mitcheldelbridge:S1229s1"
    ...
***

To download a set of items:

    client$download_items(items, destination, name)
    
where items is a list of item URIS (e.g. client$search_metadata("some query")$items, list("app.alveo.org.au/catalog/mitcheldelbridge:S1228s1", "app.alveo.org.au/catalog/mitcheldelbridge:S1228n")), destination is the target directory and name is the name of the resulting zip file. Returns the filename of the resulting file

Example Response:

    [1] "/Users/alveo/cooee.zip"
***
    
To create an item list:

    client$create_item_list(items, name)
    
where items is a list of item URIS (e.g. client$search_metadata("some query")$items, list("app.alveo.org.au/catalog/mitcheldelbridge:S1228s1", "app.alveo.org.au/catalog/mitcheldelbridge:S1228n")) and name is the name of the item list to be created

Example Response:

    $success
    [1] "31 items added to new item list TEST"


### Item List 

    item_list <- client$get_item_list("http://app.alveo.org.au/item_lists/1")

To get a list of items in the item list:

    item_list$items
***

To get a particular item from the item list:

    item_list$get_item(index)
    
where index is a 1 based index of the item from the list of items above

Example Response:

    ID: 
    [1] "mitcheldelbridge:S1220s1"
    URI: 
    [1] "http://localhost:3000/catalog/mitcheldelbridge:S1220s1"
***

To get a list of all documents of all items in the item list:

    item_list$get_item_documents()
    
Example Response:

    [1] "http://localhost:3000/catalog/mitcheldelbridge:S1220n/document/S1220n.wav" 
    [2] "http://localhost:3000/catalog/mitcheldelbridge:S1220s1/document/S1220s1.wav"
***    

To download the item list:

    item_list$download(destination)
    
where destination is the target directory. Returns the filename of the resulting file

Example Response:

    [1] "/Users/alveo/cooee.zip"
***

To get an EMU segment list with segments from all annotations in item list:

    item_list$get_segment_list()

Example Response:

       labels start  end
    1       #     0  330
    2    beat   330  730
    3       #   730 1310
    4    boot  1310 1770
    ...
***

To get the number of items in the item list:

    item_list$num_items()


### Item

    item <- item_list$get_item(1)
    item <- client$get_item("http://app.alveo.org.au/catalog/mitcheldelbridge:S1228s1")

To get metadata for an item:

    item$get_metadata()
    
Example Response:

    $`dc:identifier`
    [1] "S1220s1"
    
    $`dc:isPartOf`
    [1] "mitcheldelbridge"
    
    $`ausnc:mode`
    [1] "unspecified"
    ...
***

To get the indexable text for an item:

    item$get_indexable_text()
    
Returns empty string if no indexable text found
***

To get list of documents for an item:

    item$get_documents()
    
Example Response:

    [[1]]
    [[1]]$url
    [1] "http://localhost:3000/catalog/mitcheldelbridge:S1220s1/document/S1220s1.wav"
    
    [[1]]$`dc:type`
    [1] "Audio"
    
    [[1]]$size
    NULL
    ...
***

To get a specific document from an item:

    item$get_document(index)

where index is a 1 based index of the document from the list of documents above

Example Response:

    URI: 
    [1] "http://localhost:3000/catalog/mitcheldelbridge:S1220n/document/S1220n.wav"
    Type: 
    [1] "Audio"
    Size: 
    [1] "8.0 MB"
***

To get list of annotations for an item:

    item$get_annotations(type=NULL, label=NULL)

where type and label are optional arguments to narrow down the annotations returned

Example Response:

    $`@context`
    [1] "http://localhost:3000/schema/json-ld"
    
    $commonProperties
    $commonProperties$annotates
    [1] "http://localhost:3000/catalog/mitcheldelbridge:S1220s1/document/S1220s1.wav"
    
    
    $annotations
    $annotations[[1]]
    $annotations[[1]]$`@type`
    [1] "SecondAnnotation"
    
    $annotations[[1]]$`@id`
    [1] "http://ns.ausnc.org.au/corpora/mitcheldelbridge/annotation/28499"
    ...



### Document

    document <- item$get_document(1)
    
To get content of a document:

    document$get_content()

Note that this may return binary data
***

To download the document:

    document$download(destination)
    
where destination is the target directory. Returns the filename of the resulting file

Example Response:

    [1] "/Users/alveo/S1220s1.wav"


## Other Packages

We also have two other packages which can be used with data from the HCS vLab. These are the wrassp and emuSX libraries. 
### Installation

Download the appropriate binary from the list below to your machine.

| Package | Mac | Linux (Centos) | Windows |
| ------  | --- | -------------- | ------- |
| wrassp | [wrassp_0.0.3.tgz](https://github.com/IntersectAustralia/hcsvlab-docs/blob/master/RPackages/wrassp_0.0.3.tgz) | [wrassp_0.0.3_R_x86_64-redhat-linux-gnu.tar.gz](https://github.com/IntersectAustralia/hcsvlab-docs/blob/master/RPackages/wrassp_0.0.3_R_x86_64-redhat-linux-gnu.tar.gz) | [wrassp_0.0.3.zip](https://github.com/IntersectAustralia/hcsvlab-docs/blob/master/RPackages/wrassp_0.0.3.zip) |
| emuSX | [emuSX_0.0.8.tgz](https://github.com/IntersectAustralia/hcsvlab-docs/blob/master/RPackages/emuSX_0.0.8.tgz) | [emuSX_0.0.8_R_x86_64-redhat-linux-gnu.tar.gz](https://github.com/IntersectAustralia/hcsvlab-docs/blob/master/RPackages/emuSX_0.0.8_R_x86_64-redhat-linux-gnu.tar.gz) | [emuSX_0.0.8.zip](https://github.com/IntersectAustralia/hcsvlab-docs/blob/master/RPackages/emuSX_0.0.8.zip) |

Then inside R run the following commands:

    # install dependencies
    install.packages("rjson")
    install.packages("RCurl")
    install.packages('testthat')
    install.packages("websockets")
    install.packages("base64enc")
    
    # install package
    install.packages("<path to binary>")
    library(emuSX)
    
where `<path to binary>` is the path to the binary downloaded above

### wrassp

wrassp provides a number of functions to perform signal processing on audio files. It is able to handle URI's from the HCS vLab provided the alveo.config file as discussed at the top of this document is in place. An example usage of one of these functions is as follows:

    forest("http://app.alveo.org.au/catalog/mitcheldelbridge:S1223s1/document/S1223s1.wav", ToFile=FALSE)
    
All functions produce the following summary (with slight changes) on completion:

    In-memory Assp Data Object
    Format: WAVE (binary)
    67830 records at 16000 Hz
    Duration: 4.239375 ms
    Number of tracks: 1 
    	 audio (1 fields)

The wrassp library comes with caching functionality where it downloads URI content to a cache directory so it can be pulled straight from there the next time that URI is called. To view the contents of the cache directory simply call `viewCache()`. This opens up the cache contents file to display all the files stored in the local cache and the URIs they are mapped with. The local filenames are computated by taking an MD5 hash of the entire URI it corresponds to and appending the appropriate file extension.

    <URI> -> <MD5 has of URI + file extension>
    e.g.
    http://www.phonetik.uni-muenchen.de/~raphael/data/msajc003.wav -> 40586c2ce57890eb058efd6a4d00cf3f.wav

To empty the entire cache, call `emptyCache()` while to remove a single item from the cache call:

    removeItemFromCache("40586c2ce57890eb058efd6a4d00cf3f.wav")

which takes a single argument, that of the local filename stored in the cache.

### emuSX

emuSX is now a required package for the alveo package and is required to be installed before being able to use it