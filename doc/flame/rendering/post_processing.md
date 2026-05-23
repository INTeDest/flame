# Постобработка и шейдеры

Постобработка — это техника, используемая в разработке игр для применения визуальных эффектов к дереву компонентов после его рендеринга. После того как кадр отрисован (напрямую или растеризован в изображение), постобработка может изменить или улучшить визуальное представление.

Постобработка использует фрагментные шейдеры для создания динамических визуальных эффектов, таких как размытие, свечение, цветокоррекция, искажения и настройка освещения.

Система постобработки во Flame модульная и гибкая, она позволяет разработчикам:

- Определять собственные процессы постобработки, создавая подклассы абстрактного класса `PostProcess`.
- Применять одиночный эффект постобработки или объединять несколько эффектов в цепочки с помощью групп.
- Управлять эффектами глобально через `CameraComponent` или локально через `PostProcessComponent`.


## Ключевые компоненты системы постобработки

- **`PostProcess`**: Абстрактный базовый класс для определения пользовательских эффектов постобработки. Реализуйте логику эффекта в его методе `postProcess`.

- **`PostProcessComponent`**: Применяет постобработку именно к своим дочерним элементам, обеспечивая локальные эффекты.

- **`CameraComponent`**: Применяет постобработку глобально ко всей сцене или миру.

- **`PostProcessGroup`**: Применяет несколько процессов постобработки параллельно; полезно, когда эффекты могут быть применены независимо.

- **`PostProcessSequentialGroup`**: Применяет процессы постобработки последовательно, где каждый процесс использует результат предыдущего.


## PostProcessComponent

```{dartdoc}
:package: flame
:symbol: PostProcessComponent
:file: src/post_process/post_process_component.dart
```


## Создание пользовательского процесса постобработки

Чтобы реализовать собственный процесс постобработки:

1. Создайте подкласс `PostProcess`.
2. Переопределите метод `postProcess`, реализовав свою логику рендеринга с помощью `renderSubtree` или `rasterizeSubtree`.
3. При необходимости реализуйте методы `onLoad` и `update` для управления ресурсами и обновления эффектов каждый кадр.

Эта система позволяет легко добавлять креативные и полезные визуальные эффекты в вашу игру на Flame.


## Пример: пикселизация

Вот пример создания эффекта пикселизации с использованием фрагментного шейдера:

```dart
class PostProcessGame extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();

    world.add(
      PostProcessComponent(
        postProcess: PixelationPostProcess(),
        anchor: Anchor.center,
        children: [
          EmberPlayer(size: Vector2(100, 100)),
        ],
      ),
    );
  }
}

class PixelationPostProcess extends PostProcess {
  @override
  Future<void> onLoad() async {
    await super.onLoad();

    _fragmentProgram = await FragmentProgram.fromAsset(
      'packages/flutter_shaders/shaders/pixelation.frag',
    );
  }

  late final FragmentProgram _fragmentProgram;
  late final FragmentShader _fragmentShader = _fragmentProgram.fragmentShader();

  double _time = 0;

  @override
  void update(double dt) {
    super.update(dt);
    _time += dt;
  }

  late final myPaint = Paint()..shader = _fragmentShader;

  @override
  void postProcess(Vector2 size, Canvas canvas) {
    final preRenderedSubtree = rasterizeSubtree();

    _fragmentShader.setFloatUniforms((value) {
      value
        ..setVector(size / (20 * sin(_time)))
        ..setVector(size);
    });

    _fragmentShader.setImageSampler(0, preRenderedSubtree);

    canvas
      ..save()
      ..drawRect(Offset.zero & size.toSize(), myPaint)
      ..restore();
  }
}

```

В этом примере:

- Загружается фрагментный шейдер (`pixelation.frag`) и используется для применения эффекта пикселизации.

- Метод `rasterizeSubtree` захватывает рендеринг дерева компонентов в виде текстуры, которую шейдер использует для создания пикселизированного вывода.

- Эффект динамически изменяется с течением времени, создавая анимированный эффект пикселизации.

Этот пример демонстрирует, насколько просто добавлять визуальные эффекты в игру на Flame с помощью системы постобработки.

```{flutter-app}
:sources: ../flame/examples
:page: post_process
:show: widget code infobox
:width: 180
:height: 180
```

Файл шейдера пикселизации:

```glsl
#version 460 core

precision highp float;

#include <flutter/runtime_effect.glsl>

uniform vec2 uPixels;
uniform vec2 uSize;
uniform sampler2D uTexture;

out vec4 fragColor;

void main() {
  vec2 uv = FlutterFragCoord().xy / uSize;
  vec2 puv = round(uv * uPixels) / uPixels;
  fragColor = texture(uTexture, puv);
}
```


## Продвинутый пример: Хрустальный шар

Более сложный пример использования постобработки можно найти в [примере с Хрустальным шаром](https://examples.flame-engine.org/), где демонстрируется постобработка на уровне камеры и объединение нескольких эффектов с помощью `PostProcessSequentialGroup`.

![Пример Хрустального шара](../images/crystal_ball.png)

Вот как несколько эффектов постобработки комбинируются на камере:

```dart
class CrystalBallGame extends FlameGame<CrystalBallGameWorld> {

  CrystalBallGame() : super(
          camera: CameraComponent.withFixedResolution(
            width: kCameraSize.x,
            height: kCameraSize.y,
          ),
          world: CrystalBallGameWorld(),
        ) {
    camera.postProcess = PostProcessGroup(
      postProcesses: [
        PostProcessSequentialGroup(
          postProcesses: [
            FireflyPostProcess(),
            WaterPostProcess(),
          ],
        ),
        ForegroundFogPostProcess(),
      ],
    );
  }
}
```

В этом коде:

- Камера применяет `PostProcessGroup`, содержащий несколько эффектов.
- `PostProcessSequentialGroup` последовательно объединяет два эффекта (`FireflyPostProcess` и `WaterPostProcess`).
- Дополнительный параллельный эффект (`ForegroundFogPostProcess`) применяется вместе с последовательной группой.

Вы можете изучить исходный код [на GitHub](https://github.com/flame-engine/flame/tree/main/examples/games/crystal_ball).
