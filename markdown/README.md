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


### 2. Transaction Status Callback (Webhook)

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

Use this API to onboard your user (agent/merchant/retailer) on the platform. This is required for your users to use services on this platform.

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


### 2. Get User's Services API

Use this API to get a list of services which are currently enabled for your user.

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


### 3. Get All Services API

Use this API to get a list of all services available on the platform, along with a unique `service_code` for each service. This code can be used to check which services are enabled for a specific user.

#### Details
- **Method:** GET
- **URL Endpoint:** /tools/catalog/service-codes
- **Request Structure:**
  - Query Parameters:
    - initiator_id (string / required) - Your registered mobile number (See Platform Credentials for UAT)


### 4. Activate Service for User API

Use this API to activate a service for your user (agent/retailer/distributor) before they can start using it.

#### Details
- **Method:** PUT
- **URL Endpoint:** /admin/network/agent/{user_code}/service/{service_code}/activate
- **Request Structure:**
  - Path Parameters:
    - user_code (string / required) - User code value of the user from whom the service needs to be activated
    - service_code (string / required) - Unique code of the service which needs to be activated
  - Body Parameters:
    - initiator_id (string / required) - Your registered mobile number (See Platform Credentials for UAT)


#### Description
It is mandatory to activate a service for your users before they can start using it in production. To know the service_code for any service to activate, call the Get All Services API.

Some of the common service codes are:

| Service                         | service_code |
| ------------------------------- | ------------ |
| Vendor Payments (Fund Transfer) | 45           |
| BBPS Bill Payments              | 53           |
| Credit card Bill Payment        | 63           |
| PAN Verification                | 4            |
| Airtel CMS (Cashdrop)           | 58           |
| UPI Static QR                   | 59           |

Note: Some services (like AePS Cash Withdrawal) require additional data or steps for activation. Their service-activation APIs are separately listed within those sections (e.g., Activate AePS Fingpay & Activate AePS Fino).


### 5. Deactivate Service for User API

Use this API to deactivate a service for your user (agent/retailer/distributor).

#### Details
- **Method:** PUT
- **URL Endpoint:** /admin/network/agent/{user_code}/service/{service_code}/deactivate
- **Request Structure:**
  - Path Parameters:
    - user_code (string / required) - User code value of the agent from whom the service needs to be deactivated
    - service_code (string / required) - Unique code of the service which needs to be deactivated
  - Body Parameters:
    - initiator_id (string / required) - Your registered mobile number (See Platform Credentials for UAT)

  
### 6. Get Settlement Account Balance API

Get the current balance (E-value) of your or your user's wallet.

#### Details
- **Method:** GET
- **URL Endpoint:** /user/account/balance
- **Request Structure:**
  - Query Parameters:
    - initiator_id (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - customer_id_type (string / required) - Defaults to mobile_number
    - customer_id (string / required) - Registered mobile number for the wallet (e.g., your registered mobile number)


---

# Customer Management APIs

### 1. Get Customer Information API
Use this API to get a customer's basic profile information such as name, mobile number, eKYC status, wallet balance, monthly available limit for money transfer, etc. Use this API to check if the customer has been created on the platform. If not, create the customer before using other services (like Money Transfer) for the customer.

#### Details
- **Method:** GET
- **URL Endpoint:** /customer/profile/{customer_id}
- **Request Structure:**
  - **Path Parameters:**
    - **customer_id** (string / required) - Customer's mobile number
  - **Query Parameters:**
    - **initiator_id** (string / required) - The unique cell number with which you are onboarded on Eko's platform. For UAT, refer to [Platform Credentials](https://developers.eko.in/docs/platform-credentials)
    - **user_code** (string / required) - User code value of the retailer from whom the request is coming


#### Response Structure
You get the following key information in the `data` object of response:

| Parameter Name         | Description                                           | Use |
| ---------------------- | ----------------------------------------------------- | --- |
| available_limit        | DMT Transfer Limit left in the month for the customer | Show the customer how much more money they can transfer this month |
| used_limit             | Limit consumed in the month by the customer           | Show the customer how much money they have already transferred this month |
| state                  | State of the customer                                 | You can show the state of the customer on your portal |
|                        | **Values the parameter can have**                     | |
|                        | 1 - OTP Verification Pending                          | |
|                        | 2 - OTP Verified Customer, Non-KYC / Rejected         | |
|                        | 3 - KYC Verification Pending                          | |
|                        | 4 - Verified and Full KYC Customer                    | |
|                        | 5 - Name Change Verification Pending                  | |
|                        | 8 - Partial KYC                                       | |
| total_limit            | Total limit available for the customer                | Show the total limit of the customer on your UI |
| wallet_available_limit | Limit available in the Wallet                         | To know whether the transaction will go through the wallet for KYC customer or through a BC pipe, and calculate the commission at your end |
| bc_available_limit     | Total Limit available in the BC Pipes                 | |
| is_registered          | Whether the customer is registered on a particular pipe | 0 means the customer is not registered, 1 means registered on the particular pipe |


#### Description

> **Note:**  
> - If you get the remarks as rejected, you must call the API to create the customer for KYC with the documents for the KYC.
> - If you have initiated a name change of wallet but OTP has not been entered yet, this state means that we have only the Aadhaar card number right now, and the OVD documents have not been updated yet.

**DMT Limits for customer:**
- KYC Customer: Rs 74,500 per month (Temporarily on hold)
- Non-KYC Customer: Rs 25,000 per month


### 2. Onboard Customer API
Use this API to onboard a new customer and enable them for services like DMT (Domestic Money Transfer).

#### Details
- **Method:** POST
- **URL Endpoint:** /customer/account
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - The unique cell number with which you are onboarded on Eko's platform. For UAT, refer to [Platform Credentials](https://developers.eko.in/docs/platform-credentials)
    - **user_code** (string / required) - User code value of the retailer from whom the request is coming
    - **customer_id** (string / required) - Customer's mobile number
    - **name** (string / required) - Name of the customer as per ID
    - **dob** (date / required) - Date of birth of the customer in YYYY-MM-DD format
    - **residence_address** (array of strings / required) - Address of the customer in JSON format

#### Description
The API triggers an OTP to be delivered to the customer. Once the OTP is verified using the Verify Customer OTP API, the customer will be successfully onboarded.

**Important:**
- The name passed in the API must exactly match the name on the ID; otherwise, the transaction will fail.
- OTP will not be delivered to the customer's mobile on UAT systems.


### 3. Verify Customer OTP API
Use this API to verify a customer's mobile number via OTP. The OTP is received when the customer is created or when the OTP is resent using the Resend OTP API.

#### Details
- **Method:** PUT
- **URL Endpoint:** /customer/account/otp/verify
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - The unique cell number with which you are onboarded on Eko's platform. For UAT, refer to [Platform Credentials](https://developers.eko.in/docs/platform-credentials)
    - **user_code** (string / required) - User code value of the retailer from whom the request is coming
    - **otp** (int32 / required) - OTP which you received by calling Create Customer or Resend OTP API.
    - **customer_id** (string / required) - Mobile number of the customer


### 4. Resend OTP to Customer API
Use this API to resend the OTP to the customer for verification.

#### Details
- **Method:** POST
- **URL Endpoint:** /customer/account/otp
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string / required) - User code value of the retailer from whom the request is coming
    - **customer_id** (string / required) - Customer's mobile number


---

# Bill Payment APIs

### 1. Pay Credit Card Bill API
Use this API to pay the credit card bill for a customer. The process includes activating the service for your agent, onboarding your customer, and completing the bill payment by providing necessary details.

#### Details
- **Method:** POST
- **URL Endpoint:** /customer/payment/credit-card-bill
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **recipient_id** (string / required) - The ID that you get after adding a recipient
    - **amount** (string / required) - The payment amount
    - **client_ref_id** (string / required) - Unique reference number of your system, ensure it's as unique as possible to avoid duplication
    - **customer_id** (string / required) - ID generated using the create customer API
    - **channel** (string / required) - Defaults to 2
    
#### Description

> **Credit Card Bill Payment Flow:**
> - Activate the service for your agent by calling the Activate Service for Agent API with service_code = 63.
> - Onboard and verify the customer.
> - Use Domestic Money Transfer APIs to add recipient.
> - Complete the credit card bill payment process by providing the necessary details through the payment API.


### 2. Pay BBPS Bill API
Make payment for utility bills, mobile recharge, etc via BBPS (Bharat Bill Payment System).

#### Details
- **Method:** POST  
- **URL Endpoint:** /customer/payment/bbps
 - **Request Structure:**
    - **Body Parameters:**
      - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
      - **user_code** (string / required) - Unique code (mobile number) of your registered agent/retailer
      - **client_ref_id** (string / required) - A unique ID for every API call generated at your end
      - **utility_acc_no** (string / required) - Account number provided by operator for the bill to be paid
      - **confirmation_mobile_no** (string / required) - Customer's mobile number
      - **sender_name** (string / required) - Name of the customer
      - **operator_id** (string / required) - See the Get Operators API for operator ID
      - **amount** (string / required) - Amount value for the bill/recharge
      - **source_ip** (string / required) - IP address of the agent/retailer making the payment (for security & fraud prevention)
      - **latlong** (string / required) - Your agent's location in latitude,longitude format (for security & fraud prevention)
      - **billfetchresponse** (string) - Needs to be passed only when the value of this parameter is 1 in the Get Operator List API (value from Fetch Bill API)
      - **dob** (string / optional) - Date of birth of the policy holder (required for certain operators like LIC, in DD/MM/YYYY format)
      - **postalcode** (int32 / optional) - Postal PIN code (6-digits, required for certain operators like MSEB)
    - **Headers:**
      - **developer_key** (string / required) - Your static API key (See Guide)
      - **secret-key** (string / required) - Your dynamically generated security key for the request (See Guide)
      - **secret-key-timestamp** (string / required) - The timestamp used to generate the secret-key
      - **request_hash** (string / required) - Hash of the concatenated values: secret_key_timestamp + utility_acc_no + amount + user_code (See Guide)

#### Description

