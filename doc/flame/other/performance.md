# Производительность

Как и любой другой игровой движок, Flame старается быть максимально эффективным, не усложняя чрезмерно API. Однако в силу своего универсального характера Flame не может делать предположений о типе создаваемой игры. Это означает, что у разработчиков всегда остаётся пространство для оптимизации производительности с учётом особенностей их игры.

С другой стороны, в зависимости от используемого оборудования всегда существует некоторый жёсткий предел того, чего можно достичь с помощью Flame. Но помимо аппаратных ограничений есть ряд распространённых ошибок, с которыми могут столкнуться пользователи Flame, и которых легко избежать, следуя простым правилам. В этом разделе рассматриваются некоторые приёмы оптимизации и способы избежать типичных проблем с производительностью.

```{note}
Дисклеймер: Каждый проект на Flame сильно отличается от других. Поэтому описанные здесь решения не всегда гарантируют значительное улучшение производительности.
```


## Создание объектов в каждом кадре

Создание объектов класса — обычное дело в любом проекте или игре. Но создание объекта — довольно затратная операция. В зависимости от частоты и количества создаваемых объектов приложение может испытывать замедление.

В играх к этому нужно относиться особенно внимательно, потому что игры обычно имеют игровой цикл, который обновляется настолько быстро, насколько это возможно, где каждая итерация называется кадром. В зависимости от оборудования игра может обновляться 30, 60, 120 и даже больше раз в секунду. Это означает, что если в каждом кадре создаётся новый объект, то игра будет создавать количество объектов, равное количеству кадров в секунду.

Пользователи Flame обычно сталкиваются с этой проблемой, когда переопределяют методы `update` и `render` компонента. Например, в следующем безобидно выглядящем коде каждый кадр создаётся новый объект `Vector2` и новый `Paint`. Но данные внутри этих объектов по сути одинаковы во всех кадрах. Теперь представьте, что в игре 100 экземпляров `MyComponent`, работающей с частотой 60 FPS. Это будет означать создание 6000 (100 * 60) новых экземпляров `Vector2` и `Paint` каждую секунду.

```{note}
Это всё равно что покупать новый компьютер каждый раз, когда нужно отправить электронное письмо, или новую ручку, когда нужно что-то написать. Конечно, дело будет сделано, но экономически это не очень разумно.
```

```dart
class MyComponent extends PositionComponent {
  @override
  void update(double dt) {
    position += Vector2(10, 20) * dt;
  }

  @override
  void render(Canvas canvas) {
    canvas.drawRect(size.toRect(), Paint());
  }
}
```

Более правильный способ показан ниже. Этот код сохраняет необходимые объекты `Vector2` и `Paint` как члены класса и повторно использует их при всех вызовах `update` и `render`.

```dart
class MyComponent extends PositionComponent {
  final _direction = Vector2(10, 20);
  final _paint = Paint();

  @override
  void update(double dt) {
    position.setValues(
      position.x + _direction.x * dt, 
      position.y + _direction.y * dt,
    );
  }

  @override
  void render(Canvas canvas) {
    canvas.drawRect(size.toRect(), _paint);
  }
}
```

```{note}
Вкратце: избегайте создания ненужных объектов в каждом кадре. Даже кажущийся незначительным объект может повлиять на производительность при массовом создании.
```


## Ненужные проверки столкновений

Flame имеет встроенную систему обнаружения столкновений, которая определяет, когда любые два `Hitbox` пересекаются. В идеальном случае эта система работает в каждом кадре и проверяет столкновения. Она также достаточно умна, чтобы отфильтровать только потенциальные столкновения перед выполнением фактических проверок пересечений.

Несмотря на это, можно с уверенностью предположить, что стоимость обнаружения столкновений будет расти с увеличением количества хитбоксов. Но во многих играх разработчики не всегда заинтересованы в обнаружении столкновений между каждой возможной парой. Например, рассмотрим простую игру, где игроки могут стрелять компонентом `Bullet`, имеющим хитбокс. В такой игре разработчики, скорее всего, не заинтересованы в обнаружении столкновений между любыми двумя пулями, но Flame всё равно будет выполнять эти проверки.

