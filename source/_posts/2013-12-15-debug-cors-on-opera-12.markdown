---
layout: post
title: "Debug CORS on Opera 12"
date: 2013-12-15 13:59:46 -0800
comments: true
categories: [Web]
---

Today, I encountered a strange problem with [CORS](www.w3.org/TR/cors/)
(cross-origin resource sharing) on Opera 12. (we have been porting our
JS application to work on a device that runs Opera 12)

We have a central Authentication service that authenticates users
and this application is one of the clients. The symptom is when this
application talks to the Authentication service (via a POST request) it
keeps getting a zero HTTP status code. In generally, when a browser returns a
zero status code, it means the browser cancelled the request for some reason.

Why could Opera cancel a request to the Authentication service? (Note
that our application works fine on Chrome). Upon reading docs of CORS, I
realize that certain CORS requests (say POST) require a **preflight** request
before actually issuing the real request. Maybe it is this "preflight"
request that keeps failing and it prevents Opera from issuing a real
POST request?

To see the traffic between the application and the Authentication service, I ran
a "nodejs proxy" between them which simply logs all the requests and responses.
The logs confirm that Opera only sends a preflight request but doesn't send a
real POST request.

Why? Is it possible that the Authentication service doesn't implement the CORS
standard properly? Let's check the related headers. The following are the
response headers related to CORS:

* Access-Control-Allow-Origin
* Access-Control-Max-Age
* Access-Control-Allow-Credentials
* Access-Control-Allow-Methods
* Access-Control-Allow-Headers

The Authentication service returns the following headers:

* Access-Control-Allow-Origin: the-origin-of-our-app
* Access-Control-Max-Age: this header isn't set
* Access-Control-Allow-Credentials: true
* Access-Control-Allow-Methods: POST, GET, OPTIONS
* Access-Control-Allow-Headers: a lot of headers including Authentication

It seems all the headers make sense. Why does our app work on Chrome, but not
Opera? Eventually the HTTP status code for the preflight request caught my eyes.
The Authentication service returns **204** for the preflight request. Accordingly to
HTTP standard, **2xx** codes generally mean success. **However, all previous versions
of the CORS standard (http://www.w3.org/TR/cors/) except the latest (12/05/2013)
mandate only 200 means success.** In other words, 204 would mean a failure, which
is why Opera believes the preflight request failed and refuses to send a real
POST request.

Opera is obviously still complying with the previous versions of the
CORS standard, which is reasonable considering the latest version of the CORS
standard only came out a few days ago. It takes time to catch up with
the ever-evolving CORS standard.

Chrome, on the other hand, seems to treat any 2xx codes as success. Either they
are extremely up to date or they've always treated 2xx codes as success.

The moral of the story is the specs of a standard is every important and
contains pretty much everything you need. Read it carefully.
