[Оригинальный пост](https://plugfox.dev/change-notifier-selector/)

Бывали ли у вас ситуации, когда было необходимо перестраивать интерфейс исходя из изменений только одной переменной в вашем [ChangeNotifier](https://api.flutter.dev/flutter/foundation/ChangeNotifier-class.html)?  


Возможно вам было неудобно использовать [AnimatedBuilder](https://api.flutter.dev/flutter/widgets/AnimatedBuilder-class.html) с [ChangeNotifier](https://api.flutter.dev/flutter/foundation/ChangeNotifier-class.html)вместо [ValueListenableBuilder](https://api.flutter.dev/flutter/widgets/ValueListenableBuilder-class.html)?

Конечно, у меня тоже бывало подобное, поэтому я  сделал простое, удобное, и компактное расширение с возможностью фильтрации, конвертирующее ваш ChangeNotifier в [ValueListenable](https://api.flutter.dev/flutter/foundation/ValueListenable-class.html) .

Логика работы расширения выглядит примерно так:

`ChangeNotifier ==[select & filter]==> ValueListenable<Value>`

Как пример, снизу вы можете преобразовать свою модель `AppModel` и перестраивать лишь определённые части приложения когда параметр локализации будет изменён и только в случае, когда код языка будет изменён.

```dart
ValueListenableBuilder<Locale>(
  valueListenable: appModel.select<Locale>(
    (cn) => cn.locale,
    (prev, next) => prev.languageCode != next.languageCode
  ),
  builder: (context, locale, child) => Text(locale),
)
```

Полный исходный код моего расширения:

```dart
import 'package:flutter/foundation.dart'

show Listenable, ValueListenable, VoidCallback, ChangeNotifier;

/// Selector from [Listenable]

typedef ChangeNotifierSelector<Controller extends Listenable, Value> = Value

Function(

Controller controller,

);

/// Filter for [Listenable]

typedef ChangeNotifierFilter<Value> = bool Function(Value prev, Value next);

/// Selects a specific value from the [Listenable]

/// for subsequent use in the ValueListenableBuilder

///

/// Listenable -- selector & filter --> ValueListenable<Value>

///

/// For example:

/// ```dart

/// ValueListenableBuilder<Locale>(

/// valueListenable: appModel.select<Locale>(

/// (cn) => cn.locale,

/// (prev, next) => prev.languageCode != next.languageCode

/// ),

/// builder: (context, locale, child) => Text(locale.languageCode),

/// )

/// ```

extension ChangeNotifierSelectorExtension<Controller extends Listenable>

on Controller {

/// Transform [Listenable] in to [ValueListenable]

ValueListenable<Value> select<Value>(

ChangeNotifierSelector<Controller, Value> selector, [

ChangeNotifierFilter<Value>? test,

]) =>

_ValueListenableView<Controller, Value>(this, selector, test);

}

class _ValueListenableView<Controller extends Listenable, Value>

with ChangeNotifier

implements ValueListenable<Value> {

_ValueListenableView(

Controller controller,

ChangeNotifierSelector<Controller, Value> selector,

ChangeNotifierFilter<Value>? test,

) : _controller = controller,

_selector = selector,

_test = test;

final Controller _controller;

final ChangeNotifierSelector<Controller, Value> _selector;

final ChangeNotifierFilter<Value>? _test;

@override

Value get value => hasListeners ? _$value : _selector(_controller);

late Value _$value;

void _update() {

final newValue = _selector(_controller);

if (identical(_$value, newValue)) return;

if (!(_test?.call(_$value, newValue) ?? true)) return;

_$value = newValue;

notifyListeners();

}

@override

void addListener(VoidCallback listener) {

if (!hasListeners) {

_$value = _selector(_controller);

_controller.addListener(_update);

}

super.addListener(listener);

}

@override

void removeListener(VoidCallback listener) {

super.removeListener(listener);

if (!hasListeners) _controller.removeListener(_update);

}

@override

void dispose() {

_controller.removeListener(_update);

super.dispose();

}

}
```


#PlugFox #extension