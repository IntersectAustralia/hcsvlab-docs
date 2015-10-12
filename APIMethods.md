### API Methods
### Introduction

The Alveo API provides access to resources from the virtual lab platform in a machine readable format (JSON).    At present the API supports retrieving item lists, items, documents and collection level metadata; it does not support creating item lists.  The assumption at the moment is that a user will create an item list via the Web Application and then can write code to access the item list and individual items for processing.  

### Using the API

All requests to the API need to be authenticated via an API key which can be downloaded from the Web Application (under the user menu at the top right).  The API key is specific to a user and should not generally be included in code that is published.  (We are developing a standard configuration file location that can be used to store this).  
The API Key needs to be specified as X-API-KEY in the request header.
The API uses the same URLs as the main web application in most cases but is able to return data in JSON format when an appropriate Accept header is sent with the request.  If the Accept header is set to 'application/json' or is left blank, then the response will be sent back as JSON.   Alternatively, a '.json' extension can be added to the end of the URL to force JSON format.    A web browser will generally have a very broad Accept header that will include text/html and so will get an HTML response.

**Examples**

Command line:

	Where <host> is the vlab host, e.g. https://app.alveo.edu.au/
	$ curl -H "X-API-KEY: WUqhKgM25PJuzivjdvGt" <host>/item_lists.json
	$ curl -H "X-API-KEY: WUqhKgM25PJuzivjdvGt" -H "Accept: application/json" <host>item_lists

Python
```python
    import pycurl
    import cStringIO
    import json

    api_key = <YOUR_X_API_KEY>
    url = <YOUR_URL> # e.g.: https://app.alveo.edu.au/item_lists/1

    curl = pycurl.Curl()
    buf = cStringIO.StringIO()
    curl.setopt(curl.URL, url)
    curl.setopt(pycurl.HTTPHEADER, ['X-API-KEY: ' + api_key, 'Accept: application/json'])
    curl.setopt(curl.WRITEFUNCTION, buf.write)
    curl.perform()
    response = buf.getvalue()
    status = curl.getinfo(pycurl.HTTP_CODE)	
    response = json.loads(string)
```

Ruby
```ruby
    require 'net/http'
    require 'json'

    api_key = <YOUR_X_API_KEY>
    url = <YOUR_URL> # e.g.: https://app.alveo.edu.au/item_lists/1

    sampleUrl = URI(url)
    req = Net::HTTP::Get.new(sampleUrl)
    req['X-API-KEY'] = <YOUR_X_API_KEY>
    req['Accept'] = "application/json"

    response = Net::HTTP.start(sampleUrl.hostname, sampleUrl.port) {|http|
      http.request(req)
    }
    
    jsonResponse = JSON.parse(res.body)
```

### HTTP Response Codes

<table><tbody>
<tr>
<th> Scenario </th>
<th> HTTP Response Code </th>
</tr>
<tr>
<td> User is unathenticated </td>
<td> 401 </td>
</tr>
<tr>
<td> User is unauthorised </td>
<td> 403 </td>
</tr>
<tr>
<td> User is authorised and the requested data is returned </td>
<td> 200 </td>
</tr>
<tr>
<td> User is authorised but the requested data is not found </td>
<td> 404 </td>
</tr>
<tr>
<td> Request uses bad syntax/missing required parameters </td>
<td> 400 </td>
</tr>
</tbody></table>

<h3> API Description </h3>

<table></tbody>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>
<tr> 
<td> Get API version </td>
<td> /version </td>
<td> GET </td>
<td> API version </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<CODE>
	{"API version":"V2.0"} 
</CODE>
</td>
</tr>
<tr> 
<td> Get item lists </td>
<td> /item_lists </td>
<td> GET </td>
<td> Item Lists of the user (name, url, number of items) </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4> 
	<pre>
	<CODE>
	[
		{
			"name":"Jared's Item List 1",
			"item_list_url":"https://app.alveo.edu.au/item_lists/5",
			"num_items":1
		},
		{
			"name":"Jared's Item List 2",
			"item_list_url":"https://app.alveo.edu.au/item_lists/15",
			"num_items":3
		},
		{
			"name":"Jared's Item List 3",
			"item_list_url":"https://app.alveo.edu.au/item_lists/16",
			"num_items":3
		}
	] 
	</CODE>
	</pre>
</td>
</tr>
<tr> 
<td> Get Items from Item List </td>
<td> /item_lists/{id} </td>
<td> GET </td>
<td> Specified Item List of the user (name, list of items) </td>
<td>
Item list can be retrieved as JSON, ZIP or WARC format. The JSON format will only return the URL for the items in the Item list as shown below. The ZIP and WARC file will create a package containing the documents and metadata of the items in the Item list.
<br>The Zip file will respect the BagIt structure.
<br>Requests example:
<br>	curl -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/&lt;format&gt;" &lt;server&gt;/item_lists/{id}
<br>format= json or zip or warc
</td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
	<CODE>
	{
		"name":"Jared's Item List 2",
		"num_items":3,
		"items":[
			"https://app.alveo.edu.au/catalog/cooee/2-037",
			"https://app.alveo.edu.au/catalog/cooee/2-038",
			"https://app.alveo.edu.au/catalog/cooee/2-040"
]

		]
	} 
	</CODE>
</pre>
</td>
</tr>
<tr> 
<td> Delete Item List </td>
<td> /item_lists/{id} </td>
<td> DELETE </td>
<td> Specified Item List of the user (name, list of items) </td>
<td>
Item list can be deleted via HTTP DELETE mehod with the same endpoint as get items from item list
<br>Requests example:
<br>	curl -X DELETE -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/json" &lt;server&gt;/item_lists/{id}
</td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
	<CODE>
	{
		"success":"item list test deleted successfully"
	} 
	</CODE>
