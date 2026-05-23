# Эффект удаления

Это простой эффект, который можно прикрепить к компоненту, чтобы он был удалён из дерева игры по истечении указанной задержки:

```{flutter-app}
:sources: ../flame/examples
:page: remove_effect
:show: widget code infobox
:width: 180
:height: 160
```


```dart
component.add(RemoveEffect(delay: 3.0));
```
