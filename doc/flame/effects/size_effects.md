# Эффекты размера

Эффекты размера используются для изменения размера компонента с течением времени. Их можно применять, чтобы заставить компонент расти, сжиматься или изменять размер в определённом направлении. Размер задаётся значением `Vector2`, где x представляет ширину, а y — высоту. Эффекты можно применять к любому компоненту, реализующему интерфейс `SizeProvider`, например к `PositionComponent`. Разница между эффектами размера и масштабирования в том, что размер изменяет только размер целевого компонента, тогда как масштаб изменяет «размер» также всех его дочерних элементов.


## `SizeEffect.by`

Этот эффект изменяет размер целевого компонента относительно его текущего размера. Например, если цель имеет размер `Vector2(100, 100)`, то после применения и завершения следующего эффекта новый размер станет `Vector2(120, 50)`:

```{flutter-app}
:sources: ../flame/examples
:page: size_by_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = SizeEffect.by(
   Vector2(-15, 30),
   EffectController(duration: 1),
);
```

Размер `PositionComponent` не может быть отрицательным. Если эффект попытается установить отрицательный размер, он будет ограничен нулём.

Обратите внимание, что для работы этого эффекта целевой компонент должен реализовывать интерфейс `SizeProvider` и учитывать свой `size` при рендеринге. Лишь немногие встроенные компоненты реализуют этот API, но вы всегда можете заставить свой собственный компонент работать с эффектами размера, добавив `implements SizeEffect` в объявление класса.

Альтернативой `SizeEffect` является `ScaleEffect`, который работает более обобщённо и масштабирует как целевой компонент, так и его дочерние элементы.


## `SizeEffect.to`

Изменяет размер целевого компонента до указанного. Целевой размер не может быть отрицательным:

```{flutter-app}
:sources: ../flame/examples
:page: size_to_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = SizeEffect.to(
  Vector2(90, 80),
  EffectController(duration: 1),
);
```# Size Effects

Size effects are used to change the size of a component over time. They can be used to make a
component grow, shrink, or change its size in a specific direction. The size is specified as a
`Vector2` value, where x represents the width and y represents the height. The effects can be
applied to any component that implements the `SizeProvider` interface, such as the
`PositionComponent`. The difference between size effects and scale effects is that size only
changes the size of the target component, while scale changes the "size" of all children too.


## `SizeEffect.by`

This effect will change the size of the target component, relative to its current size. For example,
if the target has size `Vector2(100, 100)`, then after the following effect is applied and runs its
course, the new size will be `Vector2(120, 50)`:

 ```{flutter-app}
 :sources: ../flame/examples
 :page: size_by_effect
 :show: widget code infobox
 :width: 180
 :height: 160
 ```

```dart
final effect = SizeEffect.by(
   Vector2(-15, 30),
   EffectController(duration: 1),
);
```

The size of a `PositionComponent` cannot be negative. If an effect attempts to set the size to a
negative value, the size will be clamped at zero.

Note that for this effect to work, the target component must implement the `SizeProvider` interface
and take its `size` into account when rendering. Only few of the built-in components implement this
API, but you can always make your own component work with size effects by adding
`implements SizeEffect` to the class declaration.

An alternative to `SizeEffect` is the `ScaleEffect`, which works more generally and scales both the
target component and its children.


## `SizeEffect.to`

Changes the size of the target component to the specified size. Target size cannot be negative:


 ```{flutter-app}
 :sources: ../flame/examples
 :page: size_to_effect
 :show: widget code infobox
 :width: 180
 :height: 160
 ```

```dart
final effect = SizeEffect.to(
  Vector2(90, 80),
  EffectController(duration: 1),
);
```
