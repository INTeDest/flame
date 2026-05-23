# FlameGame

Каждой игре необходим центральный объект, управляющий игровым циклом — непрерывным процессом обновления состояния и отрисовки кадров, который лежит в основе всех игр реального времени. Во Flame эту роль выполняет `FlameGame`, который также является корнем дерева компонентов. Если вы знакомы с Flutter, представляйте `FlameGame` как аналог `MaterialApp`: точка входа верхнего уровня, на которую опирается всё остальное.

Основой почти всех игр на Flame является класс `FlameGame`. Это корень вашего дерева компонентов. Мы называем эту компонентную систему Flame Component System (FCS). В документации FCS используется для обозначения этой системы.

Класс `FlameGame` реализует `Game` на основе `Component`. Он содержит дерево компонентов и вызывает методы `update` и `render` всех компонентов, добавленных в игру.

Компоненты можно добавлять в `FlameGame` напрямую в конструкторе с помощью именованного аргумента `children` или откуда угодно с помощью методов `add`/`addAll`. Однако чаще всего дочерние компоненты добавляются в `World`. Мир по умолчанию находится в свойстве `FlameGame.world`, и компоненты добавляются в него точно так же, как и в любой другой компонент.

Простая реализация `FlameGame`, добавляющая два компонента — один в `onLoad`, а другой напрямую в конструкторе — может выглядеть так:

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/widgets.dart';

/// Компонент, отображающий спрайт ящика размером 16×16.
class MyCrate extends SpriteComponent {
  MyCrate() : super(size: Vector2.all(16));

  @override
  Future<void> onLoad() async {
    sprite = await Sprite.load('crate.png');
  }
}

class MyWorld extends World {
  @override
  Future<void> onLoad() async {
    await add(MyCrate());
  }
}

void main() {
  final myGame = FlameGame(world: MyWorld());
  runApp(
    GameWidget(game: myGame),
  );
}
```


## Пользовательский тип World

`FlameGame` имеет обобщённый параметр типа `W`, который по умолчанию равен `World`. Указав собственный тип мира, вы сможете обращаться к геттеру `world` вашей игры напрямую с вашим конкретным типом, без приведения.

Это полезно, когда у вас есть собственный подкласс `World` и вы хотите обращаться к его свойствам или методам из класса игры:

```dart
class MyWorld extends World {
  int score = 0;
}

class MyGame extends FlameGame<MyWorld> {
  MyGame() : super(world: MyWorld());

  void incrementScore() {
    // Приведение не требуется, `world` уже типизирован как `MyWorld`.
    world.score++;
  }
}
```

При использовании этого обобщённого параметра вы **обязаны** передать соответствующий экземпляр мира в конструктор `super`. Если обобщённый тип указан, но мир не передан, будет выброшена ошибка утверждения во время выполнения.

```{note}
Если вы создаёте экземпляр игры в методе build, игра будет пересоздаваться
каждый раз при перестроении дерева Flutter, что обычно происходит чаще, чем
хотелось бы. Чтобы избежать этого, можно либо создать экземпляр игры заранее и
ссылаться на него в структуре виджетов, либо использовать конструктор
`GameWidget.controlled`.
```

Для удаления компонентов из списка `FlameGame` используются методы `remove` или `removeAll`. Первый применяется для удаления одного компонента, второй — для списка компонентов. Эти методы есть у всех `Component`, включая `World`.

`FlameGame` имеет встроенный `World` с именем `world` и экземпляр `CameraComponent` с именем `camera`. Подробнее о них можно прочитать в разделе [Camera](camera.md).


## Игровой цикл

Модуль `GameLoop` — это простая абстракция концепции игрового цикла. По сути, большинство игр строятся на двух методах:

- Метод `render` получает холст для отрисовки текущего состояния игры.
- Метод `update` получает время в секундах, прошедшее с предыдущего обновления (`delta time`), и позволяет перейти к следующему состоянию.

`GameLoop` используется всеми реализациями `Game` во Flame.


## Изменение размера

Каждый раз, когда игру необходимо изменить в размере — например, при смене ориентации, — `FlameGame` вызывает у всех `Component` методы `onGameResize`, а также передаёт эту информацию камере и вьюпорту.

Свойство `FlameGame.camera` определяет, какая точка координатного пространства должна находиться в якоре видоискателя. По умолчанию [0,0] находится в центре (`Anchor.center`) вьюпорта.


## Жизненный цикл

Обратные вызовы жизненного цикла `FlameGame` (`onLoad`, `render` и т.д.) вызываются в следующей последовательности:

```{include} diagrams/flame_game_life_cycle.md
```

Когда `FlameGame` впервые добавляется в `GameWidget`, методы жизненного цикла `onGameResize`, `onLoad` и `onMount` вызываются именно в этом порядке. Затем на каждом игровом тике последовательно вызываются `update` и `render`. Если `FlameGame` удаляется из `GameWidget`, вызывается `onRemove`. Если `FlameGame` добавляется в новый `GameWidget`, последовательность повторяется, начиная с `onGameResize`.

```{note}
Порядок вызова `onGameResize` и `onLoad` для `FlameGame` обратный по
сравнению с другими `Component`. Это сделано для того, чтобы размеры
элементов игры можно было рассчитать до загрузки или создания ресурсов.
```

Обратный вызов `onRemove` можно использовать для очистки дочерних компонентов и кешированных данных:

```dart
  @override
  void onRemove() {
    // Опционально, зависит от потребностей игры.
    removeAll(children);
    processLifecycleEvents();
    Flame.images.clearCache();
    Flame.assets.clearCache();
    // Любой другой код, который нужно выполнить при удалении игры.
  }
