---
layout: default
title: Tutorial - Pairing Beers with Analytics
---

## Welcome to the Analytics DP! ##
This document introduces the main features of the new Couchbase Analytics through examples.
Since few things in life are more important than choosing the right beer to pair with one's main dish,
this example explores the pairing of Analytics and beers.
More specifically, it shows how to create a set of Analytics shadow datasets based on the Couchbase beer-sample bucket,
together with a set of illustrative queries, as a quick way to introduce you to the new "Analytics user experience".
The complete set of steps to create and connect the sample datasets, along with a collection of runnable SQL++
queries and their expected results, are included.

This document assumes that you are comfortable using Couchbase Server and know how to query it using N1QL.
It assumes that you already have a running instance of Couchbase Server 4.5 on your favorite machine
and that you know how to interact with it using the Couchbase Console.
It also assumes that you have the Couchbase beer-sample example data bucket installed on that system.
Finally, this document assumes that you have already downloaded and successfully installed the Analytics DP
and that you have verified that it is up and running (for example, by issuing a simple SQL++ test query as shown below:

    "Let there be beer!";

As you read through this document, you should try each step for yourself on your own Analytics instance.
You can use whatever your favorite Analytics interface is to do this (for example, Analytics Workbench, cbq, or curl);
Once you have reached the end of this tutorial, you will be armed and dangerous,
having all the basic Analytics knowledge that you'll need to start down the path of exploring the power of NoSQL data analytics.
As you have already heard by now, this will hopefully prove to be a freeing experience:
Analytics SQL++ queries never touch your Couchbase data servers, running instead (in parallel) on real-time shadow copies of your data.
As a result, you can find yourself in a world where you can "ask the system anything!" -- you won't need to worry about
slowing down the front-end machines with complex queries, you won't have to create covering indexes to ask queries,
and the Analytics query language (SQL++) won't try to keep you from asking queries that would be too perforamance-costly
to ask against your front-end data.

## The World of Data in Analytics ##
In this section, you will learn about the Analytics world model for data.

### Organizing Data in Analytics ###

The top-level organizing concept in the Analytics data world is the _dataverse_.
A dataverse,short for "data universe", is a place (similar to a database or a schema in a relational DBMS)
in which to create and manage data types, datasets, and other artifacts for a given Analytics application.
In the Analytics DP, all of the data types that you might need come pre-installed, and the kind of datasets
that you will create are called "shadow datasets".
These datasets are containers - collections of JSON objects - that contain real-time shadow copies of selected front-end data.
When you start using a Analytics instance for the first time, it starts out "empty";
it contains no data other than the Analytics system catalogs (which live in a special dataverse called the "Metadata" dataverse).
To store your data in Analytics, you can create a dataverse and then use it to hold the _datasets_ for your own data.
However, you also get a "default" dataverse for free, and Analytics will just use that if you don't create a new one.
Let's put these concepts to work.

Our sample scenario here involves information about beers and their breweries.
We'll use the "Default" dataverse, to keep things simple, so our first task is to tell Analytics about
the Couchbase Server data that we want to shadow and the shadow datasets where we want it to live.
The following Analytics DDL statements show how we can tell Analytics about the "beer-sample" bucket in Couchbase Server,
where all of the beer and brewery information resides, and ask Analytics to shadow the data using two datasets, breweries and beers.

    CREATE BUCKET beerBucket WITH {"name":"beer-sample","nodes":"127.0.0.1"};
    CREATE SHADOW DATASET breweries ON beerBucket WHERE `type` = "brewery";
    CREATE SHADOW DATASET beers ON beerBucket WHERE `type` = "beer";

The first statement (_CREATE BUCKET_) gives Analytics the information needed to access the data of interest from Couchbase Server.
The next two statements (_CREATE SHADOW DATASET_) create the target datasets in Analytics for the information of interest.
Notice how _WHERE_ clauses are utilized to direct beer-related data into separate, type-specific datasets for easier querying.
Each of these datasets will be hash-partitioned (shared) across all of the nodes running instances of the Analytics service.
To initiate the shadowing relationship of these datasets to the front-end data, one more step is needed, namely:

    CONNECT BUCKET beerBucket WITH {"password":""};

> **Note:** The Developer Preview saves the username and password for Couchbase Server as plain text in the clear.

Once you have run this statement, Analytics will begin making its copy of the front-end data and continuously monitor it for changes.

### What's Lurking in the Shadows? ###

The next thing you will probably want to do is to verify that your shadow datasets are really there and being populated.
The following SQL++ _SELECT_ statements are one good way to accomplish that:

    SELECT VALUE ds FROM Metadata.`Dataset` ds WHERE ds.DataverseName = "Default";

    SELECT VALUE COUNT(*) FROM breweries;
    SELECT * FROM breweries ORDER BY name LIMIT 1;

    SELECT VALUE COUNT(*) FROM beers;
    SELECT * FROM beers ORDER BY name LIMIT 1;

The first statement above looks in the Analytics system catalogs for datasets that have been created in the "Default" dataverse.
At this point, assuming that you are just getting started, you will see only your two new shadow datasets:

    [ {
        "GroupName": "DEFAULT_NG_ALL_NODES",
        "MetatypeName": "DCPMeta",
        "CompactionPolicyProperties": [
            {
                "Value": "1073741824",
                "Name": "max-mergable-component-size"
            },
            {
                "Value": "5",
                "Name": "max-tolerance-component-count"
            }
        ],
        "Hints": [

        ],
        "WhereClause": "`type` = \"beer\"",
        "DatasetType": "INTERNAL",
        "Timestamp": "Thu Oct 13 15:43:26 PDT 2016",
        "DatasetId": 102,
        "DatatypeDataverseName": "Metadata",
        "InternalDetails": {
            "FileStructure": "BTREE",
            "PartitioningKey": [
                [
                    "id"
                ]
            ],
            "PartitioningStrategy": "HASH",
            "Autogenerated": false,
            "PrimaryKey": [
                [
                    "id"
                ]
            ],
            "KeySourceIndicator": [
                1
            ]
        },
        "BucketDataverseName": "Default",
        "BucketName": "beerBucket",
        "DatatypeName": "AnyObject",
        "DatasetName": "beers",
        "DataverseName": "Default",
        "CompactionPolicy": "prefix",
        "MetatypeDataverseName": "Metadata",
        "PendingOp": 0
    }, {
        "GroupName": "DEFAULT_NG_ALL_NODES",
        "MetatypeName": "DCPMeta",
        "CompactionPolicyProperties": [
            {
                "Value": "1073741824",
                "Name": "max-mergable-component-size"
            },
            {
                "Value": "5",
                "Name": "max-tolerance-component-count"
            }
        ],
        "Hints": [

        ],
        "WhereClause": "`type` = \"brewery\"",
        "DatasetType": "INTERNAL",
        "Timestamp": "Thu Oct 13 15:43:26 PDT 2016",
        "DatasetId": 101,
        "DatatypeDataverseName": "Metadata",
        "InternalDetails": {
            "FileStructure": "BTREE",
            "PartitioningKey": [
                [
                    "id"
                ]
            ],
            "PartitioningStrategy": "HASH",
            "Autogenerated": false,
            "PrimaryKey": [
                [
                    "id"
                ]
            ],
            "KeySourceIndicator": [
                1
            ]
        },
        "BucketDataverseName": "Default",
        "BucketName": "beerBucket",
        "DatatypeName": "AnyObject",
        "DatasetName": "breweries",
        "DataverseName": "Default",
        "CompactionPolicy": "prefix",
        "MetatypeDataverseName": "Metadata",
        "PendingOp": 0
    } ]

The next two pairs of SQL++ statements above ask Analytics to give you record counts and a sample record from each of the shadow datasets.
The four resulting return values should be as follows:

    [ 1412 ]

    [ {
        "breweries": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -97.7697,
                "lat": 30.2234
            },
            "country": "United States",
            "website": "http:\/\/512brewing.com\/",
            "code": "78745",
            "address": [
                "407 Radam, F200"
            ],
            "city": "Austin",
            "phone": "512.707.2337",
            "name": "(512) Brewing Company",
            "description": "(512) Brewing Company is a microbrewery located in the heart of Austin that brews for the community using as many local, domestic and organic ingredients as possible.",
            "state": "Texas",
            "type": "brewery",
            "updated": "2010-07-22 20:00:20"
        }
    } ]

    [ 5891 ]

    [ {
        "beers": {
            "abv": 0.0,
            "name": "#17 Cream Ale",
            "upc": 0,
            "description": "",
            "style": "American-Style Lager",
            "brewery_id": "big_ridge_brewing",
            "type": "beer",
            "category": "North American Lager",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        }
    } ]


## SQL++: Querying Your Analytics Data ##

Congratulations! You now have your Couchbase Server beer-related data being shadowed in Analytics.
You can start running ad hoc queries against your breweries and beers datasets.

Analytics supports the SQL++ query language, and it is a
SQL-inspired language designed for working with semistructured data.
SQL++ has much in common with SQL, but there are differences due to the data model that SQL++ is designed to serve.
SQL was designed in the 1970's to interact with the flat, schema-ified world of relational databases.
SQL++ is designed for the nested, schema-less (or schema-optional, in Analytics) world of NoSQL systems.
SQL++ offers a mostly familiar paradigm for experienced SQL users to use to query and manipulate data in Analytics.
SQL++ is also related to N1QL, the current query language used in Couchbase Server.
SQL++ is really a functional superset of N1QL that is a bit closer to SQL,
and the differences between N1QL and SQL++ will eventually disappear (that is, SQL++ is really the future of N1QL).

In this section we introduce SQL++ via a set of example queries, along with their expected results,
based on the data above, to help you get started.
Many of the most important features of SQL++ are presented in this set of representative queries.
For more details, see
[SQL++ Language Reference](1_intro.html),
and a complete list of the built-in functions is available in the [Function Reference](function-ref.html).

