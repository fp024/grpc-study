# 언어 가이드 (proto3)

> 원문: https://developers.google.com/protocol-buffers/docs/proto3

### 이 페이지의 내용

- 메시지 타입 정의
- 스칼라 값 타입
- 기본 값
- 열거
- 다른 메시지 타입 사용
- 중첩 타입
- 메시지 타입 업데이트
- Unknown 필드
- Any
- Oneof
- 맵
- 패키지
- 서비스 정의
- JSON 매핑
- 옵션
- 클래스 생성하기

이 가이드는 `.proto` 파일 구문과 `.proto` 파일에서 데이터 액세스 클래스를 생성하는 방법을 포함하여 프로토콜 버퍼 데이터를 구조화하기 위해 프로토콜 버퍼 언어를 사용하는 방법을 설명합니다. 프로토콜 버퍼 언어의 **proto3** 버전을 다룹니다. **proto2** 구문에 대한 정보는 [Proto2 언어 가이드](https://developers.google.com/protocol-buffers/docs/proto)를 참조하세요.

이것은 레퍼런스 가이드입니다. 이 문서에 설명된 많은 기능을 사용하는 단계별 예제를 보려면 선택한 언어에 대한 [튜토리얼](https://developers.google.com/protocol-buffers/docs/tutorials)를 참조하십시오(현재 proto2만 해당, 더 많은 proto3 문서가 곧 제공될 예정임).



## 메시지 타입 정의

먼저 매우 간단한 예를 살펴보겠습니다. 각 검색 요청에 쿼리 문자열, 관심 있는 특정 결과 페이지 및 페이지당 결과 가 있는 검색 요청 메시지 타입을 정의하려고 한다고 가정해 보겠습니다. 다음은 메시지 타입을 정의하는 데 사용하는 `.proto` 파일입니다.

```protobuf
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

* 파일의 첫 번째 줄은 `proto3` 구문을 사용하고 있음을 지정합니다. 이것을 지정하지 않으면 프로토콜 버퍼 컴파일러는 당신이 [proto2](https://developers.google.com/protocol-buffers/docs/proto)를 사용하고 있다고 가정할 것입니다. 이것은 비어있지 않은 주석이 없는 첫번째 줄이여야합니다.
* `SearchRequest` 메시지 정의는 이 타입의 메시지에 포함하려는 각 데이터 조각에 대해 하나씩 세 개의 필드(이름/값 쌍)를 지정합니다. 각 필드에는 이름과 타입이 있습니다.



### 필드 타입 지정 

위의 예에서 모든 필드는 스칼라 타입입니다: 두 개의 정수(`page_number`와  `result_per_page`) 와 문자열(`query`). 그러나 열거 및 기타 메시지 유형을 포함하여 필드에 대한 복합 타입을 지정할 수도 있습니다.



### 필드 번호 할당

보시다시피 메시지 정의의 각 필드에는 **고유한 번호**가 있습니다. 이 필드 번호는 [메시지 이진 형식 (message binary format)](https://developers.google.com/protocol-buffers/docs/encoding)의 필드를 식별하는 데 사용되며 메시지 타입이 사용 중이면 변경해서는 안 됩니다. 1에서 15까지 범위의 필드 번호는 필드 번호와 필드 타입을 포함하여 인코딩하는 데 1바이트가 소요됩니다([프로토콜 버퍼 인코딩](https://developers.google.com/protocol-buffers/docs/encoding#structure)에서 이에 대한 자세한 내용을 확인할 수 있음). 16에서 2047 사이의 필드 번호는 2바이트를 사용합니다. 따라서 매우 자주 발생하는 메시지 요소에 대해 1에서 15까지의 숫자를 예약해야 합니다. 앞으로 추가될 수 있는 자주 발생하는 요소를 위한 공간을 남겨두는 것을 잊지 마세요.

지정할 수 있는 가장 작은 필드 번호는 1이고 가장 큰 필드 번호는 2<sup>29</sup> - 1 또는 536,870,911입니다. 또한 19000에서 19999까지의 숫자(`FieldDescriptor::kFirstReservedNumber`에서 `FieldDescriptor::kLastReservedNumber`까지)는 프로토콜 버퍼 구현을 위해 예약되어 있으므로 사용할 수 없습니다. `.proto`에서 이러한 예약된 숫자 중 하나를 사용하면 프로토콜 버퍼 컴파일러가 불평할 것입니다. 마찬가지로 이전에 예약된 필드 번호는 사용할 수 없습니다.



### 필드 규칙 지정

메시지 필드는 다음 중 하나일 수 있습니다:

* singular: 올바를 형식의 메시지는 이 필드 중 0개 또는 1개를 가질 수 있습니다(하나 이상은 아님). 그리고 이것은 proto3 구문의 기본 필드 규칙입니다.
* `repeated`: 이 필드는 올바른 형식의 메시지에서 여러 번(0 포함) 반복될 수 있습니다. 반복되는 값의 순서는 유지됩니다.

proto3에서 스칼라 숫자 유형의 `repeated` 필드는 기본적으로 `packed`(압축) 인코딩을 사용합니다.

[프로토콜 버퍼 인코딩](https://developers.google.com/protocol-buffers/docs/encoding#packed)에서 `packed` 인코딩에 대해 자세히 알아볼 수 있습니다.



### 더 많은 메시지 타입 추가

단일 `.proto` 파일에 여러 메시지 타입을 정의할 수 있습니다. 이것은 여러 관련 메시지를 정의하는 경우에 유용합니다. 예를 들어 `SearchResponse` 메시지 타입에 해당하는 응답 메시지 타입을 정의하려는 경우 동일한 `.proto`에 추가할 수 있습니다.

```protobuf
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

message SearchResponse {
 ...
}
```



### 주석 추가

`.proto` 파일에 주석을 추가하려면 C/C++ 스타일 `//` 및 /`* ... */` 구문을 사용하세요.

```protobuf
/* SearchRequest는 응답에 포함할 결과를 나타내는 
 * 페이지 지정 옵션과 함께 검색 쿼리를 나타냅니다. */

message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // 어떤 페이지 번호를 원하세요?
  int32 result_per_page = 3;  // 페이지당 반환할 결과 번호입니다.
}
```



### 예약된 필드

필드를 완전히 제거하거나 주석 처리하여 메시지 타입을 업데이트하면 향후 사용자가 타입을 직접 업데이트할 때 필드 번호를 재사용할 수 있습니다. 이는 나중에 데이터 손상, 개인 정보 버그 등을 포함하여 동일한 `.proto`의 이전 버전을 로드하는 경우 심각한 문제를 일으킬 수 있습니다. 이러한 일이 발생하지 않도록 하는 한 가지 방법은 삭제된 필드의 필드 번호(및/또는 JSON 직렬화 문제를 일으킬 수도 있는 이름)를 `reserved`(예약)하도록 지정하는 것입니다. 프로토콜 버퍼 컴파일러는 향후 사용자가 이러한 필드 식별자를 사용하려고 하면 불평할 것입니다.

```protobuf
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

동일한 예약 문에서 필드 이름과 필드 번호를 혼합할 수 없습니다.



### .proto로 부터 생성된 것은 무엇인가요?

`.proto`로 프로토콜 버퍼 컴파일러를 실행하면 컴파일러는 필드 값 가져오기 및 설정, 출력 스트림에 대한 메시지 직렬화, 입력 스트림으로부터의 메시지 구문 분석을 포함하여 파일에 설명된 메시지 타입으로 작업할 필요가 있는 선택된 언어로 코드를 생성합니다.

* **C++**의 경우 컴파일러는 파일에 설명된 각 메시지 타입에 대한 클래스와 함께 각 `.proto`에서 `.h` 및 `.cc` 파일을 생성합니다.
* **Java**의 경우 컴파일러는 각 메시지 타입에 대한 클래스와 메시지 클래스 인스턴스를 생성하기 위한 특수 빌더 클래스가 있는 `.java` 파일을 생성합니다.
* **Kotlin**의 경우 Java 생성 코드 외에도 컴파일러는 메시지 인스턴스 생성을 단순화하는 데 사용할 수 있는 DSL이 포함된 각 메시지 타입에 대한 **.kt** 파일을 생성합니다.
* **Python**은 조금 다릅니다. – Python 컴파일러는 `.proto`의 각 메시지 타입의 정적 설명자와 함께 모듈을 생성하며, 런타임에 필요한 Python 데이터 액세스 클래스를 만들기 위해 메타클래스와 함께 사용됩니다.
* **Go**의 경우 컴파일러는 파일의 각 메시지 타입에 대한 타입으로 `.pb.go` 파일을 생성합니다.
* **Ruby**의 경우 컴파일러는 메시지 타입이 포함된 Ruby 모듈이 있는 `.rb` 파일을 생성합니다.
* **Objective-C**의 경우 컴파일러는 파일에 설명된 각 메시지 타입에 대한 클래스와 함께 각 `.proto`에서 `pbobjc.h` 및 `pbobjc.m` 파일을 생성합니다.
* **C#**의 경우 컴파일러는 파일에 설명된 각 메시지 타입에 대한 클래스와 함께 각 `.proto`에서 `.cs` 파일을 생성합니다.
* **Dart**의 경우 컴파일러는 파일의 각 메시지 유형에 대한 클래스가 있는 `.pb.dart` 파일을 생성합니다.

선택한 언어에 대한 튜토리얼를 따라 각 언어에 대한 API 사용에 대해 자세히 알아볼 수 있습니다(곧 proto3 버전 제공 예정). API에 대한 자세한 내용은 관련 [API 레퍼런스](https://developers.google.com/protocol-buffers/docs/reference/overview)를 참조하세요.  (proto3 버전도 곧 제공 예정)





## 스칼라 값 타입

스칼라 메시지 필드는 다음 타입 중 하나를 가질 수 있습니다. 아래 표는 `.proto` 파일에 지정된 타입과 자동 생성된 클래스의 해당 타입을 보여줍니다.

| .proto 타입 | 참고                                                         | C++ 타입 | Java/Kotlin 타입<sup>[1]</sup> | Python 타입<sup>[3]</sup>           | Go 타입 | Ruby 타입                      | C# 타입    | PHP 타입                     | Dart 타입 |
| ----------- | ------------------------------------------------------------ | -------- | ------------------------------ | ----------------------------------- | ------- | ------------------------------ | ---------- | ---------------------------- | --------- |
| double      |                                                              | double   | double                         | float                               | float64 | Float                          | double     | float                        | double    |
| float       |                                                              | float    | float                          | float                               | float32 | Float                          | float      | float                        | double    |
| int32       | 가변 길이 인코딩을 사용합니다. 음수 인코딩에 비효율적 - 필드에 음수 값이 있을 가능성이 있는 경우 그 대신 sint32를 사용합니다. | int32    | int                            | int                                 | int32   | Fixnum 또는 필요에 따라 Bignum | int        | integer                      | int       |
| int64       | 가변 길이 인코딩을 사용합니다. 음수 인코딩에 비효율적 - 필드에 음수 값이 있을 가능성이 있는 경우 그 대신 sint64를 사용합니다. | int64    | long                           | int/long<sup>[4]</sup>              | int64   | Bignum                         | long       | integer/string<sup>[6]</sup> | Int64     |
| uint32      | 가변 길이 인코딩을 사용합니다.                               | uint32   | int<sup>[2]</sup>              | int/long<sup>[4]</sup>              | uint32  | Fixnum 또는 필요에 따라 Bignum | uint       | integer                      | int       |
| uint64      | 가변 길이 인코딩을 사용합니다.                               | uint64   | long<sup>[2]</sup>             | int/long<sup>[4]</sup>              | uint64  | Bignum                         | ulong      | integer/string<sup>[6]</sup> | Int64     |
| sint32      | 가변 길이 인코딩을 사용합니다. 부호 있는 정수 값입니다. 이는 일반 int32보다 음수를 더 효율적으로 인코딩합니다. | int32    | int                            | int                                 | int32   | Fixnum 또는 필요에 따라 Bignum | int        | integer                      | int       |
| sint64      | 가변 길이 인코딩을 사용합니다. 부호 있는 정수 값입니다. 일반 int64보다 음수를 더 효율적으로 인코딩합니다. | int64    | long                           | int/long<sup>[4]</sup>              | int64   | Bignum                         | long       | integer/string<sup>[6]</sup> | Int64     |
| fixed32     | 항상 4바이트입니다. 값이 종종 2<sup>28</sup>보다 큰 경우 uint32보다 효율적입니다. | uint32   | int<sup>[2]</sup>              | int/long<sup>[4]</sup>              | uint32  | Fixnum 또는 필요에 따라 Bignum | uint       | integer                      | int       |
| fixed64     | 항상 8바이트입니다. 값이 종종 2<sup>56</sup>보다 큰 경우 uint64보다 효율적입니다. | uint64   | long<sup>[2]</sup>             | int/long<sup>[4]</sup>              | uint64  | Bignum                         | ulong      | integer/string<sup>[6]</sup> | Int64     |
| sfixed32    | 항상 4바이트입니다.                                          | int32    | int                            | int                                 | int32   | Fixnum 또는 필요에 따라 Bignum | int        | integer                      | int       |
| sfixed64    | 항상 8바이트입니다.                                          | int64    | long                           | int/long<sup>[4]</sup>              | int64   | Bignum                         | long       | integer/string<sup>[6]</sup> | Int64     |
| bool        |                                                              | bool     | boolean                        | bool                                | bool    | TrueClass/FalseClass           | bool       | boolean                      | bool      |
| string      | 문자열은 항상 UTF-8 인코딩 또는 7비트 ASCII 텍스트를 포함해야 하며 2<sup>32</sup>보다 길 수 없습니다. | string   | String                         | str/unicode<sup>[5]</sup>           | string  | String (UTF-8)                 | string     | string                       | String    |
| bytes       | 2<sup>32</sup> 이하의 임의의 바이트 시퀀스를 포함할 수 있습니다. | string   | ByteString                     | str (Python 2)<br/>bytes (Python 3) | []byte  | String (ASCII-8BIT)            | ByteString | string                       | List      |

[프로토콜 버퍼 인코딩](https://developers.google.com/protocol-buffers/docs/encoding)에서 메시지를 직렬화할 때 이러한 타입이 인코딩되는 방법에 대해 자세히 알아볼 수 있습니다.

<sup>[1]</sup> Kotlin은 부호 없는(unsigned) 타입의 경우에도 Java의 해당 타입을 사용하여 혼합된 Java/Kotlin 코드베이스에서 호환성을 보장합니다.

<sup>[2]</sup> Java에서 부호 없는 32비트 및 64비트 정수는 부호 있는 정수를 사용하여 표현되며 최상위 비트는 단순히 부호 비트에 저장됩니다.

<sup>[3]</sup> 모든 경우에 값을 필드로 설정하면 타입 검사를 수행하여 유효한지 확인합니다.

<sup>[4]</sup> 64비트 또는 부호 없는 32비트 정수는 디코딩될 때 항상 long으로 표시되지만 필드를 설정할 때 int가 제공되면 int가 될 수 있습니다. 모든 경우에 값은 설정할 때 표시되는 타입에 맞아야 합니다. [2] 참조

<sup>[5]</sup> Python 문자열은 디코딩 시 유니코드로 표시되지만 ASCII 문자열이 제공되면 str이 될 수 있습니다(변경될 수 있음).

<sup>[6]</sup> 64비트 시스템에서는 정수가 사용되고 32비트 시스템에서는 문자열이 사용됩니다.





## 기본 값

메시지가 구문 분석될 때 인코딩된 메시지에 특정 단일 요소가 포함되어 있지 않으면 구문 분석된 객체의 해당 필드가 해당 필드의 기본값으로 설정됩니다. 이러한 기본값은 타입별로 다릅니다.

* 문자열의 경우 기본값은 빈 문자열입니다.
* 바이트의 경우 기본값은 빈 바이트입니다.
* 숫자 타입의 경우 기본값은 0입니다.
* 열거형의 경우 기본값은 **처음으로 정의된 열거형 값**이며 0이어야 합니다.
* 메시지 필드의 경우 필드가 설정되지 않습니다. 정확한 값은 언어에 따라 다릅니다. 자세한 내용은 [생성된 코드 가이드](https://developers.google.com/protocol-buffers/docs/reference/overview)를 참조하세요.

반복(repeated) 필드의 기본값은 비어 있습니다(일반적으로 해당 언어의 빈 목록).

스칼라 메시지 필드의 경우 메시지가 구문 분석되면 필드가 명시적으로 기본값으로 설정되었는지 여부(예: 부울 값이 false로 설정되었는지 여부) 또는 전혀 설정되지 않았는지 여부를 알 수 있는 방법이 없습니다: 메시지 유형을 정의할 때 이것을 염두에 두어야 합니다. 예를 들어, 해당 동작이 기본적으로 발생하지 않도록 하려면 `false`로 설정할 때 일부 동작을 켜는 부울 값을 사용하지 마세요. 또한 스칼라 메시지 필드를 기본값으로 설정하면 값이 와이어에 직렬화되지 않습니다.

생성된 코드에서 기본값이 작동하는 방식에 대한 자세한 내용은 선택한 언어에 대해 [생성된 코드 가이드](https://developers.google.com/protocol-buffers/docs/reference/overview)를 참조하세요.





## 열거

메시지 타입을 정의할 때 해당 필드 중 하나에 미리 정의된 값 목록 중 하나만 포함되도록 할 수 있습니다. 예를 들어, 각 `SearchRequest`에 대해 `corpus` 필드를 추가하려고 합니다. 여기서 corpus는 `UNIVERSAL`, `WEB`, `IMAGES`, `LOCAL`, `NEWS`, `PRODUCTS` 또는 `VIDEO`일 수 있습니다. 가능한 각 값에 대한 상수를 사용하여 메시지 정의에 enum을 추가하면 매우 간단하게 이 작업을 수행할 수 있습니다.

다음 예제에서는 가능한 모든 값과 `Corpus` 유형의 필드가 있는 `Corpus`라는 `enum`을 추가했습니다:

```protobuf
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 4;
}
```

보시다시피, Corpus 열거형의 첫 번째 상수는 0으로 매핑됩니다: 모든 열거형 정의는 첫 번째 요소로 0에 매핑되는 상수를 포함해야 합니다. 다음 때문입니다:

* 숫자 기본값으로 0을 사용할 수 있도록 0 값이 있어야 합니다.
* 첫 번째 열거형 값이 항상 기본값인 [proto2](https://developers.google.com/protocol-buffers/docs/proto) 의미 체계와의 호환성을 위해 0 값은 첫 번째 요소여야 합니다.

다른 열거형 상수에 동일한 값을 할당하여 별칭을 정의할 수 있습니다. 이렇게 하려면 `allow_alias` 옵션을 `true`로 설정해야 합니다. 그렇지 않으면 별칭이 발견될 때 프로토콜 컴파일러가 오류 메시지를 생성합니다.

```protobuf
message MyMessage1 {
  enum EnumAllowingAlias {
    option allow_alias = true;
    UNKNOWN = 0;
    STARTED = 1;
    RUNNING = 1;
  }
}
message MyMessage2 {
  enum EnumNotAllowingAlias {
    UNKNOWN = 0;
    STARTED = 1;
    // RUNNING = 1;  // 이 줄의 주석 처리를 제거하면 Google 내부에서는 컴파일 오류가 발생하고 외부에서는 경고 메시지가 표시됩니다.
  }
}
```

열거자 상수는 32비트 정수 범위에 있어야 합니다. `enum` 값은 와이어에서 [varint 인코딩](https://developers.google.com/protocol-buffers/docs/encoding)을 사용하므로 음수 값은 비효율적이여서 권장하지 않습니다. 위의 예와 같이 메시지 정의 내에서 또는 외부에서 `enum`을 정의할 수 있습니다. 이러한 `enum`들은 .proto 파일의 모든 메시지 정의에서 재사용할 수 있습니다. `_MessageType_._EnumType_` 구문을 사용하여 한 메시지에 선언된 enum 타입을 다른 메시지의 필드 타입으로 사용할 수도 있습니다.

`enum`을 사용하는 `.proto`로 프로토콜 버퍼 컴파일러를 실행하여 생성된 코드는 Java, Kotlin, C++에 해당하는 `enum` 또는 런타임 생성 클래스에서 정수 값을 가진 기호 상수의 집합을 만드는 데 사용되는 파이썬에 대한 특별한 `EnumDescriptor` 클래스를 가질 것입니다.

> **주의** 생성된 코드는 열거자 수에 대한 언어별 제한 사항이 적용될 수 있습니다(한 언어의 경우 수천 개 미만). 사용하려는 언어에 대한 제한 사항을 검토해보세요

역직렬화 중에 인식할 수 없는 열거형 값이 메시지에 보존되지만 메시지가 역직렬화될 때 이 값이 표시되는 방식은 언어에 따라 다릅니다. C++ 및 Go와 같이 지정된 기호 범위를 벗어난 값을 가진 개방형 열거형을 지원하는 언어에서 알 수 없는 열거형 값은 기본 정수 표현으로 단순히 저장됩니다. Java와 같은 폐쇄형 열거형을 사용하는 언어에서 열거형의 경우는 인식할 수 없는 값을 나타내는 데 사용되며 기본 정수는 특수 접근자를 사용하여 액세스할 수 있습니다. 두 경우 모두 메시지가 직렬화되면 인식할 수 없는 값이 메시지와 함께 계속 직렬화됩니다.

애플리케이션에서 메시지 `enum`으로 작업하는 방법에 대한 자세한 내용은 선택한 언어에 대해 [생성된 코드 가이드](https://developers.google.com/protocol-buffers/docs/reference/overview)를 참조하세요.



#### 예약된 값

열거형 항목을 완전히 제거하거나 주석 처리하여 열거형 유형을 업데이트하는 경우 미래의 사용자는 유형을 직접 업데이트할 때 숫자 값을 재사용할 수 있습니다. 이는 나중에 데이터 손상, 개인 정보 버그 등을 포함하여 동일한 `.proto`의 이전 버전을 로드하는 경우 심각한 문제를 일으킬 수 있습니다. 이러한 일이 발생하지 않도록 하는 한 가지 방법은 삭제된 항목의 숫자 값(및/또는 JSON 직렬화 문제를 일으킬 수도 있는 이름)을 예약하도록(`reserved`) 지정하는 것입니다. 프로토콜 버퍼 컴파일러는 미래의 사용자가 이러한 식별자를 사용하려고 하면 불평할 것입니다. `max` 키워드를 사용하여 예약된 숫자 값 범위가 가능한 최대 값까지 올라가도록 지정할 수 있습니다.

```protobuf
enum Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved "FOO", "BAR";
}
```

동일한 `reserved` 문에서 필드 이름과 숫자 값을 혼합할 수 없습니다.





## 다른 메시지 타입 사용

다른 메시지 타입을 필드 타입으로 사용할 수 있습니다. 예를 들어 각 `SearchResponse` 메시지에 `Result` 메시지를 포함하고 싶다고 가정해 보겠습니다. 이렇게 하려면 동일한 `.proto`에 `Result` 메시지 타입을 정의한 다음 `SearchResponse`에 `Result` 타입 필드를 지정할 수 있습니다.

```protobuf
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

