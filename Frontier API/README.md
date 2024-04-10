# Documentation relating to Frontier/Elite Dangerous APIs
 - [FrontierDevelopments-oAuth2-notes](FrontierDevelopments-oAuth2-notes.md) - Notes on how to utilise FDev's oAuth2 mechanism.
 - [FrontierDevelopments-CAPI-endpoints](FrontierDevelopments-CAPI-endpoints.md) = Notes on the available CAPI end points, how to use them, and what they return.
 - [example-auth-decode.json](example-auth-decode.json) - Example (elided) output from the oAuth2 /decode end point.

## Notes
### User-Agent

Frontier Developments have requested that all access to both the CAPI
and the Authorization service use a custom User-Agent header that
matches this regular expression:

		EDCD-[A-Za-z]+-[.0-9]+
e.g.

		EDCD-YourApp-0.0.1
so that they can more easily track down anyone causing issues. Obviously
the middle part should probably be something that aligns with the
application name you gave when applying for Authorization access.

## Other

 - [Other Frontier APIs](./FrontierDevelopments-other-APIs.md) -
   Community Goals, GalNet, etc.
