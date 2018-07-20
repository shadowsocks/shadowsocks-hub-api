# Shadowsocks Hub API
Shadowsocks Hub API provides a set of open and standard restful APIs for managing shadowsocks users, servers, nodes, products, accounts, and traffic. It is best suitable for companies, organizations, and groups of people to manage their internal shadowsocks infrastructures. 

Shadowsocks Hub API enables any developers to conveniently develop their own shadowsocks management UIs without reinventing the wheels of writing server-end logic. All common features have been made available in the form of restful APIs.

Shadowsocks Hub API is developed using Nodejs. It uses MySQL as its underlying database and shadowsocks-libev as its shadowsocks implementation. 

## Install (Ubuntu 16.04)

1. Install Nodejs 6 or above (8 preferred).
2. Install MySQL.
3. Download and install Shadowsocks Hub API:
    ```
    cd ~
    git clone https://github.com/shadowsocks/shadowsocks-hub-api.git
    cd ~/shadowsocks-hub-api
    npm i
    ```
4. Create a MySQL database `sshub`:
    ```
    CREATE DATABASE sshub;
    ```
5. Create an environment file `.env`:
    ```
    cd ~/shadowsocks-hub-api
    touch .env
    ```
6. Add the following configuration key value pairs to `.env`:
    ```
    JWT_SECRET = 2wk0M@ow094B^&9k3==~o2soejd$sEEo@2(
    LISTEN_PORT = 8000

    DATABASE_HOST = localhost
    DATABASE_PORT = 3306
    DATABASE_USER = root
    DATABASE_PASSWORD = d4f889df22769f54
    ```

   Change the values about the database connection to your local configuration.  
   Change the value of `JWT_SECRET` with a long and random string.  
   Change the value `LISTEN_PORT` to your preferred port for Shadowsocks Hub API to listen to.  

7. Initialize database:
    ```
    cd ~/shadowsocks-hub-api
    knex migrate:latest --env production
    ```