> **What is BBPS?**
> Bharat Bill Payment System (BBPS) is an RBI mandated system which offers integrated and interoperable bill payment services to customers across geographies with certainty, reliability, and safety of transactions.

**Workflow:**
- Activate BBPS Bill Payment service for your agent using the Activate Service for Agent API and passing service code = 53.
- Fetch bill by calling the Get Operator Parameters API to get the required parameters for a particular biller/operator.
- Parameters name passed in the payment API should match exactly as param_name returned in the API.
- The parameter value entered by the user should be verified as per the param_type and regex returned.
- Just before calling the payment API, generate the secret-key-timestamp, secret-key, and request-hash.
- Make payment by calling this API with all the required parameters.

> **Note:**  
> High Commission (Offline): 
> This allows API partners to send fetch bill and pay bill transactions via the new high commission channel, parallel to the existing instant channels. This only works for billers that have the high_commission channel available. The hc_channel is an optional parameter; you can pass its value as 1 to choose the high commissions channel. If not passed, the transaction will be processed through the "instant" channel. High commission transactions may take up to 6 hours to complete on the biller's side.

 

### 3. Fetch BBPS Bill API
Fetch a user's bill for any utility operator.

#### Details
- **Method:** GET
- **URL Endpoint:** /customer/payment/bbps/bill
- **Request Structure:**
  - **Query Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string / required) - User code value of the retailer from whom the request is coming
    - **client_ref_id** (string / required) - Unique transaction ID that you generate for every transaction
    - **utility_acc_no** (string / required) - Account number provided by operator for the bill to be fetched/paid
    - **confirmation_mobile_no** (string / required) - Customer's mobile number
    - **sender_name** (string / required) - Valid name of the customer
    - **operator_id** (string / required) - See the Get Operators API for the operator ID
    - **source_ip** (string / required) - IP of the agent/retailer making the request (for security & fraud prevention)
    - **latlong** (string / required) - Agent's location in latitude,longitude format (for security & fraud prevention)
    - **hc_channel** (int32 / optional, defaults to 0) -
      - 0 = Instant payment
      - 1 = Delayed (offline) payment with higher commissions
    - **dob** (string / optional, defaults to DD/MM/YYYY) - Date of birth of the policy holder (required for LIC)
    - **cycle_number** (string / optional) - Cycle number of the electricity bill (required for MSEB)
    - **authenticator** (string / optional) - Password provided by MSEB (required for MSEB)

#### Description

> **Note:**
> - Some operators may not support the bill fetch.
> - Ensure that the BBPS service is activated for your agent on production before using the APIs.

> **Precautions:**
> - Get the list of parameters required for each operator from the result of the Get Operator Parameters API.
> - The parameter names should be exactly the same as `param_name` specified in the operator info API.
> - The values passed in the request body should adhere to `param_type` and regex specifications.


### 4. Get BBPS Categories API
Get a list of supported categories for utility bill payments, mobile recharge, etc.

#### Details
- **Method:** GET
- **URL Endpoint:** /customer/payment/bbps/categories
- **Request Structure:**
  - **Query Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string / required) - Unique code (mobile number) of your registered agent/retailer


### 5. Get BBPS Locations API
Get a list of supported location IDs for BBPS.

#### Details
- **Method:** GET
- **URL Endpoint:** /customer/payment/bbps/locations
- **Request Structure:**
  - **Query Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string / required) - Unique code (mobile number) of your registered agent/retailer


### 6. Get BBPS Operators API
Get a list of BBPS operators filtered by a category or a location.

#### Details
- **Method:** GET
- **URL Endpoint:** /customer/payment/bbps/operators
- **Request Structure:**
  - **Query Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **category** (int32) - To filter the operator list by a category, pass the "operator_category_id" (use Get Categories API for the available list)
    - **location** (int32) - To filter the operator list by a state, pass the "operator_location_id" (use Get Locations API for the available list)

#### Response Description
You get the following key information in the `data` object of the response:
| Parameter Name            | Description                                                                                          | Use                              |
|---------------------------|------------------------------------------------------------------------------------------------------|----------------------------------|
| `high_commission_channel` | 0 = Instant bill payment; 1 = delayed (offline) bill payment with higher commissions                 | Defines the payment channel type |
| `billFetchResponse`       | If "1", use the Bill Fetch API first, and then pass the "billfetchresponse" parameter from its response to the Bill Pay API | Controls the flow of bill fetch and payment process |


### 7. Get BBPS Operator Parameters API
Get a list of parameters to be passed in the Bill Fetch or Bill Pay APIs for a given operator.

#### Details
- **Method:** GET
- **URL Endpoint:** /customer/payment/bbps/operator/{operator_id}/parameters
- **Request Structure:**
  - **Path Params:**
    - **operator_id** (int32 / required) - The id for the operator (use Get Operators API for the available list)
  - **Query Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)

#### Response Description
You get the following key information for each parameter to be passed:
| Parameter Name     | Description                                                | Use                                |
|--------------------|------------------------------------------------------------|------------------------------------|
| `param_name`       | Name of the parameter to be passed in the Pay Bill API     | Parameter to be included in request |
| `param_label`      | Label to be shown to the user for entering the parameter   | Used for displaying to the user   |
| `param_type`       | Type of the parameter: Numeric, Decimal, AlphaNumeric, or List | Defines the expected input type   |
| `regex`            | Regular expression for a valid value                      | Used to validate the user input   |
| `error_message`    | Error message to be shown if the user input does not match the regex | Provides feedback to the user   |
| `fetchBill`        | 1 = Need to call Fetch Bill API before the Pay Bill API    | Controls the sequence of API calls |
| `BBPS`             | 1 = The biller is provided by Bharat BillPay.              | Indicates BBPS biller for branding |


---

# Initiate Fund Transfer API
Initiate a fund transfer to any bank account.

#### Details
- **Method:** POST
- **URL Endpoint:** /user/payment/fund-transfer
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string / optional) - Unique code (registered mobile number) of your agent/retailer
    - **client_ref_id** (string / required) - Unique transaction ID which you will generate from your end for every transaction. Must be less than 20 characters.
    - **service_code** (int32 / required) - Pass "45" as a fixed value
    - **payment_mode** (int32 / required) - The payment mode you want to use to transfer money (NEFT, IMPS, or RTGS)
    - **recipient_name** (string / required) - Beneficiary name (as in bank records)
    - **account** (string / required) - Beneficiary bank account number
    - **ifsc** (string / required) - IFSC Code of receiver’s bank
    - **amount** (int32 / required) - Amount to transfer
    - **sender_name** (string / required) - Sender’s name
    - **tag** (string / optional) - Payment purpose; eg- Grocery
    - **latlong** (string / optional) - Sender’s location information; eg- 28.78123,72.808912
    - **beneficiary_account_type** (int32 / optional) - Beneficiary's bank account type (1 = Savings Account, 2 = Current Account)
    - **source** (string / required) - Pass "API" as a fixed value


#### Sample Response (200 OK)
```json
{
  "response_type_id": 1329,
  "data": {
    "account": "234243534",
    "client_ref_id": "Settlemet7206123420",
    "ifsc": "SBIN0000001",
    "txstatus_desc": "Initiated",
    "tx_status": 2,
    "amount": 1045.00,
    "tid": 12971397,
    "balance": 35322.2,
    "totalfee": 5.00
  },
  "message": "Transaction initiated successfully",
  "status": 0,
  "response_status_id": 0
}
```

#### Description
**Transaction Flow:**
- Activate this service (only in production) using the Activate Service for Agent API with service_code = 45.
- Use this API to initiate the fund transfer.
- Check the status using Transaction Inquiry API or set up a Transaction Status Callback.

**Setup Callback / Web-Hook:**
It is mandatory to set up your own callback (web-hook) URL using the Transaction Status Callback API to receive asynchronous updates about the transaction status.

##### Payment Mode
| payment_mode | Interpretation |
|--------------|----------------|
| 4            | NEFT           |
| 5            | IMPS           |
| 13           | RTGS           |

##### Beneficiary Account Type
| beneficiary_account_type | Interpretation        |
|--------------------------|----------------------|
| 1                        | Savings Account      |
| 2                        | Current Account      |

##### Response Error Codes
| ERROR CODES | MEANING                          | SOLUTION                                                             |
|-------------|----------------------------------|----------------------------------------------------------------------|
| 403         | Forbidden                        | Regenerate your secret key and timestamp or check if your service is activated |
| 500         | Internal Server Error           | Check if your request URL is correct or the parameters you're passing are correct |
| 415         | Unsupported Media Type          | Re-check the content/type of the request body                       |

**Note:**
Ensure that the `client_ref_id` entered is a unique combination of characters and numbers as it will help uniquely identify a transaction. Maximum length: 20 characters.


---

# UPI Payment APIs


### 1. Validate UPI VPA API
Validate the VPA (Virtual Payment Address) for a UPI Recipient.

#### Details
- **Method:** POST
- **URL Endpoint:** /customer/payment/upi/validate-vpa
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string / required) - User code value of the retailer from whom the request is coming
    - **customer_vpa** (string / required) - VPA which you want to validate
    - **client_ref_id** (string / required) - Unique transaction ID which you will generate from your end for every transaction, it will help you identify your specific transaction

#### Sample Response (200 OK)
```json
{
  "response_status_id": 0,
  "data": {
    "client_ref_id": "62421321704479941883",
    "tx_status": "0",
    "user_code": "1000834",
    "bank_id": 687,
    "fee": "0.0",
    "customer_vpa": "9999999999@upi",
    "recipient_mobile": "9999999999",
    "recipient_name": "Aastha Malik",
    "valid_vpa": 1,
    "tid": "2886242155",
    "timestamp": "Sat Jan 06 00:09:02 IST 2024"
  },
  "response_type_id": 1976,
  "message": "VPA is Valid",
  "status": 0
}
```


### 2. UPI Payment to VPA API

Pay money from your Eko wallet to a VPA ID.

