# Cassandra DB

## [Installation](https://www.datastax.com/blog/2012/01/getting-started-apache-cassandra-windows-easy-way)

## Cassandra Cluster Manager

If we use Cassandra Cluster Manager (CCM), we can use the following commands:

To create a new cluster on local machine: `ccm create easybuy -v 3.7 -n 5 -s --pwd-auth`. Where `-v` is version, `-n` is number of nodes, `-s` starts the cassandra cluster, and `--pwd-auth` to set authenticator to password authenticator.

*Default credentials are "cassandra/ cassandra".*

* `ccm list` lists all the clusters on local machine.

	`*` denotes current cluster

* `ccm status` to display status of nodes in current cluster.

* `ccm stop` stops all nodes in the cluster.

* `ccm node1 show` dislplays all information for a specific cluster.

* `ccm remove` to delete the cluster permanently. 

## Columns and Column Families

Data in Cassandra in a columnar layout, which is equivalent to a nested map of the following structure: `Map<RowKey, SortedMap<ColKey, ColValue>>`

Data for a column is stored as a column-value pair (key and value) in the database.

**Column Family** is a collection of one or more columns, each family has a primary key, which can also be a composite key.

Column Family + row key = Rows

Multiple rows form a keyspace.

### Column Family Attributes

1. `keys_cached` - number of keys to be cached
1. `rows_cahced` - number of rows to be cached 
1. `preload_row_cached`

**Super column family:** A logical grouping of columns which belong together. We cannot query individual columns inside a super column, which makes use of super columns undesirable and usually is not used. 

A column consists of 3 parts:

1. Column name
1. Column value
1. Timestamp

### Data Types

1. Simple

	* int
	* float
	* double
	* boolean
	* text

1. Advanced

	* blob
	* counter: A special column for keeping count of an event. Used instead of int to atomically increment the counter, but it must have a dedicated column family.
	* uuid
	* timestamp
	
1. Collection

Collections cannot be nested. Can associate TTL with individual elements in a collection.

	* set: A collection of one or more distinct elements.
	* list: A collection of one or more ordered elements.
	* map: data of key- value pairs.
	
	
	

### Big picture

Cluster > Keyspcae > Column family > Columns

## Cassandra Query Language (CQL)

* very similar to SQL
* case-insensitive by default, unless we use douple quotes `"`
* internally, keyspace, cloumn family, and column names are all stored in lowercase.

To interact with Cassandra we can use `cqlsh`

* `cqlsh -u cassandra -p cassandra` to **login**.

* `create role appUser with password = 'password' and LOGIN = true and superuser = true;` to create a new role (user).

* `list roles;` to list all roles.

* `drop role dummy;` to drop a role.

* `CREATE KEYSPACE catalog WITH replication={'class':'SimpleStrategy','replication_factor':'3'};` to **create a new keyspace** with replication enabled, where 'SimpleStrategy' is used for one datacenter, 'NetworkTopologyStrategy' otherwise. 

* `describe keyspace;` to **list all keyspaces**.

* `use catalog` to **select keyspace**

* To **create column family**:

		CREATE COLUMNFAMILY product(
			productId varchar,
			title text,
			length int,
			PRIMARY KEY(productId)
		);
		
* `describe columnfamilies;` to **list all column families**.

* `describe product` to **get configs** for a column family, such as storage properties, key, etc.

* `ALTER COLUMNFAMILY product ADD modelId text;` to **add a column to a column family**.

* `insert into product(productId, brand, modelId) values('MOB1', 'Nokia', '200');` to **add data** to a row to a column family.

* `insert into product(productId, brand, modelId) values('MOB1', 'Nokia', '200') USING TIMESTAMP 1468500000;` to **insert a timestamp** with row data.

* `insert into product(productId, brand, modelId) values('MOB1', 'Nokia', '200') USING TTL 86400;` to **insert time to live property** in seconds, which in that case indicates that we want to delete our data after one day. 

* `select * from product where productId = 'POST1';` to **query** data. cql supports `=`  operations on primary keys.

* `select * from product where productId IN ('POST1', 'MOB1');` to **query** data. cql supports `IN` operations on primary keys.

* `ALTER COLUMNFAMILY product ADD keyfeatures set<text>;` to add a **set**.

* `insert into product(productId, title, brand, keyfeatures) values ('COM1', 'Acer One', 'Acer', {'detachable keyboard','multitouch display'});` to add data to a **set**.

* `ALTER COLUMNFAMILY product ADD service_type list<text>;` to add a **list**.

* `insert into product(productId, title, brand, keyfeatures) values ('COM1', 'Acer One', 'Acer', ['Service 1','Service 2']);` to add data to the **list**.

* `ALTER COLUMNFAMILY product ADD camera map<text, text>;` to add a **map**.

* `insert into product(productId, title, brand, keyfeatures) values ('COM1', 'Acer One', 'Acer', {'front':'VGA','back':'5mpx'});` to add data to the **map**.

* `create columnfamily productViewCount(productId text primary key, viewcount COUNTER);` to dedicate a new column family to a **counter**.

* `update productViewCount set viewcount = viewcount + 1 where productId = 'COM1';` to increment the **counter**. If no data exists at 'COM1', a new row will be created. 

* `UPDATE product set length = 25, modelId = 's6' where productId = 'MOB1';`, to **update data**. Condition must be on primary key - all columns on the primary key must be included in the `where` clause.

* `create index indexname on columnfamily(column);` to create a secondary index.

	The `IN` operator is not supported for secondary indexes.

### List Operations

* `update product set <columnName>=<columnName>+[new value] where productId = 'MOB1'` to **add** an element to an existing **list**.

* `update product set <columnName>=<columnName>-[exact value] where productId = 'MOB1'` to **remove** an element from an existing **list**. Exact value is also case sensitive.

* `update product set <columnName>[index] = 'new value' where productId = 'MOB1'` to **replace** an element at a particular index in an existing **list**.

### Set Operations

* `update product set <columnName> = <columnName> + {new value} where productId = 'MOB1'` to **add** an element to an existing **set**.
		
	This item will be added to the beginning of the set, not the end.
	
* `update product set <columnName> = <columnName> - {exact value} where productId = 'MOB1'` to **remove** an element from an existing **set**. Exact value is also case sensitive.

### Map Operations

* `update product set <columnName> = <columnName> + {key:value} where productId = 'MOB1'` to **add** an element to an existing **map**.

* `delete <columnName>['key'] from <columnfamily> where productId = 'MOB1'` to **remove** an element from an existing **map**. Exact value is also case sensitive.

* `update product set <columnName>['key'] = 'new value' where productId = 'MOB1'` to **replace** an element at a particular index in an existing **map**.

## Primary keys

Primary key is a column, or a set of columns, which can uniquely identify rows in a column family.

# Integration with Java

## Dependency

	<dependency>
		<groupId>com.datastax.cassandra</groupId>
		<artifactId>cassandra-driver-core</artifactId>
		<version>3.0.2</version>
	</dependency>
