# Функциональный эффект

Класс `FunctionEffect` — это очень обобщённый эффект, который позволяет делать практически всё что угодно без необходимости определять новый эффект.

Он выполняет функцию, которая получает цель и прогресс эффекта, а затем пользователь сам решает, что делать с этими входными данными.

Это можно использовать, например, для изменения состояния игры, которое происходит с течением времени, но не обязательно является визуальным, в отличие от большинства других эффектов.

В следующем примере у нас есть перечисление `PlayerState`, которое мы хотим изменять с течением времени. Мы хотим переключить состояние на `yawn`, когда прогресс превысит 50%, а затем обратно на `idle`, когда прогресс превысит 80%.

```dart
enum PlayerState {
  idle,
  yawn,
}

final effect = FunctionEffect<SpriteAnimationGroupComponent<PlayerState>>(
  (target, progress) {
    if (progress > 0.5) {
      target.current = PlayerState.yawn;
    } else if(progress > 0.8) {
      target.current = PlayerState.idle;
    }
  },
  EffectController(
    duration: 10,
    infinite: true,
  ),
);
```# Function Effect

The `FunctionEffect` class is a very generic Effect that allows you to do almost anything without
having to define a new effect.

It runs a function that takes the target and the progress of the effect and then the user can
decide what to do with that input.

This could for example be used to make game state changes that happen over time, but that isn't
necessarily visual, like most other effects are.

In the following example we have a `PlayerState` enum that we want to change over time. We want to
change the state to `yawn` when the progress is over 50% and then back to `idle` when the progress
is over 80%.

```dart
enum PlayerState {
  idle,
  yawn,
}

final effect = FunctionEffect<SpriteAnimationGroupComponent<PlayerState>>(
  (target, progress) {
    if (progress > 0.5) {
      target.current = PlayerState.yawn;
    } else if(progress > 0.8) {
      target.current = PlayerState.idle;
    }
  },
  EffectController(
    duration: 10,
    infinite: true,
  ),
);
```
