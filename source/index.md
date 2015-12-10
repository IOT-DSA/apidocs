---
title: API Reference

language_tabs:
  - dart: Dart
  - java: Java
  - plaintext: Node.js
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
  - footer

search: true
---

# Introduction

This guide will introduce you to the basics of communicating with a Broker using
the DSLink SDK. We'll learn how to send data to a Broker as well as how to
subscribe to data to get updates when other, or even our, data is changed.

<aside class="notice">
While not covered in this documentation, it is important to know that an
additional component while creating a DSLink is a configuration file called
<code>dslink.json</code>. It is also suggested you also
<a href="https://github.com/IOT-DSA/docs/wiki/dslink.json">review the
documentation</a> from our Wiki on the content of a <code>dslink.json</code>
file.
</aside>

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
// For Gradle add this to your build.gradle dependencies.
compile 'org.iot-dsa:dslink:0.12.0'

// For Apache Maven add this to your build.xml dependencies.
<dependency>
    <groupId>org.iot-dsa</groupId>
    <artifactId>dslink</artifactId>
    <version>0.12.0</version>
</dependency>
```

```ruby
# Add to your Gemfile
gem 'dslink', :git => 'git://github.com/IOT_DSA/sdk-dslink-ruby.git'

#link.rb
require 'dslink'
```

```plaintext
// package.json
"dependencies": {
  "dslink": "IOT-DSA/sdk-dslink-javascript#artifacts"
}

// index.js
var DS = require('dslink');
```

```javascript
// index.html
<script src="https://raw.githubusercontent.com/IOT-DSA/sdk-dslink-javascript/artifacts/dist/dslink.browser.min.js"></script>

// DS is a globally accessed variable
```

```python
# Add to your setup.py
setup(
    install_requires=[
        "dslink"
    ]
)

# dslink.py
import dslink
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

```plaintext
var link = new DS.LinkProvider(process.argv.slice(2), 'Example-');
link.connect();
```

```js
var link = new DS.LinkProvider('http://127.0.0.1:8080/conn', 'Example-');
link.connect();
```

```java
// package ...
// import ...

public class Main extends DSLinkHandler {

    private static final Logger LOGGER = LoggerFactory.getLogger(Main.class);

    @Override
    public boolean isResponder() {
        return true;
    }

    @Override
    public void onResponderInitialized(final DSLink link) {
        LOGGER.info("Initialized");
    }

    @Override
    public void onResponderConnected(final DSLink link) {
        LOGGER.info("Connected");
    }

    public static void main(String[] args) {
        DSLinkFactory.start(args, new Main());
    }
}
```

```python
# dslink.py
import dslink

class ExampleDSLink(dslink.DSLink):
    def start(self):
        print("Starting DSLink")

if __name__ == "__main__":
    ExampleDSLink(dslink.Configuration("example", responder=True))
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

```plaintext
var myNum = Math.round(Math.random() * 50);
```

```js
var myNum = Math.round(Math.random() * 50);
```

```java
// Add this variable into your Main class.
private static final Random RANDOM = new Random();

// Generating a random number is as follows.
int num = RANDOM.nextInt(50);
```

```python
# Import the random library at the top of the file
import random

var my_num = random.randint(0, 50)
```

Before we add the new, let's create a value that we can pass, and send updates
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

```plaintext
var myNode = link.addNode('/MyNum', {
  '$name': 'My Number',
  '$type': 'int',
  '?value': myNum
});
```

```js
var myNode = link.addNode('/MyNum', {
  '$name': 'My Number',
  '$type': 'int',
  '?value': myNum
});
```

```java
// In your Main class
@Override
public void onResponderInitialized(final DSLink link) {
    Node superRoot = link.getNodeManager().getSuperRoot();
    NodeBuilder builder = superRoot.createChild("MyNum");
    builder.setDisplayName("My Number");
    builder.setValueType(ValueType.NUMBER);
    builder.setValue(new Value(0));
    final Node node = builder.build();

    // ...
}
```

```python
# In the DSLink class, implement get_default_nodes
def get_default_nodes(self):
    # Get default super_root
    root = self.get_root_node()

    # Create MyNum Node
    my_node = dslink.Node("MyNum", root) # Name the node, specify the parent
    my_node.set_display_name("My Number")
    my_node.set_type("int")
    my_node.set_value(my_num)

    # Add my_node to the super_root
    root.add_child(my_node)

    # Return the super root
    return root
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
	myNum = numGen.nextInt(50);
	myNode.updateValue(myNum);
});
```

```java
// In your Main class
@Override
public void onResponderInitialized(final DSLink link) {
    // ...

    Objects.getDaemonThreadPool().scheduleWithFixedDelay(new Runnable() {
        @Override
        public void run() {
            int num = RANDOM.nextInt(50);
            Value val = new Value(num);
            node.setValue(val);
        }
    }, 0, 5, TimeUnit.SECONDS);

    // ...
}
```

```python
def update(self):
    reactor.callLater(1, self.update) # Call again 1 second later
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

