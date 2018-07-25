## Overview
Credo360 public API allows 3rd parties to retrieve users' reputation information from Credo system. Currently, the API enables the following:

* Validate ownership of a given Credo ID (authentication)
* Retrieve user's public profile

In the future additional capabilities will be added to enable 3rd parties to submit transaction and rating information to Credo system.

The root for all API requests is:
```
https://api.credo360.com
```
All requests to the endpoint listed in subsequent sections must include an Authorization header in the following form:
```
Authorization: client [clientId]
```
where, `clientId` is a unique identifier of the 3rd party making the request. Client IDs are distributed by Credo360 to our API partners.

## Authentication
Authentication endpoints allow a 3rd party client to verify ownership of a Credo ID by a specific user. The endpoints are:

```
GET /auth/:handleId      - to retrieve available authentication methods
POST /auth/:handleId     - to generate an authentication challenge
PATCH /auth/:challengeId - to validate an authentication challenge
```

The general workflow is as follows:
1. The client generates an authentication challenge for the user. This may require querying the API to determine challenge methods available for a specific user.
2. If the challenge is a passcode, Credo delivers a passcode to the user "out of band" (e.g. via SMS). The user then supplies this passcode to the client, and the client validates the passcode against Credo API.
3. If the challenge is a signature, the client must sign the provided nonce with the user's private key and validate the signature against Credo API.

### Retrieving Available Authentication Methods
Available authentication methods for a given user can be retrieved using the following endpoint:
```
GET /auth/:handleId
```
where `handleId` is the user's Credo ID.

Response from this endpoint will contain JSON in the following format:
```JSON
{
  "methods": [ "passcode", "signature" ]
}
```
Where `methods` will contain an array of available authentication methods for the specified user. The following methods are currently possible:

| Method      | Description |
| ----------- | ----------- |
| passcode    | Authentication can be performed using a randomly generated passcode which is delivered to the user "out of band" |
| signature   | Authentication can be performed using users' private keys; the user would have to register their public key with Credo system first |

### Generating Authentication Challenge
An authentication challenge can be generated using the following endpoint:
```
POST /auth/:handleId
```
Where `handleId` is the user's Credo ID. The endpoint also accepts the following URL parameters:

| Parameter   | Description |
| ----------- | ----------- |
| method      | Defines the method which should to be used to generate an authentication challenge; possible values are: [`passcode`, `signature`]; the default is `passcode` |

Response from this endpoint will contain JSON in the following format:
```JSON
{
  "challengeId": "Zi4czDlpdMdP...3RRN9DQ",
  "method": "passcode",
  "expiresAt": 1528249689,
  "validation": {
    "length": 6,
    "attempts": 3,
    "delivery": "sms"
  }
}
```

The meaning of the properties in the above object is as follows:

| Property    | Type       | Description |
| ----------- | ---------- | ----------- |
| challengeId | string     | Unique challenge identifier; can be at most 160 characters long |
| method      | string     | Method used to generate the challenge; can be one of the following values: [`passcode`, `signature`] |
| expiresAt   | timestamp? | Unix timestamp (in seconds) indicating the time until the challenge can be validated; this property is optional and, if omitted, the challenge never expires |
| validation  | object     | An object containing validation information for the challenge; the structure of the object depends on challenge `method` as described in the following sections |

#### Passcode Validation
Passcode validation object has the following form:
```JSON
{
  "length": 6,
  "attempts": 3,
  "delivery": "sms"
}
```
The meaning of the properties in the above object is as follows:

| Property    | Type       | Description |
| ----------- | ---------- | ----------- |
| length      | integer    | Number of digits in the passcode; must be between 4 and 8 |
| attempts    | integer?   | Number of allowed validation attempts. This property is optional and, if omitted, the number of attempts is unlimited; when present must be a value between 1 and 5 |
| delivery    | string     | Delivery mode indicating how the passcode was delivered to the user; possible values are: [`otp`, `email`, `sms`] |

#### Signature Validation
Signature validation object has the following form:
```JSON
{
  "nonce": "AedIRjmL72qsnE2uzNW8uVHU1Y...PqU5q5laRROPr9Q1P03drh2BZCVhLSa5i7cBOgt"
}
```
The meaning of the properties in the above object is as follows:

| Property    | Type       | Description |
| ----------- | ---------- | ----------- |
| nonce       | string     | Value to be signed with the user's private key; will be base64 encoded 256-bit value|

### Validating Authentication Challenge
An authentication challenge can be validated using the following endpoint:
```
PATCH /auth/:challengeId
```
Where `challengeId` is uniquer identifier of the challenge that is to be validated. The endpoint also requires a request body in the following form:
```JSON
{
  "response": "783423"
}
```
Where `response` is:
* A passcode provided by the user
* Or a signature generated by signing the `nonce` value using user's private key

Response from this endpoint will contain JSON in the following form:
```JSON
{
  "handleId": "02598397346"
}
```
Where, `handleId` is the Credo ID of the user for whom the challenge has been validated.

## User Info
Information about a specific user can be retrieved via the following endpoint:
```
GET /users/:handleId
```

