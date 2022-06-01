# SimpleSAMlphp as a Service Provider(SP)
This guide will describe how to configure SimpleSAMLphp as a service provider (SP). You should previously have installed SimpleSAMLphp as described in the [SimpleSAMLphp installation instructions](https://github.com/buckbeak99/SimpleSAMLphp/blob/553616106079a137894270d614abc2ff110dd595/README.md)  
- [Prerequisite](https://github.com/buckbeak99/SimpleSAMLphp/blob/553616106079a137894270d614abc2ff110dd595/README.md)
- [Securing Apache Web Server](https://github.com/buckbeak99/Securing-Apache-Server)
- [Download and Install SimpleSAMLphp](https://github.com/buckbeak99/SimpleSAMLphp/blob/553616106079a137894270d614abc2ff110dd595/README.md))
- [Configuring Apache to serve SimpleSAMLphp](https://github.com/buckbeak99/SimpleSAMLphp/blob/553616106079a137894270d614abc2ff110dd595/README.md)
- [Configuring SimpleSAMLphp](https://github.com/buckbeak99/SimpleSAMLphp/blob/553616106079a137894270d614abc2ff110dd595/README.md)
- [Enanbling and Disabling Modules](https://github.com/buckbeak99/SimpleSAMLphp/blob/553616106079a137894270d614abc2ff110dd595/README.md)
## Configuring the SP
 The SP is configured by an entry in config/authsources.php .

This is a minimal authsources.php for a SP: 
```bash
<?php
  $config = [

      /* This is the name of this authentication source, and will be used to access it later. */
      'default-sp' => [
          'saml:SP',
      ],
  ];
```
 For more information about additional options available for the SP, see the [saml:SP reference](https://simplesamlphp.org/docs/latest/saml/sp.html) .

If you want multiple Service Providers in the same site and installation, you can add more entries in the authsources.php configuration. If so remember to set the EntityID explicitly. Here is an example: 
```bash
'sp1' => [
    'saml:SP',
  'entityID' => 'https://sp1.example.org/',
],
'sp2' => [
    'saml:SP',
  'entityID' => 'https://sp2.example.org/',
],
```
We don't need to edit this section. 
### Enablig a Certificate for your Service Provider
Some Identity Providers / Federations may require that your Service Providers holds a certificate. If you enable a certificate for your Service Provider, it may be able to sign requests and response sent to the Identity Provider, as well as receiving encrypted responses.  
Create a self-signed certificate in the `cert/` directory.
```bash
cd cert
```
paste this command
```bash
openssl req -newkey rsa:3072 -new -x509 -days 3652 -nodes -out saml.crt -keyout saml.pem
```
Then edit your authsources.php entry, and add references to your certificate:
```bash
cd config
```
Open `authsources.php` file
```bash
sudo nano authsources.php
```
Add reference to your certificate:
```bash
'default-sp' => [
    'saml:SP',
    'privatekey' => 'saml.pem',
    'certificate' => 'saml.crt',
],
```

## Adding IdPs to the SP
The service provider you are configuring needs to know about the identity providers you are going to connect to it. This is configured by metadata stored in `metadata/saml20-idp-remote.php`. 
```bash
cd metadata
```
then open `saml20-idp-remote.php`
```bash
sudo nano saml20-idp-remote.php
```
Go to your **Identity Provider's** page and click fedaration tab. Here you can see the IdP metadata. Copy it and paste in `saml20-idp-remote.php`.
Here is an example. **Don't copy this**.
```bash
$metadata['https://your-idp-domain/simplesaml/saml2/idp/metadata.php'] = [
    'metadata-set' => 'saml20-idp-remote',
    'entityid' => 'https://your-idp-domain/simplesaml/saml2/idp/metadata.php',
    'SingleSignOnService' => [
        [
            'Binding' => 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect',
            'Location' => 'https://your-idp-domain/simplesaml/saml2/idp/SSOService.php',
        ],
    ],
    'SingleLogoutService' => [
        [
            'Binding' => 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect',
            'Location' => 'https://your-idp-domain/simplesaml/saml2/idp/SingleLogoutService.php',
        ],
    ],
    'certData' => 'MIIFCzCCA3OgAwIBAgIUF4tQq0jH26vJ6wRcFngCDYHOtiAwDQYJKoZIhvcNAQELBQAwgZQxCzAJBgNVBAYTAkJEMQ4wDAYDVQQIDAVEaGFrYTEOMAwGA1UEBwwFRGhha2ExGTAXBgNVBAoMEERoYWthIFVuaXZlcnNpdHkxDDAKBgNVBAsMA0NTRTETMBEGA1UEAwwKaWRwLmR1LmNvbTEnMCUGCSqGSIb3DQEJARYYbWVzYmFoLmNzZXN1c3RAZ21haWwuY29tMB4XDTIyMDUzMDE4MDcwNFoXDTMyMDUyOTE4MDcwNFowgZQxCzAJBgNVBAYTAkJEMQ4wDAYDVQQIDAVEaGFrYTEOMAwGA1UEBwwFRGhha2ExGTAXBgNVBAoMEERoYWthIFVuaXZlcnNpdHkxDDAKBgNVBAsMA0NTRTETMBEGA1UEAwwKaWRwLmR1LmNvbTEnMCUGCSqGSIb3DQEJARYYbWVzYmFoLmNzZXN1c3RAZ21haWwuY29tMIIBojANBgkqhkiG9w0BAQEFAAOCAY8AMIIBigKCAYEA0mTfLsprrps9G6zIfLlJZCxUGrhdh8MZu3eJrb2/y8A4r30v6fvxHyQiZpVcnOUAaNFhBOUwJgxFZiTxfGPlD4W4hOWm3MzPFdftJhbNlvG28HAscGkyliRoD2GB3YTPCX56VLTa4Dsp+YdY/u0EgP6V+Pf8+IHHxLxUyzdUrGkKAPB9KedmAC5oigJmByCeYlxTAkOt0jDy7FN6Hd0x2PvqXSp9Ael8gM+buFcWose/ABzt8RMhAV4OTLmXABUQlkA35Ieeg9kPV1eG5yCCrx6E4QL36suM/yn5oNEaTQLe4Ys/tmLFzzRYlee0gECDyj8ETGECloLtDXfR+gILPT3nT6t9NxaxfG5I/wdAv62UFaJP8cVF3etqp2roTHYNGezLmxSRTEWrGQu8Xqn6Jcw9VOWfLGmRKTQebsSM1OuaNvQ0P5WiTz0iQtW1DJC+sYO5w5lId+UCAyYNhfEpKfzqhKGIFkgTWyEYvNu+DkF8QELQt5AzsIJ2jYlo4By7AgMBAAGjUzBRMB0GA1UdDgQWBBRA1FonD+2SEmvD28+dEhDL0K4HrzAfBgNVHSMEGDAWgBRA1FonD+2SEmvD28+dEhDL0K4HrzAPBgNVHRMBAf8EBTADAQH/MA0GCSqGSIb3DQEBCwUAA4IBgQCK8vRiUvueqgLg21W16Wy/7oKpOCP+JY6t7ygka5mvxETtIU8ArXY/x5DZeY2gPxrIWhFY9v0zJf9ENtjEKRpzOBqF7g5qaqXsRuNkFadZyjcIH6dUuIVY7MKHjOp3TCQ2In174AOIgykpNlYvxpOGqMBxnGFWtG6/IblcfDcxmLA2Kep+r/g2HcN8iTvSKo9s3TM1OyL2uLy29g4C4WK8fFV4rsIrHREG3EhjsOip3Yie2PcKc+Pzq2g82qaJb/Dve0DMeC9GSVc3eIoGIJFYpre7GJyAuKa7bEZbSq8+n9gTbZFlnzMhNt1AgwY3/MfAryp4RWvSQTSqkJmO5erPUNRJBJfh054I8ygjhRCDoHZ4sBU7lBYw7ROYiOoIwdjF+ChWB2PfSzuAXDfRv9gwex2r65lDT2XqM0ho9FG5J5pdXaTg2VaUwUXrUkEYmVdvQOuq4ITnw/A6q952uRmYGTqfbzWyd2zWO7lIgZPXkop4TxKrgaVRCYLf8g9Ti+8=',
    'NameIDFormat' => 'urn:oasis:names:tc:SAML:2.0:nameid-format:transient',
    'contacts' => [
        [
            'emailAddress' => 'your email',
            'contactType' => 'technical',
            'givenName' => 'your name',
        ],
    ],
];
```
For more information about available options in the idp-remote metadata files, see the [IdP remote reference](https://simplesamlphp.org/docs/latest/simplesamlphp-reference-idp-remote.html).

## Setting the default IdP
An option in the authentication source allows you to configure which IdP should be used. This is the idp option.
You will get this IdP ID on the fedaration page of your Identity Provider. Copy the entity ID and Paste it here.
```bash
cd config
```
```bash
sudo nano authsources.php
```
```bash
<?php
  $config = [

      'default-sp' => [
          'saml:SP',

          /*
           * The entity ID of the IdP this should SP should contact.
           * Can be NULL/unset, in which case the user will be shown a list of available IdPs.
           */
          'idp' => 'https://idp.example.com/simplesaml/saml2/idp/metadata.php',
      ],
  ];
```
## Exchange Metadata with the IdP
Go to your **Service Provider's** fedaration page. You will get your sp metadata. Copy the metadata and follow the steps:
```bash
sudo nano /var/www/idp.example.com/simplesamlphp/metadata/saml20-sp-remote.php
```
paste the metadata. here is an example. **Don't copy this**.
```bash
$metadata['https://your_sp_domain/simplesaml/module.php/saml/sp/metadata.php/your_sp_domain'] = [
    'SingleLogoutService' => [
        [
            'Binding' => 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect',
            'Location' => 'https://your_sp_domain/simplesaml/module.php/saml/sp/saml2-logout.php/your_sp_domain',
        ],
    ],
    'AssertionConsumerService' => [
        [
            'index' => 0,
            'Binding' => 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST',
            'Location' => 'https://your_sp_domain/simplesaml/module.php/saml/sp/saml2-acs.php/your_sp_domain',
        ],
        [
            'index' => 1,
            'Binding' => 'urn:oasis:names:tc:SAML:1.0:profiles:browser-post',
            'Location' => 'https://your_sp_domain/simplesaml/module.php/saml/sp/saml1-acs.php/your_sp_domain',
        ],
        [
            'index' => 2,
            'Binding' => 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Artifact',
            'Location' => 'https://your_sp_domain/simplesaml/module.php/saml/sp/saml2-acs.php/your_sp_domain',
        ],
        [
            'index' => 3,
            'Binding' => 'urn:oasis:names:tc:SAML:1.0:profiles:artifact-01',
            'Location' => 'https://your_sp_domain/simplesaml/module.php/saml/sp/saml1-acs.php/your_sp_domain/artifact',
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

## Test The SP
 After the metadata is configured on the IdP, you should be able to test the configuration. The admin module of SimpleSAMLphp has a tab to test authentication sources. There you should a list of authentication sources, including the one you have created for the SP.

After you click the link for that authentication source, you will be redirected to the IdP. After entering your credentials, you should be redirected back to the test page. The test page should contain a list of your attributes:
![test-the-sp](https://user-images.githubusercontent.com/43216053/171489289-f64180c7-dfad-40ee-abe9-b875333b545b.png)

For a better looking, more advanced Discovery Service with tabs and live search, you may want to use the discopower module.
For more information, please visit [SimpleSAMLphp as Service Provider Quick Start](https://simplesamlphp.org/docs/latest/simplesamlphp-sp.html).
