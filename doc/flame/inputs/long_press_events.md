# События долгого нажатия

**События долгого нажатия** происходят, когда пользователь нажимает и удерживает указатель (палец или мышь) на компоненте в течение продолжительного времени. Этот жест обычно используется для контекстных меню, поведения перетаскивания при удержании или любых действий, требующих осознанного длительного касания.

Для компонентов, которые должны реагировать на долгие нажатия, добавьте примесь `LongPressCallbacks`.

- Эта примесь добавляет в компонент четыре переопределяемых метода: `onLongPressStart`, `onLongPressMoveUpdate`, `onLongPressEnd` и `onLongPressCancel`.
- По умолчанию `isLongPressing` отслеживается автоматически и может быть доступно вашему компоненту.
- Компонент должен реализовывать `containsLocalPoint()` (уже реализован в `PositionComponent`, так что в большинстве случаев ничего делать не нужно). Этот метод позволяет Flame определить, произошло ли событие внутри компонента. Вы можете переопределить его, чтобы он всегда возвращал `true`, и тогда компонент будет получать все события долгого нажатия независимо от позиции.

```dart
class MyComponent extends PositionComponent with LongPressCallbacks {
  MyComponent() : super(size: Vector2.all(32));

  @override
  void onLongPressStart(LongPressStartEvent event) {
    super.onLongPressStart(event); // обрабатывает внутреннее обновление isLongPressing
    // Выполнить действие при распознавании долгого нажатия
  }

  @override
  void onLongPressEnd(LongPressEndEvent event) {
    super.onLongPressEnd(event); // обрабатывает внутреннее обновление isLongPressing
    // Выполнить действие при завершении долгого нажатия
  }
}
```

## Анатомия долгого нажатия

### onLongPressStart

Первое событие в последовательности долгого нажатия. Оно срабатывает, как только указатель удерживается достаточно долго, чтобы быть распознанным как долгое нажатие. По умолчанию `LongPressGestureRecognizer` Flutter использует `kLongPressTimeout` (500 мс) в качестве определения долгого нажатия.

`LongPressStartEvent` предоставляет позицию точки контакта в нескольких системах координат: `devicePosition` (координаты устройства), `canvasPosition` (координаты игрового виджета) и `localPosition` (локальные координаты компонента).

Любой компонент, получивший `onLongPressStart`, позже получит либо `onLongPressEnd` (при успешном завершении), либо `onLongPressCancel` (при отмене). В промежутке также могут поступать обновления перемещения.

Вызов `super.onLongPressStart(event)` устанавливает `isLongPressing` в `true`.

### onLongPressMoveUpdate

Срабатывает непрерывно, пока пользователь перемещает палец во время активного долгого нажатия. Это событие доставляется только компонентам, получившим начальный `onLongPressStart`.

`LongPressMoveUpdateEvent` является `DisplacementEvent` и предоставляет межкадровое смещение (`delta`), подобно `DragUpdateEvent`. Это означает, что можно использовать `localDelta` для перемещения компонента вслед за указателем (корректно учитывается зум камеры и трансформации компонента). Событие также содержит `offsetFromOrigin` — полное смещение с начала жеста.

### onLongPressEnd

Срабатывает, когда пользователь поднимает указатель после долгого нажатия. `LongPressEndEvent` содержит финальную позицию и `velocity` (скорость) указателя в момент отпускания.

Вызов `super.onLongPressEnd(event)` устанавливает `isLongPressing` в `false`.

### onLongPressCancel

Срабатывает, если жест прерывается до завершения (например, конкурирующим распознавателем жестов).

Вызов `super.onLongPressCancel(event)` устанавливает `isLongPressing` в `false`.

## Примеси

### LongPressCallbacks

Примесь `LongPressCallbacks` может быть добавлена к любому `Component`, чтобы этот компонент начал получать события долгого нажатия.

Эта примесь добавляет в компонент методы `onLongPressStart`, `onLongPressMoveUpdate`, `onLongPressEnd` и `onLongPressCancel`. Переопределите их, чтобы реализовать нужное поведение.

Компонент будет получать только те события долгого нажатия, которые начинаются *внутри* этого компонента, что определяется функцией `containsLocalPoint()`. Широко используемый класс `PositionComponent` предоставляет такую реализацию на основе своего свойства `size`.

Примесь также предоставляет свойство `isLongPressing`, которое отслеживает, активно ли сейчас долгое нажатие на компоненте. Оно управляется автоматически, если вызывать `super` в `onLongPressStart`, `onLongPressEnd` и `onLongPressCancel`.

