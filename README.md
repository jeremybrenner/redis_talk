# Redis


## Why is it important?

* NoSQL, key-value storage cache
* Flexible and fast( caching, different data types, etc.)
* Takes several data types
* Atomic commands - all clients aware of changes
* No dependencies!


## Roadblocks

* Used by many clients in many languages 
* Large spectrum of specific commands to datatypes ( good and bad )
* Not often used as a primary database in Rails
* Spent too much time trying to make chat work :(

## Takeaways!

* Has simple and complex use cases
* Great for caching ( APIs, etc.)
* Persists primarily in memory (FAST) - but can also 'snapshot' to disk
* Flexible, and has simple powerful commands for cron jobs, pub/sub
* Great docs!


## Set it up!

* Download the stable build and install ( this is the way Redis suggests )
* You can also use brew for this, but there are issues with out-of-date versions.

```
$ curl -O http://download.redis.io/redis-stable.tar.gz
$ tar xvzf redis-stable.tar.gz
$ cd redis-stable
$ make
$ make install
// you can also use 'sudo make install' if you get permission errors
$ make test 
// optional and useful, but takes some time


```
* You can then test the connection this with:

```
$ redis-server
// starts the server

$ redis-cli ping
// tests the connection to the server

PONG 

// THAT'S HILARIOUS REDIS! 
// When using the command 'redis-cli', youll drop
// directly into an interactive environment 
// for an instance on port 6379 ( this can be modified )

```
* To shut the database down

```
$ redis-cli shutdown

```


## Data Types


### Key & Value (String to String)

**Keys**

- Binary Safe - can be " " or a JPEG file
- Maximum size of 512 MB

**Values (Strings)**

- Simplest value for a key
- Good for caching simple values 

### LISTS

- "Basically a linked list" - Redis docs
- A collection of string elements sorted according to order of insertion
- Could be used for something like tweets on a  user, or a timeline


### SETS

- An unordered collection of strings
- Does not allow repeating members, will ignore duplicates
- Has sever side commands that allow a fast computations( unions, intersections, differences, etc. )
- Great for creating tracking where you want to exclude duplicates (IP of site visitors)


### SORTED SETS

- Similiar to a set, but each value can hold a 'score' which can be used to sort the elements

### HASHES

- Similiar to an object ( great for reprsenting these)
- Maps string fields to values
- Simple use would be a user with multiple fields: name, age etc.


## Commands


### K/V

**SET**, **GET**, **DEL**, and **SETNX** (SET if key does not exist)

* These allow you to set key/values, and retrieved them.

```
127.0.0.1:6379> SET user:name "Jeremy"
127.0.0.1:6379> GET user:name
"Jeremy"	

127.0.0.1:6379> DEL user:name
(integer) 1
127.0.0.1:6379> GET user:name
(nil)

```

**INCR** and **DECR**

* You can use this to increment a value on a key
* DECR will bring the value down

```
127.0.0.1:6379> SET count 5
OK

127.0.0.1:6379> INCR count
(integer) 6
	
127.0.0.1:6379> INCR count
(integer) 7
	
127.0.0.1:6379> DEL count
(integer) 1 

// this does not mean count's value equals 1 
// this is boolean confirmation return from the 
// DEL command

127.0.0.1:6379> INCR count
(integer) 1

//IMPORTANT:This is an atomic command, which allows mutiple //clients to accurately affect and access a value.
```


**EXPIRE** and **TTL**

* Really cool commands - these will let you set a cron job ( in seconds ) on a key, and its associated value.
* You set a timer on the key, and when it ends, it will be deleted from the database.
* **TTL** will allow you to check this timeout, a return of -2 means the key doesnt exist( it was already destroyed ), -1 means it cannot be removed as currently set.

```
127.0.0.1:6379> EXPIRE user 10
// after 10 seconds this key will be destroyed

127.0.0.1:6379> TTL user
// will return number of seconds left until destruction

```

### LIST

**RPUSH/LPUSH** and  **LRANGE**

* RPUSH will put a value on the end of a list.
* LPUSH will put a value on the end.
* LRANGE returns a subset of a list

```
127.0.0.1:6379> RPUSH thingsihate "Reddit reposts" "Lists" "Irony"
(integer) 3

// you can give it multiple arguments at once!

127.0.0.1:6379> LRANGE thingsihate 0 -1
1) "Reddit reposts"
2) "Lists"
3) "Irony"

// this command takes two arguments, the start and stop index, using
// 0 and -1 will return from beginning to end
```
**RPOP/LPOP**

* A reverse push, these will remove the last value on either end
 
 ```
127.0.0.1:6379> RPOP thingsihate
"Reddit reposts"

127.0.0.1:6379> LRANGE thingsihate 0 -1
1) "Reddit reposts"
2) "Lists"
 
 ```
**LLEN**

* Returns the length of a list

```
127.0.0.1:6379> LLEN friends
(integer) 0

// :'(
// :'(
// :'(

```

### SET

**SADD**, **SREM**, and **SMEMBERS**

* SADD adds a value to the set
* SREM will remove it!
* Remember this is similar to the list data type, but doesnt allow duplicates!
* SMEMBERS <set> will return the set

```
127.0.0.1:6379> SADD catnames "Captain Whiskers" "Chairman Meow" "Eugene"
(integer) 3

127.0.0.1:6379> SREM catnames "Chairman Meow"
(integer) 1

127.0.0.1:6379> SMEMBERS catnames
1) "Captain Whiskers"
2) "Eugene"

```
**SUNION**

* Combines two sets, note: this will take several sets as arguments

```
127.0.0.1:6379> SADD set1 "one" "two" "three"
(integer) 3

127.0.0.1:6379> SADD set2 "four" "five" "six"
(integer) 3

127.0.0.1:6379> SUNION set1 set2
1) "one"
2) "two"
3) "three"
4) "four"
5) "five"
6) "six"

```
**SISMEMBER**

* This will test a value against the set and return 1 for true, 0 for false

```
127.0.0.1:6379> SISMEMBER set1 "two"
(integer)1

```

### SORTED SET

**ZADD** and **ZRANGE**

* ZADD allows you to assign a value to a set, along with a score
* ZRANGE will take index arguments to return a list

```
127.0.0.1:6379> ZADD foods 3 "turkey"
(integer) 1

127.0.0.1:6379> ZADD foods 1 "pizza"
(integer) 1

127.0.0.1:6379> ZADD foods 2 "ramen"
(integer) 1

127.0.0.1:6379> ZRANGE 0 -1
1) "pizza"
2) "ramen"
3) "turkey"

```

### HASHES

**HSET**, **HMSET**, **HGET** and **HGETALL**

* HSET creates a HASH data type, that maps strings and values, while HMSET just lets you pass multiple fields and values at once.
* HGET will return a single value of a specific field, HGETALL will return the data at a given key/value pair

```
127.0.0.1:6379> HMSET user:007 firstname "James" lastname "Bond" sost "Shaken"
OK

// or you could do

127.0.0.1:6379> HSET user:007 firstname "James"
(integer1)

127.0.0.1:6379> HSET user:007 lastname "Bond"
(integer1)

127.0.0.1:6379> HSET user:007 sost "Shaken"
(integer1)

127.0.0.1:6379> HGETALL user:007
1) "firstname"
2) "James"
3) "lastname"
4) "Bond"
5) "sost"
6) "Shaken"

// or if you just want one field

127.0.0.1:6379> HGET user:007 firstname
"James"


```








##Some helpful links:


* [Setup and basic use](http://redis.io/topics/quickstart)

* [Questions about Redis](http://redis.io/topics/faq)

* [Brief summary of data types](http://redis.io/topics/data-types)

 