</pre>
</td>
</tr>
<tr> 
<td> Share Item List </td>
<td> /item_lists/{id}/share </td>
<td> POST </td>
<td> Specified Item List of the user (name, list of items) </td>
<td>
Item list can be shareed via HTTP POST mehod
<br>Requests example:
<br>	curl -X POST -d "" -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/json" &lt;server&gt;/item_lists/{id}/share
</td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
	<CODE>
	{
		"success":"Item list test is shared. Any user in the application will be able to see it."
	} 
	</CODE>
</pre>
</td>
</tr>
<tr> 
<td> Unshare Item List </td>
<td> /item_lists/{id}/unshare </td>
<td> POST </td>
<td> Specified Item List of the user (name, list of items) </td>
<td>
Item list can be unshareed via HTTP POST mehod
<br>Requests example:
<br>	curl -X POST -d "" -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/json" &lt;server&gt;/item_lists/{id}/unshare
</td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
	<CODE>
	{
		"success":"Item list test is not being shared anymore."
	} 
	</CODE>
</pre>
</td>
</tr>
<tr> 
<td> Clear Item List </td>
<td> /item_lists/{id}/clear </td>
<td> POST </td>
<td> Specified Item List of the user (name, list of items) </td>
<td>
Item list can be unshareed via HTTP POST mehod
<br>Requests example:
<br>	curl -X POST -d "" -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/json" &lt;server&gt;/item_lists/{id}/clear
</td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
	<CODE>
	{
		"success":"2 cleared from item list test"
	} 
	</CODE>
</pre>
</td>
</tr>
<tr> 
<td> Get item metadata </td>
<td> /catalog/{collection_name}/{item_id} </td>
<td> GET </td>
<td> Item Metadata, primary text URL & document URLs </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
	<CODE>
	{
		"@context": "https://app.alveo.edu.au/schema/json-ld",
		"alveo:catalog_url": "https://app.alveo.edu.au/catalog/ace/A08g",
		"alveo:metadata": {
			"dc:title": "Boy Drowns Trying to Save Pet Dog",
			"dc:created": "8 September 1986",
			"dc:identifier": "A08g",
			"dc:isPartOf": "ace",
			"ausnc:itemwordcount": "110",
			"ausnc:mode": "written",
			"ausnc:speech_style": "unspecified",
			"ausnc:interactivity": "unspecified",
			"ausnc:communication_context": "unspecified",
			"olac:discourse_type": "unspecified",
			"olac:language": "eng",
			"ace:genre": "Press Reportage",
			"ausnc:audience": "mass_market",
			"ausnc:communication_setting": "popular",
			"ausnc:communication_medium": "unspecified",
			"ausnc:publication_status": "published",
			"ausnc:source": "The Sun 8 September 1986 (2001 words)",
			"ausnc:written_mode": "print",
			"dc:publisher": "Publisher - John Fairfax and Sons Ltd",
			"ausnc:document": "A08g#Text, A08g#Original, A08g#Raw",
			"dc:type": "Text, Original, Raw",
			"dc:extent": "631, 579959, 739",
			"alveo:date_group": "1980 - 1989",
			"alveo:display_document": "A08g#Text",
			"alveo:indexable_document": "A08g#Text",
			"alveo:full_text": "BOY DROWNS TRYING TO SAVE PET DOG A BOY drowned...",
			"alveo:handle": "ace:A08g",
			"alveo:sparqlEndpoint": "https://app.alveo.edu.au/sparql/ace"
		},
		"alveo:primary_text_url": "https://app.alveo.edu.au/catalog/ace/A08g/primary_text.json",
		"alveo:annotations_url": "https://app.alveo.edu.au/catalog/ace/A08g/annotations.json",
		"alveo:documents": [{
			"alveo:url": "https://app.alveo.edu.au/catalog/ace/A08g/document/A08g-plain.txt",
			"dc:type": "Text",
			"alveo:size": "631 B",
			"rdf:type": "http://xmlns.com/foaf/0.1/Document",
			"dc:identifier": "A08g-plain.txt",
			"dc:title": "A08g#Text",
			"dc:extent": "631",
			"dc:type": "Text"
		}, {
			"alveo:url": "https://app.alveo.edu.au/catalog/ace/A08g/document/ace_a.txt",
			"dc:type": "Original",
			"alveo:size": null,
			"rdf:type": "http://xmlns.com/foaf/0.1/Document",
			"dc:identifier": "ace_a.txt",
			"dc:title": "A08g#Original",
			"dc:extent": "579959",
			"dc:type": "Original"
		}, {
			"alveo:url": "https://app.alveo.edu.au/catalog/ace/A08g/document/A08g-raw.txt",
			"dc:type": "Raw",
			"alveo:size": "739 B",
			"rdf:type": "http://xmlns.com/foaf/0.1/Document",
			"dc:identifier": "A08g-raw.txt",
			"dc:title": "A08g#Raw",
			"dc:extent": "739",
			"dc:type": "Raw"
		}]
	}
	</CODE>
</pre>
</td>
</tr>
<tr> 
<td> Get primary text </td>
<td> /catalog/{collection_name}/{item_id}/primary_text </td>
<td> GET </td>
<td> Content of item's primary_text </td>
<td> This starts a download using the primary_text content with the filename as the label attached during ingest. </td>
</tr>
<tr> 
<td> Get document </td>
<td> /catalog/{collection_name}/{item_id}/document/{filename}
<br>Example:
<br>/catalog/ace/A08g/document/A08g-raw.txt </td>
<td> GET </td>
<td> Specified document that belongs to the Item </td>
<td> This sends the file as listed in the SOURCE metadata. 
<br>Note: The filename is being passed as a url parameter, which allows us to have the unified devise error handling that comes with a .json response </td>
</tr>
<tr> 
<td> Get annotation context </td>
<td> /schema/json-ld </td>
<td> GET </td>
<td> Context for the annotations </td>
<td> Static conext linked from the annotations API call </td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
	<CODE>
	{
		"@context": {
			"@base":"http://purl.org/dada/schema/0.2/",
			"annotations":{
				"@id":"http://purl.org/dada/schema/0.2/annotations",
				"@container":"@list"
			},
			"commonProperties": {"@id":"http://purl.org/dada/schema/0.2/commonProperties"},
			"type":{"@id":"http://purl.org/dada/schema/0.2/type"},
			"start":{"@id":"http://purl.org/dada/schema/0.2/start"},
			"end":{"@id":"http://purl.org/dada/schema/0.2/end"},
			"label":{"@id":"http://purl.org/dada/schema/0.2/label"},
			"annotates":{"@id":"http://purl.org/dada/schema/0.2/annotates"}
		}
	}
	</CODE>