SQL++ is a highly composable expression language.
Even the very simple expression "1+1;" is a valid SQL++ query that evaluates to 2.
(Try it for yourself!)

Let's go ahead and try writing some queries and see about learning SQL++ through examples.

### Query 0 - Key-Based Lookup ###
For our first query, let's find a particular brewery based on its Couchbase Server key.
We can do this for the Kona Brewing company as follows:

    SELECT meta(bw) AS meta, bw AS data
    FROM breweries bw
    WHERE meta(bw).id = 'kona_brewing';

As in SQL, the query's _FROM_ clause binds the variable `bw` incrementally to the data instances residing in
the dataset named breweries.
Its _WHERE_ clause  selects only those bindings having the primary key of interest;
the key is accessed (as in N1QL) by using the _meta_ function to get to the meta-information about the records.
The _SELECT_ clause returns all of the meta-information plus the data value (the selected brewery record in this case)
for each binding that satisfies the predicate.

The expected result for this query is as follows:

    [ {
        "data": {
            "geo": {
                "accuracy": "RANGE_INTERPOLATED",
                "lon": -155.996,
                "lat": 19.642
            },
            "country": "United States",
            "website": "http:\/\/www.konabrewingco.com",
            "code": "96740",
            "address": [
                "75-5629 Kuakini Highway"
            ],
            "city": "Kailua-Kona",
            "phone": "1-808-334-1133",
            "name": "Kona Brewing",
            "description": "",
            "state": "Hawaii",
            "type": "brewery",
            "updated": "2010-07-22 20:00:20"
        },
        "meta": {
            "vbid": 30,
            "cas": 569488800022528,
            "flags": 50331648,
            "id": "kona_brewing",
            "seq": 56
        }
    } ]

Notice how the resulting record of interest has two fields whose names were requested in the _SELECT_ clause.

### Query 1 - Exact-Match Lookup ###
The SQL++ language, like SQL, supports a variety of different predicates.
For our next query, let's find the same brewery information but in a slightly simpler or cleaner way based only on the data:

    SELECT VALUE bw
    FROM breweries bw
    WHERE bw.name = 'Kona Brewing';

This query's expected result is:

    [ {
        "geo": {
            "accuracy": "RANGE_INTERPOLATED",
            "lon": -155.996,
            "lat": 19.642
        },
        "country": "United States",
        "website": "http:\/\/www.konabrewingco.com",
        "code": "96740",
        "address": [
            "75-5629 Kuakini Highway"
        ],
        "city": "Kailua-Kona",
        "phone": "1-808-334-1133",
        "name": "Kona Brewing",
        "description": "",
        "state": "Hawaii",
        "type": "brewery",
        "updated": "2010-07-22 20:00:20"
    } ]

In SQL++, you can select a single value (whether it be an atomic or scalar value or a record value
or an unordered or ordered collection value) by using a _SELECT VALUE_ clause as was done above.
If you instead use the more SQL-familier _SELECT_ clause, SQL++ will return records instead of values,
and since a given query may _SELECT_ multiple values in its result (like our first query did),
you will get a slightly differently shaped result using _SELECT_ instead of _SELECT VALUE_:

    [ {
        "bw": {
            "geo": {
                "accuracy": "RANGE_INTERPOLATED",
                "lon": -155.996,
                "lat": 19.642
            },
            "country": "United States",
            "website": "http:\/\/www.konabrewingco.com",
            "code": "96740",
            "address": [
                "75-5629 Kuakini Highway"
            ],
            "city": "Kailua-Kona",
            "phone": "1-808-334-1133",
            "name": "Kona Brewing",
            "description": "",
            "state": "Hawaii",
            "type": "brewery",
            "updated": "2010-07-22 20:00:20"
        }
    } ]

### Query 2 - Other Query Filters ###
SQL++ can apply ranges and other conditions on any data type that supports the appropriate set of comparators.
As an example, the next query applies a range condition together with a string condition to select breweries:

    SELECT VALUE bw
    FROM breweries bw
    WHERE bw.geo.lat > 60.0
      AND bw.name LIKE '%Brewing%'
    ORDER BY bw.name;

The expected result for this query is as follows:
```
    [ {
        "geo": {
            "accuracy": "RANGE_INTERPOLATED",
            "lon": -149.439,
            "lat": 61.5816
        },
        "country": "United States",
        "website": "",
        "code": "99654",
        "address": [
            "238 North Boundary Street"
        ],
        "city": "Wasilla",
        "phone": "1-907-373-4782",
        "name": "Great Bear Brewing",
        "description": "",
        "state": "Alaska",
        "type": "brewery",
        "updated": "2010-07-22 20:00:20"
    }, {
        "geo": {
            "accuracy": "ROOFTOP",
            "lon": -149.844,
            "lat": 61.1473
        },
        "country": "United States",
        "website": "http:\/\/www.midnightsunbrewing.com\/",
        "code": "99507",
        "address": [
            "8111 Dimond Hook Drive"
        ],
        "city": "Anchorage",
        "phone": "1-907-344-1179",
        "name": "Midnight Sun Brewing Co.",
        "description": "Since firing up its brew kettle in 1995, Midnight Sun Brewing Company
        has become a serious yet creative force on the American brewing front. From concept to
        glass, Midnight Sun relies on an art marries science approach, mixing tradition with
        innovation, to design and craft bold, distinctive beers for Alaska...and beyond.
        We at Midnight Sun find inspiration in the untamed spirit and rugged beauty of
        the Last Frontier and develop unique beers with equally appealing names and labels.
        But the company's true focus remains in its dedication to producing consistently
        high-quality beers that provide satisfying refreshment in all seasons... for Alaskans
        and visitors alike.  From our Pacific Northwest locale, we offer our wonderful beers
        on draft throughout Alaska and in 22-ounce bottles throughout Alaska and Oregon.
        We invite you to visit our hardworking, little brewery in South Anchorage every
        chance you get!",
        "state": "Alaska",
        "type": "brewery",
        "updated": "2010-07-22 20:00:20"
    }, {
        "geo": {
            "accuracy": "APPROXIMATE",
            "lon": -149.9,
            "lat": 61.2181
        },
        "country": "United States",
        "website": "",
        "code": "",
        "address": [

        ],
        "city": "Anchorage",
        "phone": "",
        "name": "Railway Brewing",
        "description": "",
        "state": "Alaska",
        "type": "brewery",
        "updated": "2010-07-22 20:00:20"
    }, {
        "geo": {
            "accuracy": "ROOFTOP",
            "lon": -147.622,
            "lat": 64.9583
        },
        "country": "United States",
        "website": "http:\/\/www.ptialaska.net\/~gbrady\/",
        "code": "99708",
        "address": [
            "2195 Old Steese Highway"
        ],
        "city": "Fox",
        "phone": "(907) 452-2739",
        "name": "Silver Gulch Brewing Company",
        "description": "Silver Gulch Brewing and Bottling Co. has been in operation since
        February 1998 in the small mining community of Fox, Alaska, located about 10 miles
        north of Fairbanks on the Steese Highway. Silver Gulch Brewing grew from brewmaster
        Glenn Brady's home-brewing efforts in 5-gallon batches to its current capacity of
        24-barrel (750 gallon) batches.",
        "state": "Alaska",
        "type": "brewery",
        "updated": "2010-07-22 20:00:20"
    }, {
        "geo": {
            "accuracy": "RANGE_INTERPOLATED",
            "lon": -149.896,
            "lat": 61.2196
        },
        "country": "United States",
        "website": "http:\/\/www.alaskabeers.com\/",
        "code": "99501",
        "address": [
            "717 W. 3rd Ave"
        ],
        "city": "Anchorage",
        "phone": "(907) 277-7727",
        "name": "Sleeping Lady Brewing Company",
        "description": "",
        "state": "Alaska",
        "type": "brewery",
        "updated": "2010-07-22 20:00:20"
    } ]
```
### Query 3 (and friends) - Equijoin ###
In addition to simply binding variables to data instances and returning them "whole",
an SQL++ query can construct new records to return based on combinations of variable bindings.
This gives SQL++ the power to do projections and joins much like those done using multi-table _FROM_ clauses in SQL.
For example, if we wanted a list of all breweries paired with their associated beers,
with the list enumerating the brewery name and the beer name for each such pair.
We can do this as follows in SQL++ (which also limits the answer set size to at most 20 results):

    SELECT bw.name AS brewer, br.name AS beer
    FROM breweries bw, beers br
    WHERE br.brewery_id = meta(bw).id
    ORDER BY bw.name, br.name
    LIMIT 20;

The result of this query is a sequence of new records, one for each brewery/beer pair.
Each instance in the result will be a record containing two fields, "brewer" and "beer",
containing the brewery's name and the beer's name, respectively, for each brewery/beer pair.
Notice how the use of a SQL-style _SELECT_ clause, as opposed to the new SQL++ _SELECT VALUE_
clause, automatically results in the construction of a new record value for each result.

