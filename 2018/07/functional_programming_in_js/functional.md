## Functional Programming in JavaScript 
### inspired by [Learn RxJS](http://reactivex.io/learnrx/)


We start with iteration over an array (collection)

```js
var fruits = ['apple', 'banana', 'cherry', 'dates', 'elderberry', 'fig', 'grapes', 'honeydew melon', 'indian fig','jackfruit'];

for(var i = 0, l = fruits.length; i < l; i++) {
  console.log(fruits[i]);
}

```
In the above code snippet, we are iterating over the collection one item at a time using what is called an `indexer`. We have the data at our disposal and then we are making an attempt to program **how** we want to traverse that data. Note the emphasis on *how*. This pattern of programming where we describe how a program should carry itself is called the *`imperative programming paradigm`*.  

We had our data and we enlisted a way to operate on that data, descibing how that operation should work. It is here we can have define any way we want to operate on that data, see below: 

```js

var fruits = ['apple', 'banana', 'cherry', 'dates', 'elderberry', 'fig', 'grapes', 'honeydew melon', 'indian fig','jackfruit'];

// the usual ascending way
for(var i = 0, l = fruits.length; i < l; i++) {
  console.log(fruits[i]);
}

// ======

// the unusual descending way
// using for loop
for(var i = fruits.length - 1; i >= 0; i--) {
  console.log(fruits[i]);
}
// using while loop
var i = fruits.length;
while(--i) {
  console.log(fruits[i]);
}

// ======

// iterate over and print every third element 
for(var i = 0, l = fruits.length; i < l; i++) {
  if((i % 3 != 0))
    continue;
  
  console.log(fruits[i]);
}

for(var i = 0, l = fruits.length; i < l; i+=3) {
  console.log(fruits[i]);
}

// or any other fancy way we want

```
What we want from a functional standpoint is a way wherein the grunt work of iterative over the elements should be handled by the platform, and we define what happens with each item in the collection. It should be something like 

```js
// assuming the variables are defined

collection.loopOver(takeEveryThirdElement);

/*
  Here, 
  collection            == The collection we want to loop over, fruits from above example
  loopOver              == A function defined on the collection that encapsulates looping over the collection items
  takeEveryThirdElement == A function that encapsulates the criteria for performing action on items of the collection
*/

```
This paradigm of programming, where we define our intent descriptively, rather than focusing on how, we focus on what, is *`declrative programming paradigm`*, more so like `SQL` if you will.

In this pursuit, our first member is the looper. It is a function available on the collection that provides iteration over the collection. It expects the work done on each item of the collection to be passed in so that it can carry out that task. 

<h4>I will be overwriting the `Array.prototype` to demonstrate the concept. Their behaviour may be different than the actual implementation. DO NOT DO THIS, NEVER.</h4>

```js
Array.prototype.loopOver = function(itemConsumer) {
  /*
    itemConsumer(value, index, collection)
    itemConsumer is the worker that is called for each item of the collection. 
    The currentItem, currentIndex, and the collection is passed in order to the worker
  */
  var currentIndex = -1, 
      collectionSize = this.length, 
      currentItem;

  while(++currentIndex < collectionSize) {
    currentItem = this[currentIndex]; 
    itemConsumer(currentItem, currentIndex, this); 
  }
};

var numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
var printEverySecondNumber = function(value, position, collection) {
  if(position % 2 == 0) {
    console.log(`Collection[${position}] = ${value}`);
  }
};

numbers.loopOver(printEverySecondNumber);

```
The word `loopOver` conveys the intent clearly. It is also called `each`, `forEach` to convey it will loop over each (or forEach) item of the collection and invoke the worker. The return value from the worker is not collected.

```js
Array.prototype.each 
  = Array.prototype.forEach 
  = function(itemConsumer) {
  /*
    itemConsumer(value, index, collection)
    itemConsumer is the worker that is called for each item of the collection. 
    The currentItem, currentIndex, and the collection is passed in order to the worker
  */
  var currentIndex = -1, 
      collectionSize = this.length, 
      currentItem;

  while(++currentIndex < collectionSize) {
    currentItem = this[currentIndex]; 
    itemConsumer(currentItem, currentIndex, this); 
  }
};

var numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
var printEverySecondNumber = function(value, position, collection) {
  if(position % 2 == 0) {
    console.log(`Collection[${position}] = ${value}`);
  }
};

numbers.forEach(printEverySecondNumber);
```

