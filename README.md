# DT Toolbox

Data manipulation tool:
  - Converting object to flatten version(DT) and reverse(ST);
  - Converting object from ST to DT format and reverse;
  - Add new object properties without touching the existing ones;
  - Update only existing properties of the object;
  - Compare DT objects;
  - Search and select data;
  - Extract data chunks;
  - Manipulate data stracture according needs;

Deliberately designed to work nicely with JSON and simplify data creation, searching and extracting processes.





## What is DT?

DT format is a flatten version of the standard javascript object. **Keys** are structured like folders - `'root/sub_object/property'`. **Values** are always primitives. In this documentation are mentioned two formats - ST(standard) and DT(data). ST means non-flatten version of the object. Convertion between ST and DT is possible in both direction with one important note: If ST object contains empty structure like `key: []` (*key with empty array value*), then this structure will be lost. **Purpose of DT is to strip all boilerplate structure and keep only that is value related**.





## Installation

Install by writing in your terminal:
```
npm install dt-toolbox --save
```

Once it has been installed, it can be used by writing this line of JavaScript:
```js
let dtbox = require ( 'dt-toolbox')
```




## APIs Reference
Dtbox contains two different apis. First is related to the library itself:

```
API = {
 // * DT I/O Operations
     init       : 'Start chain with data or empty'
   , load       : 'Load DT object or value'
   , preprocess : 'Convert ST to DT object. Change income data before add, update, overwrite'
   , add        : 'Add data and keep existing data'
   , update     : 'Updates only existing data'
   , overwrite  : 'Add new data to DT object. Overwrite existing fields'
   , insert     : 'Insert data on specified key, when the key represents an array'
   , spread     : 'Export DT object'
   , spreadAll  : 'Shortcut for chain: .select().all().spread()'
   , log        : 'Executes callback with errors list as argument'
   , empty      : 'Returns empty DT object'

// Compare Operations
   , identical  :  'Value compare. Reduce data to identical key/value pairs'
   , change     :  'Value compare. Reduce to key/value pairs with different values'
   , same       :  'Key compare. Returns key/value pairs where keys are the same'
   , different  :  'Key compare. Reduce data to key/value pairs that differ'
   , missing    :  'Key compare. Gets from DT key/value pairs that are missing'
       
 // * Selectors and Filters
   , select     : 'Init new selection'
   , parent     : 'Selector. Apply conditions starting from parent level'
   , folder     : 'Selector. Fullfil select with list of arguments that have specific string'
   , all        : 'Selector. Same as folder'
   , space      : 'Selector. Fullfil select with namespace members'
   , limit      : 'Filter.   Reduces amount of records in the selection'
   , keep       : 'Filter.   Keeps records in selection if check function returns true'
   , remove     : 'Filter.   Removes records from selection if check function returns true'
   , deep       : 'Filter.   Arguments ( num, direction - optional). Num mean level of deep. Deep '0' mean root members'
}

```

Second set of functions are available for DT object in a callback of 'spread' and 'preprocess' functions. More details can be found later in the example section.

```

exportAPI = {
  // * Structure Manipulation 
     assemble     : 'Remove all duplications in the keys and shrinks th possible'
   , ignoreKeys   : 'Converts object with nosense keys in array'
   , keyList      : 'Returns array of DT object keys'
   , valueList    : 'Returns array of DT object values'
   , json         : 'Return JSON format of DT object'
   , build        : 'Build ST object'
                
  // * Data Manipulation 
   , map          : 'Standard map function'
   , modifyKeys   : 'Add modified keys back to DT object'
   , keepKeys     : 'Apply test on array of keys. Keep met the criteria'
   , removeKeys   : 'Apply test on array of keys. Remove met the criteria'
   , keepValues   : 'Apply test on values. Keep met the criteria'
   , removeValues : 'Apply test on values. Remove met the criteria'
}

```



# How it works?

1. First: Create data inside DT toolbox. Load already existing DT or init with some ST object. Mix if you need with other objects by using 'add/update/overwrite'. Use 'preprocess' to adapt data package before assimilate it.

2. Select! Without selection, DT Toolbox will always return empty object. Selectors will search in dt.value and result will be accumulated. Filters will be applied to already selected data.

3. Spread the data.

DT Toolbox supports chaining syntax and is that simple. Let's see some examples...










## Examples

### Basics
Let's have a ST object:

```js
let st = {
              name    : { 
                            firstName  : 'Peter' 
                          , surname : 'Naydenov'
                        }
            , friends : [ 'Tisho', 'Dibo', 'Ivo', 'Vasil' ]
         }

```
Same object in DT will look like:

```js
{
    'root/name/firstName' : 'Peter'
  , 'root/name/surname'   : 'Naydenov'
  , 'root/friends/0'      : 'Tisho'
  , 'root/friends/1'      : 'Dibo'
  , 'root/friends/2'      : 'Ivo'
  , 'root/friends/3'      : 'Vasil'
}
```

Let's play with DT Toolbox:

