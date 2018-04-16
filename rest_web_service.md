### Practical REST Web Services
#### REST Introduction
REST Web Services Open up access to your web-server's functionality : programmatic clients can also connect via HTTP such as: mobile applications, microservices or browsers: SPA, AJAX

* REST is an architectural style that describes best practices to expose web services over HTTP
	* **RE**presentational **S**tate **T**ransfer, term by Roy Fielding
	* HTTP as application protocol, not just transport
	* Emphasizes scalability
	* Not a framework or specification


#### REST Principles
* Expose resources through URIs: model nouns, not verbs
* Resources support limited set of operations GET, PUT, POST, DELETE in case of HTTP (all have well-defined semantics)
	* Example: update an order  PUT to /orders/123  don't POST to /order/edit?id=123
</br>
==> HTTP Methods + Resource URIs  = Uniform interface


* Clients can request particular representation (Resources can support multiple representations: HTML, XML, JSON, … )
* Representations can link to other resources
	* Allows for extensions and discovery, like with web sites
* Hypermedia As The Engine of Application State
	* HATEOAS: Probably the world's worst acronym!
	* RESTful responses contain the links you need 	* justlike HTML pages do


* Stateless architecture
	* No HttpSession usage
	* GETs can be cached on URL
	* Requires clients to keep track of state
	* Part of what makes it scalable
	* Looser coupling between client and server
* HTTP headers and status codes communicate result to clients (All well-defined in HTTP Specification)


#### Why REST? Benefits of REST
* Every platform/language supports HTTP
* Unlike for example SOAP + WS-* specs
	* Easy to support many different clients: Scripts, Browsers, Applications
	* Scalability
	* Support for redirect, caching, different representations, resource identification, …
	* Support for multiple formats ( JSON and Atom are popular choices )


REST and Java: JAX-RS
* JAX-RS is a Java EE 6 standard for building RESTful applications focuses on programmatic clients, not browsers.
* There are Various implementations: Jersey (RI), RESTEasy, Restlet, CXF
	* All implementations provide Spring support
* Good option for full REST support using a standard


REST and Java: Spring-MVC
* Spring-MVC provides REST support as well
	* Using familiar and consistent programming model
	* Spring MVC does not implement JAX-RS
* Single web-application for everything
	* Traditional web-site: HTML, browsers
	* Programmatic client support (RESTful web applications, HTTP-based web services)
* RestTemplate for building programmatic clients in Java


Spring-MVC and REST
* Extending Spring MVC to support REST
	* Map requests based on HTTP method
	* Define response status
	* Message Converters
	* Access request and response body data
	* Build valid Location URIs *
* For HTTP POST responses


#### HTTP GET: Fetch a Resource
* Requirement
	* Respond only to GET requests
	* Return requested data in the HTTP Response
	* Determine requested response format


```json
GET /store/orders/123
Host: shop.spring.io
Accept: application/json, ...
...
```


```json
HTTP/1.1 200 OK
Date: …
Content-Length: 756
Content-Type: application/json
{
"order": {
"id": 123,
"items": [ … ], … }
}
```

#### Request Mapping Based on HTTP Method
* Map HTTP requests based on method allows same URL to be mapped to multiple Java methods.
* RequestMethod enumerators are: GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS, TRACE

```java
	// Get all orders (for current user typically)
	@RequestMapping(path="/orders", method=RequestMethod.GET)
	// Create a new order
	@RequestMapping(path="/orders", method=RequestMethod.POST)
```

#### Simpler Mapping Annotations ( Since Spring 4.3 )
* Alternative handler mapping shortcuts

```java
	@RequestMapping(path="/accounts”,method=RequestMethod.GET)
	//Or
	@GetMapping("/accounts")
```

* Exist for these HTTP methods
	* @GetMapping
	* @PostMapping
	* @PutMapping
	* @DeleteMapping
	* @PatchMapping

For HEAD, OPTIONS, TRACE use RequestMethod enumerators


#### Generating Response Data
* The Problem
	* HTTP GET needs to return data in response body typically in XML or JSON (Preferd to work with Java objects )
	* Avoid converting to formats manually
* The Solution
	* Marshaling via dedicated **message-converters**
	* Annotate response data with @ResponseBody

```json
HTTP/1.1 200 OK
Date: …
Content-Length: 756
Content-Type: application/json
{
"order": {
"id": 123,
"items": [ … ], … }
}
```

