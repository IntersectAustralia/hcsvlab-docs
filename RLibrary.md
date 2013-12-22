Using the HCSvLab R Package
=======

The HCS vLab R package can be found at [this location](https://github.com/IntersectAustralia/hcsvlab-r)

It is a library which provides an interface in R to interact with the HCS vLab API.


## Installation

Clone the repository at https://github.com/IntersectAustralia/hcsvlab-r to your machine.
Then inside R run the following commands:

    # install dependencies
    install.packages("rjson")
    install.packages("RCurl")
    
    # install package
    install.packages("<path to hcsvlab-r>", repos=NULL, type="source")
    library(hcsvlab)
    
where `<path to hcsvlab-r>` is the directory previously cloned

To make use of this package you also need to download the hcsvlab.config file from the HCS vLab server you are using and place it in the home directory of your machine which are as follows:

    Mac: /Users/<user>
    Linux: /home/<user>
    Windows: C:/Users/<user>/Documents


## Using the Library

To get started we first create a client to interact with the appropriate server. This can be done as follows:

    client <- RestClient(server_uri="app.hscvlab.org.au")   # where app.hcsvlab.org.au is the server you are using
    
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

where uri is the full URI of the item list e.g. "http://app.hcsvlab.org.au/item_lists/1"

Example Response:

    Name: 
    [1] "MATT"
    URI: 
    [1] "http://localhost:3000/item_lists/20"
    Items: 
    [1] "http://localhost:3000/catalog/hcsvlab:5440"
    [2] "http://localhost:3000/catalog/hcsvlab:5442"
***    
    
To get an item list by ID (this returns an ItemList object):

    client$get_item_list_by_id(id)
    
where id is the ID of the item list e.g. 1. This returns the same as above.
***

To get a specific item by URI:

    client$get_item(uri)

where uri is the full URI of the item e.g. "http://app.hcsvlab.org.au/catalog/hcsvlab:100"

Example Response:

    ID: 
    [1] "mitcheldelbridge:S1219s1"
    URI: 
    [1] "http://localhost:3000/catalog/hcsvlab:5438"
***

To search items by metadata:

    client$search_metadata(query)
    
where query is a query string e.g. "HCSvLab_collection:cooee"
Example Response:

    $num_results
    [1] 1362
    
    $items
       [1] "http://localhost:3000/catalog/hcsvlab:102" 
       [2] "http://localhost:3000/catalog/hcsvlab:1002"
    ...
***

To download a set of items:

    client$download_items(items, destination, name)
    
where items is a list of item URIS (e.g. client$search_metadata("some query")$items, list("app.hcsvlab.org.au/catalog/hcsvlab:100", "app.hcsvlab.org.au/catalog/hcsvlab:101")), destination is the target directory (Note: in Windows make sure there is no trailing slash) and name is the name of the resulting zip file. Returns the filename of the resulting file

Example Response:

    [1] "/Users/hcsvlab/cooee.zip"
***
    
To create an item list:

    client$create_item_list(items, name)
    
where items is a list of item URIS (e.g. client$search_metadata("some query")$items, list("app.hcsvlab.org.au/catalog/hcsvlab:100", "app.hcsvlab.org.au/catalog/hcsvlab:101")) and name is the name of the item list to be created

Example Response:

    $success
    [1] "31 items added to new item list TEST"


### Item List 

    item_list <- client$get_item_list("http://app.hcsvlab.org.au/item_lists/1")

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
    [1] "http://localhost:3000/catalog/hcsvlab:5442"
***

To get a list of all documents of all items in the item list:

    item_list$get_item_documents()
    
Example Response:

    [1] "http://localhost:3000/catalog/hcsvlab:5440/document/S1220n.wav" 
    [2] "http://localhost:3000/catalog/hcsvlab:5442/document/S1220s1.wav"
***    

To download the item list:

    item_list$download(destination)
    
where destination is the target directory (Note: in Windows make sure there is no trailing slash). Returns the filename of the resulting file

Example Response:

    [1] "/Users/hcsvlab/cooee.zip"
***

To get the number of items in the item list:

    item_list$num_items()


### Item

    item <- item_list$get_item(1)
    item <- client$get_item("http://app.hcsvlab.org.au/catalog/hcsvlab:100")

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
    [1] "http://localhost:3000/catalog/hcsvlab:5442/document/S1220s1.wav"
    
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
    [1] "http://localhost:3000/catalog/hcsvlab:5440/document/S1220n.wav"
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
    [1] "http://localhost:3000/catalog/hcsvlab:5442/document/S1220s1.wav"
    
    
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
    
where destination is the target directory (Note: in Windows make sure there is no trailing slash). Returns the filename of the resulting file

Example Response:

    [1] "/Users/hcsvlab/S1220s1.wav"
