---
title: "Using the CRS Sandbox"
#menuTitle: ""
chapter: false
weight: 30
---

## Introducing the CRS Sandbox

We have set up a public **CRS Sandbox** which you can use to send attacks at the CRS. You can choose between various WAF engines and CRS versions. The sandbox parses audit logs and returns our detections in an easy and useful format.

The sandbox is useful for:

- integrators and administrators: you can test out our response in case of an urgent security event, such as the Log4j vulnerability;
- exploit developers/researchers: if you have devised a payload, you can test beforehand if it will be blocked by the CRS and by which versions;
- rule writers: you can quickly check if the CRS catches a (variant of an) exploit without the hassle of setting up your own CRS instance.

## Basic usage

The sandbox is located at https://sandbox.coreruleset.org/.

An easy way to use the sandbox is to send requests to it with `curl`, although you can use any HTTPS client.

The sandbox has many options, which you can change by adding HTTP headers to your request. One is very important so we will explain it first; this is the `X-Format-Output: txt-matched-rules` header. If you add this header to your request, the sandbox will parse the WAF’s output, and return to you the matched CRS rule IDs with descriptions, and the score for your request.

### Example

```bash
curl -H "x-format-output: txt-matched-rules" https://sandbox.coreruleset.org/?file=/etc/passwd

930120 PL1 OS File Access Attempt
932160 PL1 Remote Command Execution: Unix Shell Code Found
949110 PL1 Inbound Anomaly Score Exceeded (Total Score: 10)
980130 PL1 Inbound Anomaly Score Exceeded (Total Inbound Score: 10 - SQLI=0,XSS=0,RFI=0,LFI=5,RCE=5,PHPI=0,HTTP=0,SESS=0): individual paranoia level scores: 10, 0, 0, 0
```

In this example, we sent `?file=/etc/passwd` as a GET payload. The CRS should catch the string `/etc/passwd` which is on our blocklist. Try out the command in a terminal now if you like!

You can send anything you want at the sandbox, for instance, you can send HTTP headers, POST data, use various HTTP methods, et cetera.

The sandbox will return a 200 response code, no matter if an attack was detected or not.

The sandbox will also return a `X-Unique-Id` response header. This contains a unique value that you can use to refer to your request when communicating with us. With `curl -i` you can see the returned headers.

### Example showing returned headers

```bash
curl -i -H 'x-format-output: txt-matched-rules' 'https://sandbox.coreruleset.org/?test=posix_uname()'
HTTP/1.1 200 OK
Date: Tue, 25 Jan 2022 13:53:07 GMT
Content-Type: text/plain
Transfer-Encoding: chunked
Connection: keep-alive
X-Unique-ID: YfAAw3Gq8uf24wZCMjHTcAAAANE
x-backend: apache-3.3.2

933150 PL1 PHP Injection Attack: High-Risk PHP Function Name Found
949110 PL1 Inbound Anomaly Score Exceeded (Total Score: 5)
980130 PL1 Inbound Anomaly Score Exceeded (Total Inbound Score: 5 - SQLI=0,XSS=0,RFI=0,LFI=0,RCE=0,PHPI=5,HTTP=0,SESS=0): individual paranoia level scores: 5, 0, 0, 0
```

## Default options

It’s useful to know that you can tweak the sandbox in various ways. If you don’t send any `X-` headers, the sandbox will use the following defaults.

- The default backend is _Apache 2 with ModSecurity 2.9_.
- The default CRS version is the _latest release version_, currently 3.3.2.
- The default Paranoia Level is 1, which is the least strict setting.
- By default, the response is the full audit log from the WAF, which is verbose and includes unnecessary information, hence why `X-Format-Output: txt-matched-rules` is useful.

## Changing options

Let’s say you want to try your payload on different WAF engines or CRS versions, or like the output in a different format for automated usage. You can do this by adding the following HTTP headers to your request:

- `X-CRS-Version` will pick another CRS version. Available values are `3.3.2` (default), `3.2.1`, `nightly` (which has the latest changes which are not released) and `3.4.0-dev-log4j` (which contains an experimental rule against Log4j attacks).
- `X-CRS-Paranoia-Level` will run CRS in a given paranoia level. Available values are `1` (default), `2`, `3`, `4`.
- `X-Backend: apache` (default) will send the request to **Apache 2 + ModSecurity 2.9**.
- `X-Backend: nginx` will send the request to **Nginx + ModSecurity 3**.
- `X-Backend: coraza` will send the request to **Caddy + Coraza WAF**.
- `X-Format-Output` formats the response to your use-case (human or automation). Available values are:
  - `txt-matched-rules`: human-readable list of CRS rule matches, one rule per line
  - `txt-matched-rules-extended`: same but with explanation for easy inclusion in publications
  - `json-matched-rules`: JSON formatted CRS rule matches
  - `csv-matched-rules`: CSV formatted
  - omitted/default: the WAF’s audit log is returned unmodified as JSON.

