# Служебные компоненты

Помимо основных визуальных компонентов, Flame предоставляет ряд служебных компонентов, решающих типичные задачи разработки игр: создание объектов с течением времени, рендеринг тайловых карт, обрезка областей рендеринга и интеграция виджетов Flutter в игру. Эти компоненты избавляют от написания шаблонного кода и позволяют сосредоточиться на игровой логике.


## SpawnComponent

Это невизуальный компонент, который создаёт (спавнит) другие компоненты внутри родителя `SpawnComponent`. Он отлично подходит, например, для случайного появления врагов или бонусов в заданной области.

`SpawnComponent` принимает фабричную функцию, используемую для создания новых компонентов, и область, внутри (или вдоль границ) которой они должны появляться.

Для области можно использовать классы `Circle`, `Rectangle` или `Polygon`, а если нужно создавать компоненты только вдоль краёв фигуры, установите аргумент `within` в `false` (по умолчанию `true`).

Пример показывает создание компонентов типа `MyComponent` каждые 0.5 секунды случайным образом внутри заданного круга.

Компонент поддерживает два вида фабрик. `factory` возвращает один компонент, а `multiFactory` — список компонентов, добавляемых за один шаг.

Фабричные функции принимают `int` — количество уже созданных компонентов (нумерация с 0). Если уже создано 4 компонента, пятый вызов фабрики получит `amount=4`.

`factory` с одним компонентом сохранён для обратной совместимости, поэтому в общем случае лучше использовать `multiFactory`. Однокомпонентная фабрика внутренне оборачивается в список и используется как `multiFactory`.

Если требуется создать только определённое количество компонентов, используйте аргумент `spawnCount`; по достижении лимита `SpawnComponent` прекратит спавн и удалится.

По умолчанию `SpawnComponent` добавляет компоненты к своему родителю, но можно указать другой целевой компонент через аргумент `target`. Помните, что целевой компонент должен иметь размер, если не используются аргументы `area` или `selfPositioning`.

```dart
SpawnComponent(
  factory: (i) => MyComponent(size: Vector2(10, 20)),
  period: 0.5,
  area: Circle(Vector2(100, 200), 150),
);
```

Если частота появления не должна быть фиксированной, используйте конструктор `SpawnComponent.periodRange` с аргументами `minPeriod` и `maxPeriod`. В следующем примере компонент появляется случайно внутри круга, а интервал между появлениями составляет от 0.5 до 10 секунд.

```dart
SpawnComponent.periodRange(
  factory: (i) => MyComponent(size: Vector2(10, 20)),
  minPeriod: 0.5,
  maxPeriod: 10,
  area: Circle(Vector2(100, 200), 150),
);
```

Если вы хотите задавать позицию самостоятельно внутри фабричной функции, установите `selfPositioning = true` в конструкторе — тогда вы сможете указывать позиции и игнорировать аргумент `area`.

```dart
SpawnComponent(
  factory: (i) =>
    MyComponent(position: Vector2(100, 200), size: Vector2(10, 20)),
  selfPositioning: true,
  period: 0.5,
);
```


## SvgComponent

