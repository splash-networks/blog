---
title: "Google Workspace Login for Your Web Application"
description: "Step-by-step guide to adding Google Workspace login to your web application using an OAuth consent screen and an OAuth client ID."
layout: post
image: /assets/images/google-workspace/title.png
last_modified_at: 2023-03-26
tags: [google-workspace]
---

![Google Workspace logo with Gmail, Calendar, Drive and Meet icons above a red Login button](../assets/images/google-workspace/title.png)

You can use the following steps for integrating Google Workspace login into your web application.

<!--more-->

Go to [Google API Console](https://console.cloud.google.com/apis)

Select OAuth consent screen, and select the User Type *Internal*.

![Google Cloud OAuth consent screen with the Internal user type option highlighted](../assets/images/google-workspace/1.png)

After clicking Create, enter information related to your application. Under App Information, enter *App name* and *User support email*. Under Authorized domains add your domain. Under Developer contact information add the email addresses that will receive notifications from Google.
On the next screen there is no need to change any scopes, so click on Save and Continue. Review the summary of what you entered.

Next, go to the Credentials section and click on Create Credentials.

![Google Cloud Credentials page with the Create Credentials button highlighted](../assets/images/google-workspace/2.png)

In the dropdown, select *OAuth client ID*. Under Application type, select *Web Application*. In Authorized redirect URIs enter the URI where Google should redirect the user after successful login.

On the next screen, note your *Client ID* and *Client Secret* and store them securely.

After this you need to integrate Google login into your web application. There are libraries for various languages and frameworks that you can refer to. For PHP you can use [Google API PHP Client](https://github.com/googleapis/google-api-php-client).

### Reference

[Login with Google Account using PHP](https://www.codexworld.com/login-with-google-api-using-php/)