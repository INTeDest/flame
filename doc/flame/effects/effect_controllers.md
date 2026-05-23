# Контроллеры эффектов

`EffectController` — это объект, описывающий, как эффект должен изменяться с течением времени. Если представить начальное значение эффекта как прогресс 0%, а конечное — как 100%, то задача контроллера эффекта состоит в отображении «физического» времени, измеряемого в секундах, в «логическое», которое меняется от 0 до 1.

Фреймворк Flame предоставляет несколько контроллеров эффектов:

- [`EffectController`](#effectcontroller)
- [`LinearEffectController`](#lineareffectcontroller)
- [`ReverseLinearEffectController`](#reverselineareffectcontroller)
- [`CurvedEffectController`](#curvedeffectcontroller)
- [`ReverseCurvedEffectController`](#reversecurvedeffectcontroller)
- [`PauseEffectController`](#pauseeffectcontroller)
- [`RepeatedEffectController`](#repeatedeffectcontroller)
- [`InfiniteEffectController`](#infiniteeffectcontroller)
- [`SequenceEffectController`](#sequenceeffectcontroller)
- [`SpeedEffectController`](#speedeffectcontroller)
- [`DelayedEffectController`](#delayedeffectcontroller)
- [`NoiseEffectController`](#noiseeffectcontroller)
- [`RandomEffectController`](#randomeffectcontroller)
- [`SineEffectController`](#sineeffectcontroller)
- [`ZigzagEffectController`](#zigzageffectcontroller)


## `EffectController`

Базовый класс `EffectController` предоставляет фабричный конструктор, способный создавать множество типовых контроллеров. Синтаксис конструктора следующий:

```dart
EffectController({
    required double duration,
    Curve curve = Curves.linear,
    double? reverseDuration,
    Curve? reverseCurve,
    bool alternate = false,
    double atMaxDuration = 0.0,
    double atMinDuration = 0.0,
    int? repeatCount,
    bool infinite = false,
    double startDelay = 0.0,
    VoidCallback? onMax,
    VoidCallback? onMin,
});
```

- *`duration`*: длительность основной части эффекта, т.е. сколько времени потребуется, чтобы перейти от 0 до 100%. Этот параметр не может быть отрицательным, но может быть равен нулю. Если указан только этот параметр, эффект будет линейно расти в течение `duration` секунд.

- *`curve`*: если задана, создаёт нелинейный эффект, который растёт от 0 до 100% согласно предоставленной [кривой](https://api.flutter.dev/flutter/animation/Curves-class.html).

- *`reverseDuration`*: если указана, добавляет дополнительный шаг в контроллер: после того как эффект вырос от 0 до 100% за `duration` секунд, он пойдёт в обратном направлении от 100% до 0 за `reverseDuration` секунд. Кроме того, эффект завершится на уровне прогресса 0 (обычно эффект завершается при прогрессе 1).

- *`reverseCurve`*: кривая, используемая на «обратном» шаге эффекта. Если не задана, будет использована `curve.flipped`.

- *`alternate`*: установка в true эквивалентна указанию `reverseDuration`, равного `duration`. Если `reverseDuration` уже задана, этот флаг не имеет эффекта.

- *`atMaxDuration`*: если не ноль, вставляет паузу после достижения эффектом максимального прогресса и перед обратным этапом. В течение этого времени прогресс эффекта удерживается на 100%. Если обратного этапа нет, то это будет просто пауза перед тем, как эффект будет отмечен завершённым.

- *`atMinDuration`*: если не ноль, вставляет паузу после достижения эффектом наименьшего прогресса (0) в конце обратного этапа. В течение этого времени прогресс эффекта равен 0%. Если обратного этапа нет, то эта пауза всё равно будет вставлена после «паузы на максимуме», если она присутствует, или после прямого этапа в противном случае. Кроме того, эффект теперь завершится на уровне прогресса 0.

- *`repeatCount`*: если больше единицы, заставит эффект повторяться указанное число раз. Каждая итерация будет состоять из прямого этапа, паузы на максимуме, обратного этапа, затем паузы на минимуме (пропуская те, что не были заданы).

- *`infinite`*: если true, эффект будет повторяться бесконечно и никогда не достигнет завершения. Это эквивалентно установке `repeatCount` в бесконечность.

- *`startDelay`*: дополнительное время ожидания, вставляемое перед началом эффекта. Это время ожидания выполняется только один раз, даже если эффект повторяется. В течение этого времени свойство `.started` эффекта возвращает false. Обратный вызов `onStart()` эффекта будет выполнен в конце этого периода ожидания.

  Использование этого параметра — простейший способ создать цепочку эффектов, выполняющихся один за другим (или с перекрытием).

- *`onMax`*: функция обратного вызова, которая будет вызвана сразу после достижения максимального прогресса и перед опциональной паузой и обратным этапом.

- *`onMin`*: функция обратного вызова, которая будет вызвана сразу после достижения наименьшего прогресса в конце обратного этапа и перед опциональной паузой и прямым этапом.

Контроллер эффекта, возвращаемый этим фабричным конструктором, будет состоять из нескольких более простых контроллеров, описанных ниже. Если этот конструктор окажется слишком ограниченным для ваших нужд, вы всегда можете создать собственную комбинацию из тех же строительных блоков.

В дополнение к фабричному конструктору, класс `EffectController` определяет ряд свойств, общих для всех контроллеров эффектов. Эти свойства:

- `.started`: true, если эффект уже начался. Для большинства контроллеров это свойство всегда true. Единственное исключение — `DelayedEffectController`, который возвращает false, пока эффект находится в стадии ожидания.

- `.completed`: становится true, когда контроллер эффекта завершает выполнение.

- `.progress`: текущее значение контроллера, число с плавающей точкой от 0 до 1. Эта переменная — основное «выходное» значение контроллера эффекта.

- `.duration`: общая длительность эффекта или `null`, если длительность невозможно определить (например, если длительность случайна или бесконечна).


## `LinearEffectController`

Простейший контроллер, который линейно растёт от 0 до 1 за указанную `длительность`:

```dart
final controller = LinearEffectController(3);
```


## `ReverseLinearEffectController`

Похож на `LinearEffectController`, но движется в противоположном направлении и линейно убывает от 1 до 0 за указанную длительность:

```dart
final controller = ReverseLinearEffectController(1);
```


## `CurvedEffectController`

Этот контроллер нелинейно растёт от 0 до 1 за указанную `длительность`, следуя заданной `кривой`:

```dart
final controller = CurvedEffectController(0.5, Curves.easeOut);
```


## `ReverseCurvedEffectController`

Похож на `CurvedEffectController`, но убывает от 1 до 0, следуя заданной `кривой`:

```dart
final controller = ReverseCurvedEffectController(0.5, Curves.bounceInOut);
```


## `PauseEffectController`

Удерживает прогресс на постоянном значении в течение указанного времени. Обычно `progress` равен 0 или 1:

```dart
final controller = PauseEffectController(1.5, progress: 0);
```


## `RepeatedEffectController`

Составной контроллер. Он принимает другой контроллер в качестве дочернего и повторяет его несколько раз, сбрасывая перед началом каждого следующего цикла.

```dart
final controller = RepeatedEffectController(LinearEffectController(1), 10);
```

Дочерний контроллер не может быть бесконечным. Если дочерний контроллер случайный, он будет переинициализирован с новыми случайными значениями на каждой итерации.


## `InfiniteEffectController`

Похож на `RepeatedEffectController`, но повторяет дочерний контроллер бесконечно.

```dart
final controller = InfiniteEffectController(LinearEffectController(1));
```


## `SequenceEffectController`

Выполняет последовательность контроллеров один за другим. Список контроллеров не может быть пустым.

```dart
final controller = SequenceEffectController([
  LinearEffectController(1),
  PauseEffectController(0.2),
  ReverseLinearEffectController(1),
]);
```


## `SpeedEffectController`

Изменяет длительность дочернего контроллера так, чтобы эффект протекал с заданной скоростью. Исходная длительность дочернего EffectController не важна. Дочерний контроллер должен быть подклассом `DurationEffectController`.

`SpeedEffectController` может применяться только к эффектам, для которых понятие скорости чётко определено. Такие эффекты должны реализовывать интерфейс `MeasurableEffect`. Например, к ним относятся:

- [`MoveByEffect`](move_effects.md#movebyeffect)
- [`MoveToEffect`](move_effects.md#movetoeffect)
- [`MoveAlongPathEffect`](move_effects.md#movealongpatheffect)
- [`RotateEffect.by`](rotate_effects.md#rotateeffectby)
- [`RotateEffect.to`](rotate_effects.md#rotateeffectto)

Параметр `speed` задаётся в единицах в секунду, причём понятие «единицы» зависит от целевого эффекта. Например, для эффектов перемещения это пройденное расстояние; для эффектов вращения — радианы.

```dart
final speedController =
    SpeedEffectController(LinearEffectController(0), speed: 1);
final controller =
    EffectController(speed: 1); // то же, что и speedController
```


## `DelayedEffectController`

Контроллер, который выполняет свой дочерний контроллер после заданной `задержки`. Пока контроллер находится на стадии «задержки», эффект считается «не начавшимся», т.е. его свойство `.started` возвращает `false`.

```dart
final controller = DelayedEffectController(LinearEffectController(1), delay: 5);
```


## `NoiseEffectController`

Этот контроллер проявляет шумовое поведение, т.е. случайно колеблется около нуля. Такой контроллер можно использовать для реализации разнообразных эффектов дрожания.

```dart
final controller = NoiseEffectController(duration: 0.6, frequency: 10);
```


## `RandomEffectController`

Этот контроллер оборачивает другой контроллер и делает его длительность случайной. Фактическое значение длительности генерируется заново при каждом сбросе, что делает этот контроллер особенно полезным в повторяющихся контекстах, таких как [](#repeatedeffectcontroller) или [](#infiniteeffectcontroller).

```dart
final controller = RandomEffectController.uniform(
  LinearEffectController(0),  // длительность здесь не важна
  min: 0.5,
  max: 1.5,
);
```

Пользователь может выбирать, какой источник `Random` использовать, а также точное распределение генерируемых случайных длительностей. Предусмотрены два распределения: `.uniform` и `.exponential`, любое другое может быть реализовано пользователем.


## `SineEffectController`

Контроллер, представляющий один период синусоиды. Используйте его для создания естественно выглядящих гармонических колебаний. Два перпендикулярных эффекта перемещения, управляемых `SineEffectControllers` с разными периодами, создадут [кривую Лиссажу].

```dart
final controller = SineEffectController(period: 1);
```


## `ZigzagEffectController`

Простой знакопеременный контроллер. В течение одного `period` этот контроллер будет линейно изменяться от 0 до 1, затем до -1 и обратно к 0. Используйте его для колебательных эффектов, где начальное положение должно быть центром колебаний, а не крайним значением (как это обеспечивается стандартным знакопеременным `EffectController`).

```dart
final controller = ZigzagEffectController(period: 2);
```

[кривую Лиссажу]: https://en.wikipedia.org/wiki/Lissajous_curve