#### Details
- **Method:** POST
- **URL Endpoint:** /customer/payment/upi
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **customer_id** (string / required) - Mobile number of retailer
    - **customer_vpa** (string / required) - Vpa in which you want the amount to be settled
    - **recipient_name** (string / required) - Name of the receiver who is associated with the vpa
    - **amount** (string / required) - Amount that you want to settle
    - **client_ref_id** (string / required) - Unique transaction ID which you will generate from your end for every transaction

#### Sample Response (200 OK)
```json
{
  "response_status_id": 2,
  "data": {
    "client_ref_id": "RIM10011909045679290",
    "tx_status": "2",
    "amount": "205.0",
    "tds": "",
    "balance": "46547.99",
    "fee": "5.9",
    "customer_vpa": "9999999999@upi",
    "recipient_mobile": "",
    "commission": "",
    "recipient_name": "Aastha Malik",
    "tid": "2886428136",
    "timestamp": "Tue Jan 09 14:09:02 IST 2024"
  },
  "response_type_id": 1982,
  "message": "VPA Payment Initiated",
  "status": 0
}
```


---

# UPI Collection APIs 

### 1. Generate Static QR (UPI) API 
Generate a static QR code for any agent to receive payments via UPI into their E-value wallet.

#### Details
- **Method:** POST
- **URL Endpoint:** /user/collection/upi/generate-static-qr
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **sender_id** (string / required) - Registered mobile number of the agent for which the QR code is generated
    - **name** (string / required) - Name of the agent for which QR code is generated
    - **email** (string) - Email address of the agent whose QR code is being generated

#### Response Values
| Values | Status         |
|--------|----------------|
| 0      | INACTIVE       |
| 1      | ACTIVE         |
| 2      | QR_ACTIVE      |
| 3      | QR_INACTIVE    |


#### Sample Response (200 OK)
```json
{
  "response_status_id": -1,
  "data": {
    "client_ref_id": "qr_MAQHGsBlDT218v",
    "name": "xyz",
    "graph_data": "",
    "utility_acc_no": "cust_MAQHG5z2g0k9FK",
    "sender_id": "9876543208",
    "qr_string": "upi://pay?ver=01&mode=01&pa=rpy.qreko262939687622354@icici&pn=Eko&tr=RZPMAQHGsBlDT218vqrv2&cu=INR&mc=4814&qrMedium=04&tn=PaymenttoEko",
    "status": 2
  },
  "response_type_id": 1888,
  "message": "successfully generated QR code",
  "status": 0
}
```

#### Description
Eko’s QR Payment API provides an end-to-end solution from QR code generation to payment collection. The transaction flow is as follows:

1. Activate your service for QR using Activate Service for Agent API and passing `service_code = 59`.
2. The user provides their mobile number that is the `sender_id` against which their QR code is generated.
3. If any payment is made against the QR code, the amount is reflected in the partner's wallet.
4. After the transaction, the callback request with the transaction details is sent to the partner. The transaction status of a particular transaction can be checked using the Transaction Inquiry API.

**Note:**
- To cross-check the QR code that was used for a transaction, check if the `client_ref_id` in the callback matches the `utility_acc_no` in the generated QR. If they are the same, you've identified the QR code used for the transaction.
- For each `sender_id`, only one QR string can be generated.


### 2. Generate Dynamic QR (UPI) API
Generate a Dynamic QR code for any agent to receive payments via UPI into their E-value wallet.

#### Details
- **Method:** POST
- **URL Endpoint:** /user/collection/upi/generate-static-qr
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **sender_id** (string / required) - Registered mobile number of the agent for which the QR code is generated
    - **amount** (int32 / required) - The payment amount to accept via UPI
    - **name** (string / required) - Name of the agent for which QR code is generated
    - **email** (string) - Email address of the agent whose QR code is being generated

#### Sample Response (200 OK)
```json
{
  "response_status_id": -1,
  "data": {
    "client_ref_id": "qr_MAQHGsBlDT218v",
    "name": "xyz",
    "graph_data": "",
    "utility_acc_no": "cust_MAQHG5z2g0k9FK",
    "sender_id": "9876543208",
    "qr_string": "upi://pay?ver=01&mode=01&pa=rpy.qreko262939687622354@icici&pn=Eko&tr=RZPMAQHGsBlDT218vqrv2&cu=INR&mc=4814&qrMedium=04&tn=PaymenttoEko&am=5",
    "status": 2
  },
  "response_type_id": 1888,
  "message": "successfully generated QR code",
  "status": 0
}
```

#### Description
**Note:**
- For this API, almost all process is same as Generate Static QR API, only the difference is that the QR code generated is dynamic in nature and thus an extra parameter 'amount' is added in the request'.
- The api url/endpoint is also same as static QR code generation API.


---

# KYC & Verification APIs

## 1. Bank Account APIs

### 1.1. Bank Account Verification (Penny-Drop) API
Verify a bank account number by transferring ₹1 to retrieve the name of the account holder.**

> **Note:** Not applicable for all banks. Only applicable for banks for whom account verification feature is available. This can be checked by using the Get Bank Details API.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/bank-account/penny-drop
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string / required) - User code value of the retailer from whom the request is coming
    - **customer_id** (string / required) - Registered mobile number of the customer
    - **id_type** (string / required) - It can have 2 values: ifsc or bank_code. For bank_code refer to the bank list attached below
    - **id** (string / required) - need to pass the complete value of IFSC code if ifsc is selected as id_type and bank code if bank_code is selected as id_type
    - **acc_num** (string / required) - pass complete account number which needs to be verified

#### Response Values
| response_status_id | response_type_id | message                                               |
|--------------------|------------------|-------------------------------------------------------|
| 1                  | 1796             | Recipient name not found and verification not available for this bank |
| -1                 | 61               | Success! Account details found                      |

#### Sample Response (200 OK)
```json
{
  "response_status_id": -1,
  "data": {
    "amount": "1.0",
    "fee": "7.00",
    "verification_failure_refund": "7.00",
    "is_Ifsc_required": "0",
    "tid": "2157012435",
    "client_ref_id": "AVS20181123194719311",
    "bank": "Kotak Mahindra Bank",
    "is_name_editable": "1",
    "user_code": "20810200",
    "aadhar": "",
    "recipient_name": "Unknown",
    "ifsc": "",
    "account": "1711654128"
  },
  "response_type_id": 61,
  "message": "The bank did not share the recipient name this time.",
  "status": 0
}
```


### 1.2. Bank Account Verification (Penniless) API
Verify a bank account number without transferring ₹1 to retrieve the name of the account holder.

**Note:** Not applicable for all banks. Only applicable for banks for whom account verification feature is available. This can be checked by hitting the Get Bank Details API.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/bank-account/penniless
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string / required) - User code value of the retailer from whom the request is coming
    - **customer_id** (string / required) - Registered mobile number of the customer
    - **id_type** (string / required) - It can have 2 values: ifsc or bank_code. For bank_code refer to the bank list attached below
    - **id** (string / required) - need to pass the complete value of IFSC code if ifsc is selected as id_type and bank code if bank_code is selected as id_type
    - **acc_num** (string) - pass complete account number which needs to be verified

#### Response Values
| response_status_id | response_type_id | message                                               |
|--------------------|------------------|-------------------------------------------------------|
| 1                  | 1796             | Recipient name not found and verification not available for this bank |
| -1                 | 61               | Success! Account details found                      |

#### Sample Response (200 OK)
```json
{
  "response_status_id": -1,
  "data": {
    "amount": "1.0",
    "fee": "7.00",
    "verification_failure_refund": "7.00",
    "is_Ifsc_required": "0",
    "tid": "2157012435",
    "client_ref_id": "AVS20181123194719311",
    "bank": "Kotak Mahindra Bank",
    "is_name_editable": "1",
    "user_code": "20810200",
    "aadhar": "",
    "recipient_name": "Unknown",
    "ifsc": "",
    "account": "1711654128"
  },
  "response_type_id": 61,
  "message": "The bank did not share the recipient name this time.",
  "status": 0
}
```


### 1.3. Bulk Bank Account Verification (Async) API
Use this API to verify bank account information in bulk.

**Note:** This is an asynchronous API, so you must use the Bulk Bank Account Verification Status API to get the status of each bank account. We do not currently support Deutsche Bank and Fincare Small Finance Bank because they are not live on IMPS.

#### Response Values
| response_status_id | response_type_id | message                                               |
|--------------------|------------------|-------------------------------------------------------|
| -1                 | 61               | Bulk verification started successfully               |
| 1                  | 1796             | Recipient name not found and verification not available for this bank |

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/bank-account/bulk
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string / required) - A unique ID for every API call generated at your end
    - **entries** (array of objects / required) - An array of bank account information, which needed to be verified


### 1.4. Bulk Bank Account Verification Status API
Use this API to get the details of the bulk bank account verification request. You need to enter either the `bulk_reference_id` or `client_ref_id`. If you want to get the status of a single entry, enter the particular reference ID in the request.

#### Details
- **Method:** GET
- **URL Endpoint:** /tools/kyc/bank-account/bulk/status
- **Request Structure:**
  - **Query Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string) - A unique ID for every API call generated at your end
    - **bulk_reference_id** (int32 / required) - The unique ID you receive in the response of Bulk Bank Account Verification API.

#### Response Values
| response_status_id | response_type_id | message                                               |
|--------------------|------------------|-------------------------------------------------------|
| -1                 | 61               | Bulk verification status fetched successfully        |
| 1                  | 1796             | No verification request found                        |

#### Sample Response (200 OK)
```json
{
  "response_status_id": -1,
  "data": {
    "entries": [
      {
        "reference_id": "ref_12345",
        "name_at_bank": "John Doe",
        "bank_name": "ICICI Bank",
        "utr": "ICICI123456789",
        "city": "Mumbai",
        "branch": "Andheri",
        "micr": "400000000",
        "name_match_score": "95%",
        "name_match_result": "Match",
        "account_status": "Verified",
        "account_status_code": "1"
      }
    ]
  },
  "response_type_id": 61,
  "message": "Bulk verification status fetched successfully",
  "status": 0
}
```


