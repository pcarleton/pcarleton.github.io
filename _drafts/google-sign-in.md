---
layout: post
title: "Google Sign In and Google Sheets: Step by Step in Python"
---


# Motivation

As part of my [CashCoach][cashcoach] project, I want to share and update spending reports with a user using Google Sheets.

It turns out there a lot of different options for interacting with the Google API's likely because people want to interact with them in all sorts of different scenarios.  They primarily center around 2 decisions you want to make: 1. Do you want to act as if you were the user or have a "robot" user for your project? and 2. Are you trying to use Google as authentication? Or do you have a separate way to verify identities and you want to add API capabilities?

Keeping those two options in mind helped me distinguish between the mountain of examples out there.   Up to this point, I have been running CashCoach through a command line interface and managing my authorization similar to the [Google Sheets Quickstart in Python][sheets-quickstart].   To put this in terms of the 2 questions above, 1. the app is acting as if it were the user, and 2. it is relying on my laptops authentication to restrict access to the "credentials" file, so it is not using Google to authenticate.

 This setup was not ideal for multiple users since it stored the credentials in a single file.  Additionally, I was storing a token which gave access to everything in my Google Drive which is more permissive than I would like.  In order to support multiple users and more granular access control, I decided to switch to a "robot" account and using Google to authenticate.

# Robot Account

The process of setting up a robot account (or "service account", I'll just call it a robot from now on) is detailed very nicely in the documentation for the python library [gspread][gspread-auth].  It is not too different from the flow "on behalf of the user" flow, you just use a different type of "secrets" file, and you don't have to prompt the user before interacting with the API (since your robot can only access its own sheets).   As a result, the robot needs to share whatever document you want with the user for them to see it.  Alternatively, the user could create a document and then share it with the robot account, and then the robot could modify it.  (The service account email is usually something long horrendous, so relying on a casual user to do this seemed like a bad idea.)

With the service account set up, I could make calls to the Sheets API to create a new spreadsheet and then share it with an email like this:

```
TODO:
```

The next step was to get the user to prove they owned the email.

# Google Sign-In

In order to share a spreadsheet with a user, I first needed to know what their Google account email was.  I could just ask them to enter it, but that gives me no evidence that they actually own that email.  Eventually, I want to put private data in the spreadsheet.  For this, I need a way for the user to prove who they are on separate occasions (in other words, authentication).

Fortunately, Google has streamlined this process with their [Google Sign In for Web Apps][google-signin].  Here's the gist: Google provides a javascript library which issues a request to the Google Servers with your app's "Client ID".  This opens the "OAuth 2.0 Dialog" on the user's screen which asks them whether they want to grant permission to this app.  If they select yes, the initial javascript request receives the information about the user along with a token.  This token can then be sent to your server.  Your server then issues a separate request to Google to make sure the token is legit, and then you do your own sign-in logic.





[cashcoach]:https://github.com/pcarleton/cashcoach
[sheets-quickstart]:https://developers.google.com/sheets/api/quickstart/python#step_3_set_up_the_sample
[gspread-auth]:http://gspread.readthedocs.io/en/latest/oauth2.html
[google-signin]:https://developers.google.com/identity/sign-in/web/sign-in
