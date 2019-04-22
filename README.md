# streaming_shared_preferences

StreamingSharedPreferences is a reactive, `Stream` based wrapper over the [shared_preferences plugin]().

On top of just storing key-value pairs, `StreamingSharedPreferences` allows you to **observe changes in values**.
And best of all, it's pure `Streams` - **no rxdart needed**.

## Simple usage example

To get a hold of `StreamingSharedPreferences`, _await_ on `instance`:

```dart
import 'package:streaming_shared_preferences/streaming_shared_preferences.dart';

...
final preferences = await StreamingSharedPreferences.instance;
```

Then call any of the `getXYZ()` public methods.
The public API follows the same naming convention as `shared_preferences` does, but with a little
twist - every getter returns a `Preference` object, which is a `Stream`!

For example, here's how you would get and listen to changes in an `int` with the key "counter":

```dart
// Provide a default value of 0 in case "counter" is null.
final counter = preferences.getInt('counter', defaultsTo: 0);

// "counter" is a Stream - it can do anything a Stream can!
counter.listen((value) {
  print(value);
});

// Same as preferences.setInt('counter', <value>), but no need
// to provide a key here.
counter.set(1);
counter.set(2);
counter.set(3);

// Obtain current value synchronously by calling the ".value()"
// method. In this case, "currentValue" is now 3.
final currentValue = counter.value();
```

Assuming that there's no previously stored value for `counter`, the above example will print `0`,
`1`, `2` and `3` to the console.

## Using StreamingSharedPreferences with the StreamBuilder widget

Here's the standard counter app that you get when creating a Flutter project, but with a twist.

The difference to the regular one is that the value of `counter` is **persisted locally**.
This means that the state will not get lost between app restarts.

```dart
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:streaming_shared_preferences/streaming_shared_preferences.dart';

Future<void> main() async {
  final preferences = await StreamingSharedPreferences.instance;
  runApp(MyApp(preferences));
}

class MyApp extends StatelessWidget {
  MyApp(this.preferences);
  final StreamingSharedPreferences preferences;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(preferences),
    );
  }
}

class MyHomePage extends StatefulWidget {
  MyHomePage(this.preferences);
  final StreamingSharedPreferences preferences;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  Preference<int> _counter;

  @override
  void initState() {
    super.initState();
    _counter = widget.preferences.getInt('counter', defaultsTo: 0);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Streaming SharedPreferences'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'You have pushed the button this many times:',
            ),
            StreamBuilder<int>(
              stream: _counter,
              initialData: 0,
              builder: (context, snapshot) {
                return Text(
                  '${snapshot.data}',
                  style: Theme.of(context).textTheme.display1,
                );
              },
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          final currentValue = _counter.value();
          _counter.set(currentValue + 1);
        },
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ),
    );
  }
}
```

There's one thing worth noting here: **do NOT** pass `preferences.getInt(..)` to a `StreamBuilder`
directly. Cache it instead.

To illustrate, **DO**:

```dart
class _MyHomePageState extends State<MyHomePage> {
  Preference<int> _counter;

  @override
  void initState() {
    super.initState();
    _counter = widget.preferences.getInt('counter', defaultsTo: 0);
  }

  @override
  Widget build(BuildContext context) {
    return StreamBuilder(
      stream: _counter, // Good :-)
      builder: (context, snapshot) { .. },
    );
  }
}
```

**DO NOT:**

```dart

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return StreamBuilder(
      stream: widget.preferences.getInt('counter', defaultsTo: 0), // BAD! >:-(
      builder: (context, snapshot) { .. },
    );
  }
}
```