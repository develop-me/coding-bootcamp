## Intro
Rather than relying on phpmail() or equivalent, developers should use an email API service to improve the deliverability of mail. In the long run this is simpler than configuring mail settings yourself and outsources mail delivery to mail specialists. Mailgun is a service that provides a simple API for sending and receiving mail. Once you have registered your site domain with them, and configured your dns settings to approve the service, you can then make calls to the Mailgun API and it will send emails from your domain - expertly configured for maximum deliverability.

The following document outlines how to get setup with Mailgun.  

To begin, sign up for and verify a mailgun account. Make a note of your Private API key. This can be found at:  
<a href="https://app.mailgun.com/app/dashboard">https://app.mailgun.com/app/dashboard</a>

## Mailgun API

### Sandbox Domains
For testing, a default sandbox domain comes with your account. You can find the domain at:   
<a href="https://app.mailgun.com/app/domains">https://app.mailgun.com/app/domains</a>

Sandbox domains only send to email that are verified. To validate an email for testing, go to Email Validations > Test Validation:  
<a href="https://app.mailgun.com/app/email_validations">https://app.mailgun.com/app/email_validations</a>

### Production Domains
Out of sandbox, you will require access to the site DNS provider - to verify your ownership of the production domain and permit Mailgun to send mail from it. Go to:  
<a href="https://app.mailgun.com/app/domains">https://app.mailgun.com/app/domains</a>  
and create a new domain record. It is recommended to use a subdomain of your site domain (e.g. mail.your.site.co.uk). On adding a domain you will be redirected to a page of settings. Provide the records in Domain Verification and DNS to your dns provider and allow 24-48hours to propagate.

### Code Example
There is a Mailgun PHP SDK with methods for easily interacting with the API. In your project, Install Composer: 
```bash
curl -sS https://getcomposer.org/installer | php
```
Install SDK:
```bash
composer require mailgun/mailgun-php kriswallsmith/buzz nyholm/psr7
```
Code:
```php 
// Include the Autoloader
require 'vendor/autoload.php';
use Mailgun\Mailgun;

// Instantiate the client.
$mgClient = new Mailgun('your-api-key');
$domain = "your-domain.mailgun.org"; // this should match the domain you have setup in the dashboard

// Make the call to the client.
$result = $mgClient->sendMessage("$domain",
  	array(
  		'from'    => 'Some recipient <recipient@your-domain.mailgun.org>',
    	'to'      => 'Me <petet@lunar.build>', // a verified email address
    	'subject' => 'Hello',
    	'text'    => 'Testing some Mailgun mail!')
    );

echo $result;
```

## Mailgun for Wordpress
There is a simple, well supported plugin that allows you to use Mailgun over HTTP or SMTP. This will take over your mail settings and allow you to use wp_mail() as usual.  
<a href="https://wordpress.org/plugins/mailgun/">https://wordpress.org/plugins/mailgun/</a>  
Setup by registering your domain above and providing that and your api key.
