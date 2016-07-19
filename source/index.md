---
title: API Reference

language_tabs:
  - dart: Dart
  - java: Java
  - javascript: JavaScript
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

```scala
// For Gradle add this to your build.gradle dependencies.
compile 'org.iot-dsa:dslink:0.12.0'

// For Apache Maven add this to your build.xml dependencies.
<dependency>
    <groupId>org.iot-dsa</groupId>
    <artifactId>dslink</artifactId>
    <version>0.12.0</version>
</dependency>

// For SBT add this to your build.sbt libraryDependencies:
libraryDependencies ++= Seq(
  "org.iot-dsa" % "dslink" % "0.12.0"
)
```

```ruby
# Add to your Gemfile
gem 'dslink', :git => 'git://github.com/IOT_DSA/sdk-dslink-ruby.git'

#link.rb
require 'dslink'
```

```javascript
// BROWSER
// index.html
<script src="https://raw.githubusercontent.com/IOT-DSA/sdk-dslink-javascript/artifacts/dist/dslink.browser.min.js"></script>

// DS is a globally accessed variable

// NODE
// package.json
"dependencies": {
  "dslink": "IOT-DSA/sdk-dslink-javascript#artifacts"
}

// index.js
var DS = require('dslink');
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

```csharp
Clone the SDK's source code, and make a release build with Visual Studio or MonoDevelop. The SDK will be pushed to
the NuGet repositories soon.
```

```c
# For CMake build configurations
include(ExternalProject)

ExternalProject_Add(dslink
        PREFIX dslink
        GIT_REPOSITORY "https://github.com/IOT-DSA/sdk-dslink-c.git"
        CMAKE_ARGS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
        -DCMAKE_C_FLAGS=${CMAKE_C_FLAGS}
        -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/dslink
)

set(DSLINK_DIR "${CMAKE_CURRENT_BINARY_DIR}/dslink")
include_directories("${DSLINK_DIR}/include")
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
  // Server-side:
  var link = new LinkProvider(args, 'Example-');
  // Client-side
  var link =
    new LinkProvider('http://localhost:8080/conn', 'Example-');
  link.connect();
  // ...
}
```

```javascript
// node.js
var link = new DS.LinkProvider(process.argv.slice(2), 'Example-');
// browser
var link =
  new DS.LinkProvider('http://127.0.0.1:8080/conn', 'Example-');

link.connect();
```

```java
// package ...
// import ...

public class Main extends DSLinkHandler {

    private static final Logger LOGGER =
                    LoggerFactory.getLogger(Main.class);

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

```scala
// package ...
// import ...

object Main extends DSLinkHandler {
  private val log = LoggerFactory.getLogger(getClass)
  override val isResponder = true

  override def onResponderInitialized(link: DSLink) = {
    log.info("Responder initialized")
  }

  override def onResponderConnected(link: DSLink) = {
    log.info("Responder connected")
  }

  def main(args: Array[String]): Unit = {
    DSLinkFactory.start(args, Main)
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

```csharp
using System;
using DSLink;

class ExampleDSLink : DSLinkContainer {
    public ExampleDSLink() : base(new Configuration("example", responder: true, brokerUrl: "http://broker.url/conn")){
        Console.WriteLine("Starting DSLink...");
    }

