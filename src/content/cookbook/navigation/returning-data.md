---
# title: Return data from a screen
title: 从一个页面回传数据
# description: How to return data from a new screen.
description: 如何从新页面返回数据。
tags: cookbook, 实用教程, 路由
keywords: 传参, 回传数据
js:
  - defer: true
    url: /assets/js/inject_dartpad.js
---

<?code-excerpt path-base="cookbook/navigation/returning_data/"?>

In some cases, you might want to return data from a new screen.
For example, say you push a new screen that presents two options to a user.
When the user taps an option, you want to inform the first screen
of the user's selection so that it can act on that information.

在某些场景下，我们需要在回退到上一屏时同时返回一些数据。
比如，我们跳转到新的一屏，有两个选项让用户选择，
当用户点击某个选项后会返回到第一屏，同时在第一屏可以知道用户选择的信息。

You can do this with the [`Navigator.pop()`][]
method using the following steps:

你可以使用 [`Navigator.pop()`][] 来进行以下步骤：

## Directions

## 步骤

  1. Define the home screen

     创建主屏界面

  2. Add a button that launches the selection screen

     添加按钮，点击时跳转到选择界面

  3. Show the selection screen with two buttons

     在选择界面显示两个按钮

  4. When a button is tapped, close the selection screen

     当任意一个按钮被点击，关闭选择界面回退到主屏界面

  5. Show a snackbar on the home screen with the selection

     在主屏界面显示 snackbar ，展示选中的项目

## 1. Define the home screen

## 1. 创建主屏界面

The home screen displays a button. When tapped,
it launches the selection screen.

主屏界面显示一个按钮，当点击按钮时跳转到选择界面。

<?code-excerpt "lib/main_step2.dart (HomeScreen)"?>
```dart
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Returning Data Demo')),
      // Create the SelectionButton widget in the next step.
      body: const Center(child: SelectionButton()),
    );
  }
}
```

## 2. Add a button that launches the selection screen

## 2. 添加按钮，点击时跳转到选择界面

Now, create the SelectionButton, which does the following:

接下来，我们创建 SelectionButton 按钮，它有两个功能：

  * Launches the SelectionScreen when it's tapped.

    点击时跳转到选择界面

  * Waits for the SelectionScreen to return a result.

    等待选择界面给它返回结果

<?code-excerpt "lib/main_step2.dart (SelectionButton)"?>
```dart
class SelectionButton extends StatefulWidget {
  const SelectionButton({super.key});

  @override
  State<SelectionButton> createState() => _SelectionButtonState();
}

class _SelectionButtonState extends State<SelectionButton> {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        _navigateAndDisplaySelection(context);
      },
      child: const Text('Pick an option, any option!'),
    );
  }

  Future<void> _navigateAndDisplaySelection(BuildContext context) async {
    // Navigator.push returns a Future that completes after calling
    // Navigator.pop on the Selection Screen.
    final result = await Navigator.push(
      context,
      // Create the SelectionScreen in the next step.
      MaterialPageRoute(builder: (context) => const SelectionScreen()),
    );
  }
}
```

## 3. Show the selection screen with two buttons

## 3. 在选择界面显示两个按钮

Now, build a selection screen that contains two buttons.
When a user taps a button,
that app closes the selection screen and lets the home
screen know which button was tapped.

现在来构建选择界面，它包含两个按钮，
当任意一个按钮被点击的时候，关闭选择页面回退到主屏界面，
并让主屏界面知道哪个按钮被点击了。

This step defines the UI.
The next step adds code to return data.

这一步我们来定义 UI，在下一步完成数据的返回。