### 정의 가져오기

위의 예에서 `Result` 메시지 유형은 `SearchResponse`와 동일한 파일에 정의됩니다. - 필드 유형으로 사용하려는 메시지 유형이 이미 다른 `.proto` 파일에 정의되어 있으면 어떻게 하나요?

다른 `.proto` 파일을 가져와서 정의를 사용할 수 있습니다. 다른 `.proto`의 정의를 가져오려면 파일 맨 위에 import 문을 추가합니다.

```protobuf
import "myproject/other_protos.proto";
```

기본적으로 직접 가져온 `.proto` 파일의 정의만 사용할 수 있습니다. 그러나 때로는 `.proto` 파일을 새 위치로 이동해야 할 수도 있습니다. `.proto` 파일을 직접 이동하고 모든 호출 장소를 한 번에 업데이트하는 대신 이전 위치에 플레이스 홀더 .proto 파일을 배치하여 `import public` 개념을 사용하여 모든 가져오기를 새 위치로 전달할 수 있습니다.

**참고로 Java에서는 public import 기능을 사용할 수 없습니다.**

`import public` 의존성은 `import public` 문을 포함하는 proto를 가져오는 모든 코드에 의해 전이적으로 의존할 수 있습니다. 예를 들어:

```protobuf
// new.proto
// 모든 정의가 여기로 이동되었습니다.
```

