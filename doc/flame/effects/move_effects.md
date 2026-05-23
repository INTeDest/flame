# Эффекты перемещения

Эффекты перемещения — это особый тип эффектов, изменяющих положение компонента с течением времени. Если нужно, например, переместить персонажа из одной точки в другую, заставить его прыгать или следовать по пути, можно использовать один из предопределённых эффектов перемещения.


## `MoveByEffect`

Этот эффект применяется к `PositionComponent` и смещает его на заданную величину `offset`. Это смещение относительно текущей позиции цели:

```{flutter-app}
:sources: ../flame/examples
:page: move_by_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = MoveByEffect(
  Vector2(0, -10),
  EffectController(duration: 0.5),
);
```

Если компонент сейчас находится в `Vector2(250, 200)`, то в конце эффекта его позиция будет `Vector2(250, 190)`.

К компоненту можно одновременно применить несколько эффектов перемещения. Результатом будет суперпозиция всех отдельных эффектов.


## `MoveToEffect`

Этот эффект перемещает `PositionComponent` из текущей позиции в указанную точку назначения по прямой линии.

```{flutter-app}
:sources: ../flame/examples
:page: move_to_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = MoveToEffect(
  Vector2(100, 500),
  EffectController(duration: 3),
);
```

Применять несколько таких эффектов к одному компоненту можно, но не рекомендуется.


## `MoveAlongPathEffect`

Этот эффект перемещает `PositionComponent` вдоль указанного пути относительно текущей позиции компонента. Путь может содержать нелинейные сегменты, но должен быть односвязным. Рекомендуется начинать путь с `Vector2.zero()`, чтобы избежать резких скачков позиции компонента.

```{flutter-app}
:sources: ../flame/examples
:page: move_along_path_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = MoveAlongPathEffect(
  Path()..quadraticBezierTo(100, 0, 50, -50),
  EffectController(duration: 1.5),
);
```

Необязательный флаг `absolute: true` объявляет путь в эффекте абсолютным. То есть цель «перепрыгнет» в начало пути при старте и затем будет следовать по этому пути, как если бы это была кривая, нарисованная на холсте.

Ещё один флаг `oriented: true` предписывает цели не только двигаться по кривой, но и поворачиваться в направлении, куда кривая направлена в каждой точке. С этим флагом эффект одновременно становится и эффектом перемещения, и эффектом вращения.# Move Effects

Move Effects are a special type of effects that modify the position of a component over time, if
you want to for example move your character from one point to another, make it jump, or follow a
path, then you can use one of the predefined move effects.


## `MoveByEffect`

This effect applies to a `PositionComponent` and shifts it by a prescribed `offset` amount. This
offset is relative to the current position of the target:

```{flutter-app}
:sources: ../flame/examples
:page: move_by_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = MoveByEffect(
  Vector2(0, -10),
  EffectController(duration: 0.5),
);
```

If the component is currently at `Vector2(250, 200)`, then at the end of the effect its position
will be `Vector2(250, 190)`.

Multiple move effects can be applied to a component at the same time. The result will be the
superposition of all the individual effects.


## `MoveToEffect`

This effect moves a `PositionComponent` from its current position to the specified destination
point in a straight line.

```{flutter-app}
:sources: ../flame/examples
:page: move_to_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = MoveToEffect(
  Vector2(100, 500),
  EffectController(duration: 3),
);
```

It is possible, but not recommended to attach multiple such effects to the same component.


## `MoveAlongPathEffect`

This effect moves a `PositionComponent` along the specified path relative to the component's
current position. The path can have non-linear segments, but must be singly connected. It is
recommended to start a path at `Vector2.zero()` in order to avoid sudden jumps in the component's
position.

```{flutter-app}
:sources: ../flame/examples
:page: move_along_path_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = MoveAlongPathEffect(
  Path()..quadraticBezierTo(100, 0, 50, -50),
  EffectController(duration: 1.5),
);
```

An optional flag `absolute: true` will declare the path within the effect as absolute. That is, the
target will "jump" to the beginning of the path at start, and then follow that path as if it was a
curve drawn on the canvas.

Another flag `oriented: true` instructs the target not only move along the curve, but also rotate
itself in the direction the curve is facing at each point. With this flag the effect becomes both
the move- and the rotate- effect at the same time.
