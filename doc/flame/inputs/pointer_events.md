# События указателя

```{note}
В этом документе описан новый API событий. Старый (устаревший) подход,
который всё ещё поддерживается, описан в разделе [](gesture_input.md).
```

**События указателя** — это обобщённые события Flutter типа «перемещение мыши» (для настольных или веб-приложений).

Если вам нужно реагировать на перемещение мыши внутри вашего компонента или игры, используйте примесь `PointerMoveCallbacks`.

Например:

```dart
class MyComponent extends PositionComponent with PointerMoveCallbacks {
  MyComponent() : super(size: Vector2(80, 60));

  @override
  void onPointerMove(PointerMoveEvent event) {
    // Выполнить действие в ответ на перемещение мыши (например, обновить координаты)
  }
}
```

Примесь добавляет в компонент два переопределяемых метода:

- `onPointerMove`: вызывается, когда мышь движется внутри компонента
- `onPointerMoveStop`: вызывается однократно, если компонент был под курсором, а затем мышь его покидает

По умолчанию эти методы ничего не делают — для выполнения каких-либо действий их необходимо переопределить.

Кроме того, компонент должен реализовывать метод `containsLocalPoint()` (уже реализован в `PositionComponent`, поэтому в большинстве случаев вам ничего делать не нужно). Этот метод позволяет Flame определить, произошло ли событие внутри компонента или нет.

Обратите внимание: только события мыши, происходящие внутри вашего компонента, будут передаваться ему. Однако `onPointerMoveStop` сработает один раз при первом движении мыши, которое покидает компонент, так что вы можете обработать там условия выхода.


## HoverCallbacks

Если вы хотите точно знать, наведён ли курсор на ваш компонент, или перехватывать события входа и выхода курсора, используйте более специализированную примесь `HoverCallbacks`.

Например:

```dart
class MyComponent extends PositionComponent with HoverCallbacks {

  MyComponent() : super(size: Vector2(80, 60));

  @override
  void update(double dt) {
    // используйте `isHovered`, чтобы узнать, наведён ли курсор на компонент
  }

  @override
  void onHoverEnter() {
    // Выполнить действие при входе курсора в область компонента
  }

  @override
  void onHoverExit() {
    // Выполнить действие при выходе курсора из области компонента
  }
}
```

Заметьте, что вы по-прежнему можете слушать «сырые» методы `onPointerMove` для дополнительной функциональности, только не забудьте вызвать версию `super`, чтобы сохранить поведение `HoverCallbacks`.


### Демонстрация наведения

Поиграйте с демонстрацией ниже, чтобы увидеть события наведения указателя в действии.

```{flutter-app}
:sources: ../flame/examples
:page: pointer_events
:show: widget code
```


## ScrollCallbacks

Если вы хотите обрабатывать события колесика мыши или тачпада на уровне компонента, используйте примесь `ScrollCallbacks`.

```dart
class ScrollableSquare extends RectangleComponent with ScrollCallbacks {
  @override
  void onScroll(ScrollEvent event) {
    final factor = switch (event.scrollDelta.y.sign) {
      1 => 0.9,
      -1 => 1.1,
      _ => 1.0, // игнорировать чисто горизонтальную прокрутку
    };
    scale.scale(factor);
    scale.clampScalar(0.3, 5.0);
    event.continuePropagation = false;
  }
}
```

Примесь добавляет один переопределяемый метод:

- `onScroll`: вызывается, когда событие прокрутки попадает в компонент.

`ScrollEvent` предоставляет:

- `scrollDelta`: `Vector2` со смещением прокрутки (обычно вам нужна только `scrollDelta.y`).
- `canvasPosition` / `localPosition`: позиция указателя в координатах холста и в локальных координатах.
- `continuePropagation`: установите в `false`, чтобы предотвратить распространение события на компоненты, находящиеся ниже в списке проверки попаданий (или на саму игру).

