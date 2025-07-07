# Udemy Course Microservices with Spring Boot, Docker, Kubernetes Section_11
https://www.udemy.com/course/master-microservices-with-spring-docker-kubernetes/
## spring version: 3.4.5
## gatewayservice version: 3.5.0
## Java 21


## Documentation
After adding the library, the swagger page is accessible through address 
http://localhost:8080/swagger-ui/index.html,
http://localhost:8090/swagger-ui/index.html,
http://localhost:9000/swagger-ui/index.html.

## links

### Eureka Server dashboard
http://localhost:8070/

### Link from Eureka Server to Gateway Server
http://172.24.64.1:8072/actuator/info
{
    "app": {
        "name": "gatewayserver",
        "description": "Eazy Bank Gateway Server Application",
        "version": "1.0.0"
    }
}

### Further links
http://localhost:8072/actuator
http://localhost:8072/actuator/gateway
http://localhost:8072/actuator/gateway/routes

### Example invoking an service:
POST http://localhost:8072/eazybank/accounts/api/create


### actuator links:
http://localhost:8072/actuator
http://localhost:8071/actuator
http://localhost:8070/actuator
http://localhost:8080/actuator
http://localhost:8090/actuator
http://localhost:9000/actuator


## Securing Gateway Server as a Resource server

### run Redis server
docker run -p 6379:6379 --name eazyredis -d redis
### start KeyCloak Docker container
docker run -d -p 127.0.0.1:7080:8080 -e KC_BOOTSTRAP_ADMIN_USERNAME=admin -e KC_BOOTSTRAP_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:26.3.0 start-dev

### start all microservices manually

### testing requests with no security
GET http://localhost:8072/eazybank/accounts/api/contact-info: OK
GET http://localhost:8072/eazybank/cards/api/java-version: OK
GET http://localhost:8072/eazybank/loans/api/build-info: OK


### getting the token from keycloak using curl
curl -X POST http://localhost:7080/realms/master/protocol/openid-connect/token \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=client_credentials" \
-d "client_id=eazybank-callcenter-cc" \
-d "client_secret=7tPUlOgJvGHO8Z9yBpowxkNAFpzZ10Dm" \
-d "scope=openid email profile" \ | jq -r .access_token

I was not able to invoke a crating POST request using the token:
POST http://localhost:8072/eazybank/accounts/api/create
I think, there are some issued with characters encoding.
Possible solution for this would be using a *.sh script.
See script/create_account_keycloak.sh

### Not authorized GET request
curl -X GET http://172.21.16.1:8072/eazybank/accounts/api/fetchCustomerDetails?mobileNumber=4354437688 \
-H "Accept": "*/*" | jq


### getting the token from keycloak using PowerShell Invoke-RestMethod
$response = Invoke-RestMethod -Method Post `
-Uri "http://localhost:7080/realms/master/protocol/openid-connect/token" `
-ContentType "application/x-www-form-urlencoded" `
-Body @{
grant_type    = "client_credentials"
client_id     = "eazybank-callcenter-cc"
client_secret = "7tPUlOgJvGHO8Z9yBpowxkNAFpzZ10Dm"
}

$accessToken = $response.access_token


access_token       : eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJZdGtrUDQ4bHlVTXVDWE5LYTN4Z0hlYU1VSHMxTXNQekpQRGR4dHNUZVVvIn0.eyJleHAiOjE3NTE4OTE0MTAsImlhdCI6MTc1MTg5MTM1MCwianRpIjoidHJydGNjOjc5YjQ3M2UzLWE4M2UtZDhiMS05ZmM2LWMyNjFhYjM4MGIyYSIsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6NzA4MC9yZWFsbXMvbWFzdGVyIiwiYXVkIjoiYWNjb3VudCIsInN1YiI6IjEwZjM0Y2YwLWIyN2MtNDU1Mi04ZWY2LTJlZWVh
OTFiMDVmNSIsInR5cCI6IkJlYXJlciIsImF6cCI6ImVhenliYW5rLWNhbGxjZW50ZXItY2MiLCJhY3IiOiIxIiwiYWxsb3dlZC1vcmlnaW5zIjpbIi8qIl0sInJlYWxtX2FjY2VzcyI6eyJyb2xlcyI6WyJkZWZhdWx0LXJvbGVzLW1hc3RlciIsIm9mZmxpbmVfYWNjZXNzIiwidW1hX2F1dGhvcml6YXRpb24iXX0sInJlc291cmNlX2FjY2VzcyI6eyJhY2NvdW50Ijp7InJvbGVzIjpbIm1hbmFnZS1hY2NvdW50IiwibWFuYWdlLWFjY291bnQtbGlua3MiLCJ2aWV3LXByb2Zp
bGUiXX19LCJzY29wZSI6InByb2ZpbGUgZW1haWwiLCJlbWFpbF92ZXJpZmllZCI6ZmFsc2UsImNsaWVudEhvc3QiOiIxNzIuMTcuMC4xIiwicHJlZmVycmVkX3VzZXJuYW1lIjoic2VydmljZS1hY2NvdW50LWVhenliYW5rLWNhbGxjZW50ZXItY2MiLCJjbGllbnRBZGRyZXNzIjoiMTcyLjE3LjAuMSIsImNsaWVudF9pZCI6ImVhenliYW5rLWNhbGxjZW50ZXItY2MifQ.Jgp4ecHubRwHXAgudZF66ipvZA1FkNSaoABZbPntXZ3DCL4hcXHh1xTWhvBsh7-SsvWOTZJ_SUoUL
kKCGQ67t33jmkpz1ZyS1kp2dMzbhM8qGbvDG6M-Zc0UXEiBwqBv4RrcS_toiHjFODIAuvkY1yZuyk61XvV2hL1M2pRJOhilUmEzALcdStzs6QINB0pHzNa4zrmefP9Gap320rG5KViZTSMyiZ46DFhFzp6RP8FVrIUCPm43g1AtTiNb5mEJnOeYAEybJfhpxkSaAj6gGdTQQVwyaZ5aANxg3JjK2VtayduwU35CpCd13hW3n3tv5WX3snRTGv5oma67zpE8NQ
expires_in         : 60
refresh_expires_in : 0
token_type         : Bearer
not-before-policy  : 0
scope              : profile email


Invoke-RestMethod -Method Post `
-Uri "http://localhost:8072/eazybank/accounts/api/create" `
-Headers @{ Authorization = "Bearer $accessToken" } `
-ContentType "application/json" `
-Body '{
"name": "Madan Reddy",
"email": "tutor@eazybytes",
"mobileNumber": "4354437689"
}'

statusCode statusMsg
---------- ---------
201        Account created successfully
