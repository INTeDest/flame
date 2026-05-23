# Эффекты

В разработке игр постоянно требуется плавно анимировать свойства с течением времени: перемещать персонажа, затухать элемент, масштабировать бонус. Писать ручной код интерполяции в каждом методе `update` — утомительно и чревато ошибками. Эффекты предоставляют декларативный способ описывать такие изменения, привязанные ко времени: вы добавляете эффект к компоненту, и он автоматически управляет анимацией, а по завершении удаляется.

Эффект — это специальный компонент, который может быть прикреплён к другому компоненту для изменения его свойств или внешнего вида.

Например, представьте, что вы делаете игру с собираемыми предметами-усилителями. Вы хотите, чтобы они появлялись случайным образом по карте, а через некоторое время исчезали. Конечно, можно создать спрайтовый компонент для усилителя и разместить его на карте, но можно сделать ещё лучше!

Добавим `ScaleEffect`, чтобы предмет увеличивался от 0 до 100% при первом появлении. Добавим ещё один бесконечно повторяющийся знакопеременный `MoveEffect`, чтобы предмет слегка двигался вверх-вниз. Затем добавим `OpacityEffect`, который заставит предмет «мигнуть» 3 раза; у этого эффекта будет встроенная задержка в 30 секунд (или сколько нужно, чтобы усилитель оставался на месте). Наконец, добавим `RemoveEffect`, который автоматически удалит предмет из дерева игры через заданное время (вероятно, сразу после окончания `OpacityEffect`).

Как видите, с помощью нескольких простых эффектов мы превратили простой безжизненный спрайт в гораздо более интересный предмет. И, что важнее, это не привело к усложнению кода: добавленные эффекты работают автоматически и самоудаляются из дерева игры по завершении.


## Обзор

Задача `Effect` — изменять какое-либо свойство компонента с течением времени. Для этого `Effect` должен знать начальное значение свойства, конечное значение и то, как оно должно изменяться во времени. Начальное значение обычно определяется эффектом автоматически, конечное значение явно задаётся пользователем, а изменение во времени управляется [контроллерами эффектов](effect_controllers.md).


### Effect

Базовый класс `Effect` не может использоваться сам по себе (он абстрактный), но предоставляет общую функциональность, наследуемую всеми остальными эффектами. Сюда входит:

- Возможность приостановить/возобновить эффект с помощью `effect.pause()` и `effect.resume()`. Проверить, приостановлен ли эффект, можно через `effect.isPaused`.

- Свойство `removeOnFinish` (по умолчанию true) приводит к тому, что компонент эффекта удаляется из дерева игры и очищается сборщиком мусора после завершения эффекта. Установите его в false, если планируете повторно использовать эффект после его завершения.

- Опциональный пользовательский колбэк `onComplete`, который вызывается сразу после завершения выполнения эффекта, но до его удаления из игры.

- Future `completed`, который завершается, когда эффект заканчивается.

- Метод `reset()` возвращает эффект в исходное состояние, позволяя запустить его снова.

