---
layout: post
title:  "Working with Outlook 365 API"
date:   2019-02-23 19:31:43 +0700
categories: [python, Outlook 365 API]
---

<br>

<br>

This tutortial will walk you through how to setup and work with the Outlook API v2.

<br>

<ol start="1"><li><b>Prerequites</b></li></ol>

<ul><li>Python 3</li>
<li>Libraries: O365</li></ul>

<br>

<ol start="2"><li><b>General Overview</b></li></ol>

Credit goes to <a href='https://github.com/O365/python-o365' target='_blank'>janscas</a> for writing the O365 python library.

I am writing this tutorial because I was not happy with their setup and walkthrough. With the API change from v1 to v2, basic authentication is no longer allowed. Using the latest O365 library helps with the authentication.

<br>

<br>

<ol start="3" id="setup"><li><b>Setup</b></li></ol>

<ul><li>Login at <a href='https://apps.dev.microsoft.com/' target='_blank'>Microsoft Application Registration Portal</a></li>
<li>Create an app, note your app id (client_id)</li>
<li>Generate a new password (client_secret) under "Application Secrets" section</li>
<li>Under the "Platform" section, add a new Web platform and set "https://outlook.office365.com/owa/" as the redirect URL</li>
<li>Under "Microsoft Graph Permissions" section, add the delegated permissions you want</li></ul>

<br>

<ol start="4" id="setup-code"><li><b>Setup Code</b></li></ol>

This part confused me. Once you run the below setup code, you will be given a URL to click. This will take you to Microsoft's login; after logging in to your account, you will be redirected to your Outlook mailbox. <b>Look at the URL in the browser, the token is appended to the end of the URL.</b> This is what you need to copy.

<br>

{% highlight python %}
from O365 import Connection

credentials = ('your_app_id', 'your_client_id_or_pasword')

scopes = ['offline_access','https://graph.microsoft.com/Mail.ReadWrite', 'https://graph.microsoft.com/Mail.Send'] #Change as needed, but ensure to keep offline_access
con = Connection(credentials, scopes=scopes)
url = con.get_authorization_url()

print(url)
result_url = input('Paste the result url here...') #Copy and paste the token, then press enter into the console
con.request_token(result_url)
{% endhighlight %}

<br>

<br>

<ol start="5" id="final-code"><li><b>Sending Email Code</b></li></ol>

Once you are setup, the token will automatically refresh for you. You can begin sending emails or work with the API as needed. The default email body setting is HTML, so you do not need to change settings to send HTML emails.

<br>

{% highlight python %}
from O365 import Account

credentials = ('your_app_id', 'your_client_id_or_pasword')

account = Account(credentials=credentials)
m = account.new_message()
m.to.add(str(toaddress))
m.subject = 'Your Subject'
m.body = 'html'
m.send()
{% endhighlight %}
