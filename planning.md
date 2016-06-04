# Planning

## What is it?

Instead of imperative code where you use callback calling callbacks forever, the flow of data is defined declaratively so with a sort of pipeline of events.

Separates processing of events from the flow of data.

Makes use of chaining APIs using a rich set of "combinators" which perform operations on the events, and return a new stream of events.

## Reactive streams are like Arrays

Combinators are similar to things you could do with Arrays, except they represent a list of events over time, rather than a static list of events

### Basic operations

Here are some operations that are similar in both arrays and reactive streams

- **Map:** Converts each element in an array or stream into an element in a resulting array or stream. The resulting array or stream has the same number of elements
- **Filter:** Creates a new array or stream and adds elements from the source stream based on whether they pass some sort of check. The resulting array or stream will have the same number or fewer elements
- **Concat:** Takes another array or stream as an argument, creates a new array or stream which contains all the elements from one array or stream, followed by all the elements from the first array or stream
- **Reduce:** Converts all the elements in an array or stream into a different type of value by building up a result with each element. For arrays the result is usually just the final computed value, but for reactive libraries they return either an stream which will contain just one item, or a promise which will resolve to the value

## References:

Andre Staltz is the bomb, so follow him everywhere.

[Introduction to Reactive Programming](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)

[RxJS](https://github.com/Reactive-Extensions/RxJS) is the most popular implementation.

[MostJS](https://github.com/cujojs/most/blob/master/docs/api.md) is a cool implementation of Reactive streams.

[es-observable](https://github.com/zenparsing/es-observable) is the standard being added to the JavaScript runtime. It's going to let you inter-operate with different libraries, sort of like what promises/A+ did for promise libraries.