```

```{note}
Очистка дочерних компонентов и ресурсов в `FlameGame` не выполняется
автоматически, её необходимо явно добавить в вызов `onRemove`.
```


### onHotReload

Когда во Flutter срабатывает горячая перезагрузка (только в режиме отладки), `GameWidget` вызывает `onHotReload` на `FlameGame`, который автоматически распространяет уведомление на каждый загруженный или загружающийся компонент в дереве. Переопределите этот метод в любом компоненте, чтобы перезагружать ресурсы, обновлять кешированные значения или реагировать на изменения исходного кода во время разработки:

```dart
class MyGame extends FlameGame {
  @override
  void onHotReload() {
    // Обновление состояния уровня игры, затронутого изменениями кода.
    super.onHotReload();
  }
}
```

```{note}
`onHotReload` вызывается только в режиме отладки. Компоненты, которые ещё
находятся в очереди жизненного цикла (загружаются, но ещё не смонтированы),
также получают уведомление. Всегда вызывайте `super.onHotReload()`, чтобы
событие продолжило распространяться на дочерние компоненты.
```


### dispose()

Для удобства `FlameGame` предоставляет метод `dispose()`, который выполняет всю стандартную очистку за один вызов:

```dart
  game.dispose();
```

Этот метод удаляет все дочерние компоненты из игры (вызывая `onRemove` у каждого компонента в дереве), обрабатывает все ожидающие события жизненного цикла и очищает кеши `images` и `assets`.

Разница между `dispose()` и `onRemove` заключается в том, что `dispose()` — это метод, который вы вызываете явно для выполнения очистки, а `onRemove` — это обратный вызов жизненного цикла, который вызывается автоматически при удалении игры из `GameWidget`. Вы можете использовать `dispose()` внутри `onRemove` или вызывать его независимо, когда вам нужно сбросить состояние игры.


## Режим отладки

Класс `FlameGame` предоставляет переменную `debugMode`, которая по умолчанию равна `false`. Однако её можно установить в `true`, чтобы включить отладочные возможности для компонентов игры. **Обратите внимание:** значение этой переменной передаётся компонентам при их добавлении в игру, поэтому если вы измените `debugMode` во время выполнения, это по умолчанию не повлияет на уже добавленные компоненты.

Подробнее о `debugMode` во Flame читайте в [документации по отладке](other/debug.md).


## Изменение цвета фона

Чтобы изменить цвет фона вашего `FlameGame`, переопределите метод `backgroundColor()`.

В следующем примере цвет фона устанавливается полностью прозрачным, чтобы были видны виджеты, находящиеся позади `GameWidget`. По умолчанию используется непрозрачный чёрный.

```dart
class MyGame extends FlameGame {
  @override
  Color backgroundColor() => const Color(0x00000000);
}
```

Обратите внимание, что цвет фона не может динамически изменяться во время работы игры, но если нужна динамика, вы можете просто нарисовать фон, покрывающий весь холст.


## Примесь SingleGameInstance

Опциональная примесь `SingleGameInstance` может быть применена к вашей игре, если вы создаёте приложение с единственной игрой. Это распространённый сценарий при создании игр: есть один полноэкранный `GameWidget`, содержащий один экземпляр `Game`.

Добавление этой примеси даёт преимущества в производительности в определённых ситуациях. В частности, метод `onLoad` компонента гарантированно запускается, когда этот компонент добавляется к родителю, даже если родитель сам ещё не смонтирован. Следовательно, ожидание (`await`) вызова `parent.add(component)` гарантированно завершит загрузку компонента.

Использовать примесь просто:

```dart
class MyGame extends FlameGame with SingleGameInstance {
  // ...
}
```


## Низкоуровневый API Game

```{include} diagrams/low_level_game_api.md
```

Абстрактный класс `Game` — это низкоуровневый API, который можно использовать, если вы хотите самостоятельно реализовать структуру игрового движка. Например, `Game` не реализует никаких функций `update` или `render`.

Этот класс также содержит методы жизненного цикла `onLoad`, `onMount` и `onRemove`, которые вызываются из `GameWidget` (или другого родителя) при загрузке и монтировании либо удалении игры. `onLoad` вызывается только при первом добавлении класса к родителю, тогда как `onMount` (вызывается после `onLoad`) вызывается каждый раз при добавлении к новому родителю. `onRemove` вызывается при удалении класса от родителя.

```{note}
Класс `Game` даёт больше свободы в реализации, но при его использовании вы
лишаетесь всех встроенных возможностей Flame.
```

Пример того, как может выглядеть реализация `Game`:

```dart
class MyGameSubClass extends Game {
  @override
  void render(Canvas canvas) {
    // ...
  }

