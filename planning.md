# Planning

## What is it?

Instead of imperative code where you use callback calling callbacks forever, the flow of data is defined declaratively so with a sort of pipeline of events.

Separates processing of events from the flow of data.

Makes use of chaining APIs using a rich set of "operators" which perform operations on the events, and return a new stream of events.

Also called "Observables".

## Reactive streams are like Arrays

operators are similar to things you could do with Arrays, except they represent a list of events over time, rather than a static list of events.

## Visualizing via Marble Diagrams

Streams of events rely on time, so it's useful to show how different streams will progress with regards to each other. This is done by showing each stream as a line that goes forward in time, with events noted as values along the line.

### Marble Diagram Syntax

A simple markup can be used to visualize the progress of various streams relative to each other. It's syntax involves one or more lines with the following characters:

- `-`: represents a moment in time when a reactive stream is doing nothing
- `|`: represents the end of a reactive stream. No more events will be emitted after this point
- `#`: represents an error being thrown in the stream. No more events will be emitted after this point
- `>`: represents that the stream will continue indefinitely
- `()`: represents events that occur in the same instant in time
- `abc`: Single characters or numbers are used to represent individual events in a stream

### Examples

```
------1-(234)-5|
```

```
1-2-3-4-5->
```

```
--------5#
```

## Basic operators

Most reactive stream libraries share some operators with arrays so they're the easiest to get a grip on

### Map

Converts each element in an array or stream into an element in a resulting array or stream. The resulting array or stream has the same number of elements

```
S:                 1-2-3-4-5|
S.map(x => x + 1): 2-3-4-5-6|
```

### Filter

Creates a new array or stream and adds elements from the source stream based on whether they pass some sort of check. The resulting array or stream will have the same number or fewer elements

```
S:                   1-2-3-4-5|
S.filter(x => x <3): 1-2------|
```

### Concat

Takes another array or stream as an argument, creates a new array or stream which contains all the elements from one array or stream, followed by all the elements from the first array or stream

```
S1:            1-2-3-4-5|
S2:            -6-7-8-9-|
S1.concat(S2): 1-2-3-4-5-6-7-8-9-|
```

### Reduce

Converts all the elements in an array or stream into a different type of value by building up a result with each element. For arrays the result is usually just the final computed value, but for reactive libraries they return either an stream which will contain just one item, or a promise which will resolve to the value

```
S:                        1-2-3-4-5|
S.reduce((a,b) => a * b): --------120|
```

## Advanced operators

Array operators are nice, but to take full advantage of streams, it's useful to make use of the time aspect as well as making it easy to create streams of streams

### Scan

Scan is very similar to `reduce`, but instead of calculating one final value, it'll output each intermediate value as well

```
S:                    1-2-3-4-5|
S.scan((a,b) => a*b): --2-6-24-120|
```

### Merge

Takes a variable number of streams, and merges events from all of them. Events will be relative to each other based on the time they were created, whereas in `concat` the events get emitted one stream at a time. Note that if events can happen at the same time, order isn't necessarily guaranteed, so one must be ready to handle race conditions.

```
S1:              1--2--3|
S2:              a-b---c|
S3:              --6--7-|
S1.merge(S2,S3): (1a)-(b6)2-7(3C)
```

### Combine / CombineLatest / CombineArray

Takes a series of streams and a callback. Whenever a stream changes, the last value seen from all streams is passed to the callback, and the result is used in the resulting stream. Note that different libraries have different orders for the arguments. Sometimes the callback is first, sometimes a stream is taken first. Sometimes an array of streams is taken, sometimes streams are passed in as multiple arguments.

```
S1:                        1-2-3-4|
S2:                        -5---4-|
S1.combine(S2,(a,b)=>a+b): -67-879|
```

### takeUntil

Stops a stream when another stream emits its first event.

```
S1:               1-2-3-4-5|
S2:               -----6--7|
S1.takeUntil(S2): 1-2-3-|
```

### flatMap / flatten / chain

Allows you to map over a stream, and returns streams as the result. The streams will then be mixed together for the result. Different libraries might behave slightly differently, so it's important to double check that it's doing what you want.

