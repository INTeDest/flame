# Спрайтовые компоненты

Спрайты — это двухмерные изображения (или области изображений), представляющие визуальное представление игровых объектов. Это наиболее распространённый способ отображения персонажей, предметов, фонов и других визуальных элементов в двухмерных играх. Flame предоставляет несколько компонентов на основе спрайтов, упрощающих загрузку изображений, воспроизведение анимаций и переключение между визуальными состояниями, используя при этом свойства трансформации, унаследованные от `PositionComponent`.


## SpriteComponent

Наиболее часто используемой реализацией `PositionComponent` является `SpriteComponent`, который можно создать с помощью `Sprite`:

```dart
import 'package:flame/components/component.dart';

class MyGame extends FlameGame {
  late final SpriteComponent player;

  @override
  Future<void> onLoad() async {
    final sprite = await Sprite.load('player.png');
    final size = Vector2.all(128.0);
    final player = SpriteComponent(size: size, sprite: sprite);

    // По умолчанию Vector2(0.0, 0.0), также можно задать в конструкторе
    player.position = Vector2(10, 20);

    // По умолчанию 0, также можно задать в конструкторе
    player.angle = 0;

    // Добавляет компонент
    add(player);
  }
}
```


## SpriteAnimationComponent

Этот класс используется для представления компонента, содержащего спрайты, объединённые в одну циклическую анимацию.

Пример создания простой трёхкадровой анимации с использованием трёх разных изображений:

```dart
@override
Future<void> onLoad() async {
  final sprites = [0, 1, 2]
      .map((i) => Sprite.load('player_$i.png'));
  final animation = SpriteAnimation.spriteList(
    await Future.wait(sprites),
    stepTime: 0.01,
  );
  this.player = SpriteAnimationComponent(
    animation: animation,
    size: Vector2.all(64.0),
  );
}
```

