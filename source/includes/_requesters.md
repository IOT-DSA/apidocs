# Requesters

While a `Responder` provides information on-demand, a `Requester` will
provide data when it needs to. This could be by Requesting information
that it needs, or by pushing data to an existing value. We'll look at how
to view currently available nodes and `subscribe` to a specific one.

<aside class="notice">
It is important to note, that while a requester may subscribe to a
specific node, the Requester is actually subscribing to the Broker for
that data, not to the node directly. The Broker will receive updates and
pass them along to as many Requesters. Any given responder will only have
the connection to the Broker not multiple Requesters.
</aside>

## Link

> Initialize a Requester only link

```dart
var link = new LinkProvider(args, 'RequesterExample-',
               isRequester: true, isResponder: false);
await link.connect();
var requester = link.requester;
```

```java
public class MyLink extends DSLinkHandler {
    @Override
    public boolean isRequester() {
        return true;
    }

    public static void main(String[] args) {
        DSLinkFactory.start(args, new MyLink());
    }
}
```

```js
// node.js
// var link = new DS.LinkProvider(process.argv.slice(2),
var link = new DS.LinkProvider('http://127.0.0.1:8080/conn',
          'RequesterExample-', {
            isRequester: true,
            isResponder: false
          });

link.connect().then(function() {
  return link.onRequesterReady;
}).then(function(requester) {
   // work with requester here  
});
```

```scala
object Main extends DSLinkHandler {

  private val log = LoggerFactory.getLogger(getClass)

  override def isRequester = true

  override def onRequesterInitialized(link: DSLink) = {
    log.info("Requester initialized")
  }

  override def onRequesterConnected(link: DSLink) = {
    log.info("Requester connected")
  }

  def main(args: Array[String]): Unit = {
    DSLinkFactory.start(args, Main)
  }
}
```

```python
class RequesterDSLink(dslink.DSLink):
    def start(self):
        # self.requester is the Requester class.
        pass

if __name__ == "__main__":
    RequesterDSLink(dslink.Configuration("RequesterDSLink", requester=True, responder=False))
```

```csharp
using System;
using DSLink;

class ExampleDSLink : DSLinkContainer {
    public ExampleDSLink() : base(new Configuration("example", requester: true, brokerUrl: "http://broker.url/conn")){
        Console.WriteLine("Starting DSLink...");
    }

    private static void Main() {
        new ExampleDSLink();
        Console.ReadLine(); // Keep the link running.
    }
}
```

Each link can be a Responder or a Requester (or both). At this time we're
only interested in creating a requester so we initialize our link specifically
for the requester.

After the link connects, we can the access the requester directly in order to
query the broker for nodes.



## Query Nodes

> Recursively poll nodes and children.

```dart
iterateChildren(RemoteNode nd) async {
  print('Node: ${nd.remotePath}');
  if (nd.children.isNotEmpty) {
    for (var nodeName in nd.children.keys) {
      var path = nd.remotePath + '/$nodeName';
      iterateChildren(await requester.getRemoteNode(path));
    }
  }
}
// Query initial node and pass it to our recursive function.
var exampleNode =
      await requester.getRemoteNode('/downstream/Example');
iterateChildren(exampleNode);
```

```java
// This is not available right now for the Java SDK.
```

```scala
def iterateChildren(node: Node): Unit = {
  println(s"Node: ${node.getPath}")
  val children = Option(node.getChildren) map (_.asScala) getOrElse (Map.empty)
  children foreach {
    case (name, child) => // ...
      iterateChildren(child)
  }
}
```

```js
function iterateChildren(nd) {
  console.log('Node: ' + nd.remotePath);
  var keys = Object.keys(nd.children);
  if (keys.length > 0) {
    keys.forEach(function(nodeName) {
      var path = nd.remotePath + '/' + nodeName;
      requester.getRemoteNode(path).then(function(node) {
        iterateChildren(node);
      });
    });
  }
}

// Query initial node and pass it to our recursive function.
requester.getRemoteNode('/downstream/Example')
  .then(function(exampleNode) {
    iterateChildren(exampleNode);  
  });
```

```python
def recurse(self, listresponse):
    for child_name in listresponse.node.children:
        child = listresponse.node.children[child_name]
        print(child.path)
        self.requester.list(child.path, self.recurse)

self.requester.list("/", self.recurse)
```

```csharp
public void Recurse(ListResponse response)
{
    foreach (var kp in response.Node)
    {
        Console.WriteLine(kp.Key);
        Requester.List(kp.Value.Path, Recurse);
    }
}

Requester.List("/", Recurse);
```

With the requester, we can now query a node from the broker and get a listing
of the nodes and recursively poll their children. For this sample we are going
to query the broker for the nodes of the Responder link we created previously.

<aside class="warning">
Ensure that your Responder link is connected to your broker before trying
out this code.
</aside>

