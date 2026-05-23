# Ввод жестов

Это документация по обработке жестов, добавленных непосредственно к классу игры. Чаще всего, однако, нужно определять ввод на компонентах. См., например, [TapCallbacks](tap_events.md) и [DragCallbacks](drag_events.md).

Документация по другим видам ввода:

- [Клавиатурный ввод](keyboard_input.md): для нажатий клавиш
- [Другие вводы](other_inputs.md): для джойстиков, геймпадов и т.д.


## Введение

Внутри `package:flame/gestures.dart` вы найдёте целый набор `примесей` (mixins), которые можно включить в экземпляр вашего игрового класса, чтобы получать события сенсорного ввода. Ниже представлен полный список этих `примесей` и их методов:


## Детекторы касания и мыши

```{warning}
Детекторы будут объявлены устаревшими в будущем. Вместо них рекомендуется использовать `Callbacks`.
```

```text
- TapDetector
  - onTap
  - onTapCancel
  - onTapDown
  - onLongTapDown
  - onTapUp

- SecondaryTapDetector
  - onSecondaryTapDown
  - onSecondaryTapUp
  - onSecondaryTapCancel

- TertiaryTapDetector
  - onTertiaryTapDown
  - onTertiaryTapUp
  - onTertiaryTapCancel

- DoubleTapDetector
  - onDoubleTap

- LongPressDetector
  - onLongPress
  - onLongPressStart
  - onLongPressMoveUpdate
  - onLongPressUp
  - onLongPressEnd

- VerticalDragDetector
  - onVerticalDragDown
  - onVerticalDragStart
  - onVerticalDragUpdate
  - onVerticalDragEnd
  - onVerticalDragCancel

- HorizontalDragDetector
  - onHorizontalDragDown
  - onHorizontalDragStart
  - onHorizontalDragUpdate
  - onHorizontalDragEnd
  - onHorizontalDragCancel

- ForcePressDetector
  - onForcePressStart
  - onForcePressPeak
  - onForcePressUpdate
  - onForcePressEnd

- PanDetector
  - onPanDown
  - onPanStart
  - onPanUpdate
  - onPanEnd
  - onPanCancel

- ScaleDetector
  - onScaleStart
  - onScaleUpdate
  - onScaleEnd

- MultiTouchTapDetector
  - onTap
  - onTapCancel
  - onTapDown
  - onTapUp

- MultiTouchDragDetector
  - onReceiveDrag
```

События только для мыши

```text
 - MouseMovementDetector
  - onMouseMove
 - ScrollDetector
  - onScroll
```


Смешивать продвинутые детекторы (`MultiTouch*`) с базовыми детекторами того же типа невозможно, поскольку продвинутые детекторы всегда *побеждают на арене жестов* и базовые никогда не будут вызваны. Например, нельзя одновременно использовать `MultiTouchTapDetector` и `PanDetector` — для последнего события не будут генерироваться (на это также есть утверждение).

