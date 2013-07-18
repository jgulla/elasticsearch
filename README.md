elasticsearch
=============

Notes, slides and query examples from my Elasticsearch talk, given at UberConf 2013


Install & Setup
===============

tar zxf elasticsearch-0.90.2.tar.gz
cd elasticsearch-0.90.2/
./bin/elasticsearch -f

RestClient
==========
# Some query examples use RESTClient, an open source Java tool to work with REST APIs
# Download at https://code.google.com/p/rest-client/
# Examples are in rcq folder 

# Does it work?
curl -XGET 'http://localhost:9200'


Adding Data
===========

curl -XPOST http://localhost:9200/minecraft/mob/1 -d '{ "name" : "creeper", "friendly" : "false" }'

#Add one tweet, show URL breakdown

curl -XPOST http://localhost:9200/minecraft/mob/2 -d '{ "name" : "zombie", "friendly" : "false" }'
curl -XPOST http://localhost:9200/minecraft/mob/2 -d '{ "name" : "spider", "friendly" : "false" }'
curl -XPOST http://localhost:9200/minecraft/mob/4 -d '{ "name" : "villager", "friendly" : "true" }'
curl -XPOST http://localhost:9200/minecraft/mob/5 -d '{ "name" : "chicken", "friendly" : "true" }'
curl -XPOST http://localhost:9200/minecraft/mob/6 -d '{ "name" : "cow", "friendly" : "true" }'
curl -XPOST http://localhost:9200/minecraft/mob/7 -d '{ "name" : "baby zombie", "friendly" : "false" }'


Retrieving Data
===============
curl -XGET http://localhost:9200/minecraft/mob/1?pretty=true
curl -XGET http://localhost:9200/minecraft/mob/2?pretty=true
curl -XGET http://localhost:9200/minecraft/mob/3?pretty=true
curl -XGET http://localhost:9200/minecraft/mob/4?pretty=true
curl -XGET http://localhost:9200/minecraft/mob/5?pretty=true
curl -XGET http://localhost:9200/minecraft/mob/6?pretty=true
curl -XGET http://localhost:9200/minecraft/mob/7?pretty=true

Deleting Data
=============
#add bogus data
curl -XPOST http://localhost:9200/minecraft/mob/8 -d '{ "name" : "cat", "friendly" : "true" }'

# get it
curl -XGET 'http://localhost:9200/minecraft/mob/8?pretty'

# delete it
curl -XGET 'http://localhost:9200/minecraft/mob/8?pretty'

# get it
curl -XGET 'http://localhost:9200/minecraft/mob/8?pretty'


Search
======
#get everything
curl -XGET 'http://localhost:9200/minecraft/mob/_search?pretty'

#get one thing
curl -XGET 'http://localhost:9200/minecraft/mob/_search?q=name:creeper&pretty'

#get multiple things
curl -XGET 'http://localhost:9200/minecraft/mob/_search?q=friendly:true&pretty'
 

Updating Data
=============
curl -XPUT http://localhost:9200/minecraft/mob/1?pretty -d '{ "name" : "creeper", "friendly" : "false", "rare" : "false", "valuable" : "false" }'
curl -XPOST http://localhost:9200/minecraft/mob/2 -d '{ "name" : "zombie", "friendly" : "false", "rare" : "true", "valuable" : "false"  }'
curl -XPOST http://localhost:9200/minecraft/mob/2 -d '{ "name" : "spider", "friendly" : "false", "rare" : "true", "valuable" : "false"  }'
curl -XPOST http://localhost:9200/minecraft/mob/4 -d '{ "name" : "villager", "friendly" : "true", "rare" : "true", "valuable" : "false"  }'
curl -XPOST http://localhost:9200/minecraft/mob/5 -d '{ "name" : "chicken", "friendly" : "true", "rare" : "true", "valuable" : "true"  }'
curl -XPOST http://localhost:9200/minecraft/mob/6 -d '{ "name" : "cow", "friendly" : "true", "rare" : "false", "valuable" : "true"  }'
curl -XPOST http://localhost:9200/minecraft/mob/7 -d '{ "name" : "baby zombie", "friendly" : "false", "rare" : "true", "valuable" : "false" }'

# update in place
curl -POST 'http://localhost:9200/minecraft/mob/1/_update?pretty' -d ' {"script": "ctx._source.rare=false"}'

#Can't add new fields
curl -XPOST 'http://localhost:9200/minecraft/mob/1/_update?pretty' -d ' {"script": "ctx._source.newfield=test"}'