**HttpMessageConverter** converts HTTP request/response body data
* XML (using JAXP Source, JAXB2 mapped object, Jackson-Dataformat-XML*)
* Jackson JSON*, GSON*Feed data* such as Atom/RSS
* Google protocol buffers *
* Form-based data
* Byte[], String, BufferedImage

	( Requires 3rd party libraries on classpath )
* Must enable (or no convertors defined at all!)
	* Automatic with Spring Boot
	* Or use @EnableWebMvc
	* Or define explicitly: WebMvcConfigurer
* Allows you to register extra convertors




@ResponseBody
* Use converters for response data by annotating return data with @ResponseBody
* Converter handles rendering a response
	* No ViewResolver and View involved any more!

```java
@GetMapping(path="/orders/{id}")
public @ResponseBody Order getOrder(@PathVariable("id") long id) { ... }
```
If you forget @ResponseBody, Spring MVC attempts to find a View (and fails)


#### Retrieving a Representation: GET
``` json
GET /store/orders/123
Host: shop.spring.io
Accept: application/json
...
```
``` json
HTTP/1.1 200 OK
Date: ...
Content-Length: 1456
Content-Type: application/json
{
"order": {
"id": 123,
"items": [ … ], … }
}
```

``` java
@GetMapping(path="/orders/{id}")
public @ResponseBody Order getOrder(@PathVariable("id") long id) {
return orderService.findOrderById(id);
}
```
``` java
@RequestMapping(path="/orders/{id}", method=RequestMethod.GET)
```

#### What Return Format? Accept Header
``` java
@GetMapping(path="/orders/{id}")
public @ResponseBody Order getOrder(@PathVariable("id") long id) {
return orderService.findOrderById(id);
}
```
``` json
GET /store/orders/123
Host: shop.spring.io
Accept: application/xml
...
```

``` xml
HTTP/1.1 200 OK
Date: …
Content-Length: 1456
Content-Type: application/xml
<order id=”123”>
…
</order>
```




``` json
GET /store/orders/123
Host: shop.spring.io
Accept: application/json
...
```
``` json
HTTP/1.1 200 OK
Date: …
Content-Length: 756
Content-Type: application/json
{
"order": {"id": 123,
"items": [ … ], … }
}
```

#### @RestController Simplification (Spring 4.x)
```java
@Controller
public class OrderController {

	@GetMapping(path="/orders/{id}")
	public @ResponseBody Order getOrder(@PathVariable("id") long id) {
	return orderService.findOrderById(id);
	}

	…
}
```
``` java
@RestController
public class OrderController {

	@GetMapping(path="/orders/{id}")
	public Order getOrder(@PathVariable("id") long id) {
	return orderService.findOrderById(id);
	}

	…
}
```
 ==> No need for @ResponseBody on GET methods

3
HttpEntity and ResponseEntity
* To build responses explicitly
	* Set headers, control content returned
	* Use HttpEntity or ResponseEntity

``` java
// Want to return a String as the response-body
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.TEXT_PLAIN);

HttpEntity<String> entity = new HttpEntity<String>("Hello Spring", headers);
// ResponseEntity (since Spring 4.1) supports a “fluent” API
ResponseEntity<String> response = ResponseEntity.ok()
												.contentType(MediaType.TEXT_PLAIN)
												.body("Hello Spring");
```

Setting Response Data
* Can use HttpEntity to generate a Response
	* Avoids use of HttpServletResponse (easier to test)

```java
@GetMapping(path="/orders/{id}")
public HttpEntity<Order> getOrder(long id) {
	Order order = orderService.find(id);
	return ResponseEntity.ok()
						.header("Last-Modified", order.lastUpdated())
						.body(order);
}
```
Response
body
HTTP Status 200 OK
Set extra
or custom
No need for header
@ResponseBody


#### HTTP PUT: Update a Resource
* Requirement
	* Respond only to PUT requests
	* Access data in the HTTP Request
	* Return empty response, status 204


```json
PUT /store/orders/123/items/abc
Host: www.mybank.com
Content-Type: application/json
{
"order": {
"id": 123, "items": [ … ], … }
}
```
```json
HTTP/1.1 204 No Content
Date: …
Content-Length: 0
…
```

Successful update – nothing to return


