# Утилиты

На этой странице представлена документация по некоторым служебным классам и методам.


## Класс Device

```{warning}
Многие методы этого класса работают только на мобильных платформах (Android и iOS).

Использование этих методов на других платформах не даст никакого эффекта, и при запуске в режиме отладки в консоль будет выведено предупреждение.
```

Доступ к этому классу осуществляется через `Flame.device`. Он содержит ряд методов для управления состоянием устройства, например, можно изменить ориентацию экрана и установить, должно ли приложение быть полноэкранным.


### `Flame.device.fullScreen()`

При вызове отключает все `SystemUiOverlay`, делая приложение полноэкранным.
При вызове в главном методе ваше приложение становится полноэкранным (без верхней и нижней панелей).

**Примечание:** Не действует при вызове в вебе.


### `Flame.device.setLandscape()`

Устанавливает ориентацию всего приложения (а значит, и игры) в альбомную; в зависимости от операционной системы и настроек устройства должны поддерживаться обе альбомные ориентации (левая и правая). Чтобы задать приложению строго определённую альбомную ориентацию, используйте `Flame.device.setLandscapeLeftOnly` или `Flame.device.setLandscapeRightOnly`.

**Примечание:** Не действует при вызове в вебе.


### `Flame.device.setPortrait()`

Устанавливает ориентацию всего приложения (а значит, и игры) в портретную; в зависимости от операционной системы и настроек устройства должны поддерживаться обе портретные ориентации (вверх и вниз). Чтобы задать приложению строго определённую портретную ориентацию, используйте `Flame.device.setPortraitUpOnly` или `Flame.device.setPortraitDownOnly`.

**Примечание:** Не действует при вызове в вебе.


### `Flame.device.setOrientation()` и `Flame.device.setOrientations()`

Если требуется более точный контроль разрешённых ориентаций (без прямого обращения к `SystemChrome`), можно использовать `setOrientation` (принимает один параметр `DeviceOrientation`) и `setOrientations` (принимает `List<DeviceOrientation>` для возможных ориентаций).

**Примечание:** Не действуют при вызове в вебе.


## Таймер

Flame предоставляет простой служебный класс, помогающий обрабатывать обратный отсчёт и изменения состояния таймера, подобно событиям.

Пример обратного отсчёта:

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

class MyGame extends Game {
  final TextPaint textPaint = TextPaint(
    style: const TextStyle(color: Colors.white, fontSize: 20),
  );

  final countdown = Timer(2);

  @override
  void update(double dt) {
    countdown.update(dt);
    if (countdown.finished) {
      // Лучше использовать колбэк таймера, но в некоторых случаях подходит и так
    }
  }

  @override
  void render(Canvas canvas) {
    textPaint.render(
      canvas,
      "Countdown: ${countdown.current.toString()}",
      Vector2(10, 100),
    );
  }
}
```

Пример интервала:

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

class MyGame extends Game {
  final TextPaint textPaint = TextPaint(
    style: const TextStyle(color: Colors.white, fontSize: 20),
  );
  Timer interval;

  int elapsedSecs = 0;

  MyGame() {
    interval = Timer(
      1,
      onTick: () => elapsedSecs += 1,
      repeat: true,
    );
  }

  @override
  void update(double dt) {
    interval.update(dt);
  }

  @override
  void render(Canvas canvas) {
    textPaint.render(canvas, "Elapsed time: $elapsedSecs", Vector2(10, 150));
  }
}
```

Экземпляры `Timer` также можно использовать внутри игры `FlameGame` с помощью класса `TimerComponent`.

Пример `TimerComponent`:

```dart
import 'package:flame/timer.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';

class MyFlameGame extends FlameGame {
  MyFlameGame() {
    add(
      TimerComponent(
        period: 10,
        repeat: true,
        onTick: () => print('10 seconds elapsed'),
      )
    );
  }
}
```

```{note}
`Timer` или `TimerComponent` может повторяться бесконечно, если указан аргумент `repeat: true`, или повторяться определённое количество раз, если вместе с `repeat: true` используется аргумент `tickCount`.
```