```js
let 
    dtResult
  , stResult
  , friendList
  ;

dtbox
    .init(st) // Init data. Converts ST to DT
    .select() // Starting new selection
    .all()    // Select all data
    .spread ( 'dt', dt => dtResult = dt            ) // returns DT object
    .spread ( 'dt', dt => stResult == dt.build()   ) // convert back to ST
    .select () // Start new selection. Will remove previous selection.
    .folder('friends') // select keys that contain 'friends'
    .spread ( 'dt' => friendList = dt.build() ) //= { friends :[ 'Tisho', 'Dibo', 'Ivo', 'Vasil' ] }
    .spread ( 'dt' => friendList = dt.assemble().build() ) // = [ 'Tisho', 'Dibo', 'Ivo', 'Vasil' ]
    // 'assemble' removes all duplicated elements in the keys and simplifies the result.
    
```

### Mixing objects - Add/Update/Overwrite

Add and update will consider existing data:
- 'Add' method will be applied only for non-existing properties;
- 'Update' method will works only on existing properties;

Overwrite will 'add' and 'update'.

```js

let user = { 
                name : 'Peter'
              , age  : 42
           }

dtbox
   .init ( user )
   .add  ({
                age : 25         // 'add' will ignore this. Age is already defined.
              , gender : 'male'  // Will add this.
          })
   .update ({
               age  : 50         // Will update
               eyes : 'blue'     // Will ignore this.
          })
  .overwrite ({
                age   : 43         // Will update
              , hobby : 'skating'  // Will add
          })

  // DT object (dtbox.value) will look like:
  /*
    {
        'root/name'   : 'Peter'
      , 'root/age'    : 43
      , 'root/gender' : 'male'
      , 'root/hobby'  : 'skating'
    }

  /*

```


### Parent

```js

let result;
let data = {
     'school' : [
                      { name: 'Ivan',   age: 14 }
                    , { name: 'Georgy', age: 15 }
                    , { name: 'Adi',    age: 11 }
                    , { name: 'Kati',   age: 11 }
                ]
    , 'sports' : [
                      { name: 'Iva',    age: 28 }
                    , { name: 'Stoyan', age: 36 }
                ]
    , 'work'   : [
                      { name: 'Hristo',   age: 38 }
                    , { name: 'Lachezar', age: 33 }
                    , { name: 'Veselina', age: 35 }
                    
                ]
    , 'recent' : {
                     'classmates' : [
                                         { name: 'Anton',     age: 42 }
                                       , { name: 'Miroslava', age: 42 }
                                    ]
                    , 'social' : [
                                       { name: 'Iliana', age: 61 }
                                     , { name: 'Tzvetan', age: 19 }
                                 ]
                 }
  }

// Let's create list of all contacts uder 40 years old:

dtbox
   .init ( data )
   .select ()
   .parent ( 'name', person => person.age < 40 )
   .spread ( 'dt' , dt => result = dt.assemble().ignoreKeys(0).build()   )

/*
   result will look like this:

   [ 
      { name: 'Tzvetan', age: 19 },
      { name: 'Ivan', age: 14 },
      { name: 'Iva', age: 28 },
      { name: 'Hristo', age: 38 },
      { name: 'Georgy', age: 15 },
      { name: 'Stoyan', age: 36 },
      { name: 'Lachezar', age: 33 },
      { name: 'Adi', age: 11 },
      { name: 'Veselina', age: 35 },
      { name: 'Kati', age: 11 } 
  ]

*/
```








## More 
Find more examples in `./test` folder. Almost 50 unit tests are on your disposal. Find what is possible start experimenting with the library. 

Let me know what you think by using twitter tag #dttoolbox.










## Tips

- DT format not depends on DT Toolbox. Use toolbox only when language cannot provide better tools;
- Iteration over DT Toolbox could bring performance issues(anti-pattern). Iterate over data first and then use DT Toolbox;
- Working with flat objects (DT) could be relieving experience with extras - performance and readability gain;








## Known bugs
_(Nothing yet)_










## Release History

### 1.1.2 (2017-02-05)

 - [x] Fix: ExportAPI method `map` has `index` argument;



### 1.1.1 (2017-02-04)

 - [x] Fix: Method `empty` now works as it was intended;
 - [x] Method `spreadAll` was added and could be used instead the chain: .select().all().spread()
 - [ ] Error: ExportAPI method `map` has no `index` argument;



### 1.1.0

 - [x] Method `empty` returns empty DT object;
 - [x] Compare method were added: `identical`, `change`, `same`, `different`, `missing`
 - [ ] Error: Method `empty` is actually an object;
 - [ ] Error: ExportAPI method `map` has no `index` argument;



### 1.0.2 (2017-01-14)
 - [x] Bug fix - init with files;



### 1.0.0 (2017-01-14)
 
 - [x] Initial code;
 - [x] Test package;
 - [x] Documentation;





## Credits
'dt-toolbox' was created by Peter Naydenov.





## License
'dt-toolbox' is released under the [MIT License](http://opensource.org/licenses/MIT).