- Rx: Merges all resulting streams together, so their events are interleaved based on the time they occurred at
- Most: Same as Rx. Uses either `flatMap` or `chain` as the method name
- Kefir: Same as Rx
- xstream: Has two steps, `map` to return a stream of streams, and an explicit `flatten`

```javascript
function f(x) {
    return Observable.periodic(x,x);
}
```

```
S:            1-2-3|
S.flatMap(f): 1-2-32--(23)---3|
```

### constant / mapTo

Maps all events in a stream to constant values. Rx and Kefir don't hav this, use `map` and return a constant instead.

```
S:          1-2-3-4-5|
S.mapTo(8): 8-8-8-8-8|
```

## Hot and Cold

Most of these libraries make use of a concept called `hot` and `cold` streams. By default, when you set up a series of streams, nothing actually happens. What happens is the pipeline is ready to be executed, but nothing actually subscribes to the original event sources until something subscribes on the pipeline itself.

To make a point:

```javascript
var someInput = "cats are cool".split("");
var backwardsInput = Rx.Observable
    .from(someInput)
    .map((l) => l.toUpperCase())
    .scan((acc, k) => k + acc , "");
```

This actually won't do anything since nothing is subscribing to the `backwardsInput` stream. In order for this to actually perform something, one must first call `backwardsInput.subscribe`. This will then start subscribing to things up the chain of observables until the top most one calls actually starts iterating through the source array.

```javascript
backwardsInput.subscribe((result) => console.log(result));
```

The fact that nothing actually executes until an explicit subscribe is made is what makes a stream `cold`.

The opposite is a stream that will create events regardless of when. An example of this is `mousemove` events from an element (in RX). They will start getting pushed down to any potential subscribers right away. The fact that they are active even when there is no explicit subscription is what makes them `hot`.

Not all Reactive Programming libraries have a concept of cold observables. The Observable implementation that's going to be part of the JavaScript runtime, for instance, only has a concept of `cold` observables.

Generally, most sources of data can be modelled as `cold` observables. xstream, however uses exclusively `hot` observables.

## Unicast and Multicast

In most cases, every time `subscribe` is called on a stream, it creates a new pipeline that connects to the source of data. The fact that the pipeline produces data for one subscriber at a time is what makes it `unicast`. This could lead to unexpected results when computations or side effects might be duplicated.

To avoid wasted cycles and confusing code, most libraries provide a way to turn an existing stream into a `multicast` Observable. What this means is that the entire pipeline up until pipeline up until that point is going to be shared for any subsequent observables. This is useful if you have a long chain of computations which will split up into many branches, or for cases where you want to branch out, but don't want to have duplicated side-effects or subscriptions to a data source.

Multicast observables are usually `hot`.

## References:

[Andre Staltz](https://twitter.com/andrestaltz) is the bomb, so follow him everywhere.

[Introduction to Reactive Programming](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)

[RxJS](https://github.com/Reactive-Extensions/RxJS) is the most popular implementation. Has a lot of focus on being "correct" and can sometimes get caught on unnecessary details and a lot of types. Most "mature" implementation in that it was ported over from predecessors in C# and Java

[MostJS](https://github.com/cujojs/most/blob/master/docs/api.md) is a cool implementation of Reactive streams. The concepts are presented in a simple way, and the API docs are great. Doesn't seem to be as popular as RX though.

[Kefir](http://rpominov.github.io/kefir/) is another implementation that focused on high performance and low memory usage. Apparenly slower than MostJS. Seems to have the most different API between the three.

[xstream](https://github.com/staltz/xstream) a minimal implementation by Andre Staltz. Has a few simple concepts and types to learn. Few operators that were shown to be the most often used from Rx.

[es-observable](https://github.com/zenparsing/es-observable) is the standard being added to the JavaScript runtime. It's going to let you inter-operate with different libraries, sort of like what promises/A+ did for promise libraries.

[The MostJS internals](https://github.com/cujojs/most/wiki/Architecture) provide some good insight as to how the internals work