</pre>
</td>
</tr>
<tr> 
<td> Get annotations </td>
<td>/catalog/{collection_name}/{item_id}/annotations
<br>/catalog/{collection_name}/{item_id}/annotations.json?type=maus:phonetic&label=s</td>
<td> GET </td>
<td> Annotations from this item's annotation file </td>
<td> Queries the item's annotation file for all annotations (label, start/end times). Type and label can be passed in to narrow the query </td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
	<CODE>
	{
		@context: "https://app.alveo.edu.au/schema/json-ld",
		commonProperties: {
			alveo: annotates: "https://app.alveo.edu.au/catalog/mitcheldelbridge/S1224s1/document/S1224s1.wav"
		},
		alveo: annotations: [{@
			id: "http://ns.ausnc.org.au/corpora/mitcheldelbridge/annotation/302021",
			label: "s",
			type: "http://ns.ausnc.org.au/schemas/annotation/maus/phonetic",
			@type: "dada:SecondAnnotation",
			end: "2.21",
			start: "2.18"
		}, {@
			id: "http://ns.ausnc.org.au/corpora/mitcheldelbridge/annotation/302024",
			label: "s",
			type: "http://ns.ausnc.org.au/schemas/annotation/maus/phonetic",
			@type: "dada:SecondAnnotation",
			end: "3.31",
			start: "3.28"
		}]
	}
	</CODE>
</pre>
</td>
</tr>
<tr> 
<td> Get annotation types </td>
<td> /catalog/{collection_name}/{item_id}/annotations/types
<td> GET </td>
<td> Annotation types from this item's annotations </td>
<td> Queries the item's annotations to return all the different annotation types for the item </td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
{
  item_url: "https://app.alveo.edu.au/catalog/cooee:4-340",
  annotation_types: [
    "http://ns.ausnc.org.au/schemas/annotation/cooee/ellipsis",
    "http://ns.ausnc.org.au/schemas/annotation/cooee/pageno"
  ]
}
</pre>
</td>
</tr>
<tr>
<td> Get items' documents and metadata </td>
<td> /catalog/download_items </td>
<td> POST </td>
<td> Download a file in the specified format containing the item's documents and metadata </td>
<td> This is a post request that requires a JSON set of items sent with it. Hence, cannot be replicated through a browser but through curl this can be done with something akin to:
<br>curl -H "X-API-KEY: <key>" -H "Accept: application/<format>" -H "Content-Type: application/json" -X POST -d '{"items":["<host>/catalog/cooee/2-015","<host>/catalog/cooee/2-021"]}' <host>/catalog/download_items
<br>Allowed formats = zip, warc.
<br>If format is not specified, zip will be set by default
<br>You can also specify the file format using as follow:
<br>curl -H "X-API-KEY: <key>" -H "Content-Type: application/json" -X POST -d '{"items":["<host>/catalog/cooee/2-015","<host>/catalog/cooee/2-021"]}' <host>/catalog/download_items.<format>
<br>curl -H "X-API-KEY: <key>" -H "Content-Type: application/json" -X POST -d '{"items":["<host>/catalog/cooee/2-015","<host>/catalog/cooee/2-021"]}' <host>/catalog/download_items?format=<format>
</td>
</tr>
<tr>
<td> Example Input </td>
<td colspan=4>
<pre>
	<CODE>
	{
		"items":[
			"https://app.alveo.edu.au/catalog/cooee/2-036",
			"https://app.alveo.edu.au/catalog/cooee/2-037",
			"https://app.alveo.edu.au/catalog/cooee/2-038",
			"https://app.alveo.edu.au/catalog/cooee/2-040"
		]
	} 
	</CODE>
</pre>
</td>
</tr>
<tr> 
<td> Get collections list</td>
<td> /catalog </td>
<td> GET </td>
<td> Number of collections, and collections list </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
	<CODE>
	{
		"num_collections":6,
		"collections":[
		        "https://app.alveo.edu.au/catalog/ace",
		        "https://app.alveo.edu.au/catalog/austlit",
		        "https://app.alveo.edu.au/catalog/cooee",
		        "https://app.alveo.edu.au/catalog/gcsause",
		        "https://app.alveo.edu.au/catalog/ice",
		        "https://app.alveo.edu.au/catalog/monash"
		]
	}
	</CODE>
</pre>
</td>
</tr>
<tr> 
<td> Get collection </td>
<td> /catalog/{collection_name} </td>
<td> GET </td>
<td> Collection url, name and metadata </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
	<CODE>
	{
		"collection_url":"https://app.alveo.edu.au/catalog/monash",
		"collection_name":"monash",
		"metadata": {
 			"RDF_type":"Collection",
 			"Is Located At":"Monash University, Victoria, Australia",
 			"Custodian":"simon Musgrave",
 			"Rights":"All rights reserved to Monash UNiversity",
 			"Subject":"2004 - Linguistics",
 			"Title":"Monash Corpus of Spoken English"
			... (other fields)
		}
	}
	</CODE>
</pre>
</td>
</tr>
<tr> 
<td> Search Metadata </td>
<td> 
	/catalog/search?metadata=&lt;query&gt;
	<br>Example:<br>
	/catalog/search?metadata=collection_name:cooee 
</td>
<td> GET </td>
<td> List of search results </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
	<CODE>
	{
		"num_results":4,
		"items":[
			"https://app.alveo.edu.au/catalog/cooee/2-036",
			"https://app.alveo.edu.au/catalog/cooee/2-037",
			"https://app.alveo.edu.au/catalog/cooee/2-038",
			"https://app.alveo.edu.au/catalog/cooee/2-040"
		]
	}
	</CODE>
