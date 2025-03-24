## GetX - State Management 기능 분석(focused on Obx, Rx)

### 분석 목표

- [ ] Obx와 Rx의 동작원리 분석
- [ ] Obx가 어떻게 Rx의 변화를 감지하여 리렌더링을 하는지 확인
- [ ] Obx에서 리렌더링 되는 조건은 어떤지 확인
- [ ] Obx를 사용하는 위젯의 life cycle 확인
- [ ] (결론 도출) Obx와 Rx를 어떻게 써야 제일 맛있는지 도출

### 구조 분석

- 생각보다 소스코드가 방대하거나 하지 않았음...
- 코끼리 더듬기 메타로 상태 관리 쪽만 살펴보면,
    - lib/get_rx
    - lib/get_state_management
- 이렇게 폴더 두개가 전부이며,
- 전체 파일들도 이게 전부임
- ![getx-1](./imgs/getx-1.png)

### 테스트 코드 분석

- 역시 생각보다 테스트코드가 조잡(?)하달까..
- 단순하다고 표현하는게 맞겠지만..

#### state_manager/get_obx_test.dart

- get_obx_test에서는 Obx() 위젯으로 감싸고 있는 위젯들에 controller.value를 표시하게 하고,
- 버튼 등을 트리거하여 controller.value를 변경시켰을 때,
- 위젯에 표시되는 값이 잘 변경되는지 확인

- 좀 더 복잡한 케이스가 있었으면 좋았겠다..
- Map 데이터를 변경시키는 테스트 코드를 추가하여 통과함.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:get/get.dart';

void main() {
  testWidgets("GetxController smoke test", (tester) async {
    final controller = Get.put(Controller());
    await tester.pumpWidget(
      MaterialApp(
        home: Column(
          children: [
            Obx(
              () => Column(children: [
                Text('Count: ${controller.counter.value}'),
                Text('Double: ${controller.doubleNum.value}'),
                Text('String: ${controller.string.value}'),
                Text('List: ${controller.list.length}'),
                Text('Bool: ${controller.boolean.value}'),
                Text('Map: ${controller.map.length}'),
                TextButton(
                  onPressed: controller.increment,
                  child: const Text("increment"),
                ),
                TextButton(
                  onPressed: controller.updateMap,
                  child: const Text("update"),
                ),
                Obx(() => Text('Obx: ${controller.map.length}'))
              ]),
            ),
          ],
        ),
      ),
    );

    expect(find.text("Count: 0"), findsOneWidget);
    expect(find.text("Double: 0.0"), findsOneWidget);
    expect(find.text("String: string"), findsOneWidget);
    expect(find.text("Bool: true"), findsOneWidget);
    expect(find.text("List: 0"), findsOneWidget);
    expect(find.text("Map: 0"), findsOneWidget);
    expect(find.text("Obx: 0"), findsOneWidget);

    Controller.to.increment();

    await tester.pump();

    expect(find.text("Count: 1"), findsOneWidget);

    await tester.tap(find.text('increment'));

    await tester.pump();

    expect(find.text("Count: 2"), findsOneWidget);

    await tester.tap(find.text('update'));

    await tester.pump();

    expect(find.text("Obx: 2"), findsOneWidget);
  });
}

class Controller extends GetxController {
  static Controller get to => Get.find();

  RxInt counter = 0.obs;
  RxDouble doubleNum = 0.0.obs;
  RxString string = "string".obs;
  RxList list = [].obs;
  RxMap map = {}.obs;
  RxBool boolean = true.obs;

  void increment() {
    counter.value++;
  }

  void updateMap() {
    Map<String, dynamic> testMap = {"key": "value", "key2": 2};
    map.value = testMap;
  }
}

```


#### state_manager/get_rxstate_test.dart

- 여기도 obx_test와 내용은 완전히 동일하며,
- rx 상태를 관찰하는 위젯만 GetX 위젯을 사용

### 문서 읽기

- state_management.md 문서를 읽어보자.

- 반응형 상태 관리자(Rx, Obx)는 개념적으로 Stream과 유사한 컨셉으로 설계된 듯 하다.
- ChangeNotifier 기반의 상태관리 도구는 notifyListener가 호출되면 관련된 모든 위젯이 변경되는데,
- GetX를 쓰고 있으면 딱 변경된 위젯만 다시 빌드한다고 한다.
- 또한 실제 값이 변경되었을 때에만 빌드가 된다.

- Stream : Rx변수 / StreamBuilder : Obx(() => ) 이렇게 대응된다고 봐야 한다.

- 단순형 상태 관리자(GetBuilder, GetxController)는 Provider랑 비슷한 원리로,
- ChangeNotifier 기반과 유사하게 상태 변화에 따라 수동으로 update를 실행해줘야 한다.

- GetX라는 위젯도 있고, Obx라는 위젯도 있다.
- GetX라는 위젯은 반응형 상태 관리자(Obx)와 마찬가지로 쓰일 수 있으나, controller를 초기화 할 수 있다.
- Obx는 단순히 rx 값을 구독하고 있을 뿐..
- 대신 성능면에서 GetBuilder(단순형) >>> Obx >> GetX


### 소스코드 분석

