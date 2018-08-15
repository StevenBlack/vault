---
layout: "api"
page_title: "AliCloud - Secrets Engines - HTTP API"
sidebar_current: "docs-http-secret-alicloud"
description: |-
  This is the API documentation for the Vault AliCloud secrets engine.
---

# AliCloud Secrets Engine (API)

This is the API documentation for the Vault AliCloud secrets engine. For general
information about the usage and operation of the AliCloud secrets engine, please see
the [Vault AliCloud documentation](/docs/secrets/alicloud/index.html).

This documentation assumes the AliCloud secrets engine is enabled at the `/alicloud` path
in Vault. Since it is possible to enable secrets engines at any location, please
update your API calls accordingly.

## Config management

This endpoint configures the root RAM credentials to communicate with AliCloud.

At present, this endpoint does not confirm that the provided AliCloud credentials are
valid AliCloud credentials with proper permissions.

Please see the [Vault AliCloud documentation](/docs/secrets/alicloud/index.html) for
the policies that should be attached to the access key you provide.

| Method   | Path                         | Produces               |
| :------- | :--------------------------- | :--------------------- |
| `POST`   | `/alicloud/config`           | `204 (empty body)`     |
| `GET`    | `/alicloud/config`           | `200 application/json` |

### Sample Post Request

```
$ curl \
    --header "X-Vault-Token: ..." \
    --request POST \
    --data @payload.json \
    http://127.0.0.1:8200/v1/alicloud/config
```

### Sample Post Payload

```json
{
  "access_key": "0wNEpMMlzy7szvai",
  "secret_key": "PupkTg8jdmau1cXxYacgE736PJj4cA"
}
```

### Sample Get Response Data

```json
{
    "access_key": "0wNEpMMlzy7szvai"
}
```

## Role management

The `roles` endpoint configures how Vault will manage the passwords for individual service accounts.

### Parameters

* `remote_policies` (TODO string, required) - The name of a pre-existing service account in Active Directory that maps to this role.
* `inline_policies` (TODO string, optional) - The password time-to-live in seconds. Defaults to the configuration `ttl` if not provided.
* `role_arn` (TODO string, optional) - The password time-to-live in seconds. Defaults to the configuration `ttl` if not provided.
* `ttl`
* `max_ttl`

| Method   | Path                        | Produces               |
| :------- | :---------------------------| :--------------------- |
| `GET`    | `/alicloud/roles`           | `200 application/json` |
| `GET`    | `/alicloud/role`            | `200 application/json` |
| `POST`   | `/alicloud/role/:role_name` | `204 (empty body)`     |
| `GET`    | `/alicloud/role/:role_name` | `200 application/json` |
| `DELETE` | `/alicloud/role/:role_name` | `204 (empty body)`     |

### Sample Post Request

```
$ curl \
    --header "X-Vault-Token: ..." \
    --request POST \
    --data @payload.json \
    http://127.0.0.1:8200/v1/alicloud/role/my-application
```

### Sample Post Payload Using Policies

```json
{
  "remote_policies": [
    "name:AliyunOSSReadOnlyAccess,type:System",
    "name:AliyunRDSReadOnlyAccess,type:System"
  ],
  "inline_policies": "[{\"Statement\": [{\"Action\": [\"ram:Get*\",\"ram:List*\"],\"Effect\": \"Allow\",\"Resource\": \"*\"}],\"Version\": \"1\"}]"
}
```

### Sample Get Role Response Using Policies

```json
{
	"inline_policies": [{
		"hash": "49796debb24d39b7a61485f9b0c97e04",
		"policy_document": {
			"Statement": [{
				"Action": ["ram:Get*", "ram:List*"],
				"Effect": "Allow",
				"Resource": "*"
			}],
			"Version": "1"
		}
	}],
	"max_ttl": 0,
	"remote_policies": [{
		"name": "AliyunOSSReadOnlyAccess",
		"type": "System"
	}, {
		"name": "AliyunRDSReadOnlyAccess",
		"type": "System"
	}],
	"role_arn": "",
	"ttl": 0
}
```

### Sample Post Payload Using Assume-Role

```json
{
  "role_arn": "acs:ram::5138828231865461:role/hastrustedactors"
}
```

### Sample Get Role Response Using Assume-Role

```json
{
	"inline_policies": null,
	"max_ttl": 0,
	"remote_policies": null,
	"role_arn": "acs:ram::5138828231865461:role/hastrustedactors",
	"ttl": 0
}
```

### Sample List Roles Response

Performing a `LIST` on the `/alicloud/roles` endpoint will list the names of all the roles Vault contains.

```json
[
  "policy-based",
  "role-based"
]
```

## Generate RAM Credentials

This endpoint generates dynamic RAM credentials based on the named role. This
role must be created before queried.

| Method   | Path                         | Produces               |
| :------- | :--------------------------- | :--------------------- |
| `GET`    | `/alicloud/creds/:name`      | `200 application/json` |

### Parameters

- `name` `(string: <required>)` – Specifies the name of the role to generate
  credentials against. This is part of the request URL.

### Sample Request

```
$ curl \
    --header "X-Vault-Token: ..." \
    http://127.0.0.1:8200/v1/alicloud/creds/example-role
```

### Sample Response for Roles Using Policies

```json
{
  "access_key": "0wNEpMMlzy7szvai",
  "secret_key": "PupkTg8jdmau1cXxYacgE736PJj4cA"
}

```

### Sample Response for Roles Using Assume-Role

```json
{
	"access_key": "STS.L4aBSCSJVMuKg5U1vFDw",
	"expiration": "2018-08-15T22:04:07Z",
	"secret_key": "wyLTSmsyPGP1ohvvw8xYgB29dlGI8KMiH2pKCNZ9",
	"security_token": "CAESrAIIARKAAShQquMnLIlbvEcIxO6wCoqJufs8sWwieUxu45hS9AvKNEte8KRUWiJWJ6Y+YHAPgNwi7yfRecMFydL2uPOgBI7LDio0RkbYLmJfIxHM2nGBPdml7kYEOXmJp2aDhbvvwVYIyt/8iES/R6N208wQh0Pk2bu+/9dvalp6wOHF4gkFGhhTVFMuTDRhQlNDU0pWTXVLZzVVMXZGRHciBTQzMjc0KgVhbGljZTCpnJjwySk6BlJzYU1ENUJuCgExGmkKBUFsbG93Eh8KDEFjdGlvbkVxdWFscxIGQWN0aW9uGgcKBW9zczoqEj8KDlJlc291cmNlRXF1YWxzEghSZXNvdXJjZRojCiFhY3M6b3NzOio6NDMyNzQ6c2FtcGxlYm94L2FsaWNlLyo="
}
```