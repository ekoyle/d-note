d-note
======

Disclaimer
----------

Currently, there is no code. I can't even guarantee that there will be any
code. This is just an idea. If I get it working, and code starts showing,
great. If not, I'm sorry, but it appears that I have other things on my
plate that are taking high priority.

Introduction
------------

Self destructing notes you can run on your own web server. I got the idea
from a number of websites doing pretty much the same thing:

* https://oneshar.es/
* https://privnote.com/
* https://quickforget.com/
* https://onetimesecret.com/
* https://burnnote.com/

And many more. Unfortunately, none of the above sites seem to be interested
in benefiting the community as a whole by providing their source code, even
though there seems to be a demand for it. So, that is the goal of this
project.

The name of the project is inspired by the "H-Bomb", or hydrogen bomb. I
wanted a clever name for self destructing notes that was not in use, and
something that had a familiar ring to it. "d-note" seemed to fit for
"destructive note", and as already mentioned, inspired by the hydrogen
bomb.

Ideas
-----

* Store each entry in the DB in 100% encrypted form.
    * Use AES with a randm key stored in the config.
* Require SSL for web clients.
* Default timer of 3 minutes after opening to destruction.
* Default timer of 24 hours after creating to destruction.
* Scrub table entry before deletion.
* Use MyISAM instead of InnoDB to prevent journaling.
    * SQLite? MySQL? PostgreSQL? Others?
* Require encrypted filesystem underneath DB?
* Force browsers to not cache the site.
* JavaScript option to prevent copying.
* "Spotlight" option to make screenshots ineffective.
* Image generation to create copy/pastes.
    * If copied, set JavaScript timer to clear clipboard in 1 minute.
    * Randomize clipboard to discourange key logging.

Python Documentation
====================

URL Generation
--------------

URLs for self destructing notes should not be predictable in any manner.
Thus, a 22-character base64-encoded string is generated for each
submission. This will give us enough random URLs to avoid a collision with
1 in 2^122. The code should be self-documenting, however, this might
explain things a bit more clearly.

Each URI starts with using data found in /dev/urandom by using the uuid
module with `u=uuid.uuid4()`. This gives the source of the URI 122-bits of
entropy, by using the randomness found in the UUID v4 RFC, which should be
sufficient for generating URLs.

We then encode the string using `base64.urlsafe_b64encode(u.bytes)[:22]`
from the base64 module. This gives us 22 characters for our URL. The valid
characters for our URLs are thus:

    ABCDEFGHIJKLMNOPQRSTUPVXYZabcdefghijklmnopqrstupvxyz0123456789-_

So, a valid URL for your self destructing notes could be:

    https://exmaple.com/cWQI4m3fRcW8zM_Mdeg3uQ

There are some notes to consider with this URL scheme:

* UUID v4 has the format XXXXXXXX-XXXX-4XXX-MXXX-XXXXXXXXXXXX, where '4' is
statically defined, and 'M' can be 8 9 a or b.
* With the raw bytes converted to base64, some characters are predictable.
Its format is XXXXXXXXMXNXXXXXXXXXXXO, where 'M' is either QRST,
'N' is either -26aCeGiKmOqSuWy and 'O' is either AgQw.

Regardless, the server would need to be processing 1 billion URLs every
second for 100 years before we reached the probability of 1/2 for
generating a duplicate URL.

d-note does not keep track of which URLs have been generated. Thus, it is
possible, although highly improbable, that the same URL could be generated
for two different form submissions. Of course, the application will check
against any valid notes that have not yet self destructed, but will not
check for ones that have.

Thus, it is possible that a URL that has already self destructed could be
regenerated at a different time, which has not self destructed. If the
first URL is publicly accessible, that means that the second URL could be
opened by the wrong recipient accidentally. As such, these URLs should be
kept as private as possible to prevent this from happening.

JavaScript Documentation
========================

Hashcash
--------

The point of a Hashcash implementation is to prevent form spam. I'm not
sure what the benefit of spammers would be to use self-destructing notes,
but nonetheless, I'm not really interested in entertaining it.
Implementaning Hashcash as a proof-of-work system is simple enough to
deter most spammers. The breakdown is as follows:

* Server gets client IP address.
* Server generates nonce.
* IP address and nonce become the resource string.
* The resource string is embedded invisibly into the form.
* Client then mints a Hashcash token based on the resource string found.
* The client submits the form with the minted token.
* Server verifies if the token is valid.
    * If valid, the form submits.
    * If not valid, the user is notified submission failed.

A resource string with IP address "65.100.223.163", and nonce "VILymxxv"
generated by the server could look like this:

    65100223163VILymxxv

A minted Hashcash token generated by the client would then need to look
something like this:

    1:20:120715:65100223163vilymxxv::lVb6gfTxYb1Ir5SW:COBt

This is valid, because the SHA1 hash of the above token is:

    00000bed07757ed13f1f2cebc67b616c75812b41

which starts with 20-bits of leading zeros. The work is forced on the
client, which inserts the token into the form. Even on modern hardware,
this should be a strenuous task on the client CPU, and could take up to a
second or two to create a valid token string. However, the server can
verify the token quickly (in nanoseconds).

The minting of the token should be done in the background while the user is
typing the note in the form. Thus, when the submit button is pressed, no
additional waiting is needed.

More info can be found at http://hashcash.org.