The expected result of this example SQL++ join query is shown below:

    [ {
        "brewer": "(512) Brewing Company",
        "beer": "(512) ALT"
    }, {
        "brewer": "(512) Brewing Company",
        "beer": "(512) Bruin"
    }, {
        "brewer": "(512) Brewing Company",
        "beer": "(512) IPA"
    }, {
        "brewer": "(512) Brewing Company",
        "beer": "(512) Pale"
    }, {
        "brewer": "(512) Brewing Company",
        "beer": "(512) Pecan Porter"
    }, {
        "brewer": "(512) Brewing Company",
        "beer": "(512) Whiskey Barrel Aged Double Pecan Porter"
    }, {
        "brewer": "(512) Brewing Company",
        "beer": "(512) Wit"
    }, {
        "brewer": "(512) Brewing Company",
        "beer": "One"
    }, {
        "brewer": "21st Amendment Brewery Cafe",
        "beer": "21A IPA"
    }, {
        "brewer": "21st Amendment Brewery Cafe",
        "beer": "563 Stout"
    }, {
        "brewer": "21st Amendment Brewery Cafe",
        "beer": "Amendment Pale Ale"
    }, {
        "brewer": "21st Amendment Brewery Cafe",
        "beer": "Bitter American"
    }, {
        "brewer": "21st Amendment Brewery Cafe",
        "beer": "Double Trouble IPA"
    }, {
        "brewer": "21st Amendment Brewery Cafe",
        "beer": "General Pippo's Porter"
    }, {
        "brewer": "21st Amendment Brewery Cafe",
        "beer": "North Star Red"
    }, {
        "brewer": "21st Amendment Brewery Cafe",
        "beer": "Oyster Point Oyster Stout"
    }, {
        "brewer": "21st Amendment Brewery Cafe",
        "beer": "Potrero ESB"
    }, {
        "brewer": "21st Amendment Brewery Cafe",
        "beer": "South Park Blonde"
    }, {
        "brewer": "21st Amendment Brewery Cafe",
        "beer": "Watermelon Wheat"
    }, {
        "brewer": "3 Fonteinen Brouwerij Ambachtelijke Geuzestekerij",
        "beer": "Drie Fonteinen Kriek"
    } ]

If we were feeling lazy, for example, while browsing our data casually, we can use _SELECT *_ in SQL++ to
return all of the matching user/message data:

    SELECT *
    FROM breweries bw, beers br
    WHERE br.brewery_id = meta(bw).id
    ORDER BY bw.name, br.name
    LIMIT 20;

In SQL++, this _SELECT *_ query will produce a new nested record for each user/message pair.
Each result record contains one field (named after the "breweries" variable) to hold the brewery record
and another field (named after the "beers" variable) to hold the matching beer record.
Note that the nested nature of this SQL++ _SELECT *_ result is different than traditional SQL,
as SQL was not designed to handle the richer, nested data model that underlies the design of SQL++.