<?code-excerpt "lib/main_step2.dart (SelectionScreen)"?>
```dart
class SelectionScreen extends StatelessWidget {
  const SelectionScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Pick an option')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Padding(
              padding: const EdgeInsets.all(8),
              child: ElevatedButton(
                onPressed: () {
                  // Pop here with "Yep"...
                },
                child: const Text('Yep!'),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(8),
              child: ElevatedButton(
                onPressed: () {
                  // Pop here with "Nope"...
                },
                child: const Text('Nope.'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

## 4. When a button is tapped, close the selection screen

## 4. 当任意一个按钮被点击，关闭选择界面回退到主屏界面

Now, update the `onPressed()` callback for both of the buttons.
To return data to the first screen,
use the [`Navigator.pop()`][] method,
which accepts an optional second argument called `result`.
Any result is returned to the `Future` in the SelectionButton.

接下来我们来更新两个按钮的 `onPressed()` 回调函数，
使用 [`Navigator.pop()`][] 回退界面并返回数据给主屏界面。
`Navigator.pop()` 方法可以接受第二个参数 `result`，它是可选的，
如果传递了 `result`，数据将会通过 `Future` 方法的返回值传递。

### Yep button

### Yep 按钮

<?code-excerpt "lib/main.dart (Yep)" replace="/^child: //g;/^\),$/)/g"?>
```dart
ElevatedButton(
  onPressed: () {
    // Close the screen and return "Yep!" as the result.
    Navigator.pop(context, 'Yep!');
  },
  child: const Text('Yep!'),
)
```

### Nope button

### Nope 按钮

<?code-excerpt "lib/main.dart (Nope)" replace="/^child: //g;/^\),$/)/g"?>
```dart
ElevatedButton(
  onPressed: () {
    // Close the screen and return "Nope." as the result.
    Navigator.pop(context, 'Nope.');
  },
  child: const Text('Nope.'),
)
```

## 5. Show a snackbar on the home screen with the selection

## 5. 在主屏界面显示一个 snackbar，展示选中的项目

Now that you're launching a selection screen and awaiting the result,
you'll want to do something with the information that's returned.

现在，我们跳转到选择界面并等待返回结果，当结果返回时我们可以做些事情。

In this case, show a snackbar displaying the result by using the
`_navigateAndDisplaySelection()` method in `SelectionButton`:

在本例中，我们用一个 snackbar 显示结果，
我们来更新 `SelectionButton` 类中的 `_navigateAndDisplaySelection()` 方法。

<?code-excerpt "lib/main.dart (navigateAndDisplay)"?>
```dart
// A method that launches the SelectionScreen and awaits the result from
// Navigator.pop.
Future<void> _navigateAndDisplaySelection(BuildContext context) async {
  // Navigator.push returns a Future that completes after calling
  // Navigator.pop on the Selection Screen.
  final result = await Navigator.push(
    context,
    MaterialPageRoute(builder: (context) => const SelectionScreen()),
  );

  // When a BuildContext is used from a StatefulWidget, the mounted property
  // must be checked after an asynchronous gap.
  if (!context.mounted) return;

  // After the Selection Screen returns a result, hide any previous snackbars
  // and show the new result.
  ScaffoldMessenger.of(context)
    ..removeCurrentSnackBar()
    ..showSnackBar(SnackBar(content: Text('$result')));
}
```

## Interactive example

## 交互式样例

<?code-excerpt "lib/main.dart"?>
```dartpad title="Flutter Return from Data hands-on example in DartPad" run="true"
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(title: 'Returning Data', home: HomeScreen()));
}

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Returning Data Demo')),
      body: const Center(child: SelectionButton()),
    );
  }
}

class SelectionButton extends StatefulWidget {
  const SelectionButton({super.key});

  @override
  State<SelectionButton> createState() => _SelectionButtonState();
}

class _SelectionButtonState extends State<SelectionButton> {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        _navigateAndDisplaySelection(context);
      },
      child: const Text('Pick an option, any option!'),
    );
  }

  // A method that launches the SelectionScreen and awaits the result from
  // Navigator.pop.
  Future<void> _navigateAndDisplaySelection(BuildContext context) async {
    // Navigator.push returns a Future that completes after calling
    // Navigator.pop on the Selection Screen.
    final result = await Navigator.push(
      context,
      MaterialPageRoute(builder: (context) => const SelectionScreen()),
    );

    // When a BuildContext is used from a StatefulWidget, the mounted property
    // must be checked after an asynchronous gap.
    if (!context.mounted) return;

    // After the Selection Screen returns a result, hide any previous snackbars
    // and show the new result.
    ScaffoldMessenger.of(context)
      ..removeCurrentSnackBar()
      ..showSnackBar(SnackBar(content: Text('$result')));
  }

}

class SelectionScreen extends StatelessWidget {
  const SelectionScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Pick an option')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Padding(
              padding: const EdgeInsets.all(8),
              child: ElevatedButton(
                onPressed: () {
                  // Close the screen and return "Yep!" as the result.
                  Navigator.pop(context, 'Yep!');
                },
                child: const Text('Yep!'),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(8),
              child: ElevatedButton(
                onPressed: () {
                  // Close the screen and return "Nope." as the result.
                  Navigator.pop(context, 'Nope.');
                },
                child: const Text('Nope.'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

<noscript>
  <img src="/assets/images/docs/cookbook/returning-data.gif" alt="Returning data demo" class="site-mobile-screenshot" />
</noscript>


[`Navigator.pop()`]: {{site.api}}/flutter/widgets/Navigator/pop.html
