# Комбинированный эффект

Этот эффект можно использовать для одновременного запуска нескольких других эффектов.

Комбинированный эффект также может быть попеременным (последовательность сначала будет выполняться вперёд, а затем назад), а также повторяться заданное количество раз или бесконечно.

```{flutter-app}
:sources: ../flame/examples
:page: combined_effect
:show: widget code infobox
:width: 450
:height: 350
```

```dart
final effect = CombinedEffect(
  [
    MoveEffect.by(Vector2(200, 0), EffectController(duration: 1)),
    RotateEffect.by(tau / 4, EffectController(duration: 2)),
    ScaleEffect.by(Vector2.all(1.5), EffectController(duration: 1)),
  ],
  alternate: true,
  infinite: true,
);
```
