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

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:

search: true
---

# Introduction

So you have some data that you would like to be able to send to a DGLux server
or DSA Broker, but you’re not sure how to accomplish that? This guide will
introduce you to the basics of communicating with a Broker using the DSLink SDK.
We'll learn how to send data to a Broker as well as how to subscribe to data
to get updates when other, or even our, data is changed.

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

```java
// TODO
```

By using the Git repository, we ensure that we have the most up-to-date version
of the DSLink SDK. In the future, the SDK may be made available through common
package managers for each language.

# Responder

There are several types of DSLinks we can create. The first of which is called
a *Responder*. The Responder will provide data on demand. It will only send the
data when it's requested, eg it Responds.

<aside class="notice">
The other types of Links are Requester
and a hybrid Responder and Requester. We'll look at those types later.
</aside>

## Link

> Note that we pass the application startup arguments to the link creation.

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

Generally, if there is no one actively listening, or _subscribing_ to our node
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
