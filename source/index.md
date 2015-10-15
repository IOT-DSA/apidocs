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
  - errors

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

In order to create any type of connection (Responder or Requester), we first
need to establish a link to the DSA Broker. In this example we will only
establish the bare minimum link. There are a number of configuration options
to further extend the capabilities. Refer to the documentation for the SDK
of the language of your choice.

```dart
// run.dart

main(List<String> args) {
  var link = new LinkProvider(args, 'Example-');
  link.connect();
  // ...
}
```

## Nodes

Within the DSA Broker, all of our data is broken into a collection of nodes.
A node may be a link to a separate device, or it may be multiple links running
from a server. Each link may also have children nodes.
