Community information collected about the FDev Companion API (Frontier API)
===

---

See [Frontier API](<Frontier API/README.md>) for community documentation of Frontier Development's Companion API (Frontier API), including a general walk-through of how to handle the oAuth2 flow. 

For example implementations accessing this API please see:
- [Python Example](https://github.com/Athanasius/fd-api) (not maintained as of April 4th, 2024) 
- [C# Example](https://github.com/EDDiscovery/CAPI)

---

Tue 21 Mar 10:40:34 GMT 2023
---
  It has come to light that whilst Frontier's Auth service will happily give you an Access and Refresh Token if you only ask for `scope=capi` (rather than `scope=auth capi`), the Access Token now does not contain some fields.
  This would be fine ... except the CAPI service objects if the `auth` scope is omitted:

    HTTP Status 401 - Access Token expired: {"status":401,"message":"Authentication token incomplete","missing":["email","firstname","lastname"],"tag":"<elided>"}

Thus, for the time being, the `auth` scope is required even for developers that would otherwise not need to handle `auth` data.

---

IDs returned from the FDev Companion API
====

Notes
----

- These collections are incomplete - most notably some PowerPlay-specific items are missing from outfitting.csv.
- Player-specific items are deliberately omitted from outfitting.csv - i.e. those items returned from the API with category/slotname "bobblehead"/"BobbleNN", "decal"/"DecalN", "paintjob"/"PaintJob", "enginekit"/"EngineColour", "shipkit"/"ShipKit...", "weaponkit"/"WeaponColour"
- The values in the "name" fields in these files are as shown in-game in English. These are not the same in all cases as the text values returned by the Companion API or used over [EDDN](https://github.com/jamesremuscat/EDDN/blob/master/schemas/shipyard-v1.0.json#L55).