Если у вас есть спрайт-лист, можно использовать конструктор `sequenced` из класса `SpriteAnimationData` (подробнее см. [Изображения > Анимация](../rendering/images.md#animation)):

```dart
@override
Future<void> onLoad() async {
  final size = Vector2.all(64.0);
  final data = SpriteAnimationData.sequenced(
    textureSize: size,
    amount: 2,
    stepTime: 0.1,
  );
  this.player = SpriteAnimationComponent.fromFrameData(
    await images.load('player.png'),
    data,
  );
}
```

Все компоненты анимации внутренне поддерживают `SpriteAnimationTicker`, который тикает `SpriteAnimation`. Это позволяет нескольким компонентам использовать один и тот же объект анимации.

Пример:

```dart
final sprites = [/* Ваш список спрайтов */];
final animation = SpriteAnimation.spriteList(sprites, stepTime: 0.01);

final animationTicker = SpriteAnimationTicker(animation);

// или можно попросить объект анимации создать тикер для вас
final animationTicker = animation.createTicker(); // создаёт новый тикер

animationTicker.update(dt);
```

Чтобы дождаться окончания анимации (когда она достигнет последнего кадра и не зациклена), используйте `animationTicker.completed`.

Пример:

```dart
await animationTicker.completed;

doSomething();

// или

animationTicker.completed.whenComplete(doSomething);
```

Кроме того, `SpriteAnimationTicker` имеет следующие необязательные обратные вызовы событий: `onStart`, `onFrame` и `onComplete`. Подписаться на них можно так:

```dart
final animationTicker = SpriteAnimationTicker(animation)
  ..onStart = () {
    // Выполнить действие при старте.
  };

final animationTicker = SpriteAnimationTicker(animation)
  ..onComplete = () {
    // Выполнить действие при завершении.
  };

final animationTicker = SpriteAnimationTicker(animation)
  ..onFrame = (index) {
    if (index == 1) {
      // Выполнить действие для второго кадра.
    }
  };
```

Чтобы сбросить анимацию на первый кадр при удалении компонента, установите `resetOnRemove` в `true`:

```dart
SpriteAnimationComponent(
  animation: animation,
  size: Vector2.all(64.0),
  resetOnRemove: true,
);
```


## SpriteAnimationGroupComponent

`SpriteAnimationGroupComponent` — это простая обёртка над `SpriteAnimationComponent`, позволяющая вашему компоненту содержать несколько анимаций и переключать текущую проигрываемую анимацию во время выполнения. Поскольку этот компонент является обёрткой, прослушивание событий реализуется так же, как описано в разделе [SpriteAnimationComponent](#spriteanimationcomponent).

Использование очень похоже на `SpriteAnimationComponent`, но вместо инициализации одной анимацией этот компонент получает `Map` с ключом обобщённого типа `T` и значением `SpriteAnimation`, а также текущую анимацию.

Пример:

```dart
enum RobotState {
  idle,
  running,
}

final running = await loadSpriteAnimation(/* опущено */);
final idle = await loadSpriteAnimation(/* опущено */);

final robot = SpriteAnimationGroupComponent<RobotState>(
  animations: {
    RobotState.running: running,
    RobotState.idle: idle,
  },
  current: RobotState.idle,
);

// Переключение текущей анимации на "running"
robot.current = RobotState.running;
```

Поскольку компонент работает с несколькими `SpriteAnimation`, ему требуется соответствующее количество тикеров для их обновления. Используйте геттер `animationsTickers` для доступа к карте, содержащей тикеры для каждого состояния анимации. Это полезно, если нужно зарегистрировать обратные вызовы `onStart`, `onComplete` и `onFrame`.

Пример:

```dart
enum RobotState { idle, running, jump }

final running = await loadSpriteAnimation(/* опущено */);
final idle = await loadSpriteAnimation(/* опущено */);

final robot = SpriteAnimationGroupComponent<RobotState>(
  animations: {
    RobotState.running: running,
    RobotState.idle: idle,
  },
  current: RobotState.idle,
);

robot.animationTickers?[RobotState.running]?.onStart = () {
  // Действие при старте анимации бега.
};

robot.animationTickers?[RobotState.jump]?.onStart = () {
  // Действие при старте анимации прыжка.
};

robot.animationTickers?[RobotState.jump]?.onComplete = () {
  // Действие при завершении анимации прыжка.
};

robot.animationTickers?[RobotState.idle]?.onFrame = (currentIndex) {
  // Действие в зависимости от текущего индекса кадра анимации покоя.
};
```


## SpriteGroupComponent

`SpriteGroupComponent` очень похож на свой анимационный аналог, но предназначен специально для спрайтов.

Пример:

```dart
class PlayerComponent extends SpriteGroupComponent<ButtonState>
    with HasGameReference<SpriteGroupExample>, TapCallbacks {
  @override
  Future<void> onLoad() async {
    final pressedSprite = await game.loadSprite(/* опущено */);
    final unpressedSprite = await game.loadSprite(/* опущено */);

    sprites = {
      ButtonState.pressed: pressedSprite,
      ButtonState.unpressed: unpressedSprite,
    };

    current = ButtonState.unpressed;
  }

  // обработчики касаний опущены...
}
```


## IconComponent

`IconComponent` отображает Flutter `IconData` (например, `Icons.star`) в виде компонента Flame. Иконка растеризуется в изображение один раз во время `onLoad()`, а затем каждый кадр рисуется с помощью `canvas.drawImageRect()` с использованием `Paint` компонента. Поскольку иконка визуализируется как кешированное изображение, а не текст, все эффекты на основе `Paint` работают «из коробки», включая `tint()`, `setOpacity()`, `ColorEffect`, `OpacityEffect`, `GlowEffect` и пользовательские `ColorFilter`.


### Базовое использование

```dart
import 'package:flame/components.dart';
import 'package:flutter/material.dart';

class MyGame extends FlameGame {
  @override
  Future<void> onLoad() async {
    final star = IconComponent(
      icon: Icons.star,
      iconSize: 64,
      position: Vector2(100, 100),
    );
    add(star);
  }
}
```


### Тонирование и эффекты

Иконка растеризуется белым цветом, что позволяет тонировать её в любой цвет с помощью методов `HasPaint`:

```dart
// Тонирование иконки в золотой цвет
final star = IconComponent(
  icon: Icons.star,
  iconSize: 64,
  position: Vector2(100, 100),
)..tint(const Color(0xFFFFD700));

// Установка непрозрачности
star.setOpacity(0.5);

// Или использование пользовательской краски
final icon = IconComponent(
  icon: Icons.favorite,
  iconSize: 48,
  paint: Paint()..colorFilter = const ColorFilter.mode(
    Color(0xFFFF0000),
    BlendMode.srcATop,
  ),
);
```


### Параметры конструктора

- `icon`: `IconData` для отображения (например, `Icons.star`, `Icons.favorite`).
- `iconSize`: Разрешение, с которым иконка растеризуется (по умолчанию `64`). Не зависит от отображаемого размера компонента `size`.
- `size`: Отображаемый размер компонента. Если не указан, по умолчанию `Vector2.all(iconSize)`.
- `paint`: Опциональный `Paint` для эффектов рендеринга.
- Все стандартные параметры `PositionComponent` (`position`, `scale`, `angle`, `anchor` и т.д.).


### Изменение иконки во время выполнения

Свойства `icon` и `iconSize` можно изменять после создания. Компонент автоматически перерастеризует иконку на следующем кадре:

```dart
final iconComponent = IconComponent(
  icon: Icons.play_arrow,
  iconSize: 64,
);

// Позже сменить иконку
iconComponent.icon = Icons.pause;

// Или изменить разрешение растеризации
iconComponent.iconSize = 128;
```# Sprite Components

Sprites are 2D images (or regions of images) that represent the visual appearance of game objects.
They are the most common way to display characters, items, backgrounds, and other visuals in 2D
games. Flame provides several sprite-based components that make it easy to load images, play
animations, and switch between visual states, all while benefiting from the transform properties
inherited from `PositionComponent`.


## SpriteComponent

The most commonly used implementation of `PositionComponent` is `SpriteComponent`, and it can be
created with a `Sprite`:

```dart
import 'package:flame/components/component.dart';

class MyGame extends FlameGame {
  late final SpriteComponent player;

  @override
  Future<void> onLoad() async {
    final sprite = await Sprite.load('player.png');
    final size = Vector2.all(128.0);
    final player = SpriteComponent(size: size, sprite: sprite);

    // Vector2(0.0, 0.0) by default, can also be set in the constructor
    player.position = Vector2(10, 20);

    // 0 by default, can also be set in the constructor
    player.angle = 0;

    // Adds the component
    add(player);
  }
}
```


## SpriteAnimationComponent

This class is used to represent a Component that has sprites that run in a single cyclic animation.

This will create a simple three frame animation using 3 different images:

```dart
@override
Future<void> onLoad() async {
  final sprites = [0, 1, 2]
      .map((i) => Sprite.load('player_$i.png'));
  final animation = SpriteAnimation.spriteList(
    await Future.wait(sprites),
    stepTime: 0.01,
  );
  this.player = SpriteAnimationComponent(
    animation: animation,
    size: Vector2.all(64.0),
  );
}
```

If you have a sprite sheet, you can use the `sequenced` constructor from the `SpriteAnimationData`
class (check more details on [Images > Animation](../rendering/images.md#animation)):

```dart
@override
Future<void> onLoad() async {
  final size = Vector2.all(64.0);
  final data = SpriteAnimationData.sequenced(
    textureSize: size,
    amount: 2,
    stepTime: 0.1,
  );
  this.player = SpriteAnimationComponent.fromFrameData(
    await images.load('player.png'),
    data,
  );
}
```

All animation components internally maintain a `SpriteAnimationTicker` which ticks the
`SpriteAnimation`. This allows multiple components to share the same animation object.

Example:

```dart
final sprites = [/*Your sprite list here*/];
final animation = SpriteAnimation.spriteList(sprites, stepTime: 0.01);

final animationTicker = SpriteAnimationTicker(animation);

// or alternatively, you can ask the animation object to create one for you.

final animationTicker = animation.createTicker(); // creates a new ticker

animationTicker.update(dt);
```

To listen when the animation is done (when it reaches the last frame and is not looping) you can
use `animationTicker.completed`.

Example:

```dart
await animationTicker.completed;

doSomething();

// or alternatively

animationTicker.completed.whenComplete(doSomething);
```

Additionally, `SpriteAnimationTicker` also has the following optional event callbacks: `onStart`,
`onFrame`, and `onComplete`. To listen to these events, you can do the following:

```dart
final animationTicker = SpriteAnimationTicker(animation)
  ..onStart = () {
    // Do something on start.
  };

final animationTicker = SpriteAnimationTicker(animation)
  ..onComplete = () {
    // Do something on completion.
  };

final animationTicker = SpriteAnimationTicker(animation)
  ..onFrame = (index) {
    if (index == 1) {
      // Do something for the second frame.
    }
  };
```

To reset the animation to the first frame when the component is removed, you can set
`resetOnRemove` to `true`:

```dart
SpriteAnimationComponent(
  animation: animation,
  size: Vector2.all(64.0),
  resetOnRemove: true,
);
```


## SpriteAnimationGroupComponent

`SpriteAnimationGroupComponent` is a simple wrapper around `SpriteAnimationComponent` which enables
your component to hold several animations and change the current playing animation at runtime. Since
this component is just a wrapper, the event listeners can be implemented as described in
[SpriteAnimationComponent](#spriteanimationcomponent).

Its use is very similar to the `SpriteAnimationComponent` but instead of being initialized with a
single animation, this component receives a Map of a generic type `T` as key and a
`SpriteAnimation` as value, and the current animation.

Example:

```dart
enum RobotState {
  idle,
  running,
}

final running = await loadSpriteAnimation(/* omitted */);
final idle = await loadSpriteAnimation(/* omitted */);

final robot = SpriteAnimationGroupComponent<RobotState>(
  animations: {
    RobotState.running: running,
    RobotState.idle: idle,
  },
  current: RobotState.idle,
);

// Changes current animation to "running"
robot.current = RobotState.running;
```

As this component works with multiple `SpriteAnimation`s, naturally it needs an equal number of
animation tickers to make all those animations tick. Use `animationsTickers` getter to access a map
containing tickers for each animation state. This can be useful if you want to register callbacks
for `onStart`, `onComplete` and `onFrame`.

Example:

```dart
enum RobotState { idle, running, jump }

final running = await loadSpriteAnimation(/* omitted */);
final idle = await loadSpriteAnimation(/* omitted */);

final robot = SpriteAnimationGroupComponent<RobotState>(
  animations: {
    RobotState.running: running,
    RobotState.idle: idle,
  },
  current: RobotState.idle,
);

robot.animationTickers?[RobotState.running]?.onStart = () {
  // Do something on start of running animation.
};

robot.animationTickers?[RobotState.jump]?.onStart = () {
  // Do something on start of jump animation.
};

robot.animationTickers?[RobotState.jump]?.onComplete = () {
  // Do something on complete of jump animation.
};

robot.animationTickers?[RobotState.idle]?.onFrame = (currentIndex) {
  // Do something based on current frame index of idle animation.
};
```


## SpriteGroupComponent

`SpriteGroupComponent` is pretty similar to its animation counterpart, but especially for sprites.

Example:

```dart
class PlayerComponent extends SpriteGroupComponent<ButtonState>
    with HasGameReference<SpriteGroupExample>, TapCallbacks {
  @override
  Future<void> onLoad() async {
    final pressedSprite = await game.loadSprite(/* omitted */);
    final unpressedSprite = await game.loadSprite(/* omitted */);

    sprites = {
      ButtonState.pressed: pressedSprite,
      ButtonState.unpressed: unpressedSprite,
    };

    current = ButtonState.unpressed;
  }

  // tap methods handler omitted...
}
```


## IconComponent

`IconComponent` renders a Flutter `IconData` (such as `Icons.star`) as a Flame component. The icon
is rasterized to an image once during `onLoad()` and then drawn each frame using
`canvas.drawImageRect()` with the component's `Paint`. Because the icon is rendered as a cached
image rather than as text, all paint-based effects work out of the box, including `tint()`,
`setOpacity()`, `ColorEffect`, `OpacityEffect`, `GlowEffect`, and custom `ColorFilter`s.


### Basic usage

```dart
import 'package:flame/components.dart';
import 'package:flutter/material.dart';

class MyGame extends FlameGame {
  @override
  Future<void> onLoad() async {
    final star = IconComponent(
      icon: Icons.star,
      iconSize: 64,
      position: Vector2(100, 100),
    );
    add(star);
  }
}
```


### Tinting and effects

The icon is rasterized in white, which allows you to tint it to any color using
`HasPaint` methods:

```dart
// Tint the icon gold
final star = IconComponent(
  icon: Icons.star,
  iconSize: 64,
  position: Vector2(100, 100),
)..tint(const Color(0xFFFFD700));

// Set opacity
star.setOpacity(0.5);

// Or use a custom paint
final icon = IconComponent(
  icon: Icons.favorite,
  iconSize: 48,
  paint: Paint()..colorFilter = const ColorFilter.mode(
    Color(0xFFFF0000),
    BlendMode.srcATop,
  ),
);
```


### Constructor parameters

- `icon`: The `IconData` to render (e.g., `Icons.star`, `Icons.favorite`).
- `iconSize`: The resolution at which the icon is rasterized (default `64`). This is independent
  of the component's display `size`.
- `size`: The display size of the component. Defaults to `Vector2.all(iconSize)` if not provided.
- `paint`: Optional `Paint` for rendering effects.
- All standard `PositionComponent` parameters (`position`, `scale`, `angle`, `anchor`, etc.).


### Changing the icon at runtime

Both the `icon` and `iconSize` properties can be changed after creation. The component will
automatically re-rasterize the icon on the next frame:

```dart
final iconComponent = IconComponent(
  icon: Icons.play_arrow,
  iconSize: 64,
);

// Later, swap the icon
iconComponent.icon = Icons.pause;

// Or change the rasterization resolution
iconComponent.iconSize = 128;
```
