# Introduction

This file contains the full documentation for Eko's developer APIs in markdown format.

This section contains the general information about the API, including the base URL, the version, and the authentication methods.

## 1. API Base URLs

- **Production:** `https://api.eko.in/ekoicici/v3`
- **Sandbox/UAT:** `https://staging.eko.in/ekoapi/v3`

## 2. Developer Portal Links

- **Guides:** [https://developers.eko.in/docs](https://developers.eko.in/docs)
- **API References:** [https://developers.eko.in/reference](https://developers.eko.in/reference)

## 3. Authentication

To ensure the security & integrity of your data, you must authenticate each API call by passing the following headers:

| Header | Data Type | Description |
| --- | --- | --- |
| **developer_key** | string | Your static API key shared by Eko |
| **secret-key** | string | Dynamic security key, to be generated before every request |
| **secret-key-timestamp** | string | The request timestamp, used to generate the secret-key |
| **request_hash** | string | This is required only for financial transactions. See the guide below. |

### How to get the _developer_key_?

Production developer_key will be shared after the following is done:
1.  Organisation's KYC is completed on Eko's platform (visit [https://connect.eko.in](https://connect.eko.in)).
2.  The UAT **developer\_key** can be obtained from the [platform credentials](https://developers.eko.in/v3/docs/platform-credentials) section.

### How to generate the _secret-key_ & _secret-key-timestamp_?

1. Eko will provide an _access\_key_. It must be kept a secret on your backend. If compromised, it can be regenerated.
   1.  The access\_key will be shared to you over email, after completing your organisation's KYC at [https://connect.eko.in](https://connect.eko.in)
   2.  For UAT, use the following access\_key: d2fe1d99-6298-4af2-8cc5-d97dcf46df30
2. Encode the _access\_key_ using base64 encoding.
3. Get the current timestamp (in milliseconds since UNIX epoch). It will be used as the **secret-key-timestamp**. (Check [currentmillis.com](https://currentmillis.com/) to understand the timestamp format)
4. Compute the signature by hashing salt and base64 encoded key using Hash-based message authentication code HMAC and SHA256
5. Encode the signature using base64 encoding technique. This encoded value is your **secret-key**.

#### Example Code (PHP)

```php
<?php
    $access_key = "d2fe1d99-6298-4af2-8cc5-d97dcf46df30"; // DO NOT STORE IT HERE. Read it from env or configuration files

    // Encode it using base64
    $encodedKey = base64_encode($access_key);

    // Get current timestamp in milliseconds since UNIX epoch as STRING
    // Check out https://currentmillis.com to understand the timestamp format
    $secret_key_timestamp = (int)(round(microtime(true) * 1000));

    // Computes the signature by hashing the salt with the encoded key
    $signature = hash_hmac('SHA256', $secret_key_timestamp, $encodedKey, true);

    // Encode it using base64
    $secret_key = base64_encode($signature);

    //Display
    echo "secret-key: " . $secret_key. "\n";
    echo "secret-key-timestamp: " . $secret_key_timestamp . "\n";
?>
```

> **Note:**
> 1.  The secret-key-timestamp must match with the current time.
> 2.  **If you are using .NET**, you should not send overly specific time stamps, due to differing interpretations of how extra time precision should be dropped. To avoid overly specific time stamps, manually construct dateTime objects with no more than millisecond precision.


### How to generate the _request\_hash_?

The _request\_hash_ header is only required for financial transactions for a certain amount which needs to be debited from your Eko wallet or your agent's Eko wallet (E-value).

It is used to encode critical parameters of your financial transaction such as amount, agent, recipient, etc. It ensures that the values have been validated on your server and they have not been tampered with.

Before generating the request hash you need to generate a string by concatenating some parameters in a particular order. You cannot change the sequence.

For example, sequence of concatenated string for Bill Payments = `secret_key_timestamp + utility_acc_no + amount + user_code`

After generating the concatenated string please follow the following procedure :

1. Encode your authenticator password using base64. Authenticator password will be the _access\_key_ which you have used for the secret-key generation.
   1. For UAT, the access\_key is "d2fe1d99-6298-4af2-8cc5-d97dcf46df30".
2. After encoding the key, you need to hmac the concatenated string and encoded\_key using hmac256.
3. After hmac , you need to again encode the result using the base64.

Final result after encoding will be the request\_hash.



#### Sample Code to Generate _request\_hash_ (PHP):

**Note:** The following example generates the request\_hash for the Bill Payment API by concatenating secret\_key\_timestamp, utility\_acc\_no, amount, and user\_code (in the same order). For APIs, a different set of parameters may be required. See the corresponding API docs for details.

```php
<?php

const HMAC_SHA256 = 'sha256';

function generateRequestHashForBillPay() {
	try {
    	$access_key = 'd2fe1d99-6298-4af2-8cc5-d97dcf46df30'; // DO NOT STORE IT HERE. Read it from env or configuration files
    	$encodedKey = base64_encode($access_key);
    	$timestamp = round(microtime(true) * 1000);
     	$secret_key_timestamp = (string) $timestamp;

    	$secret_key = base64_encode(hash_hmac(HMAC_SHA256, $secret_key_timestamp, $encodedKey, true));

    	$utility_acc_no = '151627591';
    	$amount = '50';
    	$user_code = '20810200';

    	$concatenatedString = $secret_key_timestamp . $utility_acc_no . $amount . $user_code;
    	$request_hash = base64_encode(hash_hmac(HMAC_SHA256, $concatenatedString, $encodedKey, true));

    	echo "secret-key: $secret_key\n";
    	echo "secret-key-timestamp: $secret_key_timestamp\n";
    	echo "request_hash: $request_hash\n";
	} catch (Exception $e) {
    	echo $e->getMessage();
	}
}
?>
```

## 4. Common Request/Response Parameters

### Common Request Headers

| Header | Data Type | Required? | Description |
| --- | --- | --- | --- |
| **developer_key** | string | Required | Your static API key shared by Eko |
| **secret-key** | string | Required | Dynamic security key, to be generated before every request |
| **secret-key-timestamp** | number | Required | The request timestamp, used to generate the secret-key |
| **request_hash** | string | Optional | This is required only for financial transactions. If required, it will be mentioned in the individual API details. |

### Common Request Parameters

| Parameter | Data Type | Required? | Description |
| --- | --- | --- | --- |
| **initiator_id** | number | Required | Your registered mobile number (See Sandbox Credentials for Testing section) |
| **user_code** | string | Required | Unique code of your registered user/agent |
| **client_ref_id** | string | Optional | Your unique reference number for the transaction |

### Common Response Parameters
The response will always be of type JSON (application/json) unless otherwise specified in the API documentation. All responses will contain the following parameters:

| Parameter | Data Type | Description |
| --- | --- | --- |
| **status** | number | Status of the response. 0 for success, non-zero for failure |
| **message** | string | Description of the status |
| **data** | object | The actual response data. Rest of the important response parameters will be passed inside this object and it will vary for each API |
| **response_status_id** | number | An alternate status code for the response. -2 = Another transaction in progress for the user, -1 = Success for GET or non-transactional APIs, 0 = Success for POST or transactional APIs, 1 = Failure, 2 = Initiated, 3 = Refund Initiated, 4 = Refunded, 5 = On Hold |
| **response_type_id** | number | A unique identifier for each response type for an API. Details can be found in the API documentation |


## 5. Status & Error Codes

### Common HTTP Response Codes

| HTTP Response Code                                                                   | Type      | Description                                                                                                                                                                      |
| ------------------------------------------------------------------------------------ | --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 200                                                                                  | OK        | Response returned by our system. See _status_, _tx\_status_, and _message_ parameters in the response to understand the correct status.                                          |
| 403                                                                                  | Forbidden | It is usually due to sending an incorrect secret-key or timestamp. Please check the generation code in the [Authentication](https://developers.eko.in/v3/docs/authauth) section. |
| Also make sure to use your auth\_key in the generation code (not the developer key). |
| 404                                                                                  | Not Found | This error occurs when you are passing the wrong request URL, please make sure to check the request                                                                              |
URL before hitting the request.
Production Base URL: [https://api.eko.in:25002/ekoicici/](https://api.eko.in:25002/ekoicici/)
Staging Base URL: [https://staging.eko.in/ekoapi/](https://staging.eko.in/ekoapi/) |
| 405 | Method Not Allowed | Check the correct HTTP request method of the APIs from [API Reference](https://developers.eko.in/v3/reference/agent-management-overview), for example, GET, POST, PUT or DELETE. |
| 415 | Unsupported Media Type | Check the _Content-Type_ of the API from its respective [API Reference](https://developers.eko.in/v3/reference/agent-management-overview) page and check if you are using the same content-type or not. Example: Content-Type = `application/x-www-form-urlencoded` |
| 500 | Internal Server Error | It usually implies that the API is not able to connect to our servers. For staging, remove the port 25004 from the URL. And, in production re-check your URL and HTTP method. |


### Common Error Messages

| message                                                                                | Resolution                                                                                                                                                         |
| -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| No mapping rule matched                                                                | In the URL swap v1 with v2 or vice-versa. If it still does not work then send the request and URL at [sales.engineer@eko.co.in.](mailto:sales.engineer@eko.co.in.) |
| \- Agent not allowed - Agent not allowed to do this transaction - Customer not allowed | Check if the service is ACTIVATED for the user\_code. If not then, activate the service for the merchant.                                                          |
| No Key for Response                                                                    | Please re-check the value of the request parameters. It is either missing or the format is incorrect.                                                              |
| Kindly use your production key                                                         | Check if you're using the correct developer key (credentials for staging cannot be used in production).                                                            |


### Response Status Codes

For all financial transactions, **_status = 0_** should be treated as successful else fail and the current state of the transaction can be retrieved from **_tx\_status_** and **_txstatus\_desc_** parameter.

For all non-financial requests, you may need to consider both status and response\_type\_id parameters.

| status | Description                                                 |
| ------ | ----------------------------------------------------------- |
| 0      | Success                                                     |
| 463    | User not found                                              |
| 327    | Enrollment done. Verification pending                       |
| 17     | User wallet already exists                                  |
| 31     | Agent can not be registered                                 |
| 132    | Sender name should only contain letters                     |
| 302    | Wrong OTP                                                   |
| 303    | OTP expired                                                 |
| 342    | Recipient already registered                                |
| 145    | Recipient mobile number should be numeric                   |
| 140    | Recipient mobile number should be 10 digit                  |
| 131    | Recipient name should only contain letters                  |
| 122    | Recipient name length should be in between 1 and 50         |
| 39     | Max recipient limit reached                                 |
| 41     | Wrong IFSC                                                  |
| 536    | Invalid recipient type format                               |
| 537    | Invalid recipient type length                               |
| 44/45  | Incomplete IFSC Code                                        |
| 48     | Recipient bank not found                                    |
| 102    | Invalid Account number length                               |
| 136    | Please provide valid ifsc format                            |
| 508    | Invalid IFSC for the selected bank                          |
| 521    | IFSC not found out in the system                            |
| 313    | Recipient registration not done                             |
| 317    | NEFT not allowed                                            |
| 53     | IMPS transaction not allowed                                |
| 55     | Error from NPCI                                             |
| 460    | Invalid channel                                             |
| 319    | Invalid Sender/Initiator                                    |
| 314    | Failed! Monthly limit exceeds                               |
| 350    | Verification failed. Recipient name not found.              |
| 344    | IMPS is not available in this bank                          |
| 46     | Invalid account details                                     |
| 168    | TID does not exist                                          |
| 1237   | ID proof number already exists in the system                |
| 585    | Customer already KYC Approved                               |
| 347    | Insufficient balance                                        |
| 945    | Sender /Beneficiary limit has been exhausted for this month |
| 544    | Transaction not processed.Bank is not available now.        |

Possible values of parameter **_tx\_status_**:

| tx\_status | txstatus\_desc                               |
| ---------- | -------------------------------------------- |
| 0          | Success                                      |
| 1          | Fail                                         |
| 2          | Response Awaited/Initiated (in case of NEFT) |
| 3          | Refund Pending                               |
| 4          | Refunded                                     |
| 5          | On Hold ( Transaction Inquiry Required)      |



## 6. Sandbox Credentials for Testing

For testing AePS Gateway, activation of services, DMT (including Bank verification), AePS API and BBPS, use the below credentials:
1. developer\_key: becbbce45f79c6f5109f848acd540567
2. Key: d2fe1d99-6298-4af2-8cc5-d97dcf46df30 (will be used to generate secret-key and secret-key-timestamp). The secret-key and secret-key-timestamp have to be generated dynamically. Refer to the link [https://developers.eko.in/docs/authentication](https://developers.eko.in/v3/docs/authentication) for the dynamic secret-key and secret-key-timestamp generation
3. initiator\_id: 9962981729
4. user\_code: 20810200

For testing Indo-Nepal APIs, use the following credentials:
1. developer\_key: becbbce45f79c6f5109f848acd540567
2. Key: d2fe1d99-6298-4af2-8cc5-d97dcf46df30 (will be used to generate secret-key and secret-key-timestamp).The secret-key and secret-key-timestamp have to be generated dynamically. Refer to the link [https://developers.eko.in/docs/authentication](https://developers.eko.in/v3/docs/authentication) for the dynamic secret-key and secret-key-timestamp generation
3. initiator\_id: 9910028267

For testing PAN Verification, use the following credentials:
1. developer\_key: becbbce45f79c6f5109f848acd540567
2. Key: d2fe1d99-6298-4af2-8cc5-d97dcf46df30 (will be used to generate secret-key and secret-key-timestamp).The secret-key and secret-key-timestamp have to be generated dynamically. Refer to the link [https://developers.eko.in/docs/authentication](https://developers.eko.in/v3/docs/authentication) for the dynamic secret-key and secret-key-timestamp generation
3. initiator\_id: 9962981729

For testing Bank Account Verification, use the following credentials:
1. developer\_key: becbbce45f79c6f5109f848acd540567
2. Key: d2fe1d99-6298-4af2-8cc5-d97dcf46df30 (will be used to generate secret-key and secret-key-timestamp).The secret-key and secret-key-timestamp have to be generated dynamically. Refer to the link [https://developers.eko.in/docs/authentication](https://developers.eko.in/v3/docs/authentication) for the dynamic secret-key and secret-key-timestamp generation
3. initiator\_id: 9962981729
4. customer\_id: 6111111111

For testing AePS Fund Settlement, use the following credentials:
1. developer\_key: becbbce45f79c6f5109f848acd540567
2. Key: d2fe1d99-6298-4af2-8cc5-d97dcf46df30 (will be used to generate secret-key and secret-key-timestamp).The secret-key and secret-key-timestamp have to be generated dynamically. Refer to the link [https://developers.eko.in/docs/authentication](https://developers.eko.in/v3/docs/authentication) for the dynamic secret-key and secret-key-timestamp generation
3. initiator\_id: 7411111111
4. user\_code : 20310006


> Important Note:
>
> Only IP which is in India will be whitelisted while going on the production mode. IP which is present outside India will not be whitelisted as per compliance
>
> Port 25002 and Port 25004 must be opened on your server in order to reach the requests from your server to our production and staging environment respectively and a connection must be made from your server. You can check if connection is being made from your server or not using the telnet command.


---

# Common APIs

### 1. Transaction Inquiry API

Get the status of a transaction using Eko TID or client_ref_id

#### Details
- **Method:** GET
- **URL Endpoint:** /tools/reference/transaction/{transaction-reference}
- **Request Structure:**
  - Path Parameters:
    - transaction-reference (string / required) - Eko TID or your client_ref_id that uniquely identifies the transaction
  - Query Parameters:
    - initiator_id (number / required) - Your registered mobile number (See Platform Credentials for UAT)
    - user_code (string / required) - Unique code of your registered agent/retailer


#### Description
**Examples:**
To check the status of a transaction using TID 12345, use following endpoint: `/tools/reference/transaction/12345

In case you have not received Eko's TID (say, due to network timeout), you can enquire about the status of a transaction using your own unique reference number (say, 567890) by using the following endpoint format: `/tools/reference/transaction/567890`

**Transaction Timeout:**
A transaction can timeout due to multiple reasons where partner bank responses could be slow or due to network connectivity, delayed or no response may occur.

In such cases transactions should not be treated as declined or failed. Ideally, it should be inquired using the Transaction Inquiry API by passing your own unique reference number i.e. client_ref_id.

Possible values of parameter *tx_status*:
| tx\_status | Description                                  |
| ---------- | -------------------------------------------- |
| 0          | Success                                      |
| 1          | Fail                                         |
| 2          | Response Awaited/Initiated (in case of NEFT) |
| 3          | Refund Pending                               |
| 4          | Refunded                                     |
| 5          | Hold (Transaction Inquiry Required)          |


### 2\. Transaction Status Callback (Webhook)

Provide your own API (webhook) URL to receive updates about the status of following types of transactions: DMT, Fund Transfer, QR, and CMS.

You can configure your own API endpoint (web-hook) to receive callbacks about the status of any transaction. This endpoint will be used by Eko to update the transaction status. We will assume that the update request has been acknowledged upon receiving a status 200 “Success” from the endpoint.

Note - The callback will be sent only when the status of the transaction is updated from initiated to success or failed.

> URL Structure:
>
> Note that only static URL structures are supported.
>
> **URL Structure:** [https://foo.bar/path](https://foo.bar/path)
>
> **Method:** POST

We will send the following payload when we hit your URL:

```json
{
    "tx_status": 0,
    "amount": 120.0,
    "payment_mode": "5",
    "txstatus_desc": "SUCCESS",
    "fee": 5.0,
    "gst": 0.76,
    "sender_name": "Flipkarti",
    "tid": 12971412,
    "beneficiary_account_type": null,
    "client_ref_id": "Settlemet7206124423",
    "old_tx_status": 2,
    "old_tx_status_desc": "Initiated",
    "bank_ref_num": "87694239",
    "ifsc": "SBIN0000001",
    "recipient_name": "Virender Singh",
    "account": "234243534",
    "timestamp": "2019-11-01 18:03:48"
}
```

> Please share the callback URL along with your Eko code on mail
>
> Email ID: [sales.engineer@eko.co.in](mailto:sales.engineer@eko.co.in)
>
> Please note that we set a common callback for QR, CMS, Payout and DMT


---

# User Management APIs

Using the APIs in this section, you can Onboard your users (or, agents) for your application and activate various services for them.

**User Onboarding Flow:**
1.  Get the list of services that you can avail for your users (agents, merchants, distributors, etc) through a single API.
2.  Onboard your user on Eko's platform with the **Onboard User** API and get unique codes for each user.
3.  Activate any service for each user by initiating the **Activate Service** API with a particular service code.
4.  Enquire about the status of your user on Eko's platform by initiating the **User Services Enquiry** API.


### 1. Onboard User API

Onboard your user (agent/merchant/retailer) on the platform. This is required for your users to use services on this platform.

#### Details
- **Method:** PUT
- **URL Endpoint:** /user/account
- **Request Structure:**
  - Body Parameters:
    - initiator_id (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - pan_number (string / required) - Pan Card Number of the agent
    - mobile (string / required) - Verified mobile number of the agent
    - first_name (string / required) - First name of the agent
    - middle_name (string) - Middle Name of the agent
    - last_name (string) - Last name of the agent
    - email (string / required) - Email ID of the agent
    - residence_address (json / required) - Residence Address of the agent
    - dob (date / required) - Date of birth of the agent in YYYY-MM-DD format
    - shop_name (string / required) - Shop name of the agent (Required for AePS onboarding)

#### Sample Response (200 OK)
```json
{
  "response_status_id": -1,
  "data": {
    "user_code": "20110002",
    "initiator_id": "9962981729"
  },
  "response_type_id": 1290,
  "message": "User onboarding successfull",
  "status": 0
}
```

#### Description
It returns a unique **user\_code** for the onboarded agent.

> **Note**
> - While onboarding a user (or, agent) in production, please make sure you are providing correct details.
> - We verify the PAN number provided in this API and match the name returned with the name passed in this API. So, please pass a valid PAN number of the same agent that you are onboarding.
> - Agent onboarding is mandatory for AePS.


### 2. Get User's Services

Get a list of services which are currently enabled for your user.

#### Details
- **Method:** GET
- **URL Endpoint:** /user/account/services
- **Request Structure:**
  - Query Parameters:
    - initiator_id (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - user_code (string / required) - Unique code (registered mobile number) of your agent/retailer

#### Description

Below are the values of status that you will get when the service inquiry is done for any user\_code:

| status | status\_desc | Description                                                         |
| ------ | ------------ | ------------------------------------------------------------------- |
| 0      | Deactivated  | User needs to upload the documents again using activate service API |
| 1      | Activated    | Service Activated for the use                                       |
| 2      | Pending      | User status is pending for this service                             |

> **Note:**
> Once the user uploads the document after the DEACTIVATED state, it will go in the PENDING state.

Check for the parameter "verification\_status" in the user service inquiry API. This parameter will tell you the action taken by our operations team.

These are the values and corresponding descriptions for the parameter `verification_status`:

| verification\_status | Description                                                                                                                     |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| 0                    | Not Applicable (Not Applicable for that service, means no verification is required by the ops team for that particular service) |
| 1                    | Verified (Documents Verified by the ops team)                                                                                   |
| 2                    | Rejected (Rejected by the Ops team, rejection reason can be seen in comments parameter)                                         |
| 3                    | Pending (Action pending from ops team)                                                                                          |

