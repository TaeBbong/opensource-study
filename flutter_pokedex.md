# flutter_pokedex

### 프로젝트 구조

```bash
lib
├───core
├───data
│   ├───entities
│   ├───repositories
│   ├───source
│   │   ├───github
│   │   │   └───models
│   │   ├───local
│   │   │   └───models
│   │   └───mappers
│   ├───states
│   │   ├───item
│   │   ├───pokemon
│   │   └───settings
│   └───usecases
├───presenter
│   ├───modals
│   ├───navigation
│   ├───pages
│   │   ├───home
│   │   │   ├───sections
│   │   │   └───widgets
│   │   ├───items
│   │   │   ├───sections
│   │   │   └───widgets
│   │   ├───pokedex
│   │   │   ├───sections
│   │   │   └───widgets
│   │   ├───pokemon_info
│   │   │   └───sections
│   │   ├───splash
│   │   └───types
│   │       └───type_entities
│   ├───themes
│   │   └───themes
│   └───widgets
└───utils
    └───extensions
```

### fvm : flutter version manager

놀랍게도 시작하자마자 프로젝트 실행하는 단계에서 막혀서 이슈 확인해봄.
일단 문제는 뭐였냐면 플러터 버전 이슈....
생각보다 이 프로젝트가 지속 업데이트가 되지는 않는 듯 하다..
아무튼 커밋 로그들을 살펴보면서 확인해보니 3.16.5 버전으로 업그레이드 한 후에 추가 업데이트는 없었던듯
내 환경에 설치된 거는 3.23 버전이었음.
플러터 이 놈들은 버전 업데이트를 하면 적당히 바뀌든가 인터페이스는 놔두고 하든가 해야 하는데,
거의 뭐 전부 Deprecated 때려버리니까 뭐 어떻게 간단히 트러블슈팅할 수준이 아니었음...

근데 그렇다고 매번 그 버전의 플러터를 받아서 설치해놓고 일일이 지정하면서 쓰긴 귀찮으니까..
python의 venv같은 도구가 없을까 하고 알아보니 fvm이라는 것이 있었다..
난 왜 이걸 여태 몰랐지

암튼 이걸 설치하고 사용하기로 결심

설치법
```bash
$ dart pub global activate fvm // fvm 설치, 활성화
$ 또는 choco/brew install fvm
$ fvm releases // flutter 버전들 리스트업
$ fvm install 3.16.5 // 특정 버전 설치
$ fvm use 3.16.5 // 해당 프로젝트 디렉토리에서 실행, 해당 프로젝트에서 해당 버전 사용
```

설치나 사용법이 쉬워서 앞으로도 애용할 듯 싶다.
대신 이후에 dart pub get 같은 명령어를 실행할 때에는 앞에 fvm을 꼭 붙여야 한다.

```bash
$ fvm dart pub get
$ fvm dart run build_runner build
```

### dart run build_runner build

build_runner에 대해 좀 알아보자.
pubspec.yaml 보니까 dev_dependencies 중에서 제너레이터가 꽤 많음

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.4.7
  json_serializable: ^6.7.1
  hive_generator: ^2.0.1
  flutter_lints: ^3.0.1
  injectable_generator: ^2.4.1
  flutter_gen_runner: ^5.4.0
  auto_route_generator: ^7.3.2
  freezed: ^2.4.6
```

얘네들을 전부 한번에 어떻게 실행시키나 봤는데,
그냥 dart run build_runner build 하니까 되더라.

.vscode에서 settings.json에다가

```json
"files.exclude": {
    "**/.gitkeep": true,
    "**/*.g.dart": true,
    "**/*.gr.dart": true,
    "**/*.gen.dart": true,
    "**/*.freezed.dart": true,
    "**/*.config.dart": true
  }
