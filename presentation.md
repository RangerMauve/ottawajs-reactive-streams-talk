<!-- .slide: data-background="center/cover url(http://i.giphy.com/kSjVq32vrk0MM.gif) no-repeat" -->

# Functional Reactive Programming

by Georgiy S.

@rangermauve

---

<!-- .slide: data-background="center/cover url(http://i.giphy.com/d2ZhIkBvogTIzOik.gif) no-repeat" -->

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
