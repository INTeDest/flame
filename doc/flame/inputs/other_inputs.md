# Другие вводы и помощники

Здесь описываются методы ввода помимо клавиатуры и мыши.

Другие документы по вводу:

- [Ввод жестов](gesture_input.md): для жестов мыши и касаний
- [Клавиатурный ввод](keyboard_input.md): для нажатий клавиш


## Джойстик

Flame предоставляет компонент для создания виртуального джойстика для приема ввода в вашей игре.
Чтобы воспользоваться этой функцией, нужно создать `JoystickComponent`, настроить его нужным образом и добавить в игру.

Для лучшего понимания рассмотрите следующий пример:

```dart
class MyGame extends FlameGame {

  @override
  Future<void> onLoad() async {
    super.onLoad();
    final image = await images.load('joystick.png');
    final sheet = SpriteSheet.fromColumnsAndRows(
      image: image,
      columns: 6,
      rows: 1,
    );
    final joystick = JoystickComponent(
      knob: SpriteComponent(
        sprite: sheet.getSpriteById(1),
        size: Vector2.all(100),
      ),
      background: SpriteComponent(
        sprite: sheet.getSpriteById(0),
        size: Vector2.all(150),
      ),
      margin: const EdgeInsets.only(left: 40, bottom: 40),
    );

    final player = Player(joystick);
    add(player);
    add(joystick);
  }
}

class Player extends SpriteComponent with HasGameReference {
  Player(this.joystick)
    : super(
        anchor: Anchor.center,
        size: Vector2.all(100.0),
      );

  /// Пикселей в секунду
  double maxSpeed = 300.0;

  final JoystickComponent joystick;

  @override
  Future<void> onLoad() async {
    sprite = await game.loadSprite('layers/player.png');
    position = game.size / 2;
  }

  @override
  void update(double dt) {
    if (joystick.direction != JoystickDirection.idle) {
      position.add(joystick.relativeDelta  * maxSpeed * dt);
      angle = joystick.delta.screenAngle();
    }
  }
}
```

В этом примере мы создали классы `MyGame` и `Player`.
`MyGame` создает джойстик, который передается в `Player` при его создании.
В классе `Player` мы реагируем на текущее состояние джойстика.

У джойстика есть несколько полей, которые меняются в зависимости от его состояния.

Для определения состояния джойстика следует использовать следующие поля:

- `intensity`: Процент [0.0, 1.0], на который ручка (knob) оттянута от центра к краю джойстика (или `knobRadius`, если он задан).
- `delta`: Абсолютная величина (в виде `Vector2`), на которую ручка смещена от центра.
- `relativeDelta`: Процентное отношение, представленное как `Vector2`, и направление, в котором ручка в данный момент оттянута от базового положения к краю джойстика.