The response from this endpoint will contain JSON in the following format:
```JSON
{
    "handle": {
        "id": "02598397346",
        "status": "valid"
    },
    "profile": {
        "name": {
            "firstName": "Irakliy",
            "lastName": "K"
        },
        "location": {
            "city": "Redondo Beach",
            "state": "California",
            "country": "us"
        }
    },
    "reputation": {
        "score": 288,
        "quality": "good",
        "segments": {
            "identity": {
                "score": 99,
                "quality": "excellent",
                "verifications": 8
            },
            "behavior": {
                "score": 77.09,
                "quality": "good",
                "transactions": 157,
                "transactors": 56,
                "ratings": 42,
                "reviews": 1
            },
            "endorsements": {
                "score": 82,
                "quality": "good",
                "contacts": 12,
                "averageScore": 262,
                "reviews": 2
            }
        }
    },
    "verifications": {
        "email": {
            "score": 5,
            "verifiedOn": 1524046183116
        },
        "phone": {
            "countryCode": 1,
            "areaCode": "760",
            "last2digits": "12",
            "score": 40,
            "verifiedOn": 1527368780427
        },
        "facebook": {
            "friends": 640,
            "score": 49,
            "verifiedOn": 1512764091433
        },
        "linkedIn": {
            "connections": 500,
            "score": 48,
            "verifiedOn": 1512764190356
        },
        "reddit": {
            "createdOn": 1460421184000,
            "score": 5,
            "verifiedOn": 1502228502704
        },
        "blockscore": {
            "document": "ssn",
            "score": 100,
            "verifiedOn": 1510957768433
        },
        "dwolla": {
            "createdOn": 1504638397323,
            "accounts": 1,
            "score": 43,
            "verifiedOn": 1504642798327
        },
        "coinbase": {
            "createdOn": 1404766971000,
            "score": 10,
            "verifiedOn": 1532143480316
        }
    },
    "payments": {
        "credoWallet": true,
        "coinbase": true
    },
    "memberSince": 1482750938147
}
```
The meaning of the properties in the above object is as follows:

| Property    | Type       | Description |
| ----------- | ---------- | ----------- |
| handle      | object     | User's Credo ID and it's status; status can be one of the following values: [`valid`, `deactivated`, `suspended`] |
| profile     | <[Profile](#user-profile)>? | User's public profile |
| reputation  | <[Reputation](#user-reputation)> | An object describing user's reputation |
| verifications | [[Verification](#user-verifications)]? | An array of user's verifications |
| payments    | object?    | An object containing a set of payments methods available to the user |
| memberSince | timestamp  | Unix timestamp (in seconds) of when the user created their account with the provider |

### User Profile
User's public profile object has the following form:

```JSON
{
  "name": {
    "first": "Alex",
    "Last": "K"
  },
  "picture": {
    "source": "facebook",
    "updatedOn": 1493840810506,
    "fullUrl": "https://...",
    "thumbUrl": "https://..."
  },
  "location": {
    "country": "us",
    "region": "California",
    "city": "Los Angeles"
  },
  "age": {
    "years": 25
  },
  "gender": "female"
}
```
The meaning of the properties in the above object is as follows:

| Property    | Type       | Description |
| ----------- | ---------- | ----------- |
| name        | object     | An object containing the user's name. The first name will always be present; the last name may or may not be present based on user's privacy settings. |
| picture     | object?    | An object containing URLs for user's profile pictures; may or may not be present based on user's privacy settings. |
| location    | object?    | An object containing user's location. May or may not be present based on user's privacy settings; when present, will always contain `country` element. |
| age         | object?    | An object containing user's age; may or may not be present based on user's privacy settings; when present, will always contain `years` element. |
| gender      | enum?      | User's gender. May or may not be present based on user's privacy settings. When present, can be one of the following values: [`male`, `female`, `other`] |

### User Reputation
User's reputation object has the following form:
```JSON
{
  "score": 288,
  "quality": "good",
  "segments": {
    "identity": {
      "score": 99,
      "quality": "excellent",
      "verifications": 8
    },
    "behavior": {
      "score": 77.09,
      "quality": "good",
      "transactions": 157,
      "transactors": 56,
      "ratings": 42,
      "reviews": 1
    },
    "endorsements": {
      "score": 82,
      "quality": "good",
      "contacts": 12,
      "averageScore": 262,
      "reviews": 2
    }
  }
}
```
The meaning of the properties in the above object is as follows:

| Property    | Type       | Description |
| ----------- | ---------- | ----------- |
| score       | integer    | User's Credo Score; will be an integer in the range between 60 and 360 |
| quality     | string     | Quality of user's reputation; can be one of the following values [`provisional`, `poor`, `fair`, `good`, `excellent`] |
| segments    | object     | An object containing summarized information about reputation segments: identity, behavior, and endorsements |

### User Verifications
User verifications are data elements that have been verified by Credo for the specified user. User object will contain an array of verification objects. The structure of verification objects is type-dependent, but all verifications will have a `verifiedOn` date (expressed in Unix timestamp). Possible verifications are listed below. This list can be expanded over time:

| Verification | Description |
| ------------ | ----------- |
| email        | The user has a verified email address |
| phone        | The user has a mobile phone with the number in the indicated `country` |
| facebook     | The user has a Facebook account with the indicated number of `friends` |
| linkedIn     | The user has a LinkedIn account with the indicated number of `connections` |
| reddit       | The user has a Reddit account with the indicated age (`createdOn`) and `karma` |
| twitter      | The user has a Twitter account with the indicated number of `followers` |
| blockscore   | The user has verified their Social Security number |
| dwolla       | The user has created a credo wallet and has verified the specified number of bank accounts |
| coinbase     | The user has verified ownership of a Coinbase account |
