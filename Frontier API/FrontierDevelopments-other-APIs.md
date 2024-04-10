# Other Frontier Developers APIs for Elite Dangerous

There are a few other APIs provider by Frontier for the game Elite
Dangerous that they've either explicitly told us about or never made a
fuss about us using.

These tend to relate to general game information, nothing
commander-specific.

No, there's no public API for general game data like "market prices at a
given station".

## Community Goals

As of 2021-05-27 we know of:

    https://api.orerve.net/2.0/website/initiatives/list?lang=en 

1. Note the `lang=en` parameter passed.  These languages are known to be
  supported:

    1. `de` - German
    1. `en` - English
    1. `fr` - French
    1. `es` - Spanish
    1. `pt` - Portugese (Brazilian?)
    1. `ru` - Russian

1. The output format will differ depending on the `Accept` HTTP header
   you send.
   
    1. `Accept: */*` will return JSON, as per the response header
     `Content-Type: application/json`.
    1. `Accept: text/xml` will return XML as per the response header
     `Content-Type: application/xml`.

## GalNet Posts

As of 2021-05-27 we know of these sources for GalNet news posts:

1. Community site RSS feed -
   `https://community.elitedangerous.com/galnet-rss`.

1. cms.zaonce.net - `https://cms.zaonce.net/en-GB/jsonapi/node/galnet_article`

    1. With `Accept: */*` this returns JSON with the response header
     `content-type: application/vnd.api+json`.
    1. You can also filter to specific articles with, e.g.

          `https://cms.zaonce.net/en-GB/jsonapi/node/galnet_article?filter%5Bfield_slug%5D=tritium-mining-marks-alliance-election-day`

      Note the URL %-escaping of `[...]` to `%5B...%5D`.

There was another endpoint that could give you JSON output, but it was
 hosted on `elitedangerous-website-backend-production.elitedangerous.com`.
That hostname no longer resolves.

## See Also

Someone else has been investigating this -
https://gist.github.com/corenting/b6ac5cf8f446f54856e08b6e287fe835