Our code `numbers.forEach(printEverySecondNumber)` now reads more descriptive and shows us what we want to do. We have still defined `how` we want to process each item, but not how the collection should be iterated over. The iterator is defined by the platform for us. This is a big leap over how we write code. We have our data collection, the platform gives us a way to access items from that collection and we just define what we want to do with that item. 

If we try to classify the type of operations we do on collections, we will find we mostly do `queries` and `projections`. Queries can be operations like ` 'find', 'filter', 'where' `, and Projections will be `'select', 'transform', 'map'` etc. We have other common operations too, which in tandem with queries and projections work up the magic - notable mention would be `'reduce', 'zip', concatAll` etc.

Its time to ask the platform for more!

Our first candidate is the `transformer`. It takes a `tranform` function which transforms members of the collection. The `tansformer` returns the transformed collection. It builds on top of the `forEach` primitive we developed before.

```js
Array.prototype.transformer 
  = Array.prototype.map 
  = Array.prototype.select
  = Array.prototype.collect
  = function(transform) {
      var _transformedCollection = [],
          _transformedValue;
      this.forEach(function(value, index, collection) {
        _transformedValue = transform(value, index, collection);
        _transformedCollection.push(_transformedValue);
      });
      return _transformedCollection
  };

function square(n) {
  return n ** 2;
}

function cube(n) {
  return n ** 3;
}

function quadruple(n) {
  return n ** 4;  
}

var input = [1, 2, 3, 4, 5],
    output;

output = input.transformer(square);
console.log(output);

output = input.map(cube);
console.log(output);

output = input.select(quadruple);
console.log(output);


// we could chain our operations like so
input.map(cube).forEach(console.log);

```

Our next candidate is `filter`. Filter takes a predicate and runs this predicate for each item of the collection. If the item passes the predicate, i.e. returns a truthy value, it is collected. Finally the collection of all items passing the predicate is returned.

```js
Array.prototype.filter 
  = Array.prototype.where
  = function(predicate) {
      var _collectionOfTruthyValues = [];
      this.forEach(function(candidate, index, collection) {
        if(predicate(candidate, index, collection)) {
          _collectionOfTruthyValues.push(candidate);
        }
      });
      return _collectionOfTruthyValues;
  };

function isEven(candidate) {
  return candidate % 2 == 0;
}

var input = [1, 2, 3, 4, 5, 6, 7 ,8, 9, 10];

input.filter(isEven).each(console.log);

```
We so far have seen the query and the projection operators and other primitives. Lets build other useful primitives. The first one will be the reducer.

Reducer, implemented as `reduce`, is a transformer that consumes the collection and returns a singleton collection (a collection with single values). The value returned is the reduced value for the collection. As in cases above, the `how` of the reducer is defined by the user, and the platform to work upon is provided by the runtime. 

The reducer can optionally take an initial value to start the reduction.

One notable mention about `reduce` is that it consumes the collection from `left-to-right`. We will see another reducer, `reduceRight`, that will consume the collection `right-to-left`.

```js

Array.prototype.reduce 
  = Array.prototype.fold
  = Array.prototype.foldL
  = function(reducer, initialValue) {
      var _reducedValue,
          _loopIndex;
      if(arguments.length === 0) {
        return this;
      } else if(arguments.length === 1) {
        _reducedValue = this[0];
        _loopIndex = 1;
      } else if(arguments.length === 2) {
        _reducedValue = initialValue;
        _loopIndex = 0;
      } else {
        throw new Error('invalid arguments');
      }
      
      while(_loopIndex < this.length) {
        _reducedValue = reducer(_reducedValue, this[_loopIndex], _loopIndex, this);
        _loopIndex++;  
      }
      return [_reducedValue];
  };

function addUp(first, second) {
  return first + second;
}

var input = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

input.reduce().each(console.log);
input.reduce(addUp).each(console.log);
input.reduce(addUp, 100).each(console.log);

```

Here is the `right-to-left` reducer.

