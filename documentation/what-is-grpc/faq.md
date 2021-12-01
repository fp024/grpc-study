# FAQ

> 원문: https://grpc.io/docs/what-is-grpc/faq/

### 내용

* gRPC란 무엇인가요?
* gRPC는 무엇을 의미하나요?
* 왜 gRPC를 사용해야 하지요?
* 누가 왜 이것을 사용나요?
* 어떤 프로그래밍 언어가 지원나요?
* gRPC 사용을 시작하려면 어떻게 해야 하나요?
* gRPC에는 어떤 라이선스가 적용되나요?
* 어떻게 기여할 수 있나요?
* 문서는 어디에 있나요?
* 로드맵은 무엇인가요?
* gRPC 릴리스는 얼마나 오래 지원되나요?
* gRPC 버전 관리 정책이란 무엇인가요?
* 최신 gRPC 버전은 무엇인가요?
* gRPC 릴리스는 언제 발생하나요?
* gRPC의 보안 취약점을 보고하려면 어떻게 해야 하나요?
* 브라우저에서 사용할 수 있나요?
* 내가 좋아하는 데이터 형식(JSON, Protobuf, Thrift, XML)으로 gRPC를 사용할 수 있을까요?
* 서비스 메시에서 gRPC를 사용할 수 있을까요?
* gRPC는 모바일 애플리케이션 개발에 어떻게 도움이 되나요?
* gRPC가 HTTP/2를 통한 바이너리 Blob보다 나은 이유는 무엇가요?
* gRPC가 REST보다 나은 / 나쁜 이유는 무엇이 있나요?
* gRPC를 어떻게 발음하나요?

다음은 자주 묻는 질문입니다. 여기에서 답을 찾아보시길 바랍니다 :-)



## gRPC란 무엇인가요?

gRPC는 어디에서나 실행할 수 있는 최신 오픈 소스 RPC(원격 프로시저 호출) 프레임워크입니다. 클라이언트와 서버 응용 프로그램이 투명하게 통신할 수 있게 하고 연결된 시스템을 더 쉽게 구축할 수 있도록 합니다.

