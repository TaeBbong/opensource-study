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