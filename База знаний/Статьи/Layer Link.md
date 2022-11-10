[Оригинальный пост](https://plugfox.dev/layer-link/)


Давайте рассмотрим способ, как мы можем создавать и отображать виджеты, которые появляются поверх других виджетов и следуют за ними при перемещении. Это чрезвычайно полезный опыт для различных элементов, таких как всплывающие окна, раскрывающиеся меню, онбоардинги и отрисовка виджета поверх и за пределами родительского виджета. При этом не стоит забывать, что если родительский виджет находится в списке или перемещается, а экран прокручивается или перемещается каким-то другим образом — оверлей не останется на месте в исходных координатах, а будет следовать за родительским элементом.

После прочтения статьи вы сможете создавать вот такие виджеты.

![[image-1.png]]

Всплывающие уведомления, подсказки, иконки и другие виджеты:

![[image-2.png]]
Онбоардинг и оверлей:
![[image-3.png]]
Или даже больше, прикрепите «полосу здоровья» к своему  прыгучему персонажу::
![[image-4.png]]


Предположим, вам нужно прикрепить оверлей к определенному виджету.

Проблема отрисовки элементов оверлея в определенных координатах не выглядит слишком сложной, хотя таковой является. Общий макет виджета с наложением можно прокручивать, преобразовывать, перемещать или видоизменять иным образом.

Другими словами, вы должны получить оверлей, следующий за виджетом, соединив два отдельных слоя (родительский виджет и его оверлей-последователь).

И тут нам на помощь приходят два виджета во Flutter. Они помогают связать родительский виджет с оверлеем: [CompositedTransformTarget](https://api.flutter.dev/flutter/widgets/CompositedTransformTarget-class.html) и [CompositedTransformFollower](https://api.flutter.dev/flutter/widgets/CompositedTransformFollower-class.html).

Если вы хотите увидеть весь код или увидеть его в режиме мастерской:

-   [Layer link workshop](https://dartpad.dev/workshops.html?webserver=https://raw.githubusercontent.com/PlugFox/layer_link/master/public)
-   [Full example](https://dartpad.dev/dbd122459cc911589cba450e1613355c)

В этой статье я продемонстрирую их использование, создав оболочку виджета, чтобы помочь в самопроверке вашего кода и автоматизировать некоторые действия, когда приложение работает в режиме отладки.

Прежде всего, мы должны создать новое стартовое приложение.

```dart
import 'dart:async';
import 'dart:developer' as dev;

import 'package:flutter/material.dart';

void main() => runZonedGuarded<void>(
      () => runApp(const App()),
      (error, stackTrace) => dev.log(
        'A error has occurred',
        stackTrace: stackTrace,
        error: error,
        name: 'main',
        level: 1000,
      ),
    );

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        title: 'Promter',
        home: Scaffold(
          appBar: AppBar(
            title: const Text('Promter'),
          ),
          body: const SafeArea(
            child: Placeholder(),
          ),
        ),
      );
}
```
После этого добавим нашу форму ввода.

Итак, усложним макет с помощью трансформаций.

```dart
import 'dart:math' as math; // +
import 'package:flutter/services.dart'; // +

/*

  ...

  home: Scaffold(
    appBar: AppBar(
      title: const Text('Promter'),
    ),
    body: const SafeArea(
      child: SignUpForm(), // <== replace placeholder with new form
    ),
  ),

  ...

*/

class SignUpForm extends StatefulWidget {
  const SignUpForm({super.key});

  @override
  State<SignUpForm> createState() => _SignUpFormState();
}

class _SignUpFormState extends State<SignUpForm> {
  final TextEditingController _firstNameController = TextEditingController();
  final TextEditingController _secondNameController = TextEditingController();
  final TextEditingController _ageController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();

  @protected
  void clearAll() {
    _firstNameController.clear();
    _secondNameController.clear();
    _ageController.clear();
    _emailController.clear();
  }

  @override
  void dispose() {
    _firstNameController.dispose();
    _secondNameController.dispose();
    _ageController.dispose();
    _emailController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Column(
        mainAxisSize: MainAxisSize.max,
        crossAxisAlignment: CrossAxisAlignment.center,
        children: <Widget>[
          const Expanded(
            child: Center(
              child: FlutterLogo(size: 120),
            ),
          ),
          Transform.rotate(
            angle: math.pi / 32,
            child: SizedBox(
              height: 108,
              child: ListView(
                padding: const EdgeInsets.all(24),
                scrollDirection: Axis.horizontal,
                physics: const BouncingScrollPhysics(
                  parent: AlwaysScrollableScrollPhysics(),
                ),
                children: <Widget>[
                  SizedBox(
                    width: 180,
                    child: TextField(
                      decoration: const InputDecoration(
                        labelText: 'First name',
                      ),
                      maxLines: 1,
                      keyboardType: TextInputType.text,
                      controller: _firstNameController,
                    ),
                  ),
                  const SizedBox(width: 48),
                  SizedBox(
                    width: 180,
                    child: TextField(
                      decoration: const InputDecoration(
                        labelText: 'Second name',
                      ),
                      maxLines: 1,
                      keyboardType: TextInputType.text,
                      controller: _secondNameController,
                    ),
                  ),
                  const SizedBox(width: 48),
                  SizedBox(
                    width: 120,
                    child: TextField(
                      decoration: const InputDecoration(
                        labelText: 'Age',
                        counterText: '',
                      ),
                      maxLength: 3,
                      keyboardType: TextInputType.number,
                      inputFormatters: [
                        FilteringTextInputFormatter.digitsOnly,
                      ],
                      maxLines: 1,
                      controller: _ageController,
                    ),
                  ),
                  const SizedBox(width: 48),
                  SizedBox(
                    width: 240,
                    child: TextField(
                      decoration: const InputDecoration(
                        labelText: 'e-Mail',
                      ),
                      maxLines: 1,
                      keyboardType: TextInputType.emailAddress,
                      controller: _emailController,
                    ),
                  ),
                ],
              ),
            ),
          ),
          const Spacer(),
        ],
      );
}
```

Теперь, когда у нас есть несколько полей для ввода текста, заполнять их каждый раз неинтересно. Создание нового виджета, который поможет нам заполнить их данными-заглушками, — хорошая идея.

Наш новый вспомогательный виджет-оболочка содержит следующий API и кнопку наложения, которая будет отображать наложение с подсказками.

Добавьте виджет с отслеживанием состояния и три миксина для состояний.

-   `_PrompterApiMixin` - API нашего состояния с методами `show` и `hide`.
-   `_PrompterBuilderMixin` - миксин, содержащий метод билд.
-   `_PrompterOverlayMixin` - миксин для логики, отвечающий за наложение и трекинг с синхронизацией положения между двумя слоями.

Создайте виджет, его стейт и реализуйте первые два миксина.

```dart
import 'package:flutter/foundation.dart' show kDebugMode; // +
import 'package:meta/meta.dart'; // +

// ...

class Prompter extends StatefulWidget {
  const Prompter({
    required this.child,
    required this.actions,
    super.key,
  });

  final Widget child;
  final Map<String, VoidCallback> actions;

  @override
  State<Prompter> createState() => _PrompterState();
}

class _PrompterState = State<Prompter>
    with _PrompterApiMixin, _PrompterBuilderMixin, _PrompterOverlayMixin;

mixin _PrompterApiMixin on State<Prompter> {
  @mustCallSuper
  @visibleForTesting
  @visibleForOverriding
  void show() {} // for further implementation

  @mustCallSuper
  @visibleForTesting
  @visibleForOverriding
  void hide() {} // for further implementation
}

mixin _PrompterBuilderMixin on _PrompterApiMixin {
  @override
  Widget build(BuildContext context) {
    if (!kDebugMode) return widget.child;
    return Stack(
      alignment: Alignment.bottomRight,
      fit: StackFit.loose,
      children: <Widget>[
        widget.child,
        Positioned(
          right: 2,
          bottom: 2,
          child: IconButton(
            icon: const Icon(Icons.arrow_drop_down_outlined),
            splashRadius: 12,
            alignment: Alignment.center,
            iconSize: 16,
            padding: EdgeInsets.zero,
            constraints: BoxConstraints.tight(
              const Size.square(16),
            ),
            onPressed: widget.actions.isEmpty ? null : show,
          ),
        ),
      ],
    );
  }
}

mixin _PrompterOverlayMixin on _PrompterApiMixin {
  // TODO: not implemented
}
```

И, конечно же, обновите нашу форму и оберните каждое текстовое поле новым виджетом.

```dart
class _SignUpFormState extends State<SignUpForm> {
  final TextEditingController _firstNameController = TextEditingController();
  final TextEditingController _secondNameController = TextEditingController();
  final TextEditingController _ageController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();

  @protected
  void clearAll() {
    _firstNameController.clear();
    _secondNameController.clear();
    _ageController.clear();
    _emailController.clear();
  }

  @override
  void dispose() {
    _firstNameController.dispose();
    _secondNameController.dispose();
    _ageController.dispose();
    _emailController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Column(
        mainAxisSize: MainAxisSize.max,
        crossAxisAlignment: CrossAxisAlignment.center,
        children: <Widget>[
          Expanded(
            child: Center(
              child: Prompter(
                actions: <String, VoidCallback>{
                  'License page': () => showLicensePage(context: context),
                  'Unfocus': () => FocusScope.of(context).unfocus(),
                  'Clear all': clearAll,
                },
                child: const FlutterLogo(size: 120),
              ),
            ),
          ),
          Transform.rotate(
            angle: math.pi / 32,
            child: SizedBox(
              height: 108,
              child: ListView(
                padding: const EdgeInsets.all(24),
                scrollDirection: Axis.horizontal,
                physics: const BouncingScrollPhysics(
                  parent: AlwaysScrollableScrollPhysics(),
                ),
                children: <Widget>[
                  SizedBox(
                    width: 180,
                    child: Prompter(
                      actions: <String, VoidCallback>{
                        'John': () => _firstNameController.text = 'John',
                      },
                      child: TextField(
                        decoration: const InputDecoration(
                          labelText: 'First name',
                        ),
                        maxLines: 1,
                        keyboardType: TextInputType.text,
                        controller: _firstNameController,
                      ),
                    ),
                  ),
                  const SizedBox(width: 48),
                  SizedBox(
                    width: 180,
                    child: Prompter(
                      actions: <String, VoidCallback>{
                        'Smith': () => _secondNameController.text = 'Smith',
                        'White': () => _secondNameController.text = 'White',
                      },
                      child: TextField(
                        decoration: const InputDecoration(
                          labelText: 'Second name',
                        ),
                        maxLines: 1,
                        keyboardType: TextInputType.text,
                        controller: _secondNameController,
                      ),
                    ),
                  ),
                  const SizedBox(width: 48),
                  SizedBox(
                    width: 120,
                    child: Prompter(
                      actions: <String, VoidCallback>{
                        '24': () => _ageController.text = '24',
                        '32': () => _ageController.text = '32',
                        '75': () => _ageController.text = '75',
                      },
                      child: TextField(
                        decoration: const InputDecoration(
                          labelText: 'Age',
                          counterText: '',
                        ),
                        maxLength: 3,
                        keyboardType: TextInputType.number,
                        inputFormatters: [
                          FilteringTextInputFormatter.digitsOnly,
                        ],
                        maxLines: 1,
                        controller: _ageController,
                      ),
                    ),
                  ),
                  const SizedBox(width: 48),
                  SizedBox(
                    width: 240,
                    child: Prompter(
                      actions: <String, VoidCallback>{
                        'a@a.a': () => _emailController.text = 'a@a.a',
                        'a@tld.dev': () => _emailController.text = 'a@tld.dev',
                      },
                      child: TextField(
                        decoration: const InputDecoration(
                          labelText: 'e-Mail',
                        ),
                        maxLines: 1,
                        keyboardType: TextInputType.emailAddress,
                        controller: _emailController,
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ),
          const Spacer(),
        ],
      );
}
```

Давайте создадим слой для нашего накладывающегося( Promter ) оверлея.

```dart
class _PromterLayout extends StatelessWidget {
  const _PromterLayout({
    required this.actions,
    required this.hide,
  });

  final Map<String, VoidCallback> actions;
  final VoidCallback hide;

  @override
  Widget build(BuildContext context) => Card(
        margin: EdgeInsets.zero,
        child: Row(
          children: <Widget>[
            for (final action in actions.entries)
              Padding(
                padding: const EdgeInsets.all(2),
                child: ActionChip(
                  label: Text(action.key),
                  onPressed: action.value,
                ),
              ),
            const VerticalDivider(width: 4),
            IconButton(
              icon: const Icon(Icons.close),
              splashRadius: 12,
              alignment: Alignment.center,
              iconSize: 16,
              padding: const EdgeInsets.all(4),
              constraints: BoxConstraints.tight(
                const Size.square(24),
              ),
              onPressed: hide,
            ),
          ],
        ),
      );
}
```

Выглядит неплохо. Но теперь нам нужно записать реализацию нашего хелпера внутри `_PrompterOverlayMixin`.

```dart
mixin _PrompterOverlayMixin on _PrompterApiMixin {
  /// Object connecting [CompositedTransformTarget]
  /// and [CompositedTransformFollower].
  final LayerLink _layerLink = LayerLink();

  /// Current overlay entry, if it exists.
  OverlayEntry? _overlayEntry;

  @override
  void show() {
    super.show();
    hide();
    // Show overlay and set new _overlayEntry
    Overlay.of(context)?.insert(
      _overlayEntry = OverlayEntry(
        builder: (context) => Positioned(
          height: 48,
          // Wrap [CompositedTransformFollower] to allow our overlay
          // entry track and follow parent [CompositedTransformTarget]
          child: CompositedTransformFollower(
            link: _layerLink,
            offset: const Offset(-2, 4),
            targetAnchor: Alignment.bottomRight,
            followerAnchor: Alignment.topRight,
            showWhenUnlinked: false,
            child: _PromterLayout(
              actions: widget.actions,
              hide: hide,
            ),
          ),
        ),
      ),
    );
  }

  @override
  void hide() {
    super.hide();
    if (_overlayEntry == null) return;
    // Remove current overlay entry if it exists.
    _overlayEntry?.remove();
    _overlayEntry = null;
  }

  @override
  void dispose() {
    hide();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) =>

      /// Wrap your widget inside [CompositedTransformTarget]
      /// for tracking capabilities
      CompositedTransformTarget(
        link: _layerLink,
        child: super.build(context),
      );
}

```

После всего вышесказанного мы можем показать [OverlayEntry](https://api.flutter.dev/flutter/widgets/OverlayEntry-class.html) с[CompositedTransformFollower](https://api.flutter.dev/flutter/widgets/CompositedTransformFollower-class.html) который будет следовать за своей целью[CompositedTransformTarget](https://api.flutter.dev/flutter/widgets/CompositedTransformTarget-class.html) используя  [LayerLink](https://api.flutter.dev/flutter/rendering/LayerLink-class.html).

Доп материалы по ссылки в дартпад: [Layer link workshop](https://dartpad.dev/workshops.html?webserver=https://raw.githubusercontent.com/PlugFox/layer_link/master/public)

Ссылка на полный пример: [Full example](https://dartpad.dev/dbd122459cc911589cba450e1613355c)

#PlugFox