## Масштаб времени

Во многих играх часто требуется создавать эффекты замедления или ускорения времени в зависимости от внутриигровых событий. Очень распространённый подход для достижения таких результатов — управление игровым временем или частотой тиков.

Для упрощения этого управления Flame предоставляет примесь `HasTimeScale`. Её можно прикрепить к любому `Component` из Flame, и она предоставляет простой API для получения/установки `timeScale`. Значение по умолчанию — `1`, что означает, что внутриигровое время компонента течёт с той же скоростью, что и реальное время. Установка в `2` заставит компонент тикать вдвое быстрее, а в `0.5` — вдвое медленнее по сравнению с реальным временем. Эта примесь также предоставляет методы `pause` и `resume`, которые можно использовать вместо ручной установки `timeScale` в 0 и 1 соответственно.

Поскольку `FlameGame` тоже является `Component`, эту примесь можно прикрепить и к `FlameGame`. Это позволит управлять масштабом времени для всех компонентов игры из одного места.

```{note}
HasTimeScale не может управлять движением BodyComponent из flame_forge2d по отдельности. Это полезно, только если нужно масштабировать время всей игры или Forge2DWorld.
```

```{flutter-app}
:sources: ../flame/examples
:page: time_scale
:show: widget code infobox
:width: 180
:height: 160
```

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';

class MyFlameGame extends FlameGame with HasTimeScale {
  void speedUp(){
    timeScale = 2.0;
  }