```

이런거 넣으니까 제너레이팅 된 파일들은 보이지 않도록 관리
편한거 같음..!

build_runner는 소스코드를 제너레이팅하는 도구는 아니고,
source_gen 류 패키지들을 관리하는 빌드 도구
build_runner는 각 패키지마다 미리 선언된 생성 흐름에 따라 각각의 source_gen을 실행시킴
실제로 소스코드를 생성하는 것은 각 패키지이다!

결국 이 프로젝트는 아주 많은 소스코드 생성 도구를 사용한다.

1. json_serializable // json_annotation 기반, freezed와 연계하여 모델 toJson, fromJson 생성
2. hive_generator // hive 기반, 키-벨류 쌍의 로컬 DB 어댑터 생성
3. injectable_generator // get_it, injectable 기반 의존성 주입 객체 생성
4. flutter_gen_runner // assets, fonts 선언을 위한 다트 파일 생성
5. auto_route_generator // auto_route 기반 페이지별 라우트 객체 생성
6. freezed // 불변 모델 생성

이렇게 많은 소스코드 생성 도구가 있었다니...

암튼 build_runner는 각 소스코드 생성 패키지가 미리 선언해놓은 build.yaml 파일을 참조하여,
소스코드 생성을 실행시켜준다고 한다.
build_runner를 디펜던시로 가지고 있는 패키지들은 build.yaml이 필수적인 듯
그래서 각 패키지 별로 build.yaml을 조금 찾아봤음.

```yaml
# freezed
targets:
  $default:
    builders:
      freezed:
        enabled: true
        generate_for:
          exclude:
            - test
            - example
          include:
            - test/integration/*
            - test/integration/**/*
      source_gen|combining_builder:
        options:
          ignore_for_file:
            - "type=lint"

builders:
  freezed:
    import: "package:freezed/builder.dart"
    builder_factories: ["freezed"]
    build_extensions: { ".dart": [".freezed.dart"] }
    auto_apply: dependents
    build_to: source
    runs_before: ["json_serializable|json_serializable"]
```

```yaml
# injectable_generator
builders:
  injectable_builder:
    import: "package:injectable_generator/builder.dart"
    builder_factories: ["injectableBuilder"]
    build_extensions: { ".dart": [".injectable.json"] }
    auto_apply: dependents
    runs_before: ["injectable_generator|injectable_config_builder"]
    build_to: cache
  injectable_config_builder:
    import: "package:injectable_generator/builder.dart"
    builder_factories: ["injectableConfigBuilder"]
    build_extensions: { ".dart": [".config.dart",".module.dart"] }
    auto_apply: dependents
    build_to: source
```

두 build.yaml 파일을 비교해보면 freezed는 targets를 별도 설정해줬고,
injectable_generator는 builders만 설정해놓았음

targets 항목은 선택사항으로, build_runner가 탐색해야 하는 프로젝트 코드의 범위를 좁혀줄 수 있음
또한 코드 생성 로직을 일부 커스텀할 수 있는데,
injectable_builder는 그럴 필요가 없을 정도로 적은 규모이기에 targets 없이 적어놓은 듯
이것은 auto_route_generator도 마찬가지.

그런데 auto_route_generator는 프로젝트 단에서 build.yaml을 따로 선언해놓았음

```yaml
targets:
  $default:
    builders:
      auto_route_generator:auto_route_generator:
        options:
          enable_cached_builds: true
        generate_for:
          - lib/presenter/pages/**/*.dart
      auto_route_generator:auto_router_generator:
        options:
          enable_cached_builds: true
        generate_for:
          - lib/presenter/navigation/navigation.dart
