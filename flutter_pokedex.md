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

1. json_serializable
2. hive_generator
3. injectable_generator
4. flutter_gen_runner
5. auto_route_generator
6. freezed

이렇게 많은 소스코드 생성 도구가 있었다니...