---
title: API Reference

language_tabs:
  - dart: Dart
  - java: Java
  - node: Node.js
  - javascript: Javascript
  - python: Python
  - ruby: Ruby
  - c: C
  - csharp: C#
  - scala: Scala

toc_footers:
  - <a href="http://iot-dsa.org/">IOT-DSA Homepage</a>
  - <a href="https://github.com/IOT-DSA/docs/wiki">IOT-DSA Wiki</a>
  - <a href="http://iot-dsa.github.io/docs/sdks/dart/index.html">Dart API Reference</a>

includes:
  - requesters

search: true
---

# Introduction

This guide will introduce you to the basics of communicating with a Broker using
the DSLink SDK. We'll learn how to send data to a Broker as well as how to
subscribe to data to get updates when other, or even our, data is changed.

# Importing

> Use the following to import the SDK.

```dart
// pubspec.yaml
dependencies:
  dslink:
    git: git://github.com/IOT-DSA/sdk-dslink-dart.git


// run.dart
import 'package:dslink/dslink.dart';
```

```ruby
# Add to your Gemfile
gem 'dslink', :git => 'git://github.com/IOT_DSA/sdk-dslink-ruby.git'

#link.rb
require 'dslink'
```

By using the Git repository, we ensure that we have the most up-to-date version
of the DSLink SDK. In the future, the SDK may be made available through common
package managers for each language.

# Responder

There are several types of DSLinks we can create. The first of which is called
a *Responder*. The Responder will provide data on demand. It will only send the
data when it's requested, eg it Responds.

<aside class="notice">
The other types of Links are a <em>Requester</em> and a hybrid <em>Responder</em>
and <em>Requester</em>. We'll look at those types later.
</aside>

## Link

> Note that in some sdks we must pass the application startup arguments to the
> link creation, other SDKs will automatically retrieve these arguments on their
> own.

```dart
// run.dart
main(List<String> args) {
  var link = new LinkProvider(args, 'Example-');
  link.connect();
  // ...
}
```

In order to create any type of connection (Responder or Requester), we first
need to establish a link to the DSA Broker. In this example we will only
establish the bare minimum link. There are a number of configuration options
to further extend the capabilities. Refer to the documentation for the SDK
of the language of your choice.

The most basic Link will accept command line arguments passed the application
as well as a name for the Link. The ending hyphen is not optional, and will not
appear in the link. The name supplied here is what will show up under your
downstream connections (by default).

## Nodes

Within the DSA Broker, all of our data is broken into a collection of nodes.
A node may be a link to a separate device, or it may be multiple links running
from a server. Each link may also have children nodes.

A node structure can be through of like a file directory tree on your computer
or a website's address/path. A Responder link will only have access to the nodes
it contains. As we will see later, a Requester can access any node that it has
permission to access.

### Generate a Random Number

> Generate a random number

```dart
// At the top of the file add the import for the Math library.
import 'dart:math';
// ...

var numGen = new Random();
var myNum = numGen.nextInt(50);
```

Before we add the new, lets create a value that we can pass, and send updates
for. In this case we'll just generate a random number between 0 and 50.

### Add node

> Add the node to our responder link.

```dart
var myNode = link.addNode('/MyNum',
// Note the raw strings, the $ must be passed as well.
  { r'$name': 'My Number',
    r'$type' : 'int',
    '?value' : myNum});
```

Next we add our node to the Link. In this case we pass the node name, `/MyNum`
and the configuration. The node name begins with a forward-slash \(`/`\)
which indicate it is appended to the root of our DSLink.

