
# pfSense REST API Documentation

---

# Introduction
pfSense API is a fast, safe, REST API package for pfSense firewalls. This works by leveraging the same PHP functions and processes used 
by pfSense's webConfigurator into API endpoints to create, read, update and delete pfSense configurations. All API 
endpoints enforce input validation to prevent invalid configurations from being made. Configurations made via API are 
properly written to the master XML configuration and the correct backend configurations are made preventing the need for
 a reboot. All this results in the fastest, safest, and easiest way to automate pfSense!


# Requirements
- pfSense 2.4.4 or later is supported
- pfSense API requires a local user account in pfSense. The same permissions required to make configurations in the 
webConfigurator are required to make calls to the API endpoints
- While not an enforced requirement, it is **strongly** recommended that you configure pfSense to use HTTPS instead of HTTP. This ensures that login credentials and/or API tokens remain secure in-transit


# Installation
To install pfSense API, simply run the following command from the pfSense shell:<br>
```
pkg add https://github.com/jaredhendrickson13/pfsense-api/releases/latest/download/pfSense-2.5-pkg-API.txz && /etc/rc.restart_webgui
```

To uninstall pfSense API, run the following command:<br>
```
pfsense-api delete
```

To update pfSense API to the latest stable version, run the following command:
```
pfsense-api update
```

To revert to a previous version of pfSense API (e.g. v1.1.7), run the following command:
```
pfsense-api revert v1.1.7
```

### Notes: 
- To install the 2.4 package, simply change the `2.5` in the install URL to `2.4`.
- In order for pfSense to apply some required web server changes, it is required to restart the webConfigurator after installing the package
- If you do not have shell access to pfSense, you can still install via the webConfigurator by navigating to 
'Diagnostics > Command Prompt' and enter the commands there
- When updating pfSense, **_you must reinstall pfSense API afterwards_**. Unfortunately, pfSense removes all existing packages and only re-installs packages found within pfSense's package repositories. Since pfSense API is not an official package in pfSense's repositories, it does not get reinstalled automatically.
- The `pfsense-api` command line tool was introduced in v1.1.0. Refer to the corresponding documentation for earlier releases.


# UI Settings & Documentation
After installation, you will be able to access the API user interface pages within the pfSense webConfigurator. These will be found under System > API. The settings tab will allow you change various API settings such as enabled API interfaces, authentication modes, and more. Additionally, the documentation tab will give you access to an embedded documentation tool that makes it easy to view the full API documentation in context to your pfSense instance.

### Notes: 
- Users must hold the `page-all` or `page-system-api` privileges to access the API page within the webConfigurator


# Authentication & Authorization
By default, pfSense API uses the same credentials as the webConfigurator. This behavior allows you to configure pfSense 
from the API out of the box, and user passwords may be changed from the API to immediately add additional security if 
needed. After installation, you can navigate to System > API in the pfSense webConfigurator to configure API
authentication. Please note that external authentication servers like LDAP or RADIUS are not supported with any 
API authentication method at this time. 

To authenticate your API call, follow the instructions for your configured authentication mode:

<details>
    <summary>Local Database (default)</summary>

Uses the same credentials as the pfSense webConfigurator. To authenticate API calls, simply add a `client-id` value containing your username and a `client-token` value containing your password to your payload. For example `{"client-id": "admin", "client-token": "pfsense"}`

</details>

<details>
    <summary>JWT</summary>

Requires a bearer token to be included in the `Authorization` header of your request. To receive a bearer token, you may make a POST request to /api/v1/access_token/ and include a `client-id` value containing your pfSense username and a `client-token` value containing your pfSense password to your payload. For example `{"client-id": "admin", "client-token": "pfsense"}`. Once you have your bearer token, you can authenticate your API call by adding it to the request's authorization header. (e.g. `Authorization: Bearer xxxxxxxx.xxxxxxxxx.xxxxxxxx`)
</details>

<details>
    <summary>API Token</summary>

Uses standalone tokens generated via the UI. These are better suited to distribute to systems as they are revocable and will only allow API authentication; not UI or SSH authentication (like the local database credentials). To generate or revoke credentials, navigate to System > API within the UI and ensure the Authentication Mode is set to API token. Then you should have the options to configure API Token generation, generate new tokens, and revoke existing tokens. Once you have your API token, you may authenticate your API call by specifying your client-id and client-token within an `Authorization` header, these values must be separated by a space. (e.g. `Authorization: client-id-here client-token-here`)

_Note: In previous versions of pfSense API, the client-id and client-token were provided via the request payload. This functionality is still supported but is not recommended. It will be removed in a future release._

</details>

### Authorization
pfSense API uses the same privileges as the pfSense webConfigurator. The required privileges for each endpoint are stated within the API documentation.


# Content Types
pfSense API can handle a few different content types. Please note, if a `Content-Type` header is not specified in your request, pfSense API will attempt to determine the
content type which may have undesired results. It is recommended you specify your preferred `Content-Type` on each request. While several content types may be enabled,
`application/json` is the recommended content type. Supported content types are:

<details>
    <summary>application/json</summary>

Parses the request body as a JSON formatted string. This is the recommended content type.

Example:

```
curl -s -H "Content-Type: application/json" -d '{"client-id": "admin", "client-token": "pfsense"}' -X GET https://pfsense.example.com/api/v1/system/arp
{
  "status": "ok",
  "code": 200,
  "return": 0,
  "message": "Success",
  "data": [
    {
      "ip": "192.168.1.1",
      "mac": "00:0c:29:f6:be:d9",
      "interface": "em1",
      "status": "permanent",
      "linktype": "ethernet"
    },
    {
      "ip": "172.16.209.139",
      "mac": "00:0c:29:f6:be:cf",
      "interface": "em0",
      "status": "permanent",
      "linktype": "ethernet"
    }
  ]
}
```

</details>

<details>
    <summary>application/x-www-form-urlencoded</summary>

Parses the request body as URL encoded parameters. Note: boolean and integer types may not be parsed using this content type.

Example:

```
curl -s -H "Content-Type: application/x-www-form-urlencoded" -X GET "https://pfsense.example.com/api/v1/system/arp?client-id=admin&client-token=pfsense"
{
  "status": "ok",
  "code": 200,
  "return": 0,
  "message": "Success",
  "data": [
    {
      "ip": "192.168.1.1",
      "mac": "00:0c:29:f6:be:d9",
      "interface": "em1",
      "status": "permanent",
      "linktype": "ethernet"
    },
    {
      "ip": "172.16.209.139",
      "mac": "00:0c:29:f6:be:cf",
      "interface": "em0",
      "status": "permanent",
      "linktype": "ethernet"
    }
  ]
}
```

</details>

# Response Codes
`200 (OK)` : API call succeeded<br>
`400 (Bad Request)` : An error was found within your requested parameters<br>
`401 (Unauthorized)` : API client has not completed authentication or authorization successfully<br>
`403 (Forbidden)` : The API endpoint has refused your call. Commonly due to your access settings found in System > API<br>
`404 (Not found)` : Either the API endpoint or requested data was not found<br>
`500 (Server error)` : The API endpoint encountered an unexpected error processing your API request<br>


# Error Codes
A full list of error codes can be found by navigating to /api/v1/system/api/error after installation. This will return
 JSON data containing each error code and their corresponding error message. No authentication is required to view the 
 error code library. This also makes API integration with third-party software easy as the API error codes and messages 
 are always just an HTTP call away!


# Queries
pfSense API contains an advanced query engine to make it easy to query specific data from API calls. For endpoints supporting `GET` requests, you may query the return data to only return data you are looking for. To query data, you may add the data you are looking for to your payload. You may specify as many query parameters as you need. In order to match the query, each parameter must match exactly, or utilize a query filter to set criteria. If no matches were found, the endpoint will return an empty array in the data field. 
<details>
    <summary>Targeting Objects</summary>
    
You may find yourself only needing to read objects with specific values set. For example, say an API endpoint normally returns this response without a query:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 0, "name": "Test", "type": "type1", "extra": {"tag": 0}}, 
        {"id": 1, "name": "Other Test", "type": "type2", "extra": {"tag": 100}},
        {"id": 2, "name": "Another Test", "type": "type1", "extra": {"tag": 200}}

    ]
}
```

If you want the endpoint to only return the objects that have their `type` value set to `type1` you could add `{"type": "type1"}` to your payload. This returns:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 0, "name": "Test", "type": "type1", "extra": {"tag": 0}}, 
        {"id": 2, "name": "Another Test", "type": "type1", "extra": {"tag": 200}}
    ]
}
```

Additionally, if you need to target values that are nested within an array, you can add `{"extra__tag": 100}` to recursively target the `tag` value within the `extra` array. This returns:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 1, "name": "Other Test", "type": "type2", "extra": {"tag": 100}}
    ]
}
```

</details>

<details>
    <summary>Query Filters</summary>
    
Query filters allow you to apply logic to the objects you target. This makes it easy to target data that meets specific criteria:

### Starts With
    
The `startswith` filter allows you to target objects whose values start with a specific substring. This will work on both string and integer data types. Below is an example response without any queries:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 0, "name": "Test", "type": "type1", "extra": {"tag": 0}}, 
        {"id": 1, "name": "Other Test", "type": "type2", "extra": {"tag": 100}},
        {"id": 2, "name": "Another Test", "type": "type1", "extra": {"tag": 200}}

    ]
}
```

If you wanted to target objects whose names started with `Other`, you could use the payload `{"name__startswith": "Other"}`. This returns:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 1, "name": "Other Test", "type": "type2", "extra": {"tag": 100}}
    ]
}
```

### Ends With
    
The `endswith` filter allows you to target objects whose values end with a specific substring. This will work on both string and integer data types. Below is an example response without any queries:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 0, "name": "Test", "type": "type1", "extra": {"tag": 0}}, 
        {"id": 1, "name": "Other Test", "type": "type2", "extra": {"tag": 100}},
        {"id": 2, "name": "Another Test", "type": "type1", "extra": {"tag": 200}}

    ]
}
```

If you wanted to target objects whose names ended with `er Test`, you could use the payload `{"name__endswith" "er Test"}`. This returns:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 1, "name": "Other Test", "type": "type2", "extra": {"tag": 100}},
        {"id": 2, "name": "Another Test", "type": "type1", "extra": {"tag": 200}}    
    ]
}
```

### Contains
    
The `contains` filter allows you to target objects whose values contain a specific substring. This will work on both string and integer data types. Below is an example response without any queries:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 0, "name": "Test", "type": "type1", "extra": {"tag": 0}}, 
        {"id": 1, "name": "Other Test", "type": "type2", "extra": {"tag": 100}},
        {"id": 2, "name": "Another Test", "type": "type1", "extra": {"tag": 200}}

    ]
}
```

If you wanted to target objects whose names contain `ther`, you could use the payload `{"name__contains": "ther"}`. This returns:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 1, "name": "Other Test", "type": "type2", "extra": {"tag": 100}},
        {"id": 2, "name": "Another Test", "type": "type1", "extra": {"tag": 200}}    
    ]
}
```

### Less Than
    
The `lt` filter allows you to target objects whose values are less than a specific number. This will work on both numeric strings and integer data types. Below is an example response without any queries:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 0, "name": "Test", "type": "type1", "extra": {"tag": 0}}, 
        {"id": 1, "name": "Other Test", "type": "type2", "extra": {"tag": 100}},
        {"id": 2, "name": "Another Test", "type": "type1", "extra": {"tag": 200}}

    ]
}
```

If you wanted to target objects whose tag is less than `100`, you could use the payload `{"extra__tag__lt": 100}`. This returns:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 0, "name": "Test", "type": "type1", "extra": {"tag": 0}}  
    ]
}
```

### Less Than or Equal To
    
The `lte` filter allows you to target objects whose values are less than or equal to a specific number. This will work on both numeric strings and integer data types. Below is an example response without any queries:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 0, "name": "Test", "type": "type1", "extra": {"tag": 0}}, 
        {"id": 1, "name": "Other Test", "type": "type2", "extra": {"tag": 100}},
        {"id": 2, "name": "Another Test", "type": "type1", "extra": {"tag": 200}}

    ]
}
```

If you wanted to target objects whose tag is less than or equal to `100`, you could use the payload `{"extra__tag__lte": 100}`. This returns:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 0, "name": "Test", "type": "type1", "extra": {"tag": 0}},
        {"id": 1, "name": "Other Test", "type": "type2", "extra": {"tag": 100}}
    ]
}
```

### Greater Than
    
The `gt` filter allows you to target objects whose values are greater than a specific number. This will work on both numeric strings and integer data types. Below is an example response without any queries:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 0, "name": "Test", "type": "type1", "extra": {"tag": 0}}, 
        {"id": 1, "name": "Other Test", "type": "type2", "extra": {"tag": 100}},
        {"id": 2, "name": "Another Test", "type": "type1", "extra": {"tag": 200}}

    ]
}
```

If you wanted to target objects whose tag is greater than `100`, you could use the payload `{"extra__tag__gt": 100}`. This returns:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 2, "name": "Another Test", "type": "type1", "extra": {"tag": 200}}  
    ]
}
```

### Greater Than or Equal To
    
The `lte` filter allows you to target objects whose values are greater than or equal to a specific number. This will work on both numeric strings and integer data types. Below is an example response without any queries:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 0, "name": "Test", "type": "type1", "extra": {"tag": 0}}, 
        {"id": 1, "name": "Other Test", "type": "type2", "extra": {"tag": 100}},
        {"id": 2, "name": "Another Test", "type": "type1", "extra": {"tag": 200}}

    ]
}
```

If you wanted to target objects whose tag is greater than or equal to `100`, you could use the payload `{"extra__tag__gte": 100}`. This returns:

```json
{
    "status":"ok",
    "code":200,
    "return":0,
    "message":"Success",
    "data": [
        {"id": 1, "name": "Other Test", "type": "type2", "extra": {"tag": 100}},
        {"id": 2, "name": "Another Test", "type": "type1", "extra": {"tag": 200}}
    ]
}
```

</details>

### Obtaining Object IDs
You may notice some API endpoints require an object `id` to update or delete objects. These IDs are not stored values, rather pfSense uses the object's array index value to locate and identify objects. Unless specified otherwise, API endpoints will use the same array index value (as returned in the `data` response field) to locate objects when updating or deleting. Some important items to note about these IDs:

- pfSense starts arrays with an index of 0. If you use a loop counter to determine the ID of a specific object, you must start that counter at 0.
- These IDs are dynamic. For example, if you have 3 static route objects stored (IDs `0`, `1`, and `2`) and you _delete_ the object with ID `1`, pfSense will resort the array so the object with ID `2` will now have an ID of `1`.
- API queries will retain the object's ID in the response. In the event that the `data` response field is no longer a sequential array due to the query, the `data` field will be represented as an associative array with the array items` key being the objects ID.

### Requirements for queries:
- API call must be successful and return `0` in the `return` field.
- Endpoints must return an array of objects in the data field (e.g. `[{"id": 0, "name": "Test"}, {"id": 1, "name": "Other Test"}]`).
- At least two objects must be present within the data field to support queries.

### Notes:
- For those using the Local database or API token authentication types, `client-id` and `client-token` are excluded from queries by default
- Both recursive queries and query filters are separated by double underscores (`__`)


# Rate limit
There is no limit to API calls at this time but is important to note that pfSense's XML configuration was not designed for quick simultaneous writes like a traditional database. It may be necessary to delay API calls in sequence to prevent unexpected behavior. Alternatively, you may limit the API to read-only mode to only allow endpoints with read (GET) access within the webConfigurator's System > API page.


# Endpoints

## Indices

