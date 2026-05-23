# Рендеринг текста

Flame предлагает несколько специализированных классов для рендеринга текста.


## Текстовые компоненты

Самый простой способ отобразить текст во Flame — использовать один из предоставляемых компонентов:

- `TextComponent` для рендеринга одной строки текста
- `TextBoxComponent` для размещения многострочного текста с ограничением по ширине и высоте, включая возможность эффекта печати. Можно использовать `newLineNotifier`, чтобы получать уведомления о добавлении новой строки. Колбэк `onComplete` позволяет выполнить функцию, когда текст полностью напечатан.
- `ScrollTextBoxComponent` расширяет `TextBoxComponent`, добавляя вертикальную прокрутку, когда текст выходит за границы заданной области.

Все компоненты показаны в [этом примере](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/rendering/text_example.dart).


### TextComponent

`TextComponent` — простой компонент, отображающий одну строку текста.

Простое использование:

```dart
class MyGame extends FlameGame {
  @override
  void onLoad() {
    add(
      TextComponent(
        text: 'Hello, Flame',
        position: Vector2.all(16.0),
      ),
    );
  }
}
```

Чтобы настроить параметры шрифта, цвета и т.п., нужно предоставить (или изменить) `TextRenderer` с соответствующей информацией. Простейшая реализация — `TextPaint`, принимающая Flutter `TextStyle`:

```dart
final regular = TextPaint(
  style: TextStyle(
    fontSize: 48.0,
    color: BasicPalette.white.color,
  ),
);

class MyGame extends FlameGame {
  @override
  void onLoad() {
    add(
      TextComponent(
        text: 'Hello, Flame',
        textRenderer: regular,
        anchor: Anchor.topCenter,
        position: Vector2(size.width / 2, 32.0),
      ),
    );
  }
}
```