```js
Array.prototype.reduceRight 
  = Array.prototype.foldR
  = function(reducer, initialValue) {
      var _reducedValue,
          _loopIndex,
          len = this.length;
      if(arguments.length === 0) {
        return this;
      } else if(arguments.length === 1) {
        _reducedValue = this[len - 1];
        _loopIndex = len - 2;
      } else if(arguments.length === 2) {
        _reducedValue = initialValue;
        _loopIndex = len - 1;
      } else {
        throw new Error('invalid arguments');
      }
      
      while(_loopIndex >= 0) {
        _reducedValue = reducer(_reducedValue, this[_loopIndex], _loopIndex, this);
        _loopIndex--;  
      }
      return [_reducedValue];
  };

function addUp(first, second) {
  return first + second;
}

function addUpSwapped(first, second) {
  return second + first ;
}


var input = ['The sum of first ten natural numbers = ', 1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

input.reduceRight().each(console.log);
input.foldR(addUp).each(console.log);
input.reduceRight(addUpSwapped).each(console.log);

```
We have so far seen many utilities that perform low level tasks, like, looping over, transforming each member, filtering, reducing, projecting among others. Building on these utilities, we will now see others now.

Many a times we have a collection of collections and we want to flatten it, i.e. reduce a 2D collection to 1D. 

```js
// Given   [ ['a', 'b'], [1,2], ['p','q'], ['x','y','z'] ]

// Expect  [ 'a', 'b', 1, 2, 'p', 'q', 'x', 'y', 'z']

```
We will build ourselves a primitive, namely, `flatten or flat or concatAll` that will do just the thing. 

```js

Array.prototype.flatten 
  = Array.prototype.flat
  = Array.prototype.concatAll 
  = function() {
      var _flatCollection = [];
      this.forEach(function(childCollection) {
        childCollection.forEach(function(item) {
          _flatCollection.push(item);
        });
        // the same thing can be achived by below line
        // _flatCollection.push.apply(_flatCollection, childCollection);
      });
      return _flatCollection;
  };

var input = [ ['a', 'b'], [1,2], ['p','q'], ['x','y','z'] ];

console.log(input.flat()); // [ 'a', 'b', 1, 2, 'p', 'q', 'x', 'y', 'z']

```
The flat goes hand in hand with map. We have a collection of collections. We map over child collections and when done, we flat that to a one dimensional collection.

To make this point clear, consider this example.

```js

var collection = [
  {
    id: 'childCollection 1',
    collection: [
      'child collection item 1'
      , 'child collection item 2'
      , 'child collection item 3'
      , 'child collection item 4'
    ]
  },
  {
    id: 'childCollection 2',
    collection: [
      'child collection item 1'
      , 'child collection item 2'
      , 'child collection item 3'
      , 'child collection item 4'
    ]
  },
  {
    id: 'childCollection 3',
    collection: [
      'child collection item 1'
      , 'child collection item 2'
    ]
  }
];

collection.map(function(c1Item) {
  return c1Item.collection.map(function(c2Item) {
    return c2Item.replace(/(\d+)$/g, function(match) {
        var collectionId = (/(\d+)$/g).exec(c1Item.id)[1];
        return `${collectionId}.${match}`;
    });
  });
}).flat();


/*

["child collection item 1.1", "child collection item 1.2", "child collection item 1.3", "child collection item 1.4", "child collection item 2.1", "child collection item 2.2", "child collection item 2.3", "child collection item 2.4", "child collection item 3.1", "child collection item 3.2"]

*/

```

The map and flat operators often go together and hence our new operator - `flatMap` aka `concatMap`. The flatMap primitive provides mapping over a collection and flattens the resulting collection.

`flatMap` used on above collection would be like 

```js
var collection = [
  {
    id: 'childCollection 1',
    collection: [
      'child collection item 1'
      , 'child collection item 2'
      , 'child collection item 3'
      , 'child collection item 4'
    ]
  },
  {
    id: 'childCollection 2',
    collection: [
      'child collection item 1'
      , 'child collection item 2'
      , 'child collection item 3'
      , 'child collection item 4'
    ]
  },
  {
    id: 'childCollection 3',
    collection: [
      'child collection item 1'
      , 'child collection item 2'
    ]
  }
];

/*
collection.map(function(c1Item) {
  return c1Item.collection.map(function(c2Item) {
    return c2Item.replace(/(\d+)$/g, function(match) {
        var collectionId = (/(\d+)$/g).exec(c1Item.id)[1];
        return `${collectionId}.${match}`;
    });
  });
}).flat();
*/

collection.flatMap(function(c1Item) {                            // map changed to flatMap here
  return c1Item.collection.map(function(c2Item) {
    return c2Item.replace(/(\d+)$/g, function(match) {
        var collectionId = (/(\d+)$/g).exec(c1Item.id)[1];
        return `${collectionId}.${match}`;
    });
  });                                                           // call to .flat() removed
});

```
Implementing flatMap