```

이 프로젝트를 만든 사람의 의중을 추리해보자면,
- auto_route_generator가 타겟으로 하는 범위를 좁히고 싶었음(pages 하위만 보면 되기 때문)
- auto_route랑 auto_router를 구분하고 싶었음(개별 페이지 라우팅 / 앱 전체 네비게이션)
- 이 때문에 앞서 설명했던 빌드 로직의 커스텀을 위해, targets를 별도 설정해놓은 걸로 보임
- 왜냐면 auto_route_generator 역시 패키지 자체는 targets를 설정하지 않은 순정상태이기 때문
- 결국 각 패키지를 만든 사람들이 각기 다른 판단을 했기 때문이라고 여겨짐

build_runner와 소스 생성 도구에 대해서도 깊이 연구하고 싶은 느낌....

### 프로젝트 구조 파악

이 프로젝트는 bloc을 기반으로 한 프로젝트로, 전통적인 bloc architecture를 따르고 있는 것 같진 않음
폴더/파일들이 많아서 좀 살펴볼게 많을 것 같은데...

일단 크게 보면
```bash
data
presenter
core
utils
di.dart
main.dart
```

그 중에서 core와 utils은 프로젝트 전체 로직과 연관이 적은 부분으로 일단 제외하면,
주요한 파트는 data와 presenter로 나뉘어지는 듯

data 영역은 DB, 모델로부터 데이터를 전달받고, 실제 비즈니스 로직을 처리해주는 bloc까지 구현
presenter 영역은 bloc widget 기반 화면을 표현하는 위젯 영역으로 구현

일단 크게 이해하면 이정도인듯

### main.dart 부터 따라가기

구조가 넘 복잡해서 일단 메인부터 따라가보기로 결정

```dart
// main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await configureDependencies(); // @InjectableInit()

  runApp(
    GlobalBlocProviders(
      child: PokedexApp(),
    ),
  );
}
```

configureDependencies()는 의존성 주입하는 코드로 확인(자동생성 코드)
GlobalBlocProviders는 앱 전체에 대한 Bloc들을 주입하기 위한 커스텀 위젯으로,

```dart
class GlobalBlocProviders extends StatelessWidget {
  final Widget child;

  const GlobalBlocProviders({
    super.key,
    required this.child,
  });

  @override
  Widget build(BuildContext context) {
    return MultiBlocProvider(
      providers: [
        BlocProvider<PokemonBloc>(
          create: (context) => getIt.get<PokemonBloc>(),
        ),
        BlocProvider<ItemBloc>(
          create: (context) => getIt.get<ItemBloc>(),
        ),
        BlocProvider<SettingsBloc>(
          create: (context) => getIt.get<SettingsBloc>(),
        )
      ],
      child: child,
    );
  }
}
```

이런 식으로 구현되어있음. 결국,

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await configureDependencies(); // @InjectableInit()

  runApp(
    MultiBlocProvider(
      providers: [
        BlocProvider<PokemonBloc>(
          create: (context) => getIt.get<PokemonBloc>(),
        ),
        BlocProvider<ItemBloc>(
          create: (context) => getIt.get<ItemBloc>(),
        ),
        BlocProvider<SettingsBloc>(
          create: (context) => getIt.get<SettingsBloc>(),
        )
      ],
      child: PokedexApp(),
    ),
  );
}
```

이거랑 똑같은 코드. 이런 부분은 굳이 싶긴 하다. 그리 복잡하지 않은 코드를 굳이 한번 더 감싼 느낌.

암튼 일단 MultiBlocProvider 상에서 선언된 Bloc은 3개로, PokemonBloc, ItemBloc, SettingsBloc이 있다.
각각의 Bloc은 @singleton annotation을 통해 injectable로 자동 생성할 수 있도록 지원하는데,
코드 생성기가 너무 많아서 그런지 아직 감이 잘 안잡히고 좀 더 복잡해보이는 경향이 있다..

일단 di는 Bloc을 싱글톤으로 제공한다는 것만 알고 넘어가자.

다음은 PokedexApp() 위젯을 선언한 app.dart의 내용이다.

```dart
// lib/presenter/app.dart
import 'dart:ui';

import 'package:flutter/material.dart';
import 'package:flutter_web_frame/flutter_web_frame.dart';
import 'package:pokedex/presenter/navigation/navigation.dart';
import 'package:pokedex/data/states/settings/settings_selector.dart';

class PokedexApp extends StatelessWidget {
  final AppRouter _router = AppRouter();

  PokedexApp({super.key});

  @override
  Widget build(BuildContext context) {
    return FlutterWebFrame(
      maximumSize: const Size(400, 800),
      backgroundColor: Colors.black12,
      enabled: MediaQuery.sizeOf(context).shortestSide > 600,
      builder: (_) => SettingsThemeSelector(
        builder: (theme) => MaterialApp.router(
          title: 'Flutter Pokedex',
          theme: theme.themeData,
          routerConfig: _router.config(),
          scrollBehavior: AppScrollBehavior(),
        ),
      ),
    );
  }
}

class AppScrollBehavior extends MaterialScrollBehavior {
  @override
  Set<PointerDeviceKind> get dragDevices => {
        PointerDeviceKind.touch,
        PointerDeviceKind.mouse,
        PointerDeviceKind.trackpad,
      };
}
```