Все опции смотрите в [API TextComponent](https://pub.dev/documentation/flame/latest/components/TextComponent-class.html).


### TextBoxComponent

`TextBoxComponent` очень похож на `TextComponent`, но предназначен для рендеринга текста внутри ограничивающей рамки с автоматическим переносом строк в соответствии с размером.

Вы можете управлять тем, должна ли рамка расти по мере добавления текста или оставаться статичной, с помощью `growingBox` в `TextBoxConfig`. Статичная рамка может иметь фиксированный размер (установка свойства `size`) либо автоматически сжиматься под содержимое.

Свойство `align` позволяет управлять горизонтальным и вертикальным выравниванием текста. Например, `align: Anchor.center` центрирует текст внутри рамки.

Для настройки полей рамки используется `margins` в `TextBoxConfig`.

Чтобы симулировать эффект печати (показ символов по одному), задайте `boxConfig.timePerChar`. Для управления эффектом: `skip()` показывает весь текст сразу, а `resetAnimation` сбрасывает анимацию к началу. После вызова `skip` для повторного воспроизведения эффекта не забудьте заново установить `timePerChar`.

Пример использования:

```dart
class MyTextBox extends TextBoxComponent {
  MyTextBox(String text) : super(
    text: text,
    textRenderer: tiny,
    boxConfig: TextBoxConfig(timePerChar: 0.05),
  );

  final bgPaint = Paint()..color = Color(0xFFFF00FF);
  final borderPaint = Paint()..color = Color(0xFF000000)..style = PaintingStyle.stroke;

  @override
  void render(Canvas canvas) {
    Rect rect = Rect.fromLTWH(0, 0, width, height);
    canvas.drawRect(rect, bgPaint);
    canvas.drawRect(rect.deflate(boxConfig.margin), borderPaint);
    super.render(canvas);
  }
}
```

Подробнее см. в [API TextBoxComponent](https://pub.dev/documentation/flame/latest/components/TextBoxComponent-class.html).


### ScrollTextBoxComponent

`ScrollTextBoxComponent` — это продвинутая версия `TextBoxComponent`, добавляющая вертикальную прокрутку, когда текст превышает размеры рамки. Полезна для диалогов, информационных панелей и т.п. Свойство `align` здесь не поддерживается.

Пример:

```dart
class MyScrollableText extends ScrollTextBoxComponent {
  MyScrollableText(Vector2 frameSize, String text) : super(
    size: frameSize,
    text: text,
    textRenderer: regular, 
    boxConfig: TextBoxConfig(timePerChar: 0.05),
  );
}
```


### TextElementComponent

Для рендеринга произвольного `TextElement` (от простого `InlineTextElement` до форматированного `DocumentRoot`) используйте `TextElementComponent`.

Простой пример с `DocumentRoot`, содержащим блоки (аналог HTML-блоков) и форматированным текстом:

```dart
  final document = DocumentRoot([
    HeaderNode.simple('1984', level: 1),
    ParagraphNode.simple(
      'Anything could be true. The so-called laws of nature were nonsense.',
    ),
    // ...
  ]);
  final element = TextElementComponent.fromDocument(
    document: document,
    position: Vector2(100, 50),
    size: Vector2(400, 200),
  );
```

Размер можно задать либо через `size` (как у любого `PositionComponent`), либо через `DocumentStyle`.

Пример со стилем:

```dart
  final style = DocumentStyle(
    width: 400,
    height: 200,
    padding: const EdgeInsets.symmetric(vertical: 10, horizontal: 14),
    background: BackgroundStyle(
      color: const Color(0xFF4E322E),
      borderColor: const Color(0xFF000000),
      borderWidth: 2.0,
    ),
  );
  final document = DocumentRoot([ ... ]);
  final element = TextElementComponent.fromDocument(
    document: document,
    style: style,
    position: Vector2(100, 50),
  );
```

Более подробный пример форматированного текста: [rich text example](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/rendering/rich_text_example.dart).

О внутреннем устройстве текстового движка см. раздел «Text Elements, Text Nodes и Text Styles» ниже.


### Flame Markdown

Для упрощённого создания `DocumentRoot` из строк с разметкой (жирный, курсив и т.д.) Flame предоставляет пакет `flame_markdown`, связывающий библиотеку `markdown` с инфраструктурой рендеринга текста.

Используйте `FlameMarkdown` и метод `toDocument`:

```dart
import 'package:flame/text.dart';
import 'package:flame_markdown/flame_markdown.dart';

// ...
final component = await TextElementComponent.fromDocument(
  document: FlameMarkdown.toDocument(
    '# Header\n'
    '\n'
    'This is a **bold** text, and this is *italic*.\n'
    '\n'
    'This is a second paragraph.\n',
  ),
  style: ...,
  position: ...,
  size: ...,
);
```


## Инфраструктура

Если вы не используете систему компонентов Flame, хотите понять внутреннее устройство, настроить шрифты и стили или создать собственные рендереры, читайте этот раздел.

- `TextRenderer`: знает, «как» отображать текст; по сути содержит информацию о стиле.
- `TextElement`: отформатированный, сверстанный фрагмент текста, включающий строку («что») и стиль («как»).

Диаграмма классов:

```{mermaid}
%%{init: { 'theme': 'dark' } }%%
classDiagram
    %% renderers
    note for TextRenderer "Это только стиль (как).
    Он знает, как взять строку и создать TextElement.
    `render` — это просто хелпер для `format(text).render(...)`. То же для `getLineMetrics`."
    class TextRenderer {
        TextElement format(String text)
        LineMetrics getLineMetrics(String text)
        void render(Canvas canvas, String text, ...)
    }
    class TextPaint
    class SpriteFontRenderer
    class DebugTextRenderer
    
    %% elements
    class TextElement {
        LineMetrics metrics
        render(Canvas canvas, ...)
    }
    class TextPainterTextElement
        
    TextRenderer --> TextPaint
    TextRenderer --> SpriteFontRenderer
    TextRenderer --> DebugTextRenderer

    TextRenderer *-- TextElement
    TextPaint *-- TextPainterTextElement
    SpriteFontRenderer *-- SpriteFontTextElement

    note for TextElement "Это текст (что) и стиль (как);
    сверстан и готов к рендерингу."
    TextElement --> TextPainterTextElement
    TextElement --> SpriteFontTextElement
    TextElement --> Others
```


### TextRenderer

`TextRenderer` — абстрактный класс для рендеринга текста. Реализации должны содержать информацию о том, «как» отображать текст (шрифт, размер, цвет и т.д.) и уметь объединять её со строкой через метод `format`, возвращая `TextElement`.

Две конкретные реализации:

- `TextPaint`: наиболее часто используется, на базе Flutter `TextPainter`
- `SpriteFontRenderer`: использует `SpriteFont` (растровый шрифт на основе спрайт-листа)
- `DebugTextRenderer`: только для Golden-тестов

Можно создать свою реализацию.

Основная задача `TextRenderer` — форматировать строку в `TextElement`, который затем можно отрисовать:

```dart
final textElement = textRenderer.format("Flame is awesome")
textElement.render(...) 
```

Также есть хелпер для прямого рендеринга:

```dart
textRenderer.render(
  canvas,
  'Flame is awesome',
  Vector2(10, 10),
  anchor: Anchor.topCenter,
);
```


#### TextPaint

`TextPaint` — встроенная реализация, использующая `TextPainter` Flutter. Настраивается через `TextStyle` (размер, цвет, шрифт и т.д.).

Пример:

```dart
const TextPaint textPaint = TextPaint(
  style: TextStyle(
    fontSize: 48.0,
    fontFamily: 'Awesome Font',
  ),
);
```

Важно импортировать правильный `TextStyle` из `package:flutter/painting.dart`, при необходимости скрывая версию из `dart:ui`.

Основные свойства `TextStyle`:

- `fontFamily`: название шрифта (например, Arial, или пользовательский)
- `fontSize`: размер в пунктах (по умолчанию 24.0)
- `height`: высота строки как множитель размера шрифта
- `color`: цвет как `ui.Color`

Подробнее о цветах см. [Colors и Palette](palette.md).


#### SpriteFontRenderer

Позволяет использовать `SpriteFont` на основе спрайт-листа. Документация в процессе подготовки (TODO).


#### DebugTextRenderer

Предназначен для Golden-тестов: рендерит текст в виде сплошных прямоугольников (каждое слово — прямоугольник), чтобы избежать проблем с платформенно-зависимым сглаживанием шрифтов и проверять вёрстку, позиционирование и размеры.


## Inline Text Elements

`TextElement` — «скомпилированный», сверстанный фрагмент текста с определённым стилем, готовый к рендерингу.

`InlineTextElement` реализует интерфейс `TextElement` с двумя методами:

```dart
  void translate(double dx, double dy);
  void draw(Canvas canvas);
```

Они, как правило, не вызываются напрямую; вместо этого используется удобный метод `render`:

```dart
  void render(
    Canvas canvas,
    Vector2 position, {
    Anchor anchor = Anchor.topLeft,
  })
```

Интерфейс также предоставляет `LineMetrics` через геттер `metrics`, содержащий информацию о размерах (ширина, высота, верхний вынос и т.д.).


## Text Elements, Text Nodes и Text Styles

Хотя обычные рендереры работают напрямую с `InlineTextElement`, существует более обширная инфраструктура для форматированного (rich) текста.

Text Elements — это надмножество Inline Text Elements, представляющее произвольный блок в документе; это конкретные объекты, готовые к рендерингу.

Они отличаются от Text Nodes (структурированные части текста) и FlameTextStyle (описания того, как должен отображаться текст).

В общем случае пользователь создаёт `TextNode` для описания rich-текста, определяет `FlameTextStyle` и генерирует `TextElement`. Для рендеринга произвольного TextElement можно использовать `TextElementComponent`.

Примеры: [rich text example](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/rendering/rich_text_example.dart).


### Text Nodes и Document Root

`DocumentRoot` не наследуется от `TextNode`, но представляет группу `BlockNode`, формирующих «страницу» или «документ» из нескольких блоков/абзацев. Он может получать глобальный стиль.

Сначала создаётся узел — обычно `DocumentRoot`, содержащий блочные узлы верхнего уровня: заголовки, параграфы или колонки. Каждый блок может содержать другие блоки или Inline Text Nodes (простой текст или форматированные фрагменты).

Иерархия узлов используется и для применения стилей (каскадирование).

Диаграмма узлов:

```{mermaid}
%%{init: { 'theme': 'dark' } }%%
graph TD
    %% Config %%
    classDef default fill:#282828,stroke:#F6BE00;

    %% Nodes %%
    TextNode("
        <big><strong>TextNode</strong></big>
        Можно представить как узел HTML DOM;
        каждый подкласс — как определённый тег.
    ")
    BlockNode("
        <big><strong>BlockNode</strong></big>
        #quot;div#quot;
    ")
    InlineTextNode("
        <big><strong>InlineTextNode</strong></big>
        #quot;span#quot;
    ")
    ColumnNode("
        <big><strong>ColumnNode</strong></big>
        группа BlockNode, расположенных в столбец
    ")
    TextBlockNode("
        <big><strong>TextBlockNode</strong></big>
        #quot;div#quot; с InlineTextNode как прямым потомком
    ")
    HeaderNode("
        <big><strong>HeaderNode</strong></big>
        #quot;h1#quot; / #quot;h2#quot; / и т.д.
    ")
    ParagraphNode("
        <big><strong>ParagraphNode</strong></big>
        #quot;p#quot;
    ")
    GroupTextNode("
        <big><strong>GroupTextNode</strong></big>
        группирует другие TextNode в одной строке
    ")
    PlainTextNode("
        <big><strong>PlainTextNode</strong></big>
        простой текст без форматирования
    ")
    ItalicTextNode("
        <big><strong>ItalicTextNode</strong></big>
        #quot;i#quot; / #quot;em#quot;
    ")
    BoldTextNode("
        <big><strong>BoldTextNode</strong></big>
        #quot;b#quot; / #quot;strong#quot;
    ")
    TextNode ----> BlockNode
    TextNode --------> InlineTextNode
    BlockNode --> ColumnNode
    BlockNode --> TextBlockNode
    TextBlockNode --> HeaderNode
    TextBlockNode --> ParagraphNode
    InlineTextNode --> GroupTextNode
    InlineTextNode --> PlainTextNode
    InlineTextNode --> BoldTextNode
    InlineTextNode --> ItalicTextNode
```


### (Flame) Text Styles

Стили применяются к узлам для генерации элементов. Все наследуются от абстрактного `FlameTextStyle` (назван так, чтобы избежать путаницы с Flutter `TextStyle`).

Они образуют древовидную структуру, где корнем выступает `DocumentStyle`; каскадирование стилей аналогично CSS.

Иерархия стилей:

```{mermaid}
%%{init: { 'theme': 'dark' } }%%
classDiagram
    %% Nodes %%
    class FlameTextStyle {
        copyWith()
        merge()
    }

    note for FlameTextStyle "Корень всех стилей.
    Не путать с TextStyle из Flutter."

    class DocumentStyle {
        <<для всего Document Root>>
        size
        padding
        background [BackgroundStyle]
        специфичные стили [для блоков и inline]
    }

    class BlockStyle {
        <<для Block Nodes>>
        margin, padding
        background [BackgroundStyle]
        text [InlineTextStyle]
    }

    class BackgroundStyle {
        <<для Block или Document>>
        color
        border
    }

    class InlineTextStyle {
        <<для любых узлов>>
        font, color
    }

    FlameTextStyle <|-- DocumentStyle
    FlameTextStyle <|-- BlockStyle
    FlameTextStyle <|-- BackgroundStyle
    FlameTextStyle <|-- InlineTextStyle
```


### Text Elements

Наконец, элементы представляют комбинацию узла («что») и стиля («как») — готовый к рендерингу сверстанный фрагмент текста.

`InlineTextElement` можно также рассматривать как комбинацию `TextRenderer` (упрощённое «как») и строки (одна строка «что»), поскольку `InlineTextStyle` может быть преобразован в конкретный `TextRenderer` через метод `asTextRenderer`, который затем используется для вёрстки каждой строки в `InlineTextElement`.

При прямом использовании рендерера возвращается `TextPainterTextElement` или `SpriteFontTextElement`.

Рекомендации по выбору подхода:

- Самый простой способ: `TextPaint` (базовый рендерер). Для него есть готовый компонент `TextComponent`.
- Для растровых шрифтов: `SpriteFontRenderer`.
- Для многострочного текста с автоматическим переносом:
  - `TextBoxComponent` (использует любой рендерер, делает свою вёрстку).
  - Система узлов и стилей для предварительной вёрстки (нет готового FCS-компонента).
- Для форматированного (rich) текста: обязательно использовать Text Nodes и Styles.