    private static void Main() {
        new ExampleDSLink();
        Console.ReadLine(); // Keep the link running.
    }
}
```

```c
#include <dslink/dslink.h>

void init(DSLink *link) {
    // ...
}

void connected(DSLink *link) {
    // ...
}

void disconnected(DSLink *link) {
    // ...
}

int main(int argc, char **argv) {
    // Any callback can be NULL
    DSLinkCallbacks cbs = {
        init,
        connected,
        disconnected
    };

    return dslink_init(argc, argv, "C-Resp", 0, 1, &cbs);
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

```js
var myNum = Math.round(Math.random() * 50);
```

```java
// Add this variable into your Main class.
private static final Random RANDOM = new Random();

// Generating a random number is as follows.
int num = RANDOM.nextInt(50);
```

```scala
import scala.util.Random

// Generating a random number is as follows.
val num = Random.nextInt(50)
```

```python
# Import the random library at the top of the file
import random

my_num = random.randint(0, 50)
```

```csharp
using System;

Random random = new Random();
random.Next(0, 50);
```

```c
#include <stdlib.h>

int x = rand();
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

```scala
// In your Main class
override def onResponderInitialized(link: DSLink) = {
  val superRoot = link.getNodeManager.getSuperRoot
  val builder = superRoot.
    createChild("MyNum").
    setDisplayName("My Number").
    setValueType(ValueType.NUMBER).
    setValue(new Value(0))
  val node = builder.build

  //...
}
```

```python
# In the DSLink class, implement get_default_nodes.
def get_default_nodes(self, super_root):
    # Create MyNum Node
    # Name the node, specify the parent
    my_node = dslink.Node("MyNum", super_root)
    my_node.set_display_name("My Number")
    my_node.set_type("int")
    my_node.set_value(my_num)

    # Add my_node to the super root
    super_root.add_child(my_node)

    # Return the super root
    return super_root
```

```csharp
var myNumNode = Responder.SuperRoot.CreateChild("MyNum")
    .SetType("int")
    .SetValue(myNum)
    .BuildNode();
```

```c
#define LOG_TAG "main"
#include <dslink/log.h>

// Inside your init handler
DSNode *rng = dslink_node_create(root, "rng", "node");
if (!rng) {
    log_warn("Failed to create the rng node\n");
    return;
}

if (dslink_node_set_meta(num, "$type", json_string("number")) != 0) {
    log_warn("Failed to set the type on the rng\n");
    dslink_node_tree_free(link, rng);
    return;
}

if (dslink_node_set_value(link, rng, json_integer(0)) != 0) {
    log_warn("Failed to set the value on the rng\n");
    dslink_node_tree_free(link, rng);
    return;
}

if (dslink_node_add_child(link, rng) != 0) {
    log_warn("Failed to add the rng node to the root\n");
    dslink_node_tree_free(link, rng);
}
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

```js
setInterval(function() {
  myNum = Math.round(Math.random() * 50);
  myNode.updateValue(myNum);
}, 1000 * 5);
```

```java
// In your Main class
@Override
public void onResponderInitialized(final DSLink link) {
    // ...

    Objects.getDaemonThreadPool()
        .scheduleWithFixedDelay(new Runnable() {
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

```scala
override def onResponderInitialized(link: DSLink) = {
    Objects.getDaemonThreadPool
        .scheduleWithFixedDelay(new Runnable {
          def run = {
          val num = Random.nextInt(50)
          node.setValue(new Value(num))
        }
    }, 0, 5, TimeUnit.SECONDS)
    //...
}
```

```python
def start(self):
    # Kick off initial loop.
    self.call_later(1, self.update)

def update(self):
    num = random.randint(0, 50)
    self.responder.get_super_root().get("/MyNum").set_value(num)

    # Call again 1 second later.
    self.call_later(1, self.update)
```

```csharp
using System.Threading;

var timer = new Timer(i =>
{
    myNumNode.Value.Set(random.Next(0, 50));
}, null, 1, 1000);
```

```c
static
void gen_number(void *data, EventLoop *loop) {
    DSLink *link = ((void **) data)[0];
    DSNode *node = ((void **) data)[1];
    int x = rand();

    dslink_node_set_value(link, node, json_integer(x));
    dslink_event_loop_schedd(loop, gen_number, data, 1000);
}

// init handler
// ...
void **a = malloc(sizeof(void *) * 2);
a[0] = link;
a[1] = rng;
dslink_event_loop_schedd(&link->loop, gen_number, a, 1000);
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

```js
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

```scala
// scheduleWithFixedDelay ...
def run = {
  if (link.getSubscriptionManager.hasValueSub(node)) {
    val num = Random.nextInt(50)
    node.setValue(new Value(num))
    }
}
```

```python
my_node = self.responder.get_super_root().get("/MyNum")
if my_node.is_subscribed():
    my_node.set_value(random.randint(0, 50))
```

```csharp
if (myNumNode.Subscribed) {
    myNumNode.Value.Set(random.Next(0, 50));
}
```

```c
if (dslink_map_contains(link->responder->value_path_subs,
                         (void *) node->path)) {
    int x = rand();
    dslink_node_set_value(link, node, json_integer(x));
}
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

```js
// Our new constructor in which we pass both our default node for
// adding a value, and our profile, or action on that node.
// Store all nodes we create to reference later.
var nodeList = [];

// Following replaces previous link initialization.
// Again, for node.js, replace the URL with process.argv.slice(2)
link = new DS.LinkProvider('http://127.0.0.1:8080/conn',
  'Example-', {
    defaultNodes: {
      AddNum: {
        $name: 'Add number',
        $is: 'addnum',
        $invokable: 'write',
        $params: [{ name: 'value', type: 'int' }]
      }
    },
    profiles: {
      addnum: function(path) {
        return new DS.SimpleActionNode(path, function(params) {
          var addVal = parseInt(params['value'], 10);
          var ndNum = nodeList.length;
          nodeList.push(link.addNode('/MyNum' + ndNum, {
            $name: 'My node #' + ndNum,
            $type: 'int',
            '?value': addVal
          }));
          // myNode.updateValue(myNum + addVal);
        });
    }
  }
});
```

```js
// Add our initial node to our node list instead of a
// single variable
nodeList.push(link.addNode('/MyNum',
  { $name: 'My Number',
    $type: 'int',
    '?value': myNum})
);

// Update our timer to provide a new random number for
// each node not just the initial one.
setInterval(function() {
  nodeList.forEach(function(myNode) {
    if(myNode.value == null) return;
    if(myNode.hasSubscriber) {
      myNum = Math.round(Math.random() * 50);
      myNode.updateValue(myNum);
    }
  });
}, 5 * 1000);
```

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

// Update our timer to provide a new random number for each node
// not just the initial one.
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

```scala
override def onResponderInitialized(link: DSLink) = {
  val superRoot = link.getNodeManager.getSuperRoot
    val builder = superRoot.createChild("UpdateNum").
      setSerializable(false).
      setDisplayName("Update Number").
      setAction(new Action(Permission.WRITE,
        new Handler[ActionResult] {
          def handle(event: ActionResult) = {
            val num = Random.nextInt(50)
            node.setValue(new Value(num))
          }
      }))
    builder.build
    // ...
}
```

```python
def start(self):
    self.responder.profile_manager.create_profile("addnum")
    self.responder.profile_manager.register_callback("addnum", self.addnum)

def get_default_nodes(self, super_root):
    my_num = dslink.Node("MyNum", super_root)
    my_num.set_display_name("My Number")
    my_num.set_type("int")
    my_num.set_value(0)

    add_num = dslink.Node("AddNum", super_root)
    add_num.set_display_name("Add number")
    add_num.set_profile("addnum")
    add_num.set_invokable("write")
    add_num.set_parameters([
        {
            "name": "Number",
            "type": "int"
        }
    ])

    super_root.add_child(my_num)
    super_root.add_child(add_num)

    return super_root

def addnum(self, parameters):
    num = int(parameters.params["Number"]) # Parse number
    self.responder.get_super_root().get("/MyNum").set_value(num) # Set value
    return [[]] # Return empty columns
```

```csharp
public ExampleDSLink() : base(/* Configuration */) {
    var myNum = Responder.SuperRoot.CreateChild("MyNum")
        .SetDisplayName("My Number")
        .SetType("int")
        .SetValue(0)
        .BuildNode();

    var addNum = Responder.SuperRoot.CreateChild("SetNum")
        .SetDisplayName("SetNumber")
        .AddParameter(new Parameter("Number", "int"))
        .SetAction(new Action(Permission.Write, parameters =>
        {
            myNum.Value.Set(parameters["Number"].Get());
            return new List<dynamic>();
        }))
        .BuildNode();
}
```

```c
void update_num(DSLink *link, DSNode *node,
                json_t *rid, json_t *params) {
    // Close the invocation stream
    json_t *top = json_object();
    if (!top) {
        return;
    }
    json_t *resps = json_array();
    if (!resps) {
        json_delete(top);
        return;
    }
    json_object_set_new_nocheck(top, "responses", resps);

    json_t *resp = json_object();
    if (!resp) {
        json_delete(top);
        return;
    }
    json_array_append_new(resps, resp);

    json_object_set_nocheck(resp, "rid", rid);
    json_object_set_new_nocheck(resp, "stream", json_string("closed"));
    dslink_ws_send_obj(link->_ws, top);
    json_delete(top);

    // Generate a new value
    node = node->parent;
    int x = rand();
    dslink_node_set_value(link, node, json_integer(x));
}

// In your init handler
// Warning: there isn't any error handling in this code snippet!
DSNode *update = dslink_node_create(rng, "UpdateNum", "node");
update->on_invocation = update_num;
dslink_node_set_meta(update, "$name", json_string("Update Number"));
dslink_node_set_meta(update, "$invokable", json_string("read"));
dslink_node_add_child(link, update);
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
// after the link = new LinkProvider(..)
// and before link.connect();
link.init();
```


```js
// after the link = new LinkProvider(..)
// and before link.connect();
link.init();
```

```java
// All the nodes are automatically reinitialized again on startup.
```

```scala
// All the nodes are automatically reinitialized again on startup.
```

```python
# The Python SDK initializes itself when implementing the class.
```

```csharp
// Will be filled in
```

> Call save on the link.

```dart
// Add outside of the for loop inside the Timer call.
link.save();
```

```js
// Add outside of the for loop inside setInterval.
link.save();
```

```java
// All the nodes are automatically saved.
```

```python
# The Python SDK saves state automatically.
```

```csharp
// Will be filled in
```

```c
// Will be filled in
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

```js
// At the top of our setInterval callback.
var rootNode = link.getNode('/');
```

```java
// Retrieves the super root of the responder.
Node superRoot = link.getNodeManager().getSuperRoot();
```

```scala
import scala.collection.JavaConverters._

// Retrieves the super root of the responder.
val superRoot = link.getNodeManager.getSuperRoot

// Iterate all the children of the super root.
superRoot.getChildren.values.asScala foreach { child =>
  // ...
}
```

```python
root_node = self.responder.get_super_root()
```

```csharp
var node = Responder.SuperRoot;
```

```c
DSNode *node = link->responder->super_root;
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

```js
/// rootNode.children returns an object where the key
/// is the node identifier and the value is the node itself.
Object.keys(rootNode.children).forEach(function(nodeName) {
  var myNode = rootNode.children[nodeName];
  // ... Existing checking for subscriber and updating value
})
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

```csharp
foreach (var pair in Responder.SuperRoot.Children) {
    // pair.Key is child name
    // pair.Value is child Node
}
```

```c
Map *children = root->children;
if (children) {
    for (MapEntry *entry = (MapEntry *) children->list.head;
         entry != NULL; entry = entry->next) {
        DSNode *node = entry->value;
    }
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

```js
/// Change the default nodes from our constructor.
/// The rest of the constructor stays the same.
defaultNodes: {
  CustomNumbers: {
    $name: 'Custom Numbers',
    AddNum: {
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
    Objects.getDaemonThreadPool().scheduleWithFixedDelay(
        new Runnable() {
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

```scala
override def onResponderInitialized(link: DSLink) = {
    val superRoot = link.getNodeManager().getSuperRoot()
    var builder =
        superRoot.createChild("Numbers").setSerializable(false)
    val numbers = builder.build

    builder = numbers.createChild("MyNum").
      setDisplayName("My Number").
      setValueType(ValueType.NUMBER).
      setValue(new Value(0))
    val node = builder.build
    Objects.getDaemonThreadPool
      .scheduleWithFixedDelay(new Runnable {
        def run = {
          if (link.getSubscriptionManager.hasValueSub(node)) {
            val num = Random.nextInt(50)
            node.setValue(new Value(num))
          }
      }
    }, 0, 1, TimeUnit.SECONDS)
}
```

```python
my_num = dslink.Node("MyNum", super_root)
# Other Node information...

add_num = dslink.Node("AddNum", my_num) # Change the parent to my_num
# Other Node information...

my_num.add_child(add_num)
super_root.add_child(my_num)
```

```csharp
var addNum = myNum.CreateChild("AddNum")
    // other factory information.
```

```c
DSNode *numbers = dslink_node_create(root, "numbers", "node");
if (!numbers) {
    log_warn("Failed to create the numbers node\n");
    return;
}

if (dslink_node_set_value(link, numbers, json_integer(0)) != 0) {
    log_warn("Failed to set the value on the numbers\n");
    dslink_node_tree_free(link, numbers);
    return;
}

if (dslink_node_add_child(link, numbers) != 0) {
    log_warn("Failed to add the numbers node to the root\n");
    dslink_node_tree_free(link, numbers);
}

// Add RNG here with numbers as the parent
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

```scala
override def onResponderInitialized(link: DSLink) = {
    // val numbers = ...
    // val node = ...
    // ...

    val builder = numbers.createChild("UpdateNum").
      setSerializable(false).
      setDisplayName("Update Number").
      setAction(new Action(Permission.WRITE,
        new Handler[ActionResult] {
          def handle(event: ActionResult) = {
            val num = Random.nextInt(50)
            node.setValue(new Value(num))
          }
      }))
    builder.build

    log.info("Initialized")
}
```

```csharp
// Done automatically.
```

```c
DSNode *rng = //...
// Add action to rng node as previously shown.
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