8. Setup digital certificate

   All requests and responses are encrypted using https. It requires you to set up a digital certificate. You may create your own self-signed certificate:  
    ```
    cd ~/shadowsocks-hub-api
    openssl req -nodes -x509 -newkey rsa:4096 -keyout server.key -out server.cert -days 365
    ```

   Alternatively, you may copy your digital certificate and key pair that you obtained from a Certificate Authority (e.g. Let's Encrypt) to `~/shadowsocks-hub-api` with the certificate named as `server.cert` and the key named as `server.key`.  

9. Shadowsocks Hub API utilize [shadowsocks-restful-api](https://github.com/shadowsocks/shadowsocks-restful-api) to manage shadowsocks protocol. Install it on every server that you would like to use as a shadowsocks node.

## Update
If you have updated Shadowsocks Hub API from an older version to the latest version, run:
```
cd ~/shadowsocks-hub-api
knex migrate:latest --env production
```

## Run
1. Run Shadowsocks Hub API:
    ```
    cd ~/shadowsocks-hub-api
    node api.js
    ```

2. Change admin credential

   For the sake of security, you should change the default admin user credential as soon as possible. This can be done by using the `login` API to obtain a token, and then using the `update user` API to change the username and password of the admin user. The default username and password for admin user are `admin@email.com` and `pleaseChangePassword`, respectively.

3. Run [shadowsocks-restful-api](https://github.com/shadowsocks/shadowsocks-restful-api) on every server that you would like to use as a shadowsocks node.

## Authentication

All the APIs except for the `login` API require an Authorization header. The header pattern is: `Authorization: Bearer <token>`. The token can be obtained from the `login` or `refresh token` API. The authentication is compatiable with [OAuth 2](https://oauth.net/2/). There are 3 APIs relating to authentication: `login`, `refresh token`, and `invalidate refresh token`.

Access token expires in 15 minutes while refresh token exipires in 1 week. Both `login` and `refresh token` APIs will produce a pair of new access token and refresh token. The difference is that `login` API requires `username` and `password` while `refresh token` API requires a valid refresh token to be provided in the `Authorization Bearer` header.

The `invalidate refresh token` API can be used to invalidate all previously obtained refresh tokens.

## Rate limit

A rate limit is applied to all APIs. The maximum number of requests allowed within 15-minute window from the same ip address is limited to 50. Requests exceeding the threshold will be refused with HTTP status code `429 Too Many Requests`.


## Bug report and feature request

Bug report and feature request are welcome. Bugs have a high priority to get addressed. Feature requests will be considered depending on their popularity and importance.

## APIs

### 1. User APIs
There are two type of users: `admin user` and `normal user`. This API is [OAuth 2](https://oauth.net/2/) compatiable. Admin user has priviliege to make any API requests. Normal user has priviliege to make API requests relating to themselves only.

#### Login
Both `admin user` and `normal user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  POST                                                             |
| Request URL:                  |  https://host_name:port/api/user/login                            |
| Request Header:               |  Content-Type: application/json                                   |
| Request Body:                 |  {"username":"your_username", "password":"your_login_password"}   |
| Response HTTP Status Code:    |  201 Created                                                      |
| Response Body:                |  {"token":"new_authentication_token", "refreshToken":"new_refresh_token"} |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -X POST -d '{"username":"admin@email.com","password":"pleaseChangePassword"}' https://localhost:8000/api/user/login
```
Response example:  
```json
{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVkYWVmYjQ3LTI4YzYtNDA1OC1hMDBiLTc1ZGI1MTQ0OGJkNyIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMTIyNDMzMiwiZXhwIjoxNTMxMjI1MjMyfQ.h82mSltnLua-XLPHyV7X2-lqe94O7dYE7Ujachg3NDY","refreshToken":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVkYWVmYjQ3LTI4YzYtNDA1OC1hMDBiLTc1ZGI1MTQ0OGJkNyIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMTIyNDMzMiwiZXhwIjoxNTMxODI5MTMyfQ.PIL-thIdKr2ji5LDOelLXPfIZQUvo3II1KrOG0-lLx4"}
```

#### Refresh Token
Both `admin user` and `normal user` can call this API. This API is [OAuth 2](https://oauth.net/2/) compatiable. You will get a set of new `token` and `refreshToken`. Note that you need supply `refreshToken` instead of `token` to `Authorization Bearer` header.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  GET                                                              |
| Request URL:                  |  https://host_name:port/api/user/refresh_token                    |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_refresh_token |
| Response HTTP Status Code:    |  200 OK                                                           |
| Response Body:                |  {"token":"new_authentication_token", "refreshToken":"new_refresh_token"} |
| Response Error Status Code:   |  401 Unauthorized <br> 500 Internal Server Error                  |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVkYWVmYjQ3LTI4YzYtNDA1OC1hMDBiLTc1ZGI1MTQ0OGJkNyIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMTIyNDMzMiwiZXhwIjoxNTMxODI5MTMyfQ.PIL-thIdKr2ji5LDOelLXPfIZQUvo3II1KrOG0-lLx4" https://localhost:8000/api/user/refresh_token
```
Response example:  
```json
{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVkYWVmYjQ3LTI4YzYtNDA1OC1hMDBiLTc1ZGI1MTQ0OGJkNyIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMTIyNDM4NSwiZXhwIjoxNTMxMjI1Mjg1fQ.U_A74GyZy9xGZsj49weYaPK99iZeBaIFkCX9GVx9_dE","refreshToken":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVkYWVmYjQ3LTI4YzYtNDA1OC1hMDBiLTc1ZGI1MTQ0OGJkNyIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMTIyNDM4NSwiZXhwIjoxNTMxODI5MTg1fQ.6OoLLVmTAsHMtDsLWPl9lS19w1WMQeyNyL9WLReEjaQ"}
```

#### Invalidate Refresh Token
Both `admin user` and `normal user` can call this API. This API is [OAuth 2](https://oauth.net/2/) compatiable. You will invalidate all previously obtained `refreshToken`. Note that you need supply `refreshToken` instead of `token` to `Authorization Bearer` header.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  POST                                                             |
| Request URL:                  |  https://host_name:port/api/user/invalidate_refresh_token         |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_refresh_token |
| Request Body:                 |  {}                                                               |
| Response HTTP Status Code:    |  201 Created                                                      |
| Response Body:                |  {}                                                               |
| Response Error Status Code:   |  401 Unauthorized <br> 500 Internal Server Error                  |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVkYWVmYjQ3LTI4YzYtNDA1OC1hMDBiLTc1ZGI1MTQ0OGJkNyIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMTIyNDMzMiwiZXhwIjoxNTMxODI5MTMyfQ.PIL-thIdKr2ji5LDOelLXPfIZQUvo3II1KrOG0-lLx4" -X POST -d '{"username":"admin@email.com","password":"pleaseChangePassword"}' https://localhost:8000/api/user/invalidate_refresh_token
```
Response example:  
```json
{}
```

#### Create User
Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  POST                                                             |
| Request URL:                  |  https://host_name:port/api/user                                  |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Request Body:                 |  {"role":"user", "email":"email_address", "password":"userPassword"} |
| Response HTTP Status Code:    |  201 Created                                                      |
| Response Body:                |  {"id","user_id"}                                                 |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 409 Conflict (user already exists) <br> 500 Internal Server Error                  |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" -X POST -d '{"role":"user","email":"user@email.com","password":"somePassword"}' https://localhost:8000/api/user
```
Response example:  
```json
{"id":"7768be69-b707-4111-a15c-84b7278bc588"}
```

#### Delete User
Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  DELETE                                                           |
| Request URL:                  |  https://host_name:port/api/user?id=user_id                      |
| Request Header:               |  Authorization: Bearer your_authentication_token                  |
| Response HTTP Status Code:    |  204 No Content                                                   |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" -X DELETE https://localhost:8000/api/user?id=7768be69-b707-4111-a15c-84b7278bc588
```

#### Get User
Both `admin user` and `normal user` can call this API. But `normal user` can get user about themselves. The `admin user` can get user about anyone.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  GET                                                              |
| Request URL:                  |  https://host_name:port/api/user?id=user_id                      |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Response HTTP Status Code:    |  200 OK                                                           |
| Response Body:                |  {"id":"user_id", "type":"EmailUser", "role":"admin", "email":"email_address", "createdTime":epoch_time, "username":"email_address", "lastLoginTime":epoch_time} |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 404 Not Found (user does not exist) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" https://localhost:8000/api/user?id=e2da99e0-7d47-4ce2-b8c8-3088e0132459
```
Response example:  
```json
{"id":"e2da99e0-7d47-4ce2-b8c8-3088e0132459","type":"EmailUser","role":"admin","email":"admin@email.com","createdTime":1530753369303,"username":"admin@email.com","lastLoginTime":1530759474686}
```

#### Update User
Both `admin user` and `normal user` can call this API. But `normal user` can get user about themselves. The `admin user` can get user about anyone.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  PUT                                                              |
| Request URL:                  |  https://host_name:port/api/user                                  |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Request Body:                 |  {"id":"user_id", "role":"user_role", "email":"new_email_address", "password":"new_password"} |
| Response HTTP Status Code:    |  200 OK                                                           |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 404 Not Found (user does not exist) <br> 409 Conflict (new email address already exists) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" -X PUT -d '{"id":"e2da99e0-7d47-4ce2-b8c8-3088e0132459", "role":"admin","email":"admin@new.email.com","password":"newPassword"}' https://localhost:8000/api/user
```


### 2. Server APIs

Servers are physical shadowsocks servers. Multiple servers are supported. Each server must have an external ip address or domain name that can be used to be connected by user shadowsocks clients.

#### Create Server

Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  POST                                                             |
| Request URL:                  |  https://host_name:port/api/server                                |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Request Body:                 |  {"ipAddressOrDomainName":"server_ip_address_or_domain_name"}     |
| Response HTTP Status Code:    |  201 Created                                                      |
| Response Body:                |  {"id","server_id"}                                               |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 409 Conflict (server already exists) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" -X POST -d '{"ipAddressOrDomainName":"127.0.0.1"}' https://localhost:8000/api/server
```
Response example:  
```json
{"id":"d0ee089d-f43d-4bae-b408-02f59db04e9c"}
```

#### Delete Server

Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  DELETE                                                           |
| Request URL:                  |  https://host_name:port/api/server?id=server_id                  |
| Request Header:               |  Authorization: Bearer your_authentication_token                  |
| Response HTTP Status Code:    |  204 No Content                                                   |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 409 Conflict (server is in use) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" -X DELETE https://localhost:8000/api/server?id=d0ee089d-f43d-4bae-b408-02f59db04e9c
```

#### Get Server

Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  GET                                                              |
| Request URL:                  |  https://host_name:port/api/server?id=server_id                  |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Response HTTP Status Code:    |  200 OK                                                           |
| Response Body:                |  {"id":"server_id", "ipAddressOrDomainName":"server_ip_address_or_domain_name", "createdTime":epoch_time} |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 404 Not Found (server does not exist) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" https://localhost:8000/api/server?id=c9be32dd-e2d6-4af4-876f-f086e82af20c
```
Response example:  
```json
{"id":"c9be32dd-e2d6-4af4-876f-f086e82af20c","ipAddressOrDomainName":"127.0.0.1","createdTime":1530773538890}
```

#### Update Server

Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  PUT                                                              |
| Request URL:                  |  https://host_name:port/api/server                                |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Request Body:                 |  {"id":"server_id", "ipAddressOrDomainName":"new_server_ip_address_or_domain_name"} |
| Response HTTP Status Code:    |  200 OK                                                           |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 404 Not Found (server does not exist) <br> 409 Conflict (new ip address or domain name already exists) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" -X PUT -d '{"id":"c9be32dd-e2d6-4af4-876f-f086e82af20c","ipAddressOrDomainName":"127.0.0.2"}' https://localhost:8000/api/server
```

### 3. Node APIs

Nodes are logical shadowsocks servers. A node has to reside in one and only one server. A server may have multiple nodes. All nodes within a server appear like different shadowsocks servers to users, but they share the same resources of the server. A node must have a unique name, a port number and password that is used by [shadowsocks-restful-api](https://github.com/shadowsocks/shadowsocks-restful-api).

#### Create Node

Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  POST                                                             |
| Request URL:                  |  https://host_name:port/api/node                                  |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Request Body:                 |  {"serverId":"server_id", "port":port_number, "password":"somePassword", "name":"someName"} |
| Response HTTP Status Code:    |  201 Created                                                      |
| Response Body:                |  {"id","node_id"}                                                 |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 404 Not Found (server does not exist) <br> 409 Conflict (node already exists) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" -X POST -d '{"serverId":"c9be32dd-e2d6-4af4-876f-f086e82af20c", "port":4001, "password":"pleaseChangeThisPassword", "name":"New York"}' https://localhost:8000/api/node
```
Response example:  
```json
{"id":"5daefb47-28c6-4058-a00b-75db51448bd7"}
```

#### Delete Node

Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  DELETE                                                           |
| Request URL:                  |  https://host_name:port/api/node?id=node_id                      |
| Request Header:               |  Authorization: Bearer your_authentication_token                  |
| Response HTTP Status Code:    |  204 No Content                                                   |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 409 Conflict (node is in use) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" -X DELETE https://localhost:8000/api/node?id=5daefb47-28c6-4058-a00b-75db51448bd7
```

#### Get Node

Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  GET                                                              |
| Request URL:                  |  https://host_name:port/api/node?id=node_id                      |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Response HTTP Status Code:    |  200 OK                                                           |
| Response Body:                |  {"id":"node_id", "server":{"id":"server_id", "ipAddressOrDomainName":"server_ip_address_or_domain_name", "createdTime":server_created_time}, "password":"some_password", "port":node_management_port_number, "name":"node_name", "comment":"come_comment", "createdTime":node_createdTime} |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 404 Not Found (node does not exist) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" https://localhost:8000/api/node?id=075a74fd-d8ab-4313-bde3-f9cb99d7215d
```
Response example:  
```json
{"id":"075a74fd-d8ab-4313-bde3-f9cb99d7215d","server":{"id":"7b15f7ba-ee6c-452d-8264-0657f202590d","ipAddressOrDomainName":"127.0.0.1","createdTime":1530790525902},"password":"pleaseChangeThisPassword","port":4001,"name":"New York","comment":null,"createdTime":1530790577846}
```

#### Update Node

Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  PUT                                                              |
| Request URL:                  |  https://host_name:port/api/node                                  |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Request Body:                 |  {"id":"node_id", "port":new_port_number, "password":"newPassword", "name":"newName"} |
| Response HTTP Status Code:    |  200 OK                                                           |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 404 Not Found (node does not exist) <br> 409 Conflict (new name already exists) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" -X PUT -d '{"id":"075a74fd-d8ab-4313-bde3-f9cb99d7215d", "port":4002, "password":"newPassword", "name":"newName", "comment":"some comment"}' https://localhost:8000/api/node
```

### 4. Product APIs

Products are different types of shadowsocks services. A product must have a unique name, traffic, and period. The period of a product defines the life span of an account associated with this product. The traffic of a product defines the maximum amount of traffic allowed for an account associated with this product with in its period.

#### Create Product

Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  POST                                                             |
| Request URL:                  |  https://host_name:port/api/product                               |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Request Body:                 |  {"name":"some name", "traffic":max_traffic_allowed, "period":"valid_period"} |
| Response HTTP Status Code:    |  201 Created                                                      |
| Response Body:                |  {"id","product_id"}                                              |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 409 Conflict (product already exists) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" -X POST -d '{"name":"dimond", "traffic":1000000000, "period":"monthly"}' https://localhost:8000/api/product
```
Response example:  
```json
{"id":"66c5a883-f729-498b-9442-f3d0bbca9345"}
```

#### Delete Product

Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  DELETE                                                           |
| Request URL:                  |  https://host_name:port/api/product?id=product_id                |
| Request Header:               |  Authorization: Bearer your_authentication_token                  |
| Response HTTP Status Code:    |  204 No Content                                                   |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 409 Conflict (product is in use) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" -X DELETE https://localhost:8000/api/product?id=66c5a883-f729-498b-9442-f3d0bbca9345
```

#### Get Product

Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  GET                                                              |
| Request URL:                  |  https://host_name:port/api/product?id=product_id                |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Response HTTP Status Code:    |  200 OK                                                           |
| Response Body:                |  {"id":"server_id", "ipAddressOrDomainName":"server_ip_address_or_domain_name", "createdTime":epoch_time} |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 404 Not Found (product does not exist) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" https://localhost:8000/api/product?id=b35aa689-756c-4752-bb7a-9b3e8e0dd699
```
Response example:  
```json
{"id":"b35aa689-756c-4752-bb7a-9b3e8e0dd699","name":"dimond","traffic":1000000000,"period":"monthly","createdTime":1530793170070}
```

#### Update Product

Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  PUT                                                              |
| Request URL:                  |  https://host_name:port/api/node                                  |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Request Body:                 |  {"id":"product_id", "name":"newName", "traffic":newTraffic, "period":"newPeriod"} |
| Response HTTP Status Code:    |  200 OK                                                           |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 404 Not Found (product does not exist) <br> 409 Conflict (new name already exists) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" -X PUT -d '{"id":"b35aa689-756c-4752-bb7a-9b3e8e0dd699", "name":"newName", "traffic":5000000000, "period":"annual"}' https://localhost:8000/api/product
```

### 5. Account APIs

Accounts contain all information needed for a shadowsocks client to connect to a node. A user has to obtain an account before being able to use a product. An account contains data of its owner, the node and product associated with the account.

#### Request Account

Both `admin user` and `normal user` can call this API. But `normal user` can request account for themselves only. The `admin user` can do so for anyone.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  POST                                                             |
| Request URL:                  |  https://host_name:port/api/account/request                       |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Request Body:                 |  {"userId":"user_id", "productId":"product_id"}                   |
| Response HTTP Status Code:    |  201 Created                                                      |
| Response Body:                |  {"id","approval_id"}                                             |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 404 Not Found (user does not exist) <br> 420 (product does not exist) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" -X POST -d '{"userId":"5daefb47-28c6-4058-a00b-75db51448bd7", "productId":"b35aa689-756c-4752-bb7a-9b3e8e0dd699"}' https://localhost:8000/api/account/request
```
Response example:  
```json
{"id":"480e723d-6e61-4440-a6dd-6ae624342a8e"}
```


#### Approve Request (create accounts)

Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  POST                                                             |
| Request URL:                  |  https://host_name:port/api/account/approve                       |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Request Body:                 |  {"id","approval_id"}                                             |
| Response HTTP Status Code:    |  201 Created                                                      |
| Response Body:                |  {"id","approval_id"}                                             |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 404 Not Found (approval_id does not exist) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" -X POST -d '{"id":"0e3f59ac-693c-4fbf-a286-6a9de41d3a74"}' https://localhost:8000/api/account/approve
```
Response example:  
```json
{"id":"0e3f59ac-693c-4fbf-a286-6a9de41d3a74"}
```

#### Delete Account

Only `admin user` can call this API.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  DELETE                                                           |
| Request URL:                  |  https://host_name:port/api/account?id=account_id                |
| Request Header:               |  Authorization: Bearer your_authentication_token                  |
| Response HTTP Status Code:    |  204 No Content                                                   |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 409 Conflict (account is in use) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" -X DELETE https://localhost:8000/api/account?id=66c5a883-f729-498b-9442-f3d0bbca9345
```

#### Get Account

Both `admin user` and `normal user` can call this API. But `normal user` can get account belonging to themselves only. The `admin user` can get any account.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  GET                                                              |
| Request URL:                  |  https://host_name:port/api/account?id=account_id                |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Response HTTP Status Code:    |  200 OK                                                           |
| Response Body:                |  {"id":"account_id", "node":{"id":"node_id", "server":{"id":"server_id", "ipAddressOrDomainName":"server_ip_address_or_domain_name", "createdTime":server_created_time}, "port":node_management_port_number, "name":"node_name", "comment": "noe_comment", "createdTime":node_created_time},"port":account_port_number, "createdTime":account__created_time, "password":"account_password", "method":"encryption_method", "approval":{"id":"approval_id", "state":"approval_state", "createdTime":approval_created_time}, "request":{"id":"request_id", "user":{"id":"user_id", "role":"user_role", "email":"user_email_address", "createdTime":user_created_time, "username":"user_email_address"}, "product":{"id":"product_id", "name":"product_name", "traffic":product_traffic, "period":"product_period", "createdTime":product_created_time}, "createdTime":request_created_time}} |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 404 Not Found (account does not exist) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" https://localhost:8000/api/account?id=b35aa689-756c-4752-bb7a-9b3e8e0dd699
```
Response example:  
```json
{"id":"c2e5a807-9525-4c23-b121-6024e2d1eebb","node":{"id":"cd41dc00-2e53-4d7f-8374-b44e82caaeaa","server":{"id":"aec2a2ad-ecb4-4a12-b90d-4acaeb8ec676","ipAddressOrDomainName":"127.0.0.1","createdTime":1530838310353},"port":4001,"name":"New York","comment":null,"createdTime":1530838367312},"port":50647,"createdTime":1530847875199,"password":"ee60fe5cf97433e9","method":"aes-256-cfb","approval":{"id":"1ab123d0-92d6-4c6d-991e-c82cd03182f2","state":"completed","createdTime":1530842543004},"request":{"id":"4da6972a-2e10-4f20-8cc0-4ad66e886607","user":{"id":"ad5ceb7f-4a6d-4577-8ed6-01e3acb2604c","role":"user","email":"user@email.com","createdTime":1530838144205,"username":"user@email.com"},"product":{"id":"48e6a637-f94e-4d8e-b71c-9793a37e7e2c","name":"dimond","traffic":1000000000,"period":"monthly","createdTime":1530838417112},"createdTime":1530842543116}}
```

#### Get Accounts by User Id

Both `admin user` and `normal user` can call this API. But `normal user` can get accounts for themselves only. The `admin user` can accounts for any users.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  GET                                                              |
| Request URL:                  |  https://host_name:port/api/account/accounts_by_user_id?id=account_id                |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Response HTTP Status Code:    |  200 OK                                                           |
| Response Body:                |  [{"id":"account_id", "node":{"id":"node_id", "server":{"id":"server_id", "ipAddressOrDomainName":"server_ip_address_or_domain_name", "createdTime":server_created_time}, "port":node_management_port_number, "name":"node_name", "comment": "noe_comment", "createdTime":node_created_time},"port":account_port_number, "createdTime":account__created_time, "password":"account_password", "method":"encryption_method", "approval":{"id":"approval_id", "state":"approval_state", "createdTime":approval_created_time}, "request":{"id":"request_id", "user":{"id":"user_id", "role":"user_role", "email":"user_email_address", "createdTime":user_created_time, "username":"user_email_address"}, "product":{"id":"product_id", "name":"product_name", "traffic":product_traffic, "period":"product_period", "createdTime":product_created_time}, "createdTime":request_created_time}},...] |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 404 Not Found (account does not exist) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" https://localhost:8000/api/account/accounts_by_user_id?id=3ec1072e-e6e3-438d-99ad-2f891b877697
```
Response example:  
```json
[{"id":"c2e5a807-9525-4c23-b121-6024e2d1eebb","node":{"id":"cd41dc00-2e53-4d7f-8374-b44e82caaeaa","server":{"id":"aec2a2ad-ecb4-4a12-b90d-4acaeb8ec676","ipAddressOrDomainName":"127.0.0.1","createdTime":1530838310353},"port":4001,"name":"New York","comment":null,"createdTime":1530838367312},"port":50647,"createdTime":1530847875199,"password":"ee60fe5cf97433e9","method":"aes-256-cfb","approval":{"id":"1ab123d0-92d6-4c6d-991e-c82cd03182f2","state":"completed","createdTime":1530842543004},"request":{"id":"4da6972a-2e10-4f20-8cc0-4ad66e886607","user":{"id":"ad5ceb7f-4a6d-4577-8ed6-01e3acb2604c","role":"user","email":"user@email.com","createdTime":1530838144205,"username":"user@email.com"},"product":{"id":"48e6a637-f94e-4d8e-b71c-9793a37e7e2c","name":"dimond","traffic":1000000000,"period":"monthly","createdTime":1530838417112},"createdTime":1530842543116}}]
```

### 6. Traffic APIs

Traffic is the amount of data transmitted by a node for an account.

#### Get Account Latest Traffic

Both `admin user` and `normal user` can call this API. But `normal user` can get latest traffic belonging to themselves only. The `admin user` can do so for any account.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  GET                                                              |
| Request URL:                  |  https://host_name:port/api/traffic/account?id=uuid?id=account_id                |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Response HTTP Status Code:    |  200 OK                                                           |
| Response Body:                |  {"usage":latest_usage}         |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 404 Not Found (account does not exist) <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A"  https://localhost:8000/api/traffic/account?id=766341be-bc67-49e2-bc44-1c9ac485a56b
```
Response example:  
```json
{"usage":298270232}
```

#### Get Account Traffic History

Both `admin user` and `normal user` can call this API. But `normal user` can get traffic history belonging to themselves only. The `admin user` can do so for any account.

|                               |                                                                   |
| :---------------------------- | :---------------------------------------------------------------- |
| Request method:               |  GET                                                              |
| Request URL:                  |  https://host_name:port/api/traffic/history?id=account_id    |
| Request Header:               |  Content-Type: application/json <br> Authorization: Bearer your_authentication_token |
| Response HTTP Status Code:    |  200 OK                                                           |
| Response Body:                |  [{"usage":accumulative_usage, "createdTime":epoch_time}...] |
| Response Error Status Code:   |  400 Bad Request <br> 401 Unauthorized <br> 500 Internal Server Error |

Request example (curl):  
```
curl -ik -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImUyZGE5OWUwLTdkNDctNGNlMi1iOGM4LTMwODhlMDEzMjQ1OSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTUzMDc1OTQ3NCwiZXhwIjoxNTMwODQ1ODc0fQ.XqS8UBj7hWNeKjaGlXjrZDHuVZWM_8thw__ojAkBG0A" https://localhost:8000/api/traffic/history?id=b12bd8d0-5943-48c4-9016-d45a654c5a1c
```
Response example:  
```json
[{"usage":0,"createdTime":1530840202934},{"usage":19380263,"createdTime":1530840300014}]
```