  @override
  void update(double dt) {
    // ...
  }
}

void main() {
  final myGame = MyGameSubClass();
  runApp(
    GameWidget(
      game: myGame,
    )
  );
}
```


## Пауза / Возобновление / Пошаговое выполнение игры

Игру Flame (`Game`) можно поставить на паузу и возобновить двумя способами:

- С помощью методов `pauseEngine` и `resumeEngine`.
- Изменяя атрибут `paused`.

При постановке `Game` на паузу `GameLoop` фактически останавливается, то есть никаких обновлений или новых отрисовок не происходит до возобновления.

Пока игра на паузе, её можно продвигать покадрово с помощью метода `stepEngine`. В финальной игре это вряд ли пригодится, но может быть очень полезно для пошаговой проверки состояния игры в процессе разработки.


### Сворачивание в фоновый режим

Игра автоматически ставится на паузу, когда приложение уходит в фон, и возобновляется при возврате на передний план. Это поведение можно отключить, установив `pauseWhenBackgrounded` в `false`.

```dart
class MyGame extends FlameGame {
  MyGame() {
    pauseWhenBackgrounded = false;
  }
}
```

В настоящее время этот флаг работает только на Android и iOS.


## Примесь HasPerformanceTracker

При оптимизации игры бывает полезно отслеживать время, затраченное на обновление и отрисовку каждого кадра. Эти данные помогают обнаружить участки кода, работающие медленно, а также визуальные области игры, которые требуют наибольшего времени на отрисовку.

Чтобы получить время обновления и отрисовки, просто добавьте примесь `HasPerformanceTracker` к классу игры.

```dart
class MyGame extends FlameGame with HasPerformanceTracker {
  // доступ к геттерам `updateTime` и `renderTime`.
}
```# FlameGame

Every game needs a central object that owns the game loop, the continuous cycle of updating state
and rendering frames that drives all real-time games. In Flame, `FlameGame` fills that role while
also serving as the root of the component tree. If you are familiar with Flutter, think of
`FlameGame` as the equivalent of `MaterialApp`: the top-level entry point that everything else
hangs off of.

The base of almost all Flame games is the `FlameGame` class. It is the root of your component
tree. We refer to this component-based system as the Flame Component System (FCS). Throughout the
documentation, FCS is used to reference this system.

The `FlameGame` class implements a `Component` based `Game`. It has a tree of components
and calls the `update` and `render` methods of all components that have been added to the game.

Components can be added to the `FlameGame` directly in the constructor with the named `children`
argument, or from anywhere else with the `add`/`addAll` methods. Most of the time however, you want
to add your children to a `World`, the default world exists under `FlameGame.world` and you add
components to it just like you would to any other component.

A simple `FlameGame` implementation that adds two components, one in `onLoad` and one directly in
the constructor can look like this:

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/widgets.dart';

/// A component that renders the crate sprite, with a 16 x 16 size.
class MyCrate extends SpriteComponent {
  MyCrate() : super(size: Vector2.all(16));

  @override
  Future<void> onLoad() async {
    sprite = await Sprite.load('crate.png');
  }
}

class MyWorld extends World {
  @override
  Future<void> onLoad() async {
    await add(MyCrate());
  }
}

void main() {
  final myGame = FlameGame(world: MyWorld());
  runApp(
    GameWidget(game: myGame),
  );
}
```