The expected result of this version of the SQL++ join query for our sample data set is as follows:

    [ {
        "br": {
            "abv": 6.0,
            "name": "(512) ALT",
            "upc": 0,
            "description": "(512) ALT is a German-style amber ale that is fermented cooler than
            typical ales and cold conditioned like a lager. ALT means “old” in German and refers
            to a beer style made using ale yeast after many German brewers had switched to newly
            discovered lager yeast. This ale has a very smooth, yet pronounced, hop bitterness
            with a malty backbone and a characteristic German yeast character. Made with 98% Organic 2-row
            and Munch malts and US noble hops.",
            "style": "German-Style Brown Ale\/Altbier",
            "brewery_id": "512_brewing_company",
            "type": "beer",
            "category": "German Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -97.7697,
                "lat": 30.2234
            },
            "country": "United States",
            "website": "http:\/\/512brewing.com\/",
            "code": "78745",
            "address": [
                "407 Radam, F200"
            ],
            "city": "Austin",
            "phone": "512.707.2337",
            "name": "(512) Brewing Company",
            "description": "(512) Brewing Company is a microbrewery located in the heart of
            Austin that brews for the community using as many local, domestic and organic
            ingredients as possible.",
            "state": "Texas",
            "type": "brewery",
            "updated": "2010-07-22 20:00:20"
        }
    }, {
        "br": {
            "abv": 7.6,
            "name": "(512) Bruin",
            "upc": 0,
            "description": "At once cuddly and ferocious, (512) BRUIN combines a smooth,
            rich maltiness and mahogany color with a solid hop backbone and stealthy
            7.6% alcohol. Made with Organic 2 Row and Munich malts, plus Chocolate and
            Crystal malts, domestic hops, and a touch of molasses, this brew has notes of
            raisins, dark sugars, and cocoa, and pairs perfectly with food and the crisp
            fall air.",
            "style": "American-Style Brown Ale",
            "brewery_id": "512_brewing_company",
            "type": "beer",
            "category": "North American Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -97.7697,
                "lat": 30.2234
            },
            "country": "United States",
            "website": "http:\/\/512brewing.com\/",
            "code": "78745",
            "address": [
                "407 Radam, F200"
            ],
            "city": "Austin",
            "phone": "512.707.2337",
            "name": "(512) Brewing Company",
            "description": "(512) Brewing Company is a microbrewery located in the heart of
            Austin that brews for the community using as many local, domestic and organic
            ingredients as possible.",
            "state": "Texas",
            "type": "brewery",
            "updated": "2010-07-22 20:00:20"
        }
    }, {
        "br": {
            "abv": 7.0,
            "name": "(512) IPA",
            "upc": 0,
            "description": "(512) India Pale Ale is a big, aggressively dry-hopped American
             IPA with smooth bitterness (~65 IBU) balanced by medium maltiness. Organic
             2-row malted barley, loads of hops, and great Austin water create an ale with
             apricot and vanilla aromatics that lure you in for more.",
            "style": "American-Style India Pale Ale",
            "brewery_id": "512_brewing_company",
            "type": "beer",
            "category": "North American Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -97.7697,
                "lat": 30.2234
            },
            "country": "United States",
            "website": "http:\/\/512brewing.com\/",
            "code": "78745",
            "address": [
                "407 Radam, F200"
            ],
            "city": "Austin",
            "phone": "512.707.2337",
            "name": "(512) Brewing Company",
            "description": "(512) Brewing Company is a microbrewery located in the heart
            of Austin that brews for the community using as many local, domestic and
            organic ingredients as possible.",
            "state": "Texas",
            "type": "brewery",
            "updated": "2010-07-22 20:00:20"
        }
    }, {
        "br": {
            "abv": 5.8,
            "name": "(512) Pale",
            "upc": 0,
            "description": "With Organic 2-row malted barley, (512) Pale is a copper
            colored American Pale Ale that balances earthy hop bitterness and hop flavor
            with a rich malty body.",
            "style": "American-Style Pale Ale",
            "brewery_id": "512_brewing_company",
            "type": "beer",
            "category": "North American Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -97.7697,
                "lat": 30.2234
            },
            "country": "United States",
            "website": "http:\/\/512brewing.com\/",
            "code": "78745",
            "address": [
                "407 Radam, F200"
            ],
            "city": "Austin",
            "phone": "512.707.2337",
            "name": "(512) Brewing Company",
            "description": "(512) Brewing Company is a microbrewery located in the heart of
            Austin that brews for the community using as many local, domestic and organic
            ingredients as possible.",
            "state": "Texas",
            "type": "brewery",
            "updated": "2010-07-22 20:00:20"
        }
    }, {
        "br": {
            "abv": 6.8,
            "name": "(512) Pecan Porter",
            "upc": 0,
            "description": "Nearly black in color, (512) Pecan Porter is made with Organic
            US 2-row and Crystal malts along with Baird’s Chocolate and Black malts. Its full
            body and malty sweetness are balanced with subtle pecan aroma and flavor from
            locally grown pecans. Yet another true Austin original!",
            "style": "Porter",
            "brewery_id": "512_brewing_company",
            "type": "beer",
            "category": "Irish Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -97.7697,
                "lat": 30.2234
            },
            "country": "United States",
            "website": "http:\/\/512brewing.com\/",
            "code": "78745",
            "address": [
                "407 Radam, F200"
            ],
            "city": "Austin",
            "phone": "512.707.2337",
            "name": "(512) Brewing Company",
            "description": "(512) Brewing Company is a microbrewery located in the heart
            of Austin that brews for the community using as many local, domestic and
            organic ingredients as possible.",
            "state": "Texas",
            "type": "brewery",
            "updated": "2010-07-22 20:00:20"
        }
    }, {
        "br": {
            "abv": 8.2,
            "name": "(512) Whiskey Barrel Aged Double Pecan Porter",
            "upc": 0,
            "description": "Our first barrel project is in kegs and on it’s way to beer bars
            around Austin. This is a bigger, bolder version of our mainstay Pecan Porter,
            with a richer finish. Two months on recently emptied Jack Daniels select barrels
            imparted a wonderful vanilla character from the oak and a pleasant amount of
            whiskey nose and flavor. All in all, I’m really proud of the hard work and effort
            put into this beer. Our first attempt at brewing it and our first attempt at
            managing barrels has paid off for everyone! Seek out this beer, but don’t put it off.
            There is a very limited number of kegs available and it might go fast…",
            "style": "Porter",
            "brewery_id": "512_brewing_company",
            "type": "beer",
            "category": "Irish Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -97.7697,
                "lat": 30.2234
            },
            "country": "United States",
            "website": "http:\/\/512brewing.com\/",
            "code": "78745",
            "address": [
                "407 Radam, F200"
            ],
            "city": "Austin",
            "phone": "512.707.2337",
            "name": "(512) Brewing Company",
            "description": "(512) Brewing Company is a microbrewery located in the heart of
            Austin that brews for the community using as many local, domestic and organic ingredients as possible.",
            "state": "Texas",
            "type": "brewery",
            "updated": "2010-07-22 20:00:20"
        }
    }, {
        "br": {
            "abv": 5.2,
            "name": "(512) Wit",
            "upc": 0,
            "description": "Made in the style of the Belgian wheat beers that are so refreshing,
            (512) Wit is a hazy ale spiced with coriander and domestic grapefruit peel.
            50% US Organic 2-row malted barley and 50% US unmalted wheat and oats make this
            a light, crisp ale well suited for any occasion.",
            "style": "Belgian-Style White",
            "brewery_id": "512_brewing_company",
            "type": "beer",
            "category": "Belgian and French Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -97.7697,
                "lat": 30.2234
            },
            "country": "United States",
            "website": "http:\/\/512brewing.com\/",
            "code": "78745",
            "address": [
                "407 Radam, F200"
            ],
            "city": "Austin",
            "phone": "512.707.2337",
            "name": "(512) Brewing Company",
            "description": "(512) Brewing Company is a microbrewery located in the heart of
            Austin that brews for the community using as many local, domestic and organic
            ingredients as possible.",
            "state": "Texas",
            "type": "brewery",
            "updated": "2010-07-22 20:00:20"
        }
    }, {
        "br": {
            "abv": 8.0,
            "name": "One",
            "upc": 0,
            "description": "Our first anniversary release is a Belgian-style strong ale
            that is amber in color, with a light to medium body. Subtle malt sweetness is
            balanced with noticeable hop flavor, light raisin and mildly spicy, cake-like
            flavors, and is finished with local wildflower honey aromas. Made with 80% Organic
            Malted Barley, Belgian Specialty grains, Forbidden Fruit yeast, domestic hops and
            Round Rock local wildflower honey, this beer is deceptively high in alcohol. ",
            "style": "Belgian-Style Pale Strong Ale",
            "brewery_id": "512_brewing_company",
            "type": "beer",
            "category": "Belgian and French Ale",
            "ibu": 0.0,
            "updated": "2010-07-29 14:11:16",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -97.7697,
                "lat": 30.2234
            },
            "country": "United States",
            "website": "http:\/\/512brewing.com\/",
            "code": "78745",
            "address": [
                "407 Radam, F200"
            ],
            "city": "Austin",
            "phone": "512.707.2337",
            "name": "(512) Brewing Company",
            "description": "(512) Brewing Company is a microbrewery located in the heart
            of Austin that brews for the community using as many local, domestic and organic
            ingredients as possible.",
            "state": "Texas",
            "type": "brewery",
            "updated": "2010-07-22 20:00:20"
        }
    }, {
        "br": {
            "abv": 7.2,
            "name": "21A IPA",
            "upc": 0,
            "description": "Deep golden color. Citrus and piney hop aromas. Assertive malt
            backbone supporting the overwhelming bitterness. Dry hopped in the fermenter
            with four types of hops giving an explosive hop aroma. Many refer to this IPA as
            Nectar of the Gods. Judge for yourself. Now Available in Cans!",
            "style": "American-Style India Pale Ale",
            "brewery_id": "21st_amendment_brewery_cafe",
            "type": "beer",
            "category": "North American Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -122.393,
                "lat": 37.7825
            },
            "country": "United States",
            "website": "http:\/\/www.21st-amendment.com\/",
            "code": "94107",
            "address": [
                "563 Second Street"
            ],
            "city": "San Francisco",
            "phone": "1-415-369-0900",
            "name": "21st Amendment Brewery Cafe",
            "description": "The 21st Amendment Brewery offers a variety of award winning house made brews and American grilled cuisine in a comfortable loft like setting. Join us before and after Giants baseball games in our outdoor beer garden. A great location for functions and parties in our semi-private Brewers Loft. See you soon at the 21A!",
            "state": "California",
            "type": "brewery",
            "updated": "2010-10-24 13:54:07"
        }
    }, {
        "br": {
            "abv": 5.0,
            "name": "563 Stout",
            "upc": 0,
            "description": "Deep black color, toasted black burnt coffee flavors and aroma. Dispensed with Nitrogen through a slow-flow faucet giving it the characteristic cascading effect, resulting in a rich dense creamy head.",
            "style": "American-Style Stout",
            "brewery_id": "21st_amendment_brewery_cafe",
            "type": "beer",
            "category": "North American Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -122.393,
                "lat": 37.7825
            },
            "country": "United States",
            "website": "http:\/\/www.21st-amendment.com\/",
            "code": "94107",
            "address": [
                "563 Second Street"
            ],
            "city": "San Francisco",
            "phone": "1-415-369-0900",
            "name": "21st Amendment Brewery Cafe",
            "description": "The 21st Amendment Brewery offers a variety of award winning house made brews and American grilled cuisine in a comfortable loft like setting. Join us before and after Giants baseball games in our outdoor beer garden. A great location for functions and parties in our semi-private Brewers Loft. See you soon at the 21A!",
            "state": "California",
            "type": "brewery",
            "updated": "2010-10-24 13:54:07"
        }
    }, {
        "br": {
            "abv": 5.2,
            "name": "Amendment Pale Ale",
            "upc": 0,
            "description": "Rich golden hue color. Floral hop with sweet malt aroma. Medium mouth feel with malt sweetness, hop quenching flavor and well-balanced bitterness.",
            "style": "American-Style Pale Ale",
            "brewery_id": "21st_amendment_brewery_cafe",
            "type": "beer",
            "category": "North American Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -122.393,
                "lat": 37.7825
            },
            "country": "United States",
            "website": "http:\/\/www.21st-amendment.com\/",
            "code": "94107",
            "address": [
                "563 Second Street"
            ],
            "city": "San Francisco",
            "phone": "1-415-369-0900",
            "name": "21st Amendment Brewery Cafe",
            "description": "The 21st Amendment Brewery offers a variety of award winning house made brews and American grilled cuisine in a comfortable loft like setting. Join us before and after Giants baseball games in our outdoor beer garden. A great location for functions and parties in our semi-private Brewers Loft. See you soon at the 21A!",
            "state": "California",
            "type": "brewery",
            "updated": "2010-10-24 13:54:07"
        }
    }, {
        "br": {
            "abv": 3.6,
            "name": "Bitter American",
            "upc": 0,
            "description": "An American session beer. Loaded with hop character and a malty presence, but lower in alcohol.",
            "style": "Special Bitter or Best Bitter",
            "brewery_id": "21st_amendment_brewery_cafe",
            "type": "beer",
            "category": "British Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -122.393,
                "lat": 37.7825
            },
            "country": "United States",
            "website": "http:\/\/www.21st-amendment.com\/",
            "code": "94107",
            "address": [
                "563 Second Street"
            ],
            "city": "San Francisco",
            "phone": "1-415-369-0900",
            "name": "21st Amendment Brewery Cafe",
            "description": "The 21st Amendment Brewery offers a variety of award winning house made brews and American grilled cuisine in a comfortable loft like setting. Join us before and after Giants baseball games in our outdoor beer garden. A great location for functions and parties in our semi-private Brewers Loft. See you soon at the 21A!",
            "state": "California",
            "type": "brewery",
            "updated": "2010-10-24 13:54:07"
        }
    }, {
        "br": {
            "abv": 9.8,
            "name": "Double Trouble IPA",
            "upc": 0,
            "description": "Deep, golden, rich malt flavor huge citrus, fruity grassy, ethanol sweetness aroma with a profound bitterness, yet balanced malt back bone with grapefruit, mellow citric overtones. Dry hopped three times in the fermenter. Brewed with over 65 lbs of hops for 300 gallons of beer. The beer to bring world peace and end the war. Bronze Medal - 2006 Imperial IPA Festival at the Bistro in Hayward, California.",
            "style": "Imperial or Double India Pale Ale",
            "brewery_id": "21st_amendment_brewery_cafe",
            "type": "beer",
            "category": "North American Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -122.393,
                "lat": 37.7825
            },
            "country": "United States",
            "website": "http:\/\/www.21st-amendment.com\/",
            "code": "94107",
            "address": [
                "563 Second Street"
            ],
            "city": "San Francisco",
            "phone": "1-415-369-0900",
            "name": "21st Amendment Brewery Cafe",
            "description": "The 21st Amendment Brewery offers a variety of award winning house made brews and American grilled cuisine in a comfortable loft like setting. Join us before and after Giants baseball games in our outdoor beer garden. A great location for functions and parties in our semi-private Brewers Loft. See you soon at the 21A!",
            "state": "California",
            "type": "brewery",
            "updated": "2010-10-24 13:54:07"
        }
    }, {
        "br": {
            "abv": 5.5,
            "name": "General Pippo's Porter",
            "upc": 0,
            "description": "Deep toffee color with rich roasty and subtle hop aroma. Chocolate flavors dominate the palate and interact with back-end sweetness.",
            "style": "Porter",
            "brewery_id": "21st_amendment_brewery_cafe",
            "type": "beer",
            "category": "Irish Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -122.393,
                "lat": 37.7825
            },
            "country": "United States",
            "website": "http:\/\/www.21st-amendment.com\/",
            "code": "94107",
            "address": [
                "563 Second Street"
            ],
            "city": "San Francisco",
            "phone": "1-415-369-0900",
            "name": "21st Amendment Brewery Cafe",
            "description": "The 21st Amendment Brewery offers a variety of award winning house made brews and American grilled cuisine in a comfortable loft like setting. Join us before and after Giants baseball games in our outdoor beer garden. A great location for functions and parties in our semi-private Brewers Loft. See you soon at the 21A!",
            "state": "California",
            "type": "brewery",
            "updated": "2010-10-24 13:54:07"
        }
    }, {
        "br": {
            "abv": 5.8,
            "name": "North Star Red",
            "upc": 0,
            "description": "Deep amber color. Subtle hop floral nose intertwined with sweet crystal malt aromas. Rich malt flavors supporting a slight bitterness finish.",
            "style": "American-Style Amber\/Red Ale",
            "brewery_id": "21st_amendment_brewery_cafe",
            "type": "beer",
            "category": "North American Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -122.393,
                "lat": 37.7825
            },
            "country": "United States",
            "website": "http:\/\/www.21st-amendment.com\/",
            "code": "94107",
            "address": [
                "563 Second Street"
            ],
            "city": "San Francisco",
            "phone": "1-415-369-0900",
            "name": "21st Amendment Brewery Cafe",
            "description": "The 21st Amendment Brewery offers a variety of award winning house made brews and American grilled cuisine in a comfortable loft like setting. Join us before and after Giants baseball games in our outdoor beer garden. A great location for functions and parties in our semi-private Brewers Loft. See you soon at the 21A!",
            "state": "California",
            "type": "brewery",
            "updated": "2010-10-24 13:54:07"
        }
    }, {
        "br": {
            "abv": 5.9,
            "name": "Oyster Point Oyster Stout",
            "upc": 0,
            "description": "Deep black color. Chocolate milk color head, providing an array of Belgian lace. Toffee and light roasty aromas and flavors. A malty sweet taste is evident but, this rich oatmeal based stout finishes dry. Made with 20 lbs. of oysters, in the boil, from our good friends at Hog Island Oyster Company.",
            "style": "American-Style Stout",
            "brewery_id": "21st_amendment_brewery_cafe",
            "type": "beer",
            "category": "North American Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -122.393,
                "lat": 37.7825
            },
            "country": "United States",
            "website": "http:\/\/www.21st-amendment.com\/",
            "code": "94107",
            "address": [
                "563 Second Street"
            ],
            "city": "San Francisco",
            "phone": "1-415-369-0900",
            "name": "21st Amendment Brewery Cafe",
            "description": "The 21st Amendment Brewery offers a variety of award winning house made brews and American grilled cuisine in a comfortable loft like setting. Join us before and after Giants baseball games in our outdoor beer garden. A great location for functions and parties in our semi-private Brewers Loft. See you soon at the 21A!",
            "state": "California",
            "type": "brewery",
            "updated": "2010-10-24 13:54:07"
        }
    }, {
        "br": {
            "abv": 5.2,
            "name": "Potrero ESB",
            "upc": 0,
            "description": "Traditional English E.S.B. made with English malt and hops. Fruity aroma with an imparted tart flavor brought about by replicating the water profile in England at Burton on Trent.",
            "style": "Special Bitter or Best Bitter",
            "brewery_id": "21st_amendment_brewery_cafe",
            "type": "beer",
            "category": "British Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -122.393,
                "lat": 37.7825
            },
            "country": "United States",
            "website": "http:\/\/www.21st-amendment.com\/",
            "code": "94107",
            "address": [
                "563 Second Street"
            ],
            "city": "San Francisco",
            "phone": "1-415-369-0900",
            "name": "21st Amendment Brewery Cafe",
            "description": "The 21st Amendment Brewery offers a variety of award winning house made brews and American grilled cuisine in a comfortable loft like setting. Join us before and after Giants baseball games in our outdoor beer garden. A great location for functions and parties in our semi-private Brewers Loft. See you soon at the 21A!",
            "state": "California",
            "type": "brewery",
            "updated": "2010-10-24 13:54:07"
        }
    }, {
        "br": {
            "abv": 5.0,
            "name": "South Park Blonde",
            "upc": 0,
            "description": "Light golden color. Sweet dry aroma with crisp, clear bitterness. Brewed with imported German hops.The perfect beer to have when you'd like to have more than one.",
            "style": "Golden or Blonde Ale",
            "brewery_id": "21st_amendment_brewery_cafe",
            "type": "beer",
            "category": "North American Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -122.393,
                "lat": 37.7825
            },
            "country": "United States",
            "website": "http:\/\/www.21st-amendment.com\/",
            "code": "94107",
            "address": [
                "563 Second Street"
            ],
            "city": "San Francisco",
            "phone": "1-415-369-0900",
            "name": "21st Amendment Brewery Cafe",
            "description": "The 21st Amendment Brewery offers a variety of award winning house made brews and American grilled cuisine in a comfortable loft like setting. Join us before and after Giants baseball games in our outdoor beer garden. A great location for functions and parties in our semi-private Brewers Loft. See you soon at the 21A!",
            "state": "California",
            "type": "brewery",
            "updated": "2010-10-24 13:54:07"
        }
    }, {
        "br": {
            "abv": 5.5,
            "name": "Watermelon Wheat",
            "upc": 0,
            "description": "The definition of summer in a pint glass. This unique, American-style wheat beer, is brewed with 400 lbs. of fresh pressed watermelon in each batch. Light turbid, straw color, with the taste and essence of fresh watermelon. Finishes dry and clean. Now Available in Cans!",
            "style": "Belgian-Style Fruit Lambic",
            "brewery_id": "21st_amendment_brewery_cafe",
            "type": "beer",
            "category": "Belgian and French Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "ROOFTOP",
                "lon": -122.393,
                "lat": 37.7825
            },
            "country": "United States",
            "website": "http:\/\/www.21st-amendment.com\/",
            "code": "94107",
            "address": [
                "563 Second Street"
            ],
            "city": "San Francisco",
            "phone": "1-415-369-0900",
            "name": "21st Amendment Brewery Cafe",
            "description": "The 21st Amendment Brewery offers a variety of award winning house made brews and American grilled cuisine in a comfortable loft like setting. Join us before and after Giants baseball games in our outdoor beer garden. A great location for functions and parties in our semi-private Brewers Loft. See you soon at the 21A!",
            "state": "California",
            "type": "brewery",
            "updated": "2010-10-24 13:54:07"
        }
    }, {
        "br": {
            "abv": 5.0,
            "name": "Drie Fonteinen Kriek",
            "upc": 0,
            "description": "",
            "style": "Belgian-Style Fruit Lambic",
            "brewery_id": "3_fonteinen_brouwerij_ambachtelijke_geuzestekerij",
            "type": "beer",
            "category": "Belgian and French Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        },
        "bw": {
            "geo": {
                "accuracy": "RANGE_INTERPOLATED",
                "lon": 4.3081,
                "lat": 50.7668
            },
            "country": "Belgium",
            "website": "http:\/\/www.3fonteinen.be\/index.htm",
            "code": "",
            "address": [
                "Hoogstraat 2A"
            ],
            "city": "Beersel",
            "phone": "32-02-\/-306-71-03",
            "name": "3 Fonteinen Brouwerij Ambachtelijke Geuzestekerij",
            "description": "",
            "state": "Vlaams Brabant",
            "type": "brewery",
            "updated": "2010-07-22 20:00:20"
        }
    } ]

