---
layout: post
title: "Session based authentication for FortiSIEM"
tags: 
  - fortisiem
  - bash
  - curl
---

I've been doing some work with FortiSIEM lately to automate reporting, while there is an API the documentation around it is pretty light leaving a lot of gaps.
The main issue I had when I first started was that basic authentication just did not work at all however I was able to figure out how to work around that by using FortiSIEM cookies.
I've outlined the process below in `bash` but you can translate this to any language.

If you get stuck please do not hesitate to reach out.

## Requirements
- `bash`
- `curl`
- `jq`

## Authentication
FortiSIEM doesn't just take your password and submit, it downloads a salt first, `SHA1`s the `salt+password` and then submits that. 
From here you will be returned 3 cookies `JSESSIONID`, `s` and `cookiesession1`.
These cookies are required for all requests and they seem to be only valid for 1 API call.

First is getting the salt to do this is pretty easy and requires no authentication.
FortiSIEM is very finicky about `headers` so while it seams weird it works.
```
curl "${HOST}/h5/sec/loginInfo?s=" \
  --fail \
  --silent \
  -H "Accept: application/json, text/plain, */*" \
  -H "Content-Type: text/plain;charset=UTF-8" \
  -d '{"userName":'${USER}'","organization":"super","domain":"Empty"}' \
  | jq -r '.salt'
```

Once we have the salt we need to combine it with our password and then `SHA1SUM` it.
FortiSIEM also requires this to be in all capital letters.
```
echo -n "${SALT}${PASS}" \
  | sha1sum \
  | cut -d' ' -f1 \
  | tr '[:lower:]' '[:upper:]'
```

Lastly we can get our authentication cookies.
We will save them out to `cookies.txt` with `-c` and in future curls we can use this file with `-b cookies.txt`.
```
curl "${HOST}/h5/sec/login?s=" \
  --fail \
  --silent \
  -c cookies.txt \
  -H "Accept: application/json, text/plain, */*" \
  -H "Content-Type: text/plain;charset=UTF-8" \
  -d '{"username":"'${USER}'","password":"'${SALTED_PASS}'","domain":"super","userDomain":"Empty"}' > /dev/null
```
    
## Usage
Once we have a valid `cookie jar` for `curl` we can send off a query to FortiSIEM like so.
```
curl "${HOST}/query/eventQuery" \
  --fail \
  --silent \
  -b cookies.txt \
  -H "Content-Type: text/xml;" \
  -d @"${XML_FILE}}"
```

This will send off async query to FortiSIEM returning the query id if the XML is valid. From here you will have to refill the `cookie.txt` file for every new call.

Next we can check the status of the query, where `100` is finished and ready for requesting data.
```
curl "${HOST}/query/progress/${QUERY_ID}" \
  --fail \
  --silent \
  -b cookies.txt
```

Finally once the query status is `100` we are able to request its data.
FortiSIEM does not let you request all the data at once but requires you to provide a starting `INDEX` and the **NUMBER** of events to request.
This is something that took me some time to figure out as the documentation states this is a **RANGE**.
```
curl "${HOST}/query/events/${QUERY_ID}/${INDEX}/${NUMBER_OF_EVENTS}" \
  --fail \
  --silent \
  -b cookies.txt
```

From here you will be provided with an `XML` file that you can process and extract data. When crafting the query make use of the date ranges and requesting to much data at once can be quite slow.