In our node configuration, we provide a user friendly display name for our link,
this name will only appear in the the Link information, but does not affect the
name or path used when trying to access the value. Additionally we provide the
type of value that this node will provide, and the value itself, which is our
random number that we generated. For information on additional configuration
options for a node please refer to
[Node Configs](https://github.com/IOT-DSA/docs/wiki/Configs)

When we add the node, it will return a local copy of the node for us to work
with directly. Alternatively we can also perform many of the same operations
directly with link itself.

## Updating a node

```dart
new Timer.periodic(const Duration(seconds: 5), (_) {
// To be filled in soon.
});
```

A node is of limited value if it only provides the initial request and nothing
further. We want to provide updates to the value as things change. For our
demonstration, we will setup a timer which updates our number once ever five
seconds.

## Checking for a subscription

> Add this to the timer function.

```dart
if(myNode.hasSubscriber) {
  // To be filled in soon
}
```

Generally, if there is no one actively listening, or *subscribing* to our node
there is not a lot of reason to update the value. So first, we will check to
see if the node has a subscriber.

## Updating the value

> Update the random number then update our value.

```dart
myNum = numGen.nextInt(50);
myNode.updateValue(myNum);
```

Once we are sure someone is subscribed to changes in our node, we generate a
new random number, then update the value of our node.

## Adding an action

```dart
// Our new constructor in which we pass both our default node for
// adding a value, and our profile, or action on that node.
var nodeList = []; // Store all nodes we create to reference later.

// Following replaces previous link initialization.
link = new LinkProvider(args, 'Example-', defaultNodes: {
  'AddNum' : {
    r'$name' : 'Add number',
    r'$is' : 'addnum',
    r'$invokable' : 'write',
    r'$params' : [{'name' : 'value', 'type' : 'int'}]
  }}, profiles: {
    'addnum' : (String path) =>
        new SimpleActionNode(path, (Map<String, dynamic>params) {
          var addVal = int.parse(params['value']);
          var ndNum = nodeList.length;
          nodeList.add(link.addNode('/MyNum$ndNum', {
            r'$name' : 'My node #$ndNum',
            r'$type' : 'int',
            '?value' : addVal
          }));
          // myNode.updateValue(myNum + addVal);
        })
});
```

```dart
// Add our initial node to our node list instead of a single variable
nodeList.add(link.addNode('/MyNum',
  { r'$name': 'My Number',
    r'$type' : 'int',
    '?value' : myNum})
);

// Update our timer to provide a new random number for each node not just
// the initial one.
new Timer.periodic(const Duration(seconds: 5), (timer) {
  for(var myNode in nodeList) {
    // Don't update if there's no value
    if (myNode.value == null) continue;
    if (myNode.hasSubscriber) {
      myNum = numGen.nextInt(50);
      myNode.updateValue(myNum);
    }
  }
});
```

Sometimes we may wish to allow our Link to respond to an action. That is, we
may want to allow a user to send a command to our link so it can perform
some type of function. That function may be to execute a command on the system,
poll or retrieve additional data, or any other functionality we may want to
include.

In our sample, we are going to add a command to allow our number generator
to provide multiple values, and initialize it with a default value.

To setup our actions is a two-step process. The first of which is to add a node
to our Link. The second is to add a `profile` to the link which will correspond
to the node. A profile can be considered a method which is executed when a node
of that type receives an `invoke` call. The return of this method is a node
which executes our desired tasks and returns, if applicable, any data. This
process will become clear soon.

<aside class="warning">
Currently, the only way to accurately provide a profile is by passing it as part
of the constructor for the LinkProvider.
</aside>

<aside class="notice">
The SDK's contain some helper classes which make the creation of a node easier.
</aside>

## Saving a link's state

> Initialize the link to load any existing saved data.

```dart
// after the link = new LinkProvider(..) and before link.connect();
link.init();
```

> Call save on the link.

```dart
// Add outside of the for loop inside the Timer call.
link.save();
```

We can now add as many values to our link as we like. And every 5 seconds, a
new number is generated for each of these nodes. However if we stop our link,
and restart it, we'll find that all of the additional nodes we created no longer
exist. Nor does it initialize with the previous values which were being stored.

When a link is saved, it will create a `node.json` which will store the state
of our link and nodes at that time. In order to properly use the data from
the save file, we also need to ensure we initialize the link prior to
connecting.

<aside class="notice">
Some SDK's will automatically initialize the node on connection, however it is
best practice to explicitly initialize the link yourself.
</aside>

Now if we restart our link, the values we added are already included when it
reconnects to our broker

## Getting child nodes

> Get the root node of our Responder Link.

```dart
// At the top of our Timer callback.
var rootNode = ~link; // Shortcut for link.getNode('/');
```

> Update for loop to iterate over the children of the root node.

```dart
/// rootNode.children returns a Map<String, Node> where String
/// is the node identifier and Node is the [Node] object itself.
for(var nodeName in rootNode.children.keys) {
  var myNode = rootNode[nodeName];
  // ... Existing checking for subscriber and updating value
}
```

After we've restarted the link, you may notice a slight problem. Any values that
were added to the node are not being updated. This is because we explicitly
tracked them in our own list rather than querying our link for its child nodes.

First we query our link for the root node (top-most node). Once we have that
we can iterate over the children nodes to update each of those values.

## Nesting Nodes

> Update the default nodes

```dart
/// Change the default nodes from our constructor.
/// The rest of the constructor stays the same.
defaultNodes: {
  'CustomNumbers' : {
    r'$name' : 'Custom Numbers',
    'AddNum' : {
      // ... Previous information still here.
    }
  }
},
```

> Update location our action adds to.

```dart
// Within our addnum profile update where we add the
// new node when the action runs.
link.addNode('/CustomNumbers/MyNum$ndNum', {
// ... rest of the code is the same.
```

As the number of nodes grows, having a series of nodes all at the top level
can become a nightmare. Often it is preferable to create a hierarchy for values.
Lets change the location of our custom numbers to be under their own node and
move the action to that node.

First create a new mapping of the default node hierarchy. In this case we're
actually wrapping our existing `AddNum` node with the new node `CustomNumbers`.

Next we just need to ensure that when the action is triggered to add a new
custom value (eg: `My Node #1`), that it gets added under the `Customer Numbers`
section as opposed to the top level node.

<aside class="notice">
It's left as an exercise to the reader to implement the required updates
to generate random values in the new location, as well as the original
MyNum value.
</aside>

The `'$name'` field of our `CustomNumbers` node is entirely optional, but
it provides an easier way of reading the node within a broker. However this
value does not need to match the name of the node in any way and does not
impact its configuration at all. It is only meta data. Full documentation on
the meta data and configuration for a node is
[available from our Wiki](https://github.com/IOT-DSA/docs/wiki/Configs#core-configs).

<aside class="warning">
Node paths should be alphanumeric only with underscores. Node paths cannot
include spaces, or a number of other special characters.
</aside>
