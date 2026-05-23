# Эффекты якоря

Эффекты якоря используются для изменения якорной точки компонента с течением времени. Якорная точка — это точка, вокруг которой компонент вращается и масштабируется.


## `AnchorByEffect`

Изменяет положение якоря цели на указанное смещение. Этот эффект также можно создать с помощью `AnchorEffect.by()`.

```{flutter-app}
:sources: ../flame/examples
:page: anchor_by_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = AnchorByEffect(
  Vector2(0.1, 0.1),
  EffectController(speed: 1),
);
```


## `AnchorToEffect`

Изменяет положение якоря цели. Этот эффект также можно создать с помощью `AnchorEffect.to()`.

```{flutter-app}
:sources: ../flame/examples
:page: anchor_to_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = AnchorToEffect(
  Anchor.center,
  EffectController(speed: 1),
);
```
