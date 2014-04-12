BytefactoryORM – http://bytefactory.ch/xojo/bytefactoryorm/
==============

**Today I decided to release my own ORM to the public. The source code is provided under the MIT License.** **BytefactoryORM is a set of classes that allows you to interact with your databases in an object-oriented manner. No need to write SQL statements anymore! It's comparable to Bob Keeney's ActiveRecord port for Xojo - though not intended to be any competition.**

### Status quo

Please note that right now, it's not even close to be finished. The code is
still highly uncommented (I know it by heart though - working with it every
day ;-)). I intend to change that. There's no documentation yet. Some code is
reeealy old and needs to be revised anyway. For a beginning, please refer to
the quick start below and the code itself (though uncommented). Furthermore,
it currently only supports SQLite. Support for other databases can be added
very easily by subclassing the DatabaseConnector class. The error handling is
yet to be improved, too. Since I open source this piece of code, I highly
encourage you not only to use it but also to contribute back. The community
will thank you.

### How it works - quick start

BytefactoryORM does not use introspection but rather queries fields through
Operator_lookup. The pro's: You don't need to create any classes and/or add
properties. The con's: no auto-complete when it comes to field and table
names. It's just a decision I had to take and I went that route. If you would
rather like to have auto-completion, I encourage you to take a look at Bob
Keeney's ActiveRecord port for Xojo. Changes to values are saved instantly -
no call to a save method or such. It makes heavy use of variants and thus is
weak-typed. By design – so please don't beat me ;-)

#### Conventions

Every table has to have an _id_ field of a numeric type that is unique. This
servers as the internal reference. Fields may **only** contain alphanumeric
characters. **Avoid underscores (_). **And no spaces, please. Underscores are
used for foreign keys to model relationships. Only. Example: You have two
tables called _albums_ and _tracks_. The table _tracks__ _then would have a
field called _albums_id_. Many-to-many relations are possible, too. Suppose
you want to map students and classes. A student can attend many classes, while
a class can have many students. You'd setup three tables: _Students, Classes
_and _Classes_Students_ (the order doesn't matter). The table
_Classes_Students_ would have three fields: _id, Classes_id, Students_id._

#### Class SqliteDatabaseConnector

This class needs to be initiated as soon as you want to connect to a database.
It's _constructor_ takes three parameters: the database file (folderitem), an
encryption key (string) and a boolean which determines whether to use WAL
mode. The _connect_ methods tries to connect and returns true in case of
success, otherwise false. The ORM supports both _lazy loading_ and _caching_
of data. Lazy loading turned on will always query the value anew (thus giving
you always the most current value, think of multi-user environments), caching
will save already queried values (speed gains) in the respective instance of
the _record_ class (see below). Lazy loading is turned on by default and can
be changed through the Connector's property _LazyLoading_ (boolean). Many of
the other methods / properties are for internal use. More documentation to
follow ;-)

#### Class Table

An instance of this object refers to a table. You can get a reference to a
table in your database through the SqliteDatabaseConnector class by calling
_myInstanceOfTheDatabaseConnectorClass.[tableName]_, where as _[tableName]_ is
the actual name of the table. Naturally, it won't auto-complete. **Don't try
to initiate an instance yourself!** The _table_ class has a couple of useful
methods: _AddRecord_ will return an empty record. As soon as you add any field
contents it will get added to the table. If you don't, it won't add anything
to the table. The _DeleteAll_ method will, well, delete all records. It
optionally takes a set of conditions (see further below) to limit deleting.
The _cascading_ option is only partly implemented yet. So avoid it right now
:-) The method _GetAllRecords_ will return all records and takes a couple of
parameters (overloaded, so take a look at it!). On one hand, you can pass an
_orderBy statement_ (string), on the other hand there's a (param)array of
_ConditionSets_ (see below). Type of return: _Record_. You can also get a
record by its _id._ To do so, use the method _GetRecordById_ that takes an
integer. There are a couple of references to "virtual tables". Ignore them for
now :-) They are intended to provide direct access to cross-table queries
(joins and so on). Thus "virtual". I haven't implemented them yet.

#### Class Record

An instance of this class represents one record or row in a table. Don't
create instances yourself, get them by the _Table_ object's methods described
above. You can access any values as follows:
_myInstanceOfARecord.[fieldName],_ whereas _[fieldName]_ is the name of your
field. Naturally, it won't auto-complete. You'll get a variant. Thus in most
of the cases you can just work with it like with any variables. In some cases
(like overloaded methods, e.g.) you might want to specifically state the type
(e.g. by adding _.StringValue, _refer to Xojo's Variant documentation).
_Record.Delete_ will delete the record in the table. _ID_ is a method that
returns the _id_ of the record.

#### Class ConditionSet &amp; Class Condition

Finally a class you may initiate yourself ;-) A _ConditionSet_ is, well, a set
of conditions you can pass to a _table's_ _GetAllRecords_ and _DeleteAll_
methods. All conditions applied to one and the same _ConditionSet_ will get
executed with _AND-logic._ If you pass _multiple_ _ConditionSets_ to the
respective methods, those will be executed with _OR-logic._ The _ConditionSet
_doesn't do much itself. It just gives you the chance to add conditions. You
do so by calling: _myOwnInstanceOfConditionSet.[fieldName], _while
_[fieldName]_ is the name of your field and naturally won't auto-complete (it
will return an instance of type _Condition _but you won't need to use that).
Then let if follow by one of the conditions defined:

  * .IsBigger [Value]
  * .IsBiggerOrEqual [Value]
  * .IsEqual [Value]
  * .IsFalse
  * .IsIn (param)array  [Value]
  * .IsLike  [Value] //not exact matches, like 'contains'
  * .IsNotEqual  [Value]
  * .IsNotIn (param)array  [Value]
  * .IsNotLike  [Value]
  * .IsNotNull
  * .IsNull
  * .IsSmaller  [Value]
  * .IsSmallerOrEqual  [Value]
  * .IsTrue
Unfortunately, those won't auto-complete either. But that's the IDE's flaw
this time ;-) Examples:
_myOwnInstanceOfConditionSet.[fieldName].IsBiggerOrEqual 100;
_myOwnInstanceOfConditionSet.[fieldName].isLike "hello world"__
&nbsp;_place_holder;

#### Last words

I think it's enough to get started. Take a look at the demo supplied with it.
As I said, it's far from being finished - but it already works pretty well.
I'd really appreciate any feedback - and contributions of course. 