**Примечание**: для использования SVG в Flame используйте пакет [`flame_svg`](https://github.com/flame-engine/flame_svg).

Этот компонент использует экземпляр класса `Svg` для представления компонента с SVG-графикой в игре:

```dart
@override
Future<void> onLoad() async {
  final svg = await Svg.load('android.svg');
  final android = SvgComponent.fromSvg(
    svg,
    position: Vector2.all(100),
    size: Vector2.all(100),
  );
}
```


## IsometricTileMapComponent

Изометрические тайловые карты часто используются в стратегиях, симуляторах и RPG для придания двухмерной карте псевдотрёхмерной перспективы. Этот компонент позволяет отрисовывать изометрическую карту на основе декартовой матрицы блоков и изометрического тайлсета.

Простой пример использования:

```dart
// Создаётся тайлсет, идентификаторы блоков назначаются автоматически последовательно,
// начиная с 0, слева направо и сверху вниз.
final tilesetImage = await images.load('tileset.png');
final tileset = SpriteSheet(image: tilesetImage, srcSize: Vector2.all(32));
// Каждый элемент — идентификатор блока, -1 означает пустоту
final matrix = [[0, 1, 0], [1, 0, 0], [1, 1, 1]];
add(IsometricTileMapComponent(tileset, matrix));
```

Он также предоставляет методы для преобразования координат, чтобы обрабатывать клики, наведение, отображать сущности поверх тайлов, добавлять выделение и т.д.

Также можно указать `tileHeight` — вертикальное расстояние между нижней и верхней плоскостями каждого кубоида (плитки). По сути, это высота передней грани вашего кубоида; обычно она равна половине (по умолчанию) или четверти размера плитки. На изображении ниже высота показана более тёмным тоном:

![Пример определения tileHeight](../../images/tile-height-example.png)

Так выглядит карта с высотой в четверть:

![Пример изометрической карты с выделением](../../images/isometric.png)

Пример приложения Flame содержит более подробный пример с разбором координат для создания выделения. [Исходный код](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/rendering/isometric_tile_map_example.dart) доступен на GitHub, а [живая версия](https://examples.flame-engine.org/#/Rendering_Isometric_Tile_Map) — в браузере.


## NineTileBoxComponent

Nine Tile Box — это прямоугольник, рисуемый с использованием сеточного спрайта.

Сеточный спрайт представляет собой сетку 3x3 из 9 блоков, соответствующих четырём углам, четырём сторонам и середине.

Углы отрисовываются без растяжения, стороны растягиваются в соответствующем направлении, а середина расширяется в обоих направлениях.

Таким образом можно получить прямоугольник, хорошо масштабируемый до любых размеров. Это полезно для создания панелей, диалогов, рамок.

Пример использования смотрите в приложении-примере [nine_tile_box](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/rendering/nine_tile_box_example.dart).


## CustomPainterComponent

`CustomPainter` — это класс Flutter, используемый с виджетом `CustomPaint` для отрисовки пользовательских фигур в приложении Flutter.

Flame предоставляет компонент `CustomPainterComponent`, который рендерит `CustomPainter` на игровом холсте.

Это позволяет переиспользовать логику отрисовки между игрой Flame и виджетами Flutter.

Пример использования смотрите в [custom_painter_component](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/widgets/custom_painter_example.dart).


## ComponentsNotifier

В большинстве случаев простого доступа к дочерним компонентам и их атрибутам достаточно для построения логики игры. Однако иногда реактивность помогает упростить код, и для этого Flame предоставляет `ComponentsNotifier` — реализацию `ChangeNotifier`, уведомляющую слушателей при каждом добавлении, удалении или ручном изменении компонента.

Например, мы хотим показать сообщение об окончании игры, когда жизни игрока достигнут нуля.

Чтобы компонент автоматически сообщал о добавлении или удалении новых экземпляров, к классу компонента можно применить примесь `Notifier`:

```dart
class Player extends SpriteComponent with Notifier {}
```

Для прослушивания изменений этого компонента используйте метод `componentsNotifier` из `FlameGame`:

```dart
class MyGame extends FlameGame {
  int lives = 2;

  @override
  void onLoad() {
    final playerNotifier = componentsNotifier<Player>()
        ..addListener(() {
          final player = playerNotifier.single;
          if (player == null) {
            lives--;
            if (lives == 0) {
              add(GameOverComponent());
            } else {
              add(Player());
            }
          }
        });
  }
}
```

Компонент-`Notifier` также может вручную уведомить слушателей об изменении. Расширим пример выше: пусть компонент интерфейса мигает, когда у игрока осталась половина здоровья. Для этого `Player` должен вручную сообщить об изменении:

```dart
class Player extends SpriteComponent with Notifier {
  double health = 1;

  void takeHit() {
    health -= .1;
    if (health == 0) {
      removeFromParent();
    } else if (health <= .5) {
      notifyListeners();
    }
  }
}
```

Тогда HUD-компонент может выглядеть так:

```dart
class Hud extends PositionComponent with HasGameReference {

  @override
  void onLoad() {
    final playerNotifier = game.componentsNotifier<Player>()
        ..addListener(() {
          final player = playerNotifier.single;
          if (player != null) {
            if (player.health <= .5) {
              add(BlinkEffect());
            }
          }
        });
  }
}
```

`ComponentsNotifier` также может пригодиться для перестроения виджетов при изменении состояния в `FlameGame`, и для этого Flame предоставляет виджет `ComponentsNotifierBuilder`.

Пример использования смотрите в [ComponentsNotifier example](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/components/components_notifier_example.dart).


## ClipComponent

`ClipComponent` — это компонент, который обрезает холст по своему размеру и форме. Это значит, что если сам компонент или любой его дочерний элемент выходит за границы `ClipComponent`, часть, находящаяся за пределами области, не будет отображаться.

`ClipComponent` принимает функцию-строитель, возвращающую `Shape`, определяющую область обрезки на основе размера.

Для удобства использования компонента есть три фабрики, предлагающие распространённые формы:

- `ClipComponent.rectangle`: Обрезает область в форме прямоугольника на основе размера.
- `ClipComponent.circle`: Обрезает область в форме круга на основе размера.
- `ClipComponent.polygon`: Обрезает область в форме многоугольника на основе точек, переданных в конструктор.

Пример использования смотрите в приложении-примере [clip_component](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/components/clip_component_example.dart).