Die-hard SQL _JOIN_-_ON_ clause syntax fans will be happy to know that SQL++ hasn't forgotten them,
so a result identical to the one immediately above can also be produced as follows:

    SELECT *
    FROM breweries bw JOIN beers br ON br.brewery_id = meta(bw).id
    ORDER BY bw.name, br.name
    LIMIT 20;

Finally (for now :-)), another more explicit SQL++ way of achieving the very same result as above is shown below:

    SELECT VALUE {"bw": bw, "br": br}
    FROM breweries bw, beers br
    WHERE br.brewery_id = meta(bw).id
    ORDER BY bw.name, br.name
    LIMIT 20;

This version of the query uses an explicit record constructor to build each result record.
(It is worth noting that the string field names "bw" and "br" in the record constructor above are both simple SQL++
expressions themselves---so in the most general case, even the resulting field names can be computed as part of the query,
making SQL++ a very powerful tool for slicing and dicing semistructured data.)

(It is worth knowing, with respect to influencing Analytics's query evaluation,
that _FROM_ and _JOIN_ clauses - also known as joins - are currently evaluated in order,
with the "left" clause probing the data of the "right" clause.)

### Query 4 - Nested Outer Join ###
In order to support joins between tables with missing or dangling join tuples, the designers of SQL ended
up shoe-horning a subset of the relational algebra into SQL's _FROM_ clause syntax, and providing a
variety of join types there for users to choose from (which SQL++ supports for SQL compatibility).
Left outer joins are particularly important in SQL, for example, to print a summary of customers and orders,
grouped by customer, without omitting those customers who haven't placed any orders yet.

The SQL++ language supports nesting, both of queries and of query results,
and the combination allows for a cleaner and more natural approach to such queries.
As an example, suppose we wanted for each brewery to produce a record that contains
the brewery name along with a list of all of the brewery's offered beer names and alcohol percentages.
In the flat (also known as 1NF) world of SQL, approximating this query would involve a left outer join between
breweries and beers, ordered by brewery, with the brewery name being repeated along side each beer's information.
In the richer ("NoSQL") world of SQL++, this use case can be handled more naturally as follows:

    SELECT bw.name AS brewer,
          (SELECT br.name, br.abv FROM beers br
           WHERE br.brewery_id = meta(bw).id) AS beers
    FROM breweries bw
    ORDER BY bw.name
    LIMIT 5;

This SQL++ query binds the variable "bw" to the records in breweries;
for each brewery, it constructs a result record containing a "brewer" field with the brewery's name plus a "beers"
field with a nested collection of records containing the beer name and alcohol percentage for each of the brewery's beers.
The nested collection field for each brewery is created using a correlated subquery.
> **Note:** While it looks like nested loops could be involved in computing the result,
Analytics recognizes the equivalence of such a query to an outerjoin, so it will use an
efficient parallel join strategy when actually computing the query's result.)

