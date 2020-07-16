### Memory leak:
  a resource leak that occurs when a computer program incorrectly manages memory allocations
  in a way that memory which is no longer required is not released after a completed task.
  For some reason, the application neglects to release memory which keeps on being consumed
  without true need for it to happen

In JS, memory is assigned automatically, each time a new array, object, string or DOM element is
created.

The biggest problem with memory leaks, is that they are inherent code flaws, not errors with outputs
which you can debug. They result from valid code that compiles.
However, if your application is running slow / crashing, then it marks the first clue to having  a 
memory leak.
The main cause of leaks is unwanted references.

### Object memory graph:
  a collection of all objects and their references/(GC root distance) in an program.
  The reference path from the most immediate context up to the top/root is called the **retaining path**.

All values in javascript are part of the object graph.
The retaining path is what classified the object as memory & when the path is cut, the subject value
cannot be reached from the root node & therefore it is garbage collected.

### Garbage collection:
  an automatic memory management process, operating at runtime, which uses the mark & sweep algorithm
  to track paths and find objects in the object graph that are referenced from their roots.
  
The mark and sweep algorithm reduces the definition of the object that is no longer requred/unreachable.
it assumes the knowledge of a set of roots/global objects & periodically start from these roots to find
objects that are referenced to them & collect all those whose references have been severed.
Garbage collection cannot be compelled in JS. It happens when it's heuristics decide it's the right time to do it
during runtime. Just like a surprise visit from the SEC to a hedge fund. That way they're are nondeterministic.

__(insert mans with vacuum cleaner here)__

Manually dereferencing is not necessary in most cases. But by simply putting variables where they need to be,
which is as local as possible is how things should work.

### Misconceptions in de-referencing
It is a common misconception that languages which rely on GC for mem mgmt, are resilient to memory leaks.
- Use of the `delete` keyword to force de-referencing does more harm than good behind the scenes, as it
changes an object's variable hidden class & makes it a generic slow object.

```
  var k = { 'x':1 }
  delete k.x // true
  k.x; // undefined
```
The delete keyword has nothing to do with freeing memory

- Setting an object to null doesn't null the object but updates its reference to null.
Using `k.x = null` is much better than using `delete`

- Global vars aren't cleaned up by the garbage collector during the life cyle of the page.
Instead use a function-local variable that goes out of scope when it is no longer needed.????



### Common memory leaks
- Accidental global variables
When referencing a global variable, you'll create a global variable.
eg. 
```
  function myFunc() {
    myVar = 'some value' // this var hasn't been declared before
    }
```
use the strict directive to mitigate this.

- Circular referencs
When 2 or more objects are created with properties that reference one another, thus creating a cycle.
```
  function myFunc() {
    var a = {};
    var b = {};
    a.x = y;
    b.y = a;
  }
  myFunc()
```
After the function has completed its execution, the vars will go out of scope & at that point
they'll become unrequired & therefore memory used should be reclaimed. However, the reference-counting
algorithm won't consider them reclaimable since each of the 2 objs  has at least one reference pointing
to them.

- Adding event listeners and forgetting to remove them
- Setting data as object in the DOM
- Forgetting to clear dead items out of the global cache
- console.log() objects 
- Unreleased timers/listeners added in componentDidMount
Registering event listeners in componentDidMount: when the component is mounted, receive events even after
the component gets unmounted. However this can be resolved by properly removing listeners in 
componentWillUnMount(). Also, placing registering/unregistering logic in HOCs helps to limit the number of places
where we need to register & unregister listeners while passing data from events as props to child components.
[Stop memory leaks with componentWillUnMount](https://egghead.io/lessons/react-stop-memory-leaks-with-componentwillunmount-lifecycle-method-in-react)



### How JS works in V8
V8 - Google's open source high performance JavaScript & Web Assembly engine written in C++.
Core pieces:
  - base compiler:
Parses Js & generates bytecode before it is executed. This code is initially not highly optimized.

  - Object model
Represents the objects in an object model. Objects are represented as associative arrays in Js,
but in V8 they are represented with hidden classes: (internal type system for optimized lookups).

  - runtime profiler
Monitors the program being run & identifies hot functions (i.e code that ends up spending a long time 
running)

  - optimizing compiler
Recompiles & optimizes the "hot" code identified by the runtime compiler such as **inlining** 
**(i.e replacing the function call site with the body of the callee)**

  - Deoptimization checks
V8 supports deoptimization, where the compiler can opt out of optimization if the compiler 
discovers that some of the assumptions made during optimization were too optimistic.
This is because, JIT compilers for dynamic languages uses type speculation to generate optimized 
code for frequently executed code sections hence making assumptions, but high performance code
requires assumption/speculation checks to ensure that the speculations made have not been violated.
In the event they are, the program reverts to executing unoptimized code. These checks are known
as deoptimization checks.

  - Garbage collector
Reclaims memory that is being used by objects which are no longer needed.


### Memory mgmt chrome dev tools
- Heap snapshots
Lets us know, which objects:
  - are currently retaining the most memory?
  - have been added since the last snapshot
  - failed to be GCed since the last snapshot?

- Allocation profilers
Lets us know
  - what code paths are allocating new heap memory ?
  - what code is making garbage for GC?

- Allocation timelines
Lets us know:
  - for objects that were not yet GCed, when were they allocated?
  - how rapidly is my garbage being collected?
