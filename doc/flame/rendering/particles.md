# Частицы

Flame предлагает простую, но мощную и расширяемую систему частиц. Ключевой концепцией этой системы является класс `Particle`, который по своему поведению очень похож на `ParticleSystemComponent`.

Самое простое использование `Particle` с `FlameGame` выглядит следующим образом:

```dart
import 'package:flame/components.dart';

// ...

game.add(
  // Оборачиваем Particle в ParticleSystemComponent,
  // который сопоставляет хуки жизненного цикла Component с хуками Particle
  // и встраивает триггер для удаления компонента.
  ParticleSystemComponent(
    particle: CircleParticle(),
  ),
);
```

При использовании `Particle` с пользовательской реализацией `Game` убедитесь, что методы `update` и `render` вызываются во время каждого тика игрового цикла.

Основные подходы к реализации желаемых эффектов частиц:

- Композиция существующих поведений.
- Использование цепочки поведений (просто синтаксический сахар для первого подхода).
- Использование `ComputedParticle`.

Композиция работает подобно виджетам Flutter, определяя эффект сверху вниз. Цепочка позволяет более гибко выражать те же деревья композиции, определяя поведения снизу вверх. Вычисляемые частицы, в свою очередь, полностью делегируют реализацию поведения вашему коду. Любой из подходов можно использовать совместно с существующими поведениями там, где это необходимо.

```dart
Random rnd = Random();

Vector2 randomVector2() => (Vector2.random(rnd) - Vector2.random(rnd)) * 200;

// Композиция.
//
// Определение эффекта частиц как набора вложенных поведений сверху вниз,
// одно внутри другого:
//
// ParticleSystemComponent
//   > ComposedParticle
//     > AcceleratedParticle
//       > CircleParticle
game.add(
  ParticleSystemComponent(
    particle: Particle.generate(
      count: 10,
      generator: (i) => AcceleratedParticle(
        acceleration: randomVector2(),
        child: CircleParticle(
          paint: Paint()..color = Colors.red,
        ),
      ),
    ),
  ),
);

// Цепочка.
//
// Выражает то же поведение, что и выше, но с более гибким API.
// Только частицы с примесью SingleChildParticle могут использоваться
// как цепочки поведений.
game.add(
  ParticleSystemComponent(
    particle: Particle.generate(
      count: 10,
      generator: (i) => pt.CircleParticle(paint: Paint()..color = Colors.red)
    )
  )
);

// Computed Particle.
//
// Все поведения определены явно. Предоставляет большую гибкость
// по сравнению со встроенными поведениями.
game.add(
  ParticleSystemComponent(
      particle: Particle.generate(
        count: 10,
        generator: (i) {
          Vector2 position = Vector2.zero();
          Vector2 speed = Vector2.zero();
          final acceleration = randomVector2();
          final paint = Paint()..color = Colors.red;

          return ComputedParticle(
            renderer: (canvas, _) {
              speed += acceleration;
              position += speed;
              canvas.drawCircle(Offset(position.x, position.y), 1, paint);
            }
        );
      }
    )
  )
);
```

Смотрите больше [примеров использования встроенных частиц в различных комбинациях](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/rendering/particles_example.dart).


## Жизненный цикл

Поведение, общее для всех `Particle`, заключается в том, что все они принимают аргумент `lifespan` (время жизни). Это значение используется, чтобы `ParticleSystemComponent` самоудалился, когда его внутренний `Particle` достигнет конца своей жизни. Время внутри самого `Particle` отслеживается с помощью класса Flame `Timer`. Его можно настроить, передав `double`, представляющий секунды (с микросекундной точностью), в конструктор соответствующего `Particle`.

```dart
Particle(lifespan: .2); // будет жить 200 мс.
Particle(lifespan: 4); // будет жить 4 с.
```

Также можно сбросить время жизни `Particle` с помощью метода `setLifespan`, который также принимает `double` в секундах.

```dart
final particle = Particle(lifespan: 2);

// ... через некоторое время.
particle.setLifespan(2) // будет жить еще 2 с.
```

В течение своей жизни `Particle` отслеживает прожитое время и предоставляет его через геттер `progress`, который возвращает значение от `0.0` до `1.0`. Это значение можно использовать подобно свойству `value` класса `AnimationController` во Flutter.

