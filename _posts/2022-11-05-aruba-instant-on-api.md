---
title: "Aruba Instant On API"
layout: post
---

![screenshot]({{ site.baseurl }}/assets/images/aruba-instant-on-api/title.png)

Aruba’s Instant On product line consists of plug and play access points and switches that are managed from the cloud. The cloud management portal – located at [https://portal.arubainstanton.com]() – is free and does not require purchasing any license.

<!--more-->

While it provides an intuitive UI for setting up and managing networks, it is not conducive for automation. For example, if someone is managing many sites and needs to apply policies in bulk to all of them, or monitor their status, it would be very time-consuming and tiresome to do it manually.

Aruba does not provide an API for interacting with their cloud portal. However, an API does exist under the hood and was reverse-engineered in [this](https://mspp.io/documenting-aruba-instant-on-sites-aruba-instant-on-api/) excellent post. It mentions many of the common API endpoints and provides Powershell scripts for communicating with the API and getting data from it.

This blog post builds upon it and extends it to include some more API endpoints as well as posting data to it in order to make configuration changes. A [Postman collection](https://documenter.getpostman.com/view/14413332/2s8YRqkAm1) is also being made available that can be useful for anyone looking to explore the API and use it in their project. Postman provides code for running the API requests in different languages such as PHP, Python, Javascript, etc.

### Using the Postman Collection

Open the companion Postman collection and download it to your system by clicking on “Run in Postman”.

![screenshot]({{ site.baseurl }}/assets/images/aruba-instant-on-api/1.png)

After opening it in Postman go to environment and add your Aruba cloud’s login credentials in it:

![screenshot]({{ site.baseurl }}/assets/images/aruba-instant-on-api/2.png)

The collection is divided into 2 folders. *Authorization* folder contains a series of requests that – when run in sequence – result in the final bearer token that is required to be used with requests in the second folder named *Requests*.

![screenshot]({{ site.baseurl }}/assets/images/aruba-instant-on-api/3.png)

### Getting Data from API

It appears that almost everything that can be done from the web portal can be done via API as well. For example, to fetch site list use the *sites* endpoint:

![screenshot]({{ site.baseurl }}/assets/images/aruba-instant-on-api/4.png)

This provides a count of sites administered by that account and includes various details such as the *id* of each site. The *id* will be used subsequently to identify that site when making site-specific requests.

The *clientSummary* endpoint gives details of clients currently connected to the network:

![screenshot]({{ site.baseurl }}/assets/images/aruba-instant-on-api/5.png)

Similarly, there is an *inventory* endpoint for getting details of the Aruba Instant On hardware on site:

![screenshot]({{ site.baseurl }}/assets/images/aruba-instant-on-api/6.png)

The companion Postman collection includes more endpoints such as *alerts*, *administration*, and *guestPortalSettings*.

### Making Changes via API

Some API endpoints can be used for making changes by issuing requests using HTTP PUT verb. Suppose we want to change the RADIUS server being used for authorizing guests on the network. We will first fetch the existing settings using *guestPortalSettings* endpoint:

![screenshot]({{ site.baseurl }}/assets/images/aruba-instant-on-api/7.png)

We will copy the complete JSON returned in response body, and substitute the new RADIUS address and secret in *serverHost* and *sharedSecret* parameters. Then we will issue a PUT request to the same API endpoint:

![screenshot]({{ site.baseurl }}/assets/images/aruba-instant-on-api/7.png)

### References

[Luke Whitelock – Aruba Instant On API](https://mspp.io/documenting-aruba-instant-on-sites-aruba-instant-on-api/)