Чтобы избежать этого, можно установить `collisionType` для компонента пули в `CollisionType.passive`. Это заставит Flame полностью пропускать любые проверки столкновений между всеми пассивными хитбоксами.

```{note}
Это не означает, что компонент пули во всех играх должен всегда иметь пассивный хитбокс. Разработчики сами решают, какие хитбоксы можно сделать пассивными, исходя из правил игры. Например, в игре Rogue Shooter из примеров Flame используется пассивный хитбокс для врагов, а не для пуль.
```


## Пул объектов

Как упоминалось в разделе «Создание объектов в каждом кадре», частое создание и уничтожение объектов может повлиять на производительность. Для компонентов, которые многократно создаются и удаляются (например, пули, частицы или враги), пул объектов является эффективным методом оптимизации.

Пул объектов переиспользует объекты вместо постоянного создания и уничтожения. Flame предоставляет класс `ComponentPool`, чтобы сделать использование пула объектов простым и эффективным.


### ComponentPool

Класс `ComponentPool` управляет пулом переиспользуемых компонентов. Он автоматически обрабатывает жизненный цикл компонента: когда компонент из пула удаляется от своего родителя, он возвращается в пул для повторного использования.

**Создание пула:**

```dart
class MyGame extends FlameGame {
  late final ComponentPool<Bullet> bulletPool;

  @override
  Future<void> onLoad() async {
    bulletPool = ComponentPool<Bullet>(
      factory: () => Bullet(),
      maxSize: 50,      // Максимальное количество пуль, сохраняемых в пуле
      initialSize: 10,  // Предварительно создать 10 пуль для немедленного использования
    );
  }
}
```

**Получение компонентов из пула:**

Когда вам нужен компонент, используйте `acquire()`, чтобы получить его из пула. Если пул пуст, новый компонент будет создан автоматически с помощью фабричной функции.

```dart
void spawnBullet(Vector2 position, Vector2 velocity) {
  final bullet = bulletPool.acquire();
  bullet.position.setFrom(position);
  bullet.velocity.setFrom(velocity);
  world.add(bullet);
}
```

**Возврат компонентов в пул:**

Компоненты возвращаются в пул **автоматически** при удалении из игрового дерева. Просто вызовите `removeFromParent()` на компоненте. Никакого ручного шага освобождения не требуется.

```dart
class Bullet extends SpriteComponent with CollisionCallbacks {
  Vector2 velocity = Vector2.zero();

  @override
  void update(double dt) {
    super.update(dt);
    position.add(velocity * dt);

    // Удалить пулю, если она уходит за экран. Автоматически возвращается в пул.
    if (position.x < -100 || position.x > game.size.x + 100) {
      removeFromParent();
    }
  }

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    // Возврат в пул при столкновении. Ручное освобождение не требуется.
    removeFromParent();
  }

  @override
  void onMount() {
    super.onMount();
    // Сбросить визуальное/внутреннее состояние здесь, чтобы компонент был чистым при переиспользовании.
    // Состояние, настроенное вызывающей стороной (позиция, скорость), НЕ следует сбрасывать здесь,
    // потому что оно устанавливается между вызовами acquire() и add().
  }
}
```


### Управление пулом

**Проверка доступных компонентов:**

Вы можете проверить, сколько компонентов в данный момент доступно в пуле:

```dart
print('Доступно пуль: ${bulletPool.availableCount}');
```

**Очистка пула:**

Если вам нужно освободить память или сбросить пул, вы можете очистить все доступные компоненты:

```dart
bulletPool.clear();
```

```{note}
Очистка влияет только на компоненты, находящиеся в данный момент в пуле. Компоненты, которые используются (получены, но ещё не освобождены), не затрагиваются.
```


### Лучшие практики

1. **Не требуется специальных примесей**: Любой подкласс `Component` можно поместить в пул. Просто передайте фабричную функцию в `ComponentPool`, и всё готово к работе.