HTTP Status Code Support
* Web apps just use a handful of status codes
	* Success: 200 OK
	* Redirect: 30x for Redirects
	* Client Error: 404 Not Found
	* Server Error: 500 (such as unhandled Exceptions)
* RESTful applications use many additional codes
	* Created Successfully: 201
	* HTTP method not supported: 405
	* Cannot generate requested response body format: 406
	* Request body not supported: 415
For a full list: https://en.wikipedia.org/wiki/List_of_HTTP_status_codes


@ResponseStatus
* To return a status code other than 200 use HttpStatus enumerator
* Note: @ResponseStatus on void methods
	* No longer want to return a view name - no View at all!
	* Method returns a response with empty body (no-content )

```java
@PutMapping("/orders/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT) // 204
public void updateOrder(...) {
// Update order
}
```


@RequestBody
* Use message converters for incoming request data
* A correct converter is chosen automatically based on content type of request
* UpdatedOrder could be mapped from XML (with JAXB2) or from JSON (with Jackson)
* Annotate Order class (if need be) for JAXB/Jackson to work

```java
@PutMapping(path="/orders/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT) // 204
public void updateOrder(@RequestBody Order updatedOrder,@PathVariable("id") long id) {
	// process updated order data and return empty response
	orderManager.updateOrder(id, updatedOrder);
}
```
0
Updating Existing Resource: PUT
```java
@PutMapping(path="/orders/{orderId}/items/{itemId}")
@ResponseStatus(HttpStatus.NO_CONTENT) // 204
		public void updateItem(@PathVariable("orderId") long orderId,
								@PathVariable("itemId") String itemId,
								@RequestBody Item item) {
		orderService.findOrderById(orderId).updateItem(itemId, item);
}
```
```json
HTTP/1.1 204 No Content
Date: …
Content-Length: 0
```
```json
PUT /store/orders/123/items/abc
Host: shop.spring.io
Content-Type: application/json
{
"order": {
"id": 123, "items": [ … ], … }
}
```
```java
@RequestMapping(path="/orders/...", method=RequestMethod.PUT)
```

#### HTTP POST: Create a new Resource
* Requirement
	* Respond only to POST requests
	* Access data in the HTTP Request
	* Return “created”, status 201
	* Generate Location header for newly created resource

```json
HTTP/1.1 201 Created
Date: ...
Content-Length: 0
Location: http://shop.spring.io/
store/orders/123/items/abc
```
```json
POST /store/orders/123/items
Host: shop.spring.io
Content-Type: application/json
{
"order": {
"id": 123,
"items": [ … ], … }
}
```
```java
@PostMapping(path="/orders/{id}/items")
public ??? createItem
(@PathVariable("id") long id, @RequestBody Item newItem)
{
// Add the new item to the order
orderService.findOrderById(id).addItem(newItem);
return ???;
}
```
```java
@RequestMapping(path="/orders/...", method=RequestMethod.POST)
```
 We can already implement most of this requirement, but how do we return the new Item location?


Building URIs
* An HTTP POST typically returns location of newly created resource in the response header
* How to create a URI?
	* **UriComponentsBuilder** allows explicit creation of URI, but uses hard-coded URLs.
		* Support for building URIs from URI template strings
		* Escapes illegal characters 	* such as %20 for a space
```java
String templateUrl = "http://store.spring.io/orders/{orderId}/items/{itemId}";	//BUT: Use of hard-coded URL not recommended
URI location = UriComponentsBuilder.fromHttpUrl(templateUrl)
	.buildAndExpand("123456","itemA").toUri();
return ResponseEntity.created(location).build();	//Convenient way to build POST response
// http://store.spring.io/orders/123456/items/item%20A
```
	* **ServletUriComponentsBuilder** provides access to the URL that invoked the current controller method
		* Use in a Controller method
		* Avoids hard-coding URL Framework puts request
```java
// Must be in a Controller method
// Example: POST /orders/{id}/items
URI location = ServletUriComponentsBuilder
												.fromCurrentRequestUri()		//URL in current thread – which builder can access
												.path("{itemId}")
												.buildAndExpand("item A")
												.toUri();
return ResponseEntity.created(location).build();
// http://.../items/item%20A
```








Better: ServletUriComponentsBuilder



page 749