- MaterialApp을 .router로 확장하여 사용 중
- router는 AppRouter()를 가져오며, 이는 lib/presenter/navigation/navigation.dart를 참조
- AppRouter()는 auto_route_generator를 통해 path와 page를 제공받아 자동으로 라우터 생성

```dart
part 'navigation.gr.dart';

@AutoRouterConfig()
class AppRouter extends _$AppRouter {
  @override
  List<AutoRoute> get routes => [
        AutoRoute(path: '/', page: SplashRoute.page),
        AutoRoute(path: '/home', page: HomeRoute.page),
        AutoRoute(path: '/pokemons', page: PokedexRoute.page),
        AutoRoute(path: '/pokemons/:id', page: PokemonInfoRoute.page),
        AutoRoute(path: '/types', page: TypeEffectRoute.page),
        AutoRoute(path: '/items', page: ItemsRoute.page),
      ];

  @override
  RouteType get defaultRouteType => const RouteType.custom(
        transitionsBuilder: TransitionsBuilders.fadeIn,
      );
}
```

GetX로 구현을 하면 보통 GetXRouter 덕분에 위처럼만 구현해놔도 알아서 잘 구현이 되었었던걸로 기억
AutoRouter가 어떻게 생성되나 살펴보면,

```dart
abstract class _$AppRouter extends RootStackRouter {
  // ignore: unused_element
  _$AppRouter({super.navigatorKey});

  @override
  final Map<String, PageFactory> pagesMap = {
    ...
    PokedexRoute.name: (routeData) {
      return AutoRoutePage<dynamic>(
        routeData: routeData,
        child: const PokedexPage(),
      );
    },
    PokemonInfoRoute.name: (routeData) {
      final pathParams = routeData.inheritedPathParams;
      final args = routeData.argsAs<PokemonInfoRouteArgs>(
          orElse: () => PokemonInfoRouteArgs(id: pathParams.getString('id')));
      return AutoRoutePage<dynamic>(
        routeData: routeData,
        child: PokemonInfoPage(
          key: args.key,
          id: args.id,
        ),
      );
    },
    ...
  };
}
```

이런식으로 구현이 되더라. id와 같이 인자가 필요한 페이지에 대해서도 위처럼 구현이 되는 것으로 확인
여기서 각각의 Route 클래스는 아래와 같이 자동 생성됨.

```dart
/// generated route for
/// [PokedexPage]
class PokedexRoute extends PageRouteInfo<void> {
  const PokedexRoute({List<PageRouteInfo>? children})
      : super(
          PokedexRoute.name,
          initialChildren: children,
        );

  static const String name = 'PokedexRoute';

  static const PageInfo<void> page = PageInfo<void>(name);
}

/// generated route for
/// [PokemonInfoPage]
class PokemonInfoRoute extends PageRouteInfo<PokemonInfoRouteArgs> {
  PokemonInfoRoute({
    Key? key,
    required String id,
    List<PageRouteInfo>? children,
  }) : super(
          PokemonInfoRoute.name,
          args: PokemonInfoRouteArgs(
            key: key,
            id: id,
          ),
          rawPathParams: {'id': id},
          initialChildren: children,
        );

  static const String name = 'PokemonInfoRoute';

  static const PageInfo<PokemonInfoRouteArgs> page =
      PageInfo<PokemonInfoRouteArgs>(name);
}

class PokemonInfoRouteArgs {
  const PokemonInfoRouteArgs({
    this.key,
    required this.id,
  });

  final Key? key;

  final String id;

  @override
  String toString() {
    return 'PokemonInfoRouteArgs{key: $key, id: $id}';
  }
}
```

자동 생성되는 코드의 구현체까지 파기에는 내용이 너무 깊고,
이런 패키지를 활용하면 이런 식으로 코드가 생성되어, 인자가 필요한 라우트에 대해서도
잘 처리가 된다, 정도만 이해하고 활용할 수 있으면 되겠다.

