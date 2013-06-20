stampr postal mail api
======================

The stampr api allows access to advanced features provided by https://stam.pr/ including variable 
data and integration with third-party systems.

**Endpoint:** 
`https://stam.pr` -- `GET https://stam.pr/api/health` 

**Throttling:** 
10 req/s

## Overview

The API can be broken up into 3 objects:

* Batch (container for mailings)
* Config (configuration for mailings)
* Mailing

### Data Relationships:

* Batches contain Mailings
* Batches have exactly one Config
* Configs can be used for multiple Batches
* Mailings must belong to a Batch

### Diagram

<pre>
  ____________________________
 /           _______________   \
|           /               \   |
|  Batches |    Mailings    |   |
|           \______________/    |
 \_____________________________/
     |
   Configs
</pre>

### Example in Pseudo Code

<pre>
Config = CreateConfig(configOptions)
Batch = CreateBatch(Config, batchOptions)
for (i = 1 to 100)
{
  Mailing = CreateMailing(Batch, mailingOptions)
}
</pre>

## Authentication

* Basic
* HMAC (Coming soon)


## Configs

Mail configurations are largely placeholders for future content.  

### Create New Config

`POST /api/configs`

Returns: `Config` Object

Create a new mailing configuration to be used with Batches.

<table border="1">
	<tr>
		<th>size (string)</th>
		<td>
			Required<br>
			<br>
			Valid values (case sensitive):
			* "standard"<br>
			* "postcard"<br>
			* "legal"<br>
			<br>
			Currently, only standard is supported.  <br>
			<br>
			** This should be hidden in the API, and always send "standard" ** 
		</td>
	</tr>
	<tr>
		<th>turnaround (string)</th>
		<td>
			Required<br>
			<br>
			Valid values (case sensitive):<br>
			* "weekend"<br>
			* "overnight"<br>
			* "threeday"<br>
			* "week"<br>
			<br>
			Currently ignored.  All mailings are processed in the order they are received regardless of the value here.<br>
			<br>
			** This should be hidden in the API, and always send "threeday" ** 
		</td>
	</tr>
	<tr>
		<th>style (string)</th>
		<td>
			Required<br>
			<br>
			Valid values (case sensitive):<br>
			* "color"<br>
			* "mono"<br>
			<br>
			Currently, only color is supported.<br>
			<br>
			** This should be hidden in the API, and always send "color" ** 
		</td>
	</tr>
	<tr>
		<th>output (string)</th>
		<td>
			Required<br>
			<br>
			Valid values (case sensitive):
			* "single"<br>
			* "double"<br>
			<br>
			Currently, only single is supported.<br>
			<br>
			** This should be hidden in the API, and always send "single" ** 
		</td>
	</tr>
	<tr>
		<th>returnenvelope (bool)</th>
		<td>
			Required<br>
			<br>
			Currently ignored.<br>	
			<br>
			** This should be hidden in the API, and always send `false` ** 
		</td>
	</tr>
</table>


### Search/List Configs

<pre>
GET /api/configs/:id
GET /api/configs/browse/all
GET /api/configs/browse/all/:paging
</pre>

Returns: `Config[]` (Paged Array of Config Objects)

* paging - 0-based page index (page size is set on the server and cannot be changed at this time)


## Batches

### Create New Batch

`POST /api/batches`

Returns: `Batch` Object

Create a new, empty batch container to be used with Mailings.

<table border="1">
	<tr>
		<th>config_id (integer)</th>
		<td>
			Required<br>
			<br>
			The configuration all the mailings in this batch will conform to.
		</td>
	</tr>
	<tr>
		<th>template (string/HTML)</th>
		<td>
			Optional<br>
			<br>
			This will be the base HTML template used for all mailings created in this batch.  Templates use mustache syntax. -- Details: http://mustache.github.io/<br>
			<br>
			Example Template:<br>
			<br>
			"Hello {{name}}!"<br>
			<br>
			And then `$name` can be sent via the Mailing and the template will be merged with the data and rendered.
		</td>
	</tr>
	<tr>
		<th>status (string)</th>
		<td>
			Optional<br>
			<br>
			Valid values (case sensitive):<br>
			* "processing" (default, if excluded)<br>
			* "hold"<br>
			<br>
			When mailings are created, they are printed and shipped as soon as possible.  Set a batch to "hold" and mailing will not be printed until the batch status is changed to processing.
		</td>
	</tr>
</table>


### Modify Batch by Batch ID

`POST /api/batches/:id`

Returns: `true` on success

<table border="1">
	<tr>
		<th>batch_id (integer)</th>
		<td>
			Required<br>
			<br>
			The batch to be updated.
		</td>
	</tr>
	<tr>
		<th>status (string)</th>
		<td>
			Required<br>
			<br>
			Valid values (case sensitive):<br>
			-"processing"<br>
			-"hold"<br>
			-"archive"<br>
			<br>
			If changing from processing to hold, all mailings currently in production will be sent.  
		</td>
	</tr>
</table>



### Delete a Batch by ID

`DELETE /api/batches/:id`

Returns: `true` on success

Batches that contain mailings cannot be deleted.  Only empty batches are allowed.


### Search/List Batches

<pre>
GET /api/batches/:id
GET /api/batches/with/:status
GET /api/batches/with/:status/:start
GET /api/batches/with/:status/:start/:end
GET /api/batches/with/:status/:start/:end/:paging
GET /api/batches/browse/:start
GET /api/batches/browse/:start/:end
GET /api/batches/browse/:start/:end/:paging
</pre>

