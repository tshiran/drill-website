# Run Drill in Embedded Mode

Install Drill and start the shell (sqlline):

```
$ tar xf apache-drill-0.7.0.tar.gz
$ cd apache-drill-0.7.0
$ bin/sqlline -u jdbc:drill:zk=local
```

Access Drill's Web UI to check that it's running: http://localhost:8047

Run a query:

```
SELECT * FROM dfs.root.`/Users/tshiran/Development/demo/data/yelp/user.json` LIMIT 1;
```
+---------------+------------+--------------+------------+------------+
| yelping_since |   votes    | review_count |    name    |  user_id   |
+---------------+------------+--------------+------------+------------+
| 2012-02       | {"funny":1,"useful":5,"cool":0} | 6            | Lee        | qtrmBGNqCvupHMHL_bKFgQ |

Notes:

* drillbit (Drill's daemon) starts automatically (on the local node) when you run sqlline (embedded mode)
* sqlline uses Drill's JDBC driver to connect to the drillbit (ZooKeeper is not needed in embedded mode, hence `zk=local`)
* You must use Distributed Mode if you want to use any other client except the Drill shell (sqlline)

# Or Run Drill in Distributed Mode (Single Node Example)

```
$ zkServer start
$ bin/drillbit.sh start
$ bin/sqlline -u jdbc:drill:zk=localhost:2181
```

Access Drill's Web UI to check that it's running: http://localhost:8047

Notes:

* Drill uses ZooKeeper for coordination and discovery; Make sure the ZooKeeper nodes are listed in conf/drill-override.conf
* Not sure if ZooKeeper is running? Run `telnet localhost 2181` and make sure it connects
* Clients (like sqlline) connect to ZooKeeper to discover the nodes in the cluster
* If you have more than one Drill cluster registered with a single ZooKeeper ensemble, specify the Drill cluster name in the connection string: `jdbc:drill:zk=localhost:2181/drill/<clustername>`

# Cluster Layout and Data Locality

* Drill maximizes data locality in order to minimize network transfers
* Best practice:
** HDFS/MapR-FS: Run a drillbit on each DataNode
** HBase/MapR-DB: Run a drillbit on each RegionServer
** MongoDB: Run a drillbit on each mongod (preferrably on the replica)

# Data Sources

* Drill supports numerous data sources via 'storage plugins'
* Smart connections - plugin can rewrite optimizer rules to push down operators

Let's set up two things before we get started:

* Enable a storage plugin in the Web UI for each data source you want to use

  TODO: Show screenshot of enabling MongoDB storage plugin
* Define workspaces for directories in which you'll want to analyze data or create views (this is just for convenience, so we won't have to specify full paths)

  TODO: Show creation of mydata workspace

# Our Real-World Data

We're going to use two datasets:

* Yelp Academic Dataset (JSON)
* US names-to-gender mapping (CSV)

```
$ pwd
/Users/tshiran/Development/demo/data
tshiran-laptop:data tshiran$ find .
.
./names.csv
./yelp
./yelp/business.json
./yelp/checkin.json
./yelp/review.json
./yelp/tip.json
./yelp/user.json
$ head -n 1 yelp/user.json
{"yelping_since": "2012-02", "votes": {"funny": 1, "useful": 5, "cool": 0}, "review_count": 6, "name": "Lee", "user_id": "qtrmBGNqCvupHMHL_bKFgQ", "friends": [], "fans": 0, "average_stars": 3.8300000000000001, "type": "user", "compliments": {}, "elite": []}
```

# Our Real-World Data (Cont.)

Let's also import one of the datasets into MongoDB:

```
$ mongoimport --host localhost --db yelp --collection users < user.json
$ mongo
MongoDB shell version: 2.6.5
> show databases;
admin  (empty)
local  0.078GB
yelp   0.453GB
> use yelp
> db.users.findOne()
{
	"_id" : ObjectId("54566cdf3237149de181a92a"),
	"yelping_since" : "2012-02",
	"votes" : {
		"funny" : 1,
		"useful" : 5,
		"cool" : 0
	},
	"review_count" : 6,
	"name" : "Lee",
	"user_id" : "qtrmBGNqCvupHMHL_bKFgQ",
	"friends" : [ ],
	"fans" : 0,
	"average_stars" : 3.83,
	"type" : "user",
	"compliments" : {
		
	},
	"elite" : [ ]
}
```

# SELECT

