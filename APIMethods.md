### API Methods
### Introduction

The HCS vLab API provides access to resources from the virtual lab platform in a machine readable format (JSON).    At present the API supports retrieving item lists, items, documents and collection level metadata; it does not support creating item lists.  The assumption at the moment is that a user will create an item list via the Web Application and then can write code to access the item list and individual items for processing.  

### Using the API

All requests to the API need to be authenticated via an API key which can be downloaded from the Web Application (under the user menu at the top right).  The API key is specific to a user and should not generally be included in code that is published.  (We are developing a standard configuration file location that can be used to store this).  
The API Key needs to be specified as X-API-KEY in the request header.
The API uses the same URLs as the main web application in most cases but is able to return data in JSON format when an appropriate Accept header is sent with the request.  If the Accept header is set to 'application/json' or is left blank, then the response will be sent back as JSON.   Alternatively, a '.json' extension can be added to the end of the URL to force JSON format.    A web browser will generally have a very broad Accept header that will include text/html and so will get an HTML response.

Eg.
Where <host> is the vlab host, eg http://ic2-hcsvlab-staging1-vm.intersect.org.au/

  $ curl -H "X-API-KEY: WUqhKgM25PJuzivjdvGt" <host>/item_lists.json

  $ curl -H "X-API-KEY: WUqhKgM25PJuzivjdvGt" -H "Accept: application/json" <host>item_lists
