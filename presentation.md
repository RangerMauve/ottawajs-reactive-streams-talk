<!-- .slide: data-background="center/cover url(http://i.giphy.com/kSjVq32vrk0MM.gif) no-repeat" -->

# Functional Reactive Programming

by Georgiy S.

@rangermauve

---

## What is it

- Declarative
- Streams of events
- "Observables"

Note: The term "Functional Reactive Programming" has been more popular lately, and for good reason. It's a nice way to organize complex applications with a lot of interleaved asynchronous events. This is done mainly by abstracting parts of the codebase with the "Observable" data type, which is basically just a stream of events that can be composed along with other streams to produce complex behaviors in e declarative way.

---

### Kinda like arrays, but over time

- Container for elements
- Ordered
- Added time component
- Operators for modifying streams and creating new ones

![Marble Diagram](https://niallconnaughton.files.wordpress.com/2015/08/mergemarble2.png?w=665)

Note: One could think of these streams as being similar to arrays. In ES5, we have a bunch of cool methods on arrays that let us transform them and work with them in a more functional way. Streams contain elements, just like arrays, but these elements are not just ordered by index, but also in time. What this means is that rather then them being available the instance a stream is defined, and instead of them all being processed at the same time, they will become available for processing throughout the lifecycle of the stream.

---

## Four main flavors

 - [RxJS](https://github.com/Reactive-Extensions/RxJS)
 - [MostJS](https://github.com/cujojs/most/blob/master/docs/api.md)
 - [Kefir](http://rpominov.github.io/kefir/)
 - [xstream](https://github.com/staltz/xstream)

Note: Rx is the most popular implementation. Has a lot of focus on being "correct" and can sometimes get caught on unnecessary details and a lot of types. Most "mature" implementation in that it was ported over from predecessors in C# and Java. Most is a cool implementation of Reactive streams. The concepts are presented in a simple way, and the API docs are great. Doesn't seem to be as popular as RX though. Kefir is another implementation that focused on high performance and low memory usage. Apparenly slower than MostJS. Seems to have the most different API between the three. xstream a minimal implementation by Andre Staltz. Has a few simple concepts and types to learn. Few operators that were shown to be the most often used from Rx.


---

## Visualizing Events

"Marble Diagrams"

```
------1-(234)-5|
```

```
1-2-3-4-5->
```

```
--------5#
```

Note: Streams can be visualized for documentation or planning purposes using a type of visualization called a "marble diagram". This consists of some simple text (or visual) language which shows time intervals, events, and the lifecycles of streams.

---

### Marble Diagram Syntax

- `-`: a moment in time
- `|`: end of a stream
- `#`: error being thrown in the stream
- `>`: the stream will continue indefinitely
- `()`: events that occur in the same time
- `abc`: individual events in a stream

---

<!-- .slide: data-background="center/cover url(http://i.giphy.com/GOevs1pZWr4xa.gif) no-repeat" -->
## Basic Operators

Note: Operators are chainable methods on observables which are used to transform one observable, into a new one with different functionality. This is what is used to make declarative pipeliens for data to go down. Most FRP libraries share some methods with Arrays. These are probably the easiest to grasp, so we'll go over them first

---

### Map

Converts each element to a new value based on a function

```
S:                 1-2-3-4-5|
S.map(x => x + 1): 2-3-4-5-6|
```

Note: Converts each element in an array or stream into an element in a resulting array or stream. The resulting array or stream has the same number of elements

---

### Filter

Goes through each element and ignores some based on a function


```
S:                   1-2-3-4-5|
S.filter(x => x <3): 1-2------|
```

Note: Creates a new array or stream and adds elements from the source stream based on whether they pass some sort of check. The resulting array or stream will have the same number or fewer elements

---

### Concat

Creates a new stream with all the elements of the original, followed by all the elements from a second stream


```
S1:            1-2-3-4-5|
S2:            -6-7-8-9-|
S1.concat(S2): 1-2-3-4-5-6-7-8-9-|
```

Note: Takes another array or stream as an argument, creates a new array or stream which contains all the elements from one array or stream, followed by all the elements from the first array or stream

---

### Reduce

Converts all elements into a new value

```
S:                        1-2-3-4-5|
S.reduce((a,b) => a * b): --------120|
```

Note: Converts all the elements in an array or stream into a different type of value by building up a result with each element. For arrays the result is usually just the final computed value, but for reactive libraries they return either an stream which will contain just one item, or a promise which will resolve to the value

---

<!-- .slide: data-background="center/cover url(http://i.giphy.com/jiFZPKBReng88.gif) no-repeat" -->
## Advanced Operators

Note: Array operators are nice, but to take full advantage of streams, it's useful to make use of the time aspect as well as making it easy to create streams of streams

---

### Scan

Reduce, but with intermediate steps

```
S:                    1-2-3-4-5|
S.scan((a,b) => a*b): --2-6-24-120|
```

Note: Scan is very similar to `reduce`, but instead of calculating one final value, it'll output each intermediate value as well

---


### Merge

Merges events from multiple streams into one, ordered by time

```
S1:              1--2--3|
S2:              a-b---c|
S3:              --6--7-|
S1.merge(S2,S3): (1a)-(b6)2-7(3C)
```

Note: Takes a variable number of streams, and merges events from all of them. Events will be relative to each other based on the time they were created, whereas in `concat` the events get emitted one stream at a time. Note that if events can happen at the same time, order isn't necessarily guaranteed, so one must be ready to handle race conditions.

---

### Combine / CombineLatest / CombineArray

Like map, but with multiple streams at once


```
S1:                        1-2-3-4|
S2:                        -5---4-|
S1.combine(S2,(a,b)=>a+b): -67-879|
```

Note: Takes a series of streams and a callback. Whenever a stream changes, the last value seen from all streams is passed to the callback, and the result is used in the resulting stream. Note that different libraries have different orders for the arguments. Sometimes the callback is first, sometimes a stream is taken first. Sometimes an array of streams is taken, sometimes streams are passed in as multiple arguments.

---

### takeUntil

Passes through elements until another stream emits

```
S1:               1-2-3-4-5|
S2:               -----6--7|
S1.takeUntil(S2): 1-2-3-|
```

Note: Returns a new stream which contains all the elements of the original, and takes another stream as an argument. The events from the original will be relayed until the second stream emits something. Then the new stream will end and stop sending more elements

---

### flatMap / flatten / chain

Maps over a stream, returning new streams. Flattens all resulting streams into a single one

```javascript
function f(x) {
    return Observable.periodic(x,x);
}
```

```
S:            1-2-3|
S.flatMap(f): 1-2-32--(23)---3|
```

Note: Allows you to map over a stream, and returns streams as the result. The streams will then be mixed together for the result. Different libraries might behave slightly differently, so it's important to double check that it's doing what you want. Rx: Merges all resulting streams together, so their events are interleaved based on the time they occurred at. Most: Same as Rx. Uses either `flatMap` or `chain` as the method name. Kefir: Same as Rx. xstream: Has two steps, `map` to return a stream of streams, and an explicit `flatten`

---

### constant / mapTo

Maps all events in a stream to a constant value.

```
S:          1-2-3-4-5|
S.mapTo(8): 8-8-8-8-8|
```

Note: Rx and Kefir don't have this, use `map` and return a constant instead.

---

## Hot and Cold Observables

- **Cold:** Doesn't push data until something is subscribed
- **Hot:** Creates data even if nothing has subscribed

Note: Most of these libraries make use of a concept called `hot` and `cold` streams. By default, when you set up a series of streams, nothing actually happens. What happens is the pipeline is ready to be executed, but nothing actually subscribes to the original event sources until something subscribes on the pipeline itself.

---

### Cold pipelines don't do anything


```javascript
var someInput = "cats are cool".split("");
var backwardsInput = Rx.Observable
    .from(someInput)
    .map((l) => l.toUpperCase())
    .scan((acc, k) => k + acc , "");
```

Note: This actually won't do anything since nothing is subscribing to the `backwardsInput` stream. In order for this to actually perform something, one must first call `backwardsInput.subscribe`. This will then start subscribing to things up the chain of observables until the top most one calls actually starts iterating through the source array.

---

### Subscribing makes things happen

```javascript
backwardsInput.subscribe((result) => console.log(result));
```

Note: The fact that nothing actually executes until an explicit subscribe is made is what makes a stream `cold`. The opposite is a stream that will create events regardless of when. An example of this is `mousemove` events from an element (in RX). They will start getting pushed down to any potential subscribers right away. The fact that they are active even when there is no explicit subscription is what makes them `hot`. Not all Reactive Programming libraries have a concept of cold observables. The Observable implementation that's going to be part of the JavaScript runtime, for instance, only has a concept of `cold` observables. Generally, most sources of data can be modelled as `cold` observables. xstream, however uses exclusively `hot` observables.

---

## Unicast and Multicast

- Unicast: Sets up a fresh source per subscription
- Multicast: Re-uses the same source between subscriptions (hot)

Note: In most cases, every time `subscribe` is called on a stream, it creates a new pipeline that connects to the source of data. The fact that the pipeline produces data for one subscriber at a time is what makes it `unicast`. This could lead to unexpected results when computations or side effects might be duplicated. To avoid wasted cycles and confusing code, most libraries provide a way to turn an existing stream into a `multicast` Observable. What this means is that the entire pipeline up until pipeline up until that point is going to be shared for any subsequent observables. This is useful if you have a long chain of computations which will split up into many branches, or for cases where you want to branch out, but don't want to have duplicated side-effects or subscriptions to a data source. Multicast observables are usually `hot`.


## Example

```javascript
var numbers = Rx.Observable.from([1,2,3,4,5]);

var mappedNumbers = numbers.map(function(number){
	console.log("Currently on:", number);
	return number - 1;
});

var sum = mappedNumbers.reduce((a,b) => a + b);

var product = mappedNumbers.reduce((a, b) => a * b);

sum.subscribe((result) => console.log("The sum is", result));
product.subscribe((result) => console.log("The product is", result));
```

---

## Forcing things to be mutlicast

- Assume things are unicast
- use `multicast` operator
- Don't overuse it, be functional

---

## References

[Andre Staltz](https://twitter.com/andrestaltz) is the bomb, so follow him everywhere.

[Introduction to Reactive Programming](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)

[es-observable](https://github.com/zenparsing/es-observable) is the standard being added to the JavaScript runtime. It's going to let you inter-operate with different libraries, sort of like what promises/A+ did for promise libraries.

[The MostJS internals](https://github.com/cujojs/most/wiki/Architecture) provide some good insight as to how the internals work

---

## Get coding!