# Create a conflict
#first update a few times
curl -XPOST http://localhost:9200/minecraft/mob/7 -d '{ "name" : "baby zombie", "friendly" : "false", "rare" : "true", "valuable" : "false" }'
curl -XPOST http://localhost:9200/minecraft/mob/7?version=2 -d '{ "name" : "baby zombie", "friendly" : "false", "rare" : "true", "valuable" : "false" }'
curl -XPOST http://localhost:9200/minecraft/mob/7?version=2 -d '{ "name" : "baby zombie", "friendly" : "false", "rare" : "true", "valuable" : "false" }'

# try and post an old version
curl -XPOST http://localhost:9200/minecraft/mob/7?version=2 -d '{ "name" : "baby zombie", "friendly" : "false", "rare" : "true", "valuable" : "false" }'

Paging Through Results
======================
curl -XGET 'http://localhost:9200/minecraft/_search?size=1&pretty' 

curl -XGET 'http://localhost:9200/minecraft/_search?size=3&pretty' 

curl -XGET 'http://localhost:9200/minecraft/_search?size=50&pretty' 



#start point
curl -XGET 'http://localhost:9200/minecraft/_search?size=3&from=0&pretty' 
curl -XGET 'http://localhost:9200/minecraft/_search?size=3&from=1&pretty' 
curl -XGET 'http://localhost:9200/minecraft/_search?size=3&from=2&pretty' 

# counting
curl -XGET 'http://localhost:9200/minecraft/mob/_count?pretty'



Multiple Indices
================
# Minecraft is more than mobs. It's about blocks! 

#Next one won't work - need an ID
curl --XPUT http://localhost:9200/minecraft/block?pretty -d '{ "name" : "cobblestone", "rare" : "false", "valuable" : "false" }'

curl -XPOST http://localhost:9200/minecraft/block/1?pretty -d '{ "name" : "cobblestone", "rare" : "false", "valuable" : "false", "nether" : "false" }'
curl -XPOST http://localhost:9200/minecraft/block/2?pretty -d '{ "name" : "gravel", "rare" : "false", "valuable" : "false", "nether" : "false" }'
curl -XPOST http://localhost:9200/minecraft/block/3?pretty -d '{ "name" : "netherack", "rare" : "false", "valuable" : "false", "nether" : "true" }'
curl -XPOST http://localhost:9200/minecraft/block/4?pretty -d '{ "name" : "diamond", "rare" : "true", "valuable" : "true", "nether" : "false" }'
curl -XPOST http://localhost:9200/minecraft/block/5?pretty -d '{ "name" : "gold", "rare" : "true", "valuable" : "true", "nether" : "false" }'
curl -XPOST http://localhost:9200/minecraft/block/6?pretty -d '{ "name" : "coal", "rare" : "true", "valuable" : "true", "nether" : "false" }'
curl -XPOST http://localhost:9200/minecraft/block/7?pretty -d '{ "name" : "sand", "rare" : "false", "valuable" : "false", "nether" : "false" }'
curl -XPOST http://localhost:9200/minecraft/block/8?pretty -d '{ "name" : "redstone", "rare" : "false", "valuable" : "true", "nether" : "false" }'


# search multiple indices
curl -XGET 'http://localhost:9200/minecraft/mob,block/_search?q=rare:true&pretty'

#get just the count
curl -XGET 'http://localhost:9200/minecraft/mob,block/_count?q=rare:true&pretty'

# Search at a higher level (across all indicies)
curl -GET 'http://localhost:9200/minecraft/_search?q=rare:true&pretty' 

# Add items
curl -XPOST http://localhost:9200/minecraft/item/1?pretty -d '{ "name" : "string", "rare" : "true", "valuable" : "true", "crafted" : "false" }'
curl -XPOST http://localhost:9200/minecraft/item/2?pretty -d '{ "name" : "shovel", "rare" : "false", "valuable" : "true", "crafted" : "true" }'
curl -XPOST http://localhost:9200/minecraft/item/3?pretty -d '{ "name" : "pickaxe", "rare" : "false", "valuable" : "true", "crafted" : "true" }'
curl -XPOST http://localhost:9200/minecraft/item/4?pretty -d '{ "name" : "bonemeal", "rare" : "false", "valuable" : "true", "crafted" : "false" }'
curl -XPOST http://localhost:9200/minecraft/item/5?pretty -d '{ "name" : "torch", "rare" : "false", "valuable" : "true", "crafted" : "true" }'
curl -XPOST http://localhost:9200/minecraft/item/6?pretty -d '{ "name" : "flint", "rare" : "true", "valuable" : "true", "crafted" : "false" }'
curl -XPOST http://localhost:9200/minecraft/item/7?pretty -d '{ "name" : "chest", "rare" : "false", "valuable" : "true", "crafted" : "true" }'