</pre>
</td>
</tr>
<tr> 
<td> Add to item list </td>
<td> /item_lists
<br>/item_lists?name=ItemListA </td>
<td> POST </td>
<td> Result of item list operation (error/success) </td>
<td> This is a post request that requires a JSON set of items sent with it. Hence, cannot be replicated through a browser but through curl this can be done with something akin to:
<br>curl -H "X-API-KEY: <key>" -H "Content-Type: application/json" -H "Accept: application/json" -X POST -d '{"num_results":2,"items":["<host>/catalog/cooee/2-015","<host>/catalog/cooee/2-021"]}' <host>/item_lists?name=A
<br>Note: the name doesn't have to be sent in as a parameter attached to the URL, it can also be sent in as part of the JSON
</td>
</tr>
<tr>
<td> Example Input </td>
<td colspan=4>
<pre>
	<CODE>
	{
		"name":"New Item List",
		"num_results":4,
		"items":[
			"https://app.alveo.edu.au/catalog/cooee/2-036",
			"https://app.alveo.edu.au/catalog/cooee/2-037",
			"https://app.alveo.edu.au/catalog/cooee/2-038",
			"https://app.alveo.edu.au/catalog/cooee/2-040"
]

		]
	}
	</CODE>
	<br>OR if name is specified as url parameter as shown in the notes above
	<br>
	<CODE>
	{
		"items":[
			"https://app.alveo.edu.au/catalog/cooee/2-036",
			"https://app.alveo.edu.au/catalog/cooee/2-037",
			"https://app.alveo.edu.au/catalog/cooee/2-038",
			"https://app.alveo.edu.au/catalog/cooee/2-040"
]

		]
	} 
	</CODE>
</pre>
</td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
	<CODE>
	Success:
		{"success":"2 items added to existing item list A"}
	Failure:
		{"error":"name parameter not found"}
		or
		{"error":"items parameter not found"}
		or
		{"error":"items parameter not an array"}
		or
		{"error":"invalid-json"} 
	</CODE>
</pre>
</td>
</tr>
<tr>
<td> Rename Item List </td>
<td> /item_lists/{item_list_id} </td>
<td> PUT </td>
<td> Item list if successful/error message if failure </td>
<td> This is a put request that takes the new name of the item list. Example curl command:
<br>curl -H "X-API-KEY:&lt;api_key&gt;" -H "Accept: application/json" -X PUT -d '{"name":"&lt;new name&gt;"}' &lt;host&gt;/item_lists/:id
</td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
	<CODE>
	Success:
	{
		"name":"Jared's Item List 2",
		"num_items":3,
		"items":[
			"https://app.alveo.edu.au/catalog/cooee/2-037",
			"https://app.alveo.edu.au/catalog/cooee/2-038",
			"https://app.alveo.edu.au/catalog/cooee/2-040"
]

		]
	} 
	Failure:
		{"error":"name can't be blank"}
		or
		{"error":"name too long"}
		or
		{"error":"couldn't rename item list"}
	</CODE>
</pre>
</td>
</tr>
<tr> 
	<td> Upload Annotation </td>
	<td> /catalog/{collection_name}/{item_id}/annotations</td>
	<td> POST </td>
	<td> Result of item list operation (error/success) </td>
	<td> This is a post request that requires a file sent with it. Hence, cannot be replicated through a browser but through curl this can be done with something akin to:
<br>curl -H "X-API-KEY:&lt;api_key&gt;" -H "Accept: application/json" -F file=@&lt;path_to_file&gt; &lt;host&gt;/catalog/:id/annotations
	</td>
</tr>
<tr>
	<td> Json-Ld Input File format </td>
	<td colspan=4>
	<pre>
	<CODE>
	{
		  "@context": {
			"@base": "http://purl.org/dada/schema/0.2/",
			"<prefix>": {
		    		"@id": "<URI>"
			},
			"<prefix>": {
		    		"@id": "<URI>"
			},
			.
			.
			.
			"<prefix>": {
		    		"@id": "<URI>"
			},
		  },
		  "@graph":[
		    {
		        "@type": "http://purl.org/dada/schema/0.2#UTF8Region",
		        "<property_name>": "value",
		        "<property_name>": "value",
		        .
		        .
		        .
		        "<property_name>": "value",
		    },
		    {
		        "@type": "http://purl.org/dada/schema/0.2#UTF8Region",
		        "<property_name>": "value",
		        "<property_name>": "value",
		        .
		        .
		        .
		        "<property_name>": "value",
		    },
		    .
		    .
		    .
		    {
		        "@type": "http://purl.org/dada/schema/0.2#UTF8Region",
		        "<property_name>": "value",
		        "<property_name>": "value",
		        .
		        .
		        .
		        "<property_name>": "value",
		    }
		  ]
	}
	</CODE>
	</pre>
	</td>
</tr>
<tr>
	<td> Example Response </td>
	<td colspan=4>	
	<pre>
		<CODE>
		Success:
			{"success":"file &lt;filename&gt; uploaded successfully"}
		Failure:
			{"error":"No Item with id '&lt;item_id&gt;' exists."}
			or
			{"error":"Uploaded file is not present or empty."}
			or
			{"error":"File already uploaded."}
			or
			{"error":"Error uploading file &lt;filename&gt;."}
		</CODE>
	</pre>
	</td>
</tr>


<tr> 
	<td> Query metadata and annotations usign sparql</td>
	<td> /sparql/&lt;collection-name&gt;?query=&lt;sparql-query&gt;</td>
	<td> GET </td>
	<td> Json formatted query result </td>
	<td> 
		curl -g -H "X-API-KEY: &lt;API_KEY&gt;" -H "Accept: application/json" "&lt;host&gt;/sparql/&lt;collection-name&gt;?query=&lt;sparql-query&gt;"
		<br>
		Example:<br>
		curl -g -H "X-API-KEY: &lt;API_KEY&gt;" -H "Accept: application/json" "&lt;host&gt;/sparql/cooee?query=select * where {?s &lt;http://purl.org/dc/terms/isPartOf&gt; ?o}"
	</td>