### 1.5. IFSC Verification API
Use this API to verify IFSC codes. You will receive the bank name, the branch that it belongs to, the supported transfer modes, and the respective MICR code.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/ifsc
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string / required) - A unique ID for every API call generated at your end
    - **ifsc** (string / required) - The IFSC information of the bank account to be validated. It should be an alphanumeric value of 11 characters. The first 4 characters should be alphabets, the 5th character should be a 0, and the remaining 6 characters should be numerals.
    
#### Response Structure
You get the following key information in the `data` object of the response:
| Name           | Data Type   | Description                                          |
|----------------|-------------|------------------------------------------------------|
| bank           | string      | The name of the bank                                 |
| ifsc           | string      | The IFSC information                                 |
| neft           | string      | The status of NEFT                                  |
| imps           | string      | The status of IMPS                                  |
| rtgs           | string      | The status of RTGS                                  |
| upi            | string      | The status of UPI                                   |
| ft             | string      | The status of fund transfer                         |
| card           | string      | The status of card                                   |
| micr           | integer     | The MICR code that can be used to identify the bank  |
| nbin           | integer     | The National Bank Identification Number (NBIN)       |
| address        | string      | The physical address information of the branch       |
| city           | string      | The city name where the branch is located            |
| state          | string      | The name of the state where the branch is located    |
| branch         | string      | The name of the branch                               |
| ifsc_subcode   | string      | The subcode of the IFSC information                  |
| category       | string      | The category of the bank                             |
| swift_code     | string      | The code that identifies banks and financial institutions worldwide. The code helps pinpoint the specific bank |

#### Sample Response (200 OK)
```json
{
  "response_status_id": -1,
  "data": {
    "bank": "State Bank of India",
    "ifsc": "SBIN0001234",
    "neft": "Available",
    "imps": "Available",
    "rtgs": "Available",
    "upi": "Available",
    "ft": "Available",
    "card": "Available",
    "micr": 123456,
    "nbin": 1234567890,
    "address": "123 Main Street, Mumbai",
    "city": "Mumbai",
    "state": "Maharashtra",
    "branch": "Andheri",
    "ifsc_subcode": "001",
    "category": "Public Sector",
    "swift_code": "SBININBB"
  },
  "response_type_id": 61,
  "message": "IFSC verification successful",
  "status": 0
}
```


## 2. PAN APIs

### 2.1. PAN Verification API
Active PAN Verification service for your agents.

**Note:** You need to call the Activate Service for Agent API for your agents with `service_code = 4` before they can use this API on production. Refer to the FAQs for any issues: [PAN Verification FAQs](https://developers.eko.in/docs/pan-verifiication-queries).

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/verify-pan
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **pan_number** (string / required) - This is the pan number that needs to be verified
    - **purpose** (int32 / required) - By Default, pass value as 1
    - **purpose_desc** (string / required) - Purpose of the PAN verification service
    - **client_ref_id** (string) - A unique ID for every API call generated at your end

#### Sample Response (200 OK)
```json
{
  "response_status_id": -1,
  "data": {
    "pan_number": "BFUPM3499K",
    "aadhaar_seeding_status": "",
    "gender": "M",
    "pan_returned_name": "A Teleservices",
    "last_name": "Teleservices",
    "aadhaar_seeding_status_code": "",
    "middle_name": "",
    "title": "Mr",
    "first_name": "A"
  },
  "response_type_id": 1255,
  "message": "PAN verification successful",
  "status": 0
}
```


### 2.2. PAN Light API
An alternate API to validate PAN information of individuals. The API helps verify the unique identifier, name of the individual, date of birth, and other information that helps in customer onboarding, KYC verification, and other fraud prevention security measures.

**Note:** The name displayed in the response is the name entered in the API request, not the registered name.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/pan-light
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **pan** (string / required) - Unique 10-character alphanumeric identifier of the individual issued by the Income Tax Department. The first 5 should be alphabets followed by 4 numbers and the 10th character should again be an alphabet.
    - **name** (string / required) - Name of the individual as per the PAN information
    - **dob** (string / required) - Date of birth of the individual as per the PAN information. The format is YYYY-MM-DD
    - **client_ref_id** (string) - A unique ID for every API call generated at your end

#### Sample Response (200 OK)
```json
{
  "response_status_id": -1,
  "data": {
    "pan": "ABCDE1234F",
    "name": "John Doe",
    "dob": "1990-01-01",
    "name_match": "Y",
    "dob_match": "Y",
    "pan_status": "E",
    "status": "VALID",
    "aadhaar_seeding_status": "Y",
    "aadhaar_seeding_status_desc": "Linked"
  },
  "response_type_id": 1255,
  "message": "PAN validation successful",
  "status": 0
}
```

#### Response Structure
You get the following key information in the `data` object of the response:

| Name                    | Data Type | Description                                                                 |
|-------------------------|-----------|-----------------------------------------------------------------------------|
| pan                     | string    | The unique 10-character alphanumeric identifier of the individual issued by the Income Tax Department. |
| name                    | string    | The name of the individual as per the PAN information.                      |
| dob                     | string    | The date of birth of the individual as per the PAN information. The format is YYYY-MM-DD. |
| name_match              | string    | The result of name match verification. If the name entered matches with the name present in the PAN information, you receive Y. The possible values are ['Y', 'N', null]. |
| dob_match               | string    | The result of the date of birth verification. If the date of birth of the individual matches with the date of birth present in the PAN information, you receive Y. The possible values are ['Y', 'N', null]. |
| pan_status              | string    | The status of the PAN information. See the table below for details.    |
| status                  | string    | The status of PAN. Possible Values are ['VALID','INVALID'].                 |
| aadhaar_seeding_status  | string    | Whether the individual linked the aadhaar information with PAN. The possible values are ['Y', 'R', 'NA', null]. |
| aadhaar_seeding_status_desc | string  | Additional information of the linking of aadhaar and PAN card.             |

##### Values for `pan_status`
| pan_status | Description                                                                |
|------------|----------------------------------------------------------------------------|
| E          | The entered PAN information is valid.                                      |
| EC         | The entered PAN information exists and is valid but marked as Acquisition. |
| N          | The entered PAN information does not exist in the database.                |
| X          | The entered PAN information has been deactivated.                          |
| F          | The entered PAN information is fake.                                       |
| D          | The entered PAN information has been deleted.                              |
| EA         | The entered PAN information is valid but marked as Amalgamation.           |
| ED         | The entered PAN information is valid but marked as Death.                  |
| EI         | The entered PAN information is valid but marked as Dissolution.            |
| EL         | The entered PAN information is valid but marked as Liquidated.             |
| EM         | The entered PAN information is valid but marked as Merger.                 |
| EP         | The entered PAN information is valid but marked as Partition.              |
| ES         | The entered PAN information is valid but marked as Split.                  |
| EU         | The entered PAN information is valid but marked as Under Liquidation.      |


### 2.3. PAN 360 API
Use this API to verify the PAN information of your customers. You can retrieve more information such as first name, last name, masked Aadhaar number, contact information, and more. Mobile number, email, and address fields will have low fill rate around 5-10% based on the recent changes to the source of the information.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/pan-360
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **pan** (string / required) - Unique 10-character alphanumeric identifier of the individual issued by the Income Tax Department. The first 5 should be alphabets followed by 4 numbers and the 10th character should again be an alphabet.
    - **name** (string / required) - Name of the individual as per the PAN information
    - **dob** (string / required) - Date of birth of the individual as per the PAN information. The format is YYYY-MM-DD
    - **client_ref_id** (string) - A unique ID for every API call generated at your end


#### Response Structure
You get the following key information in the `data` object of the response:

| Name                      | Data Type | Description                                                                 |
|---------------------------|-----------|-----------------------------------------------------------------------------|
| pan                        | string    | The PAN information entered in the API request.                             |
| name_provided              | string    | The name entered in the API request.                                         |
| registered_name            | string    | The registered name as present in the PAN information.                      |
| name_pan_card              | string    | The name as present in the PAN information.                                 |
| first_name                 | string    | The first name as present in the PAN information.                           |
| last_name                  | string    | The last name as present in the PAN information.                            |
| type                       | string    | The type of the PAN issued.                                                 |
| gender                     | string    | The gender of the individual as present in the PAN information.             |
| date_of_birth              | string    | The date of birth of the individual.                                        |
| masked_aadhaar_number      | string    | The masked Aadhaar number of the individual.                                |
| email                      | string    | The email ID of the individual.                                             |
| mobile_number              | string    | The mobile number of the individual.                                        |
| aadhaar_linked             | boolean   | The Aadhaar and PAN link status.                                            |
| address                    | object    | An object containing the address information of the individual.            |
| address.full_address       | string    | The complete address of the individual.                                     |
| address.street             | string    | The street name from the individual's address.                              |
| address.city               | string    | The city name from the individual's address.                                |
| address.state              | string    | The state name from the individual's address.                               |
| address.pincode            | integer   | The PIN code from the individual's address.                                 |
| address.country            | string    | The country name from the individual's address.                             |

#### Sample Response (200 OK)
```json
{
  "response_status_id": -1,
  "data": {
    "pan": "ABCDE1234F",
    "name_provided": "John Doe",
    "registered_name": "John Doe",
    "name_pan_card": "John Doe",
    "first_name": "John",
    "last_name": "Doe",
    "type": "Individual",
    "gender": "M",
    "date_of_birth": "1990-01-01",
    "masked_aadhaar_number": "XXXX-XXXX-1234",
    "email": "john.doe@example.com",
    "mobile_number": "9876543210",
    "aadhaar_linked": true,
    "address": {
      "full_address": "1234 Elm Street, Springfield",
      "street": "Elm Street",
      "city": "Springfield",
      "state": "Illinois",
      "pincode": 62701,
      "country": "USA"
    }
  },
  "response_type_id": 1256,
  "message": "PAN verification successful",
  "status": 0
}
```


### 2.4. Bulk PAN Verification (Async) API
Use this API to verify your customers' PAN information individually or in batches at a time. This is useful when you have to verify a large number of PAN information.

**Note:** This is an asynchronous API, so you must call the Bulk PAN Verification Status API to get the details of all the PAN information.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/pan/bulk
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string) - A unique ID for every API call generated at your end
    - **entries** (array of objects / required) - An array of PAN details for verification. PAN and name should be included. The name parameter is optional.

