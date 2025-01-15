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

- main.dart 파일은 cli 기반의 스크립트인만큼, 명령어 파싱과 파일 파싱, 읽기 쓰기 등의 기능이 주를 이루고 있음
그래서 코드들이 전반적으로 대충 써진 느낌이 있긴 함

### test/fa_icon_test.dart

일단 이 코드를 통해서 아주 기본적인 테스트 코드를 익혀보자.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter/widgets.dart';
import 'package:font_awesome_flutter/font_awesome_flutter.dart';

void main() {
  testWidgets('Can set opacity for an Icon', (WidgetTester tester) async {
    await tester.pumpWidget(
      const Directionality(
        textDirection: TextDirection.ltr,
        child: IconTheme(
          data: IconThemeData(
            color: Color(0xFF666666),
            opacity: 0.5,
          ),
          child: FaIcon(FontAwesomeIcons.accessibleIcon),
        ),
      ),
    );
    final RichText text = tester.widget(find.byType(RichText));
    expect(text.text.style!.color, const Color(0xFF666666).withOpacity(0.5));
  });
}
```

기본적으로 flutter_test.dart에는 testWidgets라는 함수가 있음
- 이는 하나의 테스트를 생성하는 도구임
- 위젯을 띄우고 그에 대한 결과를 받아야 하니 async-await 사용
- tester.pumpWidget을 통해 가상의 위젯을 생성(테스트 환경에서 위젯 트리 빌드)
- Directionality : 텍스트의 방향을 지정하는 위젯
- IconTheme(IconThemeData())를 통해 아이콘의 색상과 opacity를 설정
- tester.widget(find.byType(RichText)) 코드를 통해 RichText 타입의 위젯을 찾음
- 플러터에서 FaIcon은 RichText로 취급받아서 찾아짐 
- 그렇게 찾은 위젯의 색상이 설정한대로인지 확인

이외 테스트 케이스들을 살펴보니 사이즈 설정 제대로 되는지 확인하는 코드들이 90%
- 이를테면 theme에서 size를 설정하고 icon의 size는 디폴트일 때 사이즈가 어떻게 되는지 등
- 테스트 케이스를 어떤식으로 도출해내는지 궁금함

테스트 실행하려면 다음 명령어 실행(프로젝트 root에서)
```bash
$ flutter test
```

* 참고: flutter_test

```
주요 기능
flutter_test 패키지가 제공하는 기능들은 다음과 같습니다:

위젯 트리 빌드 및 상태 업데이트:
pumpWidget으로 위젯 트리를 빌드.
pump로 애니메이션 및 상태 변화를 반영.

Finder API:
위젯 찾기 도구 (find.text, find.byType, find.byKey 등).

상호작용 시뮬레이션:
터치, 스크롤, 드래그, 키보드 입력 등 (tap, drag, enterText).

기대값 검증:
expect와 다양한 Matcher 사용 (findsOneWidget, findsNothing 등).

모의 타이머:
테스트에서 시간 기반 동작(mock timers)을 검증.
```

보통 단위 테스트, 위젯 테스트 수준까지 할 때 활용되며,
여러 위젯을 pumpWidget 해놓고 기능을 동작시켜서 이후 값을 확인하는 등
간단한 수준의 통합 테스트는 가능함.
- 보통 통합 테스트는 integretion_test 패키지를 사용한다고 함.

### lib/font_awesome_flutter.dart

autogenerated code이긴 하지만, 결국 템플릿은 직접 작성된 내용으로 일부 볼 부분이 있음.

1. @Deprecated
```dart
  /// Alias vcard for icon [addressCard]
  @Deprecated('Use "addressCard" instead.')
  static const IconData vcard = addressCard;
```

Deprecated 된 기능에 대해서는 위와 같이 데코레이터 활용, 그리고 대체 변수를 지정해주는 것으로 처리

2. library
```dart
library font_awesome_flutter;

import 'package:flutter/widgets.dart';
import 'package:font_awesome_flutter/src/icon_data.dart';
export 'package:font_awesome_flutter/src/fa_icon.dart';
export 'package:font_awesome_flutter/src/icon_data.dart';
```

import만 써왔었는데, library와 export 문법에 대해 간단히 알아보면,
- library: 패키지 생성시 자동으로 생성되는 코드이자,
- lib/font_awesome_flutter.dart 파일과 동일한 이름의 library를 선언
- 보통 이 이름은 패키지 이름과 맞춰지는 경우가 많음(기본값이 그러함)
- export: lib/src 하위의 여러 파일들이 있다면 이를 재노출시킴으로써
- 패키지의 메인 파일(lib/font_awesome_flutter.dart)만 import해도
- 하위의 fa_icon.dart, icon_data.dart 파일도 함께 import가 되는 효과