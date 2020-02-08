# Passbook

[![Build Status](https://travis-ci.org/devartis/passbook.svg?branch=master)](https://travis-ci.org/devartis/passbook)

Python library to create Apple Wallet (.pkpass) files (Apple Wallet 
has previously been known as Passbook in iOS 6 to iOS 8).

See the [Wallet Topic Page](https://developer.apple.com/wallet/) and the
[Wallet Developer Guide](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/PassKit_PG/index.html#//apple_ref/doc/uid/TP40012195) for more information about Apple Wallet.

> If you need the server side implementation (API / WebServices) in django you should check http://github.com/devartis/django-passbook.


## Getting Started

Ensure you have M2Crypto installed

    sudo easy_install M2Crypto

Now you need an Apple developer account and WWDR certificates. I suggest you to create a certificate-dedicated folder inside your working directory (e.g. `./certs`) to contain everything concerning the certificates.

This is a standard procedure: you would have to do it also without using this library. We'll use OpenSSL to complete our work (or to do it entirely, if only on terminal), so be sure to have it installed.
You'll need the following three elements:

* Apple WWDR (_Worldwide Developer Relationship_) certificate
* Signer certificate
* Signer key

While WWDR can be obtained from [Apple PKI Portal](https://www.apple.com/certificateauthority/), to get the `signer key` and the `certificate`, you'll have to get first a `Certificate Signing Request` (`.certSigningRequest` file) and upload it to Apple Developers Portal, at [Pass Types Identifiers](https://developer.apple.com/account/ios/identifier/passTypeId) (open it, it's worth it 😜).

<br>
<hr>

> **If you don't have access to macOS** (or you are a terminal enthusiast), **follow [these steps](./non-macOS-steps.md) instead.**
<hr>


1. Create a new pass type identifier and provide it with a Name and a reverse-domain bundle id (starting with "pass."). You will put this identifier as value for `passTypeIdentifier` in `pass.json` file.
2. Confirm and register the new identifier.
3. Go back to the pass type identifiers, click on your new pass id and edit it.
4. Click "Create Certificate" button and follow the instructions until you won't download a certificate like `pass.cer`. (here you'll generate the `.certSigningRequest` file to be uploaded).
5. Open the downloaded certificate. Go in "Certificates" on left in macOS Keychain access and `right-click > Export "\<certname\>"`. Choose a password (and write it down) and you will get a PKCS#12 file (`.p12`).
6. Open terminal, place where you want to save the files and insert the following OpenSSL commands changing the contents between angular brackets. You'll have to choose a secret passphrase (and write it down) that you'll use also in the application.

    ```sh
	# Creating and changing dir
    $ mkdir "certs" && cd $_
	# Extracting key and cert from pkcs12
    $ openssl pkcs12 -in <cert-name>.p12 -clcerts -nokeys -out signerCert.pem -passin pass:<your-password>
    $ openssl pkcs12 -in <cert-name>.p12 -nocerts -out signerKey.pem -passin pass:<your-password> -passout pass:<secret-passphrase>
    ```
7. Execute step 5 also for the WWDR certificate (`.cer`) you downloaded from Apple PKI portal (default name: *AppleWWDRCA.cer*) but instead exporting it as PKCS#12 (`.p12` - you'll also be unable to do that), export it as PEM (`.pem`) file.

## Typical Usage

    #!/usr/bin/env python

    from passbook.models import Pass, Barcode, BarcodeFormat, StoreCard
    
    # Create a model object, in this example a StoreCard
    cardInfo = StoreCard()
    # Add a 'name' primary field with label 'Name' and value 'John Doe'
    cardInfo.addPrimaryField('name', 'John Doe', 'Name')

    organizationName = 'Your organization' 
    passTypeIdentifier = 'pass.com.your.organization' 
    teamIdentifier = 'AGK5BZEN3E'
    
    passfile = Pass(cardInfo, \
        passTypeIdentifier=passTypeIdentifier, \
        organizationName=organizationName, \
        teamIdentifier=teamIdentifier)
    passfile.serialNumber = '1234567' 
    passfile.barcode = Barcode(message='Barcode message', format=BarcodeFormat.QR)    

    # Including the icon and logo is necessary for the passbook to be valid.
    with open('icon.png', 'rb') as icon_file, \
         open('logo.png', 'rb') as logo_file:
     
        passfile.addFile('icon.png', icon_file)
        passfile.addFile('logo.png', logo_file)
    
    # Create and output the Passbook file (.pkpass)
    password = '123456'
    passfile.create('certificate.pem', 'key.pem', 'wwdr.pem', password , 'test.pkpass')

## Note: Getting WWDR Certificate

Certificate is available @ http://developer.apple.com/certificationauthority/AppleWWDRCA.cer

It can be exported from KeyChain into a .pem (e.g. wwdr.pem).

## Testing

You can run the tests with `py.test` or optionally with coverage support 
(install `pytest-cov` first): 

    py.test --cov
    
You can also generate a HTML report of the coverage:

    py.test --cov-report html

You can run the tests against multiple versions of Python by running `tox` 
which you need to install first.

## Credits

Developed by [devartis](http://www.devartis.com).