2. **Используйте `onMount` для сброса внутреннего состояния**: Сбрасывайте визуальные или внутренние свойства (например, кадр анимации, фазу отскока) в `onMount()`. Не сбрасывайте там состояние, настроенное вызывающей стороной (например, позицию или скорость), поскольку оно устанавливается между `acquire()` и `add()`.

3. **Просто вызывайте `removeFromParent()`**: Компоненты возвращаются в пул автоматически при удалении. Не нужно вызывать никакого ручного метода освобождения.

4. **Устанавливайте подходящий размер пула**: Установите `maxSize`, исходя из потребностей вашей игры. Слишком маленький — и вы будете часто создавать новые объекты; слишком большой — и вы будете тратить память впустую.

5. **Используйте `initialSize` для разогрева**: Установите `initialSize` для предварительного создания часто используемых компонентов, уменьшая просадки кадров во время игры.

6. **Поведение пула — LIFO**: Внутренне пул использует стек (Last In, First Out), то есть последний возвращённый компонент будет получен следующим.# Performance

Just like any other game engine, Flame tries to be as efficient as possible without making the API
too complex. But given its general purpose nature, Flame cannot make any assumption about the type of
game being made. This means game developers will always have some room for performance optimizations
based on how their game functions.

On the other hand, depending on the underlying hardware, there will always be some hard limit on what
can be achieved with Flame. But apart from the hardware limits, there are some common pitfalls that
Flame users can run into, which can be easily avoided by following some simple steps. This section tries
to cover some optimization tricks and ways to avoid the common performance pitfalls.

```{note}
Disclaimer: Each Flame project is very different from the others. As a result, solution
described here cannot guarantee to always produce a significant improvement in performance.
```


## Object creation per frame

Creating objects of a class is very common in any kind of project/game. But object creation is a somewhat
involved operation. Depending on the frequency and amount of objects that are being created, the application
can experience some slow down.

In games, this is something to be very careful of because games generally have a game loop that
updates as fast as possible, where each update is called a frame. Depending on the hardware, a
game can be updating 30, 60, 120 or even higher frames per second. This means if a new object is
created in a frame, the game will end up creating as many number of objects as the frame count
per second.

Flame users generally tend to run into this problem when they override the `update` and `render`
method of a `Component`. For example, in the following innocent looking code, a new `Vector2` and
a new `Paint` object is spawned every frame. But the data inside the objects is essentially the
same across all frames. Now imagine if there are 100 instances of `MyComponent` in a game running
at 60 FPS. That would essentially mean 6000 (100 * 60) new instances of `Vector2` and `Paint`
each will be created every second.

```{note}
It is like buying a new computer every time you want to send an email or buying
a new pen every time you want to write something. Sure it gets the job done, but
it is not very economically smart.
```

```dart
class MyComponent extends PositionComponent {
  @override
  void update(double dt) {
    position += Vector2(10, 20) * dt;
  }

  @override
  void render(Canvas canvas) {
    canvas.drawRect(size.toRect(), Paint());
  }
}
```

A better way of doing things would be something like as shown below. This code stores the required `Vector2`
and `Paint` objects as class members and reuses them across all the update and render calls.

```dart
class MyComponent extends PositionComponent {
  final _direction = Vector2(10, 20);
  final _paint = Paint();

  @override
  void update(double dt) {
    position.setValues(
      position.x + _direction.x * dt, 
      position.y + _direction.y * dt,
    );
  }

  @override
  void render(Canvas canvas) {
    canvas.drawRect(size.toRect(), _paint);
  }
}
```

```{note}
To summarize, avoid creating unnecessary objects in every frame. Even a seemingly
small object can affect the performance if spawned in high volume.
```


## Unwanted collision checks

Flame has a built-in collision detection system which can detect when any two `Hitbox`es intersect
with each other. In an ideal case, this system runs on every frame and checks for collision. It is
also smart enough to filter out only the possible collisions before performing the actual
intersection checks.

Despite this, it is safe to assume that the cost of collision detection will increase as the
number of hitboxes increases. But in many games, the developers are not always interested in
detecting collision between every possible pair. For example, consider a simple game where players
can fire a `Bullet` component that has a hitbox. In such a game it is likely that the developers
are not interested in detecting collision between any two bullets, but Flame will still perform
those collision checks.