## Custom World type

`FlameGame` has a generic type parameter `W` that defaults to `World`. By specifying a custom world
type, the `world` getter on your game will return your specific world type directly, without needing
to cast it.

This is useful when you have a custom `World` subclass and want to access its properties or methods
from within your game class:

```dart
class MyWorld extends World {
  int score = 0;
}

class MyGame extends FlameGame<MyWorld> {
  MyGame() : super(world: MyWorld());

  void incrementScore() {
    // No cast needed, `world` is already typed as `MyWorld`.
    world.score++;
  }
}
```

When using this generic parameter, you **must** pass a matching world instance to the `super`
constructor. If the generic type is specified but no world is provided, a runtime assertion error
will be thrown.

```{note}
If you instantiate your game in a build method your game will be rebuilt every
time the Flutter tree gets rebuilt, which usually is more often than you'd like.
To avoid this, you can either create an instance of your game first and
reference it within your widget structure or use the `GameWidget.controlled`
constructor.
```

To remove components from the list on a `FlameGame` the `remove` or `removeAll` methods can be used.
The first can be used if you just want to remove one component, and the second can be used when you
want to remove a list of components. These methods exist on all `Component`s, including the `World`.

The `FlameGame` has a built-in `World` called `world` and a `CameraComponent` instance called
`camera`, you can read more about those in the [Camera section](camera.md).


## Game Loop

The `GameLoop` module is a simple abstraction of the game loop concept. Basically, most games are
built upon two methods:

- The render method takes the canvas for drawing the current state of the game.
- The update method receives the delta time in seconds since the last update and allows you to
  move to the next state.

The `GameLoop` is used by all of Flame's `Game` implementations.


## Resizing

Every time the game needs to be resized, for example when the orientation is changed, `FlameGame`
will call all of the `Component`'s `onGameResize` methods and it will also pass this information to
the camera and viewport.

The `FlameGame.camera` controls which point in the coordinate space that should be at the anchor of
your viewfinder, [0,0] is in the center (`Anchor.center`) of the viewport by default.


## Lifecycle

The `FlameGame` lifecycle callbacks, `onLoad`, `render`, etc. are called in the following sequence:

```{include} diagrams/flame_game_life_cycle.md
```

When a `FlameGame` is first added to a `GameWidget` the lifecycle methods `onGameResize`, `onLoad`
and `onMount` will be called in that order. Then `update` and `render` are called in sequence for
every game tick. If the `FlameGame` is removed from the `GameWidget` then `onRemove` is called.
If the `FlameGame` is added to a new `GameWidget` the sequence repeats from `onGameResize`.

```{note}
The order of `onGameResize` and `onLoad` are reversed from that of other
`Component`s. This is to allow game element sizes to be calculated before
resources are loaded or generated.
```

The `onRemove` callback can be used to clean up children and cached data:

```dart
  @override
  void onRemove() {
    // Optional based on your game needs.
    removeAll(children);
    processLifecycleEvents();
    Flame.images.clearCache();
    Flame.assets.clearCache();
    // Any other code that you want to run when the game is removed.
  }
```

```{note}
Clean-up of children and resources in a `FlameGame` is not done automatically
and must be explicitly added to the `onRemove` call.
```


### onHotReload

When Flutter's hot reload is triggered (debug mode only), the `GameWidget` calls `onHotReload` on
the `FlameGame`, which automatically propagates the notification to every component in the tree that
is loading or loaded. Override this method on any component to reload assets, refresh cached values,
or react to source code changes during development:

```dart
class MyGame extends FlameGame {
  @override
  void onHotReload() {
    // Refresh game-level state affected by code changes.
    super.onHotReload();
  }
}
```

```{note}
`onHotReload` is only called in debug mode. Components that are still in the
lifecycle queue (loading but not yet mounted) also receive the notification.
Always call `super.onHotReload()` so the event continues to propagate to
children.
```


### dispose()

