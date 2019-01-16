# docker compose of kong and kong dashboard

- postgresql volume mapping 

- logging:fluentd+elasticsearch+kibana

#### reference

1.logging

https://docs.fluentd.org/v0.12/articles/docker-logging-efk-compose

2.kong

https://github.com/Kong/docker-kong/blob/master/compose/docker-compose.yml

3.kong dashboard

https://gist.github.com/oogali/0a3555b0f766dcecc104717203130f6e

4.kong oauth2(notice parameter:grant_type)

https://github.com/Kong/kong-oauth2-hello-world

grant_type details

https://docs.konghq.com/hub/kong-inc/oauth2/

##### grant_type as follows(four type):
config.enable_authorization_code

details:https://tools.ietf.org/html/rfc6749#section-4.1

config.enable_client_credentials

details:https://tools.ietf.org/html/rfc6749#section-4.4

config.enable_implicit_grant

details:https://tools.ietf.org/html/rfc6749#section-4.2

config.enable_password_grant(**the easiest type,personal recommendation--test first**)

datails:https://tools.ietf.org/html/rfc6749#section-4.3 

### Kong Plugin ACL Usage Example:
Spend several hours. So record the process.
Reference:
https://docs.konghq.com/hub/kong-inc/acl/
https://github.com/Kong/kong-oauth2-hello-world
 - Step 1. create a service named as "mockacl".
```shell
curl -X POST \
  --url "http://127.0.0.1:8001/services" \
  --data "name=mockacl" \
  --data "url=http://mockbin.org/request"
```
Response:
```json
    {
          "host": "mockbin.org",
          "created_at": 1547608887,
          "connect_timeout": 60000,
          "id": "24ad5ced-e5e8-4945-810f-f744fc7354bc",
          "protocol": "http",
          "name": "mockacl",
          "read_timeout": 60000,
          "port": 80,
          "path": "/request",
          "updated_at": 1547608887,
          "retries": 5,
          "write_timeout": 60000
      }
```
 - Step 2. create a route for above service named as  "mockacl"
```shell
curl -X POST \
  --url "http://127.0.0.1:8001/services/mockacl/routes" \
  --data 'hosts[]=mockbin.org' \
  --data 'paths[]=/mockacl'
```
Response:
```json
{
  "created_at": 1547609697,
  "strip_path": true,
  "hosts": [
    "mockbin.org"
  ],
  "preserve_host": false,
  "regex_priority": 0,
  "updated_at": 1547609697,
  "paths": [
    "/mockacl"
  ],
  "service": {
    "id": "24ad5ced-e5e8-4945-810f-f744fc7354bc"
  },
  "methods": null,
  "protocols": [
    "http",
    "https"
  ],
  "id": "a3cb5007-f351-481e-ac9d-02fbf3e22d0b"
}
```
Now,try the new service/api. Work fine:
```shell
curl -X GET \
  --url "http://127.0.0.1:8000/mockacl" \
  --header "Host: mockbin.org"
```
Response:
```json
{
  "startedDateTime": "2019-01-16T03:15:58.678Z",
  "clientIPAddress": "172.20.0.1",
  "method": "GET",
  "url": "http://mockbin.org/request",
  "httpVersion": "HTTP/1.1",
  "cookies": {},
  "headers": {
    "host": "mockbin.org",
    "connection": "close",
    "x-forwarded-for": "172.20.0.1, 10.1.193.136, 54.209.226.208",
    "x-forwarded-proto": "http",
    "x-forwarded-host": "mockbin.org",
    "x-forwarded-port": "80",
    "x-real-ip": "118.122.119.70",
    "kong-cloud-request-id": "c56edc7f1eb558d21f2047ef0c56b1b2",
    "kong-client-id": "mockbineast",
    "user-agent": "curl/7.29.0",
    "accept": "*/*",
    "x-request-id": "001f06cb-c332-4b4b-bb6e-53b68fc920c3",
    "via": "1.1 vegur",
    "connect-time": "0",
    "x-request-start": "1547608558674",
    "total-route-time": "0"
  },
  "queryString": {},
  "postData": {
    "mimeType": "application/octet-stream",
    "text": "",
    "params": []
  },
  "headersSize": 505,
  "bodySize": 0
}
```
 - Step 3. create ACL plugin for above service