참고) GetX Route Management

```dart
void main() {
  runApp(
    GetMaterialApp(
      initialRoute: '/',
      getPages: [
      GetPage(
        name: '/',
        page: () => MyHomePage(),
      ),
      GetPage(
        name: '/profile/',
        page: () => MyProfile(),
      ),
       //You can define a different page for routes with arguments, and another without arguments, but for that you must use the slash '/' on the route that will not receive arguments as above.
       GetPage(
        name: '/profile/:user',
        page: () => UserProfile(),
      ),
      GetPage(
        name: '/third',
        page: () => Third(),
        transition: Transition.cupertino  
      ),
     ],
    )
  );
}
```

~그저 딸깍~

다시 app.dart의 PokedexApp으로 돌아와서,
FlutterWebFrame은 무엇인가?
쉽게 말해서, 반응형을 지원 안하는데 큰 디바이스로 실행되는 경우를 대비하여,
말그대로 프레임 사이즈를 미리 지정해놓을 수 있는 위젯

```txt
// dart api
Make the frame max width / size when on large devices such as Web or Desktop.

This is very suitable to be used to limit the size of content, if your application is not yet available a responsive version if it is run on multiple devices / platforms.
```

일단 디테일은 이정도 따라가고, 다시 전체적인 개관을 살펴보기로..

### bloc과 아키텍처, 계층 구분

아쉽게도 이 친구의 README는 전혀 자세하지 않음(todo만 적어놨네..)
설계를 어떻게 했는지 사고의 흐름이 나와있으면 좋을텐데 아쉽더라
아무튼 폴더 구조를 기반으로 gpt 한테 물어보니까 
이 프로젝트는 클린 아키텍처와 Bloc 패턴을 적용했다고 함
~gpt 대단해..~

폴더 구조를 보면 앞서 확인했던 것처럼,

```
data 영역은 DB, 모델로부터 데이터를 전달받고, 실제 비즈니스 로직을 처리해주는 bloc까지 구현
presenter 영역은 bloc widget 기반 화면을 표현하는 위젯 영역으로 구현
```

이런 느낌으로 받았었음.
더 세부적으로 확인을 해보면,
일단 presenter는 더 볼거 없음. UI를 다루는 계층으로, 데이터 처리 및 비즈니스 로직과 무관
핵심은 data인데, 이 안에는 entities, repositories, source, states, usecases가 있음
각각의 기능과 구조를 봤을 때, 보통의 클린 아키텍처를 거의 따른다고 볼 수 있음
(data) source : 데이터를 실제로 가져오는 영역 // dio(http), hive(local) 기반 데이터 소스 및 네트워크 처리 기능
(domain) entities : 주로 모델을 선언해놓는 부분 // freezed 기반의 모델만 선언되어 있음
(domain) repositories : source에서 가져온 데이터를 가공 // getAllItems, getItem와 같은 메소드 구현
(domain) usecases : repository로부터 데이터를 전달받아 states(bloc)으로 전달. 이때 단일 책임 논리로, 하나의 작업만을 수행
(presentation) states : 비즈니스 로직과 UI 영역을 연결해주기 위한 bloc 구현체 부분으로, 데이터가 필요하면 usecase로부터 받아옴

```
(참고) 보통의 클린 아키텍처
(data) source - remote/local, models, services
(domain) entities, repositories, usecases
(presentation) states, pages
```

실제로는 이러한데, 여기 구현된 방식을 살펴보면 usecases가 약간 불필요한 느낌이 없지않아있음
왜냐면 usecases는 결국 repository의 메소드(getAllItems, getItem)를 호출하게 되는데,
repository와 states를 직접 연결 안 시키고 usecases를 중간에 두는 방식을 굳이 선택한 이유가 뭘까?