Tip: if you work with JSON output (either unmodified or matched rules), `jq` is a useful tool to work with the output, for example you can add `| jq .` to get a pretty-printed JSON, or use `jq` to select and filter the output.

### Advanced examples

Let’s say you want to send a payload to an old CRS version **3.2.1** and choose **Nginx + ModSecurity 3** as a backend, because this is what you are interested in. You want to get the output in JSON because you want to process the results with a script. (For now, we use `jq` to pretty-print it.)

The command would look like:

```bash
curl -H "x-backend: nginx" \
  -H "x-crs-version: 3.2.1" \
  -H "x-format-output: json-matched-rules" \
  https://sandbox.coreruleset.org/?file=/etc/passwd | jq .

[
  {
    "message": "OS File Access Attempt",
    "id": "930120",
    "paranoia_level": "1"
  },
  {
    "message": "Remote Command Execution: Unix Shell Code Found",
    "id": "932160",
    "paranoia_level": "1"
  },
  {
    "message": "Inbound Anomaly Score Exceeded (Total Score: 10)",
    "id": "949110",
    "paranoia_level": "1"
  }
]
```

Let’s say you are working on a vulnerability publication and want to add a paragraph to explain how CRS protects (or doesn’t!) against your exploit. Then the `txt-matched-rules-extended` can be a useful format for you.

```bash
curl -H 'x-format-output: txt-matched-rules-extended' \
  https://sandbox.coreruleset.org/?file=/etc/passwd

This payload has been tested against the OWASP ModSecurity Core Rule Set
web application firewall. The test was executed using the apache engine and CRS version 3.3.2.

The payload is being detected by triggering the following rules:

930120 PL1 OS File Access Attempt
932160 PL1 Remote Command Execution: Unix Shell Code Found
949110 PL1 Inbound Anomaly Score Exceeded (Total Score: 10)
980130 PL1 Inbound Anomaly Score Exceeded (Total Inbound Score: 10 - SQLI=0,XSS=0,RFI=0,LFI=5,RCE=5,PHPI=0,HTTP=0,SESS=0): individual paranoia level scores: 10, 0, 0, 0

CRS therefore detects this payload starting with paranoia level 1.
```

## Capacity

- Please do not send more than 10 requests per second.
- We will try to scale in response to demand.

## Architecture

The sandbox consists of various parts. The frontend that receives the requests runs on Openresty. It handles the incoming request, chooses and configures the backend, proxies the request to the backend, and waits for the response. Then it parses the WAF audit log and sends the matched rules back in the format chosen by the user.

![CRS sandbox diagram v3.drawio.png](https://coreruleset.org/assets/uploads/2022/01/CRS_sandbox_diagram_v3.drawio.png)

There is a backend container for every engine and version. For instance, one Apache with CRS 3.2.2, one with CRS 3.2.1, et cetera... These are normal webserver installations with a WAF and the CRS.

The backend writes their JSON logs to a volume to be read by a collector script and sent to S3 bucket and Elasticsearch.

The logs are parsed, and values like User-Agent and geolocation are extracted. We use Kibana to keep an overview of how the sandbox is used, and hopefully gain new insights about attacks.

## Known issues

In some cases, the sandbox will not properly handle and finish your request.

- _Invalid HTTP requests:_ The frontend, Openresty, is itself a HTTP server which performs parsing of the incoming request. This may not be fully transparent. For instance, it may reject an invalid HTTP request with an error 400 before it can even be sent to a backend. This happens for instance with the Apache 2.4.50 vulnerability that depended on a URL encoding violation. If you receive an error 400, your request was rejected by the frontend.
- _ReDoS_: If your request is a ReDoS attack and makes the backend spend too much time to process a regular expression, this can lead to a timeout from the backend server. Openresty will cancel the request with an error 502. If you have to wait a long time and then receive an error 502, there was likely a ReDoS situation.

## Questions and suggestions

If you have any issues with the CRS sandbox, please open a GitHub issue at [https://github.com/coreruleset/crs-sandbox/issues](https://github.com/coreruleset/crs-sandbox/issues) and we will help you as soon as possible.

If you have suggestions for extra functionality, a GitHub issue will do.

## Working on the sandbox: adding new backends

The following notes are handy for our team maintaining the sandbox.

To add a new backend:

- Each backend has its own IP address.
- docker-compose: copy-paste a back-end container. Give it a new unused IP address in the 10.5.0.\* virtual network.
- The frontend needs to know how to reach the desired backend. There is a hardcoded list in openresty/conf/access.lua with the target IP address.
- httpd-vhosts.conf needs to be changed.