</tr>
<tr>
	<td> Example Response </td>
	<td colspan=4>	
	<pre>
		<CODE>
			{
			  "head" : {
			    "vars" : [ "s", "o" ]
			  },
			  "results" : {
			    "bindings" : [ {
			      "s" : {
			        "type" : "uri",
			        "value" : "http://ns.ausnc.org.au/corpora/cooee/items/1-001"
			      },
			      "o" : {
			        "type" : "uri",
			        "value" : "http://ns.ausnc.org.au/corpora/cooee"
			      }
			    }, {
			      "s" : {
			        "type" : "uri",
			        "value" : "http://ns.ausnc.org.au/corpora/cooee/items/1-002"
			      },
			      "o" : {
			        "type" : "uri",
			        "value" : "http://ns.ausnc.org.au/corpora/cooee"
			      }
			    }, {
			      "s" : {
			        "type" : "uri",
			        "value" : "http://ns.ausnc.org.au/corpora/cooee/items/1-003"
			      },
			      "o" : {
			        "type" : "uri",
			        "value" : "http://ns.ausnc.org.au/corpora/cooee"
			      }
			    }]
			  }
			}
    </CODE>
	</pre>
	</td>
</tr>

<tr>
<td>Create a collection</td>
<td>/catalog</td>
<td>POST</td>
<td>Result of operation (error/success)</td>
<td>
<ol>
<li>Users are only authorised to create a collection if they have the role of 'admin' or 'data owner'.</li>
<li>When creating a collection, the "@id" which specifies the URI of the collection needs to be supplied. Unlike other API calls this URI will <em>NOT</em> be automatically converted into the corresponding Alveo catalog URL.</li>
<li>The collection name can either be supplied as a JSON parameter or as a URL paramaeter. See the example input for examples of this.</li>
<li>
This is a POST request that requires a JSON-LD set of collection metadata to be sent with it.
Hence, cannot be replicated through a browser but through curl this can be done with something akin to the following:
<ul>
<li>When creating a collection with the collection name supplied as a URL parameter:
<br><code>curl -X POST -H "X-API-KEY: &ltkey&gt" -H "Content-Type: application/json" -H "Accept: application/json" -d '{ "collection_metadata": &ltcollection_metadata&gt}' &ltserver&gt/catalog?name=&ltcollection_name&gt</code>
</li>
<li>When creating a collection with the collection name supplied as a JSON parameter:
<br><code>curl -X POST -H "X-API-KEY: &ltkey&gt" -H "Content-Type: application/json" -H "Accept: application/json" -d '{ "collection_metadata": &ltcollection_metadata&gt, "name":"&ltcollection_name&gt"}' &ltserver&gt/catalog</code>
</li>
</ul>
</li>
</ol>
</td>
</tr>
<td>Example Input</td>
<td colspan=4> 
The following is an example of expected input for &ltcollection_metadata&gt:
<ul>
<li><code>{ "@context": { "dc": "http://purl.org/dc/terms/", "dcterms": "http://purl.org/dc/terms/", "dcmitype": "http://purl.org/dc/dcmitype/" }, "@id": "http://colllection.name.uri", "@type": "dcmitype:Collection", "dc:creator": "Data Owner", "dc:rights": "All rights reserved to Data Owner", "dc:subject": "English Language", "dc:title": "Test" }</code></li>
</ul>
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<br>Success:
<br>{"success":"New collection '&ltcollection_name&gt' (&ltcollection_URI&gt) created"}
<br>Failure:
<br>{"error":"Permission Denied: Your role within the system does not have sufficient privileges to be able to create a collection. Please contact an Alveo administrator."}
<br>or
<br>{"error":"name parameter not found"}
<br>or
<br>{"error":"metatdata parameter not found"}
<br>or
<br>{"error":"name and metadata parameters not found"}
<br>or
<br>{"error":"invalid-json"}
<br>or
<br>{"error":"JSON-LD formatted metadata must be sent to the add collection api call as a POST request"}
<br>or
<br>{"error":"Collection '&lcollection_namet&gt' (&ltcollection_URI&gt) already exists in the system - skipping"}
</td>
</tr>

<tr>
<td>Edit a Collection</td>
<td>/catalog/{collection_name}</td>
<td>PUT</td>
<td>Result of operation (error/success)</td>
<td>Notes:
<ol>
<li>Users are only authorised to edit a collection which they own.</li>
<li>It is not neccessary to include the collection URI within the JSON-LD metadata <code>("@id":"&ltcollection_uri&gt")</code> as this will be automatically generated.</li>
<li>The "replace" JSON parameter indicates whether or not the provided metadata should replace the existing collection metadata. 
<ul>
<li>If replace is true then the entire existing collection metadata (including the context) will be replaced with the provided metadata.</li>
<li>If replace is false or unsupplied then the collection metadata will be updated with the provided metadata.</li>
</ul>
</li>
<li>This is a PUT request. Hence, cannot be replicated through a browser but through curl this can be done with something akin to the following.
<ul>
<li><code>curl -H "X-API-KEY: &ltkey&gt"  -H "Content-Type: application/json" -H "Accept: application/json" -X PUT -d '{ "replace": &lttrue/false&gt, "collection_metadata": &ltcollection_metadata&gt}' &ltserver&gt/catalog/&ltcollection_id&gt</code></li>
</ul>
</li>
</ol>
</td>
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
<ul>
<li>When updating collection metadata without specifying the "replace" parameter: <br><code>{ "collection_metadata": {"http://purl.org/dc/elements/1.1/title": "An updated test collection"}}</code></li>
<li>When updating collection metadata by setting the "replace" parameter to false: <br><code>{ "replace": false, "collection_metadata": {"http://purl.org/dc/elements/1.1/title": "An updated test collection"}}</code></li>
<li>When overwriting collection metadata by setting the "replace" parameter to true: <br><code>{ "replace": true, "collection_metadata": {"@context": {"dc": "http://purl.org/dc/elements/1.1/", "dcmitype": "http://purl.org/dc/dcmitype/"}, "@type": "dcmitype:Collection", "dc:title": "An updated test collection", "dc:subject": "English Language", "dc:creator": "Test User"}}</code></li>
</ul>
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<br>Success:
<br>{"success":"Updated collection &ltcollection_name&gt"}
<br>Failure:
<br>{"error":"Requested collection not found"}
<br>or
<br>{"error":"User is unauthorised"}
<br>or
<br>{"error":"Invalid metadata"}
</td>
</tr>

