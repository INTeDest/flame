# Структура каталогов ассетов

Игры сильно зависят от внешних ресурсов, таких как изображения для спрайтов, аудиофайлы для звуковых эффектов и тайловые карты для уровней. Единообразная организация этих файлов гарантирует, что встроенные загрузчики Flame (и собственная [система ассетов Flutter](https://docs.flutter.dev/ui/assets/assets-and-images)) смогут найти их без дополнительной настройки.

Flame предлагает структуру для вашего проекта, которая включает стандартную директорию Flutter `assets`, а также несколько вложенных папок: `audio`, `images` и `tiles`.

При использовании следующего примера кода:

```dart
class MyGame extends FlameGame {
  @override
  Future<void> onLoad() async {
    await FlameAudio.play('explosion.mp3');

    // Загрузить несколько изображений
    await Flame.images.load('player.png');
    await Flame.images.load('enemy.png');
    
    // Или загрузить все изображения из папки images
    await Flame.images.loadAllImages();

    final map1 = await TiledComponent.load('level.tmx', tileSize);
  }
}
```

Следующая файловая структура — это то место, где Flame ожидает найти файлы:

```text
.
└── assets
    ├── audio
    │   └── explosion.mp3
    ├── images
    │   ├── enemy.png
    │   ├── player.png
    │   └── spritesheet.png
    └── tiles
        ├── level.tmx
        └── map.json
```

При желании вы можете разделить папку `audio` на две подпапки: одну для `music` и одну для `sfx`.

Не забудьте добавить эти файлы в ваш файл `pubspec.yaml`:

```yaml
flutter:
  assets:
    - assets/audio/explosion.mp3
    - assets/images/player.png
    - assets/images/enemy.png
    - assets/tiles/level.tmx
```

Если вы хотите изменить эту структуру, это возможно с помощью параметра `prefix` и создания собственных экземпляров `AssetsCache`, `Images` и `AudioCache` вместо использования глобальных, предоставляемых Flame.

Кроме того, `AssetsCache` и `Images` могут принимать пользовательский [`AssetBundle`](https://api.flutter.dev/flutter/services/AssetBundle-class.html). Это можно использовать, чтобы заставить Flame искать ассеты в другом месте, отличном от `rootBundle`, например, в файловой системе.