```shell
curl -X POST http://127.0.0.1:8001/services/mockacl/plugins \
     --data "name=acl"  \
     --data "config.whitelist=aclgroup" \
     --data "config.hide_groups_header=true"
```
Now,try the new service/api. You will not be allowed to consume this service:
```shell
curl -X GET \
  --url "http://127.0.0.1:8000/mockacl" \
  --header "Host: mockbin.org"
```
The response as follows:
```json
{"message":"You cannot consume this service"}
```
ACL plugin become effective.
 - Step 4. create a Kong consumer (called "mockacluser")  and add group to above consumer
```shell
curl -X POST \
  --url "http://127.0.0.1:8001/consumers/" \
  --data "username=mockacluser"
```
Response :
```json
{
  "custom_id": null,
  "created_at": 1547621622,
  "username": "mockacluser",
  "id": "cf88cb4f-fb5b-4158-9915-eb5df988b1fd"
}
```
Add ACL group(called "aclugroup") to above consumer, the group name must be the same as "config.whitelist" of Step 3
```shell
curl -X POST \
  --url "http://127.0.0.1:8001/consumers/mockacluser/acls" \
  --data "group=aclgroup"
```
Response :
```json
{
  "group": "aclgroup",
  "created_at": 1547624003000,
  "id": "8913b7ef-54a9-422a-b2be-d04274dd5c9b",
  "consumer_id": "cf88cb4f-fb5b-4158-9915-eb5df988b1fd"
}
```
 - Step 5. follow the official description. We add basic Authorization (username is "mockacluser", password is "mockaclpw" ) to above consumer.
```shell
curl -X POST \
     --url http://127.0.0.1:8001/consumers/mockacluser/basic-auth \
     --data "username=mockacluser" \
     --data "password=mockaclpw"
```
Response :
```json
{
  "created_at": 1547624451000,
  "id": "84516d9a-a0fa-4f57-af1b-aca5319bfb37",
  "username": "mockacluser",
  "password": "9b91b3b39365d00cf0a1f283647aebbd5472013f",
  "consumer_id": "cf88cb4f-fb5b-4158-9915-eb5df988b1fd"
}
```
Follow the official description, we get the Authorization header is "bW9ja2FjbHVzZXI6bW9ja2FjbHB3". Then, we try the service as follows:
```shell
curl -X GET \
     --url "http://127.0.0.1:8000/mockacl" \
     --header "Host: mockbin.org" \
     --header "Authorization: Basic bW9ja2FjbHVzZXI6bW9ja2FjbHB3"
```
we get the response
```json
{"message":"You cannot consume this service"}
```
Why?  There are some problems which i spend a lot of time to solve. You must config authentication plugin to the service named as "mockacl" .Otherwise, you will never be allowed to consume this service. So we try to do it.
```shell
curl -X POST http://127.0.0.1:8001/services/mockacl/plugins \
    --data "name=basic-auth"  \
    --data "config.hide_credentials=true"  
```
Response:
```json
{
  "created_at": 1547625339000,
  "config": {
    "hide_credentials": true,
    "anonymous": ""
  },
  "id": "f56e2e6e-0ec9-4faf-801d-b402393f54fc",
  "enabled": true,
  "service_id": "24ad5ced-e5e8-4945-810f-f744fc7354bc",
  "name": "basic-auth"
}
```
Now, try the service/api
```shell
curl -X GET \
     --url "http://127.0.0.1:8000/mockacl" \
     --header "Host: mockbin.org" \
     --header "Authorization: Basic bW9ja2FjbHVzZXI6bW9ja2FjbHB3"
```
We get the correct result the same as the Step 2.The all about ACL plugin control.