<tr>
<td>Add items with documents to an owned collection</td>
<td>/catalog/{collection_id}</td>
<td>POST</td>
<td>Result of operation (error/success) and a list of item ids for each item added</td>
<td>Notes:
<ol>
<li>Users are only authorised to add items to a collection which they own.</li>
<li>The expected format for adding a document is to have the metadata for that document nested within the item level metadata. See the example input for examples of this.</li>
<li>When adding items or documents, the "@id" that specifies the URI of the item/document needs to be supplied. This  will then be automatically converted into Alveo catalog URLs. For example:
<br>When adding an item to a test collection if the user specifies <code>"@id":"item1"</code> then the system will convert this into <code>"@id":"http://app.avleo.edu.au/catalog/test/item1"</code>. 
<br>Similarly a document with <code>"@id":"document1"</code> will be converted to <code>"@id":"http://app.alveo.edu.au/catalog/test/item1/document/document1"</code>.
</li>
<li>The document metadata term <code>"dcterms:source":{"@id":"&ltfile_or_http_uri&gt"}</code> does not need to be provided for any documents whose contents are embedded in the JSON item metadata or are uploaded as part of the HTTP request. This is since the server will assign a location for uploaded documents and modify the aforementioned metadata document source term appropriately. However it is essential to include the document source metadata term for any documents referenced with "file://" or "http://".</li>
<li>For document files uploaded as part of the HTTP request or whose content is embedded in the JSON item metadata, for that document content or file to be associated with the metadata of a particular document the value given to the <code>"dcterms:identifier"</code> within the document metadata needs to match the name of the uploaded file or identifier of the embedded document content. For example: <br>a document with metadata containing <code>"dcterms:identifier":"sample.txt"</code> will be associated with the uploaded file "sample.txt".</li>
<li>The <code>"dcterms:isPartOf":{"@id":"corpus:&ltcollection_name&gt"}</code> should not be supplied when adding an item to a collection. This metadata will be automatically generated and will correspond with the collection that the item is being added to. If this metadata is supplied it will be overwritten when the system generates it for the corresponding corpus.</li>
<li>If an item being added contains invalid metadata that item will not be added to the collection but all other valid items shall  continue to be added to the collection. The API response will contain a list of ids of each item that was successfully added to the collection.</li>
<li>
This is a POST request that requires a JSON-LD set of collection metadata to be sent with it.
Hence, cannot be replicated through a browser but through curl this can be done with something akin to the following:
<ul>
<li>If adding an item with document(s) that are referenced (with "file://" or "http://"):
<br><code>curl -X POST -H "X-API-KEY: &ltkey&gt" -H "Accept: application/json" -H "Content-Type: application/json"  -d '{"items":[{"metadata":{&ltitem_metadata&gt}'}]}' &ltserver&gt/catalog/&ltcollection_id&gt</code>
</li>
<li>If adding an item with document(s) whose contents are embedded in the JSON item metadata:
<br><code>curl -X POST -H "X-API-KEY: &ltkey&gt" -H "Accept: application/json" -H "Content-Type: application/json"  -d '{"items":[{"documents":[{&ltdocument_metadata&gt}], "metadata":{&ltitem_metadata&gt}'}]}' &ltserver&gt/catalog/&ltcollection_id&gt</code>
</li>
<li>If adding an item with a single document uploaded as part of the HTTP request:
<br><code>curl -X POST -H "X-API-KEY: &ltkey&gt" -H "Accept: application/json" -F file=@"&ltfile_location&gt" -F items='[{&ltitem_metadata&gt}]' &ltserver&gt/catalog/&ltcollection_id&gt</code>
</li>
<li>If adding an item with multiple documents uploaded as part of the HTTP request:
<br><code>curl -X POST -H "X-API-KEY: &ltkey&gt" -H "Accept: application/json" -F file[]=@"&ltfile_1_location&gt" -F file[]=@"&ltfile_2_location&gt" -F items='[{&ltitem_metadata&gt}]' &ltserver&gt/catalog/&ltcollection_id&gt</code>
</li>
</ul>
</li>
</ol>
</td>
</tr>
<td>Example Input</td>
<td colspan=4> 
<ul>
<li>If adding an item with document(s) that are referenced (with "file://" or "http://"):
<br><code>-d '{ "items": [ { "metadata": { "@context": { "ausnc": "http://ns.ausnc.org.au/schemas/ausnc_md_model/", "corpus": "http://ns.ausnc.org.au/corpora/", "dc": "http://purl.org/dc/terms/", "dcterms": "http://purl.org/dc/terms/", "foaf": "http://xmlns.com/foaf/0.1/", "hcsvlab": "http://hcsvlab.org/vocabulary/" }, "@graph": [ { "@id": "item1", "@type": "ausnc:AusNCObject", "ausnc:document": [ { "@id": "document1", "@type": "foaf:Document", "dcterms:extent": 1234, "dcterms:identifier": "document1.txt", "dcterms:source": { "@id": "file:///data/test_collections/ausnc/test/document2.txt" }, "dcterms:title": "document1#Text", "dcterms:type": "Text" } ], "dcterms:identifier": "item1", "hcsvlab:indexable_document": { "@id": "document1.txt" }, "hcsvlab:display_document": { "@id": "document1.txt" } } ] } } ] }'</code>
</li>
<li>If adding an item with document(s) whose contents are embedded in the JSON item metadata:
<br><code>-d '{"items": [ { "documents": [ { "identifier": "document1.txt", "content": "This document had its content provided as part of the JSON request." } ], "metadata": { "@context": { "ausnc": "http://ns.ausnc.org.au/schemas/ausnc_md_model/", "corpus": "http://ns.ausnc.org.au/corpora/", "dc": "http://purl.org/dc/terms/", "dcterms": "http://purl.org/dc/terms/", "foaf": "http://xmlns.com/foaf/0.1/", "hcsvlab": "http://hcsvlab.org/vocabulary/" }, "@graph": [ { "@id": "item1", "@type": "ausnc:AusNCObject", "ausnc:document": [ { "@id": "document1.txt", "@type": "foaf:Document", "dcterms:extent": 72636, "dcterms:identifier": "document1.txt", "dcterms:title": "document1#Text", "dcterms:type": "Text" } ], "dcterms:identifier": "item1", "hcsvlab:display_document": { "@id": "document1.txt" }, "hcsvlab:indexable_document": { "@id": "document1.txt" } } ] } } ] }'
</code>
</li>
<li>If adding an item with a single document uploaded as part of the HTTP request:
<br><code>-F file=@"1-001-plain.txt" -F items='[ { "metadata": { "@context": { "ausnc": "http://ns.ausnc.org.au/schemas/ausnc_md_model/", "corpus": "http://ns.ausnc.org.au/corpora/", "dc": "http://purl.org/dc/terms/", "dcterms": "http://purl.org/dc/terms/", "foaf": "http://xmlns.com/foaf/0.1/", "hcsvlab": "http://hcsvlab.org/vocabulary/" }, "@graph": [ { "@id": "item1", "@type": "ausnc:AusNCObject", "ausnc:document": [ { "@id": "1-001-plain.txt", "@type": "foaf:Document", "dcterms:extent": 72636, "dcterms:identifier": "1-001-plain.txt", "dcterms:title": "document1#Text", "dcterms:type": "Text" } ], "dcterms:identifier": "item1", "hcsvlab:display_document": { "@id": "1-001-plain.txt" }, "hcsvlab:indexable_document": { "@id": "1-001-plain.txt" } } ] } } ]'</code>
</li>
</ul>
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<br>Success:
<br>{"success":["&ltitem_1_id&gt", "&ltitem_2_id&gt"]}
<br>or
<br>{"success":["&ltitem_id&gt"], "failures":["Unknown item contains invalid metadata"]}
<br>Failure:
<br>{"error":"JSON-LD formatted item metadata must be sent with the api request"}
<br>or
<br>{"error":"JSON item metadata is ill-formatted"}
<br>or
<br>{"error":"Requested collection not found"}
<br>or
<br>{"error":"User is unauthorised"}
<br>or
<br>{"error":"The item &ltitem_id&gt already exists in the collection &ltcollection_name&gt"}
<br>or
<br>{"error":"identifier missing from document"}
<br>or
<br>{"error":"The identifier "&ltdocument_id&gt" is used for multiple documents"}
<br>or
<br>{"error":"content missing from document &ltdocument_id&gt"}
<br>or
<br>{"error":"The file &ltfilename&gt has already been uploaded to the collection &ltcollection_name&gt"}
<br>or
<br>{"error":"Error in file parameter."}
<br>or
<br>{"error":"Uploaded file is not present or empty."}
<br>or
<br>{"error":"No items were added"}
</td>
</tr>

