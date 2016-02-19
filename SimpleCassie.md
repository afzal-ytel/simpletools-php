Apache Cassandra PHP Client

## Introduction ##

SimpleCassie is entirely stand-alone package which wrap itself around Thrift libs (licensed to Apache Software Foundation) already built into this class.

SimpleCassie supports all features of [TimeUUIDType](#UUID_Usage_Examples.md).

The latest version is **compatible only with Cassandra 7.0.x** and higher.

Please report all bugs under: http://code.google.com/p/simpletools-php/issues/list so will be able to help and comment on them easier.

**User Notice: as from SimpleCassie 0.7.1.4 defaults to i32 timestamps due to compatibility between 32 and 64 bits systems when using native thrift protocol. However for backward-compatibility with previous versions which default to i64 SimpleCassie can be forced to defaults to i64 long timestamps by calling ->useI64Timestamps(); method**

**User Notice: as from SimpleCassie 0.7.1.3 path is being reset after issuing command so you need to remember to always specify full path ->cf()->key()->.... with every command. The only exception of that is keyspace as this is being set globally per latest Cassandra API so all command are sharing lastly set keyspace**

## Usage ##
```
  <?php

  require_once('SimpleCassie.php');

  $cassie = new SimpleCassie(HOST, PORT, TIMEOUT_MS);


  //failover support added since SimpleCassie 0.7.1.2 by:
  $cassie->addNode(HOST2, PORT2, TIMEOUT_MS);
  $cassie->addNode(HOST3, PORT3, TIMEOUT_MS);

  if(!$cassie->isConnected())
    throw new Exception('Couldn\'t connect to server');

  //checking active node - since SimpleCassie 0.7.1.2
  $activeNode = $cassie->getActiveNode();

  /*
  * setting working keyspace
  * @return - (false) on failure true on success
  * working keyspace can be change at any point by running ->keyspace() method
  */
   $cassie->keyspace('Keyspace1');


  /*
  * setting new column (and key if not exist)
  * @return - (false) on failure
  */
   $cassie->keyspace('MyApp')->cf('Users')->key('user1')->column('name')->set('Marcin');
   $cassie->cf('Users')->key('user1')->column('surname')->set('Rosinski');
   
  /*
  * setting in batches
  * @return - number of batches or false on failure
  */
   $cassie->cf('Users')->key('user1')->column('name')->batch('Marcin');
   $cassie->cf('Users')->key('user1')->column('surname')->batch('Rosinski');

   //batching deletion
   $cassie->cf('Users')->key('user1')->column('columntodrop')->batch();

   //commiting all above batches
   $cassie->batchCommit();

  /*
  * delete column or row/key
  * @return - (false) on failure
  */

  //deleting column
  $cassie->cf('Users')->key('user1')->column('name')->remove();

  //deleting row
  $cassie->cf('Users')->key('user1')->remove();

   /*
  * count number of columns in row/key
  * @return - (int) on succes, false on failure
  */
  $count = $cassie->cf('Users')->key('user2')->count();

  /*
  * count number of columns with predicate: from column to column
  * @return - (int) on succes, false on failure
  */
  $count = $cassie->cf('Users')->key('user2')->column('fromColumn','toColumn')->count();

/*
  * count number of columns with predicate: from column to column for multiple keys
  * @return - (int) on succes, false on failure
  */
  $count = $cassie->cf('Users')->key('user2','user3')->column('fromColumn','toColumn')->count();

  /*
  * getting single column
  * @return - object on succes,  null on failure
  */
  $name = $cassie->keyspace('Keyspace1')->cf('Standard1')->key('user1')->column('name')->get();

  /*
  * getting multiple columns
  * @return - array of objects on success, null on failure
  */
  $user = $cassie->cf('Standard1')->key('user1')->column('name','surname')->get();

/*
  * getting multiple column values
  * @return - array of objects on success, null on failure
  */
  $user = $cassie->cf('Standard1')->key('user1')->column('name','surname')->value();

  /*
  * getting single column from multiple rows/keys
  * @return - array of objects on succes,  null on failure
  */
  $users = $cassie->cf('Standard1')->key('user1','user2')->column('name')->get();

  /*
  * getting multiple columns from multiple rows/keys
  * @return - array of objects on succes,  null on failure
  */
  $users = $cassie->cf('Standard1')->key('user1','user2')->column('name','username')->get();

  /*
  * getting slice of columns from single row/key
  * @return - array of objects on succes,  null on failure
  */
  $limit = 10;
  $reversed = false;
  $from_name = 'Puma';
  $to_name = 'Tiger';
  $friends = $cassie->cf('Standard1')->key('user1friends')->column($from_name,$to_name)->slice($limit,$reversed);

  /*
  * getting slice of columns from single supercolumn row
  * @return - array of objects on succes,  null on failure
  */
  $limit = 10;
  $reversed = false;
  $friends = $cassie->cf('Standard1')->key('user1')->supercolumn('friends')->column($from_name,$to_name)->slice($limit,$reversed);

  //resetting supercolumn for future use - deprecated since SimpleCassie 0.7.1.3 - this process has been automated
  $cassie->key('user1')->supercolumn(null);

  /*
  * getting slice of columns from multiple rows/keys
  * @return - array of objects on succes,  null on failure
  */
  $limit = 10;
  $reversed = true;
  $friends = $cassie->cf('Standard1')->key('user1friends','user2friends')->column($from_name,$to_name)->slice($limit,$reversed);

  /*
  * increment column value
  * @return - (int) new value on succes, false on failure
  */
  $new_value = $cassie->cf('Standard1')->key('user1')->column('friends')->increment();

  /*
  * decrement column value
  * @return - (int) new value on succes, false on failure
  */
  $new_value = $cassie->cf('Standard1')->key('user1')->column('friends')->decrement();


  /*
  * Getting active keyspace
  */
  echo $cassie->keyspace();

  /*
   * Range support - since SimpleCassie 0.7.1.2
   */
  $range = $cassie->cf('MyColumnFamily')->key('fromKey','toKey')->column('fromColumn','toColumn')->range($keyCount,$columnCount);

  ?>
```

## UUID Usage Examples ##
```
  <?php 

  //setting new uuid column
  $cassie->keyspace('MyApp')->cf('BlogPosts')->key('post')->column($cassie->uuid())->set('I like raw food.');

  //getting latest added post (assuming TimeUUIDType)
  $post = $cassie->cf('BlogPosts')->key('post')->slice(1);

  //getting post column uuid name in canonical form
  $uuid = $cassie->uuid($post->column->name);

  $string_form = (string) $uuid; //canonical form
  $binary_form = $uuid->uuid;

  ?>
```

## Using the Native PHP Extension ##

The PHP Native extension is giving thrift real performance boost so its highly recommend to install it.

Please apply patch if getting timeouts, more info here: https://issues.apache.org/jira/browse/THRIFT-867 - packed patch can be downloaded from http://simpletools-php.googlecode.com/files/thrift_protocol_patch.zip


### Installation on Centos ###
```
sudo yum install php-devel

cd PATH-TO-THRIFT/lib/php/src/ext/thrift_protocol
phpize
./configure --enable-thrift_protocol
make

sudo cp modules/thrift_protocol.so /usr/lib64/php/modules/

# enable module: /etc/php.d/thrift_protocol.ini
extension=thrift_protocol.so
```


## Development Roadmap ##
  * Connection Load Balancing
  * ->remove() method for multiple rows/keys - possible in batch() method, look above
  * ->increment() method for multiple rows/keys
  * ->decrement() method for multiple rows/keys
  * all describe methods

## Download Link ##
SimpleCassie.php download - http://simpletools-php.googlecode.com/files/SimpleCassie.zip

## Discussion Group ##
[SimpleCassie related discussion](http://groups.google.com/group/simpletools/browse_thread/thread/b2d5afc83e7acd5b)

## Contributes ##
<a href='http://www.33concept.com/'>33Concept Ltd</a><br>
<a href='http://www.workdigital.co.uk/'>WorkDigital Ltd</a><br>
<a href='http://www.shopperhive.co.uk/'>Shopperhive Ltd</a>