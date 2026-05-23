# Клавиатурный ввод

Здесь представлена документация по клавиатурному вводу.

Документация по другим видам ввода:

- [Ввод жестов](gesture_input.md): для жестов мыши и касаний
- [Другие вводы](other_inputs.md): для джойстиков, геймпадов и т.д.


## Введение

API клавиатуры во Flame основан на виджете [Focus](https://api.flutter.dev/flutter/widgets/Focus-class.html) Flutter.

Для настройки поведения фокуса см. раздел [Управление фокусом](#управление-фокусом).

Игра может реагировать на нажатия клавиш двумя способами: на уровне игры и на уровне компонента. Для каждого из них существует примесь, которую можно добавить к классу `Game` или `Component`.


### Получение событий клавиатуры на уровне игры

Чтобы подкласс `Game` реагировал на нажатия клавиш, добавьте к нему примесь `KeyboardEvents`.

После этого можно переопределить метод `onKeyEvent`.

Этот метод принимает два параметра: первый — [`KeyEvent`](https://api.flutter.dev/flutter/services/KeyEvent-class.html), вызвавший обратный вызов, а второй — набор нажатых в данный момент [`LogicalKeyboardKey`](https://api.flutter.dev/flutter/services/LogicalKeyboardKey-class.html).

Возвращаемое значение — [`KeyEventResult`](https://api.flutter.dev/flutter/widgets/KeyEventResult.html).

`KeyEventResult.handled` сообщает фреймворку, что нажатие клавиши было обработано внутри Flame, и следует пропустить все остальные виджеты-обработчики клавиатуры, кроме `GameWidget`.

`KeyEventResult.ignored` сообщает фреймворку, что нужно продолжить проверку этого события в других виджетах-обработчиках клавиатуры, помимо `GameWidget`. Если событие не будет обработано ни одним обработчиком, фреймворк воспроизведёт `SystemSoundType.alert`.

`KeyEventResult.skipRemainingHandlers` очень похож на `.ignored`, за исключением того, что он пропускает все остальные виджеты-обработчики и сразу воспроизводит предупреждающий звук.

Минимальный пример:

```dart
class MyGame extends FlameGame with KeyboardEvents {
  // ...
  @override
  KeyEventResult onKeyEvent(
    KeyEvent event,
    Set<LogicalKeyboardKey> keysPressed,
  ) {
    final isKeyDown = event is KeyDownEvent;

    final isSpace = keysPressed.contains(LogicalKeyboardKey.space);

    if (isSpace && isKeyDown) {
      if (keysPressed.contains(LogicalKeyboardKey.altLeft) ||
          keysPressed.contains(LogicalKeyboardKey.altRight)) {
        this.shootHarder();
      } else {
        this.shoot();
      }
      return KeyEventResult.handled;
    }
    return KeyEventResult.ignored;
  }
}
```


### Получение событий клавиатуры на уровне компонента

Для получения событий клавиатуры непосредственно в компонентах существует примесь `KeyboardHandler`.

Подобно `TapCallbacks` и `DragCallbacks`, `KeyboardHandler` может быть подмешан к любому подклассу `Component`.

`KeyboardHandler` следует добавлять только в игры, к которым подмешан `HasKeyboardHandlerComponents`.

> ⚠️ Примечание: Если используется `HasKeyboardHandlerComponents`, необходимо убрать `KeyboardEvents`
> из списка примесей игры, чтобы избежать конфликтов.

После применения `KeyboardHandler` можно переопределить метод `onKeyEvent`.

Этот метод принимает два параметра. Первый — [`KeyEvent`](https://api.flutter.dev/flutter/services/KeyEvent-class.html), вызвавший обратный вызов. Второй — набор нажатых в данный момент [`LogicalKeyboardKey`](https://api.flutter.dev/flutter/services/LogicalKeyboardKey-class.html).

Возвращаемое значение должно быть `true`, чтобы разрешить дальнейшее распространение события клавиатуры среди других компонентов. Чтобы запретить другим компонентам получать это событие, верните `false`.

Flame также предоставляет готовую реализацию — `KeyboardListenerComponent`, которую можно использовать для обработки событий клавиатуры. Как и любой другой компонент, его можно добавить как дочерний к `FlameGame` или другому `Component`:

Например, представьте `PositionComponent` с методами для перемещения по осям X и Y. Тогда следующий код можно использовать для привязки этих методов к событиям клавиш:

```dart
add(
  KeyboardListenerComponent(
    keyUp: {
      LogicalKeyboardKey.keyA: (keysPressed) { ... },
      LogicalKeyboardKey.keyD: (keysPressed) { ... },
      LogicalKeyboardKey.keyW: (keysPressed) { ... },
      LogicalKeyboardKey.keyS: (keysPressed) { ... },
    },
    keyDown: {
      LogicalKeyboardKey.keyA: (keysPressed) { ... },
      LogicalKeyboardKey.keyD: (keysPressed) { ... },
      LogicalKeyboardKey.keyW: (keysPressed) { ... },
      LogicalKeyboardKey.keyS: (keysPressed) { ... },
    },
  ),
);
```


### Управление фокусом

На уровне виджетов можно использовать API [`FocusNode`](https://api.flutter.dev/flutter/widgets/FocusNode-class.html) для управления тем, находится ли игра в фокусе или нет.

`GameWidget` имеет необязательный параметр `focusNode`, позволяющий управлять фокусом извне.

По умолчанию `autofocus` у `GameWidget` установлен в `true`, что означает, что он получит фокус сразу после монтирования. Чтобы переопределить это поведение, установите `autofocus` в `false`.

Более полный пример см. в [примере клавиатурного ввода](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/input/keyboard_example.dart).
