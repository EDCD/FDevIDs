# Introduction
## oAuth2
See [FrontierDevelopments-oAuth2-notes](FrontierDevelopments-oAuth2-notes.md)
for how to use Frontier Developments' oAuth2 system for Authorization on
the CAPI.

In all instances you'll need to use the Access Token obtained via
oAuth2 by setting an HTTP header for the request:

        Authorization: Bearer <access_token>

Note that in theory the token type could be different from 'Bearer'.
It's safest to store the token_type that comes back from the request for
an Access Token and repeat that back in this header.

---
You might occasionally see a header:

		Set-Cookie: access_token=1566841911%7C%7C<40 character hex string>; domain=companion.orerve.net; path=/; expires=Mon, 26-Aug-2019 17:51:51 UTC; secure
in the responses from the CAPI end points.  This seems to serve no
purpose, and is perhaps left over from the work to implement oAuth2 on
the service.

1. The start of the access_token value appears to be a Unix
Epoch timestamp, 86400 seconds in the past.  That matches the
expiry time on the cookie.
2. That "40 character hex string" doesn't seem to be a valid
JSON Web Token, for instance, or otherwise related to either
the Refresh or Access Tokens.

# Limit the frequency of queries

The CAPI is not designed as a real-time service to be spammed with
queries in an attempt to always be up to date.  For one thing the CAPI's
data itself can lag behind what the relevant game client is seeing.

In general try not to issue more than one query a minute.  Although
EDMarketConnector gets away with issuing all of `/profile`, `/market`,
and `/shipyard` in rapid succession upon docking, Frontier have in the
past stated that rate limiting might kick in if you perform more than 2
queries per second.

For the `/journal` endpoint you should **not** expect it to update in
realtime from live gameplay.  Ideally you should only query it once a
gameplay session is complete.

# Available CAPI hosts
You need to use the correct host for the data you are expecting to
retrieve.  As of Odyssey Update 14 this mostly pertains to the
Live/Legacy galaxy split.

The rest of this documentation will assume the Live galaxy and use its
host.

## Live CAPI host
The main CAPI host is `companion.orerve.net`. **Since Odyssey
Update 14 this only returns data for the Live galaxy.**

## Legacy CAPI host
To retrieve data pertaining to the Legacy galaxy you must use the
alternate host `legacy-companion.orerve.net`.

## CAPI for any beta of the game
In the past Frontier has sometimes provided a beta-specific CAPI host.
When it was available (this was before Odyssey Update 14 and the galaxy
split) the hostname was `pts-companion.orerve.net`.

---
# End Points

You can check the currently available endpoints by sending a bare query
to the CAPI host, i.e. `/` path component as in <https://companion.orerve.net/>.

This will produce output like:
```json
{
  "links": [
    {
      "href": "/profile",
      "rel": "CommanderProfile",
      "type": "GET",
      "title": "Commander profile, add ?language=[en|fr|de|ru|es|pt] to translate locName and LocDescriptions"
    },
    {
      "href": "/market",
      "rel": "CommodityMarket",
      "type": "GET",
      "title": "Last docked commodity market, add ?language=[en|fr|de|ru|es|pt] to translate locName and LocDescriptions"
    },
    {
      "href": "/shipyard",
      "rel": "Shipyard",
      "type": "GET",
      "title": "Last docked shipyard and outfitting"
    },
    {
      "href": "/communitygoals",
      "rel": "CommunityGoals",
      "type": "GET",
      "title": "Details on all currently active Community Goals and any contributions from this Commander, add ?language=[en|fr|de|ru|es|pt] to translate descriptions"
    },
    {
      "href": "/journal",
      "rel": "Journal",
      "type": "GET",
      "title": "Log lines from Commander Journal file, add (/:year(/:month(/:day))) to ask if previous journals are available"
    },
    {
      "href": "/fleetcarrier",
      "rel": "FleetCarrier",
      "type": "GET",
      "title": "Information about Commander's fleet carrier, add ?language=[en|fr|de|ru|es|pt] to translate locName and LocDescriptions"
    },
    {
      "href": "/visitedstars",
      "rel": "VisitedStars",
      "type": "GET",
      "title": "Download an zip archive containing the player's VisitedStarsCache.dat file. Status 102 indicates the file is being generated in the background, try again in a couple of minutes"
    }
  ]
}
```