```dart
final particle = Particle(lifespan: 2.0);

game.add(ParticleSystemComponent(particle: particle));

// Будет печатать значения от 0 до 1 с шагом .1: 0, 0.1, 0.2 ... 0.9, 1.0.
Timer.periodic(duration * .1, () => print(particle.progress));
```

`lifespan` передается всем потомкам данного `Particle`, если он поддерживает какое-либо из поведений вложенности.


## Встроенные частицы

Flame поставляется с несколькими встроенными поведениями `Particle`:

- `TranslatedParticle` перемещает своего `child` на заданный `Vector2`
- `MovingParticle` перемещает своего `child` между двумя заданными `Vector2`, поддерживает `Curve`
- `AcceleratedParticle` позволяет создавать простые физические эффекты, такие как гравитация или затухание скорости
- `CircleParticle` рендерит круги любых форм и размеров
- `SpriteParticle` рендерит Flame `Sprite` внутри эффекта `Particle`
- `ImageParticle` рендерит *dart:ui* `Image` внутри эффекта `Particle`
- `ComponentParticle` рендерит Flame `Component` внутри эффекта `Particle`
- `FlareParticle` рендерит анимацию Flare внутри эффекта `Particle`

Смотрите больше [примеров совместного использования встроенных поведений частиц](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/rendering/particles_example.dart).
Все реализации доступны в [папке particles в репозитории Flame.](https://github.com/flame-engine/flame/tree/main/packages/flame/lib/src/particles)


## TranslatedParticle

Просто смещает нижележащий `Particle` на заданный `Vector2` в пределах рендерящего `Canvas`. Не изменяет и не меняет его положение; если требуется изменение положения, рассмотрите использование `MovingParticle` или `AcceleratedParticle`. Такого же эффекта можно добиться, сместив слой `Canvas`.

```dart
game.add(
  ParticleSystemComponent(
    particle: TranslatedParticle(
      // Переместит дочерний эффект Particle в центр игрового холста.
      offset: game.size / 2,
      child: Particle(),
    ),
  ),
);
```


## MovingParticle

Перемещает дочерний `Particle` между `Vector2` `from` и `to` в течение его времени жизни. Поддерживает `Curve` через `CurvedParticle`.

```dart
game.add(
  ParticleSystemComponent(
    particle: MovingParticle(
      // Будет перемещаться от угла к углу игрового холста.
      from: Vector2.zero(),
      to: game.size,
      child: CircleParticle(
        radius: 2.0,
        paint: Paint()..color = Colors.red,
      ),
    ),
  ),
);
```


## AcceleratedParticle

Базовая физическая частица, позволяющая задать начальные `position`, `speed` и `acceleration` и позволяющая циклу `update` сделать всё остальное. Все три задаются как `Vector2`, которые можно рассматривать как векторы. Это особенно хорошо работает для физических "взрывов", но не ограничивается ими. Единица измерения значения `Vector2` — *логические пиксели/с*. Так, скорость `Vector2(0, 100)` будет перемещать дочерний `Particle` на 100 логических пикселей устройства каждую секунду игрового времени.

```dart
final rnd = Random();
Vector2 randomVector2() => (Vector2.random(rnd) - Vector2.random(rnd)) * 100;

game.add(
  ParticleSystemComponent(
    particle: AcceleratedParticle(
      // Будет выброшена из центра игрового холста
      position: game.canvasSize/2,
      // Со случайной начальной скоростью Vector2(-100..100, 0..-100)
      speed: Vector2(rnd.nextDouble() * 200 - 100, -rnd.nextDouble() * 100),
      // Ускорение вниз, имитирующее "гравитацию"
      // speed: Vector2(0, 100),
      child: CircleParticle(
        radius: 2.0,
        paint: Paint()..color = Colors.red,
      ),
    ),
  ),
);
```


## CircleParticle

`Particle`, который рендерит круг с заданным `Paint` в нулевом смещении переданного `Canvas`. Используйте совместно с `TranslatedParticle`, `MovingParticle` или `AcceleratedParticle` для достижения желаемого позиционирования.

```dart
game.add(
  ParticleSystemComponent(
    particle: CircleParticle(
      radius: game.size.x / 2,
      paint: Paint()..color = Colors.red.withValues(alpha: .5),
    ),
  ),
);
```


## SpriteParticle

Позволяет встроить `Sprite` в ваши эффекты частиц.

```dart
game.add(
  ParticleSystemComponent(
    particle: SpriteParticle(
      sprite: Sprite('sprite.png'),
      size: Vector2(64, 64),
    ),
  ),
);
```


## ImageParticle

Рендерит заданное изображение `dart:ui` в дереве частиц.

```dart
// Во время инициализации игры
await Flame.images.loadAll(const [
  'image.png',
]);

// ...

// Где-то в игровом цикле
final image = await Flame.images.load('image.png');

game.add(
  ParticleSystemComponent(
    particle: ImageParticle(
      size: Vector2.all(24),
      image: image,
    );
  ),
);
```


## ScalingParticle

Масштабирует дочерний `Particle` от `1` до `to` в течение его времени жизни.

```dart
game.add(
  ParticleSystemComponent(
    particle: ScalingParticle(
      lifespan: 2,
      to: 0,
      curve: Curves.easeIn,
      child: CircleParticle(
        radius: 2.0,
        paint: Paint()..color = Colors.red,
      )
    );
  ),
);
```


## SpriteAnimationParticle

`Particle`, который встраивает `SpriteAnimation`.
По умолчанию выравнивает `stepTime` `SpriteAnimation` так, чтобы она полностью проигрывалась за время жизни `Particle`. Это поведение можно переопределить с помощью аргумента `alignAnimationTime`.

```dart
final spriteSheet = SpriteSheet(
  image: yourSpriteSheetImage,
  srcSize: Vector2.all(16.0),
);

game.add(
  ParticleSystemComponent(
    particle: SpriteAnimationParticle(
      animation: spriteSheet.createAnimation(0, stepTime: 0.1),
    );
  ),
);
```


## ComponentParticle

Этот `Particle` позволяет встроить `Component` в эффекты частиц. `Component` может иметь свой собственный жизненный цикл `update` и может повторно использоваться в разных деревьях эффектов. Если вам нужно только добавить динамику к экземпляру определённого `Component`, рассмотрите возможность добавления его в `game` напрямую, без `Particle`.

```dart
final longLivingRect = RectComponent();

game.add(
  ParticleSystemComponent(
    particle: ComponentParticle(
      component: longLivingRect
    );
  ),
);

class RectComponent extends Component {
  void render(Canvas c) {
    c.drawRect(
      Rect.fromCenter(center: Offset.zero, width: 100, height: 100),
      Paint()..color = Colors.red
    );
  }

  void update(double dt) {
    /// Будет вызываться родительским [Particle]
  }
}
```


## ComputedParticle

`Particle`, который может помочь вам, когда:

- Стандартного поведения недостаточно
- Требуется оптимизация сложных эффектов
- Нужны пользовательские функции плавности

При создании он делегирует весь рендеринг предоставленному `ParticleRenderDelegate`, который вызывается на каждом кадре для выполнения необходимых вычислений и рендеринга чего-либо на `Canvas`.

```dart
game.add(
  ParticleSystemComponent(
    // Рендерит круг, который постепенно меняет цвет и размер
    // в течение жизни частицы.
    particle: ComputedParticle(
      renderer: (canvas, particle) => canvas.drawCircle(
        Offset.zero,
        particle.progress * 10,
        Paint()
          ..color = Color.lerp(
            Colors.red,
            Colors.blue,
            particle.progress,
          ),
      ),
    ),
  ),
)
```


## Поведение вложенности

Реализация частиц во Flame следует той же модели экстремальной композиции, что и виджеты Flutter. Это достигается путём инкапсуляции небольших фрагментов поведения в каждой частице и последующего вложения этих поведений друг в друга для достижения желаемого визуального эффекта.

Две сущности, позволяющие `Particle` вкладываться друг в друга: примесь `SingleChildParticle` и класс `ComposedParticle`.

`SingleChildParticle` может помочь вам в создании `Particle` с пользовательским поведением. Например, случайное позиционирование дочернего элемента на каждом кадре:

```dart
var rnd = Random();

class GlitchParticle extends Particle with SingleChildParticle {
  Particle child;

  GlitchParticle({
    required this.child,
    super.lifespan,
  });

  @override
  render(Canvas canvas)  {
    canvas.save();
    canvas.translate(rnd.nextDouble() * 100, rnd.nextDouble() * 100);

    // Также отрендерит дочерний элемент
    super.render();

    canvas.restore();
  }
}
```

`ComposedParticle` можно использовать как отдельно, так и внутри существующего дерева `Particle`.