```
> SELECT * FROM dfs.root.`/Users/tshiran/Development/demo/data/yelp/review.json` WHERE stars = 1 LIMIT 1;
+------------+------------+------------+------------+------------+------------+------------+-------------+
|   votes    |  user_id   | review_id  |   stars    |    date    |    text    |    type    | business_id |
+------------+------------+------------+------------+------------+------------+------------+-------------+
| {"funny":0,"useful":0,"cool":0} | Qrs3EICADUKNFoUq2iHStA | _ePLBPrkrf4bhyiKWEn4Qg | 1          | 2013-04-19 | I don't know what Dr. Goldberg was like before  moving to Arizona, but let me tell you, STAY AWAY from this doctor and this office. I was going to Dr. Johnson before he left and Goldberg took over when Johnson left. He is not a caring doctor. He is only interested in the co-pay and having you come in for medication refills every month. He will not give refills and could less about patients's financial situations. Trying to get your 90 days mail away pharmacy prescriptions through this guy is a joke. And to make matters even worse, his office staff is incompetent. 90% of the time when you call the office, they'll put you through to a voice mail, that NO ONE ever answers or returns your call. Both my adult children and husband have decided to leave this practice after experiencing such frustration. The entire office has an attitude like they are doing you a favor. Give me a break! Stay away from this doc and the practice. You deserve better and they will not be there when you really need them. I have never felt compelled to write a bad review about anyone until I met this pathetic excuse for a doctor who is all about the money. | review     | vcNAWiLM4dR7D2nwwJ7nCA |
+------------+------------+------------+------------+------------+------------+------------+-------------+
```

# Resources

* Drill tutorial
* Drill Yelp tutorial
* Drill on MongoDB tutorial
* Email drill-user with any issue - remember that Drill is pre-1.0, so a bit rough on the edges



Issues:
* Drill Web UI requires Internet access

* Error messages are horrible.
  Example for missing table:

0: jdbc:drill:zk=localhost:2181> SELECT * FROM dfs.root.`/Users/tshiran/Development/demo/data/yelp/user.jso`;
Query failed: Failure while running sql.

Error: exception while executing query: Failure while executing query. (state=,code=0)
0: jdbc:drill:zk=localhost:2181> 

* Can't use ~ to specify home directory in table path

========

0: jdbc:drill:zk=localhost:2181> SELECT name, hours.Friday FROM dfs.root.`Users/tshiran/Development/demo/data/yelp/business.json` b LIMIT 1;
Query failed: Failure while running sql.

Error: exception while executing query: Failure while executing query. (state=,code=0)
0: jdbc:drill:zk=localhost:2181> SELECT name, b.hours.Friday FROM dfs.root.`Users/tshiran/Development/demo/data/yelp/business.json` b LIMIT 1;
+------------+------------+
|    name    |   EXPR$1   |
+------------+------------+
| Eric Goldberg, MD | {"close":"17:00","open":"08:00"} |
+------------+------------+

====

0: jdbc:drill:zk=localhost:2181> SELECT name FROM dfs.root.`Users/tshiran/Development/demo/data/yelp/business.json` WHERE REPEATED_CONTAINS(categories, 'Mediterranean') LIMIT 1;
Query failed: Failure while running sql.

Error: exception while executing query: Failure while executing query. (state=,code=0)
0: jdbc:drill:zk=localhost:2181> SELECT name FROM dfs.root.`Users/tshiran/Development/demo/data/yelp/business.json` WHERE true and REPEATED_CONTAINS(categories, 'Mediterranean') LIMIT 1;
+------------+
|    name    |
+------------+
| People's Bakery |
+------------+
1 row selected (0.15 seconds)


0: jdbc:drill:zk=localhost:2181> SELECT name FROM dfs.root.`Users/tshiran/Development/demo/data/yelp/business.json` WHERE 1 and REPEATED_CONTAINS(categories, 'Mediterranean') LIMIT 1;
Query failed: Failure while running sql.

Error: exception while executing query: Failure while executing query. (state=,code=0)

======


0: jdbc:drill:zk=localhost:2181> SELECT * FROM dfs.root.`Users/tshiran/Development/demo/data/yelp/business.json` WHERE true and REPEATED_CONTAINS(categories, 'Australian');
+-------------+--------------+------------+------------+------------+------------+--------------+------------+------------+------------+------------+------------+------------+------------+---------------+
| business_id | full_address |   hours    |    open    | categories |    city    | review_count |    name    | longitude  |   state    |   stars    |  latitude  | attributes |    type    | neighborhoods |
+-------------+--------------+------------+------------+------------+------------+--------------+------------+------------+------------+------------+------------+------------+------------+---------------+
Query failed: Query stopeed., You tried to start when you are using a ValueWriter of type NullableBitWriterImpl. [ 7cced8cd-1124-4875-986f-503669c94435 on 172.19.128.211:31010 ]