Returns: `Batch[]` (Paged Array of Batch Objects)

-status - Any valid Batch status -- `'processing', 'hold'`
-start - Any date time that can be parsed by moment.js
-end - Any date time that can be parsed by moment.js
-paging - 0-based page index (page size is set on the server and cannot be changed at this time)


### Search/List Mailings belonging to Batch

<pre>
GET /api/batches/:id/mailings
GET /api/batches/:id/mailings/with/:status
GET /api/batches/:id/mailings/with/:status/:start
GET /api/batches/:id/mailings/with/:status/:start/:end
GET /api/batches/:id/mailings/with/:status/:start/:end/:paging
GET /api/batches/:id/mailings/browse/:start
GET /api/batches/:id/mailings/browse/:start/:end
GET /api/batches/:id/mailings/browse/:start/:end/:paging
</pre>

Returns: `Mailing[]` (Paged Array of Mailing Objects)

* status - Any valid Mailing status -- `'received','render','error','queued','assigned','processing','printed','shipped'`
* start - Any date time that can be parsed by moment.js.  It should be URL encoded.
* end - Any date time that can be parsed by moment.js.  It should be URL encoded.
* paging - 0-based page index (page size is set on the server and cannot be changed at this time)


## Mailings

### Create New Mailing

`POST /api/mailings`

Returns: `Mailing` Object

<table border="1">
	<tr>
		<th>batch_id (integer)</th>
		<td>
			Required<br>
			<br>
			The Batch the Mailing belongs to.
		</td>
	</tr>
	<tr>
		<th>address (string)</th>
		<td>
			Required<br>
			<br>
			The postal mailing address to send the mailing to.  <br>
			<br>
			(Not Implement Yet.  This Can be Ignored) String provided should contain an address that conforms to the USPS CASS system -- http://en.wikipedia.org/wiki/Coding_Accuracy_Support_System
		</td>
	</tr>
	<tr>
		<th>returnaddress (string)</th>
		<td>
			Required<br>
			<br>
			The postal return mailing address.<br>
			<br>
			(Not Implement Yet.  This Can be Ignored) String provided should contain an address that conforms to the USPS CASS system -- http://en.wikipedia.org/wiki/Coding_Accuracy_Support_System
		</td>
	</tr>
	<tr>
		<th>format (string)</th>
		<td>
			Required<br>
			<br>
			Valid values (case sensitive):<br>
			-"json" - Key/Value data to be merged and rendered into Batch template<br>
			-"html" - HTML to be rendered and sent as-is<br>
			-"pdf" - PDF to be mailed<br>
			-"none" - No data being provided.  Send Batch template as-is.<br>
			<br>
			This is the format of the data being provided.<br>
			<br>
			-JSON is useful for variable-data mailings (mail merge), when combined with Batch template.<br>
			-HTML is useful for the same reason, but provides more control to the end-user.  <br>
			-PDF is identical to HTML, only data is rendered as PDF.<br>
			-None allows many duplicate mailings (Batch template) to be sent.
		</td>
	</tr>
	<tr>
		<th>data (string)</th>
		<td>
			Required, unless format is "none"<br>
			<br>
			The data payload in the format specified.<br>
			<br>
			Data should be always be base64 encoded.<br>
		       Example: 
			<code>
			HtmlData = '<p>hello</p>'
			HtmlMailing.Data = Base64(HtmlData)

			PdfData = io.File('my.pdf')
			PdfMailing.Data = Base64(PdfData)
			</code>
		</td>
	</tr>
	<tr>
		<th>md5 (string) -- lowercase hex, case-sensitive</th>
		<td>
			Optional<br>
			<br>
			To ensure data integrity, an optional MD5 hash of the data can be supplied.  See `data` above for details on how to encode data prior to md5 hashing.<br>
			It is calculated as follows:<br>
			<code>
				RawData = Raw_User_Data;<br>
				Data = Base64( RawData );<br>
				Md5Hash = Md5( Data );<br>
			</code>
		</td>
	</tr>
</table>

### Delete a Mailing by Mailing ID

`DELETE /api/mailings/:id`

Returns: `true` on success

In order to delete a mailing, the mailing's status must be `received`.  Mailing statuses will only be received if the owning batch has a status of `hold`.


### Search/List Mailings 

<pre>
GET /api/mailings/:id
GET /api/mailings/with/:status
GET /api/mailings/with/:status/:start
GET /api/mailings/with/:status/:start/:end
GET /api/mailings/with/:status/:start/:end/:paging
GET /api/mailings/browse/:start
GET /api/mailings/browse/:start/:end
GET /api/mailings/browse/:start/:end/:paging
</pre>

Returns: `Mailing[]` (Paged Array of Mailing Objects)

* status - Any valid Mailing status -- `'received','render','error','queued','assigned','processing','printed','shipped'`
* start - Any date time that can be parsed by moment.js
* end - Any date time that can be parsed by moment.js
* paging - 0-based page index (page size is set on the server and cannot be changed at this time)

## Tests

### Simple Round Trip Test

`GET /api/test/ping`

Returns: Object with single key "pong" and a value of the current Date/Time of the server


### Check API health

`GET /api/health`

Returns: `string`

Returns the exact string `"ok"` (with quotes) when everything is ok. =]