#### Response Structure
You get the following key information in the `data` object of the response:

| Name            | Data Type | Description                                                                 |
|-----------------|-----------|-----------------------------------------------------------------------------|
| reference_id    | integer   | The unique ID for reference purposes. Use this ID in the Bulk PAN Verification Status API to get the details of all the PAN information. |


### 2.5. Bulk PAN Verification Status API
Use this API to get the status of the Bulk PAN Verification API request.

#### Details
- **Method:** GET
- **URL Endpoint:** /tools/kyc/pan/bulk/status
- **Request Structure:**
  - **Query Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string) - A unique ID for every API call generated at your end
    - **reference_id** (string / required) - The unique ID that you receive in the response of Bulk PAN Verification API.

#### Response Structure
You get the following key information in the `data` object of the response:
| Name             | Data Type | Description                                                                 |
|------------------|-----------|-----------------------------------------------------------------------------|
| count            | integer   | Count of entries in the entries array                                       |
| entries          | array     | List of all PAN informations for bulk verification                          |

##### Structure for `entries` array items:
Each item in the entries array in the data contains the following information:

| Name                    | Data Type | Description                                                                 |
|-------------------------|-----------|-----------------------------------------------------------------------------|
| pan                     | string    | The unique 10-character alphanumeric identifier issued by the Income Tax Department |
| type                    | string    | The type of the PAN issued                                                  |
| reference_id            | string    | The unique ID for reference purposes                                         |
| name_provided           | string    | The name entered in the API request                                          |
| registered_name         | string    | The PAN registered name                                                     |
| valid                   | string    | The status of the PAN card                                                  |
| father_name             | string    | The father's name of the PAN card holder                                    |
| message                 | string    | Details about the success or failure of the API request                     |
| name_match_score        | string    | The score for the name match verification                                   |
| name_match_result       | string    | The result of the name match verification                                   |
| aadhaar_seeding_status  | string    | Whether the individual linked the Aadhaar information with PAN              |
| last_updated_at         | string    | The last updated date                                                       |
| name_pan_card           | string    | The name displayed on the PAN card                                          |
| pan_status              | string    | The status of the PAN card                                                  |
| aadhaar_seeding_status_desc | string | Additional information of the linking of Aadhaar and PAN card              |


## 3. Aadhaar APIs 
Aadhaar card provides a unique identification number to individuals in India. This number is used as a primary identifier for various purposes, including tax compliance.

Aadhaar verification involves verifying the details present on the Aadhaar card with the UIDAI's database to ensure the accuracy and validity of the information provided.

Aadhaar verification is done in the following three steps, executed in the same order:

   - 1.Capture user's consent on your app
   - 2.Dispatch Aadhaar OTP to the user
   - 3.Verify OTP and download the Aadhaar XML containing all the details

### 3.1. Aadhaar Consent API
Use this API to capture the user's consent for Aadhaar verification.

#### Details
- **Method:** GET
- **URL Endpoint:** /tools/kyc/aadhaar/consent
- **Request Structure:**
  - **Query Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string) - User code value of the user/agent from whom the request is coming
    - **is_consent** (string / required) - Y = Yes, N = No
    - **consent_text** (string / required) - The Aadhaar number that you want to verify
    - **realsourceip** (string / required) - IP of the merchant who is making the request.

#### Response Structure
You get the following key information in the `data` object of the response:
| Name               | Data Type | Description                                              |
|--------------------|-----------|----------------------------------------------------------|
| access_key_validity| string    | The validity of the access key, expressed in epoch time |
| access_key         | string    | The access key to be used in the next API for Aadhaar OTP |
| aadhar             | string    | The Aadhaar number that was verified                    |
| message            | string    | Message indicating the result of consent                |

#### Sample Response (200 OK)
```json
{
  "response_status_id": 0,
  "data": {
    "access_key_validity": "1684482300",
    "access_key": "02e46071-c15e-46ab-9787-9a3ce79ac122",
    "aadhar": "914325935548",
    "message": "Consent Accepted"
  },
  "response_type_id": 1617,
  "message": "Karza Aadhaar Consent Signed",
  "status": 0
}
```


### 3.2. Send Aadhaar OTP API
This API sends an OTP from UIDAI to the user's mobile number linked to their Aadhaar number. The OTP is required to fetch the Aadhaar details.

**Note:** Aadhaar Consent is required to send the Aadhaar OTP.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/aadhaar/otp
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string) - User code value of the user/agent from whom the request is coming
    - **source** (string) - Defaults to NEWCONNECT
    - **aadhar** (string / required) - Aadhaar number to be verified
    - **is_consent** (string / required) - Y - Yes , N - No
    - **access_key** (string / required) - Received in response of GET Aadhaar Consent api
    - **caseId** (string / required) - Aadhaar number
    - **realsourceip** (string / required) - IP of the merchant who is making the request.

#### Response Values
| Name                  | Data Type | Description                                           |
|-----------------------|-----------|-------------------------------------------------------|
| access_key_validity   | string    | The validity of the access key, expressed in epoch time |
| access_key            | string    | The access key to be used for further processing     |
| message               | string    | Message indicating the result of sending the OTP    |

#### Sample Response (200 OK)
```json
{
  "response_status_id": 0,
  "data": {
    "access_key_validity": "1684425002",
    "access_key": "3f050691-4619-492a-8c78-a943419f52e9",
    "message": "OTP sent to registered mobile number"
  },
  "response_type_id": 1619,
  "message": "Karza Aadhaar Otp Sent",
  "status": 0
}
```

### 3.3. Fetch Aadhaar Details XML API
This API retrieves the Aadhaar details of a user in XML format.

**Note:** Aadhaar Consent and Send Aadhaar OTP are required to fetch Aadhaar details.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/aadhaar/xml-download
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string) - User code value of the retailer from whom the request is coming
    - **aadhar** (string / required) - Aadhar number you are verifiying
    - **is_consent** (string / required) - Y = Yes, N = No
    - **otp** (string / required) - OTP received on the mobile number linked with aadhaar
    - **share_code** (string) - Random 4 digit code generated on partner's end
    - **access_key** (string / required) - Received in response of GET Aadhaar OTP API
    - **realsourceip** (string) - IP of the merchant who is making the request.

#### Sample Response (200 OK)
```json
{
  "response_status_id": 0,
  "response_type_id": 1621,
  "message": "Aadhaar File Downloaded",
  "status": 0
}
```


## 4. Mobile Verification APIs

### 4.1. Send OTP API
This API sends an OTP to any mobile number in India for verification or consent.

**Note:** OTP SMS will not be actually delivered in UAT. You must test it using the production credential to actually deliver an SMS to a mobile number. In UAT, the response will include an additional `otp` parameter upon success, which can be used to test the Verify OTP API.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/mobile/otp
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string) - Unique code of your registered agent/user
    - **mobile** (string / required) - The mobile number where the OTP has to be sent
    - **intent_id** (string) - Defaults to 1. A numeric ID representing the intent for sending the OTP. See the table above. Default value is 1 for mobile verification.
    - **client_ref_id** (string) - A unique ID for every API call generated at your end

#### Sample Response (200 OK)
```json
{
  "status": 0,
  "response_status_id": 0,
  "response_type_id": 1600,
  "message": "OTP request has been sent",
  "data": {
    "user_code": "20810200",
    "otp_ref_id": "2465238",
    "client_ref_id": "1720690511969403"
  }
}
```

### 4.2. Verify OTP API
This API is used to verify the OTP sent to your agent for AePS KYC.

#### Details
- **Method:** PUT
- **URL Endpoint:** /tools/kyc/mobile/otp/verify
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string) - Unique code of your registered agent/user
    - **mobile** (string / required) - The mobile number to which the OTP was delivered
    - **otp** (string / required) - OTP received on merchant's registered mobile number
    - **otp_ref_id** (string / required) - Received in response of e-KYC OTP Request API

#### Sample Response (200 OK)
```json
{
  "response_status_id": 0,
  "data": {
    "user_code": "20810200",
    "otp_ref_id": "2465238",
    "tid": "3428302170",
    "otp_verification_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJwYXlsb2FkIjoie1widG9rZW5faWRcIjoyNjk4OSxcIm1vYmlsZVwiOlwiODgzOTU3NTQ2OVwiLFwiaW50ZW50X2lkXCI6MTZ9IiwiaXNzIjoiYXV0aDAiLCJleHAiOjM0NTA1NDg5NjcsImlhdCI6MTcyNTI3NDMzM30.-mQfRDWIBpSX4CU5T5QdUyi_ndYY2mhgf_AFobZb9Tk"
  },
  "response_type_id": 1604,
  "message": "Validation successful",
  "status": 0
}
```

### 5. Face Match API
This API is used to match two faces and get the similarity between them.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/face-match
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string) - Unique code of your registered agent/user
    - **mobile** (string / required) - The mobile number to which the OTP was delivered
    - **otp** (string / required) - OTP received on merchant's registered mobile number
    - **otp_ref_id** (string / required) - Received in response of e-KYC OTP Request API

#### Sample Response (200 OK)
```json
{
  "status": 0,
  "requestId": "1dab843f-7319-4c7b-bb57-ee2423456bbd",
  "result": {
    "match": "yes",
    "matchScore": 100,
    "reviewNeeded": "no",
    "confidence": 96.20678573846817
  },
  "statusCode": 101
}
```

## 6. GSTIN APIs

### 6.1 GSTIN Verification API
This API is used to verify if a given GSTIN information exists or not.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/gstin
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string / required) - A unique ID for every API call generated at your end
    - **GSTIN** (string / required) - The unique number assigned to businesses registered under the Goods and Services Tax (GST) system in India.
    - **business_name** (string / required) - Name of the business to which the GSTIN is issued. The maximum character limit is 100