Below is this example query's expected output:

    [ {
        "beers": [
            {
                "abv": 8.2,
                "name": "(512) Whiskey Barrel Aged Double Pecan Porter"
            },
            {
                "abv": 5.8,
                "name": "(512) Pale"
            },
            {
                "abv": 5.2,
                "name": "(512) Wit"
            },
            {
                "abv": 8.0,
                "name": "One"
            },
            {
                "abv": 7.6,
                "name": "(512) Bruin"
            },
            {
                "abv": 6.8,
                "name": "(512) Pecan Porter"
            },
            {
                "abv": 6.0,
                "name": "(512) ALT"
            },
            {
                "abv": 7.0,
                "name": "(512) IPA"
            }
        ],
        "brewer": "(512) Brewing Company"
    }, {
        "beers": [
            {
                "abv": 7.2,
                "name": "21A IPA"
            },
            {
                "abv": 5.8,
                "name": "North Star Red"
            },
            {
                "abv": 5.9,
                "name": "Oyster Point Oyster Stout"
            },
            {
                "abv": 9.8,
                "name": "Double Trouble IPA"
            },
            {
                "abv": 5.2,
                "name": "Potrero ESB"
            },
            {
                "abv": 5.2,
                "name": "Amendment Pale Ale"
            },
            {
                "abv": 3.6,
                "name": "Bitter American"
            },
            {
                "abv": 5.5,
                "name": "General Pippo's Porter"
            },
            {
                "abv": 5.0,
                "name": "South Park Blonde"
            },
            {
                "abv": 5.0,
                "name": "563 Stout"
            },
            {
                "abv": 5.5,
                "name": "Watermelon Wheat"
            }
        ],
        "brewer": "21st Amendment Brewery Cafe"
    }, {
        "beers": [
            {
                "abv": 5.0,
                "name": "Drie Fonteinen Kriek"
            },
            {
                "abv": 6.0,
                "name": "Oude Geuze"
            }
        ],
        "brewer": "3 Fonteinen Brouwerij Ambachtelijke Geuzestekerij"
    }, {
        "beers": [

        ],
        "brewer": "357"
    }, {
        "beers": [
            {
                "abv": 5.9,
                "name": "Bock Beer"
            },
            {
                "abv": 0.0,
                "name": "Classic Special Brew"
            },
            {
                "abv": 5.5,
                "name": "Gull Classic"
            },
            {
                "abv": 5.5,
                "name": "Genuine Pilsner"
            },
            {
                "abv": 5.9,
                "name": "Juleøl"
            }
        ],
        "brewer": "Aass Brewery"
    } ]

Notice that since the brewery named "Abbey Wright Brewing/Valley Inn" offers no beers, its list of beers is empty.

### Query 5 - Theta Join ###
Not all joins are expressible as equijoins and computable using equijoin-oriented algorithms.
The join predicates for some use cases involve predicates with functions; Analytics supports the
expression of such queries and will still evaluate them as best it can using nested loop based
techniques (and broadcast joins in the parallel case).

As an example of such a use case, suppose that we wanted, for each Arizona brewery, to get
the brewery's name, location, and a list of competitors' names-where competitors are other
breweries that are geographically close to their location.
In SQL++, this can be accomplished in a manner similar to the previous query, but with locality plus
name inequality instead of a simple key equality condition in the correlated query's _WHERE_ clause:

    SELECT bw1.name AS brewer, bw1.geo AS location,
          (SELECT VALUE bw2.name FROM breweries bw2
           WHERE bw2.name != bw1.name
             AND abs(bw1.geo.lat - bw2.geo.lat) <= 0.1
             AND abs(bw2.geo.lon - bw1.geo.lon) <= 0.1
          ) AS competitors
    FROM breweries bw1
    WHERE bw1.state = 'Arizona'
    ORDER BY bw1.name;

The expected result for this query is as shown below:

    [ {
        "competitors": [
            "Mudshark Brewing"
        ],
        "location": {
            "accuracy": "RANGE_INTERPOLATED",
            "lon": -114.35,
            "lat": 34.4702
        },
        "brewer": "Barley Brothers Brewery and Grill"
    }, {
        "competitors": [
            "Mogollon Brewing Company"
        ],
        "location": {
            "accuracy": "ROOFTOP",
            "lon": -111.648,
            "lat": 35.1973
        },
        "brewer": "Flagstaff Brewing"
    }, {
        "competitors": [
            "Rio Salado Brewing"
        ],
        "location": {
            "accuracy": "ROOFTOP",
            "lon": -111.916,
            "lat": 33.4194
        },
        "brewer": "Four Peaks Brewing"
    }, {
        "competitors": [

        ],
        "location": {
            "accuracy": "APPROXIMATE",
            "lon": -112.074,
            "lat": 33.4484
        },
        "brewer": "Leinenkugel's Ballyard Brewery"
    }, {
        "competitors": [
            "Flagstaff Brewing"
        ],
        "location": {
            "accuracy": "RANGE_INTERPOLATED",
            "lon": -111.59,
            "lat": 35.2156
        },
        "brewer": "Mogollon Brewing Company"
    }, {
        "competitors": [
            "Barley Brothers Brewery and Grill"
        ],
        "location": {
            "accuracy": "ROOFTOP",
            "lon": -114.341,
            "lat": 34.4686
        },
        "brewer": "Mudshark Brewing"
    }, {
        "competitors": [

        ],
        "location": {
            "accuracy": "RANGE_INTERPOLATED",
            "lon": -111.796,
            "lat": 34.8661
        },
        "brewer": "Oak Creek Brewery"
    }, {
        "competitors": [
            "Sonoran Brewing Company"
        ],
        "location": {
            "accuracy": "RANGE_INTERPOLATED",
            "lon": -111.853,
            "lat": 33.7268
        },
        "brewer": "Pinnacle Peak Patio Steakhouse & Microbrewery"
    }, {
        "competitors": [

        ],
        "location": {
            "accuracy": "ROOFTOP",
            "lon": -112.47,
            "lat": 34.542
        },
        "brewer": "Prescott Brewing Company"
    }, {
        "competitors": [
            "Four Peaks Brewing"
        ],
        "location": {
            "accuracy": "APPROXIMATE",
            "lon": -111.909,
            "lat": 33.4148
        },
        "brewer": "Rio Salado Brewing"
    }, {
        "competitors": [
            "Pinnacle Peak Patio Steakhouse & Microbrewery"
        ],
        "location": {
            "accuracy": "RANGE_INTERPOLATED",
            "lon": -111.853,
            "lat": 33.7268
        },
        "brewer": "Sonoran Brewing Company"
    }, {
        "competitors": [

        ],
        "location": {
            "accuracy": "ROOFTOP",
            "lon": -111.015,
            "lat": 32.3407
        },
        "brewer": "Thunder Canyon Brewery"
    } ]

### Query 6 - Existential Quantification ###
The expressive power of SQL++ includes support for queries involving "some" (existentially quantified)
and "all" (universally quantified) query semantics.
Quantified predicates are especially useful for querying datasets involving nested collections of records,
in order to find records where some or all of their nested sets' records satisfy a condition of interest.
To illustrate their use in such situations, we start here by using another (orthogonal) feature of SQL++,
its _WITH_ clause, to create a temporarily nested view of breweries and their beers; we will then use an
existential (_SOME_) predicate to find those breweries whose beers include at least one IPA and return the
brewery's name, phone number, and complete list of beer names and associated alcohol levels.
(This is clearly an important analytical use case, right?!)
Here is the resulting SQL++ query:

    WITH nested_breweries AS
         (
          SELECT bw.name AS brewer, bw.phone,
                (
                 SELECT br.name, br.abv FROM beers br
                 WHERE br.brewery_id = meta(bw).id
                 ORDER BY br.name
                ) AS beers
          FROM breweries bw
         )
    SELECT VALUE nb FROM nested_breweries nb
    WHERE (SOME b IN nb.beers SATISFIES b.name LIKE '%IPA%')
    ORDER BY nb.brewer
    LIMIT 5;

