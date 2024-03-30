# TLS-Somewhere
A deeply cynical but probably safe attempt to provide a way to use TLS for hidden services without manually accepting self-signed certificates.
Almost certainly won't see the light of day.
Super hare-brained.
Would work for I2P or Tor.

The basic premise is to configure a browser to accept self-signed certificates by default, then configure a browser extension to reject them and present it's own messages instead.
If the key for the TLS certificate could not have been created without the key for the hidden service, then no message should be presented at all.

Very, very obviously, no one should do this yet.
None of this works yet.
It deliberately abuses an important security feature to get something the people in charge of that security feature don't actually want.
It is **genuinely** a bizarre idea.

But I'm sure I'm right and it will work.

## How?

### First, create a **really** dangerous web browser:

0. Create a `i2p-snakeoil` certificate authority and add it to [cert9.db](https://manpages.debian.org/testing/libnss3-tools/certutil.1.en.html) for a [`single browser profile`](https://github.com/eyedeekay/i2p.plugins.firefox).
1. Distribute it's keys **everywhere** you need them, public and private. **Obviously** this makes it's real utility as a CA worse than zero, because at this point anyone can sign a certificate for any site anywhere and your single-browser-profile will accept it because you have an intentionally compromised CA. That scary thing you thought when you read the last sentence **is** the thing I am talking about. Be afraid, be very afraid.*
2. Let **anyone**, **anywhere** sign **any** site certificate with them.

### Then, fix it with a browser extension:

0. Add TLS-Somewhere to `single browser profile`.
1. TLS-Somewhere implements TLS verification, with a twist:

#### By doing this for clearnet sites:

0. Examine the certificate chain for every request.
1. If the `i2p-snakeoil` certificate appears anywhere, reject it and throw the user to an extension page explaining how the user was attacked.

#### And this for hidden services:

0. Examine the certificate chain for every request.
1. If the `i2p-snakeoil` certificate appears anywhere, proceed to:
2. Determine if the certificate was also signed by the private keys used for the hidden service.
3. If the hidden service keys were used to sign the TLS certificate, proceed to the site and browse normally. If not, proceed to the next step.
4. If the hidden service keys were *not* used to sign the TLS certificate, present a warning with an *option* to proceed.
5. Store the user's choice until the browsing session is closed.

## What?

I don't blame you. So what we've done here, hypothetically, is:

1. Tricked Firefox into accepting any certificate signed by our fake CA.
2. Added a browser extension which *rejects* the fake CA for clearnet sites.
3. Created new rules for hidden services, moving the *real* authority behind the certificate to the hidden service itself, i.e. it's own private keys.
4. Made exceptions for the `i2p-snakeoil` certificate ephemeral, lasting only the length of a browsing session.
5. Leave the behavior of other self-signed certificates unchanged.

##### Credit:

```md
[Example code provided by this Stack Exchange thread:](https://stackoverflow.com/questions/2402121/within-a-web-browser-is-it-possible-for-javascript-to-obtain-information-about)
```

```md
*But this is essential, because they took away the `about:config` option to disable TLS verification in 2018, and [`webRequest.getSecurityInfo()` does not return an array of certificates if the certificate was self-signed](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/webRequest/SecurityInfo). So to get the primitives that we need to accomplish this, we need to trick Firefox.**
**Mozilla, I get that you're not building for power-users but you sure are bumming them out.
```