As a convenience, `FlameGame` provides a `dispose()` method that handles all of the common cleanup
in a single call:

```dart
  game.dispose();
```

This removes all children from the game (triggering `onRemove` on every component in the tree),
processes all pending lifecycle events, and clears the `images` and `assets` caches.

The difference between `dispose()` and `onRemove` is that `dispose()` is a method you call
explicitly to perform cleanup, while `onRemove` is a lifecycle callback that is invoked
automatically when the game is removed from a `GameWidget`. You can use `dispose()` from within
`onRemove`, or call it independently whenever you need to reset the game state.


## Debug mode

Flame's `FlameGame` class provides a variable called `debugMode`, which by default is `false`. It
can, however, be set to `true` to enable debug features for the components of the game. **Be aware**
that the value of this variable is passed through to its components when they are added to the
game, so if you change the `debugMode` at runtime, it will not affect already added components by
default.

To read more about the `debugMode` on Flame, please refer to the [Debug Docs](other/debug.md)


## Change background color

To change the background color of your `FlameGame` you have to override `backgroundColor()`.

In the following example, the background color is set to be fully transparent, so that you can see
the widgets that are behind the `GameWidget`. The default is opaque black.

```dart
class MyGame extends FlameGame {
  @override
  Color backgroundColor() => const Color(0x00000000);
}
```

Note that the background color can't change dynamically while the game is running, but you could
just draw a background that covers the whole canvas if you would want it to change dynamically.


## SingleGameInstance mixin

An optional mixin `SingleGameInstance` can be applied to your game if you are making a single-game
application. This is a common scenario when building games: there is a single full-screen
`GameWidget` that hosts a single `Game` instance.

Adding this mixin provides performance advantages in certain scenarios. In particular, a component's
`onLoad` method is guaranteed to start when that component is added to its parent, even if the
parent is not yet mounted itself. Consequently, `await`-ing on `parent.add(component)` is guaranteed
to always finish loading the component.

Using this mixin is simple:

```dart
class MyGame extends FlameGame with SingleGameInstance {
  // ...
}
```


## Low-level Game API

```{include} diagrams/low_level_game_api.md
```

The abstract `Game` class is a low-level API that can be used when you want to implement the
functionality of how the game engine should be structured. `Game` does not implement any `update` or
`render` function for example.

The class also has the lifecycle methods `onLoad`, `onMount` and `onRemove` in it, which are
called from the `GameWidget` (or another parent) when the game is loaded + mounted, or removed.
`onLoad` is only called the first time the class is added to a parent, but `onMount` (which is
called after `onLoad`) is called every time it is added to a new parent. `onRemove` is called when
the class is removed from a parent.

```{note}
The `Game` class allows for more freedom of how to implement things, but you
are also missing out on all of the built-in features in Flame if you use it.
```

An example of what a `Game` implementation could look like:

```dart
class MyGameSubClass extends Game {
  @override
  void render(Canvas canvas) {
    // ...
  }

  @override
  void update(double dt) {
    // ...
  }
}

void main() {
  final myGame = MyGameSubClass();
  runApp(
    GameWidget(
      game: myGame,
    )
  );
}
```


## Pause/Resuming/Stepping game execution

A Flame `Game` can be paused and resumed in two ways:

- With the use of the `pauseEngine` and `resumeEngine` methods.
- By changing the `paused` attribute.

When pausing a `Game`, the `GameLoop` is effectively paused, meaning that no updates or new renders
will happen until it is resumed.

While the game is paused, it is possible to advance it frame by frame using the `stepEngine`
method. It might not be very useful in the final game, but it can be very helpful for inspecting
game state step by step during the development cycle.


### Backgrounding

The game will be automatically paused when the app is sent to the background,
and resumed when it comes back to the foreground. This behavior can be disabled by setting
`pauseWhenBackgrounded` to `false`.

```dart
class MyGame extends FlameGame {
  MyGame() {
    pauseWhenBackgrounded = false;
  }
}
```

This flag currently only works on Android and iOS.


## HasPerformanceTracker mixin

While optimizing a game, it can be useful to track the time it took for the game to update and
render each frame. This data can help in detecting areas of the code that are running hot. It can
also help
in detecting visual areas of the game that are taking the most time to render.

To get the update and render times, just add the `HasPerformanceTracker` mixin to the game class.

```dart
class MyGame extends FlameGame with HasPerformanceTracker {
  // access `updateTime` and `renderTime` getters.
}
```