The expected result in this case is as below:

    [ {
        "phone": "512.707.2337",
        "beers": [
            {
                "abv": 6.0,
                "name": "(512) ALT"
            },
            {
                "abv": 7.6,
                "name": "(512) Bruin"
            },
            {
                "abv": 7.0,
                "name": "(512) IPA"
            },
            {
                "abv": 5.8,
                "name": "(512) Pale"
            },
            {
                "abv": 6.8,
                "name": "(512) Pecan Porter"
            },
            {
                "abv": 8.2,
                "name": "(512) Whiskey Barrel Aged Double Pecan Porter"
            },
            {
                "abv": 5.2,
                "name": "(512) Wit"
            },
            {
                "abv": 8.0,
                "name": "One"
            }
        ],
        "brewer": "(512) Brewing Company"
    }, {
        "phone": "1-415-369-0900",
        "beers": [
            {
                "abv": 7.2,
                "name": "21A IPA"
            },
            {
                "abv": 5.0,
                "name": "563 Stout"
            },
            {
                "abv": 5.2,
                "name": "Amendment Pale Ale"
            },
            {
                "abv": 3.6,
                "name": "Bitter American"
            },
            {
                "abv": 9.8,
                "name": "Double Trouble IPA"
            },
            {
                "abv": 5.5,
                "name": "General Pippo's Porter"
            },
            {
                "abv": 5.8,
                "name": "North Star Red"
            },
            {
                "abv": 5.9,
                "name": "Oyster Point Oyster Stout"
            },
            {
                "abv": 5.2,
                "name": "Potrero ESB"
            },
            {
                "abv": 5.0,
                "name": "South Park Blonde"
            },
            {
                "abv": 5.5,
                "name": "Watermelon Wheat"
            }
        ],
        "brewer": "21st Amendment Brewery Cafe"
    }, {
        "phone": "800-737-2311",
        "beers": [
            {
                "abv": 0.0,
                "name": "Abbey Ale"
            },
            {
                "abv": 0.0,
                "name": "Abita Amber"
            },
            {
                "abv": 0.0,
                "name": "Abita Golden"
            },
            {
                "abv": 6.5,
                "name": "Abita Jockamo IPA"
            },
            {
                "abv": 0.0,
                "name": "Abita Light Beer"
            },
            {
                "abv": 4.75,
                "name": "Abita Purple Haze"
            },
            {
                "abv": 8.0,
                "name": "Andygator"
            },
            {
                "abv": 5.5,
                "name": "Christmas Ale"
            },
            {
                "abv": 0.0,
                "name": "Fall Fest"
            },
            {
                "abv": 0.0,
                "name": "Honey Rye Ale"
            },
            {
                "abv": 0.0,
                "name": "Purple Haze"
            },
            {
                "abv": 5.0,
                "name": "Restoration Pale Ale"
            },
            {
                "abv": 0.0,
                "name": "S.O.S"
            },
            {
                "abv": 5.1,
                "name": "Satsuma Harvest Wit"
            },
            {
                "abv": 0.0,
                "name": "Satsuma Wit"
            },
            {
                "abv": 0.0,
                "name": "Strawberry"
            },
            {
                "abv": 0.0,
                "name": "Strawberry Harvest Lager"
            },
            {
                "abv": 0.0,
                "name": "Triple Citra Hopped Satsuma Wit"
            },
            {
                "abv": 0.0,
                "name": "Turbodog"
            },
            {
                "abv": 0.0,
                "name": "Wheat"
            }
        ],
        "brewer": "Abita Brewing Company"
    }, {
        "phone": "1-907-780-5866",
        "beers": [
            {
                "abv": 5.3,
                "name": "Alaskan Amber"
            },
            {
                "abv": 10.4,
                "name": "Alaskan Barley Wine Ale"
            },
            {
                "abv": 5.0,
                "name": "Alaskan ESB"
            },
            {
                "abv": 6.2,
                "name": "Alaskan IPA"
            },
            {
                "abv": 5.2,
                "name": "Alaskan Pale"
            },
            {
                "abv": 6.5,
                "name": "Alaskan Smoked Porter"
            },
            {
                "abv": 5.7,
                "name": "Alaskan Stout"
            },
            {
                "abv": 5.3,
                "name": "Alaskan Summer Ale"
            },
            {
                "abv": 5.3,
                "name": "Alaskan White Ale"
            },
            {
                "abv": 7.0,
                "name": "Breakup Bock"
            },
            {
                "abv": 6.4,
                "name": "Winter Ale"
            }
        ],
        "brewer": "Alaskan Brewing"
    }, {
        "phone": "1-858-549-9888",
        "beers": [
            {
                "abv": 5.5,
                "name": "Anvil Ale"
            },
            {
                "abv": 10.0,
                "name": "Grand Cru 2003"
            },
            {
                "abv": 10.0,
                "name": "Grand Cru 2006"
            },
            {
                "abv": 10.0,
                "name": "Horny Devil"
            },
            {
                "abv": 7.25,
                "name": "IPA"
            },
            {
                "abv": 0.0,
                "name": "Lil Devil"
            },
            {
                "abv": 10.0,
                "name": "Old Numbskull 2003"
            },
            {
                "abv": 12.0,
                "name": "Speedway Stout"
            },
            {
                "abv": 9.5,
                "name": "Wee Heavy"
            },
            {
                "abv": 5.0,
                "name": "X Extra Pale Ale"
            }
        ],
        "brewer": "AleSmith Brewing"
    } ]

### Query 7 - Universal Quantification ###
As an example of a universally quantified SQL++ query, we can find those breweries that only have IPAs:

    WITH nested_breweries AS
         (
           SELECT bw.name AS brewer, bw.phone,
                (
                 SELECT br.name, br.abv FROM beers br
                 WHERE br.brewery_id = meta(bw).id
                ) AS beers
          FROM breweries bw
         )
    SELECT VALUE nb FROM nested_breweries nb
    WHERE (EVERY b IN nb.beers SATISFIES b.name LIKE '%IPA%')
    ORDER BY nb.brewer
    LIMIT 5;

The expected result in this case is shown below:

    [ {
        "phone": "",
        "beers": [

        ],
        "brewer": "357"
    }, {
        "phone": "570.326.3383",
        "beers": [

        ],
        "brewer": "Abbey Wright Brewing\/Valley Inn"
    }, {
        "phone": "6086633926",
        "beers": [

        ],
        "brewer": "Ale Asylum"
    }, {
        "phone": "360-588-1720",
        "beers": [

        ],
        "brewer": "Anacortes Brewing"
    }, {
        "phone": "(207) 763-3305",
        "beers": [

        ],
        "brewer": "Andrew's Brewing"
    } ]

### Query 8 - Simple Aggregation ###
Like SQL, the SQL++ language of Analytics provides support for computing aggregates over large amounts of data.
As a very simple example, the following SQL++ query computes the total number of beers in a SQL-like way:

    SELECT COUNT(*) AS num_beers FROM beers;

This query's result are:

    { "num_beers": 5891 }

If an "unwrapped" value is preferred, the following variant can be used instead:

    SELECT VALUE COUNT(b) FROM beers b;

This time the result is simply:

    5891

In SQL++, aggregate functions can be applied to arbitrary collections, including subquery results.
To illustrate, here is a less SQL-like, and also more explicit, way to express the query above:

    SELECT VALUE ARRAY_COUNT((SELECT b FROM beers b));

For each traditional SQL aggregate function _F_, SQL++ has a corresponding function _ARRAY_F_ that
can be used to perform the desired aggregate calculation.
Each such function is a regular function that takes a collection-valued argument to aggregate over.
Thus, the query above counts the results produced by the beer selection subquery, and the previous,
more SQL-like versions are just syntactic sugar for SQL++ queries that logically use _ARRAY_COUNT_.
> **Note:** Always add subqueries in parentheses ()in SQL++.

### Query 9 (and friends) - Grouping and Aggregation ###
Also like SQL, SQL++ supports grouped aggregation.
For each brewery that offers more than 30 beers,
the following group-by or aggregate query reports the number of beers that it offers.

    SELECT br.brewery_id, COUNT(*) AS num_beers
    FROM beers br
    GROUP BY br.brewery_id
    HAVING num_beers > 30
    ORDER BY num_beers DESC;

