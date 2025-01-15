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

Deprecated 된 기능에 대해서는 위와 같이 어노테이션 활용, 그리고 대체 변수를 지정해주는 것으로 처리

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
- 참고: 패키지 생성은
```bash
$ flutter create --template=package flutter_test_package
```

3. @staticIconProvider - 어노테이션 일반 설명
이런 어노테이션을 활용하여 class를 선언하는데,
일단 어노테이션이라는 개념이 우리가 원래 알고 있던 데코레이터와 비슷한 개념이라고 한다.
- 데코레이터란 보통 파이썬이나 타입스크립트에서 자주 쓰이고,
- 함수 또는 클래스를 wrapping 해줘서 앞뒤로 기능을 확장시키는 개념이다.
- 보통 flask, fastapi와 같은 웹 api 개발 시 많이 봤었다.
- 이것도 보면 결국 내가 짠 api 기능의 앞뒤로 flask, fastapi 웹 요청의 규격을 맞춰주는 개념이라고 보면 될 것 같다. 
- 물론 세부적인 내용은 직접 각 데코레이터를 확인해봐야 할듯
- 스크립트 언어여서 그런지 런타임 때 코드 앞뒤로 붙어줌
반면 어노테이션은
- 함수 또는 클래스의 동작을 확장시키기도 하지만(데코레이터와 동일하게)
- 코드 컴파일간 코드를 생성하거나(build_runner, source_gen 류) 검사할 때 필요한 메타 데이터를 주입하는 개념이다.
- 앞서 봤던 @Deprecated는 메타 데이터를 제공하는 개념인 것으로 보이고,
- @staticIconProvider는 역시 마찬가지로 메타 데이터를 제공하는 개념

4. Annotation과 Tree Shaking
이 내용은 나름 딥한 부분이라 나중에 따로 포스팅 해야 할듯
일단 간단히 내용을 적어보면, 플러터에서는 Tree Shaking이라는 기법을 통해
컴파일 시 미사용 코드를 제거하는 등 경량화를 실시함
앱 최적화 기법으로 플러터가 구현해놓은 개념인데,
앞서 확인했던 @staticIconProvider의 구현을 보면 아래와 같음.

