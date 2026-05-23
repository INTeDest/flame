# Оверлеи

Поскольку игру Flame можно обернуть в виджет, её довольно легко использовать вместе с другими виджетами Flutter в вашем дереве. Однако если вы хотите легко отображать виджеты поверх вашей игры Flame, например сообщения, экраны меню и тому подобное, можно воспользоваться API Widgets Overlay, чтобы сделать это ещё проще.

`Game.overlays` позволяет отображать любой Flutter-виджет поверх экземпляра игры. Это значительно упрощает создание, например, меню паузы или экрана инвентаря.

Эта возможность используется через методы `game.overlays.add` и `game.overlays.remove`, которые помечают оверлей для отображения или скрытия соответственно с помощью строкового аргумента-идентификатора. После этого вы можете сопоставить каждый оверлей с соответствующим виджетом в объявлении `GameWidget`, предоставив `overlayBuilderMap`.

```dart
  // Внутри вашей игры:
  final pauseOverlayIdentifier = 'PauseMenu';
  final secondaryOverlayIdentifier = 'SecondaryMenu';

  // Помечает 'SecondaryMenu' для отображения.
  overlays.add(secondaryOverlayIdentifier, priority: 1);
  // Помечает 'PauseMenu' для отображения. Priority = 0 по умолчанию,
  // что означает, что 'PauseMenu' будет отображаться под 'SecondaryMenu'
  overlays.add(pauseOverlayIdentifier);
  // Помечает 'PauseMenu' для скрытия.
  overlays.remove(pauseOverlayIdentifier);
  // Переключает оверлей 'PauseMenu'.
  overlays.toggle(pauseOverlayIdentifier);
  // Проверяет, отображается ли 'PauseMenu'
  final hasPauseMenu = overlays.isActive(pauseOverlayIdentifier);
  // Устанавливает активное состояние 'SecondaryMenu' в зависимости от условия
  overlays.setActive(secondaryOverlayIdentifier, active: !hasPauseMenu);
```

```dart
// В объявлении виджета
final game = MyGame();

Widget build(BuildContext context) {
  return GameWidget(
    game: game,
    overlayBuilderMap: {
      'PauseMenu': (BuildContext context, MyGame game) {
        return Text('Меню паузы');
      },
      'SecondaryMenu': (BuildContext context, MyGame game) {
        return Text('Вторичное меню');
      },
    },
  );
}
```

Порядок рендеринга оверлеев определяется порядком ключей в `overlayBuilderMap`.

Смотрите [пример использования функции Overlays](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/system/overlays_example.dart).
