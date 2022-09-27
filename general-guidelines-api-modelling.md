### General guidelines on building HTTP API endpoints


#### Use standart/general use HTTP codes
 - Use  conventional http codes for general stuff, like:
 	- 200 Ok - generally for success
 	- 201 Created - when creating rest entity, and its successful
 	- 204 No Content - when deleting rest entity/usually by id/ and its deleted ok
 	- 400 - general client err, bad request from the client
 	- 401 Unautorized - missing or not present auth related header... etc
 	- 403 Forbidden - the client dont have access to the endpoint - acl, rbac permissions not ok
 	- 404 Not found - get rest entity by id not found, or in general cant find resource
 	- 405 Method not allowed - if we have the endpoint, but the method is not as defined from the client
 	....
 	- 500-5xx - server side errs

#### Use reasonable http methods

 - Lets use /users, with following endpoints/basic CRUD/, ONLY success http codes defined/used :
 	- GET 	.../v1/users 		- 200 Ok, retrieves users collection, with pagination, for the currently authenticated user organisation .. etc
 	- GET 	.../v1/users/123 	- 200 Ok, gets user entity with id:123 and for that current user org
 	- POST 	.../v1/users 		- 201 created, create user entity
 	- PATCH .../v1/users/123 	- 200 Ok, updates partially or FULLY user entity, defined with id:123, org
 	- PUT .../v1/users/123 	- 200 Ok, updates FULLY user entity, defined by its id: 123, org ... etc
 	- DELETE .../v1/users/123 	- 204 No content, deletes user by its id

#### Use ReST Subresources
If we want to add document to the load, atm we get the whole load, upload the file, and add the new document entity into the current load.Documents list
 - Instead of :
 	PUT .../v1/loads/{loadID}

 - use:
 	POST .../v1/loads/{loadID}/documents

#### Use expressive url namings - simpler, the better
 - Use simple names, instead:
 	GET .../v1/getUser?id=123  - get user entity by id: 123
 	PATCH .../v1/update-user-by-id/123 - update user with id:123

 	use:
 	GET .../v1/users/123     - get user entity by its id: 123
 	PATCH .../v1/users/123   - update user by its id: 123

#### All endpoints should have versioning
 - Versioning of the endpoints is a must, eg:
 	.../v1/users
 	.../v1/users/123

#### Standart/consistent err response

 - Instead of several err resp structs,:
	{"message": "asdasd", "code": 400}
	{"type": "validation_error", "errors": ["some validation err 1", ....]}
	...

 - Better use:
 	{"trace_id": "9a8sd8kjsfksdf7s6df", message": "asdas", "type": "handler|validation\...", "errors": ...}

#### Be consistent
 - When dealing with ReST endpoints use:
	.../users
	.../users/123

	Instead of 
	.../users
	.../user/123

 - Use same/geenrally accepted headers accross all endpoints
 - Use same err response struct
 - Use 404 for not found, 400 for general client err. Return 200 ok and inside : {"error": "Try again later ..."} omg

 
#### All date fields must be the RFC3339 format
 - Its again related to concistency, but having "2022-01-02", "2022-01-02T12:00:00+04-00", "2022-01-02T12:00:00Z"... Preferably in UTC/GMT time - its up to the 
 	presentation layer/client side/ to apply needed offset, depenging on the currenly logged in user

#### Everything should be protected/eg needs authorization/ unless otherwise
 - All endpoints need to have auth by default. Endpoints like /login, /register, /reset-password ... /info should be public

#### Prefer PATCH
 - Why - because PATCH is UPDATE ONLY - partial or full/all fields/ of the defined entity
 	PUT sometimes can be update, sometimes create - eg smth like replace - update OR create if missing

#### Return fully created entity after POST .../v1/users
  - when creating user for instance, resp code is 201 Created, and body should return the fully created entity data, not just status: OK, user_id:123, message: user created

 #### ALL Collection responses must be paginated!
  - All ReST collection responses must have pagination - so that way its way easier to havigate, increase/decrease page size, know first, next, prev, and last page
 #### ReST Entities Hydration 
  - Not must have, but way better if implemented

#### Entities should include subresource links, not the real entities by default
 - Imagine order and order lines - normally, when getting order by id /for instance/, we have ord lines as links
 ```
 {
    "id": 123,
    "order_number": "000123123",
    "email": "john@noname.com",
    ..
    order_lines:[
        "https://.../v1/orders/123/order_lines/345",
        ...
    ],
    ...
 }
```
If we need the real/full entities, we must define param, like url...?expand=true, or ..?detailed=true. Then the response will have not list with order lines links, but the fully composed order lines entities, like:
```
{
    "id": 123,
    "order_number": "000123123",
    "email": "john@noname.com",
    ..
    order_lines:[
        {
            "id": 345,
            "item_id": 111,
            "qty": 2,
            ...
        }
    ],
    ...
 }
 ```