우리가 왜 gRPC를 만들었는지에 대한 배경에 대해서는 더 긴 [동기부여와 디자인 원칙](https://grpc.io/blog/principles/) 포스트를 읽어보세요.



## gRPC는 무엇을 의미하나요?

물론 **g**RPC **R**emote **P**rocedure **C**all입니다!



## 왜 gRPC를 사용해야 하지요?

주요 사용 시나리오:

* 짧은 대기 시간, 확장성이 뛰어난 분산 시스템.
* 클라우드 서버와 통신하는 모바일 클라이언트를 개발합니다.
* 정확하고 효율적이며 언어 독립적이어야 하는 새로운 프로토콜을 설계합니다.
* 확장을 가능하게 하는 층이 있는(Layered) 디자인. 인증, 로드 밸런싱, 로깅 및 모니터링 등



## 누가 왜? 이것을 사용나요?

gRPC는 [Cloud Native Computing Foundation](https://cncf.io/) (CNCF) 프로젝트입니다.

Google은 오랫동안 gRPC에서 많은 기본 기술과 개념을 사용해 왔습니다. 현재 구현은 Google의 여러 클라우드 제품과 Google 외부 API에서 사용되고 있습니다. 또한 [Square](https://corner.squareup.com/2015/02/grpc.html), [Netflix](https://github.com/Netflix/ribbon), [CoreOS](https://blog.gopheracademy.com/advent-2015/etcd-distributed-key-value-store-with-grpc-http2/), [Docker](https://blog.docker.com/2015/12/containerd-daemon-to-control-runc/), [CockroachDB](https://github.com/cockroachdb/cockroach), [Cisco](https://github.com/CiscoDevNet/grpc-getting-started), [Juniper Networks](https://github.com/Juniper/open-nti) 및 기타 여러 조직과 개인이 사용하고 있습니다. 



## 어떤 프로그래밍 언어가 지원나요?

[공식적으로 지원되는 언어 및 플랫폼](../)을 참조해보세요.



## gRPC 사용을 시작하려면 어떻게 해야 하나요?

[여기](../quick-start.md)의 지침에 따라 gRPC 설치를 시작할 수 있습니다.



## gRPC에는 어떤 라이선스가 적용되나요?

모든 구현은 [Apache 2.0](https://github.com/grpc/grpc/blob/master/LICENSE)에 따라 라이선스가 부여됩니다.



## 어떻게 기여할 수 있나요?

[기여자](https://grpc.io/community/#contribute)는 매우 환영하며 저장소는 GitHub에서 호스팅됩니다. 커뮤니티 피드백, 추가 사항 및 버그를 기다리겠습니다. 개인 기여자와 기업 기여자 모두 CLA에 서명해야 합니다. gRPC와 관련된 프로젝트에 대한 아이디어가 있으면 [여기](https://github.com/grpc/grpc-contrib/blob/master/CONTRIBUTING.md)에서 지침을 읽고 제출하세요. GitHub의 [gRPC 에코시스템](https://github.com/grpc-ecosystem) 조직 아래 프로젝트 목록이 점점 늘어나고 있습니다.



## 문서는 어디에 있나요?

여기 grpc.io에서 [문서](../)를 확인해보세요.



## 로드맵은 무엇인가요?

gRPC 프로젝트에는 RFC 프로세스가 있으며 이를 통해 구현을 위해 새로운 기능을 설계하고 승인합니다. [이 저장소](https://github.com/grpc/proposal)에서 추적됩니다.



## gRPC 릴리스는 얼마나 오래 지원되나요?

gRPC 프로젝트는 LTS 릴리스를 수행하지 않습니다. 위의 롤링 릴리스 모델을 고려하여 현재, 최신 릴리스 및 그 이전 릴리스를 지원합니다. 여기서 지원은 버그 수정 및 보안 수정을 의미합니다.



## gRPC 버전 관리 정책이란 무엇인가요?

[여기](https://github.com/grpc/grpc/blob/master/doc/versioning.md)에서 gRPC 버전 관리 정책을 참조하세요.



## 최신 gRPC 버전은 무엇인가요?

최신 릴리스 태그는 v1.41.0입니다. (2021/12/01 기준)



## gRPC 릴리스는 언제 발생하나요?

gRPC 프로젝트는 마스터 브랜치의 끝이 항상 안정적인 모델에서 작동합니다. 프로젝트(다양한 런타임에 걸쳐)는 최선의 노력으로 6주마다 체크포인트 릴리스를 제공하는 것을 목표로 합니다. [여기](https://github.com/grpc/grpc/blob/master/doc/grpc_release_schedule.md)에서 출시 일정을 참조하세요.



## gRPC의 보안 취약점을 보고하려면 어떻게 해야 하나요?

gRPC의 보안 취약점을 보고하려면 [여기](https://github.com/grpc/proposal/blob/master/P4-grpc-cve-process.md)에 설명된 프로세스를 따르세요.



## 브라우저에서 사용할 수 있나요?

[gRPC-Web](https://github.com/grpc/grpc-web) 프로젝트는 일반적으로 사용 가능(Generally Available)합니다.



## 내가 좋아하는 데이터 형식(JSON, Protobuf, Thrift, XML)으로 gRPC를 사용할 수 있나요?

예. gRPC는 여러 콘텐츠 유형을 지원하도록 확장 가능하도록 설계되었습니다. 초기 릴리스에는 Protobuf에 대한 지원과  FlatBuffer 및 Thrift와 같은 다른 콘텐츠 유형에 대한 외부 지원이 포함되며, 다양한 수준의 완성도를 가지고 있습니다.



## 서비스 메시에서 gRPC를 사용할 수 있을까요?

예. gRPC 애플리케이션은 다른 애플리케이션과 마찬가지로 서비스 메시에 배포할 수 있습니다. gRPC는 사이드카(sidecar) 프록시 없이 서비스 메시에 gRPC 애플리케이션을 배포할 수 있는 [xDS API](https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol)도 지원합니다. gRPC에서 지원되는 프록시리스 서비스 메시 기능이 [여기](https://github.com/grpc/grpc/blob/master/doc/grpc_xds_features.md)에 나열됩니다.



## gRPC는 모바일 애플리케이션 개발에 어떻게 도움이 되나요?

gRPC 및 Protobuf는 서비스를 정확하게 정의하고 iOS, Android 및 백엔드를 제공하는 서버를 위한 안정적인 클라이언트 라이브러리를 자동 생성하는 쉬운 방법을 제공합니다. 클라이언트는 대역폭을 절약하고 더 적은 수의 TCP 연결로 더 많은 작업을 수행하며 CPU 사용량과 배터리 수명을 절약하는 데 도움이 되는 고급 스트리밍 및 연결 기능을 활용할 수 있습니다.



## gRPC가 HTTP/2를 통한 바이너리 Blob보다 나은 이유는 무엇가요?

이것은 주로 gRPC가 유선에 있는 것입니다. 그러나 gRPC는 일반적인 HTTP 라이브러리가 일반적으로 제공하지 않는 플랫폼 전반에 걸쳐 일관되게 더 높은 수준의 기능을 제공하는 라이브러리 세트이기도 합니다. 이러한 기능의 예는 다음과 같습니다:

* 애플리케이션 계층에서 흐름 제어와 상호 작용
* 단계적(cascading) 호출 취소
* 로드 밸런싱 & 장애 조치



## gRPC가 REST보다 나은 / 나쁜 이유는 무엇이 있나요?

gRPC는 HTTP/2를 통한 HTTP 의미 체계를 크게 따르지만 명시적으로 전이중 스트리밍을 허용합니다. 경로, 쿼리 매개변수 및 페이로드 본문에서 호출 매개변수를 구문 분석하면 대기 시간과 복잡성이 추가되므로 호출 디스패치 중 성능상의 이유로 정적 경로를 사용하기 때문에 일반적인 REST 규칙과 다릅니다. 또한 HTTP 상태 코드보다 API 사용 사례에 더 직접적으로 적용할 수 있다고 생각되는 일련의 오류를 공식화했습니다.



## gRPC를 어떻게 발음하나요?

Jee-Arr-Pee-See.



---

## 의견 / 진행

* 공식문서외에도 https://grpc.io/blog/ 의 글도 봐야할 것 같다.