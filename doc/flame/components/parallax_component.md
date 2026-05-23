# ParallaxComponent

Параллаксная прокрутка — классический приём в разработке игр, когда фоновые слои движутся с разной скоростью, создавая иллюзию глубины. Объекты, расположенные ближе к камере, перемещаются быстрее, чем удалённые. Так же, как при взгляде из окна автомобиля: близкие деревья проносятся мимо, а далёкие горы едва движутся. Этот эффект делает двухмерные игровые миры более захватывающими и часто применяется в сайд-скроллерах, платформерах и экранах меню.

Этот `Component` позволяет отрисовывать фон с ощущением глубины, накладывая друг на друга несколько прозрачных изображений, каждое из которых (или анимация — `ParallaxRenderer`) движется с собственной скоростью.

Простейший `ParallaxComponent` создаётся так:

```dart
@override
Future<void> onLoad() async {
  final parallaxComponent = await loadParallaxComponent([
    ParallaxImageData('bg.png'),
    ParallaxImageData('trees.png'),
  ]);
  add(parallaxComponent);
}
```

ParallaxComponent также может «загружать себя сам», реализуя метод `onLoad`:

```dart
class MyParallaxComponent extends ParallaxComponent<MyGame> {
  @override
  Future<void> onLoad() async {
    parallax = await game.loadParallax([
      ParallaxImageData('bg.png'),
      ParallaxImageData('trees.png'),
    ]);
  }
}

class MyGame extends FlameGame {
  @override
  void onLoad() {
    add(MyParallaxComponent());
  }
}
```

Так создаётся статичный фон. Если нужен движущийся параллакс (в чём, собственно, и заключается его смысл), это можно сделать несколькими способами в зависимости от желаемой детализации настроек каждого слоя.

Самый простой путь — задать именованные опциональные параметры `baseVelocity` и `velocityMultiplierDelta` во вспомогательной функции `load`. Например, чтобы фоновые изображения двигались по оси X тем быстрее, чем они «ближе»:

```dart
@override
Future<void> onLoad() async {
  final parallaxComponent = await loadParallaxComponent(
    _dataList,
    baseVelocity: Vector2(20, 0),
    velocityMultiplierDelta: Vector2(1.8, 1.0),
  );
}
```

Вы можете изменить `baseSpeed` и `layerDelta` в любой момент, например, когда персонаж прыгает или игра ускоряется.

```dart
@override
void onLoad() {
  final parallax = parallaxComponent.parallax;
  parallax.baseSpeed = Vector2(100, 0);
  parallax.velocityMultiplierDelta = Vector2(2.0, 1.0);
}
```

По умолчанию изображения выравниваются по нижнему левому краю, повторяются по оси X и пропорционально масштабируются так, чтобы покрыть высоту экрана. Если требуется изменить это поведение — например, для игры не в жанре сайд-скроллера, — можно задать параметры `repeat`, `alignment` и `fill` для каждого `ParallaxRenderer` и добавить их в `ParallaxLayer`, которые затем передаются в конструктор `ParallaxComponent`.

Продвинутый пример:

```dart
final images = [
  loadParallaxImage(
    'stars.jpg',
    repeat: ImageRepeat.repeat,
    alignment: Alignment.center,
    fill: LayerFill.width,
  ),
  loadParallaxImage(
    'planets.jpg',
    repeat: ImageRepeat.repeatY,
    alignment: Alignment.bottomLeft,
    fill: LayerFill.none,
  ),
  loadParallaxImage(
    'dust.jpg',
    repeat: ImageRepeat.repeatX,
    alignment: Alignment.topRight,
    fill: LayerFill.height,
  ),
];

final layers = images.map(
  (image) => ParallaxLayer(
    await image,
    velocityMultiplier: images.indexOf(image) * 2.0,
  )
);

final parallaxComponent = ParallaxComponent.fromParallax(
  Parallax(
    await Future.wait(layers),
    baseVelocity: Vector2(50, 0),
  ),
);
```

- Изображение звёзд в этом примере будет повторяться по обеим осям, выравниваться по центру и масштабироваться, заполняя ширину экрана.
- Изображение планет будет повторяться по оси Y, выравниваться по нижнему левому краю и не масштабироваться.
- Изображение пыли будет повторяться по оси X, выравниваться по верхнему правому краю и масштабироваться по высоте экрана.

Завершив настройку `ParallaxComponent`, добавьте его в игру, как и любой другой компонент (`game.add(parallaxComponent`).
Не забудьте прописать изображения в файле `pubspec.yaml` как assets, иначе они не будут найдены.

Файл `Parallax` содержит расширение игры, добавляющее методы `loadParallax`, `loadParallaxLayer`, `loadParallaxImage` и `loadParallaxAnimation`, которые автоматически используют кеш изображений вашей игры вместо глобального. То же относится и к файлу `ParallaxComponent`, но он предоставляет `loadParallaxComponent`.

Если нужен полноэкранный `ParallaxComponent`, просто опустите аргумент `size`, и он примет размер игры, а также будет подстраиваться при изменении размера или ориентации экрана.

Flame предоставляет два вида `ParallaxRenderer`: `ParallaxImage` и `ParallaxAnimation`. `ParallaxImage` — это статичный рендерер изображений, а `ParallaxAnimation`, как следует из названия, — рендерер на основе анимации и кадров.
Также можно создавать собственные рендереры, расширив класс `ParallaxRenderer`.

Три примера реализации можно найти в [каталоге примеров](https://github.com/flame-engine/flame/tree/main/examples/lib/stories/parallax).