<tr>
<td>Update an item</td>
<td>/catalog/{collection_name}/{item_name}</td>
<td>PUT</td>
<td>Result of operation (error/success)</td>
<td>Notes:
<ol>
<li>Users are only authorised to update an item from collection which they own.</li>
<li>This is a PUT request. Hence, cannot be replicated through a browser but through curl this can be done with something akin to the following.
<ul>
<li><code>curl -H "X-API-KEY: &ltkey&gt" -H "Content-Type: application/json" -H "Accept: application/json" -X PUT -d '{"metadata": &ltitem_metadata&gt}' &ltserver&gt/catalog/&ltcollection_id&gt/&ltitem_id&gt</code></li>
</ul>
</li>
</ol>
</td>
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
The following is an example of expected input for &ltitem_metadata&gt:
<ul>
<li><code>{"http://ns.ausnc.org.au/schemas/ausnc_md_model/mode":"An updated test mode"}</code></li>
<li><code>{"@context":{"ausnc":"http://ns.ausnc.org.au/schemas/ausnc_md_model/"}, "ausnc:speech_style":"An updated speech style"}</code></li>
</ul>
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<br>Success:
<br>{"success":"Updated item &ltitem_id&gt in collection &ltcollection_id&gt"}
<br>Failure:
<br>{"error":"Requested collection not found"}
<br>or
<br>{"error":"Requested item not found"}
<br>or
<br>{"error":"User is unauthorised"}
<br>or
<br>{"error":"Invalid metadata"}
</td>
</tr>

<tr>
<td>Delete an item from an owned collection</td>
<td>/catalog/{collection_id}/{item_id}</td>
<td>DELETE</td>
<td>Result of operation (error/success)</td>
<td>Notes:
<ol>
<li>Users are only authorised to delete an item from collection which they own.</li>
<li>Deleting an item from a collection also deletes all of that items documents and the corresponding document audits.</li>
<li>This is a DELETE request. Hence, cannot be replicated through a browser but through curl this can be done with something akin to the following.
<ul>
<li><code>curl -H "X-API-KEY: &ltapi_key&gt" -H "Accept: application/json" -X DELETE &ltserver&gt/catalog/&ltcollection_id&gt/&ltitem_id&gt</code></li>
</ul>
</li>
</ol>
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<br>Success:
<br>{"success":"Deleted the item &ltitem_id&gt (and its documents) from collection &ltcollection_id&gt"}
<br>Failure:
<br>{"error":"User is unauthorised"}
<br>or
<br>{"error":"Requested collection not found"}
<br>or
<br>{"error":"Requested item not found"}
</td>
</tr>