Flame предоставляет множество встроенных эффектов, и вы также можете [создавать свои собственные](#создание-новых-эффектов). Включены следующие эффекты:

- [`MoveByEffect`](move_effects.md#movebyeffect)
- [`MoveToEffect`](move_effects.md#movetoeffect)
- [`MoveAlongPathEffect`](move_effects.md#movealongpatheffect)
- [`RotateAroundEffect`](rotate_effects.md#rotatearoundeffect)
- [`RotateEffect.by`](rotate_effects.md#rotateeffectby)
- [`RotateEffect.to`](rotate_effects.md#rotateeffectto)
- [`ScaleEffect.by`](scale_effects.md#scaleeffectby)
- [`ScaleEffect.to`](scale_effects.md#scaleeffectto)
- [`SizeEffect.by`](size_effects.md#sizeeffectby)
- [`SizeEffect.to`](size_effects.md#sizeeffectto)
- [`AnchorByEffect`](anchor_effects.md#anchorbyeffect)
- [`AnchorToEffect`](anchor_effects.md#anchortoeffect)
- [`OpacityToEffect`](color_effects.md#opacitytoeffect)
- [`OpacityByEffect`](color_effects.md#opacitybyeffect)
- [`ColorEffect`](color_effects.md#coloreffect)
- [`SequenceEffect`](sequence_effect.md)
- [`CombinedEffect`](combined_effect.md)
- [`RemoveEffect`](remove_effect.md)
- [`FunctionEffect`](function_effect.md)


## Создание новых эффектов

Хотя Flame предоставляет широкий набор встроенных эффектов, со временем их может оказаться недостаточно. К счастью, создавать новые эффекты очень просто.

Каждый эффект наследуется от базового класса `Effect`, возможно, через один из более специализированных абстрактных подклассов, таких как `ComponentEffect<T>` или `Transform2DEffect`.

Конструктор класса `Effect` требует экземпляр `EffectController` в качестве аргумента. В большинстве случаев вы захотите передавать этот контроллер из своего конструктора. К счастью, контроллер эффекта инкапсулирует большую часть сложности реализации эффекта, поэтому вам не нужно заботиться о воссоздании этой функциональности.

Наконец, вам потребуется реализовать единственный метод `apply(double progress)`, который будет вызываться на каждом тике обновления, пока эффект активен. В этом методе вы должны вносить изменения в цель вашего эффекта.

Кроме того, вы можете реализовать колбэки `onStart()` и `onFinish()`, если есть действия, которые необходимо выполнить при начале или окончании эффекта.

При реализации метода `apply()` рекомендуется использовать только относительные обновления. То есть изменять целевое свойство, увеличивая/уменьшая его текущее значение, а не устанавливая свойство в фиксированное значение напрямую. Таким образом, несколько эффектов смогут воздействовать на один и тот же компонент, не мешая друг другу.


## Эффекты против декораторов

Хотя эффекты и декораторы иногда могут достигать схожих визуальных результатов (например, изменение прозрачности или цвета), у них разные характеристики производительности и визуальные особенности:

- **Эффекты** быстры и, как правило, изменяют свойство одного компонента. При применении к группе они воздействуют на каждого потомка индивидуально.
- **Декораторы** более мощные, но более медленные. Они используют `saveLayer`, чтобы сплющить всё поддерево компонентов в один слой перед применением эффекта. Это необходимо для корректной отрисовки составных объектов с прозрачностью или сложными фильтрами.

Более подробное сравнение см. в [документации по декораторам](../rendering/decorators.md).


## См. также

- [Примеры различных эффектов](https://examples.flame-engine.org/).

```{toctree}
:hidden:

Контроллеры эффектов        <effect_controllers.md>
Эффекты перемещения         <move_effects.md>
Эффекты вращения            <rotate_effects.md>
Эффекты масштабирования     <scale_effects.md>
Эффекты размера             <size_effects.md>
Эффекты якоря               <anchor_effects.md>
Цветовые эффекты            <color_effects.md>
Последовательный эффект     <sequence_effect.md>
Комбинированный эффект      <combined_effect.md>
Эффект удаления             <remove_effect.md>
Функциональный эффект       <function_effect.md>
```# Effects

In game development, smoothly animating properties over time (moving a character, fading an element,
scaling a power-up) is a constant need. Writing manual interpolation code in every `update` method
is repetitive and error-prone. Effects provide a declarative way to describe these time-based
changes: you attach an effect to a component, and it automatically handles the animation, then
removes itself when finished.

An effect is a special component that can attach to another component in order to modify its
properties or appearance.

For example, suppose you are making a game with collectible power-up items. You want these power-ups
to generate randomly around the map and then de-spawn after some time. Obviously, you could make a
sprite component for the power-up and then place that component on the map, but we could do even
better!

Let's add a `ScaleEffect` to grow the item from 0 to 100% when the power-up first appears. Add
another infinitely repeating alternating `MoveEffect` in order to make the item move slightly up
and down. Then add an `OpacityEffect` that will "blink" the item 3 times, this effect will have a
built-in delay of 30 seconds, or however long you want your power-up to stay in place. Lastly, add
a `RemoveEffect` that will automatically remove the item from the game tree after the specified
time (you probably want to time it right after the end of the `OpacityEffect`).

As you can see, with a few simple effects we have turned a simple lifeless sprite into a much more
interesting item. And what's more important, it didn't result in an increased code complexity: the
effects, once added, will work automatically, and then self-remove from the game tree when
finished.


## Overview

The function of an `Effect` is to effect a change over time in some component's property. In order
to achieve that, the `Effect` must know the initial value of the property, the final value, and how
it should progress over time. The initial value is usually determined by an effect automatically,
the final value is provided by the user explicitly, and progression over time is handled by
[EffectControllers](effect_controllers.md).


### Effect

The base `Effect` class is not usable on its own (it is abstract), but it provides some common
functionality inherited by all other effects. This includes:

- The ability to pause/resume the effect using `effect.pause()` and `effect.resume()`. You can
  check whether the effect is currently paused using `effect.isPaused`.

- Property `removeOnFinish` (which is true by default) will cause the effect component to be
  removed from the game tree and garbage-collected once the effect completes. Set this to false
  if you plan to reuse the effect after it is finished.

- Optional user-provided `onComplete`, which will be invoked when the effect has just
  completed its execution but before it is removed from the game.

- A `completed` future that completes when the effect finishes.

- The `reset()` method reverts the effect to its original state, allowing it to run once again.

There are multiple pre-built effects provided by Flame, and you can also
[create your own](#creating-new-effects). The following effects are included:

- [`MoveByEffect`](move_effects.md#movebyeffect)
- [`MoveToEffect`](move_effects.md#movetoeffect)
- [`MoveAlongPathEffect`](move_effects.md#movealongpatheffect)
- [`RotateAroundEffect`](rotate_effects.md#rotatearoundeffect)
- [`RotateEffect.by`](rotate_effects.md#rotateeffectby)
- [`RotateEffect.to`](rotate_effects.md#rotateeffectto)
- [`ScaleEffect.by`](scale_effects.md#scaleeffectby)
- [`ScaleEffect.to`](scale_effects.md#scaleeffectto)
- [`SizeEffect.by`](size_effects.md#sizeeffectby)
- [`SizeEffect.to`](size_effects.md#sizeeffectto)
- [`AnchorByEffect`](anchor_effects.md#anchorbyeffect)
- [`AnchorToEffect`](anchor_effects.md#anchortoeffect)
- [`OpacityToEffect`](color_effects.md#opacitytoeffect)
- [`OpacityByEffect`](color_effects.md#opacitybyeffect)
- [`ColorEffect`](color_effects.md#coloreffect)
- [`SequenceEffect`](sequence_effect.md)
- [`CombinedEffect`](combined_effect.md)
- [`RemoveEffect`](remove_effect.md)
- [`FunctionEffect`](function_effect.md)


## Creating new effects

Although Flame provides a wide array of built-in effects, eventually you may find them to be
insufficient. Luckily, creating new effects is very simple.

Each effect extends the base `Effect` class, possibly via one of the more specialized abstract
subclasses such as `ComponentEffect<T>` or `Transform2DEffect`.

The `Effect` class' constructor requires an `EffectController` instance as an argument. In most
cases you may want to pass that controller from your own constructor. Luckily, the effect controller
encapsulates much of the complexity of an effect's implementation, so you don't need to worry about
re-creating that functionality.

Lastly, you will need to implement a single method `apply(double progress)` that will be called at
each update tick while the effect is active. In this method you are supposed to make changes to the
target of your effect.

In addition, you may want to implement callbacks `onStart()` and `onFinish()` if there are any
actions that must be taken when the effect starts or ends.

When implementing the `apply()` method we recommend to use relative updates only. That is, change
the target property by incrementing/decrementing its current value, rather than directly setting
that property to a fixed value. This way multiple effects would be able to act on the same component
without interfering with each other.


## Effects vs Decorators

While effects and decorators can sometimes achieve similar visual results (like changing opacity
or color), they have different performance and visual characteristics:

- **Effects** are fast and generally change a property on a single component. When applied to
  a group, they affect each child individually.
- **Decorators** are more powerful but slower. They use `saveLayer` to flatten a whole
  component subtree into a single layer before applying an effect. This is essential for
  correctly rendering composite objects with transparency or complex filters.

See the [Decorators documentation](../rendering/decorators.md) for a more detailed comparison.


## See also

- [Examples of various effects](https://examples.flame-engine.org/).

```{toctree}
:hidden:

Effect Controllers        <effect_controllers.md>
Move Effects              <move_effects.md>
Rotate Effects            <rotate_effects.md>
Scale Effects             <scale_effects.md>
Size Effects              <size_effects.md>
Anchor Effects            <anchor_effects.md>
Color Effects             <color_effects.md>
Sequence Effect           <sequence_effect.md>
Combined Effect           <combined_effect.md>
Remove Effect             <remove_effect.md>
Function Effect           <function_effect.md>
```
