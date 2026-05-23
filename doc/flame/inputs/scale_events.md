# События масштабирования

**События масштабирования** происходят, когда пользователь выполняет жест сведения или разведения двух пальцев. Одновременно может происходить только один жест масштабирования.

Для компонентов, которые должны реагировать на события масштабирования, добавьте примесь `ScaleCallbacks`.

- Эта примесь добавляет в компонент три переопределяемых метода: `onScaleStart`, `onScaleUpdate`, `onScaleEnd`. По умолчанию эти методы ничего не делают; для выполнения каких-либо действий их необходимо переопределить.
- Кроме того, компонент должен реализовывать метод `containsLocalPoint()` (уже реализован в `PositionComponent`, поэтому в большинстве случаев ничего делать не нужно). Этот метод позволяет Flame определить, произошло ли событие внутри компонента или нет.

```dart
class MyComponent extends PositionComponent with ScaleCallbacks {
  MyComponent() : super(size: Vector2(180, 120));

   @override
   void onScaleStart(ScaleStartEvent event) {
     // Выполнить действие в ответ на событие масштабирования
   }
}
```

## Анатомия масштабирования

### onScaleStart

Это первое событие в последовательности масштабирования. Обычно событие доставляется самому верхнему компоненту в фокальной точке (точка в центре линии, образованной двумя пальцами), имеющему примесь `ScaleCallbacks`. Однако, установив флаг `event.continuePropagation` в true, вы можете разрешить событию распространяться на нижележащие компоненты.

Объект `ScaleStartEvent`, связанный с этим событием, содержит координаты первой фокальной точки, распознанной распознавателем жеста масштабирования. Эта точка доступна в нескольких системах координат: `devicePosition` задана в системе координат всего устройства, `canvasPosition` — в системе координат игрового виджета, а `localPosition` предоставляет позицию в локальной системе координат компонента.

Любой компонент, получивший `onScaleStart`, впоследствии также будет получать события `onScaleUpdate` и `onScaleEnd`.

### onScaleUpdate

Это событие генерируется непрерывно, пока пользователь перемещает пальцы по экрану. Оно не генерируется, если пользователь держит пальцы неподвижно.

Реализация по умолчанию доставляет это событие всем компонентам, которые получили предыдущий `onScaleStart`. Если точка касания всё ещё находится внутри компонента, то `event.localPosition` вернёт позицию этой точки в локальной системе координат. Однако, если пользователь уводит пальцы за пределы компонента, свойство `event.localPosition` вернёт точку с координатами NaN. Аналогично, `event.renderingTrace` в этом случае будет пустым. Однако свойства `canvasPosition` и `devicePosition` события останутся действительными.

Кроме того, `ScaleUpdateEvent` содержит `focalPointDelta` — величину перемещения фокальной точки с момента предыдущего `onScaleUpdate` или с момента `onScaleStart`, если это первое обновление масштабирования после его начала.

Свойство `event.timestamp` измеряет время, прошедшее с начала масштабирования. Его можно использовать, например, для вычисления скорости движения.

Свойство `event.rotation` измеряет угол поворота в радианах между линией, образованной двумя пальцами в начале, и линией, образованной на момент вызова этого события.

Свойство `event.scale` измеряет отношение длин линии между двумя пальцами в начале и линии, образованной на момент вызова этого события.

### onScaleEnd

Это событие возникает, когда пользователь поднимает пальцы и, таким образом, завершает жест масштабирования. С этим событием не связана никакая позиция.

## Примеси

### ScaleCallbacks

Примесь `ScaleCallbacks` может быть добавлена к любому `Component`, чтобы этот компонент начал получать события масштабирования.

Эта примесь добавляет в компонент методы `onScaleStart`, `onScaleUpdate`, `onScaleEnd`, которые по умолчанию ничего не делают, но могут быть переопределены для реализации реальной функциональности.

Ещё одна важная деталь: компонент будет получать только те события масштабирования, которые начинаются *внутри* этого компонента, что определяется функцией `containsLocalPoint()`. Широко используемый класс `PositionComponent` предоставляет такую реализацию на основе своего свойства `size`. Таким образом, если ваш компонент наследуется от `PositionComponent`, убедитесь, что правильно задали его размер. Если же ваш компонент наследуется от чистого `Component`, то метод `containsLocalPoint()` необходимо реализовать вручную.