```js
Array.prototype.flatMap 
  = Array.prototype.concatMap 
  = function(mapper) {
      return this.map(function(item) {
        return mapper(item);
      }).flat();
  };
```

Another useful primitive while working with data is the ability to combine data from different collections and output a single collection. Commonly referred to as `zip`. `Zip` takes collections and returns a new collection containing values that result from combining individial values taken from the collections. Combining will stop if we have exhausted any of the collection.

```js
// zip(...collections, combiner);
Array.prototype.zip = function(...args) {
  var len = args.length,
      combiner,
      zipCollections,
      zipLen,
      zipCounter = -1,
      zipArgs,
      result = [];


  if(len == 0) {
    return this;
  } else if(len == 1) {
    throw new Error('Invalid arguments. zip(...collections, combiner)');
  } else if(len >= 2 && typeof (combiner = args.pop()) == 'function') {
    zipCollections = [this];
    zipCollections.push.apply(zipCollections, args);
    zipLen = zipCollections
    .map(function(c) {
      return c.length;
    })
    .reduce(function(min, next) {
      return next < min ? next : min;
    });

    while(++zipCounter < zipLen) {
      zipArgs = zipCollections.map(function(collection) {
        return collection[zipCounter];
      });
      result.push(combiner.apply(null, zipArgs));
    }
  }
  return result;
};

function combiner(index, letterSmall, letterCapital) {
  return `${index} : ${letterCapital} ( ${letterSmall} )`;
}

var input = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    lettersSmall = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'],
    lettersCapital = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J'];


input.zip(lettersSmall, lettersCapital, combiner).each(i => console.log(i));

```

Phew! Interesting, terse and brain draining.










Below are the following primitives that we have developed 

```js

Array.prototype.each 
  = Array.prototype.forEach 
  = function(itemConsumer) {
      /*
        itemConsumer(value, index, collection)
        itemConsumer is the worker that is called for each item of the collection. 
        The currentItem, currentIndex, and the collection is passed in order to the worker
      */
      var currentIndex = -1, 
          collectionSize = this.length, 
          currentItem;

      while(++currentIndex < collectionSize) {
        currentItem = this[currentIndex]; 
        itemConsumer(currentItem, currentIndex, this); 
      }
  };

Array.prototype.transformer 
  = Array.prototype.map 
  = Array.prototype.select
  = Array.prototype.collect
  = function(transform) {
      var _transformedCollection = [],
          _transformedValue;
      this.forEach(function(value, index, collection) {
        _transformedValue = transform(value, index, collection);
        _transformedCollection.push(_transformedValue);
      });
      return _transformedCollection
  };

Array.prototype.filter 
  = Array.prototype.where
  = function(predicate) {
      var _collectionOfTruthyValues = [];
      this.forEach(function(candidate, index, collection) {
        if(predicate(candidate, index, collection)) {
          _collectionOfTruthyValues.push(candidate);
        }
      });
      return _collectionOfTruthyValues;
  };

Array.prototype.reduce 
  = Array.prototype.fold
  = Array.prototype.foldL
  = function(reducer, initialValue) {
      var _reducedValue,
          _loopIndex;
      if(arguments.length === 0) {
        return this;
      } else if(arguments.length === 1) {
        _reducedValue = this[0];
        _loopIndex = 1;
      } else if(arguments.length === 2) {
        _reducedValue = initialValue;
        _loopIndex = 0;
      } else {
        throw new Error('invalid arguments');
      }
      
      while(_loopIndex < this.length) {
        _reducedValue = reducer(_reducedValue, this[_loopIndex], _loopIndex, this);
        _loopIndex++;  
      }
      return [_reducedValue];
  };

Array.prototype.reduceRight 
  = Array.prototype.foldR
  = function(reducer, initialValue) {
      var _reducedValue,
          _loopIndex,
          len = this.length;
      if(arguments.length === 0) {
        return this;
      } else if(arguments.length === 1) {
        _reducedValue = this[len - 1];
        _loopIndex = len - 2;
      } else if(arguments.length === 2) {
        _reducedValue = initialValue;
        _loopIndex = len - 1;
      } else {
        throw new Error('invalid arguments');
      }
      
      while(_loopIndex >= 0) {
        _reducedValue = reducer(_reducedValue, this[_loopIndex], _loopIndex, this);
        _loopIndex--;  
      }
      return [_reducedValue];
  };

Array.prototype.flatten 
  = Array.prototype.flat
  = Array.prototype.concatAll 
  = function() {
      var _flatCollection = [];
      this.forEach(function(childCollection) {
        childCollection.forEach(function(item) {
          _flatCollection.push(item);
        });
        // the same thing can be achived by below line
        // _flatCollection.push.apply(_flatCollection, childCollection);
      });
      return _flatCollection;
  };

Array.prototype.flatMap 
  = Array.prototype.concatMap 
  = function(mapper) {
      return this.map(function(item) {
        return mapper(item);
      }).flat();
  };

// zip(...collections, combiner);
Array.prototype.zip = function(...args) {
  var len = args.length,
      combiner,
      zipCollections,
      zipLen,
      zipCounter = -1,
      zipArgs,
      result = [];


  if(len == 0) {
    return this;
  } else if(len == 1) {
    throw new Error('Invalid arguments. zip(...collections, combiner)');
  } else if(len >= 2 && typeof (combiner = args.pop()) == 'function') {
    zipCollections = [this];
    zipCollections.push.apply(zipCollections, args);
    zipLen = zipCollections
    .map(function(c) {
      return c.length;
    })
    .reduce(function(min, next) {
      return next < min ? next : min;
    });

    while(++zipCounter < zipLen) {
      zipArgs = zipCollections.map(function(collection) {
        return collection[zipCounter];
      });
      result.push(combiner.apply(null, zipArgs));
    }
  }
  return result;
};

```

