# SimpleSAMLphp as an Identity Provider
This guide will describe how to configure SimpleSAMLphp as an identity provider (SP). You should previously have installed SimpleSAMLphp as described in the [SimpleSAMLphp installation instructions](https://github.com/buckbeak99/SimpleSAMLphp/blob/553616106079a137894270d614abc2ff110dd595/README.md)  
- [Prerequisite](https://github.com/buckbeak99/SimpleSAMLphp/blob/553616106079a137894270d614abc2ff110dd595/README.md)
- [Securing Apache Web Server](https://github.com/buckbeak99/Securing-Apache-Server)
- [Download and Install SimpleSAMLphp](https://github.com/buckbeak99/SimpleSAMLphp/blob/553616106079a137894270d614abc2ff110dd595/README.md)
- [Configuring Apache to serve SimpleSAMLphp](https://github.com/buckbeak99/SimpleSAMLphp/blob/553616106079a137894270d614abc2ff110dd595/README.md)
- [Configuring SimpleSAMLphp](https://github.com/buckbeak99/SimpleSAMLphp/blob/553616106079a137894270d614abc2ff110dd595/README.md)
- [Enanbling and Disabling Modules](https://github.com/buckbeak99/SimpleSAMLphp/blob/553616106079a137894270d614abc2ff110dd595/README.md)
## Enabling the Identity Provider functionality
The first that must be done is to enable the identity provider functionality. This is done by editing config/config.php . The option enable.saml20-idp controls whether SAML 2.0 IdP support is enabled. Enable it by assigning true to them:
```bash
 'enable.saml20-idp' => true,
```
## Configuring the authentication module 
The `exampleauth:UserPass` authentication module is part of the exampleauth module. This module isn't enabled by default, so you will have to enable it. In `config.php` , search for the `module.enable` key and set exampleauth to `true`:
```bash
 'module.enable' => [
         'exampleauth' => true,
         …
    ],
 ```
The next step is to create an authentication source with this module. An authentication source is an authentication module with a specific configuration. Each authentication source has a name, which is used to refer to this specific configuration in the IdP configuration. Configuration for authentication sources can be found in `config/authsources.php`.  

In this setup, this file should contain a single entry: 
```bash
<?php
$config = [
  'example-userpass' => [
    'exampleauth:UserPass',
    'student:studentpass' => [
      'uid' => ['student'],
      'eduPersonAffiliation' => ['member', 'student'],
    ],
    'employee:employeepass' => [
      'uid' => ['employee'],
      'eduPersonAffiliation' => ['member', 'employee'],
    ],
  ],
];
```
This configuration creates two users - `student` and `employee` , with the passwords `studentpass` and `employeepass` . The username and password are stored in the array index ( `student:studentpass` for the student -user). The attributes for each user are configured in the array referenced by the index. So for the student user, these are: 
```bash
[
  'uid' => ['student'],
  'eduPersonAffiliation' => ['member', 'student'],
],
```
The attributes will be returned by the IdP when the user logs on.   
##  Creating a self signed certificate 
The IdP needs a certificate to sign its SAML assertions with. Here is an example of an openssl -command which can be used to generate a new private key and the corresponding self-signed certificate. The private key and certificate go into the directory defined in the certdir setting (defaults to `cert/` )

This key and certificate can be used to sign SAML messages: 
```bash
  openssl req -newkey rsa:3072 -new -x509 -days 3652 -nodes -out example.org.crt -keyout example.org.pem
```
The certificate above will be valid for 10 years. 
##  Configuring the IdP 
 The SAML 2.0 IdP is configured by the metadata stored in `metadata/saml20-idp-hosted.php`. This is a minimal configuration:
 ```bash
 <?php
  $metadata['__DYNAMIC:1__'] = [
      /*
       * The hostname for this IdP. This makes it possible to run multiple
       * IdPs from the same configuration. '__DEFAULT__' means that this one
       * should be used by default.
       */
      'host' => '__DEFAULT__',

      /*
       * The private key and certificate to use when signing responses.
       * These can be stored as files in the cert-directory or retrieved
       * from a database.
       */
      'privatekey' => 'example.org.pem',
      'certificate' => 'example.org.crt',

      /*
       * The authentication source which should be used to authenticate the
       * user. This must match one of the entries in config/authsources.php.
       */
      'auth' => 'example-userpass',
  ];
 ```
  For more information about available options in the idp-hosted metadata files, see the [IdP hosted reference](https://simplesamlphp.org/docs/latest/simplesamlphp-reference-idp-hosted.html) . 
  ## Using the uri NameFormat on attributes 
  The interoperable SAML 2 profile specifies that attributes should be delivered using the `urn:oasis:names:tc:SAML:2.0:attrname-format:uri NameFormat`. We therefore recommended enabling this in new installations. This can be done by adding the following to the saml20-idp-hosted configuration:  
  ```bash
   'attributes.NameFormat' => 'urn:oasis:names:tc:SAML:2.0:attrname-format:uri',
  'authproc' => [
      // Convert LDAP names to oids.
      100 => ['class' => 'core:AttributeMap', 'name2oid'],
  ],
 ```
 
## Adding SPs to the IdP
The identity provider you are configuring needs to know about the service providers you are going to connect to it. This is configured by metadata stored in `metadata/saml20-sp-remote.php` . Go to your service providers simplesamlphp's installation page and open federation tab. Here you will get the metadata for that specific SP. Then copy it and paste it in the idp file. Find `metadata` directory in simplesaml application. Open `saml20-sp-remote.php` with an editor and paste the SP's metadata. This is a minimal example of a `metadata/saml20-sp-remote.php` metadata file for a SimpleSAMLphp SP: **Don't copy this, it's just an example**
```bash
$metadata['https://yourhostname/simplesaml/module.php/saml/sp/metadata.php/your-host-name'] = [
    'SingleLogoutService' => [
        [
            'Binding' => 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect',
            'Location' => 'https://yourhostname/simplesaml/module.php/saml/sp/saml2-logout.php/syour-host-name',
        ],
    ],
    'AssertionConsumerService' => [
        [
            'index' => 0,
            'Binding' => 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST',
            'Location' => 'https://your-host-name/simplesaml/module.php/saml/sp/saml2-acs.php/your-host-name',
        ],
        [
            'index' => 1,
            'Binding' => 'urn:oasis:names:tc:SAML:1.0:profiles:browser-post',
            'Location' => 'https://your-host-name/simplesaml/module.php/saml/sp/saml1-acs.php/your-host-name',
        ],
        [
            'index' => 2,
            'Binding' => 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Artifact',
            'Location' => 'https://your-host-name/simplesaml/module.php/saml/sp/saml2-acs.php/your-host-name',
        ],
        [
            'index' => 3,
            'Binding' => 'urn:oasis:names:tc:SAML:1.0:profiles:artifact-01',
            'Location' => 'https://your-host-name/simplesaml/module.php/saml/sp/saml1-acs.php/your-host-name/artifact',
        ],
    ],
    'contacts' => [
        [
            'emailAddress' => 'your email',
            'contactType' => 'technical',
            'givenName' => 'your name',
        ],
    ],
    'certData' => 'MIIFCzCCA3OgAwIBAgIUE5hXCloUaNCNjkGg7x0lnMVbRAUwDQYJKoZIhvcNAQELBQAwgZQxCzAJBgNVBAYTAkJEMQ4wDAYDVQQIDAVEaGFrYTEOMAwGA1UEBwwFRGhha2ExGTAXBgNVBAoMEERoYWthIFVuaXZlcnNpdHkxDDAKBgNVBAsMA0NTRTETMBEGA1UEAwwKc3AxLmR1LmNvbTEnMCUGCSqGSIb3DQEJARYYbWVzYmFoLmNzZXN1c3RAZ21haWwuY29tMB4XDTIyMDUzMDE3MzYyMloXDTMyMDUyOTE3MzYyMlowgZQxCzAJBgNVBAYTAkJEMQ4wDAYDVQQIDAVEaGFrYTEOMAwGA1UEBwwFRGhha2ExGTAXBgNVBAoMEERoYWthIFVuaXZlcnNpdHkxDDAKBgNVBAsMA0NTRTETMBEGA1UEAwwKc3AxLmR1LmNvbTEnMCUGCSqGSIb3DQEJARYYbWVzYmFoLmNzZXN1c3RAZ21haWwuY29tMIIBojANBgkqhkiG9w0BAQEFAAOCAY8AMIIBigKCAYEAvDe2hetthZPm6VuII1W7wtymmBnfhNR9Kg0VHMG8i+cYx6e49im4EcHfbmwQh3jf21dFnt06Qg2qfZaxvoGVgSzOXC6wOLMgLdr1QBzSrri3JZmW/u8KgOqT6lYFFHzGapKPlDqOwijvQBYzGKIzvpzhVvGC54PM6ri2HQCuc32WiiDXBBr9eXLmTMBagG8VfFX5Aw2+8pVks/VSHZ+F5QrSV9/ov7iINIXRc4DqDk08eOjKitgCwkODmz+16HtmtQgnosTdFOTupBXrf2AH5j8QmNAUleDmfncAmwhcugWPP64xm/IkIxeknmYRDdbpBaBoR4TYRO/UVCfNDBrUmoypdr/Wh5WLk/UIIS4calXnFjv/1l91yTsNx4KuYPuFcRWcBbPRJeUgcGqerFwItOWXXAY7on07rdbq3Zd76qhw2WCeo0xDCEC1rBCWtiNAFxGEJUF5FENf14tOGfznMedKB3gMY+GxJRcF5lt3TaeepnwsMb9rLg3wHAZAbqMXAgMBAAGjUzBRMB0GA1UdDgQWBBTeCF6r/I/Sym64VUmx4YjgKXiiYTAfBgNVHSMEGDAWgBTeCF6r/I/Sym64VUmx4YjgKXiiYTAPBgNVHRMBAf8EBTADAQH/MA0GCSqGSIb3DQEBCwUAA4IBgQBwqamjmVi+nxt34YgvBTYaaGJRLU0ZLJxDuaTSAeDOiGT9CSFXOM77+TOyZX2YwYc+0prZxuZlUV4E7HgLlDPnsDkTqh4tKAqVWHJn9NbHdKmafFrCGKCPN/w4YxWvDUcMHtfpdubnd2maTEyUcboG5lHPNXkYCWdgrKxGCflnpV55+7EjWNUf3xPfefAmFK64N7SnojK7eedKbYNnTEb1cQVWXXuFpqqjVm0Y1BVHj8WNkiJ1m/o/ILhPAXmzzWw9npUC9GH9rrfoRRNBPyR6HkMvODCl9nVypFvsFQORSwDopqpHdJTODunWDFqtR8qvdV30Bgni6wtwze6oHSJb7ciuee7UumuMOrp1+fgwzT9SO/0RJaBjuqfOiJ9nfQ0sUyt4pwXjfYIPYZnanMu8dCLCxcLD9gKnBcn7kb0+VDZISk8pB0B2/lN7Pp1PKnaZqK7PzbWeH6EXCqXsA5+DTaHVUT+rXEqiWuyGoN9WpICFU27DQ2jlZfpRFZI5WV0=',
];

```
For more information, please visit [SimpleSAMLPhp Identity Provider Quick Start](https://simplesamlphp.org/docs/latest/simplesamlphp-idp.html).