Если ваш компонент является частью более крупной иерархии, он будет получать события масштабирования только в том случае, если все его предки корректно реализовали `containsLocalPoint`.

### isScaling

Примесь `ScaleCallbacks` предоставляет геттер `isScaling`, который возвращает `true`, пока компонент активно масштабируется. Он устанавливается в `true` в начале `onScaleStart` и обратно в `false` при `onScaleEnd`. Его можно использовать, например, для изменения внешнего вида компонента во время жеста масштабирования.

### scaleThreshold

События масштабирования не срабатывают немедленно при касании экрана двумя пальцами. Сначала должно быть преодолено небольшое пороговое значение движения. По умолчанию пальцы должны разойтись или сойтись минимум на 5% (коэффициент масштабирования 1.05), прежде чем будет вызван `onScaleStart`. Это предотвращает случайные жесты масштабирования, когда пользователь просто кладёт два пальца без намерения масштабировать.

Порог можно изменить, получив доступ к `MultiDragScaleDispatcher` из вашей игры и установив `scaleThreshold` до монтирования любого компонента с `ScaleCallbacks`:

```dart
class MyGame extends FlameGame {
  @override
  Future<void> onLoad() async {
    final dispatcher = MultiDragScaleDispatcher()..scaleThreshold = 1.02;
    registerKey(const MultiDragScaleDispatcherKey(), dispatcher);
    add(dispatcher);
  }
}
```

Меньшее значение делает распознаватель более чувствительным (реагирует на меньшие движения пальцев), в то время как большее значение требует более осознанного жеста для вызова событий масштабирования.

## Совмещение с DragCallbacks

Компонент может одновременно использовать и `ScaleCallbacks`, и `DragCallbacks`. Когда присутствуют обе примеси, жесты одним пальцем порождают события перетаскивания, а жесты двумя пальцами — как масштабирования, так и перетаскивания. Это полезно для компонентов, которые должны перетаскиваться одним пальцем и масштабироваться щипком или вращаться двумя пальцами.

```dart
class InteractiveRectangle extends RectangleComponent
    with ScaleCallbacks, DragCallbacks {

  double _initialAngle = 0;

  @override
  void onDragUpdate(DragUpdateEvent event) {
    position += event.localDelta;
  }

  @override
  void onScaleStart(ScaleStartEvent event) {
    super.onScaleStart(event);
    _initialAngle = angle;
  }

  @override
  void onScaleUpdate(ScaleUpdateEvent event) {
    angle = _initialAngle + event.rotation;
  }
}
```# Scale Events

**Scale events** occur when the user moves two fingers in a pinch in, or in a pinch out move.
Only one single scale gesture can occur at the same time.

For those components that you want to respond to scale events, add the `ScaleCallbacks` mixin.

- This mixin adds three overridable methods to your component: `onScaleStart`, `onScaleUpdate`,
  `onScaleEnd`. By default, these methods do nothing; they need to be overridden in order to
  perform any function.