События прокрутки доставляются **всем** компонентам под указателем (не только самому верхнему), если только не установить `continuePropagation = false`, чтобы остановить дальнейшее всплытие.

Вы также можете подмешать `ScrollCallbacks` непосредственно к вашему `FlameGame`, чтобы обрабатывать прокрутку на уровне игры.


### Демонстрация прокрутки

```{flutter-app}
:sources: ../flame/examples
:page: scroll
:show: widget code
```# Pointer Events

```{note}
This document describes the new events API. The old (legacy) approach,
which is still supported, is described in [](gesture_input.md).
```

**Pointer events** are Flutter's generalized "mouse-movement"-type events (for desktop or web).

If you want to interact with mouse movement events within your component or game, you can use the
`PointerMoveCallbacks` mixin.

For example:

```dart
class MyComponent extends PositionComponent with PointerMoveCallbacks {
  MyComponent() : super(size: Vector2(80, 60));

  @override
  void onPointerMove(PointerMoveEvent event) {
    // Do something in response to the mouse move (e.g. update coordinates)
  }
}
```

The mixin adds two overridable methods to your component:

- `onPointerMove`: called when the mouse moves within the component
- `onPointerMoveStop`: called once if the component was being hovered and the mouse leaves

By default, each of these methods does nothing, they need to be overridden in order to perform any
function.

In addition, the component must implement the `containsLocalPoint()` method (already implemented in
`PositionComponent`, so most of the time you don't need to do anything here). This method allows
Flame to know whether the event occurred within the component or not.

Note that only mouse events happening within your component will be proxied along. However,
`onPointerMoveStop` will be fired once on the first mouse movement that leaves your component, so
you can handle any exit conditions there.


## HoverCallbacks

If you want to specifically know if your component is being hovered or not, or if you want to hook
into hover enter and exit events, you can use a more dedicated mixin called `HoverCallbacks`.

For example:

```dart
class MyComponent extends PositionComponent with HoverCallbacks {

  MyComponent() : super(size: Vector2(80, 60));

  @override
  void update(double dt) {
    // use `isHovered` to know if the component is being hovered
  }

  @override
  void onHoverEnter() {
    // Do something in response to the mouse entering the component
  }

  @override
  void onHoverExit() {
    // Do something in response to the mouse leaving the component
  }
}
```

Note that you can still listen to the "raw" onPointerMove methods for additional functionality, just
make sure to call the `super` version to enable the `HoverCallbacks` behavior.


### Hover Demo

Play with the demo below to see the pointer hover events in action.

```{flutter-app}
:sources: ../flame/examples
:page: pointer_events
:show: widget code
```


## ScrollCallbacks

If you want to handle mouse-wheel or trackpad scroll events at the component level, use the
`ScrollCallbacks` mixin.

```dart
class ScrollableSquare extends RectangleComponent with ScrollCallbacks {
  @override
  void onScroll(ScrollEvent event) {
    final factor = switch (event.scrollDelta.y.sign) {
      1 => 0.9,
      -1 => 1.1,
      _ => 1.0, // ignore horizontal-only scrolls
    };
    scale.scale(factor);
    scale.clampScalar(0.3, 5.0);
    event.continuePropagation = false;
  }
}
```

The mixin adds one overridable method:

- `onScroll`: called when a scroll event hits the component.

The `ScrollEvent` provides:

- `scrollDelta`: a `Vector2` with the scroll offset (typically you only need `scrollDelta.y`).
- `canvasPosition` / `localPosition`: the pointer position in canvas and local coordinates.
- `continuePropagation`: set to `false` to prevent the event from reaching components further
  down the hit-test list (or the game itself).

Scroll events are delivered to **all** components under the pointer (not just the topmost one),
unless set `continuePropagation = false` is set to stop it from bubbling further.

You can also mix `ScrollCallbacks` directly into your `FlameGame` to handle scrolling at the
game level.


### Scroll Demo

```{flutter-app}
:sources: ../flame/examples
:page: scroll
:show: widget code
```
