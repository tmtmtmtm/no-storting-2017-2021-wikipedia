Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Tracking page
=====================

I set up a Listeria page for the term. Version before updates is at:
https://www.wikidata.org/w/index.php?title=Wikidata:WikiProject_every_politician/Norway/data/Storting/2017-2021&oldid=1241041205

Only one person currently has an 'elected in' qualifier.

Step 2: Set up the metadata
===========================

Edited [add_P39.js script](add_P39.js) to configure the P39 data, and
source URL.

Step 3: Get local copy of Wikidata information
==============================================

    wd ee --dry add_P39.js | jq -r '"\(.claims.P39.value) \(.claims.P39.qualifiers.P2937)"' | 
      xargs wd sparql members.js | tee wikidata.json

Step 4: Scrape
==============

Comparison/source = https://no.wikipedia.org/wiki/Liste_over_stortingsrepresentanter_2017%E2%80%932021

    wb ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
      xargs bundle exec ruby scraper.rb | tee wikipedia.csv

The constituency info here is all mismatched, as Wikidata has distinct
items for the electoral district but Wikipedia links to the associated
administrative district instead. For now I've told the scraper to just
ignore this column: we can always add them back later if we need them
for something.

Step 6: Add missing 'elected in' qualifiers
===========================================

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json | fgrep P2715 |
      wd aq --batch --summary "Add elected in qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

167 additions made as https://tools.wmflabs.org/editgroups/b/wikibase-cli/b5778feee16ca

One person (Q6513706) couldn't be automatically updated as he has two
P39s, and we don't have a start date field here to match with. If there
had been more, I'd have considered rescraping with a fixed date field
added, but with just one it's easier to add the election qualifier by hand.

Step 7: Refresh the Tracking Page
==================================

New version at 
https://www.wikidata.org/w/index.php?title=Wikidata:WikiProject_every_politician/Norway/data/Storting/2017-2021&oldid=1241053812