```java
// scheduleWithFixedDelay ...
@Override
public void run() {
    if (link.getSubscriptionManager().hasValueSub(node)) {
        int num = RANDOM.nextInt(50);
        Value val = new Value(num);
        node.setValue(val);
    }
}
```

```python
if my_node.is_subscribed():
    print("Node is subscribed")
```

In our simple example, we only want to update a value if there is someone
actively listening, or *subscribing*, to our node. We can check a node
specifically to see if it has a subscriber.

A value, however, should be set with at least an initial value, as once the value
is updated, it is possible that a subscription to the node may occur at any time.

<aside class="notice">
In many situations it is preferable to have the latest value always ready rather
than a value which may be hours, or even days, old when the the node receives
its next subscription request.
</aside>

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

```java
// In your Main class
@Override
public void onResponderInitialized(final DSLink link) {
    // Node superRoot = ...
    // NodeBuilder builder = ...
    // ...
    // final Node node = ...
    // ...

    builder = superRoot.createChild("UpdateNum");
    builder.setSerializable(false);
    builder.setDisplayName("Update Number");
    builder.setAction(new Action(Permission.WRITE,
                                  new Handler<ActionResult>() {
        @Override
        public void handle(ActionResult event) {
            int num = RANDOM.nextInt(50);
            Value val = new Value(num);
            node.setValue(val);
        }
    }));
    builder.build();

    // ...
}
```

```python
def start(self):
    self.profile_manager.create_profile("addnum")
    self.profile_manager.register_callback("addnum", self.addnum)

def get_default_nodes(self):
    root = self.get_root_node()

    my_num = dslink.Node("MyNum", root)
    my_num.set_display_name("My Number")
    my_num.set_type("int")
    my_num.set_value(0)

    add_num = dslink.Node("AddNum", root)
    add_num.set_display_name("Add number")
    add_num.set_profile("addnum")
    add_num.set_invokable("write")
    add_num.set_parameters([
        {
            "name": "Number",
            "type": "int"
        }
    ])

    root.add_child(my_num)
    root.add_child(add_num)

    return root

def addnum(self, parameters):
    num = int(parameters.params["Number"]) # Parse number
    self.super_root.get("/MyNum").set_value(num) # Set value
    return [[]] # Return empty columns
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

```java
// All the nodes are automatically reinitialized again on startup.
```

```python
# The Python SDK initializes itself when implementing the class.
```

> Call save on the link.

```dart
// Add outside of the for loop inside the Timer call.
link.save();
```

```java
// All the nodes are automatically saved.
```

```python
# The Python SDK saves state automatically.
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

```java
// Retrieves the super root of the responder.
Node superRoot = link.getNodeManager().getSuperRoot();
```

```python
root_node = self.super_root
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

```java
// Iterate all the children of the super root.
for (Node child : superRoot.getChildren().values()) {
    // ...
}
```

```python
for child_name in root_node.children:
    child = root_node.children[child_name]
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

```java
@Override
public void onResponderInitialized(final DSLink link) {
    Node superRoot = link.getNodeManager().getSuperRoot();
    NodeBuilder builder = superRoot.createChild("Numbers");
    builder.setSerializable(false);
    final Node numbers = builder.build();

    builder = numbers.createChild("MyNum");
    builder.setDisplayName("My Number");
    builder.setValueType(ValueType.NUMBER);
    builder.setValue(new Value(0));
    final Node node = builder.build();
    Objects.getDaemonThreadPool().scheduleWithFixedDelay(new Runnable() {
        @Override
        public void run() {
            if (link.getSubscriptionManager().hasValueSub(node)) {
                int num = RANDOM.nextInt(50);
                Value val = new Value(num);
                node.setValue(val);
            }
        }
    }, 0, 1, TimeUnit.SECONDS);
}
```

```python
my_num = dslink.Node("MyNum", root)

add_num = dslink.Node("AddNum", root)

my_num.add_child(add_num)
root.add_child(my_num)
```

> Update location our action adds to.

```dart
// Within our addnum profile update where we add the
// new node when the action runs.
link.addNode('/CustomNumbers/MyNum$ndNum', {
// ... rest of the code is the same.
```

```java
@Override
public void onResponderInitialized(final DSLink link) {
    // final Node numbers = ...
    // final Node node = ...
    // ...

    builder = numbers.createChild("UpdateNum");
    builder.setSerializable(false);
    builder.setDisplayName("Update Number");
    builder.setAction(new Action(Permission.WRITE,
                                  new Handler<ActionResult>() {
        @Override
        public void handle(ActionResult event) {
            int num = RANDOM.nextInt(50);
            Value val = new Value(num);
            node.setValue(val);
        }
    }));
    builder.build();

    LOGGER.info("Initialized");
}
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
Node paths should be alphanumeric only with spaces or underscores. Node paths
cannot include a number of special characters.
</aside>
