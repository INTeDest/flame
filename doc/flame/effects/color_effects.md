# Эффекты цвета

Цветовые эффекты используются для изменения цвета компонента с течением времени. Их можно применять для тонирования компонента, изменения его непрозрачности или наложения цветового фильтра.


## ColorEffect

Этот эффект изменяет базовый цвет краски, в результате чего отображаемый компонент тонируется заданным цветом в указанном диапазоне.

Пример использования:

```{flutter-app}
:sources: ../flame/examples
:page: color_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = ColorEffect(
  const Color(0xFF00FF00),
  EffectController(duration: 1.5),
  opacityFrom: 0.2,
  opacityTo: 0.8,
);
```

Аргументы `opacityFrom` и `opacityTo` определяют, «насколько сильно» цвет будет применён к компоненту. В этом примере эффект начнётся с 20% и дойдёт до 80%.

**Примечание:** Из-за особенностей реализации этого эффекта и того, как работает класс `ColorFilter` во Flutter, данный эффект нельзя комбинировать с другими `ColorEffect`. Если к компоненту добавлено более одного такого эффекта, сработает только последний.


## `OpacityToEffect`

Этот эффект изменяет непрозрачность цели с течением времени до указанного значения альфа-канала. Он может применяться только к компонентам, реализующим `OpacityProvider`.

```{flutter-app}
:sources: ../flame/examples
:page: opacity_to_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = OpacityEffect.to(
  0.2,
  EffectController(duration: 0.75),
);
```

Если компонент использует несколько красок (paints), эффект можно направить на одну или несколько из них с помощью параметра `target`. Примесь `HasPaint` реализует `OpacityProvider` и предоставляет API для лёгкого создания провайдеров для нужных идентификаторов красок. Для одного `paintId` можно использовать `opacityProviderOf`, а для нескольких — `opacityProviderOfList`.

```{flutter-app}
:sources: ../flame/examples
:page: opacity_effect_with_target
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = OpacityEffect.to(
  0.2,
  EffectController(duration: 0.75),
  target: component.opacityProviderOfList(
    paintIds: const [paintId1, paintId2],
  ),
);
```

Значение непрозрачности 0 соответствует полностью прозрачному компоненту, а 1 — полностью непрозрачному. Удобные конструкторы `OpacityEffect.fadeOut()` и `OpacityEffect.fadeIn()` анимируют цель соответственно до полной прозрачности или полной видимости.


## `OpacityByEffect`

Этот эффект изменяет непрозрачность цели относительно указанного значения альфа-канала. Например, следующий эффект изменит непрозрачность цели на `90%`:

```{flutter-app}
:sources: ../flame/examples
:page: opacity_by_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = OpacityEffect.by(
  0.9,
  EffectController(duration: 0.75),
);
```

В настоящее время этот эффект может применяться только к компонентам с примесью `HasPaint`. Если целевой компонент использует несколько красок, эффект можно направить на любой отдельный цвет с помощью параметра `paintId`.


## GlowEffect

```{note}
Этот эффект пока экспериментальный, и его API может измениться в будущем.
```

Этот эффект накладывает светящуюся тень вокруг цели в соответствии с указанной `glow-strength` (силой свечения). Цвет тени будет соответствовать цвету краски цели. Например, следующий эффект применит светящуюся тень вокруг цели с силой `10`:

```{flutter-app}
:sources: ../flame/examples
:page: glow_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = GlowEffect(
  10.0,
  EffectController(duration: 3),
);
```

В настоящее время этот эффект может применяться только к компонентам с примесью `HasPaint`.


## `HueToEffect`

Этот эффект изменяет оттенок (hue) цели с течением времени до указанного угла в радианах. Он может применяться только к компонентам, реализующим `HueProvider`.

```dart
final effect = HueEffect.to(
  pi / 2,
  EffectController(duration: 3),
);
```


## `HueByEffect`

Этот эффект поворачивает оттенок цели на указанный угол в радианах. Он может применяться только к компонентам, реализующим `HueProvider`.

```{flutter-app}
:sources: ../flame/examples
:page: hue_effect
:show: widget code infobox
:width: 180
:height: 160
```

```dart
final effect = HueEffect.by(
  2 * pi,
  EffectController(duration: 3),
);
```

Оба эффекта могут быть нацелены на любой компонент, реализующий `HueProvider`. Примесь `HasPaint` реализует `HueProvider` и автоматически обрабатывает необходимые обновления `ColorFilter`.

> [!TIP]
> **Замечание о производительности**: `HueEffect` чрезвычайно эффективен, поскольку изменяет `colorFilter` объекта `Paint` напрямую. Если у вас много компонентов, предпочтите этот эффект вместо `HueDecorator`, который использует `saveLayer()` и создаёт гораздо большую нагрузку.
