# TLS-Somewhere
A deeply cynical but probably safe attempt to provide a way to use TLS for hidden services without manually accepting self-signed certificates. Almost certainly won't see the light of day.

The basic premise is to configure a browser to accept self-signed certificates by default, then configure a browser extension to reject them and present it's own messages instead.
If the key for the TLS certificate could not have been created without the key for the hidden service, then no message should be presented at all.

Credit:

Example code provided by this Stack Exchange thread: https://stackoverflow.com/questions/2402121/within-a-web-browser-is-it-possible-for-javascript-to-obtain-information-about