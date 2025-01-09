# font_awesome_flutter

## 분석 1단계 - README, CHANGELOG, pubspec.yaml 등 문서 확인하기

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

- cli로 실행되는 다트 파일
- 인자 파싱도 하고
- FortAwesome/Font-Awesome에서 폰트, json 등 다운로드
- 업데이트 전부 한 다음에
- pub get 하고 종료(adjustPubspecFontIncludes() 함수에서 명령어 실행)

- 그 외에도 여러 함수 있긴 함
- 옵션 설정해야 동작하는 기능들