```dart
class LongPressSquare extends RectangleComponent with LongPressCallbacks {
  @override
  void onLongPressStart(LongPressStartEvent event) {
    super.onLongPressStart(event);
    paint.color = Colors.red;
  }

  @override
  void onLongPressMoveUpdate(LongPressMoveUpdateEvent event) {
    position += event.localDelta;
  }

  @override
  void onLongPressEnd(LongPressEndEvent event) {
    super.onLongPressEnd(event);
    paint.color = Colors.blue;
  }
}
```# Long Press Events

**Long press events** occur when the user presses and holds a pointer (finger or mouse) on a
component for a sustained period. This gesture is commonly used for context menus, drag-to-move
behaviors, or any action that requires a deliberate, sustained touch.

For components that should respond to long press events, add the `LongPressCallbacks` mixin.

- This mixin adds four overridable methods to your component: `onLongPressStart`,
  `onLongPressMoveUpdate`, `onLongPressEnd`, and `onLongPressCancel`.
- By default, `isLongPressing` is tracked automatically and can be accessed by your component.
- The component must implement `containsLocalPoint()` (already implemented in `PositionComponent`,
  so most of the time you don't need to do anything here). This method allows Flame to know whether
  the event occurred within the component or not. You can override it to `true` to receive all
  long press events regardless of position.

```dart
class MyComponent extends PositionComponent with LongPressCallbacks {
  MyComponent() : super(size: Vector2.all(32));

  @override
  void onLongPressStart(LongPressStartEvent event) {
    super.onLongPressStart(event); // handles internal updating of isLongPressing
    // Do something when the long press is recognized
  }

  @override
  void onLongPressEnd(LongPressEndEvent event) {
    super.onLongPressEnd(event); // handles internal updating of isLongPressing
    // Do something when the long press finishes
  }
}
```


## Long press anatomy


### onLongPressStart

The first event in a long press sequence. It fires once the pointer has been held down long enough
to be recognized as a long press. By default, Flutter's `LongPressGestureRecognizer` uses
`kLongPressTimeout` (500ms) as a long press definition.

The `LongPressStartEvent` provides the position of the contact point in multiple coordinate
systems: `devicePosition` (device coordinates), `canvasPosition` (game widget coordinates), and
`localPosition` (component-local coordinates).

Any component that receives `onLongPressStart` will later receive either `onLongPressEnd` (on
success) or `onLongPressCancel` (if cancelled). Move updates may also be delivered in between.

Calling `super.onLongPressStart(event)` sets `isLongPressing` to `true`.


### onLongPressMoveUpdate

Fires continuously as the user moves their finger during an active long press. This event is only
delivered to components that received the initial `onLongPressStart`.

The `LongPressMoveUpdateEvent` is a `DisplacementEvent` that provides a frame-to-frame delta,
just like `DragUpdateEvent`. That means you can use `localDelta` to move a component following
the pointer (it correctly accounts for camera zoom and component transforms).
The event also carries `offsetFromOrigin` for the total displacement since the gesture started.


### onLongPressEnd

Fires when the user lifts their pointer after a long press. The `LongPressEndEvent` includes the
final position and the `velocity` of the pointer at the moment of release.

Calling `super.onLongPressEnd(event)` sets `isLongPressing` to `false`.


### onLongPressCancel

Fires if the gesture is interrupted before completing (e.g. by a competing gesture recognizer).

Calling `super.onLongPressCancel(event)` sets `isLongPressing` to `false`.


## Mixins


### LongPressCallbacks

The `LongPressCallbacks` mixin can be added to any `Component` for that component to start
receiving long press events.

This mixin adds methods `onLongPressStart`, `onLongPressMoveUpdate`, `onLongPressEnd`, and
`onLongPressCancel` to the component. Override them to implement behavior.

A component will only receive long press events that originate *within* that component, as judged
by the `containsLocalPoint()` function. The commonly-used `PositionComponent` class provides such
an implementation based on its `size` property.

The mixin also provides an `isLongPressing` property that tracks whether a long press gesture
is currently active on the component. This is managed automatically when you call `super` in
`onLongPressStart`, `onLongPressEnd`, and `onLongPressCancel`.

```dart
class LongPressSquare extends RectangleComponent with LongPressCallbacks {
  @override
  void onLongPressStart(LongPressStartEvent event) {
    super.onLongPressStart(event);
    paint.color = Colors.red;
  }

  @override
  void onLongPressMoveUpdate(LongPressMoveUpdateEvent event) {
    position += event.localDelta;
  }

  @override
  void onLongPressEnd(LongPressEndEvent event) {
    super.onLongPressEnd(event);
    paint.color = Colors.blue;
  }
}
```