## Profile
		GET <https://companion.orerve.net/profile>

provides access to the Cmdr's profile.  What follows is the last known
information about the output.  This will obviously be subject to change
as Frontier changes things.

Where 'FDev ID' or 'FDev Symbol' is mentioned please see <https://github.com/EDCD/FDevIDs>.

1. `commander`: Information about the Cmdr
	1. `name`: The Cmdr's Name
	1. `id`: Numerical ID
	1. `docked`: Whether currently docked
	1. `alive`: boolean
	1. `credits`: Credit Balance
	1. `debt`: Any current debt amount
	1. `currentShipId`: Numerical ID of current ship (index within
	   the Cmdr's currently owned ships, not necessarily unique over
	   time due to sell/buy).
	1. `rank`: Ranks
		1. `combat`
		1. `trade`
		1. `explore`
		1. `cqc`
		1. `empire`
		1. `federation`
		1. `power`
		1. `service`: ???
		1. `crime`
	1. `capabilities`: .e.g. whether the account has Horizons, and if it
	   can buy the Cobra Mk IV.
1. `lastStarport`: Information about the last StarPort they were docked at
	1. `name`: Name
	1. `id`: Numerical ID
	1. `faction`: Allegiance to a Super Power, if any
	1. `minorfaction`: Minor Faction that owns this asset
	1. `services`: Available services
1. `lastSystem`: The last System they were in
	1. Name
	1. Allegiance ('faction')
	1. Numerical ID
		NB: This has been observed to sometimes be the
		id64/SystemAddress of the system, and other times be
		some other id.  The `ship` section below then contains
		`id` matching this *and* `systemaddress` with the id64.
1. `ship`: Current Ship data.  Note that this duplicates some data
   that's at the top level.
	1. `id`: Current ship's Numerical ID (see `currentShipId` above)
	1. `name`: Ship Name
	1. `value` Ship Value, with breakdown cargo/modules/hull
		1. `hull`
		1. `modules`
		1. `cargo`
		1. `total`
		1. `unloaned`
	1. `free`: Whether this ship was free ?
	1. `shipName`: Current ship's Name
	1. `shipID`: Ship ID
	1. `station`: Current Station, if applicable
		1. `id`: Numerical ID
		1. `name`: Name of station
	1. `starsystem`: Current star system
		1. `id`
		1. `name`
		1. `systemaddress`: id64
	1. `alive`: boolean
	1. `health`
		1. `hull`
		1. `shield`
		1. `shieldup`: boolean
		1. `integrity`
		1. `paintwork`
	1. `cockpitBreached`: boolean
	1. `oxygenRemaining`: Oxygen Remaining
	1. `modules`: The first key in each entry describes which slot
	   the module is for, and its size.
		1. `module`:
			1. `id`: Numerical FDev ID
			1. `name`: FDev Symbol
			1. `locName`: Localised human readable name
			1. `locDescription`: Localised module
			   description.
			1. `value`: Price bought for ('value')
			1. `free`: ??? Whether it came free with a ship?
			   It's 'false' on everything, even 'Planetary
			   Approach Suite' and the Fuel Tank.
			1. `health`: Current Health
			1. `on`: Powered Status (on/off)
			1. `priority`: Power Priority
		1. `engineer`: Details of currently applied engineering
			1. `engineerName`: Which engineer was used
			1. `engineerId`: Engineer's numerical ID
			1. `recipeName`: Non-localised blueprint name
			1. `recipeLocName`: Localised blueprint name
			1. `recpieLocDescription`: Localised blueprint
			   description
			1. `recipeLevel`: Rank of the blueprint
		1. `WorkInProgress_modifications`: Details about the
		   effects of the applied blueprint
		1. `specialModifications`: "Special Effects", empty
		   array if none
	1. `launchBays`: Information about in-stock SRVs.  The top level key
   defines slot and size.
		1. `name`: 'testbuggy' for an SRV
		1. `locName`: "SRV Scarab"
		1. `rebuilds`: How many spares???
		1. `loadout`: "starter"
		1. `loadoutName`: "Starter" (Localised?)
