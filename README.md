# SimpleSAMLphp Configuration
SimpleSAMLphp is an open-source PHP authentication application that provides support for SAML 2.0 as a Service Provider (SP) or Identity Provider (IdP).

SAML (Security Assertion Markup Language) is a secure XML-based communication mechanism for exchanging authentication and authorization data between organizations and applications. It’s often used to implement Web SSO (Single Sign On). This eliminates the need to maintain multiple authentication credentials across multiple organizations. Simply put, you can use one identity, like a username and password, to access multiple applications.

## Prerequisites
To continue with SimpleSAMLphp configuration, you have to these things in your device.
- Linux Operating System: Ubuntu 18.04/ Ubuntu 20.04
- Apache Web Server: [Securing Apache Web Server](https://github.com/buckbeak99/Securing-Apache-Server)
- PHP 
## Download and Installing SimpleSAMLphp
Download SimpleSAMLphp from the project’s website: [SimpleSAMLphp.org](https://simplesamlphp.org/download/)  
After downloading the tar.gz file, go to the download directory and extract it using the command:  
```bash
tar xzf simplesamlphp-1.19.5.tar.gz
```
then move the SimpleSAMLphp to the `/var/www/example.com/` directory. Please, at first setting your [apache server](https://github.com/buckbeak99/Securing-Apache-Server). Then install it.  
```bash
mv simplesamlphp-x.y.z simplesamlphp
```
copy to `/var/www/example.com/` directory
```bash
sudo cp -r simplesamlphp /var/www/example.com/
```
### Configuring Apache to serve SimpleSAMLphp
Add these lines in your virtualhost file. Open `example.com.conf` file and add this in the `IfModule` section after `DocumentROOT /var/www/example.com`.
```bash
sudo nano /etc/apache/sites-available/example.com.conf
```
Add these lines..
```bash
SetEnv SIMPLESAMLPHP_CONFIG_DIR /var/www/example.com/simplesamlphp/config

Alias /simplesaml /var/www/example.com/simplesamlphp/www

<Directory /var/www/example.com/simplesamlphp/www>
     Require all granted
</Directory>
```
### Configuring SimpleSAMLphp
- Go to SimpleSAMLphp directory.
```bash
cd simplesamlphp
```
- Update your package list
```bash
sudo apt update
```
- Then install the paackages
```bash
sudo apt install php-xml php-mbstring php-curl php-memcache php-ldap memcached
```
- Restart Apache
```bash
sudo systemctl restart apache2
```
- Go to Config directory
```bash
cd config
```
Open config.php file
```bash
sudo nano config.php
```
Set the administrator password by locating the `auth.adminpassword` line and replacing the default value of 
123 with a more secure password. This password lets you access some of the pages in your SimpleSAMLphp 
installation web interface:  
```bash
'auth.adminpasswoed' => 'MyNewPass5!',
```
Next, set a secret salt, which should be a randomly-generated string of characters.
Some parts of SimpleSAMLphp use this salt to create cryptographically secure hashes. 
You’ll get errors if the salt isn’t changed from the default value.  
```bash
'secretsalt' => 'MyNewPass5!',
```
Then set the technical contact information. This information will be available in the generated metadata, 
and SimpleSAMLphp will send automatically-generated error reports to the email address you specify. 
Locate the following section:
```bash
'technicalcontact_name'     => 'Administrator',
'technicalcontact_email'    => 'na@example.org',
```
Replace `Administrator` and `na@example.org` with your name and email.  

Then set the timezone you would like to use. Locate this section:  
```bash
'timezone' => null,
```
Replace null with a preferred time zone from this list of timezones for PHP. Be sure to enclose the value in quotes.
Save and close the file. You should now be able to access the site in your browser by visiting `https://your_domain/simplesaml`. 
You’ll see the following screen in your browser:
![simplesamlphpinstallation](https://user-images.githubusercontent.com/43216053/171479139-d3a6e30b-e5f6-4971-94f0-9f20fa2d4c79.png)

To make sure your PHP installation meets all requirements for SimpleSAMLphp to run smoothly, 
select the Configuration tab and click on the Login as administrator link. 
Then use the administrator password you set in the configuration file in Step 3.  

Once logged in, you’ll see a list of required and optional PHP extensions used by SimpleSAMLphp. 
Check that you have installed every extension except predis/predis:  
![checking-your-application](https://user-images.githubusercontent.com/43216053/171479637-517a832c-444d-4a07-88df-6d3e50a3c07d.png)

If there are any required components missing, review this tutorial and install the missing components before you move on.

You’ll also see a link that says Sanity check of your SimpleSAMLphp setup. Click this link to get a list of checks applied to your setup to see whether they are successful.
### Enabling and disabling modules
If you want to enable some of the modules that are installed with SimpleSAMLphp, but are disabled by default, you can do that in the configuration:
```bash
 'module.enable' => [
         'exampleauth' => true, // Setting to TRUE enables.
         'saml' => false, // Setting to FALSE disables.
         'core' => null, // Unset or NULL uses default for this module.
    ],
```
 Set to true the modules you want to enable, and to false those that you want to disable.

Prior to SSP V2 you could enable or disable modules by setting empty files with names ( enable , disable , default-enable ) in the module's root directory. You need to now use the module.enable config option.  

This is the same configuration for both SP and IdP.
- [Check how to configure SimpleSAMLphp as a Service Provicer](Identity-Provider.md)
- [Check how to configure SimpleSAMLphp as a Identity Provider](https://github.com/buckbeak99/SimpleSAMLphp/blob/7afad291d4935e9734d0487227102fe17fa0a2ce/Service-Provider.md)