<tr>
<td>Add a document to an owned item</td>
<td>/catalog/{collection_id}/{item_id}</td>
<td>POST</td>
<td>Result of operation (error/success)</td>
<td>
Notes:
<ol>
<li>Users are only authorised to add documents to existing items in collections which they own.</li>
<li>When adding a document a shortened form of the "@id" which specifies the document URI needs to be supplied. This will be automatically converted into Alveo catalog URLs. For example, if adding a document named "document1.txt" then the shortened document URI <code>"@id":"document1.txt"</code> should be supplied and will be automatically converted to <code>"@id":"http://app.alveo.edu.au/catalog/test/item1/document/document1"</code>.</li>
<li>The identifiers in the supplied document metadata should have matching values, specifically the "@id" and the "dcterms:identifier" or "dc:identifier". This also applies to the document filename if a file is uploaded or a referenced file is used.</li>
<li> When uploading files or including the document content as JSON the document source metadata  <code>"dcterms:source":{"@id":"&ltfile_or_http_uri&gt"}</code> does not need to be supplied. Instead the system will automatically assign a location for these document files and generate this metadata accordingly. However it is essential to include the document source metadata term for any documents referenced with "file://" or "http://".</li>
<li>This is a POST request that requires a JSON-LD set of document metadata to be sent with it. Hence, cannot be replicated through a browser but through curl this can be done with something akin to the following.
<ul>
<li>If adding a a document referenced (with "file://" or "http://"): <code>curl -X POST -H "X-API-KEY: &ltkey&gt" -H "Accept: application/json" -H "Content-Type: application/json"  -d '{"metadata":{&ltitem_metadata&gt}' &ltserver&gt/catalog/&ltcollection_id&gt/&ltitem_id&gt</code></li>
<li>If adding a document whose content is embedded in JSON:
<code>curl -X POST -H "X-API-KEY: &ltkey>" -H "Accept: application/json" -H "Content-Type: application/json"  -d '{"document_content": "&ltdocument_content&gt", "metadata":{&ltitem_metadata&gt}' &ltserver&gt/catalog/&ltcollection_id&gt/&ltitem_id&gt</code></li>
<li>If adding an item with a single document uploaded as part of the HTTP request:
<code>curl -X POST -H "X-API-KEY: &ltkey&gt" -H "Accept: application/json" -F file=@"&ltfile_location&gt" -F metadata='{&ltitem_metadata&gt}' &ltserver&gt/catalog/&ltcollection_id&gt/&ltitem_id&gt</code></li>
</ul>
</li>
</ol>
</td>
</tr>
</tr>
<td>Example Input</td>
<td colspan=4>
<ul>
<li>If adding a document referenced (with "file://" or "http://"): <code>-d '{ "metadata": { "@context": { "dcterms": "http://purl.org/dc/terms/", "foaf": "http://xmlns.com/foaf/0.1/" }, "@id": "document2.txt", "@type": "foaf:Document", "dcterms:identifier": "document2.txt", "dcterms:title": "document2#Text", "dcterms:type": "Text", "dcterms:source": { "@id": "file:///data/test_collections/ausnc/test/document2.txt" } } }'</code></li>
<li>If adding a document whose contents are embedded in the JSON: <code>-d '{ "document_content": "Hello World!", "metadata": { "@context": { "dcterms": "http://purl.org/dc/terms/", "foaf": "http://xmlns.com/foaf/0.1/" }, "@id": "document2.txt", "@type": "foaf:Document", "dcterms:identifier": "document2.txt", "dcterms:title": "document2#Text", "dcterms:type": "Text" } }'</code></li>
<li>If adding a document uploaded as part of the HTTP request: <code>-F file=@"document2.txt" -F metadata='{ "@context": { "dcterms": "http://purl.org/dc/terms/", "foaf": "http://xmlns.com/foaf/0.1/" }, "@id": "document2.txt", "@type": "foaf:Document", "dcterms:identifier": "document2.txt", "dcterms:title": "document2#Text", "dcterms:type": "Text" }'</code></li>
</ul>
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4>
<br>Success:
<br>{"success":"Added the document &ltdocument_filename&gt to item &ltitem_id&gt in collection &ltcollection_id&gt"}
<br>Failure:
<br>{"error":"User is unauthorised"}
<br>or
<br>{"error":"Requested collection not found"}
<br>or
<br>{"error":"Requested item not found"}
<br>or
<br>{"error":"JSON-LD formatted item metadata must be sent with the api request"}
<br>or
<br>{"error":"JSON item metadata is ill-formatted"}
<br>or
<br>{"error":"The document &ltdocument_id&gt already exists in the collection &ltcollection_name&gt"}
<br>or
<br>{"error":"content missing from document &ltdocument_id&gt"}
<br>or
<br>{"error":"The file &ltfilename&gt has already been uploaded to the collection &ltcollection_name&gt"}
<br>or
<br>{"error":"Error in file parameter."}
<br>or
<br>{"error":"Uploaded file is not present or empty."}
</td>
</tr>

<tr>
<td>Delete a document from an owned item</td>
<td>/catalog/{collection_id}/{item_id}/document/{document_filename}</td>
<td>DELETE</td>
<td>Result of operation (error/success)</td>
<td>Notes:
<ol>
<li>Users are only authorised to delete documents from items in collections which they own.</li>
<li>This is a DELETE request. Hence, cannot be replicated through a browser but through curl this can be done with something akin to the following.
<ul>
<li><code>curl -H "X-API-KEY: &ltapi_key&gt"  -H "Accept: application/json" -X DELETE &ltserver&gt/catalog/&ltcollection_id&gt/&ltitem_id&gt/document/&ltdocument_filename&gt</code></li>
</ul>
</li>
</ol>
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<br>Success:
<br>{"success":"Deleted the document &ltdocument_filename&gt from item &ltitem_id&gt in collection &ltcollection_id&gt"}
<br>Failure:
<br>{"error":"User is unauthorised"}
<br>or
<br>{"error":"Requested collection not found"}
<br>or
<br>{"error":"Requested item not found"}
<br>or
<br>{"error":"Requested document not found"}
</td>
</tr>


</table>