이를테면 데이터를 받아와 필터링하는 기능이 필요하다고 생각해보자.
이때 이 기능은 어찌보면 데이터의 가공이라기 보단, 비즈니스 로직에 더 가깝다고 볼 수 있다.
범용성을 고려하면, 특정 케이스에 대해서 필요한 데이터 전달은 repository에 넣기에 너무 특수하다고 느낄 수 있음
repository는 범용적으로 데이터를 가져와 처리하는 기능으로 두고,
usecases에서는 각 비즈니스 로직(기능)에 맞게 repository로부터 전달받은 데이터를 가공하도록 하는 것.

```
// gpt 피셜 usecase와 repository 분리한 이유
비즈니스 로직 분리: BLoC이나 Repository에 비즈니스 로직이 포함되지 않도록 캡슐화.
테스트 용이성: UseCase 단위로 독립적인 테스트 가능.
재사용성: 다양한 곳에서 동일한 비즈니스 로직을 재사용.
데이터 소스 변경의 유연성: 데이터 소스 변경 시 최소한의 수정으로 시스템 유지 가능.
```

실제로 폴더를 구성하고 구현하는 것은 스타일마다 다를 수 있으니,
클린 아키텍처의 설계 개념을 이해하는게 더 중요할 듯 하다.

클린 아키텍처의 주요 개념은
1. 추상화 개념으로 관심사를 분리시키고,
2. 의존도를 낮춘다.
3. 종속성 규칙을 지킨다.
* 고수준 정책 : Business Logic, Entities
* 저수준 정책 : UI, Presentations, Controllers
코드의 종속성은 저수준->고수준을 가리키며, 고수준 정책 영역은 저수준의 변경에 변화를 받지 않는다.

다른 이론들은 모두 알아서 지키는, 자유도가 높지만
의존성/종속성 관련해서는 철저히 지킬 것을 권고하고 있음
결국 고수준과 저수준을 잘 나눠서 영향을 받지 않도록, 안전하게 확장 가능한 코드를 만들겠다는 원칙

### 클린 아키텍처의 시선에서 바라본 프로젝트 구조

앞서 클린 아키텍처를 어느정도 확인했었고,
data / domain / presentation 영역으로 구분됨을 확인
이 프로젝트는 아래와 같은 구조를 보이며

(data) source : 데이터를 실제로 가져오는 영역 // dio(http), hive(local) 기반 데이터 소스 및 네트워크 처리 기능
(domain) entities : 주로 모델을 선언해놓는 부분 // freezed 기반의 모델만 선언되어 있음
(domain) repositories : source에서 가져온 데이터를 가공 // getAllItems, getItem와 같은 메소드 구현
(domain) usecases : repository로부터 데이터를 전달받아 states(bloc)으로 전달. 이때 단일 책임 논리로, 하나의 작업만을 수행
(presentation) states : 비즈니스 로직과 UI 영역을 연결해주기 위한 bloc 구현체 부분으로, 데이터가 필요하면 usecase로부터 받아옴

원래는 domain에 들어있어야 하는 폴더들도 일부 data와 presentation 쪽으로 합쳐서,
domain을 따로 두지 않는 구조로 작성.
프로젝트 규모에 맞춰 선택한 판단으로 보여지며,
결국 각 세부 폴더의 기능이 적절히 동작한다면 클린 아키텍처로 설계되었다고 볼 수 있겠음

기능 몇개를 예시로 따라가보면서 구조를 제대로 이해해보겠다.

1) Pokemon 목록 조회

```dart
// 0. Home/header.dart에서 Route push
_CategoryCard(
  title: 'Pokedex',
  color: AppColors.teal,
  onPressed: () => context.router.push(const PokedexRoute()),
),

// 1. Router에서 /pokemons
AutoRoute(path: '/pokemons', page: PokedexRoute.page),
```

