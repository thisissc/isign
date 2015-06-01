# Where to get certificates and keys

## Account

We already have an account for the Real Device Cloud, not owned by any single developer, tied to the
`mobdev@saucelabs.com` mail alias. Look in Passpack for the details about how to sign into this account.

If you want to make your own account, say for development on your
own machine: first, find an administrator for our developer organization at
Apple. At this moment, @neilk, @filmaj, and @admc are all admins. Get them to 
invite you to the organization under your @saucelabs.com account, in the iOS Developer Program.

You'll get email from Apple, which will prompt you to set up your account. Set up passwords and 
so on as usual.

**Troubleshooting:** you may have to click on 
the mailed invite link once to set up your account, and then return to your mail to click 
on that invite link again to actually activate your account. Also, in general, things on the Apple site
work better with Safari, so if something doesn't work, try that browser.

## Setting up credentials for iresign

### Apple certificates

The `applecerts.pem` file can be constructed by these steps. In theory you can export them from 
Keychain Access, but when I tried it the certs were outdated. I think there's a way to update 
them, but this procedure worked for me:

```bash
    $ curl 'https://www.apple.com/appleca/AppleIncRootCertificate.cer' > AppleIncRootCertificate.cer
    $ curl 'http://developer.apple.com/certificationauthority/AppleWWDRCA.cer' > AppleWWDRCA.cer
    $ openssl x509 -inform der -in AppleIncRootCertificate.cer -outform pem -out AppleIncRootCertificate.pem
    $ openssl x509 -inform der -in AppleWWDRCA.cer -outform pem -out AppleWWDRCA.pem
    $ cat AppleWWDRCA.pem AppleIncRootCertificate.pem > applecerts.pem
```

Here's a conceptual explanation of what we're doing:

Download the following certs from [Apple's Certificate Authority Page](https://www.apple.com/certificateauthority/)

* Apple Inc. Root Certificate
* Worldwide Developer Relations Certificate

Then convert these to [PEM](http://en.wikipedia.org/wiki/Privacy-enhanced_Electronic_Mail) format.

Then, concatenate them together. **This file can now serve as the 'apple certs' for code signing.**

### Your keys and certificates

If you intend to use this account to sign apps for later testing with Appium, it's important that 
everything in this account be for "Developer", not "Distribution". At this moment, Appium uses 
Instruments to install apps and otherwise control the phone, and Instruments won't touch an app that
is signed for Distribution. (Incidentally, welcome to [Richard Stallman's nightmare](http://www.gnu.org/philosophy/right-to-read.en.html).)

The following procedure works as of May 2015, for adding a new account from scratch. You need to use a Mac to follow
these instructions, as they rely on Apple tools to get you started.

Log into your Apple account.

Go to *Certificates, Identifiers, & Profiles*.

Go to *Certificates > Development* in the left-hand column.

Press the plus icon to add a new certificate.

It will ask what type of certificate you need. If this account is meant to sign apps for Appium testing, 
select *iOS App Development*.

It will then instruct you to use Keychain Access to generate a Certificate Signing Request. In effect you are 
going to create a private/public key pair, and then make a little file that says "Hey Apple, please sign this public key 
and make a certificate for it, thus associating both keys with my Apple account!"

Follow the instructions and save that CSR to disk. Press Continue.

Then, the Apple website will ask you to upload that CSR. Do so, and it will create a certificate for your account in your
organization. This certificate might need to be approved by an admin before you can download it.

Once it's approved, download it!

It will probably be named something generic like `ios_development.cer`, so rename it to something more meaningful and
put it somewhere safe.

Import that .cer into Keychain. Keychain will detect that it has an associated private key, and 
in views where you see keys, the certificate will be "inside" the key, and vice versa.

Finally, let's export these.

In Keychain Access, open the *Keys*. Find the key you created and export it as a `.p12` file. If Keychain asks you for
a password to protect this file, just leave it blank. **This is your key file**.

In Keychain Access, open the *Certificates*. Find the certificate you created and export it as a `.cer' file. Then,
convert it to a `.pem` file with `openssl`:

```bash
    $ openssl x509 -inform der -in <your.cer> -outform pem -out <your.pem>
```

This PEM file can now serve as **your certificate file for code signing**.

### Provisioning profile

One more hoop to jump through. Apple will allow you to sign an app, but it will only work on a number of devices which
have been registered with Apple. We will be registering each of those devices as we deploy them. For the moment, our
development iOS devices are also in the same pool.

We just need to tell Apple that your user is allowed to deploy on those devices. The file that proves this is 
called a provisioning profile.

In the Developer portal, go to *Provisioning Profiles*, and create a new development profile. (You can modify an old one, but it's
painful -- the existing versions of the profile expire or something.)

In *Select App ID*, use 'iOS RDC' -- I think anything with our Apple organizational unit plus dot-star works (`JWKXD469L2.\*`)

Next, in 'Select certificates', select the certificates you want, which probably includes the  you care about.


