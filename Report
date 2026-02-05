# Form Submission Abuse via Session-Based Rate Limiting Bypass and Missing Input Validation


# IMPORTANT

>For Privacy and security purposes original domain of the company, in any of its appearances was changed to \<DOMAIN>


# Title 

Form Submission Abuse via Session-Based Rate Limiting Bypass and Missing Input Validation
# Summary

The course registration form endpoint allows unauthenticated submissions with no server-side input validation and applies rate limiting only per session

Additionally, though Cloudflare Turnstile appears to be implemented on the frontend, CAPTCHA tokens are not validated on the server's side. The request is accepted even when the CAPTCHA field is empty

By refreshing the page (otherwise generating new session cookies), the rate limit is immediately reset, allowing unlimited submissions. Arbitrary values are accepted for all fields, allowing spam and inbox flooding

That allows an attacker to send unlimited fake registrations to the company email address

# Affected endpoint

Company's email and
```bash
https://webflow.com/api/v1/form/<form_ID>
```

Origin
```bash
https://<DOMAIN>
```

# Steps to Reproduce

## Submit the arbitrary 

#### Send the following request
```bash
POST /api/v1/form/<form_ID> HTTP/1.1
Host: webflow.com
Content-Length: 1385
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9

Accept: application/json, text/javascript, */*; q=0.01
Sec-Ch-Ua: "Not(A:Brand";v="8", "Chromium";v="144"
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36
Origin: https://<DOMAIN>
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://<DOMAIN>
Accept-Encoding: gzip, deflate, br
Priority: u=1, i
Connection: keep-alive

name=Email+Form&pageId=<form_ID>&elementId=eaf62be8-a5b5-788c-0f66-fd05318abb86&domain=<DOMAIN>=https%3A%2F%2F<DOMAIN>%2F&test=false&fields%5BName%5D=Last+test&fields%5BEmail%5D=test%40gmal.com&fields%5BTelegram%5D=%40test&fields%5Bcf-turnstile-response%5D=0.BxCSUQv_NEnzg4PQ5C3wb93CmeBrpKFuDdZISLNR-o8psM5fMBItJ1VwG67jYQ2ykJ07AKYKMQ4xlYC2VS4hGTTJVweqXsdSGhlm_Xpd6RirOgqKbhrD7k_1I1vq6grxpEyc1LydtvMN9WiDN0KD6Gsybn3Czv7TuIVukVWhO6RkIIq00qPb0RY9XsI5Oy_eZY8axROuvgsapKqQPrLK6oRWHN0IkARMyejmX0FLPjEeaU_D3QjofOIh7Nwo_1S8O9HrZJovLXPW-OC0BXw3L9YdRfUzDOFq-zxa1GzO74FxRR3BqfeRAJIaOGsvD0M510daydD9tSQPp_tDFb-QzG7n-4gQDq4_y3L7VlOQlue8p5Lplq8jDipg64-uPQmWMwLnTFOuiiNoBszElQjIlhiBos55k4meKfPpfkm6Scxi4-wOxbEGCglCT0R_BRm1Z-NlbQ3J5D2dmzLmKLwe6161bh_D7avy_sBB8DDdFhX0ZoY3X6aiR51ZE3QZ49hX3gKJkSmm6ncfe99JJ-FxtLP7dpXV_7WGg_0zTDasrlKsWRyGhVGWSNvxa3g_DsOLER9DqGvnufEX6qnafgB6Y_fm-5tAeewu9DGCbojXeD6qoMJDF0XuPVJ3C7mTmPOIB1IeD_Z-FypOK7SFSPl6KdLqj6lpiMv7d-anchbxe8LUmFz6FKcqZEWv_F0FuRo18yHqjVVqWMbz4loMr8HxMfeyzbg_WBFYLIozz_Gj4luLe9L8scXMZYz1aOvtroFqOUAAmdLL3q1Stn_c1Y7gSSvZwQ9htouSomFr3rahCvoSDMd4tZQeN-AHGkrXDPPSgaTIpJAOW19UDABDC7jxae34f2PSwVzIiQPULmoiIOvHAJhoe62DW9fhBiULgsryUiKTbZzYM0Dy9qk4pdW1RiIFViaUVBnbG9-YjoduQm-RRoDoqGB1PQxJcmQcmNHV.SjFCAuFAUDqU2UU8L8ruFQ.e6eb77223c38405764e57235973490c626f46276deaa45bff8f026b757c4db60&dolphin=false
```
The request succeeds and returns 
``` bash
HTTP 200 OK
```

Invalid values and empty CAPTCHA are accepted and forwarded to the company
#### Observe rate limiting 
Response headers
```bash
X-Ratelimit-Remaining: 9
X-Ratelimit-Limit: 10
```
After 10 requests, submissions are blocked for this session.

## Bypass rate limiting

- Refresh the page ( or delete cookies ) to obtain a new session )
- Repeat the same POST request
- Rate limit is reset
- This can be repeated indefinitely
## Expected result

- CAPTCHA token must be validated server-side
- Server-side validation rejects malformed fields
- Rate limiting persists across session
- Automated submissions are blocked
## Actual result

- CAPTCHA field may be empty and request still accepted
- Arbitrary field values are accepted
- Rate limiting is tied only to session cookies
- Refreshing the page resets the limit
- Possible unlimited submissions
## Impact

An unauthenticated attacker can

- Send unlimited fake registrations
- Flood company gmail inbox
- Perform automated spam attacks
- Degrade availability of email services
- Abuse the form as a relay for malicious contents

>Attack requires no authentication and can be fully automated 
## PoC ( Proof of Concept )

Submit malformed data
```bash
fields[Name]=123
fields[Email]=123@123
fields[Telegram]=123
fields[cf-turstile-]=
```
Refresh the page to reset session and repeat

## Severity 

>High
## CWE
- CWE-307 - Improper Restriction of Excessive Requests
- CWE-20 - Improper Input Validation
- CWE-799 - Improper Control of Interaction Frequency