#### Sample Response (200 OK)
```json
{
  "status": 0,
  "response_status_id": 0,
  "response_type_id": 1625,
  "message": "GSTIN Verification Success",
  "data": {
    "GSTIN": "27AAAAA1234A1Z1",
    "additional_address_array": [
      {
        "address": "Some Building, Some Street, Some Location",
        "split_address": {
          "building_name": "Some Building",
          "street": "Some Street",
          "location": "Some Location",
          "district": "Some District",
          "state": "Some State",
          "city": "Some City",
          "flat_number": "12A",
          "latitude": "19.0760",
          "longitude": "72.8777",
          "pincode": "400001"
        }
      }
    ],
    "cancellation_date": "2022-01-01",
    "center_jurisdiction": "Central",
    "constitution_of_business": "Private Limited",
    "date_of_registration": "2021-01-01",
    "gst_in_status": "Active",
    "last_update_date": "2023-01-01",
    "legal_name_of_business": "ABC Pvt Ltd",
    "message": "GSTIN Verified",
    "nature_of_business_activities": "Retail",
    "principal_place_address": "Some Building, Some Street, Some Location",
    "principal_place_split_address": {
      "building_name": "Some Building",
      "street": "Some Street",
      "location": "Some Location",
      "district": "Some District",
      "state": "Some State",
      "city": "Some City",
      "flat_number": "12A",
      "latitude": "19.0760",
      "longitude": "72.8777",
      "pincode": "400001"
    },
    "state_jurisdiction": "Maharashtra",
    "status_code": 1,
    "taxpayer_type": "Regular",
    "valid": true
  }
}
```

### 6.2. Fetch GSTIN with PAN API
This API is used to fetch the list of GSTIN associated with the PAN information.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/gstin-with-pan
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string) - A unique ID for every API call generated at your end
    - **pan** (string / required) - Unique 10-character alphanumeric identifier of the individual issued by the Income Tax Department. The first 5 should be alphabets followed by 4 numbers and the 10th character should again be an alphabet.

#### Sample Response (200 OK)
```json
{
  "status": 0,
  "response_status_id": 0,
  "response_type_id": 1626,
  "message": "GSTIN with PAN Fetch Success",
  "data": {
    "pan": "ABCDE1234F",
    "gstin_list": [
      {
        "gstin": "27ABCDE1234F1Z5",
        "status": "Active",
        "state": "Maharashtra"
      },
      {
        "gstin": "27ABCDE1234F1Z6",
        "status": "Inactive",
        "state": "Karnataka"
      }
    ]
  }
}
```

## 7. Digilocker APIs

### 7.1 Digilocker Consent API
This API is used to create a DigiLocker URL to retrieve and verify Aadhaar information of your customer.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/digilocker
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string / required) - A unique ID for every API call generated at your end
    - **document_requested** (array of strings / required) - Defaults to ["AADHAAR"]. A list of customer documents required for verification. Currently, only "AADHAAR" is supported.
    - **redirect_url** (string / required) - A URL to take the user to after completing the DigiLocker journey. It will contain the verification_id that can be used to get the status of the verification.
#### Sample Response (200 OK)
```json
{
  "status": 0,
  "response_status_id": 0,
  "response_type_id": 1627,
  "message": "DigiLocker URL creation success",
  "data": {
    "reference_id": 123456,
    "url": "https://digilocker.gov.in/link-verification?verification_id=123456",
    "document_requested": ["AADHAAR"],
    "redirect_url": "https://your-redirect-url.com/verification-status?verification_id=123456"
  }
}
```
### 7.2 DigiLocker Verification Status API
This API is used to get the status of the DigiLocker verification.

#### Details
- **Method:** GET
- **URL Endpoint:** /tools/kyc/digilocker/status
- **Request Structure:**
  - **initiator_id** (string, required): Your registered mobile number (See Platform Credentials for UAT).
  - **client_ref_id** (string, optional): A unique ID for every API call generated at your end.
  - **reference_id** (integer, required): The unique ID that you received in the Create DigiLocker URL API response.

#### Sample Response (200 OK)
```json
{
  "status": 0,
  "response_status_id": 0,
  "response_type_id": 1628,
  "message": "DigiLocker Verification Status Retrieved",
  "data": {
    "user_details": {
      "name": "John Doe",
      "dob": "1990-05-15",
      "gender": "Male",
      "eaadhaar": "Available",
      "mobile": "9876543210"
    },
    "document_requested": ["AADHAAR"],
    "document_consent": ["Y"]
  }
}
```
### 7.3 Get Document from DigiLocker API
This API is used to get your customer's document details from DigiLocker.

#### Details
- **Method:** GET
- **URL Endpoint:** /tools/kyc/digilocker/document/{document_type}
- **Request Structure:**
  - **document_type** (string, required): The type of document to be verified. Currently, only "AADHAAR" is supported. Defaults to AADHAAR.
  - **initiator_id** (string, required): Your registered mobile number (See Platform Credentials for UAT).
  - **client_ref_id** (string, required): A unique ID for every API call generated at your end.
  - **reference_id** (string, required): A unique ID that you receive in the response of Create DigiLocker URL API.

#### Sample Response (200 OK)
```json
{
  "status": 0,
  "response_status_id": 0,
  "response_type_id": 1630,
  "message": "DigiLocker Document Retrieved",
  "data": {
    "care_of": "John Doe's Father",
    "dob": "1990-05-15",
    "gender": "Male",
    "name": "John Doe",
    "photo_link": "https://link_to_photo.com",
    "split_address": {
      "country": "India",
      "dist": "Delhi",
      "house": "123",
      "landmark": "Near Central Park",
      "pincode": "110001",
      "po": "New Delhi",
      "state": "Delhi",
      "street": "MG Road",
      "subdist": "Central",
      "vtc": "Delhi"
    },
    "uid": "123456789012",
    "reference_id": 123456
  }
}
```

### 8. Vehicle RC API
This API is used to verify the authenticity of vehicle details. It provides complete information about the vehicle, including the owner, chassis number, registration date, registration number, and more.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/vehicle-rc
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string) - A unique ID for every API call generated at your end
    - **vehicle_number** (string / required) - The registration number of the vehicle

#### Sample Response (200 OK)
```json
{
  "status": 0,
  "response_status_id": 0,
  "response_type_id": 1700,
  "message": "Vehicle RC Details Retrieved",
  "data": {
    "reference_id": 123456,
    "status": "Active",
    "reg_no": "DL1234567890",
    "class": "Sedan",
    "chassis": "ABC1234567890",
    "engine": "XYZ9876543210",
    "vehicle_manufacturer_name": "Toyota",
    "model": "Corolla",
    "vehicle_color": "White",
    "type": "Private",
    "norms_type": "BS6",
    "body_type": "Sedan",
    "owner_count": "1",
    "owner": "John Doe",
    "owner_father_name": "Mr. Doe",
    "mobile_number": "9876543210",
    "rc_status": "Active",
    "status_as_on": "2024-12-16",
    "reg_authority": "Delhi RTO",
    "reg_date": "2022-05-01",
    "vehicle_manufacturing_month_year": "2021-06",
    "rc_expiry_date": "2025-05-01",
    "vehicle_tax_upto": "2025-05-01",
    "vehicle_insurance_company_name": "XYZ Insurance",
    "vehicle_insurance_upto": "2025-05-01",
    "vehicle_insurance_policy_number": "INS123456789",
    "rc_financer": "ABC Bank",
    "present_address": "123, MG Road, Delhi",
    "split_present_address": {
      "district": ["Delhi"],
      "state": [["Delhi"]],
      "city": ["Delhi"],
      "pincode": "110001",
      "country": ["India"],
      "address_line": "123, MG Road"
    },
    "permanent_address": "123, MG Road, Delhi",
    "split_permanent_address": {
      "district": ["Delhi"],
      "state": [["Delhi"]],
      "city": ["Delhi"],
      "pincode": "110001",
      "country": ["India"],
      "address_line": "123, MG Road"
    },
    "vehicle_cubic_capacity": "1800cc",
    "gross_vehicle_weight": "1500kg",
    "unladen_weight": "1200kg",
    "vehicle_category": "Sedan",
    "vehicle_cylinders_no": "4",
    "vehicle_seat_capacity": "5",
    "vehicle_sleeper_capacity": "0",
    "vehicle_standing_capacity": "0",
    "wheelbase": "2600mm",
    "vehicle_number": "DL1234567890",
    "pucc_number": "PUCC123456",
    "pucc_upto": "2025-05-01",
    "blacklist_status": "No",
    "blacklist_details": {},
    "challan_details": {},
    "permit_issue_date": "2023-06-01",
    "permit_number": "PERMIT123",
    "permit_type": "Private",
    "permit_valid_from": "2023-06-01",
    "permit_valid_upto": "2025-06-01",
    "non_use_status": "No",
    "non_use_from": "",
    "non_use_to": "",
    "national_permit_number": "NP123456",
    "national_permit_upto": "2025-06-01",
    "national_permit_issued_by": "Delhi RTO",
    "is_commercial": false,
    "noc_details": {}
  }
}
```

### 9. Driving Licence API
This API is used to verify the driving license of your customer. It retrieves details such as the type of licence, issue date, expiry date, and more.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/driving-license
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string) - A unique ID for every API call generated at your end
    - **dl_number** (string / required) - The driving licence number of the individual for verification.
    - **dob** (string / required) - The date of birth of the individual as present in the driving licence. The accepted format is YYYY-MM-DD.