java.lang.RuntimeException: java.sql.SQLException: Failure while executing query.
	at sqlline.SqlLine$IncrementalRows.hasNext(SqlLine.java:2514)
	at sqlline.SqlLine$TableOutputFormat.print(SqlLine.java:2148)
	at sqlline.SqlLine.print(SqlLine.java:1809)
	at sqlline.SqlLine$Commands.execute(SqlLine.java:3766)
	at sqlline.SqlLine$Commands.sql(SqlLine.java:3663)
	at sqlline.SqlLine.dispatch(SqlLine.java:889)
	at sqlline.SqlLine.begin(SqlLine.java:763)
	at sqlline.SqlLine.start(SqlLine.java:498)
	at sqlline.SqlLine.main(SqlLine.java:460)
	
	Schema-free JSON document model 
	
	=====
	
	0: jdbc:drill:zk=localhost:2181> SELECT name, categories FROM dfs.root.`Users/tshiran/Development/demo/data/yelp/business.json` WHERE true and REPEATED_CONTAINS(categories, 'Australian');
	+------------+------------+
	|    name    | categories |
	+------------+------------+
	| The Australian AZ | ["Bars","Burgers","Nightlife","Australian","Sports Bars","Restaurants"] |
	+------------+------------+
	1 row selected (3.068 seconds)
	0: jdbc:drill:zk=localhost:2181> SELECT business_id, name, categories FROM dfs.root.`Users/tshiran/Development/demo/data/yelp/business.json` WHERE true and REPEATED_CONTAINS(categories, 'Australian');
	Query failed: Failure while running sql.

	Error: exception while executing query: Failure while executing query. (state=,code=0)
	0: jdbc:drill:zk=localhost:2181> 
	
====


0: jdbc:drill:zk=localhost:2181> SELECT * FROM dfs.root.`Users/tshiran/Development/demo/data/yelp/business.json` LIMIT 1;
+-------------+--------------+------------+------------+------------+------------+--------------+------------+------------+------------+------------+------------+------------+------------+---------------+
| business_id | full_address |   hours    |    open    | categories |    city    | review_count |    name    | longitude  |   state    |   stars    |  latitude  | attributes |    type    | neighborhoods |
+-------------+--------------+------------+------------+------------+------------+--------------+------------+------------+------------+------------+------------+------------+------------+---------------+
| vcNAWiLM4dR7D2nwwJ7nCA | 4840 E Indian School Rd
Ste 101
Phoenix, AZ 85018 | {"Tuesday":{"close":"17:00","open":"08:00"},"Friday":{"close":"17:00","open":"08:00"},"Monday":{"close":"17:00","open":"08:00"},"Wednesday":{"close":"17:00","open":"08:00"},"Thursday":{"close":"17:00","open":"08:00"},"Sunday":{},"Saturday":{}} | true       | ["Doctors","Health & Medical"] | Phoenix    | 7            | Eric Goldberg, MD | -111.983758 | AZ         | 3.5        | 33.499313  | {"By Appointment Only":true,"Good For":{},"Ambience":{},"Parking":{},"Music":{},"Hair Types Specialized In":{},"Payment Types":{},"Dietary Restrictions":{}} | business   | []            |
+-------------+--------------+------------+------------+------------+------------+--------------+------------+------------+------------+------------+------------+------------+------------+---------------+
1 row selected (0.204 seconds)
0: jdbc:drill:zk=localhost:2181> SELECT attributes FROM dfs.root.`Users/tshiran/Development/demo/data/yelp/business.json` LIMIT 1;
Query failed: Failure while running fragment.[ 4e82c23c-0181-4164-a1ae-f2d3f9be24d9 on 172.19.128.211:31010 ]


Error: exception while executing query: Failure while executing query. (state=,code=0)
0: jdbc:drill:zk=localhost:2181> 

	
===

What happened to all the error messages?

=====

Query hangs in sqlline after restarting drillbit (sqlline should reconnect gracefully)


===

Question: When running Drill on both Mongo and Hadoop, where should the drillbits be? On Hadoop only? Spread across both?

Do we store data in the distributed cache? Or just let the storage plugin do the caching? Do we still use Hazelcast?

Cluster of commodity servers
Daemon (drillbit) on each node
No dependency on other execution engines (MapReduce, Spark or Tez)
Better performance and manageability
Additional components:
ZooKeeper maintains ephemeral cluster membership information
Hazelcast (embedded in drillbit) maintains metadata information (eg, queue depth, cached query plans)


---

What does this do:

alter system set `store.json.all_text_mode` = true;