# search multiple indices
curl -XGET 'http://localhost:9200/minecraft/mob,block,item/_search?q=rare:true&pretty'

#get just the count
curl -XGET 'http://localhost:9200/minecraft/mob,block,item/_count?q=rare:true&pretty'

# Search at a higher level (across all indicies)
curl -GET 'http://localhost:9200/minecraft/_search?q=rare:true&pretty' 

#change around which index to use, see differences


Mappings
========
# Now create an inventory so we can have a count
curl -POST http://localhost:9200/minecraft/inventory/1 -d '{ "item" : "pickaxe", "quantity" : 1, "enchanted" : "false } '

# Get mapping
curl -XGET 'http://localhost:9200/minecraft/inventory/_mapping?pretty'
# notice enchanted is a string
#can't change fields
!
#start over. delete index
curl -XDELETE 'http://localhost:9200/minecraft/inventory'

#Now re-add data
curl -POST http://localhost:9200/minecraft/inventory/1 -d '{ "item" : "pickaxe", "quantity" : 1, "enchanted" : false, "player" : "nicholas" } '
curl -POST http://localhost:9200/minecraft/inventory/2 -d '{ "item" : "shovel", "quantity" : 1, "enchanted" : false, "player" : "nicholas" } '
curl -POST http://localhost:9200/minecraft/inventory/3 -d '{ "item" : "sword", "quantity" : 1, "enchanted" : true, "player" : "nicholas" } '
curl -POST http://localhost:9200/minecraft/inventory/4 -d '{ "item" : "coal", "quantity" : 55, "enchanted" : false, "player" : "nicholas" } '
curl -POST http://localhost:9200/minecraft/inventory/5 -d '{ "item" : "redstone", "quantity" : 21, "enchanted" : false, "player" : "nicholas" } '
curl -POST http://localhost:9200/minecraft/inventory/6 -d '{ "item" : "diamond", "quantity" : 17, "enchanted" : false, "player" : "nicholas" } '
curl -POST http://localhost:9200/minecraft/inventory/7 -d '{ "item" : "iron", "quantity" : 47, "enchanted" : false, "player" : "nicholas" } '
curl -POST http://localhost:9200/minecraft/inventory/8 -d '{ "item" : "pickaxe", "quantity" : 1, "enchanted" : false, "player" : "dad" } '
curl -POST http://localhost:9200/minecraft/inventory/9 -d '{ "item" : "coal", "quantity" : 4, "enchanted" : false, "player" : "dad" } '
curl -POST http://localhost:9200/minecraft/inventory/10 -d '{ "item" : "diamond", "quantity" : 0, "enchanted" : false, "player" : "dad" } '
curl -POST http://localhost:9200/minecraft/inventory/11 -d '{ "item" : "sword", "quantity" : 1, "enchanted" : false, "player" : "dad" } '

# What does nick have?
curl -XGET 'http://localhost:9200/minecraft/inventory/_search?q=player:nicholas&pretty'
 
#what does dad have?
curl -XGET 'http://localhost:9200/minecraft/inventory/_search?q=player:dad&pretty'
 
# Who has diamonds?
curl -XGET 'http://localhost:9200/minecraft/inventory/_search?q=item:diamond&pretty' 

Filters & Queries
=================
#Load up with RestClient   


search_inv_all.rcq
search_field_req.rcq
search_range_req.rcq
search_filter_and_query.rcq # play w/ searching for different items

Clustering
./bin/elasticsearch -f
 http://localhost:9200/_cluster/health?pretty

#next tab
./bin/elasticsearch -f
 http://localhost:9200/_cluster/health?pretty

#Then kill one
 

More Queries
============
#Load up with RestClient   
Enchanted Pickaxe: search_multi_field_query_and_enchanted.rcq

search_query_and_range.rcq
search_multi_field_query.rcq
search_multi_field_query_and.rcq
search_multi_field_query_and_enchanted.rcq

#find enchanted sword, bool format     
search_multi_field_query_item_and_player_bool.rcq

# Also by player (3 search terms)
search_multi_field_query_item_and_enchanted_and_player_bool.rcq


Plugins
=======
#Site plugin.
cd ./elasticsearch-head
open index.html

#GREAT place to help build queries! go to query tab
https://github.com/mobz/elasticsearch-head

#elasticHQ
http://www.elastichq.org/gettingstarted.html
unzip royrusso-elasticsearch-HQ-7d52c5d.zip # (note filename may change)
cd ./elasticsearch-head/royrusso-elasticsearch-HQ-7d52c5d
open index.html
#connect to http://localhost:9200 (top of page)