#### Sample Response (200 OK)
```json
{
  "status": 0,
  "response_status_id": 0,
  "response_type_id": 1800,
  "message": "Driving Licence Details Retrieved",
  "data": {
    "dl_number": "DL1234567890",
    "dob": "1990-01-01",
    "status": "Valid",
    "badge_details": [
      {
        "badge_issue_date": "2021-01-01",
        "badge_no": "BADGE12345",
        "class_of_vehicle": ["Light Motor Vehicle", "Heavy Goods Vehicle"]
      }
    ],
    "dl_validity": {
      "non_transport": {
        "from": "2021-01-01",
        "to": "2031-01-01"
      },
      "hazardous_valid_till": "2031-01-01",
      "transport": {
        "from": "2021-01-01",
        "to": "2031-01-01"
      },
      "hill_valid_till": "2031-01-01"
    },
    "details_of_driving_licence": {
      "date_of_issue": "2021-01-01",
      "date_of_last_transaction": "2024-12-16",
      "status": "Active",
      "last_transacted_at": "2024-12-16",
      "name": "John Doe",
      "father_or_husband_name": "Mr. Doe",
      "address_list": [
        {
          "complete_address": "123, MG Road, Delhi",
          "type": "Permanent",
          "split_address": {
            "district": ["Delhi"],
            "state": [["Delhi"]],
            "city": ["Delhi"],
            "pincode": "110001",
            "country": ["India"],
            "address_line": "123, MG Road"
          }
        }
      ],
      "photo": "https://link_to_photo.com/photo.jpg",
      "cov_details": [
        {
          "class_of_vehicle": "Light Motor Vehicle"
        },
        {
          "class_of_vehicle": "Heavy Goods Vehicle"
        }
      ]
    }
  }
}
```

### 10. Voter ID API
This API is used to verify the authenticity of your customer's voter ID. You need to provide the Electoral Photo Identity Card (EPIC) number, and it retrieves complete details including assembly and parliamentary constituency information.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/voter-id
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string) - A unique ID for every API call generated at your end
    - **epic_number** (string / required) - The unique identification number assigned to each voter ID
    - **name** (string) - The name of the voter ID card holder.

#### Sample Response (200 OK)
```json
{
  "status": 0,
  "response_status_id": 0,
  "response_type_id": 1800,
  "message": "Voter ID Details Retrieved",
  "data": {
    "name": "John Doe",
    "name_in_regional_lang": "जॉन डो",
    "age": "34",
    "relation_type": "Father",
    "relation_name": "Mr. Doe",
    "relation_name_in_regional_lang": "श्री डो",
    "father_name": "Mr. Doe",
    "dob": "1990-01-01",
    "gender": "Male",
    "address": "123, MG Road, Delhi",
    "split_address": {
      "district": ["Delhi"],
      "state": [["Delhi"]],
      "city": ["Delhi"],
      "pincode": "110001",
      "country": ["India"],
      "address_line": "123, MG Road"
    },
    "epic_number": "EPIC1234567890",
    "state": "Delhi",
    "assembly_constituency_number": "42",
    "assembly_constituency": "Delhi East",
    "parliamentary_constituency_number": "12",
    "parliamentary_constituency": "East Delhi",
    "part_number": "001",
    "part_name": "Delhi-01",
    "serial_number": "1234",
    "polling_station": "MG Road Polling Station"
  }
}
```
### 11. Passport API
This API is used to verify passport information (only Indian passports) and ensure the identity of your customer. Provide the passport file number in the request to fetch the details.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/passport
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string) - A unique ID for every API call generated at your end
    - **file_number** (string / required) - The unique alphanumeric code that identifies an individual's passport application.
    - **dob** (string / required) - The date of birth of the passport holder. The format is YYYY-MM-DD.
    - **name** (string) - The name of the passport holder.

#### Sample Response (200 OK)
```json
{
  "status": 0,
  "response_status_id": 0,
  "response_type_id": 1900,
  "message": "Passport Details Retrieved",
  "data": {
    "file_number": "P123456789",
    "name": "John Doe",
    "dob": "1990-01-01",
    "application_type": "Fresh",
    "application_received_date": "2024-01-01"
  }
}
```

### 12. CIN API
This API is used to retrieve information from the Corporate Identification Number (CIN) such as business incorporation date, director(s) details, CIN status, and more.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/cin
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string) - A unique ID for every API call generated at your end
    - **cin** (string / required) - The unique alphanumeric identifier assigned to companies registered under the Ministry of Corporate Affairs, India.

#### Sample Response (200 OK)
```json
{
  "status": 0,
  "response_status_id": 0,
  "response_type_id": 1901,
  "message": "CIN Details Retrieved",
  "data": {
    "cin": "U12345DL1999PTC012345",
    "company_name": "Tech Innovations Pvt Ltd",
    "registration_number": 123456,
    "incorporation_date": "1999-12-01",
    "cin_status": "Active",
    "email": "info@techinnovations.com",
    "incorporation_country": "India",
    "director_details": [
      {
        "dob": "1980-05-15",
        "designation": "Managing Director",
        "address": "123 Tech Park, Delhi, India",
        "din": "123456789012345",
        "name": "John Doe"
      },
      {
        "dob": "1985-09-25",
        "designation": "Director",
        "address": "456 Business Street, Mumbai, India",
        "din": "987654321098765",
        "name": "Jane Smith"
      }
    ]
  }
}
```

### 13. Employee Details API
This API retrieves an individual's recent employment details such as member ID, joining date, and exit date of the company. Verifying the employment information mitigates risk and prevents fraud.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/advance-employment
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string) - A unique ID for every API call generated at your end
    - **phone** (string / required) - The phone number of the employee. (conditional mandatory)
    - **pan** (string) - The PAN information of the employee. (conditional mandatory)
    - **uan** (string) - The unique number assigned to every employee contributing to the Employees' Provident Fund (EPF). (conditional mandatory)
    - **dob** (string) - The date of birth information of the employee. The format is YYYY-MM-DD. Employee date of birth (conditional mandatory)
    - **employee_name** (string) - The name of the employee. (conditional mandatory)
    - **employer_name** (string) - The name of the employer. (conditional mandatory)

#### Sample Response (200 OK)
```json
{
  "status": 0,
  "response_status_id": 0,
  "response_type_id": 1901,
  "message": "Employee Details Retrieved",
  "data": {
    "input": {
      "phone": "9876543210",
      "pan": "ABCDE1234F",
      "uan": "123456789012",
      "dob": "1990-01-15",
      "employee_name": "John Doe",
      "employer_name": "Tech Innovations Pvt Ltd"
    },
    "uan_details": [
      {
        "uan": "123456789012",
        "source": "EPFO",
        "source_score": 95,
        "basic_details": {
          "gender": "Male",
          "dob": "1990-01-15",
          "employee_confidence_score": 90,
          "employee_name": "John Doe",
          "phone": "9876543210",
          "aadhaar_verified": true
        },
        "employment_details": {
          "member_id": "EMP123456",
          "establishment_id": "EST12345",
          "exit_date": "2024-12-01",
          "joining_date": "2020-01-01",
          "leave_reason": "Resigned",
          "establishment_name": "Tech Innovations Pvt Ltd",
          "employer_confidence_score": 90
        },
        "additional_details": {
          "aadhaar": "1234 5678 9012",
          "email": "john.doe@techinnovations.com",
          "PAN": "ABCDE1234F",
          "ifsc": "SBI0001234",
          "bank_account": "9876543210",
          "bank_address": "Tech Innovations Branch",
          "relative_name": "Jane Doe",
          "relation": "Sister"
        }
      }
    ],
    "recent_employment_details": {
      "employee_details": {
        "member_id": "EMP123456",
        "exit_date": "2024-12-01",
        "joining_date": "2020-01-01",
        "uan": "123456789012",
        "epfo": {
          "recent": true,
          "name_unique": true,
          "pf_filings_details": true
        },
        "employed": false,
        "employee_name_match": true,
        "exit_date_marked": true
      },
      "employer_details": {
        "establishment_id": "EST12345",
        "establishment_name": "Tech Innovations Pvt Ltd",
        "setup_date": "2010-06-15",
        "ownership_type": "Private",
        "employer_confidence_score": 90,
        "employer_name_match": true,
        "pf_filing_details": [
          {
            "total_amount": 50000,
            "employees_count": 200,
            "wage_month": "2024-11"
          }
        ]
      }
    }
  }
}
```

### 14. IP API
This API verifies the location, proxy details, city risk score, and proxy type risk score of an IP address.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/ip
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string) - A unique ID for every API call generated at your end
    - **ip_address** (string / required) - The IP address that you need to verify.

#### Sample Response (200 OK)
```json
{
  "status": "success",
  "response_status_id": 0,
  "response_type_id": 2001,
  "message": "IP Verification Success",
  "data": {
    "ip_address": "192.168.1.1",
    "proxy_type": "Residential Proxy",
    "country_code": "IN",
    "country_name": "India",
    "region_name": "Maharashtra",
    "city_name": "Mumbai",
    "city_risk_score": "Low",
    "proxy_type_risk_score": "Medium"
  }
}
```

### 15. Reverse Geocoding API
This API converts geolocation coordinates (latitude and longitude) into readable location information for verification purposes.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/reverse-geocoding
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string) - A unique ID for every API call generated at your end
    - **latitude** (string / required) - The latitude of the geolocation (eg: 28.527554)
    - **longitude** (string) - The longitude of the geolocation (eg: 77.043832)

#### Sample Response (200 OK)
```json
{
  "status": "success",
  "response_status_id": 0,
  "response_type_id": 2001,
  "message": "Reverse Geocoding Success",
  "data": {
    "latitude": "28.527554",
    "longitude": "77.043832",
    "address": "123 Example Street, Example City, Example State",
    "city": "Example City",
    "state": "Example State",
    "statecode": "EX",
    "countrycode": "IN",
    "pincode": "110001",
    "score": 98,
  }
}
```

### 16. Name Match API
This API verifies names that have enormous variations. Provide the names you want to verify, and we will tell you whether they match and provide the reason.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/kyc/name-match
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **client_ref_id** (string) - A unique ID for every API call generated at your end
    - **name_1** (string / required) - The name that you want to verify.
    - **name_2** (string / required) - The name that you want to verify with name_1.

#### Response Description
The response will contain the following information in the `data` object:

| Name          | Data Type | Description                                                      |
|---------------|-----------|------------------------------------------------------------------|
| name_1        | string    | Name that you entered for verification.                          |
| name_2        | string    | Another name that you entered for verification with name_1.     |
| score         | number    | Score of the name match verification.                            |
| reason        | string    | Justification for the match score.                               |

