# Getting started with Angular

Angular is a platform and framework for building single-page client applications using HTML and TypeScript. Angular is written in TypeScript. It implements core and optional functionality as a set of TypeScript libraries that you import into your apps.

> [!TIP] HTTP request parts
>
> - Headers: we use headers to send additional information to the server, like the content type, the authorization token, etc, these informations are wanted before processing the request (reading the body, the query parameters, etc).
> - Body: when data have a very large volume and i need into be secure i use the body, secure as it doesn't appear in the URL like the query parameters or the query string.
> - query parameters
> - Query string

> [!NOTE] HTTP request methods
>
> The most common HTTP methods are:
>
> - GET: to get data from the server.
> - POST: to send data to the server.
> - PUT: to update data on the server.
> - DELETE: to delete data from the server.
> - PATCH: to update data partially on the server.

> [!Question] Get vs Post
>
> | Get                                            | Post                                                        |
> | ---------------------------------------------- | ----------------------------------------------------------- |
> | Used to request data from a specified resource | Used to submit data to be processed to a specified resource |
> | data is sent in the URL                        | data is sent in the body                                    |
> | data is visible to everyone in the URL         | data is not visible to everyone in the body                 |

> [!question] can i send data in the body if the method is GET?
>
> No, you can't send data in the body if the method is GET, you can only send data in the body if the method is POST, PUT, PATCH, DELETE, etc.

> [!question] Is the body data encoded by default?
>
> Yes, the body data is encoded by default, and the most common encoding type is `application/x-www-form-urlencoded`.

> [!TIP] HTTP status codes
>
> - 1xx: Informational
> - 2xx: Success
> - 3xx: Redirection
> - 4xx: Client Error
> - 5xx: Server Error

> [!TIP] Response parts
>
> It's usually a json object that contains the following parts:
>
> - Success: a boolean that indicates if the request was successful or not.
> - Data: the data that the server sends back to the client.
> - Message: a message that the server sends back to the client.
> - Status code: the status code of the response.
>
> some additional info maybe sent like pagination info, etc.
