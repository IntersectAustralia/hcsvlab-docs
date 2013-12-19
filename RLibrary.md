Using the HCSvLab R Package
=======

The HCS vLab R package can be found at [this location](https://github.com/IntersectAustralia/hcsvlab-r)

It is a library which provides an interface in R to interact with the HCS vLab API.


## Installation

Clone the repository at https://github.com/IntersectAustralia/hcsvlab-r to your machine.
Then inside R run the following commands:

    install.packages("<path to hcsvlab-r>", repos=NULL, type="source")
    library(hcsvlab)
    
where `<path to hcsvlab-r>` is the directory previously cloned


## Using the Library

To get started we first create a client to interact with the appropriate server. This can be done as follows:

    client <- RestClient(server_uri="app.hscvlab.org.au")   # where app.hcsvlab.org.au is the server you are using
    
### Client

To get the API version:

    client$get_api_version()
    
Example Response:

    "V2.3"


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
    
    
To get an item list by ID (this returns an ItemList object):

    client$get_item_list(id)
    
where uri is the ID of the item list e.g. "1". This returns the same as above.


To get a specific item by URI:

    client$get_item(uri)

where uri is the full URI of the item e.g. "http://app.hcsvlab.org.au/catalog/hcsvlab:100"

Example Response:

    ID: 
    [1] "mitcheldelbridge:S1219s1"
    URI: 
    [1] "http://localhost:3000/catalog/hcsvlab:5438"


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
    

To download a set of items:

    client$download_items(items, destination, name)
    
where items is a list of item URIS (e.g. client$search_metadata("some query")$items), destination is the target directory and name is the name of the resulting zip file. Returns the filename of the resulting file

Example Response:

    [1] "/Users/hcsvlab/cooee.zip"
    
    
To create an item list:

    client$create_item_list(items, name)
    
where items is a list of item URIS (e.g. client$search_metadata("some query")$items) and name is the name of the item list to be created

Example Response:

    $success
    [1] "31 items added to new item list TEST"
    


    
    