1. `ships`: Information about all ships the Commander owns.  NB: This
   could be a simple array (if all ship indexes are contiguous from
   `0`), or a dictionary indexed with a *string* version of the ship ID
   if there are any other gaps.
   The data is the same format as in the `ship` key.

---

## Shipyard

    	GET <https://companion.orerve.net/shipyard>

Fetches the last visited shipyard for the authenticated commander.

CommodityId, EconomyId, ShipId can be found in the [EDCD/FDevIDs repo](https://github.com/EDCD/FDevIDs)

1. `id` - Station ID
1. `name` - Station name
1. `outpostType` - What type of outpost it is, examples: `starport`, `fleetcarrier`
	- NB: Might not match the type in Journal events
1. `imported` - Commodities that are imported to this station (dictionary, commodity id and key)
   - Example `"128049162": "Cobalt"`
1. `exported` - Commodities that are exported from this station (dictionary, commodity id and key)
   - Example `"128049162": "Cobalt"`
1. `services` - Services the station provides and their status
	1. `dock` - Status of Dock (`ok`, `unavailable`, `private`)
	1. `contacts` - Status of Contacts (`ok`, `unavailable`, `private`)
	1. `exploration` - Status of Exploration (`ok`, `unavailable`, `private`)
	1. `commodities` - Status of Commodities (`ok`, `unavailable`, `private`)
	1. `refuel` - Status of Refuel (`ok`, `unavailable`, `private`)
	1. `repair` - Status of Repair (`ok`, `unavailable`, `private`)
	1. `rearm` - Status of Rearm (`ok`, `unavailable`, `private`)
	1. `outfitting` - Status of Outfitting (`ok`, `unavailable`, `private`)
	1. `shipyard` - Status of Shipyard (`ok`, `unavailable`, `private`)
	1. `crewlounge` - Status of Crewlounge (`ok`, `unavailable`, `private`)
	1. `searchrescue` - Status of Search & Rescue (`ok`, `unavailable`, `private`)
	1. `techbroker` - Status of Tech broker (`ok`, `unavailable`, `private`)
	1. `stationmenu` - Status of Station menu (`ok`, `unavailable`, `private`)
	1. `shop` - Status of shop (`ok`, `unavailable`, `private`)
	1. `engineer` - Status of engineers (`ok`, `unavailable`, `private`)
1. `economies` - What different types of economies you can expect from the station (dictionary, economyId as key)
	- Example `"23": { "name" : "HighTech", "proportion": 0.7 }`
1. `prohibited` - Which commodities are prohibited on this station (dictionary, commodityId and key)
   - Example `"128049162": "Cobalt"`
1. `modules` - The list of modules on this station (dictionary, moduleId as key)
	1. `id` - ModuleId
	1. `category` - What category this module is
	1. `name` - Name for the module
	1. `cost` - Module cost
	1. `sku` - The SKU for the module
	1. `stock` - How many are available of this item, `-1` means unlimited
1. `ships` - The list of available ships on this station
	1. `shipyard_list` - The actual list (Dictionary, ship name as key)
		1. `id` - ShipId
		1. `name` - Ship name
		1. `basevalue` - The base cost of a new ship
		1. `sku` - The SKU for the ship
		1. `stock` - How many of this ship we have in stock

---

## Fleetcarrier

    	GET <https://companion.orerve.net/fleetcarrier>

provides access to information about the Cmdr's fleet carrier.

### Response codes

| Code  | Status          | Description                                                                                          |
| :---- | :-------------- | :--------------------------------------------------------------------------------------------------- |
| `200` | OK              | This means you got the Fleet Carrier Data                                      |
| `204` | No Content | This means that the player does not own a Fleet Carrier |

### Expected Output

1. `name`: The carrier's callsign and name information
    1. `callsign`: The carrier's callsign, in the format A1A-A1A.
    1. `vanityName`: The carrier's name, hex encoded, before any filtering is applied.
    1. `filteredVanityName`: The carrier's name, hex encoded, after applying FDev's obscenity filter.

1. `currentStarSystem`: Carrier's current system.
1. `balance`: Amount of credits in carrier's bank account. This is separate from the owner CMDRs.
1. `fuel`: Amount of Tritium fuel in the carrier's Tritium reserves. (Max 1000)

1. `state`: Current state of the carrier. Known states are:
    * `normalOperation`: Operating normally.
    * `debtState`: Services offline due to carrier being out of funds.
    * `pendingDecommission`: Carrier has entered the decommission process.

1. `theme`: Livery theme for the carrier. Known liveries:
    * `SearchAndRescue`
    * `Mining`
    * `Trader`
    * `Explorer`
    * `AntiXeno`
    * `BountyHunter`

1. `dockingAccess`: Who can dock with the fleet carrier. Known states:
    * `all`
    * `squadronfriends`
    * `friends`
    * `none`

1. `notoriousAccess`: Can CMDRs with notoriety dock? Boolean.

1. `capacity`: Capacity usage
    1. `shipPacks`: Capacity used by ship packs (for shipyard)
    1. `modulePacks`: Capacity used for module packs (for outfitting)
    1. `cargoForSale`: Capacity used for cargo that is listed for sale on market
    1. `cargoNotForSale`: Capacity used for cargo that is NOT listed
    1. `cargoSpaceReserved`: Capacity reserved for cargo that is listed to BUY on market
    1. `crew`: Capacity used for crewing carrier modules
    1. `freeSpace`: Unused capacity
    1. `microresourceCapacityTotal`: Capacity for all micro resources (on foot materials)
    1. `microresourceCapacityFree`: Unused micro resources capacity
    1. `microresourceCapacityUsed`: Capacity used for micro resources
    1. `microresourceCapacityReserved`: Capacity reserved for micro resources that are listed to BUY at bartender.

1. `itinerary`: Ships' itinerary
    1. `completed`: Completed jumps (list)
        1. `departureTime`: Time carrier left system, `None` if currently in system
        1. `arrivalTime`: Time carrier arrived in system
        1. `state`: ?? (always `success` in seen entries)
        1. `visitDurationSeconds`: How long carrier was/has been in system, in seconds
        1. `starsystem`: Name of starsystem
    1. `totalDistanceJumpedLY`: Total travel distance for carrier
    1. `currentJump`: System name for current plotted jump, otherwise `None`

1. `marketFinances`: Market information for the carrier
    1. `cargoTotalValue`: Computed value of entire cargo
    1. `allTimeProfit`: Total profit earned by carrier market
    1. `numCommodsForSale`: Number of commodities listed for sale on carrier market
    1. `numCommodsPurchaseOrders`: Number of commodities listed to buy on carrier market
    1. `balanceAllocForPurchaseOrders`: How many credits are allocated for fulfilling buy orders

1. `blackmarketFinances`: Black market information for the carrier.
    1. `cargoTotalValue`: Computed value of entire cargo
    1. `allTimeProfit`: Total profit earned by carrier market
    1. `numCommodsForSale`: Number of commodities listed for sale on carrier market
    1. `numCommodsPurchaseOrders`: Number of commodities listed to buy on carrier market
    1. `balanceAllocForPurchaseOrders`: How many credits are allocated for fulfilling buy orders

1. `finance`:
    1. `bankBalance`: Total credits in fleet carrier's account
    1. `bankReservedBalance`: Balance reserved for carrier upkeep
    1. `taxation`: Taxation rate for service use on the carrier
    1. `service_taxation`: Individual taxation rates for each service (Dict)
    	1. `bartender`: Taxation rate in % for bartender services
        1. `pioneersupplies`: Taxation rate in % for Pioneer supply services
        1. `rearm`: Taxation rate in % for rearming services
        1. `refuel`: Taxation rate in % for refuelling services
        1. `repair`: Taxation rate in % for repair services
        1. `shipyard`: Taxation rate in % for shipyard services
        1. `outfitting`: Taxation rate in % for outfitting services
    1. `numServices`: Number of services active on carrier
    1. `numOptionalServices`: Number of optional services active on carrier
    1. `debtThreshold`: Maximum negative credit balance before carrier automatically decommissions
    1. `maintenance`: Total cost pending for weekly maintenance
    1. `maintenanceToDate`: Total paid for maintenance in carrier lifetime
    1. `coreCost`: Current base weekly cost for carrier
    1. `servicesCost`: Current pending weekly cost for services, active or suspended
    1. `servicesCostToDate`: Total paid for services in carrier lifetime
    1. `jumpsCost`: Current pending cost for jumps made in the past week
    1. `numJumps`: Number of jumps made in the past week
    1. `bartender`: Detailed information about bartender micro resource services (Dict)
    	1. `microresourcesTotalValue`: Computed value of all micro resources (on foot materials) in stock
    	1. `allTimeProfit`: Total profit earned on sales of micro resources
    	1. `microresourcesForSale`: Number of micro resources listed for sale at bartender
    	1. `microresourcesPurchaseOrders`: Number of micro resources listed to buy at bartender
    	1. `profitHistory`: History of profit on recent transactions (List of ints)

1. `servicesCrew`: Enumerates assigned crew on the carrier. All services share the
                    same crewMember data as shown in first example.
   1. `refuel`: If refuel facility is installed, shows data on crew member.
        * `crewMember`:
            1. `name`: Crew member's name
            1. `gender`: Crew member's gender (`F` or `M`)
            1. `enabled`: Service enabled, `YES` or `NO`
            1. `faction`: Crew member's native faction
            1. `salary`: Weekly salary for crew member
            1. `avatarStr`: A string, containing data needed to represent the NPC's avatar.
                Example: `img://avatar:Seed:1913354398/Expression:eExpression_Positive/Labels:Gender|female-Faction|Federation-StarportContactType|CarrierRefuel-NPCType|StarportContact/DisableColourCorrection:0/`
            1. `lastEdit`: When crew member was last changed
        * `invoicesWeekToDate`: List containing elements as below
            1. `wages`: Amount paid/owed
            1. `from`: From date
            1. `until`: To date
            1. `type`: ??, known states `current` and `expected`
        * `status`: ?? known states: `ok`
   1. `repair`: Repair facility crew member information, if installed.
   1. `rearm`: Rearm facility crew member information, if installed.
   1. `shipyard`: Shipyard facility crew member information, if installed.
   1. `outfitting`: Outfitting facility crew member information, if installed.
   1. `voucherredemption`: Voucher redemption facility crew member information, if installed.
   1. `exploration`: Interstellar cartographer facility crew member information, if installed.
   1. `blackmarket`: Black market facility crew member information, if installed.
   1. `bartender`: Bartender market facility crew member information, if installed.
   1. `vistagenomics`: Exobiology facility crew member information, if installed.
   1. `pioneersupplies`: On-foot supply facility crew member information, if installed.

1. `cargo`: A list of all cargo items on board, in the following format:
    1. `commodity`: Non-localized commodity name
    1. `mission`: Whether the commodity is mission attached, boolean
    1. `qty`: Quantity of commodity (Always 1, in our experience)
    1. `value`: Value of commodity
    1. `stolen`: Whether the commodity is flagged as stolen
    1. `locName`: Localized name of commodity, follows carrier owner's localization

1. `orders`: Information on all buy and sell orders
    1. `commodities`: Buy and sell orders for standard cargo commodities
        1. `sales`: A list of sell orders, in the following format:
            1. `name`: The commodity name
            1. `stock`: The quantity for sale
            1. `price`: The sell price per item
            1. `blackmarket`: Boolean, `true` if this item is for sale on the black market
        1. `purchases`: A list of buy orders, in the following format:
            1. `name`: The commodity name
            1. `total`: The total buy order
            1. `outstanding`: The remaining outstanding to fulfil the buy order
            1. `price`: The buy price per item
    1. `onfootmicroresources`: Buy and sell orders for micro resources (on foot materials)
        1. `sales`: A Dict containing sell orders, or an empty list if there are no sell orders. The Dict key is the sell order ID (str) and the contents being a Dict in the following format:
            1. `id`: A unique identifier (int)
            1. `name`: The resource name
            1. `locName`: Localised resource name
            1. `stock`: The quantity for sale
            1. `price`: The sell price per item
        1. `purchases`: A list of buy orders, in the following format:
            1. `name`: The resource name
            1. `locName`: The localised resource name
            1. `total`: The total buy order
            1. `outstanding`: The remaining outstanding to fulfil the buy order
            1. `price`: The buy price per item

1. `carrierLocker`: All microresources (on foot materials) stored on the carrier, organised by type.
    1. `assets`: A list of all stored assets
        1. `id`: A unique identifier (int)
        1. `quantity`: The number of items stored
        1. `name`: The Resource name
        1. `locName`: The localised resource name
    1. `goods`: A list of all stored goods
        1. `id`: A unique identifier (int)
        1. `quantity`: The number of items stored
        1. `name`: The Resource name
        1. `locName`: The localised resource name
    1. `data`: A list of all stored data
        1. `id`: A unique identifier (int)
        1. `quantity`: The number of items stored
        1. `name`: The Resource name
        1. `locName`: The localised resource name

1. `reputation`: A list of faction reputations, in the following format.
    1. `majorFaction`: Name of faction
    1. `score`: Reputation score, 0 to 100
    
    Known valid majorFaction strings are `empire`, `federation`, `independent` and `alliance`.
   
1. `market`: Market information for the carrier, same as journal market entries
    1. `id`: Numerical unique ID for the market
    1. `name`: Market name, same as callsign
    1. `outpostType`: Always `fleetcarrier` in this context
    1. `imported`: List of imported commodities (carrier buys)
    1. `exported`: List of exported commodities (carrier sells)
    1. `services`: Array of service status
        1. `commodities`:
        1. `carrierfuel`:
        1. `refuel`:
        1. `repair`:
        1. `rearm`:
        1. `shipyard`:
        1. `outfitting`:
        1. `blackmarket`:
        1. `voucherredemption`:
        1. `exploration`:
        1. `carriermanagement`:
        1. `stationmenu`:
        1. `dock`:
        1. `crewlounge`:
        1. `engineer`:
        1. `contacts`: 
		
       For all the above, the following state strings are possible: `ok`: Active and available, 
	   `unavailable`: Not installed, or `private`: Only accessible by owner

    1. `economies`: An array, always set to the same:
        * `136`:
            1. `name`: Always `Carrier`
            1. `proportion`: Always `1`
    1. `prohibited`: Which commodities are prohibited on this station
    (dictionary, commodityId and key).
    	This isn't always populated.  When it is, it may be related to
	what's prohibited in the system in which the Fleet Carrier is
	situated.
    1. `commodities`: A list of available commodities
        1. `id`: Commodity ID
        1. `categoryname`: Commodity category name
        1. `name`: Non-localized string for commodity, as in journal data
        1. `stock`: Number of commodity in stock
        1. `buyPrice`: Price to buy commodity FROM carrier
        1. `sellPrice`: Price to sell commodity TO carrier
        1. `demand`: Number of commodity carrier is willing to buy
        1. `legality`: ?? 
        1. `meanPrice`: ??
        1. `demandBracket`: ??
        1. `stockBracket`: ??
        1. `locName`: Localized name of commodity, depending on fleet carrier owner's locale

1. `ships`: Shipyard information
   * `shipyard_list`: List containing available ships

1. `modules`: Available modules in outfitting

---

## Market

    	GET <https://companion.orerve.net/market>

Get the market data from the last docked station/fleet carrier/settlement and contains their services and status, what type of economics it has, what they import, export and what is forbidden. (And also the commodities, with price info and stock/demand)

CommodityId can be found in the [EDCD/FDevIDs repo](https://github.com/EDCD/FDevIDs)

1. `id` - Station ID
1. `name` - Station name
1. `outpostType` - What type of outpost it is, examples: `starport`, `fleetcarrier`
	- NB: Might not match the type in Journal events
1. `imported` - Which commodities are imported to this station (dictionary, commodity id and key)
   - Example `"128049162": "Cobalt"`
1. `exported` - Which commodities are exported from this station (dictionary, commodity id and key)
   - Example `"128049162": "Cobalt"`
1. `services` - What services the station provides and their status
	1. `dock` - Status of Dock (`ok`, `unavailable`, `private`)
	1. `contacts` - Status of Contacts (`ok`, `unavailable`, `private`)
	1. `exploration` - Status of Exploration (`ok`, `unavailable`, `private`)
	1. `commodities` - Status of Commodities (`ok`, `unavailable`, `private`)
	1. `refuel` - Status of Refuel (`ok`, `unavailable`, `private`)
	1. `repair` - Status of Repair (`ok`, `unavailable`, `private`)
	1. `rearm` - Status of Rearm (`ok`, `unavailable`, `private`)
	1. `outfitting` - Status of Outfitting (`ok`, `unavailable`, `private`)
	1. `shipyard` - Status of Shipyard (`ok`, `unavailable`, `private`)
	1. `crewlounge` - Status of Crewlounge (`ok`, `unavailable`, `private`)
	1. `searchrescue` - Status of Search & Rescue (`ok`, `unavailable`, `private`)
	1. `techbroker` - Status of Tech broker (`ok`, `unavailable`, `private`)
	1. `stationmenu` - Status of Station menu (`ok`, `unavailable`, `private`)
	1. `shop` - Status of shop (`ok`, `unavailable`, `private`)
	1. `engineer` - Status of engineers (`ok`, `unavailable`, `private`)
1. `economies` - What different types of economies you can expect from the station (dictionary, economyId as key)
	- Example `"23": { "name" : "HighTech", "proportion": 0.7 }`
1. `prohibited` - Which commodities are prohibited on this station (dictionary, commodityId as key)
   - Example `"128049162": "Cobalt"`
1. `commodities` - All available commodities and their info
	1. `id` - Commodity ID
	1. `name` - Commodity name
	1. `legality` - Status of the legality of this commodity at this station, empty string = legal
	1. `buyPrice` - The buy price at the moment of the visit
	1. `sellPrice` - The sell price at the moment of the visit
	1. `meanPrice` - The mean price at the momenf of the visit
	1. `demandBracket` - How much in demand this commodity is, value between `0` and `3`
	1. `stockBracket` - How much in stock this commodity has (bracket), value between `0` and `3`
	1. `stock` - How many items are in stock of this commodity
	1. `demand` - How many items the station has demand for (more is better)
	1. `statusFlags`
	1. `categoryName` - Category the commodity belongs to
	1. `locName` - Localised name

---

## Journal

Gives access to the authenticated players journals, all sessions are combined into a single file for a single request

### Request endpoints

-   `GET <https://companion.orerve.net/journal>`
-   `GET <https://companion.orerve.net/journal/[year (4 numbers)]/[month (2 numbers)]/[day (2 numbers)]>`

Example requests:

```plain
GET <https://companion.orerve.net/journal>
```

This request give you the current day journal data

```plain
GET <https://companion.orerve.net/journal/2020/12/30>
```

This request would give you the journal for the specific date, in this case the 30th December, 2020.

### Response codes

| Code  | Status          | Description                                                                                          |
| :---- | :-------------- | :--------------------------------------------------------------------------------------------------- |
| `200` | OK              | This means you got the entire journal for the specified request                                      |
| `204` | No Content | This means that the player has not (yet) played this day |
| `206` | Partial Content | The request did not get the entire journal, best solution is to keep trying until you get `200 - OK` |
| `401` | Unauthorized    | See [HTTP Status Codes](#http-status-codes) for more information                                     |

### Expected output

To know what kind of output you can except from the `/journal` endpoint, we recommend reading the [Journal Manual](http://hosting.zaonce.net/community/journal/v31/Journal_Manual_v31.pdf) to see what you can encounter.

---

## HTTP Status Codes

1. 401 - Unauthorized - Since the 'September Update' patch on 2019-09-17
   the CAPI servers use this HTTP status code to signal that the
   provided Access Token is invalid (certainly for when it's expired).

   The body containing:

		{"status":401,"message":"JWT has incorrect\/unexpected fields"}
1. 418 - "I'm a teapot" - used to signal that the service is down for
   maintenance.  Technically Frontier shouldn't be using this as it's a
   joke from a couple of April Fools' RFCs:
   <https://tools.ietf.org/html/rfc2324>
   <https://tools.ietf.org/html/rfc7168>

1. 422 - "Unprocessable Entity" - previously used to signal that a provided
   Access Token was invalid, likely due to it being expired.  This changed with
   the 'September Update' patch in 2019.