Жестовый API Flame предоставляется виджетами жестов Flutter, включая
[GestureDetector](https://api.flutter.dev/flutter/widgets/GestureDetector-class.html),
[RawGestureDetector](https://api.flutter.dev/flutter/widgets/RawGestureDetector-class.html)
и [MouseRegion](https://api.flutter.dev/flutter/widgets/MouseRegion-class.html). Вы также можете прочитать подробнее о
[системе жестов Flutter](https://api.flutter.dev/flutter/gestures/gestures-library.html).


## PanDetector и ScaleDetector

Если добавить `PanDetector` вместе с `ScaleDetector`, Flutter выдаст довольно непонятное утверждение:

```{note}
Наличие одновременно распознавателя жестов панорамирования и распознавателя
жестов масштабирования избыточно; масштабирование является надмножеством
панорамирования.

Просто используйте распознаватель масштабирования.
```

Это может показаться странным, но `onScaleUpdate` вызывается не только при изменении масштаба, но и при всех событиях панорамирования/перетаскивания. Поэтому, если вам нужны оба этих детектора, придётся обрабатывать логику обоих внутри `onScaleUpdate` (а также `onScaleStart` и `onScaleEnd`).

Например, если вы хотите перемещать камеру при панорамировании и изменять масштаб при жесте масштабирования, можно сделать так:

```dart
  void clampZoom() {
    camera.viewfinder.zoom = camera.viewfinder.zoom.clamp(0.05, 3.0);
  }

  late double startZoom;

  @override
  void onScaleStart(_) {
    startZoom = camera.viewfinder.zoom;
  }

  @override
  void onScaleUpdate(ScaleUpdateInfo info) {
    final currentScale = info.scale.global;
    if (!currentScale.isIdentity()) {
      camera.viewfinder.zoom = startZoom * currentScale.y;
      clampZoom();
    } else {
      final zoom = camera.viewfinder.zoom;
      final delta = (info.delta.global..negate()) / zoom;
      camera.moveBy(delta);
    }
  }
```

В примере выше события панорамирования обрабатываются с помощью `info.delta`, а масштабирования — с помощью `info.scale`, хотя теоретически все они исходят от событий масштабирования.

Это также показано в
[примере масштабирования](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/camera_and_viewport/zoom_example.dart).


## Курсор мыши

Также можно изменить текущий курсор мыши, отображаемый над областью `GameWidget`. Для этого в классе `Game` используйте следующий код:

```dart
mouseCursor.value = SystemMouseCursors.move;
```

Чтобы сразу инициализировать `GameWidget` с пользовательским курсором, можно использовать свойство `mouseCursor`:

```dart
GameWidget(
  game: MouseCursorGame(),
  mouseCursor: SystemMouseCursors.move,
);
```


## Система координат событий

У событий, имеющих позиции, например `Tap*` или `Drag`, атрибут `eventPosition` содержит два поля: `global` и `widget`. Ниже приведено краткое описание каждого из них.


### global

Позиция, в которой произошло событие, относительно всего экрана, аналогично `globalPosition` в нативных событиях Flutter.


### widget

Позиция, в которой произошло событие, относительно позиции и размера `GameWidget`, аналогично `localPosition` в нативных событиях Flutter.


## Пример

```dart
class MyGame extends FlameGame with TapDetector {
  // Другие методы опущены

  @override
  bool onTapDown(TapDownInfo info) {
    print("Касание вниз на ${info.eventPosition.widget}");
    return true;
  }

  @override
  bool onTapUp(TapUpInfo info) {
    print("Касание вверх на ${info.eventPosition.widget}");
    return true;
  }
}
```

Также вы можете посмотреть более полные примеры в
[каталоге примеров ввода](https://github.com/flame-engine/flame/tree/main/examples/lib/stories/input/).


### GestureHitboxes

Примесь `GestureHitboxes` используется для более точного распознавания жестов на ваших `Component`. Допустим, у вас есть довольно круглый камень в виде `SpriteComponent`. Вы не хотите регистрировать ввод в углах изображения, где камня фактически нет, поскольку `PositionComponent` по умолчанию прямоугольный. Тогда вы можете использовать примесь `GestureHitboxes`, чтобы задать более точный круг или многоугольник (или другую фигуру), попадание в который будет считаться событием на вашем компоненте.

Добавлять новые хитбоксы к компоненту с примесью `GestureHitboxes` можно так же, как показано ниже в примере для `Collidable`.

Подробнее об определении хитбоксов рассказано в разделе хитбоксов документации по
[обнаружению столкновений](../collision_detection.md#shapehitbox).

Пример использования можно увидеть в
[примере gesture hitboxes](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/input/gesture_hitboxes_example.dart).# Gesture Input

This is documentation for gesture inputs attached directly on the game class, most of the time you
want to detect input on your components instead, see for example the [TapCallbacks](tap_events.md)
and [DragCallbacks](drag_events.md) for that.

For other input documents, see also:

- [Keyboard Input](keyboard_input.md): for keystrokes
- [Other Inputs](other_inputs.md): For joysticks, game pads, etc.


## Intro

Inside `package:flame/gestures.dart` you can find a whole set of `mixin`s which can be included on
your game class instance to be able to receive touch input events. Below you can see the full list
of these `mixin`s and its methods:


## Touch and mouse detectors

```{warning}
Detectors will be deprecated in the future. Prefer `Callbacks` instead.
```

```text
- TapDetector
  - onTap
  - onTapCancel
  - onTapDown
  - onLongTapDown
  - onTapUp

- SecondaryTapDetector
  - onSecondaryTapDown
  - onSecondaryTapUp
  - onSecondaryTapCancel

- TertiaryTapDetector
  - onTertiaryTapDown
  - onTertiaryTapUp
  - onTertiaryTapCancel

- DoubleTapDetector
  - onDoubleTap

- LongPressDetector
  - onLongPress
  - onLongPressStart
  - onLongPressMoveUpdate
  - onLongPressUp
  - onLongPressEnd

- VerticalDragDetector
  - onVerticalDragDown
  - onVerticalDragStart
  - onVerticalDragUpdate
  - onVerticalDragEnd
  - onVerticalDragCancel

- HorizontalDragDetector
  - onHorizontalDragDown
  - onHorizontalDragStart
  - onHorizontalDragUpdate
  - onHorizontalDragEnd
  - onHorizontalDragCancel

- ForcePressDetector
  - onForcePressStart
  - onForcePressPeak
  - onForcePressUpdate
  - onForcePressEnd

- PanDetector
  - onPanDown
  - onPanStart
  - onPanUpdate
  - onPanEnd
  - onPanCancel

- ScaleDetector
  - onScaleStart
  - onScaleUpdate
  - onScaleEnd

- MultiTouchTapDetector
  - onTap
  - onTapCancel
  - onTapDown
  - onTapUp

- MultiTouchDragDetector
  - onReceiveDrag
```

Mouse only events

```text
 - MouseMovementDetector
  - onMouseMove
 - ScrollDetector
  - onScroll
```


It is not possible to mix advanced detectors (`MultiTouch*`) with basic detectors of the same
kind, since the advanced detectors will *always win the gesture arena* and the basic detectors will
never be triggered. So for example, you can't use both `MultiTouchTapDetector` and `PanDetector`
together, since no events will be triggered for the latter (there is also an assertion for this).

Flame's GestureApi is provided by Flutter's Gesture Widgets, including
[GestureDetector widget](https://api.flutter.dev/flutter/widgets/GestureDetector-class.html),
[RawGestureDetector widget](https://api.flutter.dev/flutter/widgets/RawGestureDetector-class.html)
and [MouseRegion widget](https://api.flutter.dev/flutter/widgets/MouseRegion-class.html), you can
also read more about
[Flutter's gesture system](https://api.flutter.dev/flutter/gestures/gestures-library.html).


## PanDetector and ScaleDetector

If you add a `PanDetector` together with a `ScaleDetector` you will be prompted with a quite cryptic
assertion from Flutter that says:

```{note}
Having both a pan gesture recognizer and a scale gesture recognizer is
redundant; scale is a superset of pan.

Just use the scale gesture recognizer.
```

This might seem strange, but `onScaleUpdate` is not only triggered when the scale should be changed,
but for all pan/drag events too. So if you need to use both of those detectors you'll have to handle
both of their logic inside `onScaleUpdate` (+`onScaleStart` and `onScaleEnd`).

For example you could do something like this if you want to move the camera on pan events and zoom
on scale events:

```dart
  void clampZoom() {
    camera.viewfinder.zoom = camera.viewfinder.zoom.clamp(0.05, 3.0);
  }

  late double startZoom;

  @override
  void onScaleStart(_) {
    startZoom = camera.viewfinder.zoom;
  }

  @override
  void onScaleUpdate(ScaleUpdateInfo info) {
    final currentScale = info.scale.global;
    if (!currentScale.isIdentity()) {
      camera.viewfinder.zoom = startZoom * currentScale.y;
      clampZoom();
    } else {
      final zoom = camera.viewfinder.zoom;
      final delta = (info.delta.global..negate()) / zoom;
      camera.moveBy(delta);
    }
  }
```

In the example above the pan events are handled with `info.delta` and the scale events with
`info.scale`, although they are theoretically both from underlying scale events.

This can also be seen in the
[zoom example](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/camera_and_viewport/zoom_example.dart).


## Mouse cursor

It is also possible to change the current mouse cursor displayed on the `GameWidget` region. To do
so the following code can be used inside the `Game` class

```dart
mouseCursor.value = SystemMouseCursors.move;
```

To already initialize the `GameWidget` with a custom cursor, the `mouseCursor` property can be used

```dart
GameWidget(
  game: MouseCursorGame(),
  mouseCursor: SystemMouseCursors.move,
);
```


## Event coordinate system

On events that have positions, like for example `Tap*` or `Drag`, you will notice that the
`eventPosition` attribute includes 2 fields: `global` and `widget`. Below you will find a brief
explanation about each of them.


### global

The position where the event occurred considering the entire screen, same as
`globalPosition` in Flutter's native events.


### widget

The position where the event occurred relative to the `GameWidget` position and size, same as
`localPosition` in Flutter's native events.


## Example

```dart
class MyGame extends FlameGame with TapDetector {
  // Other methods omitted

  @override
  bool onTapDown(TapDownInfo info) {
    print("Player tap down on ${info.eventPosition.widget}");
    return true;
  }

  @override
  bool onTapUp(TapUpInfo info) {
    print("Player tap up on ${info.eventPosition.widget}");
    return true;
  }
}
```

You can also check more complete examples in the
[input examples directory](https://github.com/flame-engine/flame/tree/main/examples/lib/stories/input/).


### GestureHitboxes

The `GestureHitboxes` mixin is used to more accurately recognize gestures on top of your
`Component`s. Say that you have a fairly round rock as a `SpriteComponent` for example, then you
don't want to register input that is in the corner of the image where the rock is not displayed,
since a `PositionComponent` is rectangular by default. Then you can use the `GestureHitboxes` mixin
to define a more accurate circle or polygon (or another shape) for which the input should be within
for the event to be registered on your component.

You can add new hitboxes to the component that has the `GestureHitboxes` mixin just like they are
added in the below `Collidable` example.

More information about how to define hitboxes can be found in the hitbox section of the
[collision detection](../collision_detection.md#shapehitbox) docs.

An example of how to use it can be seen in the
[gesture hitboxes example](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/input/gesture_hitboxes_example.dart).
