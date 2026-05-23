# GameWidget

`GameWidget` — это мост между Flutter и Flame. Поскольку игры Flame сами по себе не являются виджетами Flutter, `GameWidget` оборачивает экземпляр `Game` и помещает его в дерево виджетов Flutter, как и любой другой [виджет](https://docs.flutter.dev/get-started/fundamentals/widgets). Это позволяет сочетать полноэкранную игру с элементами пользовательского интерфейса Flutter (панели навигации, оверлеи, диалоговые окна) или встраивать игру лишь как часть макета вашего приложения.

```{dartdoc}
:package: flame
:symbol: GameWidget
:file: src/game/game_widget/game_widget.dart

[ClipRect]: https://api.flutter.dev/flutter/widgets/ClipRect-class.html
[FocusNode]: https://api.flutter.dev/flutter/widgets/FocusNode-class.html
[RepaintBoundary]: https://api.flutter.dev/flutter/widgets/RepaintBoundary-class.html
```


## Поведение при проверке попадания (Hit Test Behavior)

Аргумент `behavior` управляет тем, как `GameWidget` участвует в проверке попаданий (hit testing) Flutter. Он определяет, будут ли события указателя (касания, перетаскивания и т.д.) поглощаться игрой или пропускаться к виджетам, расположенным ниже в дереве виджетов.

Существует три возможных значения из `HitTestBehavior` Flutter:

- **`HitTestBehavior.opaque`** (по умолчанию): Игра поглощает все события указателя на всей своей поверхности, не позволяя виджетам позади получать их. Это классическое поведение, при котором игра действует как сплошной слой.

- **`HitTestBehavior.deferToChild`**: Игра перехватывает события только в тех позициях, где существует компонент с обратными вызовами событий (например, `TapCallbacks`). События в позициях без интерактивных компонентов проходят к виджетам позади `GameWidget`. Это полезно, когда игра накладывается поверх пользовательского интерфейса Flutter, и вы хотите, чтобы нижележащие виджеты оставались интерактивными в областях, которые игре не нужно обрабатывать.

- **`HitTestBehavior.translucent`**: Игра получает события там, где у неё есть компоненты, обрабатывающие события, но всегда также разрешает проверку попадания для виджетов позади. И игра, и виджеты позади могут получить одно и то же событие.


### Пропуск касаний

Распространённым сценарием является размещение `GameWidget` поверх других виджетов Flutter в `Stack`. По умолчанию игра блокирует любое взаимодействие с нижележащими виджетами. Чтобы касания проходили к этим виджетам, установите `behavior` в `HitTestBehavior.deferToChild`:

```dart
Widget build(BuildContext context) {
  return Stack(
    children: [
      // Виджеты Flutter снизу
      Center(
        child: ElevatedButton(
          onPressed: () => print('Button tapped!'),
          child: const Text('Tap me'),
        ),
      ),
      // Игра сверху, пропускающая касания
      Positioned.fill(
        child: GameWidget(
          game: MyGame(),
          behavior: HitTestBehavior.deferToChild,
        ),
      ),
    ],
  );
}
```

В такой конфигурации касание области без интерактивных игровых компонентов достигнет `ElevatedButton` позади игры. Касание игрового компонента, использующего `TapCallbacks`, будет обработано игрой.

```{note}
При использовании `deferToChild` или `translucent` `FlameGame` определяет, есть ли в данной позиции интерактивный компонент, обходя дерево компонентов с помощью `componentsAtPoint`. Игры, напрямую расширяющие низкоуровневый класс `Game`, по умолчанию сообщают о попадании по всей своей поверхности; переопределите `containsEventHandlerAt`, чтобы изменить это поведение.
```# Game Widget

The `GameWidget` is the bridge between Flutter and Flame. Since Flame games are not Flutter widgets
by themselves, the `GameWidget` wraps a `Game` instance and places it into the Flutter widget tree,
just like any other [widget](https://docs.flutter.dev/get-started/fundamentals/widgets). This lets
you combine a full-screen game with Flutter UI elements (navigation bars, overlays, dialogs) or
embed a game as only part of your app's layout.

```{dartdoc}
:package: flame
:symbol: GameWidget
:file: src/game/game_widget/game_widget.dart

[ClipRect]: https://api.flutter.dev/flutter/widgets/ClipRect-class.html
[FocusNode]: https://api.flutter.dev/flutter/widgets/FocusNode-class.html
[RepaintBoundary]: https://api.flutter.dev/flutter/widgets/RepaintBoundary-class.html
```


## Hit Test Behavior

The `behavior` argument controls how the `GameWidget` participates in Flutter's hit testing. This
determines whether pointer events (taps, drags, etc.) are absorbed by the game or allowed to pass
through to widgets underneath it in the widget tree.

There are three possible values from Flutter's `HitTestBehavior`:

- **`HitTestBehavior.opaque`** (default): The game absorbs all pointer events on its entire surface,
  preventing any widgets behind it from receiving them. This is the classic behavior where the game
  acts as a solid layer.

- **`HitTestBehavior.deferToChild`**: The game only intercepts events at positions where a component
  with event callbacks (e.g. `TapCallbacks`) exists. Events at positions with no interactive
  components pass through to widgets behind the `GameWidget`. This is useful when layering a game on
  top of Flutter UI and you want the underlying widgets to remain interactive in areas the game
  doesn't need to handle.

- **`HitTestBehavior.translucent`**: The game receives events where it has event-handling
  components, but always allows widgets behind it to be hit-tested as well. Both the game and the
  widgets behind it can receive the same event.


### Allowing taps to pass through

A common use case is placing a `GameWidget` on top of other Flutter widgets in a `Stack`. By
default, the game will block all interaction with the widgets underneath. To let taps pass through
to those widgets, set `behavior` to `HitTestBehavior.deferToChild`:

```dart
Widget build(BuildContext context) {
  return Stack(
    children: [
      // Flutter widgets underneath
      Center(
        child: ElevatedButton(
          onPressed: () => print('Button tapped!'),
          child: const Text('Tap me'),
        ),
      ),
      // Game on top, letting taps pass through
      Positioned.fill(
        child: GameWidget(
          game: MyGame(),
          behavior: HitTestBehavior.deferToChild,
        ),
      ),
    ],
  );
}
```

In this setup, tapping an area with no interactive game components will reach the `ElevatedButton`
behind the game. Tapping a game component that uses `TapCallbacks` will be handled by the game
instead.

```{note}
When using `deferToChild` or `translucent`, `FlameGame` determines whether a
position has an interactive component by traversing the component tree via
`componentsAtPoint`. Games that directly extend the low-level `Game` class
report a hit on their entire surface by default; override
`containsEventHandlerAt` to customize this.
```