```dart
// Pokedex.dart에서 pokemon_grid.dart를 통해 목록 보여주기
// pokedex.dart
part 'sections/fab_menu.dart';
part 'sections/pokemon_grid.dart'; // 이런식으로 하위 파일들을 묶기도 하는구나..

@RoutePage()
class PokedexPage extends StatefulWidget {
  const PokedexPage({super.key});

  @override
  State<StatefulWidget> createState() => _PokedexPageState();
}

// sections/pokemon_grid.dart
part of '../pokedex.dart'; // 이런식으로 하위 파일들을 묶기도 하는구나..

class _PokemonGrid extends StatefulWidget {
  const _PokemonGrid();

  @override
  _PokemonGridState createState() => _PokemonGridState();
}

class _PokemonGridState extends State<_PokemonGrid> {
  static const double _endReachedThreshold = 200;

  final GlobalKey<NestedScrollViewState> _scrollKey = GlobalKey();

  PokemonBloc get pokemonBloc => context.read<PokemonBloc>();

  @override
  void initState() {
    super.initState();

    scheduleMicrotask(() {
      pokemonBloc.add(const PokemonLoadStarted());
      _scrollKey.currentState?.innerController.addListener(_onScroll);
    });
  }
  ...
}
```
#### part, import 

part와 import의 차이에 대해 Flutter가 제공하는 설명은,

> In Dart, private members are accessible within the same library. With import you import a library and can access only its public members. With part/part of you can split one library into several files and private members are accessible for all code within these files.

> import를 하면 그것의 public 요소들만 접근 가능, part를 통해 라이브러리를 쪼개면 해당 파일들에 있는 private(_)에도 접근 가능

결국 part, part of는 한 라이브러리를 구성하는 관계 표시를 위한 것으로, 
다른 라이브러리를 가져오는 import랑은 다른 느낌.

#### StatefulWidget + Other State Manager?

_PokemonGrid는 StatefulWidget으로 구성됨
다른 상태 관리 패키지를 활용하다보면 Stateless를 일괄적용하기 마련인데,
여기서는 Bloc과 함께 StatefulWidget을 사용함.
왜 그렇게 했을까??

결국은 또 관심사의 분리였다...

UI에 대한 변화는 데이터 로직과는 별도로 관리되는게 좋고,
UI에 대한 처리는 위젯 단에서 처리하는게 더 적절한 방법이라고 한다. (gpt 피셜)
결론적으로 UI 이벤트는 위젯, 즉 StatefulWidget에서 처리하는게 더 직관적이고 적절하며,
이는 다른 상태관리 도구(Provider, GetX)에서도 동일하게 적용되는 논리임.

앞으로 다른 프로젝트를 리뷰할 때에도 비슷하게 구현했는지 확인해봐야겠음.
(이거 땜에 맨날 위젯에서 이벤트 처리할 때 onInit 이런거 처리하느라 힘들었는데...)

### (PR) NetworkImage fetch failure 고치기(Retry)

프로젝트를 실행해서 앱을 보다보면, 이미지를 제대로 못 불러오는 경우를 확인할 수 있음(로그)
NetworkImage를 가져오는 부분에서 관련 오류가 발생하는데,
Retry를 여러번 하여 가져올 수 있도록 하면 가능할 듯 싶음
이거 고쳐서 PR하는 것을 시도해보자!

관련해서 flutter_image 패키지(공식)에 NetworkImageWithRetry 위젯이 있는데, 일단 이걸로 대충 넣었을 때에는 제대로 동작하지 않음
현재 프로젝트에는 cached_network_image 위젯을 적용, 사용 중임
둘다 웹에서는 지원안된대서 모바일로 실행 테스트 결과 : 잘 동작함!! 잘 가져옴..!!
(플러터 웹은 진짜 쓰레기다 이거 꼭 뜯어 고치던가 해야지)
CachedNetworkImage 위젯의 경우 Retry 관련 옵션이 없음
flutter_image 패키지의 NetworkImageWithRetry 위젯은 캐싱이 제대로 되지 않음
둘의 단점을 보완하여 합체할 수 있는 적절한 위젯 패키지를 만들 필요가 있겠다..!

흠 일단 그럼 NetworkImage fetch failure는 웹 환경에서만 발생하는거고,
cached_network_image 패키지 또한 문제 없는 걸로 확인.
근데 cached_network_image repo에서는 web도 지원하는 것으로 확인
브라우저 캐싱만 될지 몰라도, 일단 되긴 되야 하는데...

관련해서 확인해야 할 사항은
1. 기본 위젯(Image.network)을 사용해도 이미지가 안가져와 지는가?
2. 가져와지면 cached_network_image의 문제, 아니라면 cors나 https와 같은 웹 보안 자체 문제
