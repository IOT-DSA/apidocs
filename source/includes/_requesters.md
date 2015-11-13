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

With the requester, we can now query a node from the broker and get a listing
of the nodes and recursively poll their children. For this sample we are going
to query the broker for the nodes of the Responder link we created previously.

<aside class="warning">
Ensure that your Responder link is connected to your broker before trying
out this code.
</aside>

You'll notice that we query for the node with each child, rather than
accessing it from the parent node. This is due to the fact that when a
remote node is returned, the content it contains is limited to prevent
large amounts of data from being transferred when only a small subset may
be required. Part of the restricted content is the full child node, and its
own children. Also included in the restricted content is the value of a node,
as depending on the node, the value may be anything from a single integer to
a fully populated table, to even a file on the system.

<!-- TODO: Aside mentioning that you can only subscribe to a value not to
    an entire node with subnodes. -->