* [ACCESS_TOKEN](#access_token)

  * [Request Access Token](#1-request-access-token)

* [FIREWALL/ALIAS](#firewallalias)

  * [Create Firewall Aliases](#1-create-firewall-aliases)
  * [Delete Firewall Aliases](#2-delete-firewall-aliases)
  * [Read Firewall Aliases](#3-read-firewall-aliases)
  * [Update Firewall Aliases](#4-update-firewall-aliases)

* [FIREWALL/ALIAS/ENTRY](#firewallaliasentry)

  * [Create Firewall Alias Entries](#1-create-firewall-alias-entries)
  * [Delete Firewall Alias Entries](#2-delete-firewall-alias-entries)

* [FIREWALL/APPLY](#firewallapply)

  * [Apply Firewall](#1-apply-firewall)

* [FIREWALL/NAT/ONE_TO_ONE](#firewallnatone_to_one)

  * [Create NAT 1-to-1 Mappings](#1-create-nat-1-to-1-mappings)
  * [Delete NAT 1-to-1 Mappings](#2-delete-nat-1-to-1-mappings)
  * [Read NAT 1-to-1 Mappings](#3-read-nat-1-to-1-mappings)
  * [Update NAT 1-to-1 Mappings](#4-update-nat-1-to-1-mappings)

* [FIREWALL/NAT/OUTBOUND](#firewallnatoutbound)

  * [Read Outbound NAT Settings](#1-read-outbound-nat-settings)
  * [Update Outbound NAT Settings](#2-update-outbound-nat-settings)

* [FIREWALL/NAT/OUTBOUND/MAPPING](#firewallnatoutboundmapping)

  * [Create Outbound NAT Mappings](#1-create-outbound-nat-mappings)
  * [Delete Outbound NAT Mappings](#2-delete-outbound-nat-mappings)
  * [Read Outbound NAT Mappings](#3-read-outbound-nat-mappings)
  * [Update Outbound NAT Mappings](#4-update-outbound-nat-mappings)

* [FIREWALL/NAT/PORTFOWARD](#firewallnatportfoward)

  * [Create NAT Port Forwards](#1-create-nat-port-forwards)
  * [Delete NAT Port Forwards](#2-delete-nat-port-forwards)
  * [Read NAT Port Forwards](#3-read-nat-port-forwards)
  * [Update NAT Port Forwards](#4-update-nat-port-forwards)

* [FIREWALL/RULE](#firewallrule)

  * [Create Firewall Rules](#1-create-firewall-rules)
  * [Delete Firewall Rules](#2-delete-firewall-rules)
  * [Read Firewall Rules](#3-read-firewall-rules)
  * [Update Firewall Rules](#4-update-firewall-rules)

* [FIREWALL/SCHEDULE](#firewallschedule)

  * [Create Schedule](#1-create-schedule)
  * [Delete Schedule](#2-delete-schedule)
  * [Read schedules](#3-read-schedules)
  * [Update Schedule](#4-update-schedule)

* [FIREWALL/SCHEDULE/TIME_RANGE](#firewallscheduletime_range)

  * [Create Schedule Time Range](#1-create-schedule-time-range)
  * [Delete Schedule Time Range](#2-delete-schedule-time-range)

* [FIREWALL/STATES](#firewallstates)

  * [Read Firewall States](#1-read-firewall-states)

* [FIREWALL/STATES/SIZE](#firewallstatessize)

  * [Read Firewall State Size](#1-read-firewall-state-size)
  * [Update Firewall State Size](#2-update-firewall-state-size)

* [FIREWALL/TRAFFIC_SHAPER](#firewalltraffic_shaper)

  * [Create Traffic Shaper](#1-create-traffic-shaper)
  * [Delete Traffic Shaper](#2-delete-traffic-shaper)
  * [Read Traffic Shapers](#3-read-traffic-shapers)
  * [Update Traffic Shaper](#4-update-traffic-shaper)

* [FIREWALL/TRAFFIC_SHAPER/LIMITER](#firewalltraffic_shaperlimiter)

  * [Create Limiter](#1-create-limiter)
  * [Delete limiter](#2-delete-limiter)
  * [Read Limiters](#3-read-limiters)

* [FIREWALL/TRAFFIC_SHAPER/LIMITER/BANDWIDTH](#firewalltraffic_shaperlimiterbandwidth)

  * [Create Limiter Bandwidth](#1-create-limiter-bandwidth)
  * [Delete Limiter Bandwidth](#2-delete-limiter-bandwidth)

* [FIREWALL/TRAFFIC_SHAPER/LIMITER/QUEUE](#firewalltraffic_shaperlimiterqueue)

  * [Create Limiter Queue](#1-create-limiter-queue)
  * [Delete Limiter Queue](#2-delete-limiter-queue)

* [FIREWALL/TRAFFIC_SHAPER/QUEUE](#firewalltraffic_shaperqueue)

  * [Create Traffic Shaper Queue](#1-create-traffic-shaper-queue)
  * [Delete Traffic Shaper Queue](#2-delete-traffic-shaper-queue)

* [FIREWALL/VIRTUAL_IP](#firewallvirtual_ip)

  * [Create Virtual IPs](#1-create-virtual-ips)
  * [Delete Virtual IPs](#2-delete-virtual-ips)
  * [Read Virtual IPs](#3-read-virtual-ips)
  * [Update Virtual IPs](#4-update-virtual-ips)

* [INTERFACE](#interface)

  * [Create Interfaces](#1-create-interfaces)
  * [Delete Interfaces](#2-delete-interfaces)
  * [Read Interfaces](#3-read-interfaces)
  * [Update Interfaces](#4-update-interfaces)

* [INTERFACE/APPLY](#interfaceapply)

  * [Apply Interfaces](#1-apply-interfaces)

* [INTERFACE/VLAN](#interfacevlan)

  * [Create Interface VLANs](#1-create-interface-vlans)
  * [Delete Interface VLANs](#2-delete-interface-vlans)
  * [Read Interface VLANs](#3-read-interface-vlans)
  * [Update Interface VLANs](#4-update-interface-vlans)

* [ROUTING/APPLY](#routingapply)

  * [Apply Routing](#1-apply-routing)

* [ROUTING/GATEWAY](#routinggateway)

  * [Create Routing Gateways](#1-create-routing-gateways)
  * [Delete Routing Gateways](#2-delete-routing-gateways)
  * [Read Routing Gateways](#3-read-routing-gateways)
  * [Update Routing Gateways](#4-update-routing-gateways)

* [ROUTING/STATIC_ROUTE](#routingstatic_route)

  * [Create Static Routes](#1-create-static-routes)
  * [Delete Static Routes](#2-delete-static-routes)
  * [Read Static Routes](#3-read-static-routes)
  * [Update Static Routes](#4-update-static-routes)

* [SERVICES](#services)

  * [Read Services](#1-read-services)
  * [Restart All Services](#2-restart-all-services)
  * [Start All Services](#3-start-all-services)
  * [Stop All Services](#4-stop-all-services)

* [SERVICES/DDNS](#servicesddns)

  * [Read Dynamic DNS](#1-read-dynamic-dns)

* [SERVICES/DHCPD](#servicesdhcpd)

  * [Read DHCPd Service Configuration](#1-read-dhcpd-service-configuration)
  * [Restart DHCPd Service](#2-restart-dhcpd-service)
  * [Start DHCPd Service](#3-start-dhcpd-service)
  * [Stop DHCPd Service](#4-stop-dhcpd-service)
  * [Update DHCPd Service Configuration](#5-update-dhcpd-service-configuration)

* [SERVICES/DHCPD/LEASE](#servicesdhcpdlease)

  * [Read DHCPd Leases](#1-read-dhcpd-leases)

* [SERVICES/DHCPD/STATIC_MAPPING](#servicesdhcpdstatic_mapping)

  * [Create DHCPd Static Mappings](#1-create-dhcpd-static-mappings)
  * [Delete DHCPd Static Mappings](#2-delete-dhcpd-static-mappings)
  * [Read DHCPd Static Mappings](#3-read-dhcpd-static-mappings)
  * [Update DHCPd Static Mappings](#4-update-dhcpd-static-mappings)

* [SERVICES/DPINGER](#servicesdpinger)

  * [Restart DPINGER Service](#1-restart-dpinger-service)
  * [Start DPINGER Service](#2-start-dpinger-service)
  * [Stop DPINGER Service](#3-stop-dpinger-service)

* [SERVICES/NTPD](#servicesntpd)

  * [Read NTPd Service](#1-read-ntpd-service)
  * [Restart NTPd Service](#2-restart-ntpd-service)
  * [Start NTPd Service](#3-start-ntpd-service)
  * [Stop NTPd Service](#4-stop-ntpd-service)
  * [Update NTPd Service](#5-update-ntpd-service)

* [SERVICES/NTPD/TIME_SERVER](#servicesntpdtime_server)

  * [Create NTPd Time Server](#1-create-ntpd-time-server)
  * [Delete NTPd Time Server](#2-delete-ntpd-time-server)

* [SERVICES/SSHD](#servicessshd)

  * [Read SSHd Configuration](#1-read-sshd-configuration)
  * [Restart SSHd Service](#2-restart-sshd-service)
  * [Start SSHd Service](#3-start-sshd-service)
  * [Stop SSHd Service](#4-stop-sshd-service)
  * [Update SSHd Configuration](#5-update-sshd-configuration)

* [SERVICES/SYSLOGD](#servicessyslogd)

  * [Restart SYSLOGd Service](#1-restart-syslogd-service)
  * [Start SYSLOGd Service](#2-start-syslogd-service)
  * [Stop SYSLOGd Service](#3-stop-syslogd-service)

* [SERVICES/UNBOUND](#servicesunbound)

  * [Apply Pending Unbound Changes](#1-apply-pending-unbound-changes)
  * [Read Unbound Configuration](#2-read-unbound-configuration)
  * [Restart Unbound Service](#3-restart-unbound-service)
  * [Start Unbound Service](#4-start-unbound-service)
  * [Stop Unbound Service](#5-stop-unbound-service)

* [SERVICES/UNBOUND/HOST_OVERRIDE](#servicesunboundhost_override)

  * [Create Unbound Host Override](#1-create-unbound-host-override)
  * [Delete Unbound Host Override](#2-delete-unbound-host-override)
  * [Read Unbound Host Override](#3-read-unbound-host-override)
  * [Update Unbound Host Override](#4-update-unbound-host-override)

* [SERVICES/UNBOUND/HOST_OVERRIDE/ALIAS](#servicesunboundhost_overridealias)

  * [Create Unbound Host Override Alias](#1-create-unbound-host-override-alias)

* [STATUS/CARP](#statuscarp)

  * [Read CARP Status](#1-read-carp-status)
  * [Update CARP Status](#2-update-carp-status)

* [STATUS/GATEWAY](#statusgateway)

  * [Read Gateway Status](#1-read-gateway-status)

* [STATUS/INTERFACE](#statusinterface)

  * [Read Interface Status](#1-read-interface-status)

* [STATUS/LOG](#statuslog)

  * [Read Configuration History Status Log](#1-read-configuration-history-status-log)
  * [Read DHCP Status Log](#2-read-dhcp-status-log)
  * [Read Firewall Status Log](#3-read-firewall-status-log)
  * [Read System Status Log](#4-read-system-status-log)

* [STATUS/SYSTEM](#statussystem)

  * [Read System Status](#1-read-system-status)

* [SYSTEM/API](#systemapi)

  * [Read System API Configuration](#1-read-system-api-configuration)
  * [Read System API Error Library](#2-read-system-api-error-library)
  * [Update System API Configuration](#3-update-system-api-configuration)

* [SYSTEM/ARP](#systemarp)

  * [Delete System ARP Table](#1-delete-system-arp-table)
  * [Read System ARP Table](#2-read-system-arp-table)

* [SYSTEM/CERTIFICATE](#systemcertificate)

  * [Create System Certificates](#1-create-system-certificates)
  * [Delete System Certificates](#2-delete-system-certificates)
  * [Read System Certificates](#3-read-system-certificates)

* [SYSTEM/CONFIG](#systemconfig)

  * [Read System Configuration](#1-read-system-configuration)

* [SYSTEM/DNS](#systemdns)

  * [Read System DNS](#1-read-system-dns)
  * [Update System DNS](#2-update-system-dns)

* [SYSTEM/DNS/SERVER](#systemdnsserver)

  * [Create System DNS Server](#1-create-system-dns-server)
  * [Delete System DNS Server](#2-delete-system-dns-server)

* [SYSTEM/HALT](#systemhalt)

  * [Halt System](#1-halt-system)

* [SYSTEM/HOSTNAME](#systemhostname)

  * [Read System Hostname](#1-read-system-hostname)
  * [Update System Hostname](#2-update-system-hostname)

* [SYSTEM/REBOOT](#systemreboot)

  * [Create System Reboot](#1-create-system-reboot)

* [SYSTEM/TABLE](#systemtable)

  * [Read System Tables](#1-read-system-tables)

* [SYSTEM/TUNABLE](#systemtunable)

  * [Create System Tunables](#1-create-system-tunables)
  * [Delete System Tunables](#2-delete-system-tunables)
  * [Read System Tunables](#3-read-system-tunables)
  * [Update System Tunables](#4-update-system-tunables)

* [SYSTEM/VERSION](#systemversion)

  * [Read System Version](#1-read-system-version)

* [USER](#user)

  * [Create Users](#1-create-users)
  * [Delete Users](#2-delete-users)
  * [Read Users](#3-read-users)
  * [Update Users](#4-update-users)

* [USER/AUTH_SERVER](#userauth_server)

  * [Create LDAP Auth Servers](#1-create-ldap-auth-servers)
  * [Create RADIUS Auth Servers](#2-create-radius-auth-servers)
  * [Delete Auth Servers](#3-delete-auth-servers)
  * [Delete LDAP Auth Servers](#4-delete-ldap-auth-servers)
  * [Delete RADIUS Auth Servers](#5-delete-radius-auth-servers)
  * [Read Auth Servers](#6-read-auth-servers)
  * [Read LDAP Auth Servers](#7-read-ldap-auth-servers)
  * [Read RADIUS Auth Servers](#8-read-radius-auth-servers)

* [USER/GROUP](#usergroup)

  * [Create User Group](#1-create-user-group)
  * [Delete User Group](#2-delete-user-group)

* [USER/PRIVILEGE](#userprivilege)

  * [Create User Privileges](#1-create-user-privileges)
  * [Delete User Privileges](#2-delete-user-privileges)


--------


## ACCESS_TOKEN
API endpoints used to receive access tokens used to authenticate API requests. Only applicable when the API authentication mode is set to JWT.



### 1. Request Access Token


Receive a temporary access token using your pfSense local database credentials.


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/access_token
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| client-id | string | Specify the client's pfSense username |
| client-token | string | Specify the client's pfSense password |



***Body:***

```js        
{
	"client-id": "admin", 
	"client-token": "pfsense"
}
```



## FIREWALL/ALIAS



### 1. Create Firewall Aliases


Add a new host, network or port firewall alias.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-aliases-edit`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/alias
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Name of new alias. _Only alpha-numeric and underscore characters are allowed, other characters will be replaced_ |
| type | string | Alias type. Current supported alias types are`host`, `network`, and `port` aliases. |
| descr | string | Description of new alias (optional) |
| address | string or array | Array of values to add to alias. A single value may be specified as string. |
| detail | string or array | Array of descriptions for alias values. Descriptions must match the order the that they are specified in the `address` array. Single descriptions may be specified as string |



***Body:***

```js        
{
	"name": "RFC1918",
	"type": "network",
	"descr": "Networks reserved in RFC1918",
	"address": ["10.0.0.0/8", "172.16.0.0/12", "192.168.0.0/24"],
	"detail": ["Class A","Class B","Class C"]
}
```



### 2. Delete Firewall Aliases


Delete an existing alias and reload filter.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-aliases-edit`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/alias
```



***Body:***

```js        
{
	"id": "RFC1918"
}
```



### 3. Read Firewall Aliases


Read firewall aliases.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-aliases`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/alias
```



### 4. Update Firewall Aliases


Modify an existing firewall alias.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-aliases-edit`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/alias
```



***Body:***

```js        
{
	"id": "RFC1918",
	"name": "UPDATED_RFC1918",
	"type": "network",
	"descr": "Example description",
	"address": ["192.168.0.0/16", "172.16.0.0/12", "10.0.0.0/8"],
	"detail": ["Class A","Class B","Class C"]
}
```



## FIREWALL/ALIAS/ENTRY



### 1. Create Firewall Alias Entries


Add new entries to an existing firewall alias.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-aliases-edit`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/alias/entry
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Name of alias to add new address values |
| address | string or array | Array of values to add to alias. A single value may be specified as string. |
| detail | string or array | Array of descriptions for alias values. Descriptions must match the order the that they are specified in the `address` array. Single descriptions may be specified as string. If you pass In less `detail` values than `address` values, a default auto-created detail will be applied to the remaining values. (optional) |



***Body:***

```js        
{
	"name": "RFC1918",
	"address": ["10.0.0.0/8", "172.16.0.0/12", "192.168.0.0/16"],
	"detail": "Hypervisor"
}
```



### 2. Delete Firewall Alias Entries


Delete existing entries from an existing firewall alias.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-aliases-edit`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/alias/entry
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Name of alias to delete address values from |
| address | string | Array of values to delete from alias. A single value may be specified as string. |



***Body:***

```js        
{
	"name": "RFC1918",
	"address": ["10.0.0.0/8", "172.16.0.0/12"]
}
```



## FIREWALL/APPLY



### 1. Apply Firewall


Apply pending firewall changes. This will reload all filter items. This endpoint returns no data.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-rules`, `page-firewall-rules-edit`, `page-firewall-aliases`, `page-firewall-aliases-edit`, `page-firewall-nat-1-1`, `page-firewall-nat-1-1-edit`, `page-firewall-nat-outbound`, `page-firewall-nat-outbound-edit`, `page-firewall-nat-portforward`, `page-firewall-nat-portforward-edit`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/apply
```



***Body:***

```js        
{
}
```



## FIREWALL/NAT/ONE_TO_ONE



### 1. Create NAT 1-to-1 Mappings


Add a new NAT 1:1 Mapping.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-nat-1-1-edit`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/nat/one_to_one
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| interface | string | Set which interface the mapping will apply to. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0). Floating rules are not supported.  |
| src | string | Set the source address of the mapping. This may be a single IP, network CIDR, alias name, or interface. When specifying an interface, you may use the physical interface ID, the descriptive interface name, or the pfSense ID. To use only interface address, add `ip` to the end of the interface name otherwise the entire interface's subnet is implied. To negate the context of the source address, you may prepend the address with `!` |
| dst | string | Set the destination address of the mapping. This may be a single IP, network CIDR, alias name, or interface. When specifying an interface, you may use the physical interface ID, the descriptive interface name, or the pfSense ID. To only use interface address, add `ip` to the end of the interface name otherwise the entire interface's subnet is implied. To negate the context of the source address, you may prepend the address with `!` |
| external | string | Specify IPv4 or IPv6 external address to map Inside traffic to. This Is typically an address on an uplink Interface. |
| natreflection | string | Set the NAT reflection mode explicitly. Options are `enable` or `disable`. (optional) |
| descr | string | Set a description for the mapping (optional) |
| disabled | boolean | Disable the mapping upon creation (optional) |
| nobinat | boolean | Disable binat. This excludes the address from a later, more general, rule. (optional) |
| top | boolean | Add this mapping to top of access control list (optional) |
| apply | boolean | Specify whether or not you would like this 1:1 mapping to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are creating multiple 1:1 mappings at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/firewall/apply` endpoint. Otherwise, If you are only creating a single 1:1 mapping, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
	"interface": "WAN",
	"src": "any",
	"dst": "em0ip",
	"external": "1.2.3.4",
	"natreflection": "enable",
	"descr": "Test 1:1 NAT entry",
	"nobinat": true,
	"top": false,
    "apply": true
}
```



### 2. Delete NAT 1-to-1 Mappings


Delete an existing NAT 1:1 mapping by ID.<br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-nat-1-1-edit`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/nat/one_to_one
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | string or integer | Specify the 1:1 NAT mapping ID to delete |
| apply | boolean | Specify whether or not you would like this 1:1 mapping deletion to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are deleting multiple 1:1 mappings at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/firewall/apply` endpoint. Otherwise, If you are only deleting a single 1:1 mapping, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
	"id": 0, 
    "apply": true
}
```



### 3. Read NAT 1-to-1 Mappings


Read 1:1 NAT mappings.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-nat-1-1`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/nat/one_to_one
```



***Body:***

```js        
{
    
}
```



### 4. Update NAT 1-to-1 Mappings


Update an existing NAT 1:1 Mapping.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-nat-1-1-edit`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/nat/one_to_one
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | integer | Specify the ID of the 1:1 mapping to update. |
| interface | string | Update which interface the mapping will apply to. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0). (optional) |
| src | string | Update the source address of the mapping. This may be a single IP, network CIDR, alias name, or interface. When specifying an interface, you may use the physical interface ID, the descriptive interface name, or the pfSense ID. To use only interface address, add `ip` to the end of the interface name otherwise the entire interface's subnet is implied. To negate the context of the source address, you may prepend the address with `!` |
| dst | string | Update the destination address of the mapping. This may be a single IP, network CIDR, alias name, or interface. When specifying an interface, you may use the physical interface ID, the descriptive interface name, or the pfSense ID. To only use interface address, add `ip` to the end of the interface name otherwise the entire interface's subnet is implied. To negate the context of the source address, you may prepend the address with `!` (optional) |
| external | string | Update the IPv4 or IPv6 external address to map Inside traffic to. This Is typically an address on an uplink Interface. (optional) |
| natreflection | string | Update the NAT reflection mode explicitly. Options are `enable` or `disable`. (optional) |
| descr | string | Update the description for the mapping (optional) |
| disabled | boolean | Enable or disable the mapping upon update. True to disable, false to enable. (optional) |
| nobinat | boolean | Enable or disable binat. This excludes the address from a later, more general, rule. True to disable binat, false to enable binat. (optional) |
| top | boolean | Move this mapping to top of access control list upon update (optional) |
| apply | boolean | Specify whether or not you would like this 1:1 mapping update to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are updating multiple 1:1 mappings at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/firewall/apply` endpoint. Otherwise, If you are only updating a single 1:1 mapping, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
	"interface": "LAN",
	"src": "10.0.0.0/24",
	"dst": "!1.2.3.4",
	"external": "4.3.2.1",
	"natreflection": "disable",
	"descr": "Updated test 1:1 NAT entry",
    "disabled": true,
	"nobinat": false,
	"top": true,
    "apply": false
}
```



## FIREWALL/NAT/OUTBOUND



### 1. Read Outbound NAT Settings


Read outbound NAT mode settings.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-nat-outbound`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/nat/outbound
```



***Body:***

```js        
{
    
}
```



### 2. Update Outbound NAT Settings


Update outbound NAT mode settings.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-nat-outbound`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/nat/outbound
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| mode | string | Update the outbound NAT mode. Options are `automatic` to automatically generate outbound NAT rules, `hybrid` to support both automatic and manual outbound NAT rules , `advanced` to require all rules to be entered manually, or `disabled` to disable outbound NAT altogether. If updating to `advanced` from `automatic` or `hybrid`, the API will automatically create manual entries for each automatically generated outbound NAT entry. |



***Body:***

```js        
{
    
}
```



## FIREWALL/NAT/OUTBOUND/MAPPING



### 1. Create Outbound NAT Mappings


Create new outbound NAT mappings.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-nat-outbound-edit`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/nat/outbound/mapping
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| interface | string | Set which interface the mapping will apply to. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0).  |
| protocol | string | Set which transfer protocol the mapping will apply to.  |
| src | string | Set the source address of the firewall rule. This must be an IP, CIDR, alias or any. |
| dst | string | Set the destination address of the firewall rule. This may be a single IP, network CIDR, or alias name. To negate the context of the  address, you may prepend the address with `!` |
| srcport | string or integer | Set the TCP and/or UDP source port of the firewall rule. This is only necessary if you have specified the `protocol` to `tcp`, `udp`, `tcp/udp` |
| dstport | string or integer | Set the TCP and/or UDP destination port of the firewall rule. This is only necessary if you have specified the `protocol` to `tcp`, `udp`, `tcp/udp` |
| target | string | Specify the external IP to map this traffic to. This may be an IP address, IP subnet, alias, or empty string to use the Interface address.  |
| natport | string | Set the TCP and/or UDP  port or port range to utilize when NATing (optional) |
| staticnatport | boolean | Enable or disable static NAT ports. When enabling this field, any existing `natport` value will be lost. Defaults to false. (optional) |
| descr | string | Set a description for the rule (optional) |
| poolopts | string | Set the outbound NAT pool option for load balancing. Options are `round-robin`, `round-robin sticky-address`, `random`, `random sticky-address`, `source-hash`, `bitmask` or empty string for default. (optional) |
| source_hash_key | string | Set a custom key hash to use when utilizing the `source-hash` NAT pool option. Value must start with `0x` following a 32 digit hex value. If this field is not specified, a random key hash will be generated. This field Is only available when `poolopts` Is set to `source-hash`.   (optional) |
| disabled | boolean | Disable the rule upon creation. Defaults to false. (optional) |
| nonat | boolean | Enable or disable NAT for traffic that matches this rule. True for no NAT, false to enable NAT. Defaults to false. (optional) |
| top | boolean | Add this mapping to top of access control list. Defaults to false. (optional) |
| apply | boolean | Specify whether or not you would like this outbound NAT mapping to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are creating multiple outbound NAT mappings at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/firewall/apply` endpoint. Otherwise, If you are only creating a single outbound NAT mapping, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
	"interface": "WAN",
	"protocol": "tcp",
	"src": "any",
	"srcport": "433",
	"dst": "em0ip",
	"dstport": "443",
	"target": "192.168.1.123",
	"local-port": "443",
	"natreflection": "purenat",
	"descr": "Forward pb to lc",
	"nosync": true,
	"top": false
}
```



### 2. Delete Outbound NAT Mappings


Update existing outbound NAT mappings.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-nat-outbound-edit`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/nat/outbound/mapping
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | integer | Specify the ID of the outbound NAT mapping to update |
| apply | boolean | Specify whether or not you would like this outbound NAT mapping deletion to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are deleting multiple outbound NAT mappings at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/firewall/apply` endpoint. Otherwise, If you are only deleting a single outbound NAT mapping, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
	"interface": "WAN",
	"protocol": "tcp",
	"src": "any",
	"srcport": "433",
	"dst": "em0ip",
	"dstport": "443",
	"target": "192.168.1.123",
	"local-port": "443",
	"natreflection": "purenat",
	"descr": "Forward pb to lc",
	"nosync": true,
	"top": false
}
```



### 3. Read Outbound NAT Mappings


Read existing outbound NAT mode mappings.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-nat-outbound`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/nat/outbound/mapping
```



***Body:***

```js        
{
    
}
```



### 4. Update Outbound NAT Mappings


Update existing outbound NAT mappings.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-nat-outbound-edit`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/nat/outbound/mapping
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | integer | Specify the ID of the outbound NAT mapping to update |
| interface | string | Update the interface the mapping will apply to. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0). (optional) |
| protocol | string | Update the transfer protocol the mapping will apply to. (optional) |
| src | string | Update the source address of the firewall rule. This must be an IP, CIDR, alias or any. (optional) |
| dst | string | Update the destination address of the firewall rule. This may be a single IP, network CIDR, or alias name. To negate the context of the  address, you may prepend the address with `!` (optional) |
| srcport | string or integer | Update the TCP and/or UDP source port of the firewall rule. This is only necessary if you have specified the `protocol` to `tcp`, `udp`, `tcp/udp` (optional) |
| dstport | string or integer | Update the TCP and/or UDP destination port of the firewall rule. This is only necessary if you have specified the `protocol` to `tcp`, `udp`, `tcp/udp` (optional) |
| target | string | Update the external IP to map this traffic to. This may be an IP address, IP subnet, alias, or empty string to use the Interface address. (optional) |
| natport | string | Update the TCP and/or UDP  port or port range to utilize when NATing (optional) |
| staticnatport | boolean | Enable or disable static NAT ports. When enabling this field, any existing `natport` value will be lost. Defaults to false. (optional) |
| descr | string | Update the description for the rule (optional) |
| poolopts | string | Update the outbound NAT pool option for load balancing. Options are `round-robin`, `round-robin sticky-address`, `random`, `random sticky-address`, `source-hash`, `bitmask` or empty string for default. (optional) |
| source_hash_key | string | Update the hash to  a custom key hash to use when utilizing the `source-hash` NAT pool option. Value must start with `0x` following a 32 digit hex value. If this field is not specified, a random key hash will be generated. This field Is only available when `poolopts` Is set to `source-hash`.   (optional) |
| disabled | boolean | Disable the rule upon creation. Defaults to false. (optional) |
| nonat | boolean | Enable or disable NAT for traffic that matches this rule. True for no NAT, false to enable NAT. Defaults to false. (optional) |
| top | boolean | Move this mapping to top of access control list. Defaults to false. (optional) |
| apply | boolean | Specify whether or not you would like this outbound NAT mapping update to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are updating multiple outbound NAT mappings at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/firewall/apply` endpoint. Otherwise, If you are only updating a single outbound NAT mapping, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
	"interface": "WAN",
	"protocol": "tcp",
	"src": "any",
	"srcport": "433",
	"dst": "em0ip",
	"dstport": "443",
	"target": "192.168.1.123",
	"local-port": "443",
	"natreflection": "purenat",
	"descr": "Forward pb to lc",
	"nosync": true,
	"top": false
}
```



## FIREWALL/NAT/PORTFOWARD



### 1. Create NAT Port Forwards


Add a new NAT port forward rule.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-nat-portforward-edit`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/nat/port_forward
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| interface | string | Set which interface the rule will apply to. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0). Floating rules are not supported.  |
| protocol | string | Set which transfer protocol the rule will apply to. If `tcp`, `udp`, `tcp/udp`, you must define a source and destination port |
| src | string | Set the source address of the firewall rule. This may be a single IP, network CIDR, alias name, or interface. When specifying an interface, you may use the physical interface ID, the descriptive interface name, or the pfSense ID. To use only interface address, add `ip` to the end of the interface name otherwise the entire interface's subnet is implied. To negate the context of the source address, you may prepend the address with `!` |
| dst | string | Set the destination address of the firewall rule. This may be a single IP, network CIDR, alias name, or interface. When specifying an interface, you may use the physical interface ID, the descriptive interface name, or the pfSense ID. To only use interface address, add `ip` to the end of the interface name otherwise the entire interface's subnet is implied. To negate the context of the source address, you may prepend the address with `!` |
| srcport | string or integer | Set the TCP and/or UDP source port of the firewall rule. This is only necessary if you have specified the `protocol` to `tcp`, `udp`, `tcp/udp` |
| dstport | string or integer | Set the TCP and/or UDP destination port of the firewall rule. This is only necessary if you have specified the `protocol` to `tcp`, `udp`, `tcp/udp` |
| target | string | Specify the IP to forward traffic to  |
| local-port | string | Set the TCP and/or UDP  port to forward traffic to. This is only necessary if you have specified the `protocol` to `tcp`, `udp`, `tcp/udp`. Port ranges may be specified using colon or hyphen. |
| natreflection | string | Set the NAT reflection mode explicitly (optional) |
| descr | string | Set a description for the rule (optional) |
| disabled | boolean | Disable the rule upon creation (optional) |
| top | boolean | Add this port forward rule to top of access control list (optional) |
| apply | boolean | Specify whether or not you would like this port forward to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are creating multiple port forwards at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/firewall/apply` endpoint. Otherwise, If you are only creating a single port forward, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
	"interface": "WAN",
	"protocol": "tcp",
	"src": "any",
	"srcport": "433",
	"dst": "em0ip",
	"dstport": "443",
	"target": "192.168.1.123",
	"local-port": "443",
	"natreflection": "purenat",
	"descr": "Forward pb to lc",
	"nosync": true,
	"top": false
}
```



### 2. Delete NAT Port Forwards


Delete an existing NAT rule by ID.<br>
_Note: use caution when making this call, removing rules may result in API/webConfigurator lockout_<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-nat-portforward-edit`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/nat/port_forward
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | string or integer | Specify the rule ID to delete |
| apply | boolean | Specify whether or not you would like this port forward deletion to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are deleting multiple port forwards at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/firewall/apply` endpoint. Otherwise, If you are only deleting a single port forward, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
	"id": 0
}
```



### 3. Read NAT Port Forwards


Read NAT port forward rules.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-nat-portforward`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/nat/port_forward
```



***Body:***

```js        
{
    
}
```



### 4. Update NAT Port Forwards


Update an existing port forward rule.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-nat-portforward-edit`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/nat/port_forward
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| Id | Integer | Specify the ID of the port forward rule to update. |
| interface | string | Update the interface the rule will apply to. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0). Floating rules are not supported. (optional) |
| protocol | string | Update which transfer protocol the rule will apply to. If `tcp`, `udp`, `tcp/udp`, you must define a source and destination port. (optional) |
| src | string | Update the source address of the firewall rule. This may be a single IP, network CIDR, alias name, or interface. When specifying an interface, you may use the physical interface ID, the descriptive interface name, or the pfSense ID. To use only interface address, add `ip` to the end of the interface name otherwise the entire interface's subnet is implied. To negate the context of the source address, you may prepend the address with `!` (optional) |
| dst | string | Update the destination address of the firewall rule. This may be a single IP, network CIDR, alias name, or interface. When specifying an interface, you may use the physical interface ID, the descriptive interface name, or the pfSense ID. To only use interface address, add `ip` to the end of the interface name otherwise the entire interface's subnet is implied. To negate the context of the source address, you may prepend the address with `!` (optional) |
| srcport | string or integer | Update the TCP and/or UDP source port of the firewall rule. This is only necessary if you have specified the `protocol` to `tcp`, `udp`, `tcp/udp` (optional) |
| dstport | string or integer | Update the TCP and/or UDP destination port of the firewall rule. This is only necessary if you have specified the `protocol` to `tcp`, `udp`, `tcp/udp` (optional) |
| target | string | Update the IP to forward traffic to (optional) |
| local-port | string | Update the TCP and/or UDP  port to forward traffic to. This is only necessary if you have specified the `protocol` to `tcp`, `udp`, `tcp/udp`. Port ranges may be specified using colon or hyphen. (optional) |
| natreflection | string | Update the NAT reflection mode explicitly (optional) |
| descr | string | Update a description for the rule (optional) |
| disabled | boolean | Enable or disable the rule upon creation. True to disable, false to enable (optional) |
| top | boolean | Move this port forward rule to top of access control list (optional) |
| apply | boolean | Specify whether or not you would like this port forward update to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are updating multiple port forwards at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/firewall/apply` endpoint. Otherwise, If you are only updating a single port forward, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
	"interface": "WAN",
	"protocol": "tcp",
	"src": "any",
	"srcport": "433",
	"dst": "em0ip",
	"dstport": "443",
	"target": "192.168.1.123",
	"local-port": "443",
	"natreflection": "purenat",
	"descr": "Forward pb to lc",
	"nosync": true,
	"top": false
}
```



## FIREWALL/RULE



### 1. Create Firewall Rules


Add a new firewall rule.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-rules-edit`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/rule
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| type | string | Set a firewall rule type (`pass`, `block`, `reject`) |
| interface | string | Set which interface the rule will apply to. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0). Floating rules are not supported.  |
| ipprotocol | string | Set which IP protocol(s) the rule will apply to (`inet`, `inet6`, `inet46`) |
| protocol | string | Set which transfer protocol the rule will apply to. If `tcp`, `udp`, `tcp/udp`, you must define a source and destination port |
| icmptype | string or array | Set the ICMP subtype of the firewall rule. Multiple values may be passed in as array, single values may be passed as string. _Only available when `protocol` is set to `icmp`. If `icmptype` is not specified all subtypes are assumed_ |
| src | string | Set the source address of the firewall rule. This may be a single IP, network CIDR, alias name, or interface. When specifying an interface, you may use the physical interface ID, the descriptive interface name, or the pfSense ID. To use only interface address, add `ip` to the end of the interface name otherwise the entire interface's subnet is implied. To negate the context of the source address, you may prepend the address with `!` |
| dst | string | Set the destination address of the firewall rule. This may be a single IP, network CIDR, alias name, or interface. When specifying an interface, you may use the physical interface ID, the descriptive interface name, or the pfSense ID. To only use interface address, add `ip` to the end of the interface name otherwise the entire interface's subnet is implied. To negate the context of the source address, you may prepend the address with `!` |
| srcport | string or integer | Set the TCP and/or UDP source port or port alias of the firewall rule. This is only necessary if you have specified the `protocol` to `tcp`, `udp`, `tcp/udp` |
| dstport | string or integer | Set the TCP and/or UDP destination port or port alias of the firewall rule. This is only necessary if you have specified the `protocol` to `tcp`, `udp`, `tcp/udp` |
| gateway | string | Set the routing gateway traffic will take upon match (optional) |
| sched | string | Set a firewall schedule to apply to this rule. This must be an existing firewall schedule name. (optional) |
| dnpipe | string | Specify a traffic shaper limiter (in) queue for this rule. This must be an existing traffic shaper limiter or queue. This field is required if a `pdnpipe` value is provided.Ioptional) |
| pdnpipe | string | Specify a traffic shaper limiter (out) queue for this rule. This must be an existing traffic shaper limiter or queue. This value cannot match the `dnpipe` value and must be a child queue if `dnpipe` is a child queue, or a parent limiiter if `dnpipe` is a parent limiter. (optional) |
| defaultqueue | string | Specify a default traffic shaper queue to apply to this rule. This must be an existing traffic shaper queue name. This field is required when an `ackqueue` value is provided. (optional) |
| ackqueue | string | Specify an acknowledge traffic shaper queue to apply to this rule. This must be an existing traffic shaper queue and cannot match the `defaultqueue` value. (optional) |
| disabled | boolean | Disable the rule upon creation (optional) |
| descr | string | Set a description for the rule (optional) |
| log | boolean | Enabling rule matched logging (optional) |
| top | boolean | Add firewall rule to top of access control list (optional) |
| apply | boolean | Specify whether or not you would like this rule to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are creating multiple rules at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/firewall/apply` endpoint. Otherwise, If you are only creating a single rule, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
	"type": "block",
	"interface": "wan",
	"ipprotocol": "inet",
	"protocol": "tcp/udp",
	"src": "172.16.209.138",
	"srcport": "any",
	"dst": "em0ip",
	"dstport": 443,
	"descr": "This is a test rule added via API",
	"top": true
}
```



### 2. Delete Firewall Rules


Delete an existing firewall rule.<br>
_Note: use caution when making this call, removing rules may result in API/webConfigurator lockout_<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-rules-edit`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/rule
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| tracker | string or integer | Specify the rule tracker ID to delete |
| apply | boolean | Specify whether or not you would like this rule deletion to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are deleting multiple rules at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/firewall/apply` endpoint. Otherwise, If you are only deleting a single rule, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
	"tracker": 1600981596
}
```



### 3. Read Firewall Rules


Read firewall rules.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-rules`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/rule
```



***Body:***

```js        
{
    "client-id": "admin",
    "client-token": "pfsense"
}
```



### 4. Update Firewall Rules


Update an existing firewall rule.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-rules-edit`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/rule
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| tracker | string or Integer | Specify the tracker ID of the rule to update |
| type | string | Update the firewall rule type (`pass`, `block`, `reject`) (optional) |
| interface | string | Update the interface the rule will apply to. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0). Floating rules are not supported. (optional) |
| ipprotocol | string | Update which IP protocol(s) the rule will apply to (`inet`, `inet6`, `inet46`) (optional) |
| protocol | string | Update the transfer protocol the rule will apply to. If `tcp`, `udp`, `tcp/udp`, you must define a source and destination port. (optional) |
| icmptype | string or array | Update the ICMP subtype of the firewall rule. Multiple values may be passed in as array, single values may be passed as string. _Only available when `protocol` is set to `icmp`. If `icmptype` is not specified all subtypes are assumed_ (optional) |
| src | string | Update the source address of the firewall rule. This may be a single IP, network CIDR, alias name, or interface. When specifying an interface, you may use the physical interface ID, the descriptive interface name, or the pfSense ID. To use only interface address, add `ip` to the end of the interface name otherwise the entire interface's subnet is implied. To negate the context of the source address, you may prepend the address with `!` (optional) |
| dst | string | Update the destination address of the firewall rule. This may be a single IP, network CIDR, alias name, or interface. When specifying an interface, you may use the physical interface ID, the descriptive interface name, or the pfSense ID. To only use interface address, add `ip` to the end of the interface name otherwise the entire interface's subnet is implied. To negate the context of the source address, you may prepend the address with `!` (optional) |
| srcport | string or integer | Update the TCP and/or UDP source port or port alias of the firewall rule. This is only necessary if you have specified the `protocol` to `tcp`, `udp`, `tcp/udp` (optional) |
| dstport | string or integer | Update the TCP and/or UDP destination port or port alias of the firewall rule. This is only necessary if you have specified the `protocol` to `tcp`, `udp`, `tcp/udp` |
| gateway | string | Update the routing gateway traffic will take upon match (optional) |
| sched | string | Update the firewall schedule to apply to this rule. This must be an existing firewall schedule name. Provide an empty string to remove the configured schedule from the rule. (optional) |
| dnpipe | string | Update the traffic shaper limiter (in) queue for this rule. This must be an existing traffic shaper limiter or queue. This field is required if a `pdnpipe` value is provided. To unset this value, pass in an empty string. This will also unset the existing `pdnpipe` value. (optional) |
| pdnpipe | string | Update the traffic shaper limiter (out) queue for this rule. This must be an existing traffic shaper limiter or queue. This value cannot match the `dnpipe` value and must be a child queue if `dnpipe` is a child queue, or a parent limiiter if `dnpipe` is a parent limiter. To unset this value, pass in an empty string. (optional) |
| defaultqueue | string | Update the default traffic shaper queue to apply to this rule. This must be an existing traffic shaper queue name. This field is required when an `ackqueue` value is provided. To unset this field, pass in an empty string. This will also unset the existing `ackqueue` value. (optional) |
| ackqueue | string | Update acknowledge traffic shaper queue to apply to this rule. This must be an existing traffic shaper queue and cannot match the `defaultqueue` value. To unset this field, pass in an empty string. (optional) |
| disabled | boolean | Disable the rule upon modification (optional) |
| descr | string | Update the description of the rule (optional) |
| log | boolean | Enable rule matched logging (optional) |
| top | boolean | Move firewall rule to top of access control list (optional) |
| apply | boolean | Specify whether or not you would like this rule update to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are updating multiple rules at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/firewall/apply` endpoint. Otherwise, If you are only updating a single rule, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
    "tracker": 1600981687,
	"type": "pass",
	"interface": "wan",
	"ipprotocol": "inet",
	"protocol": "icmp",
	"src": "172.16.209.138",
	"dst": "em0ip",
	"descr": "This is an updated test rule added via API",
	"top": true
}
```



## FIREWALL/SCHEDULE



### 1. Create Schedule


Add a firewall schedule.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-schedules-edit`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/schedule
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Specify a name for this firewall schedule. This value must be between 1 and 32 characters, and can only contain alphanumerics, underscores and hyphens. This value must be unique from all other firewall schedules.  |
| descr | string | Specify a description for this firewall schedule (optional) |
| timerange | array | Specify an array of time range objects for this schedule. Each object must contain the required fields as specified by the /api/v1/firewall/schedule/time_range endpoint. At least 1 time range object must be specified. |



***Body:***

```js        
{
    "name": "Test_Schedule",
    "descr": "Test firewall schedule",
    "timerange": [
        {
            "month": "1,3,5",
            "day": "10,20,25",
            "hour": "0:15-20:00",
            "rangedescr": "On Jan 1, March 20, and May 25 from 0:15 to 20:00"
        },
        {
            "position": "1,2,3,4,5",
            "hour": "10:15-12:00",
            "rangedescr": "Every Mon, Tues, Wed, Thur, Fri from 10:15 to 12:00"
        }
    ]
}
```



### 2. Delete Schedule


Delete an existing firewall schedule.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-schedules-edit`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/schedule
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Specify the name of the firewall schedule to delete. |



***Body:***

```js        
{
    "name": "Test_Schedule"
}
```



### 3. Read schedules


Read all existing firewall schedules.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-schedules`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/schedule
```



### 4. Update Schedule


Update an existing firewall schedule.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-schedules-edit`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/schedule
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Specify the name of the firewall schedule to update. This field Is used to locate the object and cannot be used to change the name of the schedule. |
| descr | string | Update the description for this firewall schedule (optional) |
| timerange | array | Update the array of time range objects for this schedule. Each object must contain the required fields as specified by the /api/v1/firewall/schedule/time_range endpoint. If this field is passed in, at least 1 time range object must be specified. Specifying this field will overwrite any existing time ranges for this schedule. (optional) |



***Body:***

```js        
{
    "name": "Test_Schedule",
    "descr": "Updated test firewall schedule",
    "timerange": [
        {
            "month": "2,4,6",
            "day": "10,20,25",
            "hour": "0:15-20:00",
            "rangedescr": "On Feb 1, Apr 20, and Jun 25 from 0:15 to 20:00"
        },
        {
            "position": "6,7",
            "hour": "10:15-12:00",
            "rangedescr": "Every Sat, Sun from 10:15 to 12:00"
        }
    ]
}
```



## FIREWALL/SCHEDULE/TIME_RANGE



### 1. Create Schedule Time Range


Add a time range to an existing firewall schedule.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-schedules-edit`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/schedule/time_range
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Specify the name of the schedule to add this time range to. This must be an existing firewall schedule name. |
| position | string | Specify a comma-separated list of week days the schedule will be enable for. Options are `1` for Monday, `2` for Tuesday, `3` for Wednesday, `4` for Thursday, `5` for Friday, `6` for Saturday, and `7` for Sunday. For example, all weekdays would be formatted as such `"1,2,3,4,5"`. (optional) |
| month | string | Specify a comma-separated list of months the schedule will be enabled for. This will directly correspond with each value in the `day` field. For example, to enable this schedule on March 1st, and August 26th, the `month` field should be formatted as `"3,8"` and the `day` field should be formatted as `"1,26"`. This field will be required if the `position` field was not specified. This field cannot be set if the `position` field was set. |
| day | string | Specify a comma-separated list of days the schedule will be enabled for. This will directly correspond with each value in the `month` field. For example, to enable this schedule on March 1st, and August 26th, the `month` field should be formatted as `"3,8"` and the `day` field should be formatted as `"1,26"`. This field will be required if the `position` field was not specified. This field cannot be set if the `position` field was set. |
| hour | string | Specify the hour range this schedule will be enabled for. This must be two 24-hour time  strings formatted as either HH:MM or H:MM separated by `-` (e.g. `9:00-17:00`). The first time range must be less than the second time range. The minutes portion of the time range must be either `00`, `15`, `30`, `45`, or `59`. |
| rangedescr | string | Specify a description for this time range. (optional) |



***Body:***

```js        
{
    "name": "Test_Schedule",
    "month": "1,3,5",
    "day": "20,24,5",
    "hour": "9:00-17:00",
    "rangedescr": "Enable on Jan 20, Mar 24, and May 5 from 9AM to 5PM"
}
```



### 2. Delete Schedule Time Range


Delete a time range from an existing firewall schedule.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-schedules-edit`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/schedule/time_range
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Specify the name of the schedule to delete this time range from. This must be an existing firewall schedule name. |
| id | integer | Specify the ID of the time range to delete. This will be the configuration array index of the `timerange` object within the `schedule` object. |



***Body:***

```js        
{
    "name": "Test_Schedule",
    "id": 0
}
```



## FIREWALL/STATES



### 1. Read Firewall States


Read the current firewall states.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-diagnostics-statessummary`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/states
```



## FIREWALL/STATES/SIZE



### 1. Read Firewall State Size


Read the maximum firewall state size, the current firewall state size, and the default firewall state size.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-diagnostics-statessummary`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/states/size
```



### 2. Update Firewall State Size


Modify the maximum number of firewall state table entries allowed by the system<br>_Note: use caution when making this call, setting the maximum state table size to a value lower than the current number of firewall state entries WILL choke the system_<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-advanced-firewall`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/states/size
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| maximumstates | string or integer | Set the maximum number of firewall state entries. Specify an integer above 10, or string "default" to revert to the system default size |



***Body:***

```js        
{
	"maximumstates": "2000"
}
```



## FIREWALL/TRAFFIC_SHAPER



### 1. Create Traffic Shaper


Add a traffic shaper policy to an interface.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-trafficshaper`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/traffic_shaper
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| interface | string | Specify the interface to create the traffic shaper policy for. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0).  |
| scheduler | string | Specify the scheduler type for this traffic shaper. Choices are `HFSC`, `CBQ`, `FAIRQ`, `CODELQ`, and `PRIQ`. |
| bandwidthtype | string | Specify the bandwidth type to use when setting the bandwidth. Choices are `%` for percentage based bandwidth, `b` for bits, `Kb` for kilobits, `Mb` for megabits, and `Gb` for gigabits. |
| bandwidth | integer | Specify the bandwidth of this traffic shaper. This must be a numeric value of `1` or greater. If you have set the `bandwidthtype` to `%`, this value must be `100` or less. |
| enabled | boolean | Enable or disable this traffic shaper upon creation. Specify `true` to enable or `false` to disable. Defaults to `true`. (optional) |
| qlimit | integer | Specify the queue limit value for this traffic shaper. If set, this must be a value of `1` or greater. Defaults to no limit. (optional) |
| tbrconfig | integer | Specify the TBR size  for this traffic shaper. If set, this must be a value of `1` or greater. (optional) |
| apply | integer | Specify whether this traffic shaper should be applied immediately after creation. If `true`, the filter will be reloaded after creation. Otherwise, if `false`, the filter will not be reloaded and the change will not be applied until the next filter reload. Defaults to `false`. (optional) |



***Body:***

```js        
{
    "interface": "lan",
    "scheduler": "PRIQ",
    "bandwidthtype": "Gb",
    "bandwidth": 1,
    "enabled": false,
    "qlimit": 1000,
    "tbrconfig": 1000,
    "apply": true
}
```



### 2. Delete Traffic Shaper


Delete the traffic shaper policy for a specific interface.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-trafficshaper`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/traffic_shaper
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| interface | string | Specify the interface of the traffic shaper policy to delete. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0).  |
| apply | boolean | Specify whether this traffic shaper deletion should be applied immediately after deleting. If `true`, the filter will be reloaded after deleting. Otherwise, if `false`, the filter will not be reloaded and the change will not be applied until the next filter reload. Defaults to `false`. (optional) |



***Body:***

```js        
{
    "interface": "wan"
}
```



### 3. Read Traffic Shapers


Read all configured traffic shapers.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-trafficshaper`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/traffic_shaper
```



### 4. Update Traffic Shaper


Update an existing traffic shaper policy for an interface.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-trafficshaper`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/traffic_shaper
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| interface | string | Specify the interface of the traffic shaper policy to update. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0).  |
| scheduler | string | Update the scheduler type for this traffic shaper. Choices are `HFSC`, `CBQ`, `FAIRQ`, `CODELQ`, and `PRIQ`. (optional) |
| bandwidthtype | string | Update the bandwidth type to use when setting the bandwidth. Choices are `%` for percentage based bandwidth, `b` for bits, `Kb` for kilobits, `Mb` for megabits, and `Gb` for gigabits. If the bandwidth type is changed, a new `bandwidth` value will also be required. (optional) |
| bandwidth | integer | Update the bandwidth of this traffic shaper. This must be a numeric value of `1` or greater. If you have set the `bandwidthtype` to `%`, this value must be `100` or less. If the `bandwidthtype` value has been changed, this field will be required. (optional)  |
| enabled | boolean | Enable or disable this traffic shaper. Specify `true` to enable or `false` to disable.  (optional) |
| qlimit | integer | Update the queue limit value for this traffic shaper. If set, this must be a value of `1` or greater. (optional) |
| tbrconfig | integer | Update the TBR size  for this traffic shaper. If set, this must be a value of `1` or greater. (optional) |
| apply | integer | Specify whether this traffic shaper should be applied immediately after updating. If `true`, the filter will be reloaded after creation. Otherwise, if `false`, the filter will not be reloaded and the change will not be applied until the next filter reload. Defaults to `false`. (optional) |



***Body:***

```js        
{
    "interface": "lan",
    "scheduler": "PRIQ",
    "bandwidthtype": "%",
    "bandwidth": 50,
    "enabled": true,
    "qlimit": 10000,
    "tbrconfig": 10000,
    "apply": true
}
```



## FIREWALL/TRAFFIC_SHAPER/LIMITER



### 1. Create Limiter


Add a traffic shaper limiter.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-trafficshaper-limiter`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/traffic_shaper/limiter
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Specify a name for this limiter. This value must only contain alphanumerics, underscore and/or hyphens, must be 32 characters or less, and must be unique from all other limiters Including child queues. |
| bandwidth | array | Specify an array of bandwidth objects to assign to this limiter. Each object will require the fields as specified by the /api/v1/firewall/traffic_shaper/limiter/bandwidth endpoint documentation. At least one bandwidth object must be specified to create the limiter. |
| mask | string | Specify a limiter address mask type. Options are `none` for no mask, `srcaddress` for source addresses, or `dstaddress` for destination addresses. Defaults to `none`. (optional) |
| maskbits | integer | Specify the subnet bitmask to apply the limiter to. This value must be between 1 and 32. This field is only available when `mask` is set to `srcaddress` or `dstaddress`. Defaults to `32`. (optional) |
| maskbitsv6 | integer | Specify the IPv6 subnet bitmask to apply the limiter to. This value must be between 1 and 128. This field is only available when `mask` is set to `srcaddress` or `dstaddress`. Defaults to `128`. (optional) |
| description | string | Specify a description for this limiter. (optional) |
| aqm | string | Specify the Queue Management Algorithm for this limiter to use. Options are `droptail`, `codel`, `pie`, `red`, or `gred`.  |
| sched | string | Specify the scheduler for this limiter to use. Options are `wf2q+`, `fifo`, `qfq`, `rr`, `prio`, `fq_codel` or `fq_pie`. |
| enabled | boolean | Enable or disable this limiter upon creation. If `true`, the limiter will be enabled. If `false`, the limiter will be disabled. Defaults to `true`. (optional) |
| ecn | boolean | Enable Explicit Congestion Notification (ECN). If `true`, the limiter will be enabled. If `false`, the limiter will be disabled. Defaults to `false`. Note: not every AQM and/or scheduler supports ECN. (optional) |
| qlimit | integer | Specify the queue limit value for this limiter. This must be a numeric value of 1 or greater. (optional) |
| delay | integer | Specify the delay value for this limiter. This must be a numeric value between 0 and 10000. Defaults to 0. (optional) |
| plr | float | Specify the packet loss rate value for this limiter. This must be a numeric value between 0 and 1. Defaults to 0. (optional) |
| buckets | integer | Specify the buckets value for this limiter. This must be a numeric value between 16 and 65535.  (optional) |
| param_codel_target | integer | Specify the CoDel target parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `codel`. Defaults to `5`. (optional) |
| param_codel_interval | integer | Specify the CoDel interval parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `codel`. Defaults to `100`. (optional) |
| param_pie_target | integer | Specify the PIE target parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `pie`. Defaults to `15`. (optional) |
| param_pie_tupdate | integer | Specify the PIE tupdate parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `pie`. Defaults to `15`. (optional) |
| param_pie_alpha | integer | Specify the PIE alpha parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `pie`. Defaults to `125`. (optional) |
| param_pie_beta | integer | Specify the PIE beta parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `pie`. Defaults to `1250`. (optional) |
| param_pie_max_burst | integer | Specify the PIE max burst parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `pie`. Defaults to `150000`. (optional) |
| param_pie_max_ecnth | integer | Specify the PIE max ecnth parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `pie`. Defaults to `99`. (optional) |
| param_red_w_q | integer | Specify the RED w_q parameter for this limiter. This must be a numeric value of 1 or greater. Only available when the `aqm` is set to `red`. Defaults to `1`. (optional) |
| param_red_min_th | integer | Specify the RED min_th parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `red`. Defaults to `0`. (optional) |
| param_red_max_th | integer | Specify the RED max_th parameter for this limiter. This must be a numeric value of 1 or greater. Only available when the `aqm` is set to `red`. Defaults to `1`. (optional) |
| param_red_max_p | integer | Specify the RED max_p parameter for this limiter. This must be a numeric value of 1 or greater. Only available when the `aqm` is set to `red`. Defaults to `1`. (optional) |
| param_gred_w_q | integer | Specify the GRED w_q parameter for this limiter. This must be a numeric value of 1 or greater. Only available when the `aqm` is set to `gred`. Defaults to `1`. (optional) |
| param_gred_min_th | integer | Specify the GRED min_th parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `gred`. Defaults to `0`. (optional) |
| param_gred_max_th | integer | Specify the GRED max_th parameter for this limiter. This must be a numeric value of 1 or greater. Only available when the `aqm` is set to `gred`. Defaults to `1`. (optional) |
| param_gred_max_p | integer | Specify the GRED max_p parameter for this limiter. This must be a numeric value of 1 or greater. Only available when the `aqm` is set to `gred`. Defaults to `1`. (optional) |
| param_fq_codel_target | integer | Specify the FQ_CoDel target parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `sched` is set to `fq_codel`. Defaults to `5`. (optional) |
| param_fq_codel_interval | integer | Specify the FQ_CoDel interval parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `sched` is set to `fq_codel`. Defaults to `100`. (optional) |
| param_fq_codel_quantum | integer | Specify the FQ_CoDel quantum parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `sched` is set to `fq_codel`. Defaults to `1514`. (optional) |
| param_fq_codel_limit | integer | Specify the FQ_CoDel limit parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `sched` is set to `fq_codel`. Defaults to `10240`. (optional) |
| param_fq_codel_flow | integer | Specify the FQ_CoDel flow parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `sched` is set to `fq_codel`. Defaults to `1024`. (optional) |
| param_fq_pie_target | integer | Specify the FQ_PIE target parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `sched` is set to `fq_pie`. Defaults to `15`. (optional) |
| param_fq_pie_tupdate | integer | Specify the FQ_PIE tupdate parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `sched` is set to `fq_pie`. Defaults to `15`. (optional) |
| param_fq_pie_alpha | integer | Specify the FQ_PIE alpha parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `sched` is set to `fq_pie`. Defaults to `125`. (optional) |
| param_fq_pie_beta | integer | Specify the FQ_PIE beta parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `sched` is set to `fq_pie`. Defaults to `1250`. (optional) |
| param_fq_pie_max_burst | integer | Specify the FQ_PIE max burst parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `sched` is set to `fq_pie`. Defaults to `150000`. (optional) |
| param_fq_pie_max_burst | integer | Specify the FQ_PIE max ecnth parameter for this limiter. This must be a numeric value of 0 or greater. Only available when the `sched` is set to `fq_pie`. Defaults to `99`. (optional) |
| apply | boolean | Specify whether or not this limiter should be applied immediately after creation. If set to `true`, the firewall filter will reload and the limiter will be applied. If set to`false`, the firewall filter will not be reloaded and the limiter will not be applied until the filter Is reloaded. Defaults to `false`. (optional) |



***Body:***

```js        
{
    "name": "Test_Limiter",
    "bandwidth": [{"bw": 100, "bwscale": "Mb"}],
    "mask": "srcaddress",
    "maskbits": 31,
    "description": "Unit test",
    "aqm": "codel",
    "sched": "fq_pie",
    "delay": 1,
    "plr": 0.01,
    "buckets": 16,
    "apply": true
}
```



### 2. Delete limiter


Delete traffic shaper limiter.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-trafficshaper-limiter`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/traffic_shaper/limiter
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Specify the name of the limiter to delete. |
| apply | boolean | Specify whether or not this change should be applied immediately after deletion. If set to `true`, the firewall filter will reload. If set to`false`, the firewall filter will not be reloaded and the limiter deletion will not be applied until the filter Is reloaded. Defaults to `false`. (optional) |



### 3. Read Limiters


Read all traffic shaper limiters.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-trafficshaper-limiter`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/traffic_shaper/limiter
```



## FIREWALL/TRAFFIC_SHAPER/LIMITER/BANDWIDTH



### 1. Create Limiter Bandwidth


Create a limiter bandwidth setting.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-trafficshaper-limiter`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/traffic_shaper/limiter/bandwidth
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Specify the name of the parent limiter to add this bandwidth object to.  |
| bw | integer | Specify the amount of bandwidth allotted to the parent limiter. This must be a numeric value of 1 or greater. |
| bwscale | string | Specify the bandwidth scale type. Options are `b` for b/s, `Kb` for Kb/s, or `Mb` for Mb/s. |
| bwsched | string | Specify a schedule for this bandwidth setting. This must be an existing firewall schedule name. Defaults to `none`. (optional) |
| apply | boolean | Specify whether this bandwidth object should be applied immediately after creation. If set to `true`, the firewall filter will reload  immediately after creation. If set to `false`, the firewall filter will not be reloaded the and bandwidth object will not be applied to the backend configuration until the filter Is reloaded. Defaults to `false`. (optional) |



***Body:***

```js        
{
    "name": "Test_Limiter",
    "bw": 100,
    "bwscale": "Mb",
    "bwsched": "Test_Schedule"
}
```



### 2. Delete Limiter Bandwidth


Delete a limiter bandwidth setting.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-trafficshaper-limiter`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/traffic_shaper/limiter/bandwidth
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Specify the name of the parent limiter to delete this bandwidth object from.  |
| id | integer | Specify the ID of the bandwidth object to delete. The ID will be the array index of the object to delete within the limiter's bandwith-items array.  |
| apply | boolean | Specify whether this bandwidth object deletion should be applied immediately. If set to `true`, the firewall filter will reload  immediately after deletion. If set to `false`, the firewall filter will not be reloaded the and bandwidth object will not be removed from the backend configuration until the filter Is reloaded. Defaults to `false`. (optional) |



***Body:***

```js        
{
    "name": "Test_Limiter",
    "id": 0
}
```



## FIREWALL/TRAFFIC_SHAPER/LIMITER/QUEUE



### 1. Create Limiter Queue


Add a child queue to an existing traffic shaper limiter.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-trafficshaper-limiter`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/traffic_shaper/limiter/queue
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| limiter | string | Specify the name of the parent limiter to add this queue to.  |
| name | string | Specify a name for this limiter queue. This value must only contain alphanumerics, underscore and/or hyphens, must be 32 characters or less, and must be unique from all other limiters Including child queues. |
| mask | string | Specify a limiter address mask type. Options are `none` for no mask, `srcaddress` for source addresses, or `dstaddress` for destination addresses. Defaults to `none`. (optional) |
| maskbits | integer | Specify the subnet bitmask to apply the limiter queue to. This value must be between 1 and 32. This field is only available when `mask` is set to `srcaddress` or `dstaddress`. Defaults to `32`. (optional) |
| maskbitsv6 | integer | Specify the IPv6 subnet bitmask to apply the limiter queue to. This value must be between 1 and 128. This field is only available when `mask` is set to `srcaddress` or `dstaddress`. Defaults to `128`. (optional) |
| description | string | Specify a description for this limiter queue. (optional) |
| aqm | string | Specify the Queue Management Algorithm for this limiter queue to use. Options are `droptail`, `codel`, `pie`, `red`, or `gred`.  |
| enabled | boolean | Enable or disable this limiter queue upon creation. If `true`, the limiter will be enabled. If `false`, the limiter will be disabled. Defaults to `true`. (optional) |
| ecn | boolean | Enable Explicit Congestion Notification (ECN). If `true`, the limiter queue will be enabled. If `false`, the limiter will be disabled. Defaults to `false`. Note: not every AQM and/or scheduler supports ECN. (optional) |
| qlimit | integer | Specify the queue limit value for this limiter queue. This must be a numeric value of 1 or greater. (optional) |
| weight | integer | Specify the weight value for this limiter queue. This must be a numeric value between 1 and 100. (optional) |
| plr | float | Specify the packet loss rate value for this limiter queue. This must be a numeric value between 0 and 1. Defaults to 0. (optional) |
| buckets | integer | Specify the buckets value for this limiter queue. This must be a numeric value between 16 and 65535.  (optional) |
| param_codel_target | integer | Specify the CoDel target parameter for this limiter queue. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `codel`. Defaults to `5`. (optional) |
| param_codel_interval | integer | Specify the CoDel interval parameter for this limiter queue. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `codel`. Defaults to `100`. (optional) |
| param_pie_target | integer | Specify the PIE target parameter for this limiter queue. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `pie`. Defaults to `15`. (optional) |
| param_pie_tupdate | integer | Specify the PIE tupdate parameter for this limiter queue. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `pie`. Defaults to `15`. (optional) |
| param_pie_alpha | integer | Specify the PIE alpha parameter for this limiter queue. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `pie`. Defaults to `125`. (optional) |
| param_pie_beta | integer | Specify the PIE beta parameter for this limiter queue. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `pie`. Defaults to `1250`. (optional) |
| param_pie_max_burst | integer | Specify the PIE max burst parameter for this limiter queue. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `pie`. Defaults to `150000`. (optional) |
| param_pie_max_ecnth | integer | Specify the PIE max ecnth parameter for this limiter queue. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `pie`. Defaults to `99`. (optional) |
| param_red_w_q | integer | Specify the RED w_q parameter for this limiter queue. This must be a numeric value of 1 or greater. Only available when the `aqm` is set to `red`. Defaults to `1`. (optional) |
| param_red_min_th | integer | Specify the RED min_th parameter for this limiter queue. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `red`. Defaults to `0`. (optional) |
| param_red_max_th | integer | Specify the RED max_th parameter for this limiter queue. This must be a numeric value of 1 or greater. Only available when the `aqm` is set to `red`. Defaults to `1`. (optional) |
| param_red_max_p | integer | Specify the RED max_p parameter for this limiter queue. This must be a numeric value of 1 or greater. Only available when the `aqm` is set to `red`. Defaults to `1`. (optional) |
| param_gred_w_q | integer | Specify the GRED w_q parameter for this limiter queue. This must be a numeric value of 1 or greater. Only available when the `aqm` is set to `gred`. Defaults to `1`. (optional) |
| param_gred_min_th | integer | Specify the GRED min_th parameter for this limiter queue. This must be a numeric value of 0 or greater. Only available when the `aqm` is set to `gred`. Defaults to `0`. (optional) |
| param_gred_max_th | integer | Specify the GRED max_th parameter for this limiter queue. This must be a numeric value of 1 or greater. Only available when the `aqm` is set to `gred`. Defaults to `1`. (optional) |
| param_gred_max_p | integer | Specify the GRED max_p parameter for this limiter queue. This must be a numeric value of 1 or greater. Only available when the `aqm` is set to `gred`. Defaults to `1`. (optional) |
| apply | boolean | Specify whether or not this limiter queue should be applied immediately after creation. If set to `true`, the firewall filter will reload and the limiter queue will be applied. If set to`false`, the firewall filter will not be reloaded and the limiter queue will not be applied until the filter Is reloaded. Defaults to `false`. (optional) |



***Body:***

```js        
{
    "limiter": "Test_Limiter",
    "name": "Test_Queue",
    "mask": "srcaddress",
    "maskbits": 31,
    "description": "Unit test",
    "aqm": "codel",
    "weight": 1,
    "plr": 0.01,
    "buckets": 16,
    "apply": true
}
```



### 2. Delete Limiter Queue


Delete a queue from an existing traffic shaper limiter.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-trafficshaper-limiter`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/traffic_shaper/limiter/queue
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| limiter | string | Specify the name of the parent limiter to delete this queue from. |
| name | string | Specify the name of the queue to delete.  |
| apply | boolean | Specify whether or not this limiter queue removal should be applied immediately after deletion. If set to `true`, the firewall filter will reload and the limiter queue will be deleted forever. If set to`false`, the firewall filter will not be reloaded and the limiter queue will not be removed until the filter is reloaded. Defaults to `false`. (optional) |



***Body:***

```js        
{
    "limiter": "Test_Limiter",
    "name": "Test_Queue"
}
```



## FIREWALL/TRAFFIC_SHAPER/QUEUE



### 1. Create Traffic Shaper Queue


Add a queue to an traffic shaper interface.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-trafficshaper-queues`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/traffic_shaper/queue
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| interface | string | Specify the parent interface to create the traffic shaper queue for. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0).  |
| name | string | Specify a name for this queue. This name must be 15 characters or less, and unique from all other queues on the parent interface. |
| priority | integer | Specify the priority of this queue. This field Is only available when the parent traffic shaper's `scheduler` Is set to  |
| description | string | Specify a description for this queue. (optional) |
| enabled | boolean | Enable or disable this traffic queue upon creation. Specify `true` to enable or `false` to disable. Defaults to `true`. (optional) |
| default | boolean | Specify whether or not this queue should be set as the default queue for the parent. Set to `true` to enable. Defaults to `false`. (optional) |
| red | boolean | Set the Random Early Detection option for this queue. Set `true` to enable. Defaults to `false`. (optional) |
| rio | boolean | Set the Random Early Detection In and Out option for this queue. Set `true` to enable. Defaults to `false`. (optional) |
| ecn | boolean | Set the Explicit Congestion Notification option for this queue. Set `true` to enable. Defaults to `false`. (optional)  |
| codel | boolean | Set the Codel Active Queue option for this queue. Set `true` to enable. Defaults to `false`. (optional) |
| qlimit | integer | Specify the queue limit value for this queue. If set, this must be a value of `1` or greater. Defaults to no limit. (optional) |
| bandwidthtype | string | Specify the bandwidth type to use when setting the bandwidth. Choices are `%` for percentage based bandwidth, `b` for bits, `Kb` for kilobits, `Mb` for megabits, and `Gb` for gigabits. Only available when the parent traffic shaper's `scheduler` Is set to `HFSC`, `CBQ`, or `FAIRQ`.  (optional) |
| bandwidth | integer | Specify the bandwidth of this traffic shaper. This must be a numeric value of `1` or greater. If you have set the `bandwidthtype` to `%`, this value must be `100` or less. Only available when the parent traffic shaper's `scheduler` Is set to `HFSC`, `CBQ`, or `FAIRQ`. The sum of this value and all other child queues must be less than the parent traffic shaper's `bandwidth` value. (optional) |
| buckets | string | Set the bandwidth buckets value for this queue. Only available when the parent traffic shaper's `scheduler` Is set to `FAIRQ`. (optional)  |
| hogs | string | Set the bandwidth hogs value for this queue. Only available when the parent traffic shaper's `scheduler` Is set to `FAIRQ`. (optional) |
| borrow | boolean | Allow queue to borrow bandwidth from other queues when available. Only available when the parent traffic shaper's `scheduler` Is set to `CBQ`. (optional) |
| upperlimit | boolean | Enable the upperlimit (max bandwidth) option for this queue. Set `true` to enable. Defaults to `false`. Only available when parent traffic shaper's `scheduler` is set to `HFSC`. (optional) |
| upperlimit1 | string | Set the first upper limit value (m1). This value must be a bandwidth formatted string (e.g. `100Mb`, `1Gb`, etc.). Only available when the parent traffic shaper's `scheduler` is set to `HFSC` and the `upperlimit` value is enabled. (optional) |
| upperlimit2 | integer | Set the second upper limit value (d). This value must an integer of 1 or greater. This value will required If an `upperlimit1` value was specified. Only available when the parent traffic shaper's `scheduler` is set to `HFSC` and the `upperlimit` value is enabled. (optional) |
| upperlimit3 | string | Set the third upper limit value (m2). This value must be a bandwidth formatted string (e.g. `100Mb`, `1Gb`, etc.). Only available when the parent traffic shaper's `scheduler` is set to `HFSC` and the `upperlimit` value is enabled. This value Is required if the`upperlimit` value is enabled. |
| linkshare | boolean | Enable the linkshare (shared bandwidth) option for this queue. Set `true` to enable. Defaults to `false`. Only available when parent traffic shaper's `scheduler` is set to `HFSC`. (optional) |
| linkshare1 | string | Set the first linkshare value (m1). This value must be a bandwidth formatted string (e.g. `100Mb`, `1Gb`, etc.). Only available when the parent traffic shaper's `scheduler` is set to `HFSC` and the `linkshare` value is enabled. (optional) |
| linkshare2 | integer | Set the second linkshare value (d). This value must an integer of 1 or greater. This value will required If an `linkshare1` value was specified. Only available when the parent traffic shaper's `scheduler` is set to `HFSC` and the `linkshare` value is enabled. (optional) |
| linkshare3 | string | Set the third linkshare value (m2). This value must be a bandwidth formatted string (e.g. `100Mb`, `1Gb`, etc.). Only available when the parent traffic shaper's `scheduler` is set to `HFSC` and the `linkshare` value is enabled. This value Is required if the`linkshare` value is enabled. |
| realtime | boolean | Enable the realtime (minimum bandwidth) option for this queue. Set `true` to enable. Defaults to `false`. Only available when parent traffic shaper's `scheduler` is set to `HFSC`. (optional) |
| realtime1 | string | Set the first realtime value (m1). This value must be a bandwidth formatted string (e.g. `100Mb`, `1Gb`, etc.). Only available when the parent traffic shaper's `scheduler` is set to `HFSC` and the `realtime` value is enabled. (optional) |
| realtime2 | integer | Set the second realtime value (d). This value must an integer of 1 or greater. This value will required If an `realtime1` value was specified. Only available when the parent traffic shaper's `scheduler` is set to `HFSC` and the `realtime` value is enabled. (optional) |
| realtime3 | string | Set the third realtime value (m2). This value must be a bandwidth formatted string (e.g. `100Mb`, `1Gb`, etc.). Only available when the parent traffic shaper's `scheduler` is set to `HFSC` and the `realtime` value is enabled. This value Is required if the`linkshare` value is enabled. |
| apply | integer | Specify whether this traffic shaper should be applied immediately after creation. If `true`, the filter will be reloaded after creation. Otherwise, if `false`, the filter will not be reloaded and the change will not be applied until the next filter reload. Defaults to `false`. (optional) |



***Body:***

```js        
{
    "interface": "lan",
    "scheduler": "PRIQ",
    "bandwidthtype": "Gb",
    "bandwidth": 1,
    "enabled": false,
    "qlimit": 1000,
    "tbrconfig": 1000,
    "apply": true
}
```



### 2. Delete Traffic Shaper Queue


Delete a queue from a traffic shaper interface.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-trafficshaper-queues`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/traffic_shaper/queue
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| interface | string | Specify the parent interface to delete the traffic shaper queue from. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0).  |
| name | string | Specify a name of the queue to delete. This must be an existing queue name configured on the parent traffic shaper. |



***Body:***

```js        
{
    "interface": "lan",
    "scheduler": "PRIQ",
    "bandwidthtype": "Gb",
    "bandwidth": 1,
    "enabled": false,
    "qlimit": 1000,
    "tbrconfig": 1000,
    "apply": true
}
```



## FIREWALL/VIRTUAL_IP



### 1. Create Virtual IPs


Add a new virtual IP.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-virtualipaddress-edit`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/virtual_ip
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| mode | string | Set our virtual IP mode (`ipalias`, `carp`, `proxyarp`, `other`) |
| interface | string | Set the interface that will listen for the virtual IP. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0) |
| subnet | string | Set the IP or subnet (CIDR) for the virtual IP(s) |
| descr | string | Set a description for the new virtual IP (optional) |
| noexpand | boolean | Disable IP expansion for the virtual IP address. This is only available on `proxyarp` and `other` mode virtual IPs (optional) |
| vhid | string or integer | Set the VHID for the new CARP virtual IP. Only available for `carp` mode. Leave unspecified to automatically detect the next available VHID. (optional) |
| advbase | string or integer | Set an advertisement base for the new CARP virtual IP. Only available for `carp` mode. Leave unspecified to assume default value of `1`. (optional) |
| advskew | string or integer | Set an advertisement skew for the new CARP virtual IP. Only available for `carp` mode. Leave unspecified to assume default value of `0`. (optional) |
| password | string | Set a password for the new CARP virtual IP. Required for `carp` mode. |



***Body:***

```js        
{
	"mode": "carp",
	"interface": "em0",
	"subnet": "172.16.77.239/32",
	"password": "testpass",
	"descr": "This is a virtual IP added via pfSense API"
}
```



### 2. Delete Virtual IPs


Delete an existing virtual IP.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-virtualipaddress-edit`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/virtual_ip
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | string or integer | Specify the virtual IP ID to delete |



***Body:***

```js        
{
	"id": 0
}
```



### 3. Read Virtual IPs


Read virtual IP assignments.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-virtualipaddresses`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/virtual_ip
```



***Body:***

```js        
{

}
```



### 4. Update Virtual IPs


Update an existing virtual IP.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-firewall-virtualipaddress-edit`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/firewall/virtual_ip
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | integer | Specify the virtual IP ID to update |
| mode | string | Update the virtual IP mode (`ipalias`, `carp`, `proxyarp`, `other`) (optional) |
| interface | string | Update the interface that will listen for the virtual IP. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0) |
| subnet | string | Set the IP or subnet (CIDR) for the virtual IP(s) (optional) |
| descr | string | Update a description for the new virtual IP (optional) |
| noexpand | boolean | Disable IP expansion for the virtual IP address. This is only available on `proxyarp` and `other` mode virtual IPs (optional) |
| vhid | string or integer | Update the VHID for the new CARP virtual IP. Only available for `carp` mode. Leave unspecified to automatically detect the next available VHID. (optional) |
| advbase | string or integer | Update the advertisement base for the new CARP virtual IP. Only available for `carp` mode. Leave unspecified to assume default value of `1`. (optional) |
| advskew | string or integer | Update the advertisement skew for the new CARP virtual IP. Only available for `carp` mode. Leave unspecified to assume default value of `0`. (optional) |
| password | string | Update the password for the new CARP virtual IP. Required If updating to `carp` mode. |



***Body:***

```js        
{
    "id": 0,
	"mode": "carp",
	"interface": "em1",
	"subnet": "172.16.77.252/32",
	"password": "testpass",
	"descr": "Updated API descr",
    "advskew": 5
}
```



## INTERFACE
API endpoints that create, read, update and delete network interfaces.



### 1. Create Interfaces


Add a new interface.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-interfaces-assignnetworkports`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/interface
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| if | string | Specify the physical interface to configure |
| descr | string | Set a descriptive name for the new interface |
| enable | boolean | Enable the interface upon creation. Do not specify this value to leave disabled. |
| spoofmac | string | Set a custom MAC address to the interface (optional) |
| mtu | string or integer | Set a custom MTU for this interface (optional) |
| mss | string or integer | Set a custom MSS for the this interface (optional) |
| media | string | Set a custom speed/duplex setting for this interface (optional) |
| type | string | Set the interface IPv4  configuration type (optional) |
| type6 | string | Set the interface IPv6  configuration type (optional) |
| ipaddr | string | Set the interface's static IPv4 address (required if `type` is set to `staticv4`) |
| subnet | string or integer | Set the interface's static IPv4 address's subnet bitmask (required if `type` is set to `staticv4`) |
| gateway | string | Set the interface network's upstream gateway. This is only necessary on WAN/UPLINK interfaces (optional) |
| ipaddrv6 | string | Set the interface's static IPv6 address (required if `type6` is set to `staticv6`) |
| subnetv6 | string or integer | Set the interface's static IPv6 address's subnet bitmask (required if `type6` is set to `staticv6`) |
| gatewayv6 | string | Set the interface network's upstream IPv6 gateway. This is only necessary on WAN/UPLINK interfaces (optional) |
| ipv6usev4iface | boolean | Allow IPv6 to use IPv4 uplink connection (optional) |
| dhcphostname | string | Set the IPv4 DHCP hostname. Only available when `type` is set to `dhcp` (optional) |
| dhcprejectfrom | string or array | Set the IPv4 DHCP rejected servers. You may pass values in as array or as comma separated string. Only available when `type` is set to `dhcp` (optional) |
| alias-address | string | Set the IPv4 DHCP address alias. The value in this field is used as a fixed alias IPv4 address by the DHCP client (optional) |
| alias-subnet | string or integer | Set the IPv4 DHCP address aliases subnet (optional) |
| adv_dhcp_pt_timeout | string or integer | Set the IPv4 DHCP protocol timeout interval. Must be numeric value greater than 1 (optional) |
| adv_dhcp_pt_retry | string or integer | Set the IPv4 DHCP protocol retry interval. Must be numeric value greater than 1 (optional) |
| adv_dhcp_pt_select_timeout | string or integer | Set the IPv4 DHCP protocol select timeout interval. Must be numeric value greater than 0 (optional) |
| adv_dhcp_pt_reboot | string or integer | Set the IPv4 DHCP protocol reboot interval. Must be numeric value greater than 1 (optional) |
| adv_dhcp_pt_backoff_cutoff | string or integer | Set the IPv4 DHCP protocol backoff cutoff interval. Must be numeric value greater than 1 (optional) |
| adv_dhcp_pt_initial_interval | string or integer | Set the IPv4 DHCP protocol initial interval. Must be numeric value greater than 1 (optional) |
| adv_dhcp_config_advanced | boolean | Enable the IPv4 DHCP advanced configuration options. This enables the DHCP options listed below (optional) |
| adv_dhcp_send_options | string | Set a custom IPv4 `send` option (optional) |
| adv_dhcp_request_options | string | Set a custom IPv4 `request` option (optional) |
| adv_dhcp_request_options | string | Set a custom IPv4 `required` option (optional) |
| adv_dhcp_option_modifiers | string | Set a custom IPv4 optional modifier (optional) |
| adv_dhcp_config_file_override | boolean | Enable local DHCP configuration file override (optional) |
| adv_dhcp_config_file_override_file | string | Set the custom DHCP configuration file's absolute path. This file must exist beforehand (optional) |
| dhcpvlanenable | boolean | Enable DHCP VLAN prioritization (optional) |
| dhcpcvpt | string or integer | Set the DHCP VLAN priority. Must be a numeric value between 1 and 7 (optional) |
| gateway-6rd | string | Set the 6RD interface IPv4 gateway address. Only required when `type6` is set to `6rd` |
| prefix-6rd | string | Set the 6RD IPv6 prefix assigned by the ISP. Only required when `type6` is set to `6rd` |
| prefix-6rd-v4plen | integer or string | Set the 6RD IPv4 prefix length. This is typically assigned by the ISP. Only available when `type6` is set to `6rd` |
| track6-interface | string | Set the Track6 dynamic IPv6 interface. This must be a dynamically configured IPv6 interface. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0). Only required with `type6` is set to `track6` |
| track6-prefix-id-hex | string or integer | Set the IPv6 prefix ID. The value in this field is the (Delegated) IPv6 prefix ID. This determines the configurable network ID based on the dynamic IPv6 connection. The default value is 0. Only available when `type6` is set to `track6` |
| apply | boolean | Specify whether or not you would like this interface to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are creating multiple interfaces at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/interface/apply` endpoint. Otherwise, If you are only creating a single interface, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
	"if": "em1.3",
	"descr": "asdf",
	"enable": "",
	"type": "staticv4",
	"ipaddr": "10.250.0.1",
	"subnet": "24",
	"blockbogons": true
}
```



### 2. Delete Interfaces


Delete an existing interface. __Note: interface deletions will be applied immediately, there is no need to apply interface changes afterwards__<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-interfaces-assignnetworkports`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/interface
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| if | string | Specify the interface to delete. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0) |



***Body:***

```js        
{
	"if": "em1.3"
}
```



### 3. Read Interfaces


Read interface assignments and configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-interfaces-assignnetworkports`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/interface
```



***Body:***

```js        
{
}
```



### 4. Update Interfaces


Update an existing interface.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-interfaces-assignnetworkports`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/interface
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | string | Specify the Interface to update. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0) |
| if | string | Update the physical interface configured (optional) |
| descr | string | Update the descriptive name of the interface (optional) |
| enable | boolean | Enable or disable the Interface (optional) |
| spoofmac | string | Update the MAC address of the interface (optional) |
| mtu | string or integer | Update the MTU for this interface (optional) |
| mss | string or integer | Update the MSS for the this interface (optional) |
| media | string | Update the speed/duplex setting for this interface (optional) |
| type | string | Update the interface IPv4  configuration type (optional) |
| type6 | string | Update the interface IPv6  configuration type (optional) |
| ipaddr | string | Update the interface's static IPv4 address (required if `type` is set to `staticv4`) |
| subnet | string or integer | Update the interface's static IPv4 address's subnet bitmask (required if `type` is set to `staticv4`) |
| gateway | string | Update the interface network's upstream gateway. This is only necessary on WAN/UPLINK interfaces (optional) |
| ipaddrv6 | string | Update the interface's static IPv6 address (required if `type6` is set to `staticv6`) |
| subnetv6 | string or integer | Update the interface's static IPv6 address's subnet bitmask (required if `type6` is set to `staticv6`) |
| gatewayv6 | string | Update the interface network's upstream IPv6 gateway. This is only necessary on WAN/UPLINK interfaces (optional) |
| ipv6usev4iface | boolean | Enable or disable IPv6 over IPv4 uplink connection (optional) |
| dhcphostname | string | Update the IPv4 DHCP hostname. Only available when `type` is set to `dhcp` (optional) |
| dhcprejectfrom | string or array | Update the IPv4 DHCP rejected servers. You may pass values in as array or as comma separated string. Only available when `type` is set to `dhcp` (optional) |
| alias-address | string | Update the IPv4 DHCP address alias. The value in this field is used as a fixed alias IPv4 address by the DHCP client (optional) |
| alias-subnet | string or integer | Update the IPv4 DHCP address aliases subnet (optional) |
| adv_dhcp_pt_timeout | string or integer | Update the IPv4 DHCP protocol timeout interval. Must be numeric value greater than 1 (optional) |
| adv_dhcp_pt_retry | string or integer | Update the IPv4 DHCP protocol retry interval. Must be numeric value greater than 1 (optional) |
| adv_dhcp_pt_select_timeout | string or integer | Update the IPv4 DHCP protocol select timeout interval. Must be numeric value greater than 0 (optional) |
| adv_dhcp_pt_reboot | string or integer | Update the IPv4 DHCP protocol reboot interval. Must be numeric value greater than 1 (optional) |
| adv_dhcp_pt_backoff_cutoff | string or integer | Update the IPv4 DHCP protocol backoff cutoff interval. Must be numeric value greater than 1 (optional) |
| adv_dhcp_pt_initial_interval | string or integer | Update the IPv4 DHCP protocol initial interval. Must be numeric value greater than 1 (optional) |
| adv_dhcp_config_advanced | boolean | Enable or disable the IPv4 DHCP advanced configuration options. This enables the DHCP options listed below (optional) |
| adv_dhcp_send_options | string | Update the IPv4 `send` option (optional) |
| adv_dhcp_request_options | string | Update the IPv4 `request` option (optional) |
| adv_dhcp_request_options | string | Update the IPv4 `required` option (optional) |
| adv_dhcp_option_modifiers | string | Update the IPv4 option modifier (optional) |
| adv_dhcp_config_file_override | boolean | Enable or disable local DHCP configuration file override (optional) |
| adv_dhcp_config_file_override_file | string | Update the custom DHCP configuration file's absolute path. This file must exist beforehand (optional) |
| dhcpvlanenable | boolean | Enable or disable DHCP VLAN prioritization (optional) |
| dhcpcvpt | string or integer | Update the DHCP VLAN priority. Must be a numeric value between 1 and 7 (optional) |
| gateway-6rd | string | Update the 6RD interface IPv4 gateway address. Only required when `type6` is set to `6rd` |
| prefix-6rd | string | Update the 6RD IPv6 prefix assigned by the ISP. Only required when `type6` is set to `6rd` |
| prefix-6rd-v4plen | integer or string | Update the 6RD IPv4 prefix length. This is typically assigned by the ISP. Only available when `type6` is set to `6rd` |
| track6-interface | string | Update the Track6 dynamic IPv6 interface. This must be a dynamically configured IPv6 interface. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0). Only required with `type6` is set to `track6` |
| track6-prefix-id-hex | string or integer | Update the IPv6 prefix ID. The value in this field is the (Delegated) IPv6 prefix ID. This determines the configurable network ID based on the dynamic IPv6 connection. The default value is 0. Only available when `type6` is set to `track6` |
| apply | boolean | Specify whether or not you would like this interface update to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are updating multiple interfaces at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/interface/apply` endpoint. Otherwise, If you are only updating a single interface, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
	"if": "em1.3",
	"descr": "asdf",
	"enable": "",
	"type": "staticv4",
	"ipaddr": "10.250.0.1",
	"subnet": "24",
	"blockbogons": true
}
```



## INTERFACE/APPLY



### 1. Apply Interfaces


Apply pending interface updates. This will apply the current configuration for each interface. This endpoint returns no data.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-interfaces-assignnetworkports`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/interface/apply
```



***Body:***

```js        
{
}
```



## INTERFACE/VLAN



### 1. Create Interface VLANs


Add a new VLAN interface.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-interfaces-vlan-edit`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/interface/vlan
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| if | string | Set the parent interface to add the new VLAN to |
| tag | string or integer | Set the VLAN tag to add to the parent interface |
| pcp | string or integer | Set a 802.1Q VLAN priority. Must be an integer value between 0 and 7 (optional) |
| descr | string | Set a description for the new VLAN interface |



***Body:***

```js        
{
	"if": "em1",
	"tag": "3",
	"descr": "New VLAN"
}
```



### 2. Delete Interface VLANs


Delete VLAN assignment and configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-interfaces-vlan-edit`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/interface/vlan
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| vlanif | string | Delete VLAN by full interface name (e.g. `em0.2`).  |
| id | string or integer | Delete VLAN by pfSense ID. Required if `vlanif` was not specified previously. If `vlanif` is specified, this value will be overwritten. |



***Body:***

```js        
{
	"vlanif": "em1.3"
}
```



### 3. Read Interface VLANs


Read VLAN assignments and configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-interfaces-vlan`, `page-interfaces-vlan-edit`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/interface/vlan
```



***Body:***

```js        
{
}
```



### 4. Update Interface VLANs


Modify an existing VLAN configuration

_Note: Unlike the pfSense webConfigurator, you CAN modify an existing VLAN (including parent interface and VLAN tag) while the VLAN is assigned to an active interface. When doing so, please expect up to 5 seconds of downtime to the associated VLAN while the backend configuration is changed_<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-interfaces-vlan-edit`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/interface/vlan
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| vlanif | string | Select VLAN to modify by full interface ID (e.g. `em0.2`) |
| id | string or integer | Select VLAN to modify by pfSense ID. Required if `vlanif` was not specified previously. If `vlanif` is specified, this value will be overwritten. |
| if | string | Set the parent interface to add the new VLAN to |
| tag | string or integer | Set the VLAN tag to add to the parent interface |
| pcp | string or integer | Set a 802.1Q VLAN priority. Must be an integer value between 0 and 7 (optional) |
| descr | string | Set a description for the new VLAN interface |



***Body:***

```js        
{
	"vlanif": "em1.3",
	"tag": 770,
	"pcp": 3,
	"descr": "Test description"
}
```



## ROUTING/APPLY



### 1. Apply Routing


Apply pending routing changes. This endpoint returns no data.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-gateways`, `page-system-gateways-editgateway`, `page-system-staticroutes`, `page-system-staticroutes-editroute`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/routing/apply
```



***Body:***

```js        
{
}
```



## ROUTING/GATEWAY



### 1. Create Routing Gateways


Create new routing gateways.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-gateways-editgateway`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/routing/gateway
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| interface | string | Set which interface the gateway will apply to. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0).  |
| ipprotocol | string | Set what IP protocol this gateway will serve. Options are `inet` for IPv4, or `inet6` for IPv6. |
| name | string | Set a descriptive name for this gateway. This name must be unique, and can only contains alphanumeric characters and underscores. |
| gateway | string | Set the IP address of the gateway. This value must be a valid IPv4 address If you have set your `ipprotocol` value to `inet`, or it must be a valid IPv6 address if you have set your `ipprotocol` value to `inet6`. Unlike the pfSense webConfigurator, this address Is not restricted to an address within the interfaces subnet by default. |
| monitor | string | Set the IP address used to monitor this gateway. This is usually only necessary if the `gateway` IP does not accept ICMP probes. Defaults to the `gateway` address. (optional) |
| disabled | boolean | Disable this gateway upon creation. Defaults to false. (optional) |
| monitor_disable | boolean | Disable gateway monitoring for this gateway. Defaults to false. (optional) |
| action_disable | boolean | Disable any action taken on gateway events for this gateway. This will consider the gateway always up. Defaults to false. (optional) |
| force_down | boolean | Force this gateway to always be considered down. Defaults to false. (optional) |
| descr | string | Set a description for this gateway (optional) |
| weight | integer | Set this gateways weight when utilizing gateway load balancing within a gateway group. This value must be between 1 and 30. Defaults to 1. (optional) |
| data_payload | integer | Set a data payload to send on ICMP packets sent to the gateway monitor IP. This value must be a positive integer. Defaults to 1. (optional) |
| latencylow | integer | Set the low threshold in milliseconds for latency. Any packet that exceeds this threshold will be trigger a minor latency gateway event. This value must be a positive integer that is less than the `latencyhigh` value. Defaults to 200. (optional) |
| latencyhigh | integer | Set the high threshold in milliseconds for latency. Any packet that exceeds this threshold will be trigger a major latency gateway event. This value must be a positive integer that is greater than the `latencylow` value. Defaults to 500. (optional) |
| losslow | integer | Set the low threshold for packet loss In %. If total packet loss exceeds this percentage, a minor packet loss gateway event will be triggered. This value must be greater or equal to 1 and be less than the `losshigh` value. Defaults to 10. (optional) |
| losshigh | integer | Set the high threshold for packet loss In %. If total packet loss exceeds this percentage, a major packet loss gateway event will be triggered. This value must be greater than the `losslow` value and be less or equal to 100. Defaults to 20. (optional) |
| interval | integer | Set how often gateway monitor ICMP probes will be sent in milliseconds. This value must be greater than or equal to 1 and less than or equal to 3600000. Defaults to 500. (optional) |
| loss_interval | integer | Set how long the gateway monitor will wait (in milliseconds) for response packets before considering the packet lost. This value must be greater than or equal to the `latencyhigh` value. Defaults to 2000. (optional) |
| time_period | integer | Set the time period In milliseconds for gateway monitor metrics to be averaged. This value must be greater than twice the probe interval plus the loss interval. Defaults to 60000. (optional) |
| alert_interval | integer | Set the time interval in milliseconds which alert conditions will be checked. This value must be greater than or equal to the `interval` value. Defaults to 1000. (optional) |
| apply | boolean | Specify whether or not you would like this gateway to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are creating multiple gateways at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/routing/apply` endpoint. Otherwise, If you are only creating a single gateway, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
    "interface": "wan",
    "name": "TEST_GATEWAY",
    "ipprotocol": "inet",
    "gateway": "172.16.209.1",
    "monitor": "172.16.209.250",
    "descr": "Test gateway"
}
```



### 2. Delete Routing Gateways


Delete existing routing gateways. __Note: gateway deletions will be applied immediately, there is no need to apply routing changes afterwards__<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-gateways-editgateway`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/routing/gateway
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | integer | Specify the ID of the gateway to delete. _Note: If you are obtaining the ID via GET request, the ID will be included in the response within the `attribute` field of the object._ |



***Body:***

```js        
{

}
```



### 3. Read Routing Gateways


Read routing gateways.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-gateways`]

_Note: Currently, GET requests to this endpoint return verbose backend gateway information rather than the gateway objects as they appear in the configuration. This discrepancy makes it difficult to interact with gateway objects via API. Because of this, GET requests to this endpoint will be refactored in a future release._


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/routing/gateway
```



***Body:***

```js        
{

}
```



### 4. Update Routing Gateways


Update existing routing gateways.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-gateways-editgateway`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/routing/gateway
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | integer | Specify the ID of the gateway to update. _Note: If you are obtaining the ID via GET request, the ID will be included in the response within the `attribute` field of the object._ |
| interface | string | Update the interface the gateway will apply to. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0). (optional) |
| ipprotocol | string | Update the IP protocol this gateway will serve. Options are `inet` for IPv4, or `inet6` for IPv6. If you are changing the protocol, you will also be required to update the `gateway` and/or `monitor` values to match the specified protocol. (optional) |
| name | string | Update the descriptive name for this gateway. This name must be unique, and can only contain alphanumeric characters and underscores. (optional) |
| gateway | string | Update the IP address of the gateway. This value must be a valid IPv4 address If you have set your `ipprotocol` value to `inet`, or it must be a valid IPv6 address if you have set your `ipprotocol` value to `inet6`. Unlike the pfSense webConfigurator, this address Is not restricted to an address within the interfaces subnet by default. (optional) |
| monitor | string | Set the IP address used to monitor this gateway. This is usually only necessary if the `gateway` IP does not accept ICMP probes. Defaults to the `gateway` address. (optional) |
| disabled | boolean | Enable or disable this gateway upon update. True to disable, false to enable. (optional) |
| monitor_disable | boolean | Enable or disable gateway monitoring for this gateway. True to disable, false to enable. (optional) |
| action_disable | boolean | Enable or disable any action taken on gateway events for this gateway. If disabled, this will consider the gateway always up. True for disabled, false for enable. (optional) |
| force_down | boolean | Enable or disable forcing this gateway to always be considered down. True for enable, false for disable. (optional) |
| descr | string | Update the description for this gateway (optional) |
| weight | integer | Update this gateways weight when utilizing gateway load balancing within a gateway group. This value must be between 1 and 30. (optional) |
| data_payload | integer | Update the data payload to send on ICMP packets sent to the gateway monitor IP. This value must be a positive integer. (optional) |
| latencylow | integer | Update the low threshold in milliseconds for latency. Any packet that exceeds this threshold will be trigger a minor latency gateway event. This value must be a positive integer that is less than the `latencyhigh` value.  (optional) |
| latencyhigh | integer | Update the high threshold in milliseconds for latency. Any packet that exceeds this threshold will be trigger a major latency gateway event. This value must be a positive integer that is greater than the `latencylow` value. (optional) |
| losslow | integer | Update the low threshold for packet loss In %. If total packet loss exceeds this percentage, a minor packet loss gateway event will be triggered. This value must be greater or equal to 1 and be less than the `losshigh` value. (optional) |
| losshigh | integer | Update the high threshold for packet loss In %. If total packet loss exceeds this percentage, a major packet loss gateway event will be triggered. This value must be greater than the `losslow` value and be less or equal to 100. (optional) |
| interval | integer | Update how often gateway monitor ICMP probes will be sent in milliseconds. This value must be greater than or equal to 1 and less than or equal to 3600000. (optional) |
| loss_interval | integer | Update how long the gateway monitor will wait (in milliseconds) for response packets before considering the packet lost. This value must be greater than or equal to the `latencyhigh` value. (optional) |
| time_period | integer | Update the time period In milliseconds for gateway monitor metrics to be averaged. This value must be greater than twice the probe interval plus the loss interval. (optional) |
| alert_interval | integer | Update the time interval in milliseconds which alert conditions will be checked. This value must be greater than or equal to the `interval` value. (optional) |
| apply | boolean | Specify whether or not you would like these gateway updates to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are updating multiple gateways at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/routing/apply` endpoint. Otherwise, If you are only updating a single gateway, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
    "id": 0,
    "name": "UPDATED_TEST_GATEWAY",
    "ipprotocol": "inet6",
    "gateway": "2001:0db8:85a3:0000:0000:8a2e:0370:7334",
    "monitor": "2001:0db8:85a3:0000:0000:8a2e:0370:7334",
    "descr": "Updated Unit Test",
    "disabled": true,
    "action_disable": true,
    "monitor_disable": true,
    "weight": 2,
    "data_payload": 5,
    "latencylow": 300,
    "latencyhigh": 600,
    "interval": 2100,
    "loss_interval": 2500,
    "action_interval": 1040,
    "time_period": 66000,
    "losslow": 5,
    "losshigh": 10
}
```



## ROUTING/STATIC_ROUTE



### 1. Create Static Routes


Create a new static route.<br>
_Note: Unlike the webConfigurator, this endpoint will allow you to override existing routes. There are practical use cases for this functionality and it is important to use caution when adding routes_<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-staticroutes-editroute`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/routing/static_route
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| network | string | Specify an IPv4 CIDR, IPv6 CIDR or network alias this route will apply to |
| gateway | string | Specify the name of the gateway traffic matching this route will use |
| descr | string | Leave a description of this route (optional) |
| disabled | boolean | Disable this route upon creation (optional) |
| apply | boolean | Specify whether or not you would like this route to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are creating multiple routes at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/routing/apply` endpoint. Otherwise, If you are only creating a single route, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
    "network": "10.1.1.1/24",
    "gateway": "WAN_DHCP"

}
```



### 2. Delete Static Routes


Delete existing static routes. __Note: route deletions will be applied immediately, there is no need to apply routing changes afterwards__<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-staticroutes-editroute`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/routing/static_route
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | Integer | Specify the ID of the static route to delete |



***Body:***

```js        
{
    "id": 0
}
```



### 3. Read Static Routes


Read existing static routes.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-staticroutes`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/routing/static_route
```



***Body:***

```js        
{

}
```



### 4. Update Static Routes


Update an existing static route.<br>
_Note: Unlike the webConfigurator, this endpoint will allow you to override existing routes. There are practical use cases for this functionality and it is important to use caution when adding routes_<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-staticroutes-editroute`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/routing/static_route
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | Integer | Specify the ID of the static route to update |
| network | string | Update the IPv4 CIDR, IPv6 CIDR or network alias this route will apply to (optional) |
| gateway | string | Update the name of the gateway traffic matching this route will use (optional) |
| descr | string | Update description of this route (optional) |
| disabled | boolean | Disable this route (optional) |
| apply | boolean | Specify whether or not you would like this route update to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are updating multiple routes at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/routing/apply` endpoint. Otherwise, If you are only updating a single route, you may set this true to apply it immediately. Please note, this will default to true for routing updates, if this is set to false when updating routes, the existing route will be removed from the routing table until the changes are applied! Defaults to true. (optional) |



***Body:***

```js        
{
    "id": 0,
    "network": "10.1.2.0/24",
    "gateway": "WAN_DHCP",
    "descr": "New Description"

}
```



## SERVICES
API endpoints that create, read, update, and delete system services.



### 1. Read Services


Read available services.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/services
```



***Body:***

```js        
{
    
}
```



### 2. Restart All Services


Restart all available services.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/restart
```



***Body:***

```js        
{
    
}
```



### 3. Start All Services


Start all available services.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/start
```



***Body:***

```js        
{
    
}
```



### 4. Stop All Services


Stop all available services.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/stop
```



***Body:***

```js        
{
    
}
```



## SERVICES/DDNS



### 1. Read Dynamic DNS


Read configured dynamic DNS settings and statuses.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-services-dynamicdnsclients`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/services/ddns
```



***Body:***

```js        
{
    
}
```



## SERVICES/DHCPD



### 1. Read DHCPd Service Configuration


Read the current DHCPd configuration.<br>


_Requires at least one of the following privileges:_ [`page-all`, `page-services-dhcpserverpage-services-dhcpserver`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/services/dhcpd
```



***Body:***

```js        
{
    
}
```



### 2. Restart DHCPd Service


Restart the dhcpd service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/dhcpd/restart
```



***Body:***

```js        
{
    
}
```



### 3. Start DHCPd Service


Start the dhcpd service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/dhcpd/start
```



***Body:***

```js        
{
    
}
```



### 4. Stop DHCPd Service


Stop the dhcpd service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/dhcpd/stop
```



***Body:***

```js        
{
    
}
```



### 5. Update DHCPd Service Configuration


Update the current DHCPd configuration.<br>


_Requires at least one of the following privileges:_ [`page-all`, `page-services-dhcpserverpage-services-dhcpserver`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/services/dhcpd
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| interface | string | Specify which interface's DHCP configuration to update. You may specify either the interface's descriptive name, the pfSense ID (wan, lan, optx), or the physical interface id (e.g. igb0). This Interface must host a static IPv4 subnet that has more than one available within the subnet. |
| enable | boolean | Enable or disable the DHCP server for this Interface (optional) |
| range_from | string | Update the DHCP pool's start IPv4 address. This must be an available address within the Interface's subnet and be less than the `range_to` value. (optional) |
| range_to | string | Update the DHCP pool's end IPv4 address. This must be an available address within the Interface's subnet and be greater than the `range_from` value. (optional) |
| dnsserver | string or array | Update the DNS servers to include In DHCP leases. Multiple values may be passed in as an array or single values may be passed in as a string. Each value must be a valid IPv4 address. Alternatively, you may pass In an empty array to revert to the system default. (optional) |
| domain | string | Update the domain name to Include In the DHCP lease. This must be a valid domain name or an empty string to assume the system default (optional) |
| domainsearchlist | string or array | Update the search domains to include In the DHCP lease. You may pass In an array for multiple entries or a string for single entries. Each entry must be a valid domain name. (optional) |
| mac_allow | string or array | Update the list of allowed MAC addresses. You may pass In an array for multiple entries or a string for single entries. Each entry must be a full or partial MAC address. Alternatively, you may specify an empty array to revert to default (optional) |
| mac_deny | string or array | Update the list of denied MAC addresses. You may pass In an array for multiple entries or a string for single entries. Each entry must be a full or partial MAC address. Alternatively, you may specify an empty array to revert to default (optional) |
| gateway | string | Update the gateway to include In DHCP leases. This value must be a valid IPv4 address within the Interface's subnet. Alternatively, you can pass In an empty string to revert to the system default. (optional) |
| ignorebootp | boolean | Update whether or not to ignore BOOTP requests. True to Ignore, false to allow. (optional) |
| denyunknown | boolean | Update whether or not to ignore unknown MAC addresses. If true, you must specify MAC addresses in the `mac_allow` field or add a static DHCP entry to receive DHCP requests. (optional) |



***Body:***

```js        
{
    "interface": "lan",
    "enable": true,
    "ignorebootp": false,
    "denyunknown": false,
    "range_from": "192.168.1.25",
    "range_to": "192.168.1.50",
    "dnsserver": ["1.1.1.1"],
    "gateway": "192.168.1.2",
    "domainsearchlist": ["example.com"],
    "domain": "example.com",
    "mac_allow": ["00:00:00:01:E5:FF", "00:00:00:01:E5"],
    "mac_deny": []
}
```



## SERVICES/DHCPD/LEASE



### 1. Read DHCPd Leases


Read the current DHCPd leases.<br>


_Requires at least one of the following privileges:_ [`page-all`, `page-status-dhcpleases`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/services/dhcpd/lease
```



***Body:***

```js        
{
    
}
```



## SERVICES/DHCPD/STATIC_MAPPING



### 1. Create DHCPd Static Mappings


Create new DHCPd static mappings.<br>


_Requires at least one of the following privileges:_ [`page-all`, `page-services-dhcpserver-editstaticmapping`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/dhcpd/static_mapping
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| interface | string | Specify which interface to create the static mapping |
| mac | string | Specify the full MAC address of the host this static mapping Is for |
| ipaddr | string | Specify the IPv4 address the MAC address will be assigned |
| cid | string | Specify a client identifier (optional) |
| hostname | string | Specify a hostname for this host (optional) |
| domain | string | Specify a domain for this host (optional) |
| descr | string | Specify a description for this mapping (optional) |
| dnsserver | string or array | Specify the DNS servers to assign this client. Multiple values may be passed in as an array or single values may be passed in as a string. Each value must be a valid IPv4 address. Alternatively, you may pass In an empty array to revert to the system default. (optional) |
| domainsearchlist | string or array  | Specify the search domains to assign to this host. You may pass In an array for multiple entries or a string for single entries. Each entry must be a valid domain name. (optional) |
| gateway | string | Specify the gateway to assign this host. This value must be a valid IPv4 address within the Interface's subnet. Alternatively, you can pass In an empty string to revert to the system default. (optional) |
| arp_table_static_entry | boolean | Specify whether or not a static ARP entry should be created for this host (optional) |



***Body:***

```js        
{
    "interface": "lan",
    "mac": "ac:de:48:00:11:22",
    "ipaddr": "192.168.1.254",
    "cid": "a098b-zpe9s-1vr45",
    "descr": "This is a DHCP static mapping created by pfSense API",
    "hostname": "test-host",
    "domain": "example.com",
    "dnsserver": ["1.1.1.1"],
    "domainsearchlist": ["example.com"],
    "gateway": "192.168.1.1",
    "arp_table_static_entry": true
}
```



### 2. Delete DHCPd Static Mappings


Delete existing DHCPd static mappings.<br>


_Requires at least one of the following privileges:_ [`page-all`, `page-services-dhcpserver-editstaticmapping`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/services/dhcpd/static_mapping
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | integer | Specify the ID of the static mapping to delete |
| interface | string | Specify which interface to update the static mapping |



***Body:***

```js        
{
    "id": 0,
    "interface": "lan"
}
```



### 3. Read DHCPd Static Mappings


Read the current DHCPd static mappings.<br>


_Requires at least one of the following privileges:_ [`page-all`, `page-services-dhcpserver-editstaticmapping`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/services/dhcpd/static_mapping
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| interface | string | Specify which interface to read static mappings from |



***Body:***

```js        
{
    "interface": "lan"
}
```



### 4. Update DHCPd Static Mappings


Update existing DHCPd static mappings.<br>


_Requires at least one of the following privileges:_ [`page-all`, `page-services-dhcpserver-editstaticmapping`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/services/dhcpd/static_mapping
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | integer | Specify the ID of the static mapping to update |
| interface | string | Specify which interface to update the static mapping |
| mac | string | Update the full MAC address of the host this static mapping Is for |
| ipaddr | string | Update the IPv4 address the MAC address will be assigned |
| cid | string | Update the client identifier (optional) |
| hostname | string | Update the hostname for this host (optional) |
| domain | string | Update the domain for this host (optional) |
| descr | string | Update the description for this mapping (optional) |
| dnsserver | string or array | Update the DNS servers to assign this client. Multiple values may be passed in as an array or single values may be passed in as a string. Each value must be a valid IPv4 address. Alternatively, you may pass In an empty array to revert to the system default. (optional) |
| domainsearchlist | string or array  | Update the search domains to assign to this host. You may pass In an array for multiple entries or a string for single entries. Each entry must be a valid domain name. (optional) |
| gateway | string | Update the gateway to assign this host. This value must be a valid IPv4 address within the Interface's subnet. Alternatively, you can pass In an empty string to revert to the system default. (optional) |
| arp_table_static_entry | boolean | Update whether or not a static ARP entry should be created for this host (optional) |



***Body:***

```js        
{
    "id": 0,
    "interface": "lan",
    "mac": "ac:de:48:00:11:22",
    "ipaddr": "192.168.1.250",
    "cid": "updated-a098b-zpe9s-1vr45",
    "descr": "This is an updated DHCP static mapping created by pfSense API",
    "hostname": "updated-test-host",
    "domain": "updated.example.com",
    "dnsserver": ["8.8.8.8", "8.8.4.4", "1.1.1.1"],
    "domainsearchlist": ["updated.example.com", "extra.example.com"],
    "gateway": "192.168.1.2",
    "arp_table_static_entry": false
}
```



## SERVICES/DPINGER



### 1. Restart DPINGER Service


Restart the dhcpd service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/dpinger/restart
```



***Body:***

```js        
{
    
}
```



### 2. Start DPINGER Service


Start the dpinger service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/dpinger/start
```



***Body:***

```js        
{
    
}
```



### 3. Stop DPINGER Service


Stop the dpinger service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/dpinger/stop
```



***Body:***

```js        
{
    
}
```



## SERVICES/NTPD



### 1. Read NTPd Service


Read the ntpd service configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-services-ntpd`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/services/ntpd
```



***Body:***

```js        
{
    
}
```



### 2. Restart NTPd Service


Restart the ntpd service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/ntpd/restart
```



***Body:***

```js        
{
    
}
```



### 3. Start NTPd Service


Start the ntpd service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/ntpd/start
```



***Body:***

```js        
{
    
}
```



### 4. Stop NTPd Service


Stop the dpinger service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/ntpd/stop
```



***Body:***

```js        
{
    
}
```



### 5. Update NTPd Service


Update the ntpd service configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-services-ntpd`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/services/ntpd
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| interface | array | Update the Interfaces NTPd will listen on. This must be an array of strings. You may specify either the physical Interface ID, the pfSense Interface ID or the descriptive Interface name. To match any Interface, simply pass In an empty array. e.g. `["wan", "em1", "lo0", "LAN"]` (optional) |
| time_servers | array | Update the time servers used by the system. This must be an array of objects. Each object may use the parameters available In the /api/v1/services/ntpd/time_server endpoint. To revert to the default timeserver, you may pass In an empty array. Specifying this field will overwrite any existing time servers. To simply add or remove time servers, use the /api/v1/services/ntpd/time_server endpoint as it Is less taxing on the system. e.g. `[{"timeserver": "pool.ntp.org", "ispool": true", "prefer": true, "noselect": false}]` (optional) |
| orphan | integer | Update the orphan mode value. This must be a value between `1` and `15`. Orphan mode allows the system clock to be used when no other clocks are available. The number here specifies the stratum reported during orphan mode and should normally be set to a number high enough to insure that any other servers available to clients are preferred over this server. (optional) |
| leapsec | string | Update the leap seconds configuration. This should be the contents of the leap seconds file received from the IERS. (optional) |
| statsgraph | boolean | Enable or disable RRD graphs of NTP metrics. (optional) |
| logpeer | boolean | Enable or disable the logging of peer messages. (optional) |
| logsys | boolean | Enable or disable the logging of system messages. (optional) |
| clockstats | boolean | Enable or disable the logging of reference clock statistics. (optional) |
| loopstats | boolean | Enable or disable the logging of clock discipline statistics. (optional) |
| peerstats | boolean | Enable or disable the logging of NTP peer statistics. (optional) |



***Body:***

```js        
{
    "interface": ["WAN", "lan", "lo0"],
    "orphan": 15,
    "logpeer": true,
    "logsys": true,
    "clockstats": true,
    "loopstats": false,
    "peerstats": true,
    "statsgraph": true,
    "timeservers": [
      {"timeserver": "ntp.pool.org", "ispool": true, "prefer": true, "noselect": false}
    ]
  }
```



## SERVICES/NTPD/TIME_SERVER



### 1. Create NTPd Time Server


Add a new NTP time server to the NTPd configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-services-ntpd`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/ntpd/time_server
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| timeserver | string | Specify the IP or hostname of the NTP time server to add. |
| ispool | boolean | Specify whether or not this time server represents an NTP server pool or a single NTP server.  |
| noselect | boolean | Enable or disable this NTP time server from selection. If `true`, the time server will be eligible for selection. If `false, the NTP server will remain In the configuration but not used. |
| prefer | boolean | Enable or disable this NTP time server as a preferred NTP time server. If `true`, this time server will take preference over other time servers. |



***Body:***

```js        
{
    "timeserver": "ntp.pool.org", 
    "ispool": true, 
    "prefer": true, 
    "noselect": false
}
```



### 2. Delete NTPd Time Server


Delete an existing NTP time server from the NTPd configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-services-ntpd`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/services/ntpd/time_server
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| timeserver | string | Specify the IP or hostname of the NTP time server to delete. If more than one time server exists with this value, only the first match will be deleted. In the case that all time servers were deleted, the default time server `pool.ntp.org` will be written. |



***Body:***

```js        
{
    "timeserver": "ntp.pool.org"
}
```



## SERVICES/SSHD



### 1. Read SSHd Configuration


Read sshd configuration. <br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-advanced-admin`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/services/sshd
```



***Body:***

```js        
{
    
}
```



### 2. Restart SSHd Service


Restart the sshd service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/sshd/restart
```



***Body:***

```js        
{
    
}
```



### 3. Start SSHd Service


Start the sshd service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/sshd/start
```



***Body:***

```js        
{
    
}
```



### 4. Stop SSHd Service


Stop the sshd service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/sshd/stop
```



***Body:***

```js        
{
    
}
```



### 5. Update SSHd Configuration


Modify the sshd service configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-advanced-admin`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/services/sshd
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| enable | boolean | Enable or disable sshd. Do not specify to retain existing value (optional) |
| sshdkeyonly | string | Set the sshd auth type. Use `disabled` for password or pubkey, `enabled` for pubkey only, or `both` to require both a password and pubkey. Do not specify to retain existing value (optional) |
| sshdagentforwarding | boolean | Enable or disable sshd agent forwarding. Do not specify to retain existing value (optional) |
| port | integer or string | Set the port sshd will bind to. Do not specify to retain existing value (optional) |



***Body:***

```js        
{
	"enable": true,
	"sshdkeyonly": "both",
	"sshdagentforwarding": false,
	"port": 2222
}
```



## SERVICES/SYSLOGD



### 1. Restart SYSLOGd Service


Restart the syslogd service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/syslogd/restart
```



***Body:***

```js        
{
    
}
```



### 2. Start SYSLOGd Service


Start the syslogd service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/syslogd/start
```



***Body:***

```js        
{
    
}
```



### 3. Stop SYSLOGd Service


Stop the syslogd service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/syslogd/stop
```



***Body:***

```js        
{
    
}
```



## SERVICES/UNBOUND



### 1. Apply Pending Unbound Changes


Apply pending DNS Resolver (Unbound) changes.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-services-dnsresolver`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/unbound/apply
```



***Body:***

```js        
{

}
```



### 2. Read Unbound Configuration


Read DNS Resolver (Unbound) configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-services-dnsresolver`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/services/unbound
```



***Body:***

```js        
{
    
}
```



### 3. Restart Unbound Service


Restart the DNS Resolver (Unbound) service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/unbound/restart
```



***Body:***

```js        
{
    
}
```



### 4. Start Unbound Service


Start the DNS Resolver (Unbound) service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/unbound/start
```



***Body:***

```js        
{
    
}
```



### 5. Stop Unbound Service


Stop the DNS Resolver (Unbound) service.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-services`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/unbound/stop
```



***Body:***

```js        
{
    
}
```



## SERVICES/UNBOUND/HOST_OVERRIDE



### 1. Create Unbound Host Override


Add a new host override to DNS Resolver (Unbound).<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-services-dnsresolver-edithost`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/unbound/host_override
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| host | string | Hostname of new DNS A record |
| domain | string | Domain of new DNS A record |
| ip | string | IPv4 or IPv6 of new DNS A record |
| descr | string | Description of host override (optional) |
| aliases | array | Hostname aliases (CNAME) for host override (optional) |
| apply | boolean | Specify whether or not you would like this host override to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are creating multiple host overrides at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/services/unbound/apply` endpoint. Otherwise, If you are only creating a single host override, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
	"host": "test",
	"domain": "example.com",
	"ip": "127.0.0.1",
	"descr": "This is a test host override added via pfSense API!",
	"aliases": [
        {
            "host": "test2", 
            "domain": "example.com", 
            "descr": "This is a test host override alias that will also resolve to this IP!"
        }
    ]
}
```



### 2. Delete Unbound Host Override


Delete host overrides from DNS Resolver (Unbound).<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-services-dnsresolver-edithost`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/services/unbound/host_override
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | integer | Specify the ID of the host override to delete |
| apply | boolean | Specify whether or not you would like this host override deletion to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are deleting multiple host overrides at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/services/unbound/apply` endpoint. Otherwise, If you are only deleting a single host override, you may set this to true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
    "id": 0,
    "apply": true
}
```



### 3. Read Unbound Host Override


Read host overrides from DNS Resolver (Unbound).<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-services-dnsresolver`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/services/unbound/host_override
```



### 4. Update Unbound Host Override


Modify host overrides from DNS Resolver (Unbound).<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-services-dnsresolver-edithost`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/services/unbound/host_override
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | integer | Specify the ID of the host override to update |
| host | string | Update the hostname of this host override (optional) |
| domain | string | Update the domain name of this host override (optional) |
| ip | string | Update the IPv4/IPv6 address of this host override (optional) |
| descr | string | Update the description of this host override (optional) |
| aliases | array | Update the aliases for this host override. This will replace any existing entries. (optional) |
| apply | boolean | Specify whether or not you would like this host override update to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are updating multiple host overrides at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/services/unbound/apply` endpoint. Otherwise, If you are only updating a single host override, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
	"host": "updated_test",
	"domain": "example.com",
	"ip": "127.0.0.2",
	"descr": "This is a test host override update via pfSense API!",
	"aliases": [
        {
            "host": "test2", 
            "domain": "example.com", 
            "descr": "This is an updated host override alias that will also resolve to this IP!"
        },
        {
            "host": "updated_to_add", 
            "domain": "example.com", 
            "descr": "This is a test host override alias that was also added during the update!"
        }
    ]
}
```



## SERVICES/UNBOUND/HOST_OVERRIDE/ALIAS



### 1. Create Unbound Host Override Alias


Add a new host override alias to DNS Resolver (Unbound).<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-services-dnsresolver-edithost`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/services/unbound/host_override/alias
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | integer | Specify the ID of the host override to apply this alias to. |
| host | string | Specify the hostname of the alias |
| domain | string | Specify the domain name of the alias |
| description | string | Description of alias (optional) |
| apply | boolean | Specify whether or not you would like this host override alias to be applied immediately, or simply written to the configuration to be applied later. Typically, if you are creating multiple host override aliases at once it Is best to set this to false and apply the changes afterwards using the `/api/v1/services/unbound/apply` endpoint. Otherwise, If you are only updating a single host override alias, you may set this true to apply it immediately. Defaults to false. (optional) |



***Body:***

```js        
{
    "id": 0,
	"host": "alias",
	"domain": "example.com",
	"descr": "This is a test host override alias added via pfSense API!",
    "apply": true
}
```



## STATUS/CARP



### 1. Read CARP Status


Read the CARP (failover) status.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-carp`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/status/carp
```



***Body:***

```js        
{
    
}
```



### 2. Update CARP Status


Read the CARP (failover) status.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-carp`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/status/carp
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| enable | boolean | Enable or disable CARP. Requires bool (optional) |
| maintenance_mode | boolean | Enable or disable CARP maintenance mode. Requires bool (optional) |



***Body:***

```js        
{
	"enable": true,
	"maintenance_mode": true
}
```



## STATUS/GATEWAY



### 1. Read Gateway Status


Read gateway status and metrics.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-gateway`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/status/gateway
```



***Body:***

```js        
{
    
}
```



## STATUS/INTERFACE



### 1. Read Interface Status


Read interface status and metrics.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-status-interfaces`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/status/interface
```



***Body:***

```js        
{
    
}
```



## STATUS/LOG



### 1. Read Configuration History Status Log


Read the configuration history log.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-diagnostics-configurationhistory`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/status/log/config_history
```



***Body:***

```js        
{

}
```



### 2. Read DHCP Status Log


Read the dhcpd.log file.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-diagnostics-logs-dhcp`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/status/log/dhcp
```



***Body:***

```js        
{

}
```



### 3. Read Firewall Status Log


Read the filter.log file.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-diagnostics-logs-firewall`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/status/log/firewall
```



***Body:***

```js        
{

}
```



### 4. Read System Status Log


Read the system.log file.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-diagnostics-logs-system`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/status/log/system
```



***Body:***

```js        
{

}
```



## STATUS/SYSTEM



### 1. Read System Status


Read system status and metrics. All usage values are represented as a decimal percentage. Temperature readings require thermal sensor and/or driver configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-dashboard-widgets`, `page-dashboard-all`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/status/system
```



***Body:***

```js        
{
    
}
```



## SYSTEM/API



### 1. Read System API Configuration


Read current API configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-api`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/system/api
```



***Body:***

```js        
{
    
}
```



### 2. Read System API Error Library


Read our error code library. This function does NOT require authentication and is intended to be used by third-party scripts/software to translate error codes to their full error message.


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/system/api/error
```



***Body:***

```js        
{
    
}
```



### 3. Update System API Configuration


Update the API configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-api`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/system/api
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| enable | boolean | Disable the API. If set to `false`, the API will be disable and no further API requests can be made. In most cases this Is not necessary. (optional) |
| persist | boolean | Enable/disable persistent API configuration. If set to `true`, pfSense API will store a copy of the API configuration In the case a system update or package update Is needed and/or the API configuration must be restored. If set to `false`, all API configuration will be lost whenever the system updates, the package Is updated, or the package Is deleted. It Is recommended to keep this feature enabled. (optional) |
| readonly | boolean | Enable read only mode. If set to `true`, the API will only answer read (GET) requests. This also means you will not be able to disable read only mode from the API.  |
| allow_options | boolean | Enable/disable the OPTIONS request method from API responses. If set to `true`, the API will answer OPTIONS requests. If set to `false`, the API will return a 405 Method Not Allowed response. (optional) |
| available_interfaces | array | Update the Interfaces that are allowed to answer API requests. Each Item In the array must be a valid physical Interface ID (e.g. `"em0"`), pfSense Interface ID, (e.g. `"opt1"`),  or descriptive Interface name (e.g. `"WAN"`). Additionally you may add `"localhost"` to allow local API requests, or add `"any"` to allow any Interface to answer API requests. It Is best practice to only allow Inside Interfaces to answer API requests, or use firewall rules to filter requests made to outside Interfaces. (optional) |
| authmode | string | Update the API authentication mode. Choices are `"local"` for local database authentication, `"jwt"` for JWT bearer token authentication, and `"token"` for standalone API token authentication. (optional) |
| jwt_exp | integer | Update the JWT expiration interval (in seconds). Value must be an Integer greater or equal to `300` and less than or equal to `86400`. This Is only applicable when the `authmode` setting Is set to `jwt`. (optional) |
| keyhash | string | Update the hashing algorithm to use when generating API tokens. Choices are `"sha256"`, `"sha384"`, `"sha512"`, and `"md5"`. This Is only applicable when the `authmode` setting Is set to `token`. (optional) |
| keybytes | integer | Update the key byte strength to use when generating API tokens. Choices are `16`, `32` and `64`. This Is only applicable when the `authmode` setting Is set to `token`. (optional) |
| custom_headers | array | Update the custom response headers for the API to return in API responses. This must be an array of key-value pairs (e.g. `{"custom-header": "custom-header-value}`. To revert custom headers to the default state, simply pass in an empty array. In most cases, custom headers are not necessary. An example use case for custom headers is setting CORS policy headers required by some frontend web applications. (optional) |



***Body:***

```js        
{
    "persist": false, 
    "jwt_exp": 86400, 
    "authmode": "token",
    "hashalgo": "sha512", 
    "keybytes": 64, 
    "allowed_interfaces": ["WAN"],
    "custom_headers": {"custom-header-1": "Value1", "custom-header-2": "Value2"},
    "allow_options": true
}
```



## SYSTEM/ARP



### 1. Delete System ARP Table


Delete an IP from the ARP table.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-diagnostics-arptable`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/system/arp
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| ip | string | IPv4 address to delete from ARP table |



***Body:***

```js        
{
	"ip": "192.168.1.5"
}
```



### 2. Read System ARP Table


Read the current ARP table.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-diagnostics-arptable`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/system/arp
```



***Body:***

```js        
{
    
}
```



## SYSTEM/CERTIFICATE



### 1. Create System Certificates


Add a new SSL certificate.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-certmanager`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/system/certificate
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| method | string | Set the method used to add the certificate. Current supported methods (`import`) |
| cert | string | Specify the Base64 encoded PEM certificate to import |
| key | string | Specify the corresponding Base64 encoded certificate key |
| descr | string | Set a descriptive name for the certificate |
| active | boolean | Set this certificate as the active certificate used by the webConfigurator (optional) |



***Body:***

```js        
{
	"method": "import",
	"cert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUZxekNDQTVPZ0F3SUJBZ0lVQi9rT2RoMzdTZnRxeHRqL1MxSTRkUTQyYXRvd0RRWUpLb1pJaHZjTkFRRUwKQlFBd1pURUxNQWtHQTFVRUJoTUNWVk14Q3pBSkJnTlZCQWdNQWxWVU1RMHdDd1lEVlFRSERBUlBjbVZ0TVNFdwpId1lEVlFRS0RCaEpiblJsY201bGRDQlhhV1JuYVhSeklGQjBlU0JNZEdReEZ6QVZCZ05WQkFNTURuUmxjM1F1CmMyVmpiV1YwTG1Odk1CNFhEVEl3TURJd05ESXdNelV3...",
	"key": "LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUpRZ0lCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQ1N3d2dna29BZ0VBQW9JQ0FRREQ5RkNLU1U3SmY0QngKeWlKNkNOWGhOckI0ZVhjTk9TTm9GUVJIbXlsV2dHbEN5djMydFdicmF3RFhhQzk2aVpOSTFzNG5qWTdQT3BlWgpoNmFlaTJ5NllheS9VWWtOUkZGQmp4WlZlLzRwS2pKeXBQRlFBUlpMVko2TlNXaU5raGkwbDlqeWtacTlEbkFnCk1mclZyUEo1YktDM3JJVV...",
	"descr": "webGui Cert",
	"active": true
}
```



### 2. Delete System Certificates


Delete an existing certificate.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-certmanager`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/system/certificate
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| refid | string | Specify the refid of the certificate to delete (required if `id` and `descr` are not defined) |
| id | string | Specify the id number of the certificate to delete (required if `refid` and `descr` are not defined) |
| descr | string | Specify the description of the certificate to delete (required if `id` and `refid` are not defined) _Note: if multiple certificates exist with the same name, only the first matching certificate will be deleted_ |



***Body:***

```js        
{
	"refid": "0"
}
```



### 3. Read System Certificates


Read installed SSL certificates.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-certmanager`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/system/certificate
```



***Body:***

```js        
{
    
}
```



## SYSTEM/CONFIG



### 1. Read System Configuration


Reads entire pfSense configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-diagnostics-backup-restore`, `page-diagnostics-command`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/system/config
```



***Body:***

```js        
{
    
}
```



## SYSTEM/DNS



### 1. Read System DNS


Read current system DNS configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/system/dns
```



***Body:***

```js        
{
    
}
```



### 2. Update System DNS


Modify the existing system DNS configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/system/dns
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| dnsserver | array or string | Specify which DNS servers to assign the system. Single values may be passed in as string, multiple values may be passed in as array of strings. Any existing configuration will be overwritten (optional) |
| dnsallowoverride | boolean | Enable or disable DNS override on WAN interfaces configured using DHCP (optional) |
| dnslocalhost | boolean | Enable or disable local system DNS queries (optional) |



***Body:***

```js        
{
	"dnsserver": ["8.8.4.4", "1.1.1.1", "8.8.8.8"],
	"dnslocalhost": true,
	"dnsallowoverride": false
}
```



## SYSTEM/DNS/SERVER



### 1. Create System DNS Server


Add a new DNS server to our system DNS servers.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/system/dns/server
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| dnsserver | array or string | Specify the IP(s) of the DNS servers to add. Single values may be specified as string, multiple values may be specified as array |



***Body:***

```js        
{
	"dnsserver": "1.1.1.1"
}
```



### 2. Delete System DNS Server


Delete existing system DNS servers.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/system/dns/server
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| dnsserver | array or string | Specify the IP(s) of the DNS servers to delete. Single values may be specified as string, multiple values may be specified as array |



***Body:***

```js        
{
	"dnsserver": ["8.8.4.4", "1.1.1.1"]
}
```



## SYSTEM/HALT



### 1. Halt System


Halts the pfSense system.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-diagnostics-haltsystem`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/system/halt
```



***Body:***

```js        
{
    
}
```



## SYSTEM/HOSTNAME



### 1. Read System Hostname


Read the system hostname.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/system/hostname
```



***Body:***

```js        
{
    
}
```



### 2. Update System Hostname


Modify the system hostname.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/system/hostname
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| hostname | string | Set the hostname portion of the system hostname. Do not specify to retain existing value (optional) |
| domain | string | Set the domain portion of the system hostname. Do not specify to retain existing value (optional) |



***Body:***

```js        
{
	"hostname": "hostname",
	"domain": "domain.com"
}
```



## SYSTEM/REBOOT



### 1. Create System Reboot


Reboots the pfSense system.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-diagnostics-rebootsystem`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/system/reboot
```



***Body:***

```js        
{
    
}
```



## SYSTEM/TABLE



### 1. Read System Tables


Read existing table entries.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-diagnostics-tables`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/system/table
```



***Body:***

```js        
{

}
```



## SYSTEM/TUNABLE



### 1. Create System Tunables


Create a new sysctl tunable.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-advanced-sysctl`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/system/tunable
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| tunable | string | Specify the name of this tunable. This should be a value recognized by sysctl |
| value | string or Integer | Specify the value to set this tunable |
| descr | string | Specify a description for this tunable (optional) |



***Body:***

```js        
{
    "tunable": "test3",
    "value": 1,
    "descr": "A test tunable added via pfSense API"
}
```



### 2. Delete System Tunables


Delete an existing sysctl tunable.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-advanced-sysctl`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/system/tunable
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | string | Specify the name of the tunable to delete |



***Body:***

```js        
{
    "id": "test"
}
```



### 3. Read System Tunables


Read current sysctl tunable configuration.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-advanced-sysctl`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/system/tunable
```



***Body:***

```js        
{

}
```



### 4. Update System Tunables


Update an existing sysctl tunable.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-advanced-sysctl`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/system/tunable
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| id | string | Specify the name of the tunable to update |
| tunable | string | Update the name of this tunable. This should be a value recognized by sysctl. (optional) |
| value | string or Integer | Update the value to set this tunable (optional) |
| descr | string | Update a description for this tunable (optional) |



***Body:***

```js        
{
    "id": "testnew",
    "tunable": "test",
    "value": "2",
    "descr": "A new test tunable added via pfSense API"
}
```



## SYSTEM/VERSION



### 1. Read System Version


Read pfSense version data. This includes base version, patch version, buildtime, and last commit.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-dashboard-widgets`, `page-diagnostics-command`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/system/version
```



***Body:***

```js        
{
    
}
```



## USER
API endpoints that create, read, update and delete pfSense users and authentication management.



### 1. Create Users


Add a new pfSense user to the local database.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-usermanager`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/user
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| username | string | Username to assign new user |
| password | string | Password to assign new user |
| descr | string | Descriptive name to assign new user (optional) |
| authorizedkeys | string | Base64 encoded authorized SSH keys to assign new user (optional) |
| ipsecpsk | string | IPsec pre-shared key to assign new user (optional) |
| disabled | boolean | Disable user account upon creation. Do not include to leave enabled. (optional) |
| expires | string | Expiration date for user account in  MM/DD/YYYY format (optional) |



***Body:***

```js        
{
	"disabled": true,
	"username": "new_user",
	"password": "changeme",
	"descr": "NEW USER",
	"authorizedkeys": "test auth key",
	"ipsecpsk": "test psk",
	"expires": "11/22/2020"
}
```



### 2. Delete Users


Delete an existing pfSense user from the local database.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-usermanager`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/user
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| username | string | Username to to delete |



***Body:***

```js        
{
	"username": "new_user"
}
```



### 3. Read Users


Reads users from the local user database.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-usermanager`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/user
```



***Body:***

```js        
{
    
}
```



### 4. Update Users


Update an existing user.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-usermanager`]


***Endpoint:***

```bash
Method: PUT
Type: RAW
URL: https://{{$hostname}}/api/v1/user
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| username | string | Username to modify |
| password | string | Change user password (optional) |
| descr | string | Descriptive name to assign the user (optional) |
| authorizedkeys | string | Base64 encoded authorized keys to add to user (optional) |
| ipsecpsk | string | IPsec pre-shared key to assign to user (optional) |
| disabled | boolean | Disable user account  (optional) |
| expires | string | Expiration date for user account in  MM/DD/YYYY format (optional) |



***Body:***

```js        
{
	"disabled": true,
	"username": "new_user",
	"password": "changeme",
	"descr": "NEW USER",
	"authorizedkeys": "test auth key",
	"ipsecpsk": "test psk",
	"expires": "11/22/2020"
}
```



## USER/AUTH_SERVER



### 1. Create LDAP Auth Servers


Add a new remote LDAP authentication server<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-authservers`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/user/auth_server/ldap
```


***Headers:***

| Key | Value | Description |
| --- | ------|-------------|
| Content-Type | application/json |  |



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Specify a descriptive name for the authentication server |
| host | string | Specify the remote hostname or IP of the LDAP server to query |
| ldap_port | string or integer | Specify the remote LDAP port |
| ldap_urltype | string | Specify the transport method to use for LDAP queries (`standard`, `starttls`, `encrypted`) |
| ldap_protver | string or integer | Specify the LDAP version used by the remote authentication server |
| ldap_scope | string | Specify the LDAP search scope (`subtree` for entire subtree, `one` for one level) |
| ldap_basedn | string | Specify the base DN of LDAP queries |
| ldap_authcn | string | Specify authentication container(s) |
| ldap_extended_query | string | Specify extended LDAP queries. Specifying any value enables LDAP extended queries (optional) |
| ldap_binddn | string | Specify the bind DN to use for authentication |
| ldap_bindpw | string | Specify the bind password to use for authentication |
| ldap_attr_user | string | Specify the LDAP user naming attribute |
| ldap_attr_group | string | Specify the LDAP group naming attribute |
| ldap_attr_member | string | Specify the LDAP group member attribute |
| ldap_attr_groupobj | string | Specify a group object class. Specifying any value enables RFC2307 mode (optional) |
| ldap_utf8 | boolean | Enable UTF-8 encoded LDAP parameters (optional) |
| ldap_timeout | string or integer | Set the connection timeout for LDAP requests. Defaults to 20 (optional) |
| active | boolean | Set this authentication server as the system default (optional) |



***Body:***

```js        
{
	"name": "TEST_AUTHSERVER",
	"host": "ldap.com",
	"ldap_port": 636,
	"ldap_urltype": "encrypted",
	"ldap_protver": "3",
	"ldap_timeout": 12,
	"ldap_scope": "subtree",
	"ldap_basedn": "dc=test,dc=com",
    "ldap_authcn": "ou=people,dc=test,dc=com;ou=groups,dc=test,dc=com",
    "ldap_attr_user": "uid",
    "ldap_attr_group": "cn",
    "ldap_attr_member": "memberof",
    "ldap_attr_groupobj": "posixGroup",
    "ldap_binddn": "cn=pfsense,dc=test,dc=com",
    "ldap_bindpw": "asdf",
    "active": false
}
```



### 2. Create RADIUS Auth Servers


Delete an existing RADIUS authentication server.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-authservers`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/user/auth_server/radius
```


***Headers:***

| Key | Value | Description |
| --- | ------|-------------|
| Content-Type | application/json |  |



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Specify a descriptive name for the RADIUS authentication server to create. This name must be unique from all other authentication servers on the system. |
| host | string | Specify the IP or hostname of the remote RADIUS authentication server. |
| radius_secret | string | Specify the shared secret of the remote RADIUS authentication server.  |
| radius_auth_port | integer | Specify the remote RADIUS authentication server's authentication port. If no value is specified, the authentication service on the remote RADIUS server will not be used. This field is optional if a `radius_acct_port` value is specified. (optional) |
| radius_acct_port | integer | Specify the remote RADIUS authentication server's accounting port. If no value is specified, the accounting service on the remote RADIUS server will not be used. This field is optional if a `radius_auth_port` value is specified.(optional) |
| radius_timeout | integer | Specify the amount of time (in seconds) to wait for the remote RADIUS server to respond before timing out. This value must be `1` or greater. Defaults to `5`. (optional) |
| radius_nasip_attribute | string |  |
| active | boolean | Specify whether pfSense should use this authentication server by default after creation.  (optional) |
| radius_protocol | string | Specify the RADIUS authentication protocol to use when communicating with the remote RADIUS server. Options are `PAP`, `CHAP_MD5`, `MSCHAPv1` or `MSCHAPv2`. Defaults to `MSCHAPv2`. (optional) |



***Body:***

```js        
{
    "name": "TEST_RADIUS",
    "host": "123",
    "radius_secret": "testsecret",
    "radius_auth_port": 1812,
    "radius_acct_port": 1813,
    "radius_protocol": "MSCHAPv2",
    "radius_timeout": 5,
    "radius_nasip_attribute": "wan"
}
```



### 3. Delete Auth Servers


Delete an existing authentication server. This can be either a RADIUS or LDAP authentication server.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-authservers`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/user/auth_server
```


***Headers:***

| Key | Value | Description |
| --- | ------|-------------|
| Content-Type | application/json |  |



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Specify the name of the  authentication server to delete |



***Body:***

```js        
{
	"name": "test"
}
```



### 4. Delete LDAP Auth Servers


Delete an existing LDAP authentication server<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-authservers`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/user/auth_server/ldap
```


***Headers:***

| Key | Value | Description |
| --- | ------|-------------|
| Content-Type | application/json |  |



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Specify the name of the LDAP authentication server to delete |



***Body:***

```js        
{
	"name": "LDAP_AUTHSERVER_0"
}
```



### 5. Delete RADIUS Auth Servers


Delete an existing RADIUS authentication server.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-authservers`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/user/auth_server/radius
```


***Headers:***

| Key | Value | Description |
| --- | ------|-------------|
| Content-Type | application/json |  |



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| name | string | Specify the name of the RADIUS authentication server to delete |



***Body:***

```js        
{
	"name": "test"
}
```



### 6. Read Auth Servers


Read configured LDAP and RADIUS authentication servers<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-authservers`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/user/auth_server
```



***Body:***

```js        
{
    
}
```



### 7. Read LDAP Auth Servers


Read configured LDAP authentication servers<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-authservers`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/user/auth_server/ldap
```


***Headers:***

| Key | Value | Description |
| --- | ------|-------------|
| Content-Type | application/json |  |



***Body:***

```js        
{
    
}
```



### 8. Read RADIUS Auth Servers


Read configured RADIUS authentication servers<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-authservers`]


***Endpoint:***

```bash
Method: GET
Type: RAW
URL: https://{{$hostname}}/api/v1/user/auth_server/radius
```



***Body:***

```js        
{
    
}
```



## USER/GROUP



### 1. Create User Group


Add a user to an existing group.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-groupmanager`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/user/group
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| username | string | Username to grant new privilege |
| group | string | Name of group to assign. Multiple groups may be assigned at once if passed in as array. Group name must match exactly as is displayed in webConfigurator. |



***Body:***

```js        
{
	"username": "new_user",
	"group": ["admins"]
}
```



### 2. Delete User Group


Delete a user from an existing group.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-groupmanager`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/user/group
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| username | string | Username to remove from group |
| group | string | Name of group to delete. Multiple groups may be deleted at once if passed in as array. Group name must match exactly as is displayed in webConfigurator. |



***Body:***

```js        
{
	"username": "new_user",
	"group": "test"
}
```



## USER/PRIVILEGE



### 1. Create User Privileges


Add a new privilege to an existing user.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-usermanager-addprivs`]


***Endpoint:***

```bash
Method: POST
Type: RAW
URL: https://{{$hostname}}/api/v1/user/privilege
```



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| username | string | Username to grant new privilege |
| priv | string | Name of new privilege to assign. Multiple privileges may be assigned at once if passed in as  array. Privilege name will match the POST data name found in the webConfigurator.  |



***Body:***

```js        
{
	"username": "new_user",
	"priv": ["page-all", "page-system-usermanager"]
}
```



### 2. Delete User Privileges


Delete an existing privilege from an existing user.<br><br>

_Requires at least one of the following privileges:_ [`page-all`, `page-system-usermanager-addprivs`]


***Endpoint:***

```bash
Method: DELETE
Type: RAW
URL: https://{{$hostname}}/api/v1/user/privilege
```


***Headers:***

| Key | Value | Description |
| --- | ------|-------------|
| Content-Type | application/json |  |



***Query params:***

| Key | Value | Description |
| --- | ------|-------------|
| username | string | Username to remove privilege |
| priv | string | Name of new privilege to delete. Multiple privileges may be deleted at once if passed in as  array. Privilege name will match the POST data name found in the webConfigurator.  |



***Body:***

```js        
{
	"username": "new_user",
	"priv": "page-system-usermanager-addprivs"
}
```



---
[Back to top](#pfsense-rest-api-documentation)
