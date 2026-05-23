# Начало работы


## О Flame

Flame — это модульный игровой движок на Flutter, предоставляющий полный набор готовых решений для игр. Он использует мощную инфраструктуру Flutter, но упрощает код, необходимый для создания ваших проектов.

Движок предлагает простую, но эффективную реализацию игрового цикла и все необходимые функции, которые могут понадобиться в игре: ввод, изображения, спрайты, спрайт-листы, анимации, обнаружение столкновений и компонентную систему, которую мы называем Flame Component System (сокращённо FCS).

Мы также предоставляем отдельные пакеты, расширяющие функциональность Flame, их можно найти в разделе [Bridge Packages](bridge_packages/bridge_packages.md).

Вы можете выбирать и использовать только те части, которые нужны, поскольку все они независимы и модульны.

Движок и его экосистема постоянно улучшаются сообществом, поэтому не стесняйтесь обращаться, открывать issues и PR, а также вносить предложения.

Поставьте нам звёздочку, если хотите помочь движку стать известнее и развить сообщество. :)


## Установка

Добавьте пакет `flame` в зависимости вашего `pubspec.yaml`, выполнив следующую команду:

```console
flutter pub add flame
```

Последнюю версию можно найти на [pub.dev](https://pub.dev/packages/flame/install).

Затем выполните `flutter pub get`, и вы готовы начать использовать его!


## Начало работы

В папке [tutorials](https://github.com/flame-engine/flame/tree/main/doc/tutorials) есть несколько руководств, которые помогут вам начать.

Простые примеры для всех возможностей можно найти в папке [examples](https://github.com/flame-engine/flame/tree/main/examples).

Для запуска Flame используйте `GameWidget` — это обычный виджет, который может находиться в любом месте вашего дерева виджетов. Вы можете использовать его как корневой виджет приложения или как дочерний элемент другого виджета.

Вот простой пример использования `GameWidget`:

```dart
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(
    GameWidget(
      game: FlameGame(),
    ),
  );
}
```

Во Flame мы предлагаем концепцию, называемую Flame Component System (FCS), — это способ организации игровых объектов, упрощающий управление ими. Подробнее об этом можно прочитать в разделе [Components](flame/components/components.md).

Когда вы начинаете новую игру, вам нужно либо расширить класс `FlameGame`, либо класс `World`. `FlameGame` — это корень вашей игры, отвечающий за игровой цикл и компоненты. Класс `World` — это компонент, который можно использовать для создания игрового мира.

Итак, для создания простой игры можно сделать что-то подобное:

```dart
import 'package:flame/game.dart';
import 'package:flame/components.dart';
import 'package:flutter/widgets.dart';

void main() {
  runApp(
    GameWidget(
      game: FlameGame(world: MyWorld()),
    ),
  );
}

class MyWorld extends World {
  @override
  Future<void> onLoad() async {
    add(Player(position: Vector2(0, 0)));
  }
}
```

Как видите, мы создали класс `MyWorld`, расширяющий `World`. Мы переопределили метод `onLoad`, чтобы добавить в мир компонент `Player` (которого пока не существует). В классе `FlameGame` по умолчанию есть камера, которая наблюдает за миром, и по умолчанию она смотрит на точку (0, 0) мира в центре экрана. Чтобы узнать больше о камере и мире, прочитайте раздел [Camera Component](flame/camera.md).

Компонент `Player` может быть любым нужным вам типом компонента. Для начала мы рекомендуем использовать класс `SpriteComponent` — компонент, который может отображать спрайт (изображение) на экране.

Например, так:

```dart
import 'package:flame/components.dart';
import 'package:flame/geometry.dart';
import 'package:flame/extensions.dart';

class Player extends SpriteComponent {
  Player({super.position}) :
    super(size: Vector2.all(200), anchor: Anchor.center);

  @override
  Future<void> onLoad() async {
    sprite = await Sprite.load('player.png');
  }
}
```

В этом примере мы создали класс `Player`, расширяющий `SpriteComponent`. Мы переопределили `onLoad`, чтобы установить спрайт компонента, загружаемый из файла изображения `player.png`. Изображение должно находиться в папке `assets/images` вашего проекта (см. [Assets Directory Structure](flame/structure.md)), и его нужно добавить в [секцию assets](https://docs.flutter.dev/ui/assets/assets-and-images) файла `pubspec.yaml`. В этом классе мы также устанавливаем размер компонента 200x200 и [якорь](flame/components/position_component.md#anchor) в центр, передавая их в конструктор суперкласса. Пользователь класса `Player` может задавать позицию компонента при его создании (`Player(position: Vector2(0, 0))`).

Для обработки ввода в компоненте вы можете добавить любой из наших [примесей ввода](flame/inputs/inputs.md) к компоненту. Например, чтобы обрабатывать касания, добавьте примесь `TapCallbacks` к компоненту игрока, и вы будете получать события касания в границах компонента. Если же нужно обрабатывать касания по всему миру, добавьте `TapCallbacks` к расширенному классу `World`.

Следующий пример обрабатывает касания компонента игрока: когда по игроку нажимают, его размер увеличивается на 50 пикселей по ширине и высоте.

```dart
import 'package:flame/components.dart';
import 'package:flame/geometry.dart';
import 'package:flame/extensions.dart';

class Player extends SpriteComponent with TapCallbacks {
  Player({super.position}) :
    super(size: Vector2.all(200), anchor: Anchor.center);

  @override
  Future<void> onLoad() async {
    sprite = await Sprite.load('player.png');
  }
  
  @override
  void onTapUp(TapUpEvent info) {
    size += Vector2.all(50);
  }
}
```

Это лишь простой пример того, как начать работу с Flame. Существует гораздо больше функций, которые вы можете использовать (и, вероятно, они вам понадобятся) для создания вашей игры, но этот пример должен дать вам хорошую отправную точку.

Также посмотрите репозиторий [awesome flame](https://github.com/flame-engine/awesome-flame#user-content-articles--tutorials) — он содержит множество отличных руководств и статей, написанных сообществом, которые помогут вам начать работу с Flame.


## За рамками движка

В зависимости от специфики игры могут потребоваться сложные наборы функций. Некоторые из них выходят за рамки экосистемы Flame Engine. В этом разделе они перечислены, а также даны рекомендации по пакетам и сервисам, которые можно использовать:


### Мультиплеер (сетевой код)

Flame не включает никаких сетевых функций, которые могут понадобиться для создания многопользовательских онлайн-игр.

Если вы разрабатываете мультиплеерную игру, вот несколько рекомендаций по пакетам и сервисам:

- [Nakama](https://github.com/obrunsmann/flutter_nakama/): Сервер с открытым исходным кодом, созданный для поддержки современных игр и приложений.
- [Firebase](https://firebase.google.com/): Предоставляет десятки сервисов, которые можно использовать для создания простых мультиплеерных возможностей.
- [Supabase](https://supabase.com/): Более дешёвая альтернатива Firebase, основанная на Postgres.
