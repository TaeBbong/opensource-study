# font_awesome_flutter

## 분석 1단계 - README, CHANGELOG, pubspec.yaml 등 문서 확인하기

### 폴더 구조

```bash
├───.dart_tool
├───.github
│   └───ISSUE_TEMPLATE
├───example
│   ├───lib
├───lib
│   ├───fonts
│   └───src
├───test
└───util
    └───lib
```

일단 폴더 구조는 심플한 편
1) (메인) lib 폴더
2) test 폴더
3) example 폴더
-------------------------------
여기까지는 기본 구성

4) util/lib : 별도 기능을 수행하는 소스코드 포함
5) .github : ISSUE_TEMPLATE을 정의하여 유저/개발자의 request에 대한 양식을 제시 ==> 의미 있는 듯

추가적으로 파일들 일부 살펴보면

1) lib/src/fa_icon.dart, icon_data.dart : 사실상 프로젝트 소스코드의 전반, font_awesome_flutter.dart 파일을 생성하는데 기반이 되는 듯 보임
2) configurator.sh, bat : util을 실행하기 위한 OS 의존 스크립트

```bash
#!/usr/bin/env bash
pushd "$(dirname "$0")" > /dev/null || exit    # pushd: cd를 하면서 현재 경로를 스택에 저장
cd ..
dart ./util/lib/main.dart "$@"
popd > /dev/null || exit   # popd: 스택에 있는 경로를 pop하면서 거길로 이동
```

```bash
C:\util 경로를 스택에 넣고, C:\util로 이동
cd .. => C:\ 로 이동완료
dart ./util/lib/main.dart 실행 with arguments
popd => 다시 C:\util로 이동됨
```

왜 이렇게 하지?? 걍 dart ./lib/main.dart 하면 안되나?
- 스크립트를 실행하는 위치에 따라 제대로 동작하지 않을 수도 있기 때문..
- 어찌되었든 현재 이 스크립트의 경로가 A라고 하면,
- 그 경로의 ../util/lib/main.dart의 위치가 바로 우리가 원하는 main.dart이기 때문
- 근데 그렇다고 해도, 걍 A/lib/main.dart 이렇게 하면 되지 않나?
- 굳이 cd .. ???
- 이거는 분명 다른 스크립트, 프로젝트가 섞여 있어서
- 스크립트가 모여있는 경로로 다시 돌아와야 하기 때문인 듯

- 전반적인 환경에서는 동작에 무리가 없어보임, 하지만 pushd, popd가 설치되어 있지 않은 경우(경량화 리눅스) 미동작하며,
- 상대경로를 찾아가는 과정에서 문제가 발생할 수 있음(스크립트 실행 위치에 따라)
- 어차피 이 프로젝트는 flutter를 위함이고, dart는 모두 설치되어있으니 dart 기반 스크립트를 작성하면 되지 않을까 싶긴 함

```dart
// gpt 버전 코드(제시)
import 'dart:io';

void main(List<String> arguments) {
  // 현재 스크립트 디렉토리 가져오기
  final scriptDir = File(Platform.script.toFilePath()).parent;

  // 상위 디렉토리로 이동
  final projectRoot = scriptDir.parent;

  // 실행할 Dart 파일 경로
  final dartFile = projectRoot.uri.resolve('util/lib/main.dart').toFilePath();

  // Dart 파일이 존재하는지 확인
  if (!File(dartFile).existsSync()) {
    stderr.writeln('Error: main.dart not found at $dartFile');
    exit(1);
  }

  // Dart 파일 실행
  final result = Process.runSync(
    'dart',
    [dartFile, ...arguments],
    workingDirectory: projectRoot.toFilePath(),
  );

  // 결과 출력
  stdout.write(result.stdout);
  stderr.write(result.stderr);

  // 종료 코드 반환
  exit(result.exitCode);
}
```

### 내용

- 플러터에서 사용할 수 있는 아이콘 팩 위젯 패키지.
- [Font Awesome](https://fontawesome.com/icons)에서 무료로 제공되는 아이콘을 위젯 형태로 제공.
- 위 링크에서 제공하는 아이콘 이름과 동일하나, lower camel case로 작성되어 있음
- 예: angle-double-up => angleDoubleUp

- Customizing 지원 => configurator.sh or .bat
- 모든 옵션은 상호운용적(interoperable)이다..?
- 기본적으로 arguments, icons.json 없이 실행하면 font awesome 최신 무료 아이콘으로 업데이트
- configurator.sh 스크립트 분석결과
- dart .\util\lib\main.dart 실행

### util/lib/main.dart 분석 결과

cli로 실행되는 다트 파일
인자 파싱도 하고
FortAwesome/Font-Awesome에서 폰트, json 등 다운로드
업데이트 전부 한 다음에
pubspec.yaml 버전 업데이트 시키고(숫자 올리기)
pub get 하고 종료(adjustPubspecFontIncludes() 함수에서 명령어 실행)

- 이 코드에서 핵심적으로 봐야하는 내용은 main.dart를 cli로 실행했을 때,
- 어떤 파일들이 생성되고 추가되는가임

- 현재까지 확인된 파일들은
1) example/lib/icons.dart
2) lib/fonts/*.ttf
3) lib/font_awesome_flutter.dart

여기서 2번은 다운로드된 파일, 1, 3번 파일은 main.dart를 통해 auto-generated 된 코드이다.
일반적인 코드 제너레이팅은 source_gen와 같은 패키지 의존성을 활용하는데,
이거는 보니까 format string으로 제법 수동 방식의 내용 쓰기를 선택했다.
내용이 규격화 되어 있고 자유도가 없기 때문에(사실상 새로운 버전의 파일을 매핑시키기만 하면 되는거라)
이런 방식을 채택한듯.


