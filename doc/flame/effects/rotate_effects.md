# Эффекты вращения

Эффекты вращения используются для изменения ориентации компонента с течением времени. С их помощью можно заставить компонент вращаться, поворачиваться к цели или вращаться вокруг точки. Угол поворота задаётся в радианах, и эффекты могут применяться к любому компоненту, имеющему свойство вращения, например `PositionComponent`.


## `RotateEffect.by`

Поворачивает цель по часовой стрелке на указанный угол относительно её текущей ориентации. Угол задаётся в радианах. Например, следующий эффект повернёт цель на 90º (= [tau]/4 радиан) по часовой стрелке:

```{flutter-app}
:sources: ../flame/examples
:page: rotate_by_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = RotateEffect.by(
  tau/4,
  EffectController(duration: 2),
);
```


## `RotateEffect.to`

Поворачивает цель по часовой стрелке до указанного угла. Например, следующий код повернёт цель в направлении на восток (0º — север, 90º = [tau]/4 — восток, 180º = tau/2 — юг, 270º = tau*3/4 — запад):

```{flutter-app}
:sources: ../flame/examples
:page: rotate_to_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = RotateEffect.to(
  tau/4,
  EffectController(duration: 2),
);
```


## `RotateAroundEffect`

Поворачивает цель по часовой стрелке на указанный угол относительно её текущей ориентации вокруг заданного центра. Угол в радианах. Например, следующий эффект повернёт цель на 90º (= [tau]/4 радиан) по часовой стрелке вокруг точки (100, 100).

```{flutter-app}
:sources: ../flame/examples
:page: rotate_around_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = RotateAroundEffect(
  tau/4,
  center: Vector2(100, 100),
  EffectController(duration: 2),
);
```

[tau]: https://en.wikipedia.org/wiki/Tau_(mathematical_constant)