```protobuf
// old.proto
// 이것은 모든 클라이언트가 import하는 proto입니다.
import public "new.proto";
import "other.proto";
```

```protobuf
// client.proto
import "old.proto";
// old.proto와 new.proto의 정의를 사용하지만 other.proto는 사용하지 않습니다.
```

프로토콜 컴파일러는 `-I`/`--proto_path` 플래그를 사용하여 프로토콜 컴파일러 명령줄에 지정된 디렉터리 집합에서 가져온 파일을 검색합니다. 플래그가 지정되지 않은 경우 컴파일러가 호출된 디렉토리를 찾습니다. 일반적으로 `--proto_path`플래그를 프로젝트의 루트로 설정하고 모든 가져오기에 대해 [정규화된 이름](https://en.wikipedia.org/wiki/Fully_qualified_name)을 사용해야 합니다.

### proto2 메시지 타입 사용

[proto2](https://developers.google.com/protocol-buffers/docs/proto) 메시지 타입을 가져와서 proto3 메시지에서 사용하거나 그 반대의 경우도 가능합니다. 그러나 proto2 열거형은 proto3 구문에서 직접 사용할 수 없습니다(가져온 proto2 메시지에서는 사용해도 괜찮음).





## 중첩 타입

다음 예와 같이 다른 메시지 타입 내에서 메시지 타입을 정의하고 사용할 수 있습니다. – 여기에서 `Result` 메시지는 `SearchResponse` 메시지 내부에 정의됩니다:

```protobuf
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

이 메시지 타입을 상위 메시지 타입 외부에서 재사용하려면 `_Parent_._Type_`으로 참조하세요.

```protobuf
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```

원하는 만큼 메시지를 중첩할 수 있습니다.

```protobuf
message Outer {                  // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```





## 메시지 타입 업데이트

기존 메시지 유형이 더 이상 모든 요구 사항을 충족하지 못하는 경우  – 예를 들어, 메시지 형식에 추가 필드가 있어야 함 -  이전 형식으로 생성된 코드를 계속 사용하고 싶음, 그러나 걱정하지 마세요! 기존 코드를 손상시키지 않고 메시지 타입을 업데이트하는 것은 매우 간단합니다. 다음 규칙만 기억하세요:

* 기존 필드의 필드 번호를 변경하지 마세요.
* 새 필드를 추가하면 "이전" 메시지 형식을 사용하여 코드로 직렬화된 모든 메시지를 새로 생성된 코드로 구문 분석할 수 있습니다. 새 코드가 이전 코드에서 생성된 메시지와 적절하게 상호 작용할 수 있도록 이러한 요소의 기본값을 염두에 두어야 합니다. 마찬가지로 새 코드로 생성된 메시지는 이전 코드로 구문 분석할 수 있습니다: 이전 바이너리는 구문 분석할 때 새 필드를 무시합니다. 자세한 내용은 Unknown 필드 섹션을 참조하세요.
* 필드 번호가 업데이트된 메시지 타입에서 다시 사용되지 않는 한 필드를 제거할 수 있습니다. "OBSOLETE_" 접두사를 추가하여 필드 이름을 바꾸거나 필드 번호를 예약(reserved)하여 `.proto`파일의 향후 사용자가 실수로 번호를 재사용할 수 없도록 할 수 있습니다.
* `int32`, `uint32`, `int64`, `uint64` 및 `bool`은 모두 호환됩니다. – 이는 앞으로 또는 이전 버전과의 호환성을 손상시키지 않고 이러한 타입 중 하나에서 다른 타입으로 필드를 변경할 수 있음을 의미합니다. 숫자가 해당 유형에 맞지 않는 와이어에서 구문 분석되면 C++에서 해당 유형으로 숫자를 캐스팅한 것과 동일한 효과를 얻을 수 있습니다(예: 64비트 숫자가 int32로 읽히는 경우, 32비트로 잘립니다.)
* `sint32` 및 `sint64`는 서로 호환되지만 다른 정수 유형과 호환되지 않습니다.
* `string`과 `byte`는 byte가 유효한 UTF-8이면 호환됩니다.
* byte에 인코딩된 버전의 메시지가 포함된 경우 포함된 메시지는 `byte`와 호환됩니다.
* `fixed32`는 `sfixed32`와 호환되고 `fixed64`는 `sfixed64`와 호환됩니다.
* `string`, `byte` 및 message 필드의 경우 `optional`은 `repeated`와 호환됩니다.
* repeated 필드의 직렬화된 데이터가 입력으로 주어지면 이 필드가 `optional` 일 것으로 예상하는 클라이언트는 기본 타입 필드인 경우 마지막 입력 값을 사용하거나 message 타입 필드인 경우 모든 입력 요소를 병합합니다. 이것은 일반적으로 bool 및 enum을 포함한 숫자 유형에 안전하지 않습니다. 숫자 유형의 repeated 필드는 압축 형식으로 직렬화될 수 있으며, 이는 `optional` 필드가 예상되는 경우 올바르게 구문 분석되지 않습니다.
* `enum`은 와이어 형식 측면에서 `int32`, `uint32`, `int64` 및 `uint64`와 호환됩니다(값이 맞지 않으면 잘림에 유의). 그러나 클라이언트 코드는 메시지가 역직렬화될 때 이를 다르게 처리할 수 있습니다. 예를 들어, 인식할 수 없는 proto3 `enum` 타입은 메시지에 보존되지만 메시지가 역직렬화될 때 이것이 표현되는 방식은 언어에 따라 다릅니다. Int 필드는 항상 값을 유지합니다.
* 단일 값을 **새로운** `oneof`의 구성원으로 변경하는 것은 안전하고 바이너리 호환이 가능합니다. 코드가 한 번에 둘 이상을 설정하지 않는다고 확신하는 경우 여러 필드를 새로운 `oneof`로 이동하는 것이 안전할 수 있습니다.



## Unknown 필드

Unknown 필드는 파서가 인식하지 못하는 필드를 나타내는 잘 구성된 프로토콜 버퍼 직렬화된 데이터입니다. 예를 들어, 이전 바이너리가 새 바이너리에서 보낸 데이터를 새 필드로 구문 분석할 때 새 필드는 이전 바이너리에서 알 수 없는 필드가 됩니다.

원래 proto3 메시지는 구문 분석 중에 항상 Unknown 필드를 버렸지만 버전 3.5에서는 proto2 동작과 일치하도록 알 수 없는 필드의 보존을 다시 도입했습니다. 버전 3.5 이상에서 Unknown  필드는 구문 분석 중에 유지되고 직렬화된 출력에 포함됩니다.



## Any

`Any` 메시지 유형을 사용하면 .proto 정의 없이 메시지를 포함된 유형으로 사용할 수 있습니다. `Any`에는 `bytes`로 직렬화된 임의의 메시지와 해당 메시지 유형에 대해 전역적으로 고유한 식별자 역할을 하고 해당 메시지 유형으로 확인되는 URL이 포함됩니다. `Any` 유형을 사용하려면 `google/protobuf/any.proto`를 import 해야 합니다.

```protobuf
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

지정된 메시지 타입의 기본 타입 URL은 `type.googleapis.com/_packagename_._messagename_`입니다. 다른 언어 구현은 런타임 라이브러리 도우미를 지원하여 타입이 안전한 방식으로 모든 값을 압축 및 압축 해제합니다. – 예를 들어, Java에서 Any 유형에는 특별한 `pack()` 및 `unpack()` 접근자가 있는 반면 C++에는 `PackFrom()` 및 `UnpackTo()` 메서드가 있습니다.

```c++
// 임의의 메시지 유형을 Any에 저장합니다.
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// Any에서 임의의 메시지 읽기.
ErrorStatus status = ...;
for (const Any& detail : status.details()) {
  if (detail.Is<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

**현재 `Any` 타입으로 작업하기 위한 런타임 라이브러리가 개발 중에 있습니다.**

[proto2 구문](https://developers.google.com/protocol-buffers/docs/proto)에 이미 익숙하다면 Any는 [확장](https://developers.google.com/protocol-buffers/docs/proto#extensions)을 허용할 수 있는 proto2 메시지와 유사하게 임의의 proto3 메시지를 보유할 수 있습니다.





## Oneof

많은 필드가 있고 동시에 최대 하나의 필드가 설정되는 메시지가 있는 경우 oneof 기능을 사용하여 이 동작을 적용하여 메모리를 절약할 수 있습니다.

Oneof 필드는 oneof 공유 메모리의 모든 필드를 제외하고 일반 필드와 같으며 최대 한 필드를 동시에 설정할 수 있습니다. oneof의 멤버를 설정하면 다른 모든 멤버가 자동으로 지워집니다. 선택한 언어에 따라 특별한 `case()` 또는 `WhereOneof()` 메서드를 사용하여 oneof의 어떤 값이 설정되어 있는지 확인(있는 경우)할 수 있습니다.

### Oneof 사용

`.proto`에서 oneof를 정의하려면 `oneof` 키워드 뒤에 oneof 이름을 사용합니다. 이 경우 `test_oneof`:

```protobuf
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

그런 다음 oneof 정의에 oneof 필드를 추가합니다. `map` 필드 및 `repeated` 필드를 제외한 모든 타입의 필드를 추가할 수 있습니다.

생성된 코드에서 필드 중 하나에는 일반 필드와 동일한 getter 및 setter가 있습니다. 또한 oneof에서 어떤 값(있는 경우)이 설정되어 있는지 확인하는 특별한 방법을 사용할 수도 있습니다. 관련 [API 레퍼런스](https://developers.google.com/protocol-buffers/docs/reference/overview)에서 선택한 언어의 oneof API에 대해 자세히 알아볼 수 있습니다.



### Oneof 기능

* oneof 필드를 설정하면 oneof의 다른 모든 멤버들이 자동으로 지워집니다. 따라서 필드 중 하나를 여러 개 설정하면 마지막으로 설정한 필드에만 값이 유지됩니다.

  ```c++
  SampleMessage message;
  message.set_name("name");
  CHECK(message.has_name());
  message.mutable_sub_message();   // name 필드를 지웁니다.
  CHECK(!message.has_name());
  ```

* 파서가 와이어에서 동일한 oneof의 여러 멤버를 만나면 마지막으로 본 멤버만 구문 분석된 메시지에 사용됩니다.

* `oneof` 는 반복(`repeated`) 할 수 없습니다.

* 리플렉션 API는 oneof 필드에서 작동합니다.

* oneof 필드를 기본값으로 설정하면(예: int32 oneof 필드를 0으로 설정) 해당 oneof 필드의 "case"가 설정되고 값이 와이어에 직렬화됩니다.

* C++를 사용하는 경우 코드로 인해 메모리 충돌이 발생하지 않는지 확인하세요. `set_name()` 메서드를 호출하여 `sub_message`가 이미 삭제되었기 때문에 다음 샘플 코드가 충돌합니다.

  ```c++
  SampleMessage msg1;
  msg1.set_name("name");
  SampleMessage msg2;
  msg2.mutable_sub_message();
  msg1.swap(&msg2);
  CHECK(msg1.has_sub_message());
  CHECK(msg2.has_name());
  ```

  

### 이전 버전과의 호환성 문제

필드 중 하나를 추가하거나 제거할 때 주의하세요. oneof의 값을 검사할 때 `None`/`NOT_SET`이 반환되면 oneof가 설정되지 않았거나 oneof의 다른 버전에 있는 필드로 설정되었음을 의미할 수 있습니다. 와이어의 Unknown 필드가 oneof의 구성원인지 여부를 알 수 있는 방법이 없기 때문에 차이점을 알 수 있는 방법이 없습니다.

**태그 재사용 문제**

* **필드를 `oneof` 안쪽 또는 밖으로 이동**: 메시지가 직렬화되고 구문 분석된 후 일부 정보가 손실될 수 있습니다(일부 필드는 지워짐).  그러나 단일 필드를 새 `oneof`로 안전하게 이동할 수 있으며 한 필드만 설정된 것으로 알려진 경우 여러 필드를 이동할 수 있습니다. 
* **`oneof` 필드를 삭제하고 다시 추가**: 이렇게 하면 메시지가 직렬화되고 구문 분석된 후 현재 설정된 oneof 필드가 지워질 수 있습니다.
* **`oneof` 분할 또는 병합:** 이것은 일반 필드를 이동하는 것과 유사한 문제가 있습니다.





## 맵

데이터 정의의 일부로 연관 맵을 생성하려는 경우 프로토콜 버퍼는 편리한 바로 가기 구문을 제공합니다/ㅣ

```protobuf
map<key_type, value_type> map_field = N;
```

...여기서 `key_type`은 정수 또는 문자열 타입이 될 수 있습니다(따라서 부동 소수점 타입 및 `byte`를 제외한 모든 스칼라 유형). 열거형은 유효한 `key_type`이 아닙니다. `value_type`은 다른 맵을 제외한 모든 타입이 될 수 있습니다.

예를 들어 각 `Project` 메시지가 문자열 키와 연결된 project 맵을 작성하려는 경우 다음과 같이 정의할 수 있습니다.

```protobuf
map<string, Project> projects = 3;
```

* 맵 필드는 반복(`repeated`)할 수 없습니다.
* 맵 값의 와이어 형식 순서 및 맵 반복 순서는 정의되지 않았으므로 맵 항목이 특정 순서로 되어 있다고 기대할 수 없습니다.
* `.proto`에 대한 텍스트 형식을 생성할 때 맵은 키로 정렬됩니다. 숫자 키는 숫자로 정렬됩니다.
* 와이어에서 구문 분석하거나 병합할 때 중복 맵 키가 있는 경우 마지막으로 본 키가 사용됩니다. 텍스트 형식에서 맵을 구문 분석할 때 중복 키가 있는 경우 구문 분석이 실패할 수 있습니다.
* 키를 제공하지만 맵 필드에 값을 제공하지 않는 경우 필드가 직렬화될 때의 동작은 언어에 따라 다릅니다. C++, Java, Kotlin 및 Python에서 타입의 기본값은 직렬화되지만 다른 언어에서는 아무 것도 직렬화되지 않습니다.

생성된 맵 API는 현재 모든 proto3 지원 언어에서 사용할 수 있습니다. 관련 [API 레퍼런스](https://developers.google.com/protocol-buffers/docs/reference/overview)에서 선택한 언어의 맵 API에 대해 자세히 알아볼 수 있습니다.



### 이전 버전과의 호환성

맵 구문은 와이어상의 다음과 동일하므로 맵을 지원하지 않는 프로토콜 버퍼 구현은 여전히 데이터를 처리할 수 있습니다.

```protobuf
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

맵을 지원하는 모든 프로토콜 버퍼 구현은 위의 정의에서 허용할 수 있는 데이터를 생성하고 허용해야 합니다.





## 패키지

프로토콜 메시지 타입 간의 이름 충돌을 방지하기 위해 선택적 패키지 지정자를 `.proto` 파일에 추가할 수 있습니다.

```protobuf
package foo.bar;
message Open { ... }
```

그런 다음 메시지 타입의 필드를 정의할 때 패키지 지정자를 사용할 수 있습니다.

```protobuf
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

패키지 지정자가 생성된 코드에 영향을 미치는 방식은 선택한 언어에 따라 다릅니다.

* **C++**에서 생성된 클래스는 C++ 네임스페이스 내부에 래핑됩니다. 예를 들어, Open은 foo::bar 네임스페이스에 있습니다.
* **Java** 및 **Kotlin**에서는 `.proto` 파일에 `java_package` 옵션을 명시적으로 제공하지 않는 한 패키지가 Java 패키지로 사용됩니다.
* **Python**에서 패키지 지시문은 무시됩니다. Python 모듈은 파일 시스템의 위치에 따라 구성되기 때문입니다.
* **Go**에서 패키지는 `.proto` 파일에 `go_package` 옵션을 명시적으로 제공하지 않는 한 Go 패키지 이름으로 사용됩니다.
* **Ruby**에서 생성된 클래스는 중첩된 Ruby 네임스페이스 안에 래핑되어 필요한 Ruby 대문자 스타일로 변환됩니다(첫 글자는 대문자로, 첫 글자가 글자가 아닌 경우 `PB_`가 앞에 추가됨). 예를 들어, `Open`은 `Foo::Bar` 네임스페이스에 있습니다.
* **C#**에서는 `.proto` 파일에 `csharp_namespace` 옵션을 명시적으로 제공하지 않는 한 PascalCase로 변환한 후 패키지가 네임스페이스로 사용됩니다. 예를 들어 `Open`은 `Foo.Bar` 네임스페이스에 있습니다.



### 패키지 및 이름 확인(Resolution)

프로토콜 버퍼 언어의 타입 이름 확인은 C++처럼 작동합니다: 먼저 가장 안쪽 범위가 검색되고 다음으로 가장 안쪽 범위가 검색되는 방식으로 각 패키지가 상위 패키지의 "내부"로 간주됩니다. 선행 '.'는 (예: `.foo.bar.Baz`) 대신 가장 바깥쪽 범위에서 시작하는 것을 의미합니다. 

프로토콜 버퍼 컴파일러는 가져온 .proto 파일을 구문 분석하여 모든 타입 이름을 확인합니다. 각 언어의 코드 생성기는 범위 지정 규칙이 다른 경우에도 해당 언어의 각 타입을 참조하는 방법을 알고 있습니다.





## 서비스 정의

RPC(원격 프로시저 호출) 시스템에서 메시지 타입을 사용하려는 경우 `.proto` 파일에 RPC 서비스 인터페이스를 정의하면 프로토콜 버퍼 컴파일러가 선택한 언어로 서비스 인터페이스 코드와 스텁을 생성합니다. 예를 들어, `SearchRequest`를 받고 `SearchResponse`를 반환하는 메서드로 RPC 서비스를 정의하려는 경우 다음과 같이 `.proto` 파일에 정의할 수 있습니다.

```protobuf
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

프로토콜 버퍼와 함께 사용하는 가장 간단한 RPC 시스템은 [gRPC](https://grpc.io/)입니다: Google에서 개발한 언어이고 플랫폼 중립적인 오픈 소스 RPC 시스템입니다. gRPC는 특히 프로토콜 버퍼와 잘 작동하며 특별한 프로토콜 버퍼 컴파일러 플러그인을 사용하여 `.proto` 파일에서 직접 관련 RPC 코드를 생성할 수 있습니다.

gRPC를 사용하지 않으려면 자체 RPC 구현과 함께 프로토콜 버퍼를 사용할 수도 있습니다. 이에 대한 자세한 내용은 [Proto2 언어 가이드](https://developers.google.com/protocol-buffers/docs/proto#services)에서 확인할 수 있습니다.

또한 프로토콜 버퍼용 RPC 구현을 개발하기 위해 진행 중인 여러 타사 프로젝트가 있습니다. 우리가 알고 있는 프로젝트에 대한 링크 목록은 [타사 애드온 위키 페이지](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md)를 참조하세요.





## JSON 매핑

Proto3는 JSON의 표준 인코딩을 지원하므로 시스템 간에 데이터를 더 쉽게 공유할 수 있습니다. 인코딩은 아래 표에 타입별로 설명되어 있습니다.

JSON으로 인코딩된 데이터에 값이 없거나 값이 null인 경우 프로토콜 버퍼로 구문 분석할 때 적절한 기본값으로 해석됩니다. 필드에 프로토콜 버퍼의 기본값이 있는 경우 공간을 절약하기 위해 기본적으로 JSON 인코딩 데이터에서 필드가 생략됩니다. 구현은 JSON 인코딩 출력에서 기본값이 있는 필드를 내보내는 옵션을 제공할 수 있습니다.

| proto3                 | JSON          | JSON 예제                               | 참고                                                         |
| ---------------------- | ------------- | --------------------------------------- | ------------------------------------------------------------ |
| message                | object        | {"fooBar": v, "g": null, …}             | JSON 객체를 생성합니다. 메시지 필드 이름은 lowerCamelCase로 매핑되어 JSON 객체 키가 됩니다. json_name 필드 옵션이 지정되면 지정된 값이 대신 키로 사용됩니다. 파서는 lowerCamelCase 이름(또는 json_name 옵션으로 지정된 이름)과 원래 proto 필드 이름을 모두 허용합니다. `null`은 모든 필드 유형에 대해 허용되는 값이며 해당 필드 유형의 기본값으로 처리됩니다. |
| enum                   | string        | "FOO_BAR"                               | proto에 지정된 열거형 값의 이름이 사용됩니다. 파서는 열거형 이름과 정수 값을 모두 허용합니다. |
| map<K,V>               | object        | {"k": v, …}                             | 모든 키는 문자열로 변환됩니다.                               |
| repeated V             | array         | [v, …]                                  | `null`은 빈 목록 []으로 허용됩니다.                          |
| bool                   | true, false   | true, false                             |                                                              |
| string                 | string        | "Hello World!"                          |                                                              |
| bytes                  | base64 string | "YWJjMTIzIT8kKiYoKSctPUB+"              | JSON 값은 패딩이 있는 표준 base64 인코딩을 사용하여 문자열로 인코딩된 데이터입니다. 패딩이 있거나 없는 표준 또는 URL-safe base64 인코딩이 허용됩니다. |
| int32, fixed32, uint32 | number        | 1, -10, 0                               | JSON 값은 십진수입니다. 숫자나 문자열이 허용됩니다.          |
| int64, fixed64, uint64 | string        | "1", "-10"                              | JSON 값은 10진수 문자열입니다. 숫자나 문자열이 허용됩니다.   |
| float, double          | number        | 1.1, -10.0, 0, "NaN", "Infinity"        | JSON 값은 숫자 또는 특수 문자열 값 "NaN", "Infinity" 및 "-Infinity" 중 하나입니다. 숫자나 문자열이 허용됩니다. 지수 표기법도 허용됩니다. -0은 0과 동일한 것으로 간주됩니다. |
| Any                    | object        | {"@type": "url", "f": v, … }            | Any에 특별한 JSON 매핑이 있는 값이 포함되어 있으면 {"@type": xxx, "value": yyy}와 같이 변환됩니다. 그렇지 않으면 값이 JSON 객체로 변환되고 "@type" 필드가 삽입되어 실제 데이터 유형을 나타냅니다. |
| Timestamp              | string        | "1972-01-01T10:00:20.021Z"              | 생성된 출력은 항상 Z-정규화되고 0, 3, 6 또는 9개의 소수 자릿수를 사용하는 RFC 3339를 사용합니다. "Z" 이외의 오프셋도 허용됩니다. |
| Duration               | string        | "1.000340012s", "1s"                    | 생성된 출력은 필요한 정밀도에 따라 항상 0, 3, 6 또는 9개의 소수 자릿수와 접미사 "s"를 포함합니다. 나노초 정밀도에 맞고 접미사 "s"가 필요한 한 모든 소수 자릿수(없음)도 허용됩니다. |
| Struct                 | object        | { … }                                   | 모든 JSON 개체. `struct.proto`를 참조해보세요.               |
| Wrapper types          | 다양한 타입들 | 2, "2", "foo", true, "true", null, 0, … | 래퍼는 데이터 변환 및 전송 중에 null이 허용되고 보존된다는 점을 제외하고 JSON에서 래핑된 기본 유형과 동일한 표현을 사용합니다. |
| FieldMask              | string        | "f.fooBar,h"                            | `field_mask.proto`를 참조해보세요.                           |
| ListValue              | array         | [foo, bar, …]                           |                                                              |
| Value                  | value         |                                         | 모든 JSON 값. 자세한 내용은 `google.protobuf.Value`를 확인하세요. |
| NullValue              | null          |                                         | JSON null                                                    |
| Empty                  | object        | {}                                      | 빈 JSON 객체                                                 |



### JSON 옵션

proto3 JSON 구현은 다음 옵션을 제공할 수 있습니다:

* **기본값이 있는 필드 내보내기**: 기본값이 있는 필드는 proto3 JSON 출력에서 기본적으로 생략됩니다. 구현은 이 동작과 출력 필드를 기본값으로 재정의하는 옵션을 제공할 수 있습니다.
* **unknown 필드 무시**: Proto3 JSON 파서는 기본적으로 unknown 필드를 거부해야 하지만 구문 분석에서 unknown 필드를 무시하는 옵션을 제공할 수 있습니다.
* **lowerCamelCase 이름 대신 proto 필드 이름 사용**: 기본적으로 proto3 JSON 프린터는 필드 이름을 lowerCamelCase로 변환하고 이를 JSON 이름으로 사용해야 합니다. 구현은 대신 proto 필드 이름을 JSON 이름으로 사용하는 옵션을 제공할 수 있습니다. 변환된 lowerCamelCase 이름과 proto 필드 이름을 모두 수락하려면 Proto3 JSON 파서가 필요합니다.
* **열거형 값을 문자열 대신 정수로 내보내기**: 열거형 값의 이름은 기본적으로 JSON 출력에서 사용됩니다. enum 값의 숫자 값을 대신 사용하는 옵션이 제공될 수 있습니다.





## 옵션





## 클래스 생성하기







---

## 의견

* 예전에 proto3버전인지 2버전을 봤었는지 정확하게 기억이 안난다.,  그런데 Java 튜토리얼을 보니 proto2 기준 같다.

* placeholder : 빠져 있는 다른 것을 대신하는 기호나 텍스트의 일부
* 그런데 wire가 뭐지?