You'll notice that we query for each child node, rather than accessing it from
the parent node. This is due to the fact that when a remote node is returned,
the content it contains is limited to prevent large amounts of data from being
transferred when only a small subset may be required. Part of the restricted
content is the full child node, and its own children. Also included in the
restricted content is the value of a node, as depending on the node, the value
may be anything from a single integer to a fully populated table, to even a
file on the system.

## List Nodes

```dart
// Replace print('Node: ${nd.remotePath}'); in
// iterateChildren with:
requester.list(nd.remotePath).listen(listUpdates);

// A new function outside of main
void listUpdates(RequesterListUpdate update) {
  print('Node - ${update.node.name}');
  print('\tChanges: ${update.changes}');
}
```

```scala
val request = ListRequest("...path...")
requester.list(request, new Handler[ListResponse] {
  def handle(event: ListResponse) = {
    //...
  }
})
```

```js
// Replace console.log('Node: ' + nd.remotePath) in
// iterateChildren with:
requester.list(nd.remotePath).on("data", listUpdates);

// a new function that is globally defined
function listUpdates(update) {
  console.log('Node - ' + update.node.name);
  console.log('\tChanges: ' + update.changes);
}
```

```python
def list(self, listresponse):
    print(listresponse.node.name)
    for child in listresponse.node.children:
        print(child)

self.requester.list("/path/to/node", self.list)
```

```csharp
public void List(ListResponse response)
{
    Console.WriteLine(response.Node.Name);
}

Requester.List("/", List);
```

```java
// Initial Request
link.getRequester().list(new ListRequest("/downstream/Example"), new Handler<ListResponse>() {
    @Override
    public void handle(ListResponse event) {
        iterateChildren(event.getNode());
    }
});

public void iterateChildren(Node node) {
    System.out.println("Node: " + node.getPath());
    Map<String, Node> children = node.getChildren();
    if (children == null) {
        return;
    }
    for (Node child : children.values()) {
        iterateChildren(child);
    }
}
```

Iterating over children is great when we initialize our connection, however
it won't report any changes to those nodes, such as if a new child node is
added, removed or modified. The list method on the Requester takes the full
path of the node, and will call a method every time a change occurs to the
node which is passed to it.

The `list` method will provide an update object which contains the node which
was changed, and the configuration, attributes or children which were changed.
It will only provide the updates of the specific node, and not any children
of that node, so we need to ensure we call it on each node we want to keep
updated on.

With the Responder and Requester links running, try adding a new node to our
custom numbers and observe the output from the Requester link we've created.

## Subscribing to Nodes

```dart
// Replace requester.list(nd.remotePath).listen(listUpdates);
// in iterate children with:
if (nd.getConfig(r'$type') == 'int') {
  requester.subscribe(nd.remotePath,
    (ValueUpdate update) => subscribeUpdates(update, nd.name));
}

// A new function outside of main.
void subscribeUpdates(ValueUpdate update, String name) {
  print('Name: $name ValueUpdate: ${update.value}');
}
```

```java
link.getRequester().subscribe("/downstream/Example/someValue", new Handler<SubscriptionValue>() {
    @Override
    public void handle(SubscriptionValue event) {
        System.out.println("Value updated to: " + event.getValue());
    }
});
```

```scala
requester.subscribe("...path...", new Handler[SubscriptionValue] {
  def handle(event: SubscriptionValue) = {
    println(s"Update received: path=${event.getPath}, value=${event.getValue}")
  }
})
```

```js
// Replace requester.list(nd.remotePath).on("data", listUpdates)
// in iterateChildren with:
if (nd.getConfig('$type') === 'int') {
  requester.subscribe(nd.remotePath,
    function(update) { subscribeUpdates(update, nd.name); });
}

// a new function that is globally defined
function subscribeUpdates(update, name) {
  console.log('Name: ' + name + ' ValueUpdate: ' + update.value);
}
```

```python
# data is (value, updated_time)
def value_updates(self, data):
    print("Value updated to %s at %s" % (data[0], data[1]))

self.requester.subscribe("/path/to/node", self.value_updates)
```

```csharp
public void SubscriptionUpdate(SubscriptionUpdate update)
{
    Console.WriteLine(update.Value);
}

Requester.Subscribe("/path/to/node", SubscriptionUpdate);
```

When listing nodes, we see the changes in a node's configuration including
any new children. However we do not receive the actual values when they're
updated by the Responder. To get the change of a values, and only the change
in that value, we need to `subscribe` to that node.

Using `subscribe` will activate the `hasSubscriber` flag on a node. It will
also receive any updates to that node. In the sample we also verify that the
node we're looking has a `$type` value of `int`.

<aside class="notice">
You can only subscribe to a single value. You <strong>cannot</strong> subscribe
to a parent node and receive all child updates.
</aside>