#### Sample Response (200 OK)
```json
{
  "data": {
    "name_1": "John Doe",
    "name_2": "Jon Doe",
    "score": 85,
    "reason": "High similarity, possible variation in spelling."
  },
  "status": "success"
}
```
---

# Marketing & Communication APIs

### Send SMS API
This API sends a promotional text message (SMS) to one or more mobile numbers in India.

#### Details
- **Method:** POST
- **URL Endpoint:** /tools/marketing/sms
- **Request Structure:**
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string) - Unique code of your registered agent/user
    - **mobile** (string / required) - The mobile number to which the SMS has to be sent
    - **text** (string) - The body of the SMS

#### Process
1. **DLT Registration by partner**: The partner needs to complete the DLT registration for the SMS service.
2. **Unique SMS Sender ID registration**: Register a unique SMS Sender ID with a maximum of 5 characters.
3. **DLT SMS registration for each SMS template**: The partner must register each SMS template with DLT.
4. **DLT Registration ID mapping**: The partner’s SMS templates are mapped with the DLT Registration ID by Eko (Complete file must be uploaded at once).
5. **Configure SMS signature with Eko**: The partner needs to configure the SMS signature with Eko, which will be appended at the end of each SMS.

---

# Refunds APIs

## 1. Initiate Refund API
This API is used to safely refund cash to a customer in case their transaction fails.

#### Details
- **Method:** POST
- **URL Endpoint:** /customer/payment/refund/{tid}
- **Request Structure:**
  - **Path Parameters:**
    - **tid** (string / required) - Eko's transaction ID
  - **Body Parameters:**
    - **initiator_id** (string / required) - Your registered mobile number (See Platform Credentials for UAT)
    - **user_code** (string / required) - User code value of the retailer from whom the request is coming
    - **otp** (int32 / required) - OTP shared with customer on the registered mobile number


#### Sample Response (200 OK)
```json
{
  "response_status_id": 0,
  "data": {
    "refund_tid": "2147591637",
    "amount": "5000.00",
    "tds": "7.1",
    "balance": "2.22263731286E9",
    "fee": "50.0",
    "currency": "INR",
    "commission_reverse": "28.38",
    "tid": "13192443",
    "timestamp": "2018-10-30T12:00:14.058Z",
    "refunded_amount": "5050.00"
  },
  "response_type_id": 74,
  "message": "Refund done",
  "status": 0
}
```
#### Description
When a transaction fails, we automatically send an OTP to the customer. Ask for that OTP from the customer and call this API with the OTP. This will act as consent that you have actually refunded the cash to the customer. After this API call, the eValue will be refunded into your account.


### 2. Resend Refund OTP API
This API resends an OTP to the customer to initiate the refund process for a failed transaction.

#### Details
- **Method:** POST
- **URL Endpoint:** /customer/payment/refund/{tid}/otp
- **Request Structure:**
  - **Path Params:**
    - **tid** (int64, required): TID of the transaction for which the OTP needs to be resent.
  - **Body Params:**
    - **initiator_id** (string, required): Your registered mobile number (See Platform Credentials for UAT).


#### Sample Response (200 OK)
```json
{
  "message": "success! OTP has been sent",
  "response_type_id": 169,
  "response_status_id": -1,
  "status": 0,
  "data": {
    "tid": "12212118",
    "otp": "4878925010"
  }
}
```

### 3. Get Pending Refunds API
This API retrieves a list of all failed transactions for a customer that are in a refund-pending state.

#### Details
- **Method:** GET
- **URL Endpoint:** /customer/payment/refunds
- **Request Structure:**
  - **Query Params:**
    - **initiator_id** (string, required): Your registered mobile number (See Platform Credentials for UAT).
    - **user_code** (string, required): User code value of the retailer from whom the request is coming.
    - **customer_id** (string, required): Customer's mobile number.


#### Sample Response (200 OK)
```json
{
  "response_status_id": 0,
  "data": {
    "refund_tid": "2147591637",
    "amount": "5000.00",
    "tds": "7.1",
    "balance": "2.22263731286E9",
    "fee": "50.0",
    "currency": "INR",
    "commission_reverse": "28.38",
    "tid": "13192443",
    "timestamp": "2018-10-30T12:00:14.058Z",
    "refunded_amount": "5050.00"
  },
  "response_type_id": 74,
  "message": "Refund done",
  "status": 0
}
```
#### Description
This API is used to safely refund cash to a customer in case their transaction fails.

- When the transaction fails, we automatically send an OTP to the customer.
- Ask for that OTP from the customer and call this API with the OTP.
- This will act as a consent that you have actually refunded back the cash to the customer. After this API call, we will refund the eValue into your account.

---

# Bank APIs

### 1. Get Banks API

Retrieve a list of banks.

### Details
- **Method:** GET
- **URL Endpoint:** /tools/reference/banks
- **Query Parameters:**
  - **initiator_id** (string, required): Your registered mobile number.
  - **user_code** (string, required): User code value of the retailer from whom the request is coming.

### Sample Response (200 OK)
```json
{
  "response_status_id": 0,
  "data": {
    "isverificationavailable": "0",
    "code": "IDFB",
    "ifsc_status": 4,
    "user_code": "20810200",
    "bank_id": 262,
    "name": "IDFC Bank",
    "available_channels": 0
  },
  "response_type_id": 466,
  "message": "Bank Details Found",
  "status": 0
}
```

### 2. Get Bank Details API
This API retrieves the details of a bank, such as its name, bank code, available channels, whether IFSC is required, and if account verification is supported.

This API fetches details of a bank based on the bank code.

#### Details
- **Method:** GET
- **URL Endpoint:** /tools/reference/bank/{bank_code}
- **Request Structure:**
  - **Path Params:**
    - **bank_code** (string, required): Refer to the bank list for respective bank codes.
  - **Query Params:**
    - **initiator_id** (string, required): Your registered mobile number (See Platform Credentials for UAT).
    - **user_code** (string, required): User code value of the retailer from whom the request is coming.
    - **ifsc** (string, optional): IFSC code of the bank.

#### Description
#### Parameter Details:
- **available_channel**:
  - `0`: All - Both channels IMPS and NEFT are available for the bank.
  - `1`: NEFT - Only NEFT mode is available.
  - `2`: IMPS - Only IMPS mode is available.

- **ifsc_status**:
  - `1`: Bank short-code (e.g., SBIN) works for both IMPS and NEFT.
  - `2`: Bank short-code works for IMPS only.
  - `3`: System can generate logical IFSC for both IMPS and NEFT.
  - `4`: IFSC is required.

- **isVerificationAvailable**:
  - `0`: Bank is not available for account verification.
  - `1`: Bank is available for account verification.



#### Sample Response (200 OK)
```json
{
  "response_status_id": 0,
  "data": {
    "isverificationavailable": "0",
    "code": "IDFB",
    "ifsc_status": 4,
    "user_code": "20810200",
    "bank_id": 262,
    "name": "IDFC Bank",
    "available_channels": 0
  },
  "response_type_id": 466,
  "message": "Bank Details Found",
  "status": 0
}
```

### 3. Get IFSC Details API
This API retrieves the bank and branch details for a given IFSC code.

#### Details
- **Method:** GET
- **URL Endpoint:** /tools/reference/banks/ifsc/{ifsc}
- **Request Structure:**
  - **Path Params:**
    - **ifsc** (string, required): IFSC code of the bank.
  - **Query Params:**
    - **initiator_id** (string, required): Your registered mobile number (See Platform Credentials for UAT).
    - **user_code** (string, required): User code value of the retailer from whom the request is coming.

  
#### Sample Response (200 OK)
```json
{
  "response_status_id": 0,
  "data": {
    "isverificationavailable": "0",
    "code": "IDFB",
    "ifsc_status": 4,
    "user_code": "20810200",
    "bank_id": 262,
    "name": "IDFC Bank",
    "available_channels": 0
  },
  "response_type_id": 466,
  "message": "Bank Details Found",
  "status": 0
}
```
---

# Saved Transactions APIs

### 1. Get Saved Transactions API
This API retrieves a list of all saved transactions for an agent.

#### Details
- **Method:** GET
- **URL Endpoint:** /customer/payment/saved
- **Request Structure :**
  - **Query Params:**
      - **initiator_id** (string, required): Your registered mobile number (See Platform Credentials for UAT).
      - **user_code** (string, required): Unique code of your registered agent/retailer.


### 2. Commit Saved Transactions API
This API commits one or more saved transactions.

#### Details
- **Method:** PUT
- **URL Endpoint:** /customer/payment/saved
- **Request Structure:**
  - **Body Params:**
    - **initiator_id** (string, required): Your registered mobile number (See Platform Credentials for UAT).
    - **user_code** (string, required): Unique code of your registered agent/retailer.


### 3. Cancel Saved Transaction API
This API cancels a saved transaction.

#### Details
- **Method:** DELETE
- **URL Endpoint:** /customer/payment/saved/{tid}
- **Request Structure:**
  - **Path Params:**
    - **tid** (string, required): TID of the saved transaction to cancel.
  - **Body Params:**
    - **initiator_id** (string, required): Your registered mobile number (See Platform Credentials for UAT).
    - **user_code** (string, required): Unique code of your registered agent/retailer.


### 4. Schedule Saved Transaction for Commit API
This API schedules a saved transaction to automatically commit at a later time, if sufficient funds are available.

#### Details
- **Method:** PUT
- **URL Endpoint:** /customer/payment/schedule/{tid}
- **Request Structure:**
  - **Path Params:**
    - **tid** (string, required): TID of the saved transaction to schedule.
  - **Body Params:**
    - **initiator_id** (string, required): Your registered mobile number (See Platform Credentials for UAT).
    - **user_code** (string, required): Unique code of your registered agent/retailer.


### 5. Get Scheduled Transactions API
This API retrieves a list of scheduled transactions for an agent.

#### Details
- **Method:** GET
- **URL Endpoint:** /customer/payment/scheduled
- **Request Structure:**
  - **Query Params:**
    - **initiator_id** (string, required): Your registered mobile number (See Platform Credentials for UAT).
    - **user_code** (string, required): Unique code of your registered agent/retailer.