- In addition, the component must implement the `containsLocalPoint()` method (already implemented
  in `PositionComponent`, so most of the time you don't need to do anything here). This method
  allows Flame to know whether the event occurred within the component or not.

```dart
class MyComponent extends PositionComponent with ScaleCallbacks {
  MyComponent() : super(size: Vector2(180, 120));

   @override
   void onScaleStart(ScaleStartEvent event) {
     // Do something in response to a scale event
   }
}
```


## Scale anatomy


### onScaleStart

This is the first event that occurs in a scale sequence. Usually, the event will be delivered to the
topmost component at the focal point (the point at the center of the line formed by the two fingers)
 with the `ScaleCallbacks` mixin. However, by setting the flag
`event.continuePropagation` to true, you can allow the event to propagate to the components below.

The `ScaleStartEvent` object associated with this event will contain
the coordinate of the first focal point
recognized by the scale gesture recognizer. This point is available in multiple coordinate system:
`devicePosition` is given in the coordinate system of the entire device, `canvasPosition` is in the
coordinate system of the game widget, and `localPosition` provides the position in the component's
local coordinate system.

Any component that receives `onScaleStart` will later be receiving `onScaleUpdate` and `onScaleEnd`
events as well.


### onScaleUpdate

This event is fired continuously as user drags their finger across the screen. It will not fire if
the user is holding their finger still.

The default implementation delivers this event to all the components that received the previous
`onScaleStart`. If the point of touch is still within the component, then
`event.localPosition` will give the position of that point in the local coordinate system. However,
if the user moves their finger away from the component, the property `event.localPosition` will
return a point whose coordinates are NaNs. Likewise, the `event.renderingTrace` in this case will be
empty. However, the `canvasPosition` and `devicePosition` properties of the event will be valid.

In addition, the `ScaleUpdateEvent` will contain `focalPointDelta` --
the amount the focal point has moved since the
previous `onScaleUpdate`, or since the `onScaleStart` if this is the first scale-update after a scale-
start.

The `event.timestamp` property measures the time elapsed since the beginning of the scale. It can be
used, for example, to compute the speed of the movement.

The `event.rotation` property measures the angle of rotation in radians, between the line formed
from the two fingers at the start, and the line formed when this event is called.

The `event.scale` property measures the ratio of length between the line formed
from the two fingers at the start, and the line formed when this event is called.


### onScaleEnd

This event is fired when the user lifts their finger and thus stops the scale gesture. There is no
position associated with this event.


## Mixins


### ScaleCallbacks

The `ScaleCallbacks` mixin can be added to any `Component` in order for that component to start
receiving scale events.

This mixin adds methods `onScaleStart`, `onScaleUpdate`, `onScaleEnd` to the
component, which by default don't do anything, but can be overridden to implement any real
functionality.

Another crucial detail is that a component will only receive scale events that originate *within*
that component, as judged by the `containsLocalPoint()` function. The commonly-used
`PositionComponent` class provides such an implementation based on its `size` property. Thus, if
your component derives from a `PositionComponent`, then make sure that you set its size correctly.
If, however, your component derives from the bare `Component`, then the `containsLocalPoint()`
method must be implemented manually.

If your component is a part of a larger hierarchy, then it will only receive scale events if its
ancestors have all implemented the `containsLocalPoint` correctly.


### isScaling

The `ScaleCallbacks` mixin provides an `isScaling` getter that returns `true` while the component is
actively being scaled. This is set to `true` at the start of `onScaleStart` and back to `false` at
`onScaleEnd`. It can be used, for example, to change the component's visual appearance during a scale
gesture.


### scaleThreshold

Scale events are not fired immediately when two fingers touch the screen. Instead, a small movement
threshold must be crossed first. By default, the fingers must spread or pinch by at least 5% (a
scale factor of 1.05) before `onScaleStart` is called. This prevents accidental scale gestures when
the user simply places two fingers without intending to scale.

The threshold can be changed by accessing the `MultiDragScaleDispatcher` from your game and setting
`scaleThreshold` before any `ScaleCallbacks` component mounts:

```dart
class MyGame extends FlameGame {
  @override
  Future<void> onLoad() async {
    final dispatcher = MultiDragScaleDispatcher()..scaleThreshold = 1.02;
    registerKey(const MultiDragScaleDispatcherKey(), dispatcher);
    add(dispatcher);
  }
}
```

A lower value makes the recognizer more sensitive (reacts to smaller pinch movements), while a
higher value requires a more deliberate gesture before scale events fire.


## Combining with DragCallbacks

A component can use both `ScaleCallbacks` and `DragCallbacks` at the same time. When both mixins are
present, single-finger gestures produce drag events and two-finger gestures produce both scale and
drag events. This is useful for components that should be draggable with one finger and
pinch-to-zoom or rotatable with two fingers.

```dart
class InteractiveRectangle extends RectangleComponent
    with ScaleCallbacks, DragCallbacks {

  double _initialAngle = 0;

  @override
  void onDragUpdate(DragUpdateEvent event) {
    position += event.localDelta;
  }

  @override
  void onScaleStart(ScaleStartEvent event) {
    super.onScaleStart(event);
    _initialAngle = angle;
  }

  @override
  void onScaleUpdate(ScaleUpdateEvent event) {
    angle = _initialAngle + event.rotation;
  }
}
```