Creating a new Resource: POST
```java
@PostMapping(path="/orders/{id}/items")
public ResponseEntity<Void> createItem
(@PathVariable("id") long id, @RequestBody Item newItem) {
		// Add the new item to the order
		orderService.findOrderById(id).addItem(newItem);
		// Build the location URI of the new item
		URI location = ServletUriComponentsBuilder.fromCurrentRequestUri()
												.path("{itemId}")
												.buildAndExpand(newItem.getId())
												.toUri();
		// Explicitly create a 201 Created response
		return ResponseEntity.created(location).build();
}
```
```java
@RequestMapping(path="/orders/...", method=RequestMethod.POST)
```
Assume this
call also set
an item-id
@ResponseStatus
not needed

#### HTTP DELETE: Delete a new Resource
* Requirement
	* Respond only to DELETE requests
	* Return empty response, status 204
	```json
HTTP/1.1 204 No Content
Date: …
Content-Length: 0
```
```json
DELETE /store/orders/123/items/abc
Host: shop.spring.io
Content-Length: 0
...
```


Deleting a Resource: DELETE
```java
@DeleteMapping(path="/orders/{orderId}/items/{itemId}")
@ResponseStatus(HttpStatus.NO_CONTENT) // 204
public void deleteItem(@PathVariable("orderId") long orderId, @PathVariable("itemId") String itemId) {
	orderService.findOrderById(orderId).deleteItem(itemId);
}
```

```json
DELETE /store/orders/123/items/abc
Host: shop.spring.io
...
```

```json
HTTP/1.1 204 No Content
Date: …
Content-Length: 0
```


```java
@RequestMapping(path="/orders/...", method=RequestMethod.DELETE)
```

1
Putting it all Together
* Many new concepts
	* @ResponseStatus
	* HTTP Message Converters
	* @RequestBody, @ResponseBody
	* @RestController
	* HttpEntity, ResponseEntity
	* ServletUriComponentsBuilder
What we have
learned?

##### RestTemplate
RestTemplate provides access to RESTful services
</br> It supports all the HTTP methods.

|HTTP Method |RestTemplate Method|
|:-------------:|:-------------|
|DELETE |delete(String url, Object… urlVariables)|
|GET |getForObject(String url, Class<T> responseType, Object… urlVariables)|
|HEAD |headForHeaders(String url, Object… urlVariables)|
|OPTIONS |optionsForAllow(String url, Object… urlVariables)|
|POST |postForLocation(String url, Object request, Object… urlVariables)|
||postForObject(String url, Object request, Class<T> responseType, Object… uriVariables)|
|PUT | put(String url, Object request, Object… urlVariables)|

4
Defining a RestTemplate
* Just call constructor in your code
	* Setups default HttpMessageConverters internally
* Same as on the server, depending on classpath
```java
RestTemplate template = new RestTemplate();
```
5
RestTemplate Usage Examples
```java
RestTemplate template = new RestTemplate();
String uri = "http://example.com/store/orders/{id}/items";
// GET all order items for an existing order with ID 1:
OrderItem[] items = template.getForObject(uri, OrderItem[].class, "1");		// {id} = 1
// POST to create a new item
OrderItem item = // create item object
URI itemLocation = template.postForLocation(uri, item, "1");	// {id} = 1
// PUT to update the item
item.setAmount(2);
template.put(itemLocation, item);
// DELETE to remove that item again
template.delete(itemLocation);
```



Using HttpEntity and ResponseEntity
* Access response headers and body
```java
ResponseEntity<String> response = restTemplate.getForEntity(itemUrl, String.class);		//Body returned as String, but object mapping still an option
assert(response.getStatusCode().equals(HttpStatus.OK));
String content = response.getBody();	//Body returned as String, but object mapping still an option
```

* Setup your own request
```java
HttpEntity<OrderItem> request = new HttpEntity<>(item);
request.getHeaders().add(HttpHeaders.AUTHORIZATION, "Basic " + getBase64EncodedLogPass());
ResponseEntity<Void> response = restTemplate.exchange(itemUrl, HttpMethod.POST, request, Void.class);
assert(response.getStatusCode().equals(HttpStatus.CREATED));
```



Summary
* REST is an architectural style that can be applied to HTTP-based applications
	* Useful for supporting diverse clients and building highly scalable systems
* Spring-MVC adds REST support using a familiar programming model (but without Views)
	* @ResponseStatus, @RequestBody, @ResponseBody
	* HttpEntity, ResponseEntity, UriComponentsBuilder
	* HTTP Message Converters
* Clients use RestTemplate to access RESTful servers