To avoid this, you can set the `collisionType` for bullet component to `CollisionType.passive`. Doing
so will cause Flame to completely skip any kind of collision check between all the passive hitboxes.

```{note}
This does not mean bullet component in all games must always have a passive hitbox.
It is up to the developers to decide which hitboxes can be made passive based on
the rules of the game. For example, the Rogue Shooter game in Flame's examples uses
passive hitbox for enemies instead of the bullets. 
```


## Object Pooling

As mentioned in the "Object creation per frame" section, creating and destroying objects
frequently can impact performance. For components that are spawned and removed repeatedly
(like bullets, particles, or enemies), object pooling is an effective optimization technique.

Object pooling reuses objects instead of constantly creating and destroying them. Flame
provides the `ComponentPool` class to make object pooling easy and efficient.


### ComponentPool

The `ComponentPool` class manages a pool of reusable components. It automatically handles
the component lifecycle: when a pooled component is removed from its parent, it is
returned to the pool for reuse.

**Creating a pool:**

```dart
class MyGame extends FlameGame {
  late final ComponentPool<Bullet> bulletPool;

  @override
  Future<void> onLoad() async {
    bulletPool = ComponentPool<Bullet>(
      factory: () => Bullet(),
      maxSize: 50,      // Maximum number of bullets to keep in the pool
      initialSize: 10,  // Pre-create 10 bullets for immediate use
    );
  }
}
```

**Acquiring components from the pool:**

When you need a component, use `acquire()` to get one from the pool. If the pool is empty,
a new component will be created automatically using the factory function.

```dart
void spawnBullet(Vector2 position, Vector2 velocity) {
  final bullet = bulletPool.acquire();
  bullet.position.setFrom(position);
  bullet.velocity.setFrom(velocity);
  world.add(bullet);
}
```

**Returning components to the pool:**

Components are returned to the pool **automatically** when they are removed from the game
tree. Simply call `removeFromParent()` on the component. There is no manual release step.

```dart
class Bullet extends SpriteComponent with CollisionCallbacks {
  Vector2 velocity = Vector2.zero();

  @override
  void update(double dt) {
    super.update(dt);
    position.add(velocity * dt);

    // Remove bullet if it goes off screen. Automatically returned to pool.
    if (position.x < -100 || position.x > game.size.x + 100) {
      removeFromParent();
    }
  }

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    // Return to pool on collision. No manual release needed.
    removeFromParent();
  }

  @override
  void onMount() {
    super.onMount();
    // Reset visual/internal state here so the component is clean when reused.
    // Caller-configured state (position, velocity) should NOT be reset here
    // because it is set between acquire() and add().
  }
}
```


### Pool Management

**Checking available components:**

You can check how many components are currently available in the pool:

```dart
print('Available bullets: ${bulletPool.availableCount}');
```

**Clearing the pool:**

If you need to free up memory or reset the pool, you can clear all available components:

```dart
bulletPool.clear();
```

```{note}
Clearing only affects components currently in the pool. Components that are in use
(acquired but not yet released) are not affected.
```


### Best Practices

1. **No special mixin needed**: Any `Component` subclass can be pooled. Just pass a factory
   function to `ComponentPool` and you're ready to go.

2. **Use `onMount` to reset internal state**: Reset visual or internal properties (e.g.
   animation frame, bounce phase) in `onMount()`. Do not reset caller-configured state
   (like position or velocity) there, since those are set between `acquire()` and `add()`.

3. **Just call `removeFromParent()`**: Components are returned to the pool automatically
   when removed. There is no manual release method to call.

4. **Set appropriate pool sizes**: Set `maxSize` based on your game's needs. Too small and
   you'll create new objects frequently; too large and you'll waste memory.

5. **Use `initialSize` for warm-up**: Set `initialSize` to pre-create commonly used
   components, reducing frame drops during gameplay.

6. **Pool behavior is LIFO**: The pool uses a stack (Last In, First Out) internally, meaning
   the most recently returned component will be the next one acquired.