  void slowDown(){
    timeScale = 1.0;
  }
}
```


## Расширения

Flame включает набор полезных расширений, призванных помочь разработчику сокращёнными и преобразующими методами. Здесь представлен обзор этих расширений.

Все они могут быть импортированы из `package:flame/extensions.dart`


### Canvas

Методы:

- `scaleVector`: Аналогичен методу `canvas scale`, но принимает аргумент `Vector2`.
- `translateVector`: Аналогичен методу `canvas translate`, но принимает аргумент `Vector2`.
- `renderPoint`: рисует одну точку на холсте (в основном для отладки).
- `renderAt` и `renderRotated`: если вы выполняете рендеринг непосредственно на `Canvas`, эти функции позволяют легко манипулировать координатами для отрисовки в нужных местах. Они изменяют матрицу преобразования `Canvas`, но затем сбрасывают её.


### Color

Методы:

- `darken`: Затемняет оттенок цвета на величину от 0 до 1.
- `brighten`: Осветляет оттенок цвета на величину от 0 до 1.

Фабрики:

- `ColorExtension.fromRGBHexString`: Разбирает RGB-цвет из корректной hex-строки (например, #1C1C1C).
- `ColorExtension.fromARGBHexString`: Разбирает ARGB-цвет из корректной hex-строки (например, #FF1C1C1C).


### Image

Методы:

- `pixelsInUint8`: Получает пиксельные данные изображения как `Uint8List` в формате `ImageByteFormat.rawRgba`.
- `getBoundingRect`: Возвращает ограничивающий прямоугольник изображения как `Rect`.
- `size`: Размер изображения как `Vector2`.
- `darken`: Затемняет каждый пиксель изображения на величину от 0 до 1.
- `brighten`: Осветляет каждый пиксель изображения на величину от 0 до 1.


### Offset

Методы:

- `toVector2`: Создаёт `Vector2` из `Offset`.
- `toSize`: Создаёт `Size` из `Offset`.
- `toPoint`: Создаёт `Point` из `Offset`.
- `toRect`: Создаёт `Rect`, начинающийся в (0,0) с правым нижним углом, равным [Offset].


### Rect

Методы:

- `toOffset`: Создаёт `Offset` из `Rect`.
- `toVector2`: Создаёт `Vector2`, начинающийся в (0,0) и идущий к размеру `Rect`.
- `containsPoint`: Проверяет, содержит ли этот `Rect` точку `Vector2`.
- `intersectsSegment`: Проверяет, пересекает ли сегмент, образованный двумя `Vector2`, данный `Rect`.
- `intersectsLineSegment`: Проверяет, пересекает ли `LineSegment` данный `Rect`.
- `toVertices`: Превращает четыре угла `Rect` в список `Vector2`.
- `toFlameRectangle`: Преобразует этот `Rect` в `Rectangle` Flame.
- `toMathRectangle`: Преобразует этот `Rect` в `math.Rectangle`.
- `toGeometryRectangle`: Преобразует этот `Rect` в `Rectangle` из flame-geom.
- `transform`: Трансформирует `Rect` с помощью `Matrix4`.

Фабрики:

- `RectExtension.getBounds`: Строит `Rect`, представляющий границы списка `Vector2`.
- `RectExtension.fromCenter`: Строит `Rect` из центральной точки (используя `Vector2`).


### math.Rectangle

Методы:

- `toRect`: Преобразует `math.Rectangle` в ui `Rect`.


### Size

Методы:

- `toVector2`: Создаёт `Vector2` из `Size`.
- `toOffset`: Создаёт `Offset` из `Size`.
- `toPoint`: Создаёт `Point` из `Size`.
- `toRect`: Создаёт `Rect` начинающийся в (0,0) с размером `Size`.


### Vector2

Этот класс из пакета `vector_math`, и у нас есть несколько полезных методов расширения поверх того, что предлагает этот пакет.

Методы:

- `toOffset`: Создаёт `Offset` из `Vector2`.
- `toPoint`: Создаёт `Point` из `Vector2`.
- `toRect`: Создаёт `Rect`, начинающийся в (0,0) с размером `Vector2`.
- `toPositionedRect`: Создаёт `Rect`, начинающийся в [x, y] из `Vector2` и имеющий размер из аргумента `Vector2`.
- `lerp`: Линейно интерполирует `Vector2` в направлении другого `Vector2`.
- `rotate`: Поворачивает `Vector2` на угол в радианах, вращение происходит вокруг опционально заданного `Vector2`, иначе вокруг центра.
- `scaleTo`: Изменяет длину `Vector2` до указанной длины без изменения направления.
- `moveToTarget`: Плавно перемещает `Vector2` в целевом направлении на заданное расстояние.

Фабрики:

- `Vector2Extension.fromInts`: Создаёт `Vector2`, принимая целые числа.

Операторы:

- `&`: Комбинирует два `Vector2` для формирования `Rect`, слева должна быть начальная точка, справа — размер.
- `%`: Остаток от деления по x и y отдельно для двух `Vector2`.


### Matrix4

Этот класс из пакета `vector_math`. Мы создали несколько методов расширения поверх того, что уже предлагает `vector_math`.

Методы:

- `translate2`: Перемещает `Matrix4` на заданный `Vector2`.
- `transform2`: Создаёт новый `Vector2`, трансформируя данный `Vector2` с помощью `Matrix4`.
- `transformed2`: Трансформирует входной `Vector2` и записывает результат в выходной `Vector2`.

Геттеры:

- `m11`: Первая строка, первый столбец.
- `m12`: Первая строка, второй столбец.
- `m13`: Первая строка, третий столбец.
- `m14`: Первая строка, четвёртый столбец.
- `m21`: Вторая строка, первый столбец.
- `m22`: Вторая строка, второй столбец.
- `m23`: Вторая строка, третий столбец.
- `m24`: Вторая строка, четвёртый столбец.
- `m31`: Третья строка, первый столбец.
- `m32`: Третья строка, второй столбец.
- `m33`: Третья строка, третий столбец.
- `m34`: Третья строка, четвёртый столбец.
- `m41`: Четвёртая строка, первый столбец.
- `m42`: Четвёртая строка, второй столбец.
- `m43`: Четвёртая строка, третий столбец.
- `m44`: Четвёртая строка, четвёртый столбец.

Фабрики:

- `Matrix4Extension.scale`: Создаёт масштабированную `Matrix4`. Можно передать `Vector4` или `Vector2` в качестве первого аргумента, либо отдельные значения x, y, z типа double.