```dart
class _StaticIconProvider {
  const _StaticIconProvider();
}

/// Annotation for classes that only provide static const [IconData] instances.
///
/// This is a hint to the font tree shaker to ignore the constant instances
/// of [IconData] appearing in the declaration of this class when tree-shaking
/// unused code points from the bundled font.
///
/// Classes with this annotation must have only "static const" members. The
/// presence of any non-const [IconData] instances will preclude apps
/// importing the declaration into their application from being able to use
/// icon tree-shaking during release builds, resulting in larger font assets.
///
/// ```dart
/// @staticIconProvider
/// abstract final class MyCustomIcons {
///   static const String fontFamily = 'MyCustomIcons';
///   static const IconData happyFace = IconData(1, fontFamily: fontFamily);
///   static const IconData sadFace = IconData(2, fontFamily: fontFamily);
/// }
/// ```
const Object staticIconProvider = _StaticIconProvider();
```

staticIconProvider라는 Annotation에 대한 구현이 이것으로 끝이다...
근데 설명을 보면 tree shaker에게 힌트를 제공한다고 나와있는데,
이걸 여기서 어떻게 힌트를 주나 싶어서 flutter builder 쪽 소스코드를 까봤다.

찾아보니 일단 결론은 builder에 staticIconProvider에 대한 내용이 이미 선언되어 있어서,
어떤 라이브러리가 @staticIconProvider를 사용하면
플러터 단에서 이를 인지하고 빌드할 때 최적화를 할 수 있는 것이다.
이는 flutter_tools/build_system/icon_tree_shaker.dart에 선언되어 있다. (세부 경로는 미술)

```dart
class IconTreeShaker {
    ...
    Future<Map<String, List<int>>> _findConstants(File dart, File constFinder, File appDill) async {
    final List<String> cmd = <String>[
      dart.path,
      constFinder.path,
      '--kernel-file',
      appDill.path,
      '--class-library-uri',
      'package:flutter/src/widgets/icon_data.dart',
      '--class-name',
      'IconData',
      '--annotation-class-name',
      '_StaticIconProvider',
      '--annotation-class-library-uri',
      'package:flutter/src/widgets/icon_data.dart',
    ];
    // [vuln] processManager.run으로 명령어 실행할 때 취약점 안생기나?
    _logger.printTrace('Running command: ${cmd.join(' ')}');
    final ProcessResult constFinderProcessResult = await _processManager.run(cmd);

    if (constFinderProcessResult.exitCode != 0) {
      throw IconTreeShakerException._('ConstFinder failure: ${constFinderProcessResult.stderr}');
    }
    final Object? constFinderMap = json.decode(constFinderProcessResult.stdout as String);
    if (constFinderMap is! Map<String, Object?>) {
      throw IconTreeShakerException._(
        'Invalid ConstFinder output: expected a top level JSON object, '
        'got $constFinderMap.',
      );
    }
    final _ConstFinderResult constFinderResult = _ConstFinderResult(constFinderMap);
    if (constFinderResult.hasNonConstantLocations) {
      _logger.printError(
        'This application cannot tree shake icons fonts. '
        'It has non-constant instances of IconData at the '
        'following locations:',
        emphasis: true,
      );
      for (final Map<String, Object?> location in constFinderResult.nonConstantLocations) {
        _logger.printError(
          '- ${location['file']}:${location['line']}:${location['column']}',
          indent: 2,
          hangingIndent: 4,
        );
      }
      throwToolExit(
        'Avoid non-constant invocations of IconData or try to '
        'build again with --no-tree-shake-icons.',
      );
    }
    return _parseConstFinderResult(constFinderResult);
  }
}
```

더 깊게 분석하기엔 쫌 어려워서 요정도 수준에서 받아들이고 넘어가자면,
- IconTreeShaker라는 클래스에서 말그대로 Icon과 관련된 asset의 경량화를 위한 기능들을 구현
- 이 중에서 const를 찾아서 경량화해주는 constFinder가 있는데,
- 이때 인자로 staticIconProvider Annotation을 넘겨 받게 됨
- staticIconProvider의 모든 필드는 static const 이기 때문에,
- 이렇게 Annotation을 const finder가 인지하게 되면
- 해당 Annotation이 붙은 클래스의 필드가 모두 const라는 걸 알 수 있음(믿고 가는거지)
- 우리는 (엄밀하게 알진 못하지만 대충) const로 선언된 값들은 빌드할 때 이점을 갖고 있다는 것을 알기에
    - const : 메모리에 한번만 올라오고 계속 같은 개체 사용
- 해당 Annotation을 활용하여 Icon에 대한 const를 유지하며 경량화를 시킬 수 있겠다!
- 뿐만 아니라 Tree Shaker의 주 역할은 unused const를 제거함에 있기 때문에,
- 아마 이런 제거하는 로직이 또 구현이 되어 있을거고,
- 결국 모든 const를 Tree Shaker가 인지하는 과정이 중요하기 때문에
- 이런 Annotation이 의미를 갖게 될 듯

요 다음에 Tree Shaker와 Annotation 특집 기사를 써야 할 것 같은데,
이때 최적화 결과에 대한 검증 기법도 알아놓으면 좋겠다(Dart DevTools 활용)

참고 : 플러터 Tree Shaker 종류

Code Tree Shaker	
- 사용되지 않는 Dart 코드(클래스, 함수, 변수 등)를 제거하여 앱 크기를 최적화 / Dart 코드
Resource Tree Shaker	
- 사용되지 않는 리소스(아이콘, 이미지, JSON 등)를 제거하여 빌드 크기를 최적화 / 폰트, 이미지, 기타 에셋 데이터