### Exercises

Exercise 24: Retrieve each video's id, title, middle interesting moment time, and smallest box art url. 
This is a variation of the problem we solved earlier. This time each video has an interesting moments collection, each representing a time during which a screenshot is interesting or representative of the title as a whole. Notice that both the boxarts and interestingMoments arrays are located at the same depth in the tree. Retrieve the time of the middle interesting moment and the smallest box art url simultaneously with zip(). Return an {id, title, time, url} object for each video.


```js
var movieLists = [
			{
				name: "New Releases",
				videos: [
					{
						"id": 70111470,
						"title": "Die Hard",
						"boxarts": [
							{ width: 150, height:200, url:"http://cdn-0.nflximg.com/images/2891/DieHard150.jpg" },
							{ width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/DieHard200.jpg" }
						],
						"url": "http://api.netflix.com/catalog/titles/movies/70111470",
						"rating": 4.0,
						"interestingMoments": [
							{ type: "End", time:213432 },
							{ type: "Start", time: 64534 },
							{ type: "Middle", time: 323133}
						]
					},
					{
						"id": 654356453,
						"title": "Bad Boys",
						"boxarts": [
							{ width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/BadBoys200.jpg" },
							{ width: 140, height:200, url:"http://cdn-0.nflximg.com/images/2891/BadBoys140.jpg" }

						],
						"url": "http://api.netflix.com/catalog/titles/movies/70111470",
						"rating": 5.0,
						"interestingMoments": [
							{ type: "End", time:54654754 },
							{ type: "Start", time: 43524243 },
							{ type: "Middle", time: 6575665}
						]
					}
				]
			},
			{
				name: "Instant Queue",
				videos: [
					{
						"id": 65432445,
						"title": "The Chamber",
						"boxarts": [
							{ width: 130, height:200, url:"http://cdn-0.nflximg.com/images/2891/TheChamber130.jpg" },
							{ width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/TheChamber200.jpg" }
						],
						"url": "http://api.netflix.com/catalog/titles/movies/70111470",
						"rating": 4.0,
						"interestingMoments": [
							{ type: "End", time:132423 },
							{ type: "Start", time: 54637425 },
							{ type: "Middle", time: 3452343}
						]
					},
					{
						"id": 675465,
						"title": "Fracture",
						"boxarts": [
							{ width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/Fracture200.jpg" },
							{ width: 120, height:200, url:"http://cdn-0.nflximg.com/images/2891/Fracture120.jpg" },
							{ width: 300, height:200, url:"http://cdn-0.nflximg.com/images/2891/Fracture300.jpg" }
						],
						"url": "http://api.netflix.com/catalog/titles/movies/70111470",
						"rating": 5.0,
						"interestingMoments": [
							{ type: "End", time:45632456 },
							{ type: "Start", time: 234534 },
							{ type: "Middle", time: 3453434}
						]
					}
				]
			}
		];

function smallestBoxart(candidate, current) {
  return candidate.width * candidate.height < current.width * current.height 
    ? candidate : current;
}

function middleInterestingMoment(moment) {
  return moment.type == 'Middle';
}

movieLists.flatMap(function(movieList) {
	return movieList.videos.flatMap(function(video) {
    var smallestBoxArt = video.boxarts.reduce(smallestBoxart);
    var interestingMoment = video.interestingMoments.filter(middleInterestingMoment); 
		return 	smallestBoxArt.zip(interestingMoment, function(boxart, interestingMoment) { 
							return { 
                id: video.id, 
								title: video.title, 
                time: interestingMoment.time, 
                url: boxart.url
              };
					});
	});
});

```



