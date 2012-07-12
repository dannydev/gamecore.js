# gamecore.js
gamecore.js is a framework to help build high-performance (and large) games using javascript.

It is comprised of:

* Classes - an implementation of class.js (static inheritance, super methods and introspection)
* General purpose object pooling - garbage collection bad, automatic super-fast object pooling good!
* Class and object IDs
* LinkedList - a high-performance double linked list
* Device - a device independent map (well, a start at one anyway)

# Why?

We ([Playcraft](http://playcraftlabs.com)) like building games in javascript, but along the way we found some issues:

* Object-oriented - call us old fashioned, but object-orientation is just in our dna now, so we took what
 we felt was the best of the javascript class systems (thanks to Prototype and class.js/Javascript MVC) and
 tweaked a few things.
* Pooling - garbage collection is a pain for a high-performance game, so rather than implementing it sporadically,
we built a way to easily pool any class. Ashley Gullen wrote a [good post](http://www.scirra.com/blog/76/how-to-write-low-garbage-real-time-javascript)
 on this recently.
* Linked Lists - we needed a way of storing game objects in a super-fast way, so we included a high-speed linked list.
* We included some other tools, like a simple performance measurement, Tim Down's awesome hashtable and a device lookup
for some general game shimming.

Lastly, and probably most importantly, we open-sourced all this so that other library developers (creating code for
use in games) can implement things like object pooling easily (and thus making it useful in a gaming context).

# Using

## gamecore.Class
A modified version of class.js to cater to static inheritance and deep object cloning.
Based almost completely on class.js (Javascript MVC -- Justin Meyer, Brian Moschel, Michael Mayer and others)
(http://javascriptmvc.com/contribute.html).
Some portions adapted from Prototype JavaScript framework, version 1.6.0.1 (c) 2005-2007 Sam Stephenson

Easy class creation:
```javascript
  var Fighter = gamecore.Base.extend('Fighter',
  {
      // static (this is inherited as well)
      firingSpeed: 1000
  },
  {
      // instance

      hp: 0,
      lastFireTime: 0,

      init: function(hp)    // instance constructor
      {
          this.hp = hp;
      },

      fire: function()
      {
          this._super(); // super methods!

          // do firing!
      }
  });

 var gunship = new Fighter(100);
```

Introspection:
```javascript
  gamecore.Base.extend('Fighter.Gunship');
  Fighter.Gunship.shortName; // ‘Gunship’
  Fighter.Gunship.fullName;  // ‘Fighter.Gunship’
  Fighter.Gunship.namespace; // ‘Fighter’
```

Setup method will be called prior to any init -- nice if you want to do things without needing the
users to call _super in the init, as well as for normalizing parameters.
```javascript
  setup: function()
  {
     this.objectId = this.Class.totalObjects++;
     this.uniqueId = this.Class.fullName + ':' + this.objectId;
  }
```

## gamecore.Base

A base class providing logging, object counting and unique object id's

Examples:

Unique ID and total objects:
```javascript
var Fighter = gamecore.Base.extend('Fighter', {}, {});
var fighter1 = new Fighter();
var fighter2 = new Fighter();
fighter1.uniqueId;    // -> 'Fighter:0'
fighter2.uniqueId;    // -> 'Fighter:1'
Fighter.totalObjects; // -> 2
```
Logging: (log, info, warn, error, debug)
```javascript
fighter1.warn('oops'); // == console.log('Fighter:0 [WARN] oops');
```

## gamecore.Pooled
Easy (high-performance) object pooling.

A pool of objects for use in situations where you want to minimize object life cycling (and
subsequently garbage collection). It also serves as a very high speed, minimal overhead
collection for small numbers of objects.

This class maintains mutual set of doubly-linked lists in order to differentiate between
objects that are in use and those that are unallocated from the pool. This allows for much
faster cycling of only the in-use objects.

Pools are managed by class type, and will auto-expand as required. You can create a custom initial pool
size by deriving from the Pool class and statically overriding INITIAL_POOL_SIZE.

Keep in mind that objects that are pooled are not constructed; they are "reset" when handed out.
You need to "acquire" one and then reset its state, usually via a static create factory method.

Example:
```javascript
Point = gamecore.Pooled('Point',  // derive from gamecore.Pooled
{
  // Static constructor
  create:function (x, y)   // super will handle allocation from a managed pool of objects
                           // the pool will autoexpand as required
  {
     var n = this._super();
     n.x = x;
     n.y = y;
     return n;
  }
},
{
   x:0, y:0,   // instance

   init: function(x, y)
   {
      this.x = x;
      this.y = y;
   }
}
```
To then access the object from the pool, use create, instead of new. Then release it.
```javascript
var p = Point.create(100, 100);
// ... do something
p.release();
```
## gamecore.PerformanceMeasure

A simple tool for measuring performance in ms and an (experimental) memory usage tracker.
```javascript
var measure = new gamecore.PerformanceMeasure('A test');
// ... do something
console.log(measure.end()); // end returns a string you can easily log
```

## gamecore.LinkedList

A high-speed doubly linked list of objects. Note that for speed reasons (using a dictionary lookup of
cached nodes) there can only be a single instance of an object in the list at the same time. Adding the same
object a second time will result in a silent return from the add method.

In order to keep a track of node links, an object must be able to identify itself with a getUniqueId() function.

To add/remove an item use:
```javascript
list.add(newItem);
list.remove(newItem);
```
You can iterate using the first and nextLinked members, such as:
```javascript
   var next = list.first;
   while (next)
   {
       next.obj.DOSOMETHING();
       next = next.nextLinked;
   }
```
## gamecore.Device
Static class with lots of device information, including:
* pixelRatio - pixel ratio of the display (iPhone4 will return 2, everything else is a 1 generally)
* isiPhone - is an iphone
* isiPhone4 - is an iphone 4
* isiPad - is an ipad
* isAndroid - is an android device
* isTouch - has a touch interface
* isFirefox - is firefox
* isChrome - is chrome
* isOpera - is opera
* isIE - is internet explorer
* ieVersion - which version of explorer is it
* requestAnimFrame - a platform shimmed requestAnimFrame that falls back to setTimeout
* hasMemoryProfiling - determines if you can get access to heap memory

To enable memory profiling on Chrome, use:
--enable-memory-info

To access memory use getUsedHeap() and getTotalHeap().


## gamecore.Hashtable
Tim Down's awesome hashtable.

Example:
```javascript
var map = new gamecore.Hashtable();
map.put('test1', obj);
var obj = map.get('test1');
```

# Contribute

You can contribute to the core code by forking it and make a pull request.

* Testing across a broader range of browsers.
* A plugin system.
* Stats for the pooling system (object counts by type etc).
* Type introspection: RocketTurret.isa(‘EnemyBase’);
* Memory leak protection (or detection) (autorelease?)
* Interface enforcement at runtime (implements x)
* Expand gamecore.Device to cover fullscreen api, mouse lock, audio, etc
* Base math functions (using pooled objects and lots of caching -- we're working on this)

# Bugs

Email us at support@playcraftlabs.com; we really like it if you included things like:

1. A test case
2. Error messages
3. Line numbers of offending code
4. Browser/Device you are testing on

# License

See the included license.txt file.

