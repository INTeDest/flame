# Виджеты

Одним из преимуществ разработки игр с использованием Flutter является возможность применять обширный инструментарий Flutter для построения пользовательских интерфейсов. Flame стремится расширить эти возможности, вводя виджеты, созданные специально с учётом потребностей игр.

Здесь вы найдёте все доступные виджеты, предоставляемые Flame.

Все виджеты также можно увидеть в изолированной среде [Dashbook](https://github.com/bluefireteam/dashbook) в [каталоге примеров виджетов](https://github.com/flame-engine/flame/tree/main/examples/lib/stories/widgets).


## NineTileBoxWidget

Nine Tile Box — это прямоугольник, рисуемый с использованием сеточного спрайта.

Сеточный спрайт представляет собой сетку 3x3 из 9 блоков, соответствующих 4 углам, 4 сторонам и середине.

Углы отрисовываются без растяжения, стороны растягиваются в соответствующем направлении, а середина расширяется в обоих направлениях.

`NineTileBoxWidget` реализует `Container` по этому стандарту. Этот шаблон также реализован в виде компонента в `NineTileBoxComponent`, так что вы можете добавить эту функциональность непосредственно в ваш `FlameGame`. Подробнее см. в [документации по NineTileBoxComponent](../components/utility_components.md#ninetileboxcomponent).

Пример использования (без `NineTileBoxComponent`):

```dart
import 'package:flame/widgets';

NineTileBoxWidget(
    image: image, // экземпляр dart:ui image
    tileSize: 16, // Ширина/высота тайла на вашем сеточном изображении
    destTileSize: 50, // Размеры тайла на холсте
    child: SomeWidget(), // Любой Flutter-виджет
)
```


## SpriteButton

`SpriteButton` — это простой виджет, создающий кнопку на основе спрайтов Flame. Это может быть очень полезно, когда нужно сделать кнопки нестандартного вида. Например, когда вам проще достичь желаемого внешнего вида, нарисовав кнопку в графическом редакторе, чем создавая её напрямую во Flutter.

Как использовать:

```dart
SpriteButton(
    onPressed: () {
      print('Pressed');
    },
    label: const Text('Sprite Button', style: const TextStyle(color: const Color(0xFF5D275D))),
    sprite: _spriteButton,
    pressedSprite: _pressedSprite,
    // Опционально, будет показан, когда onPressed равен null.
    disabledSprite: _disabledSprite,
    height: _height,
    width: _width,
)
```


## SpriteWidget

`SpriteWidget` — это виджет, используемый для отображения [Sprite](../rendering/images.md#sprite) в дереве виджетов.

Вот как его использовать:

```dart
SpriteWidget(
    sprite: yourSprite,
    anchor: Anchor.center,
)
```


## SpriteAnimationWidget

`SpriteAnimationWidget` — это виджет, используемый для отображения [SpriteAnimations](../rendering/images.md#animation) в дереве виджетов.

Вот как его использовать:

```dart
SpriteAnimationWidget(
    animation: _animation,
    animationTicker: _animationTicker,
    playing: true,
    anchor: Anchor.center,
)
```