Exercise 26: Converting from Arrays to Deeper Trees
Let's try creating a deeper tree structure. This time we have 4 separate arrays each containing lists, videos, boxarts, and bookmarks respectively. Each object has a parent id, indicating its parent. We want to build an array of list objects, each with a name and a videos array. The videos array will contain the video's id, title, bookmark time, and smallest boxart url. In other words we want to build the following structure:

`output shema required`
```js
[
	{
		"name": "New Releases",
		"videos": [
			{
				"id": 65432445,
				"title": "The Chamber",
				"time": 32432,
				"boxart": "http://cdn-0.nflximg.com/images/2891/TheChamber130.jpg"
			},
			{
				"id": 675465,
				"title": "Fracture",
				"time": 3534543,
				"boxart": "http://cdn-0.nflximg.com/images/2891/Fracture120.jpg"
			}
		]
	},
	{
		"name": "Thrillers",
		"videos": [
			{
				"id": 70111470,
				"title": "Die Hard",
				"time": 645243,
				"boxart": "http://cdn-0.nflximg.com/images/2891/DieHard150.jpg"
			},
			{
				"id": 654356453,
				"title": "Bad Boys",
				"time": 984934,
				"boxart": "http://cdn-0.nflximg.com/images/2891/BadBoys140.jpg"
			}
		]
	}
]
```
```js

var lists = [
			{
				"id": 5434364,
				"name": "New Releases"
			},
			{
				"id": 65456475,
				name: "Thrillers"
			}
		],
		videos = [
			{
				"listId": 5434364,
				"id": 65432445,
				"title": "The Chamber"
			},
			{
				"listId": 5434364,
				"id": 675465,
				"title": "Fracture"
			},
			{
				"listId": 65456475,
				"id": 70111470,
				"title": "Die Hard"
			},
			{
				"listId": 65456475,
				"id": 654356453,
				"title": "Bad Boys"
			}
		],
		boxarts = [
			{ videoId: 65432445, width: 130, height:200, url:"http://cdn-0.nflximg.com/images/2891/TheChamber130.jpg" },
			{ videoId: 65432445, width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/TheChamber200.jpg" },
			{ videoId: 675465, width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/Fracture200.jpg" },
			{ videoId: 675465, width: 120, height:200, url:"http://cdn-0.nflximg.com/images/2891/Fracture120.jpg" },
			{ videoId: 675465, width: 300, height:200, url:"http://cdn-0.nflximg.com/images/2891/Fracture300.jpg" },
			{ videoId: 70111470, width: 150, height:200, url:"http://cdn-0.nflximg.com/images/2891/DieHard150.jpg" },
			{ videoId: 70111470, width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/DieHard200.jpg" },
			{ videoId: 654356453, width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/BadBoys200.jpg" },
			{ videoId: 654356453, width: 140, height:200, url:"http://cdn-0.nflximg.com/images/2891/BadBoys140.jpg" }
		],
		bookmarks = [
			{ videoId: 65432445, time: 32432 },
			{ videoId: 675465, time: 3534543 },
			{ videoId: 70111470, time: 645243 },
			{ videoId: 654356453, time: 984934 }
		];

lists.map(function(list) {
  return {
    name: list.name,
    videos: 
      videos.filter(function(video) {
        return video.listId == list.id
      })
      .concatMap(function(video) {
        return bookmarks.filter(function(bookmark) {
          return bookmark.videoId == video.id;
        })
        .zip(
          boxarts.reduce(function(r, c) {
            return r.width*r.height < c.width*c.height ? r : c
          }),
          function(bookmark, boxart) {
            return {
              id: video.id,
              title: video.title,
              time: bookmark.time,
              boxart: boxart.url
            }
          }
        )
      })
  }
});

```


That will be it for this post.