Если вы хотите создать кнопки вместе с джойстиком, обратите внимание на [`HudButtonComponent`](#hudbuttoncomponent).

Полный код реализации джойстика можно найти в [примере джойстика](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/input/joystick_example.dart).
Вы также можете увидеть [JoystickComponent в действии](https://examples.flame-engine.org/#/Input_Joystick), чтобы посмотреть на живой пример работы функции ввода с джойстика в игре.

В качестве дополнительного задания изучите [продвинутый пример джойстика](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/input/joystick_advanced_example.dart).
Посмотрите, что еще могут делать продвинутые функции, в [живой демонстрации](https://examples.flame-engine.org/#/Input_Joystick_Advanced).


## HudButtonComponent

`HudButtonComponent` — это кнопка, которую можно определить с отступами от края `Viewport`, а не с помощью позиции. Она принимает два `PositionComponent`: `button` и `buttonDown`. Первый используется для состояния покоя кнопки, а второй отображается, когда кнопка нажата. Второй компонент необязателен, если вы не хотите менять внешний вид кнопки при нажатии или обрабатываете это через `button`.

Как следует из названия, эта кнопка по умолчанию является HUD-элементом, то есть она остается статичной на экране, даже если камера игры перемещается. Вы также можете использовать этот компонент как не-HUD, установив `hudButtonComponent.respectCamera = true;`.

Если вы хотите реагировать на нажатие и отпускание кнопки (что обычно и требуется), можно либо передать функции обратного вызова в аргументы `onPressed` и `onReleased`, либо расширить компонент и переопределить `onTapDown`, `onTapUp` и/или `onTapCancel`, реализовав свою логику там.


## SpriteButtonComponent

`SpriteButtonComponent` — это кнопка, определяемая двумя `Sprite`: один представляет состояние, когда кнопка нажата, а второй — когда отпущена.


## ButtonComponent

`ButtonComponent` — это кнопка, определяемая двумя `PositionComponent`: один для нажатого состояния, второй для отпущенного. Если вы хотите использовать только спрайты для кнопки, воспользуйтесь [](#spritebuttoncomponent), но этот компонент может быть полезен, если, например, вы хотите сделать кнопкой `SpriteAnimationComponent` или что-то еще, не являющееся чистым спрайтом.


## Геймпад

У Flame есть специальный плагин для поддержки внешних игровых контроллеров (геймпадов).
Подробнее в [репозитории Gamepads](https://github.com/flame-engine/gamepads).


## AdvancedButtonComponent

`AdvancedButtonComponent` имеет отдельные состояния для каждой из фаз указателя.
Внешний вид можно настроить для каждого состояния, и каждое состояние представлено `PositionComponent`.

Вот поля, которые можно использовать для настройки внешнего вида `AdvancedButtonComponent`:

- `defaultSkin`: Компонент, отображаемый на кнопке по умолчанию.
- `downSkin`: Компонент, отображаемый при щелчке или касании кнопки.
- `hoverSkin`: Компонент, отображаемый при наведении на кнопку (на десктопе и в вебе).
- `defaultLabel`: Компонент, показываемый поверх обложек (skins). Автоматически выравнивается по центру.
- `disabledSkin`: Компонент, отображаемый, когда кнопка отключена.
- `disabledLabel`: Компонент, показываемый поверх обложек, когда кнопка отключена.


## ToggleButtonComponent

[ToggleButtonComponent] — это [AdvancedButtonComponent], который может переключаться между выбранным и невыбранным состоянием.

Помимо уже существующих обложек, [ToggleButtonComponent] содержит следующие обложки:

- `defaultSelectedSkin`: Компонент, отображаемый, когда кнопка выбрана.
- `downAndSelectedSkin`: Компонент, отображаемый, когда выбираемая кнопка выбрана и нажата.
- `hoverAndSelectedSkin`: Отображение при наведении на выбираемую и выбранную кнопку (десктоп и веб).
- `disabledAndSelectedSkin`: Для состояния, когда кнопка выбрана и отключена.
- `defaultSelectedLabel`: Компонент, показываемый поверх обложек, когда кнопка выбрана.


## Примесь IgnoreEvents

Если вы не хотите, чтобы поддерево компонентов получало события, можно использовать примесь `IgnoreEvents`.
После добавления этой примеси можно отключить события для компонента и его потомков, установив `ignoreEvents = true` (значение по умолчанию при добавлении примеси), а затем снова включить, установив `false`, когда нужно снова получать события.

Это может быть сделано в целях оптимизации, поскольку в настоящее время все события проходят через всё дерево компонентов.# Other Inputs and Helpers

This includes documentation for input methods besides keyboard and mouse.

For other input documents, see also:

- [Gesture Input](gesture_input.md): for mouse and touch pointer gestures
- [Keyboard Input](keyboard_input.md): for keystrokes


## Joystick

Flame provides a component capable of creating a virtual joystick for taking input for your game.
To use this feature, you need to create a `JoystickComponent`, configure it the way you want, and
add it to your game.

Check out the following example to get a better understanding:

```dart
class MyGame extends FlameGame {

  @override
  Future<void> onLoad() async {
    super.onLoad();
    final image = await images.load('joystick.png');
    final sheet = SpriteSheet.fromColumnsAndRows(
      image: image,
      columns: 6,
      rows: 1,
    );
    final joystick = JoystickComponent(
      knob: SpriteComponent(
        sprite: sheet.getSpriteById(1),
        size: Vector2.all(100),
      ),
      background: SpriteComponent(
        sprite: sheet.getSpriteById(0),
        size: Vector2.all(150),
      ),
      margin: const EdgeInsets.only(left: 40, bottom: 40),
    );

    final player = Player(joystick);
    add(player);
    add(joystick);
  }
}

class Player extends SpriteComponent with HasGameReference {
  Player(this.joystick)
    : super(
        anchor: Anchor.center,
        size: Vector2.all(100.0),
      );

  /// Pixels/s
  double maxSpeed = 300.0;

  final JoystickComponent joystick;

  @override
  Future<void> onLoad() async {
    sprite = await game.loadSprite('layers/player.png');
    position = game.size / 2;
  }

  @override
  void update(double dt) {
    if (joystick.direction != JoystickDirection.idle) {
      position.add(joystick.relativeDelta  * maxSpeed * dt);
      angle = joystick.delta.screenAngle();
    }
  }
}
```

In this example, we created the classes `MyGame` and `Player`.
`MyGame` creates a joystick which is passed to the `Player` when the latter is created.
In the `Player` class we act upon the current state of the joystick.

The joystick has a few fields that change depending on what state it is in.

Following fields should be used to know the state of the joystick:

- `intensity`: The percentage [0.0, 1.0] that the knob is dragged from the epicenter to the edge of
  the joystick (or `knobRadius` if that is set).
- `delta`: The absolute amount (defined as a `Vector2`) that the knob is dragged from its epicenter.
- `relativeDelta`: The percentage, presented as a `Vector2`, and direction that the knob is currently
  pulled from its base position to a edge of the joystick.

If you want to create buttons to go with your joystick, check out
[`HudButtonComponent`](#hudbuttoncomponent).

For the complete code on implementing the joystick, check out the
[Joystick Example](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/input/joystick_example.dart).
You can also view the
[JoystickComponent in action](https://examples.flame-engine.org/#/Input_Joystick)
to see a live example of the joystick input function integrated into a game.

For an additional challenge, explore the
[Advanced Joystick Example](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/input/joystick_advanced_example.dart).
See what else the advanced features can do in the
[live demo](https://examples.flame-engine.org/#/Input_Joystick_Advanced).


## HudButtonComponent

A `HudButtonComponent` is a button that can be defined with margins to the edge of the `Viewport`
instead of with a position. It takes two `PositionComponent`s. `button` and `buttonDown`, the first
is used for when the button is idle and the second is shown when the button is being pressed. The
second one is optional if you don't want to change the look of the button when it is pressed, or if
you handle this through the `button` component.

As the name suggests this button is a hud by default, which means that it will be static on your
screen even if the camera for the game moves around. You can also use this component as a non-hud by
setting `hudButtonComponent.respectCamera = true;`.

If you want to act upon the button being pressed (which would be the common thing to do) and released,
you can either pass in callback functions as the `onPressed` and `onReleased` arguments, or you can
extend the component and override `onTapDown`, `onTapUp` and/or `onTapCancel` and implement your
logic there.


## SpriteButtonComponent

A `SpriteButtonComponent` is a button that is defined by two `Sprite`s, one that represents
when the button is pressed and one that represents when the button is released.


## ButtonComponent

A `ButtonComponent` is a button that is defined by two `PositionComponent`s, one that represents
when the button is pressed and one that represents when the button is released. If you only want
to use sprites for the button, use the [](#spritebuttoncomponent) instead, but this component can be
good to use if you for example want to have a `SpriteAnimationComponent` as a button, or anything
else which isn't a pure sprite.


## Gamepad

Flame has a dedicated plugin to support external game controllers (gamepads).
Find more information in the [Gamepads repository](https://github.com/flame-engine/gamepads).


## AdvancedButtonComponent

The `AdvancedButtonComponent` have separate states for each of the different pointer phases.
The skin can be customized for each state and each skin is represented by a `PositionComponent`.

These are the fields that can be used to customize the looks of the `AdvancedButtonComponent`:

- `defaultSkin`: Component that will be displayed by default on the button.
- `downSkin`: Component displayed when the button is clicked or tapped.
- `hoverSkin`: Component displayed when the button is hovered. (desktop and web).
- `defaultLabel`: Component shown on top of skins. Automatically aligned to center.
- `disabledSkin`: Component displayed when button is disabled.
- `disabledLabel`: Component shown on top of skins when button is disabled.


## ToggleButtonComponent

The [ToggleButtonComponent] is an [AdvancedButtonComponent] that can switch between selected
and not selected.

In addition to the already existing skins, the [ToggleButtonComponent] contains the following skins:

- `defaultSelectedSkin`: The component to display when the button is selected.
- `downAndSelectedSkin`: The component that is displayed when the selectable button is selected and
  pressed.
- `hoverAndSelectedSkin`: Hover on selectable and selected button (desktop and web).
- `disabledAndSelectedSkin`: For when the button is selected and in the disabled state.
- `defaultSelectedLabel`: Component shown on top of the skins when button is selected.


## IgnoreEvents mixin

If you don't want a component subtree to receive events, you can use the `IgnoreEvents` mixin.
Once you have added this mixin you can turn off events to reach a component and its descendants by
setting `ignoreEvents = true` (default when the mixin is added), and then set it to `false` when you
want to receive events again.

This can be done for optimization purposes, since all events currently go through the whole
component tree.
