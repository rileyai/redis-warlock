warlock
=======

Battle-hardened distributed locking using redis. 

IORedis version of [node-redis-warlock](https://github.com/TheDeveloper/warlock)

## Requirements

* [ioredis](https://github.com/luin/ioredis)
* Redis `v2.6.12` or above. If you're running a Redis version from `v2.6.0` to `v2.6.11` inclusive use `v0.0.7` of this module.

## Install

    npm install node-redis-warlock

## Usage

```javascript

var Warlock = require('redis-warlock');
var Redis = require('ioredis');

// Establish a redis client and pass it to warlock
var redis = new Redis();
var warlock = Warlock(redis);

// Set a lock
var key = 'test-lock';
var ttl = 10000; // Lifetime of the lock

warlock.lock(key, ttl, function(err, unlock){
  if (err) {
    // Something went wrong and we weren't able to set a lock
    return;
  }

  if (typeof unlock === 'function') {
    // If the lock is set successfully by this process, an unlock function is passed to our callback.
    // Do the work that required lock protection, and then unlock() when finished...
    //
    // do stuff...
    //
    unlock();
  } else {
    // Otherwise, the lock was not established by us so we must decide what to do
    // Perhaps wait a bit & retry...
  }
});

// set a lock optimistically
var key = 'opt-lock';
var ttl = 10000;
var maxAttempts = 4; // Max number of times to try setting the lock before erroring
var wait = 1000; // Time to wait before another attempt if lock already in place
warlock.optimistic(key, ttl, maxAttempts, wait, function(err, unlock) {});

// unlock using the lock id
var key = 'test-lock-2';
var ttl = 10000;
var lockId;

warlock.lock(key, ttl, function(err, _, id) {
  lockId = id;
});

// each client who knows the lockId can release the lock
warlock.unlock(key, lockId, function(err, result) {
  if(result == 1) {
    // unlocked successfully
  }
});

```

## ProTips

* Read my [Distributed locks using Redis](https://engineering.gosquared.com/distributed-locks-using-redis) article and Redis' author's [A proposal for more reliable locks using Redis](http://antirez.com/news/77) to learn more about the theory of distributed locks using Redis.