The _FROM_ clause incrementally binds the variable _br_ to beers, and the _GROUP BY_ clause groups
the beers by their associated brewery id.
Unlike SQL, where data is tabular-flat-the data model underlying SQL++ allows for nesting.
Thus, due to the _GROUP BY_ clause, the _SELECT_ clause in this query sees a sequence of _br_ groups,
with each such group having an associated _brewery_id_ variable value (that is the producing brewery's id).
In the context of the _SELECT_ clause, _brewery_id_ is bound to the brewery's id and _br_
is now re-bound (due to grouping) to the _set_ of beers issued by that brewery.
The _SELECT_ clause yields a result record containing the brewery's id and the count of the items
in the associated beer set.
The query result will contain one such record per brewery id.

Below is the expected result for this query over the sample data:

    [ {
        "num_beers": 57,
        "brewery_id": "midnight_sun_brewing_co"
    }, {
        "num_beers": 49,
        "brewery_id": "rogue_ales"
    }, {
        "num_beers": 38,
        "brewery_id": "anheuser_busch"
    }, {
        "num_beers": 37,
        "brewery_id": "egan_brewing"
    }, {
        "num_beers": 37,
        "brewery_id": "troegs_brewing"
    }, {
        "num_beers": 36,
        "brewery_id": "boston_beer_company"
    }, {
        "num_beers": 34,
        "brewery_id": "titletown_brewing"
    }, {
        "num_beers": 34,
        "brewery_id": "f_x_matt_brewing"
    }, {
        "num_beers": 33,
        "brewery_id": "sierra_nevada_brewing_co"
    }, {
        "num_beers": 32,
        "brewery_id": "stone_brewing_co"
    }, {
        "num_beers": 31,
        "brewery_id": "southern_tier_brewing_co"
    } ]

Analytics has multiple evaluation strategies available for processing grouped aggregate queries.
For grouped aggregation, the system knows how to employ both sort-based and hash-based aggregation methods,
with sort-based methods being used by default and a hint being available to suggest that a different approach
(hashing) be used in processing a particular SQL++ query.

The following query is nearly identical to the previous one, but adds a hash-based aggregation hint:

    SELECT br.brewery_id, COUNT(*) AS num_beers
    FROM beers br
    /*+ hash */
    GROUP BY br.brewery_id
    HAVING num_beers > 30
    ORDER BY num_beers DESC;

Here is the expected result (the same result, but in a slightly different order):

    [ {
        "num_beers": 57,
        "brewery_id": "midnight_sun_brewing_co"
    }, {
        "num_beers": 49,
        "brewery_id": "rogue_ales"
    }, {
        "num_beers": 38,
        "brewery_id": "anheuser_busch"
    }, {
        "num_beers": 37,
        "brewery_id": "egan_brewing"
    }, {
        "num_beers": 37,
        "brewery_id": "troegs_brewing"
    }, {
        "num_beers": 36,
        "brewery_id": "boston_beer_company"
    }, {
        "num_beers": 34,
        "brewery_id": "titletown_brewing"
    }, {
        "num_beers": 34,
        "brewery_id": "f_x_matt_brewing"
    }, {
        "num_beers": 33,
        "brewery_id": "sierra_nevada_brewing_co"
    }, {
        "num_beers": 32,
        "brewery_id": "stone_brewing_co"
    }, {
        "num_beers": 31,
        "brewery_id": "southern_tier_brewing_co"
    } ]

### Query 10 - Grouping and Limits ###
In some use cases it is not necessary to compute the entire answer to a query.
In some cases, just having the first _N_ or top _N_ result is sufficient.
This is expressible in SQL++ using the _LIMIT_ clause combined with the _ORDER BY_ clause.
(You may have noticed that we have used the _LIMIT_ clause all along to keep the
result set sizes in this document manageable.)

The following SQL++ query returns the top three breweries based on their numbers of offered beers.
It also illustrates the use of multiple aggregate functions to compute various alcohol content statistics
for their beers as well as the use of a join to identify the breweries by name rather than by id:

    SELECT bw.name,
           COUNT(*) AS num_beers,
           AVG(br.abv) AS abv_avg,
           MIN(br.abv) AS abv_min,
           MAX(br.abv) AS abv_max
    FROM breweries bw, beers br
    WHERE br.brewery_id = meta(bw).id
    GROUP BY bw.name
    ORDER BY num_beers DESC
    LIMIT 3;

The expected result for this query is:

    [ {
        "num_beers": 57,
        "name": "Midnight Sun Brewing Co.",
        "abv_max": 16.0,
        "abv_avg": 7.767543859649121,
        "abv_min": 0.0
    }, {
        "num_beers": 49,
        "name": "Rogue Ales",
        "abv_max": 11.5,
        "abv_avg": 4.688775510204081,
        "abv_min": 0.0
    }, {
        "num_beers": 38,
        "name": "Anheuser-Busch",
        "abv_max": 8.0,
        "abv_avg": 3.8052631578947373,
        "abv_min": 0.0
    } ]

### Everything Must Change  ###
So far we have been walking through the SQL++ query capabilities of Analytics.
What really makes Analytics interesting, however, is that it brings this query power to bear on your nearly-current Couchbase Server
data, enabling you to harness the power of parallelism in Analytics to analyze what's going on with your data "up front" in essentially
real time, without perturbing your front-end applications' performance (or the resulting front-end user experience).
Before closing this tutorial, let's take a very quick look at that aspect of Analytics.

To start, the following SQL++ query lists all of the Kona Brewery's current beer offerings:

    SELECT meta(b).id, b as beer FROM beers b
    WHERE b.brewery_id = "kona_brewing"
    ORDER BY meta(b).id;

The result of this query is a list of the following seven beers:

    [ {
        "id": "kona_brewing-black_sand_porter",
        "beer": {
            "abv": 0.0,
            "name": "Black Sand Porter",
            "upc": 0,
            "description": "",
            "style": "Porter",
            "brewery_id": "kona_brewing",
            "type": "beer",
            "category": "Irish Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        }
    }, {
        "id": "kona_brewing-fire_rock_pale_ale",
        "beer": {
            "abv": 5.8,
            "name": "Fire Rock Pale Ale",
            "upc": 0,
            "description": "",
            "style": "American-Style Pale Ale",
            "brewery_id": "kona_brewing",
            "type": "beer",
            "category": "North American Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        }
    }, {
        "id": "kona_brewing-lilikoi_wheat_ale",
        "beer": {
            "abv": 0.0,
            "name": "Lilikoi Wheat Ale",
            "upc": 0,
            "description": "",
            "style": "Light American Wheat Ale or Lager",
            "brewery_id": "kona_brewing",
            "type": "beer",
            "category": "Other Style",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        }
    }, {
        "id": "kona_brewing-longboard_lager",
        "beer": {
            "abv": 5.5,
            "name": "Longboard Lager",
            "upc": 0,
            "description": "",
            "style": "American-Style Lager",
            "brewery_id": "kona_brewing",
            "type": "beer",
            "category": "North American Lager",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        }
    }, {
        "id": "kona_brewing-pipeline_porter",
        "beer": {
            "abv": 0.0,
            "name": "Pipeline Porter",
            "upc": 0,
            "description": "Pipeline Porter is smooth and dark with a distinctive roasty aroma and earthy complexity from its diverse blends of premium malted barley. This celebration of malt unites with freshly roasted 100% Kona coffee grown at Cornwell Estate on Hawaii’s Big Island, lending a unique roasted aroma and flavor. A delicate blend of hops rounds out this palate-pleasing brew.",
            "style": "Porter",
            "brewery_id": "kona_brewing",
            "type": "beer",
            "category": "Irish Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        }
    }, {
        "id": "kona_brewing-stout",
        "beer": {
            "abv": 0.0,
            "name": "Stout",
            "upc": 0,
            "description": "",
            "style": "American-Style Stout",
            "brewery_id": "kona_brewing",
            "type": "beer",
            "category": "North American Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        }
    }, {
        "id": "kona_brewing-wailua",
        "beer": {
            "abv": 0.0,
            "name": "Wailua",
            "upc": 0,
            "description": "Wailua is Hawaiian for two fresh water streams mingling. This was just the inspiration we needed for our Limited Release wheat ale brewed with tropical passion Fruit. A refreshing citrusy, sun-colored ale with the cool taste of Hawaii.",
            "style": "Light American Wheat Ale or Lager",
            "brewery_id": "kona_brewing",
            "type": "beer",
            "category": "Other Style",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        }
    } ]

To illustrate Analytics in action, suppose that Kona's marketing department determines that a light beer is needed.
You can use your favorite Couchbase Server interface to modify the front-end server's beer-sample content accordingly, for example:

    INSERT INTO `beer-sample` ( KEY, VALUE )
      VALUES
      (
        "kona_brewing-skimboard_light_ale",
        {"name": "Skimboard Light Ale", "abv": 4.0, "ibu": 0.0, "srm": 0.0, "upc": 0, "type": "beer", "brewery_id": "kona_brewing", "updated": "2010-07-22 20:00:20", "description": "", "style": "Light Beer", "category": "North American Ale"}
      )
    RETURNING META().id as docid, *;

Analytics will shadow this change, updating the beers shadow dataset as a result.
Go ahead and rerun the Analytics SQL++ query that lists the Kona Brewery's beer offerings:

    SELECT meta(b).id, b as beer FROM beers b
    WHERE b.brewery_id = "kona_brewing"
    ORDER BY meta(b).id;

The result of the query now reflects the new beer offering:

    [ {
        "id": "kona_brewing-black_sand_porter",
        "beer": {
            "abv": 0.0,
            "name": "Black Sand Porter",
            "upc": 0,
            "description": "",
            "style": "Porter",
            "brewery_id": "kona_brewing",
            "type": "beer",
            "category": "Irish Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        }
    }, {
        "id": "kona_brewing-fire_rock_pale_ale",
        "beer": {
            "abv": 5.8,
            "name": "Fire Rock Pale Ale",
            "upc": 0,
            "description": "",
            "style": "American-Style Pale Ale",
            "brewery_id": "kona_brewing",
            "type": "beer",
            "category": "North American Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        }
    }, {
        "id": "kona_brewing-lilikoi_wheat_ale",
        "beer": {
            "abv": 0.0,
            "name": "Lilikoi Wheat Ale",
            "upc": 0,
            "description": "",
            "style": "Light American Wheat Ale or Lager",
            "brewery_id": "kona_brewing",
            "type": "beer",
            "category": "Other Style",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        }
    }, {
        "id": "kona_brewing-longboard_lager",
        "beer": {
            "abv": 5.5,
            "name": "Longboard Lager",
            "upc": 0,
            "description": "",
            "style": "American-Style Lager",
            "brewery_id": "kona_brewing",
            "type": "beer",
            "category": "North American Lager",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        }
    }, {
        "id": "kona_brewing-pipeline_porter",
        "beer": {
            "abv": 0.0,
            "name": "Pipeline Porter",
            "upc": 0,
            "description": "Pipeline Porter is smooth and dark with a distinctive roasty aroma and earthy complexity from its diverse blends of premium malted barley. This celebration of malt unites with freshly roasted 100% Kona coffee grown at Cornwell Estate on Hawaii’s Big Island, lending a unique roasted aroma and flavor. A delicate blend of hops rounds out this palate-pleasing brew.",
            "style": "Porter",
            "brewery_id": "kona_brewing",
            "type": "beer",
            "category": "Irish Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        }
    }, {
        "id": "kona_brewing-skimboard_light_ale",
        "beer": {
            "abv": 4,
            "name": "Skimboard Light Ale",
            "description": "",
            "upc": 0,
            "style": "Light Beer",
            "brewery_id": "kona_brewing",
            "category": "North American Ale",
            "type": "beer",
            "ibu": 0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0
        }
    }, {
        "id": "kona_brewing-stout",
        "beer": {
            "abv": 0.0,
            "name": "Stout",
            "upc": 0,
            "description": "",
            "style": "American-Style Stout",
            "brewery_id": "kona_brewing",
            "type": "beer",
            "category": "North American Ale",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        }
    }, {
        "id": "kona_brewing-wailua",
        "beer": {
            "abv": 0.0,
            "name": "Wailua",
            "upc": 0,
            "description": "Wailua is Hawaiian for two fresh water streams mingling. This was just the inspiration we needed for our Limited Release wheat ale brewed with tropical passion Fruit. A refreshing citrusy, sun-colored ale with the cool taste of Hawaii.",
            "style": "Light American Wheat Ale or Lager",
            "brewery_id": "kona_brewing",
            "type": "beer",
            "category": "Other Style",
            "ibu": 0.0,
            "updated": "2010-07-22 20:00:20",
            "srm": 0.0
        }
    } ]

To further illustrate Analytics in action, suppose that Kona's CEO determines that a light beer is in fact NOT needed.
You can use your favorite N1QL interface again to modify the Couchbase Server's beer-sample content accordingly, that is:

    DELETE FROM `beer-sample` b USE KEYS "kona_brewing-skimboard_light_ale";

Finally, if you run the SQL++ Kona beer list query on Analytics once again, you will find that the Kona CEO's wishes have been
shadowed in Analytics as well.

## Further Help ##
That's it! You are now armed and dangerous with respect to semistructured data management using Analytics via SQL++.
For more information, see
[SQL++ Language Reference](1_intro.html),
and a complete list of the built-in functions is available in the [Function Reference](function-ref.html).

Analytics is a powerful new NoSQL analytics capbability; we hope that you'll find it useful in exploring and analyzing your
Couchbase Server data without having to worry about front-end impact or doing ETL stunts to make your analyses possible.
Use it wisely, and remember: "With great power comes great responsibility..." :-)
Do let us know how you like it...!
