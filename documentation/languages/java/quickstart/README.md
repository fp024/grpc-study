# 빠른 시작

이 가이드에서는 간단하게 동작하는 예제를 통해 Java에서 gRPC를 시작할 수 있습니다.



### 내용

* 전제 조건
* 예제 코드 가져오기
* 예제 실행
* gRPC 서비스 업데이트
* 앱 업데이트
  * 서버 업데이트
  * 클라이언트 업데이트
* 업데이트된 앱 실행
* 다음에는 무엇을 해야할까요?



## 전제 조건

* [JDK](https://jdk.java.net/) 버전 7 또는 그 이상



## 예제 코드 가져오기

예제 코드는 [grpc-java](https://github.com/grpc/grpc-java) 리포지토리의 일부입니다.

1. 리포지토리를 zip 파일로 다운로드하고 압축을 풀거나 리포지토리를 복제합니다.

   ```bash
   $ git clone -b v1.42.1 https://github.com/grpc/grpc-java
   ```

2. 예제 디렉토리로 변경하세요

   ```bash
   cd grpc-java/examples
   ```

   



## 예제 실행

`example` 예제 디렉토리에서:

1. 클라이언트와 서버를 컴파일합니다.

   ```bash
   $ ./gradlew installDist
   ```

2. 서버를 실행합니다.

   ```bash
   $ ./build/install/examples/bin/hello-world-server
   INFO: Server started, listening on 50051
   ```
   
3. 다른 터미널에서 클라이언트를 실행합니다.

   ```bash
   $ ./build/install/examples/bin/hello-world-client
   INFO: Will try to greet world ...
   INFO: Greeting: Hello world
   ```

   축하합니다! 방금 gRPC로 클라이언트-서버 애플리케이션을 실행했습니다.

>### 참고
>
>이 페이지에 표시된 클라이언트 및 서버 추적 출력에서 타임스탬프를 생략했습니다.





## gRPC 서비스 업데이트

이 섹션에서는 추가 서버 메서드를 추가하여 애플리케이션을 업데이트합니다. gRPC 서비스는 [프로토콜 버퍼](https://developers.google.com/protocol-buffers)를 사용하여 정의됩니다. `.proto` 파일에서 서비스를 정의하는 방법에 대한 자세한 내용은 [기본 튜토리얼](https://grpc.io/docs/languages/java/basics/)을 참조하세요. 지금은 서버와 클라이언트 스텁에 클라이언트에서 HelloRequest 매개 변수를 사용하고 서버에서 HelloReply를 반환하는 SayHello() RPC 메서드가 있고 메서드가 다음과 같이 정의되어 있다는 것만 알아야 합니다.

```protobuf
// 인사말 서비스 정의입니다.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// 사용자 이름이 포함된 요청 메시지입니다.
message HelloRequest {
  string name = 1;
}

// 인사말이 포함된 응답 메시지
message HelloReply {
  string message = 1;
}
```

`src/main/proto/helloworld.proto`를 열고 `SayHello()`와 동일한 요청 및 응답 타입으로 새 `SayHelloAgain()` 메서드를 추가합니다:

```protobuf
// 인사말 서비스 정의입니다.
service Greeter {
  // 인사말을 보냅니다.
  rpc SayHello (HelloRequest) returns (HelloReply) {}
  // 또다른 인사말을 보냄
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// 사용자 이름이 포함된 요청 메시지입니다.
message HelloRequest {
  string name = 1;
}

// 인사말이 포함된 응답 메시지
message HelloReply {
  string message = 1;
}
```

파일을 저장하는 것을 잊지 마세요!





## 앱 업데이트

예제를 빌드할 때 빌드 프로세스는 생성된 gRPC 클라이언트 및 서버 클래스를 포함하는 GreeterGrpc.java를 다시 생성합니다. 이것은 또한 요청 및 응답 타입을 채우고 직렬화하고 검색하기 위한 클래스를 다시 생성합니다. 

그러나 여전히 예제 앱의 손으로 쓴 부분에서 새 메서드를 구현하고 호출해야 합니다. 



### 서버 업데이트

동일한 디렉토리에서 `src/main/java/io/grpc/examples/helloworld/HelloWorldServer.java`를 엽니다. 다음과 같이 새 메서드를 구현합니다:

```java
private class GreeterImpl extends GreeterGrpc.GreeterImplBase {

  @Override
  public void sayHello(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
    HelloReply reply = HelloReply.newBuilder().setMessage("Hello " + req.getName()).build();
    responseObserver.onNext(reply);
    responseObserver.onCompleted();
  }
 
  // 새로 추가할 메서드 - 시작
  @Override
  public void sayHelloAgain(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
    HelloReply reply = HelloReply.newBuilder().setMessage("Hello again " + req.getName()).build();
    responseObserver.onNext(reply);
    responseObserver.onCompleted();
  }
  // 새로 추가할 메서드 - 끝
}
```



### 클라이언트 업데이트

동일한 디렉토리에서 `src/main/java/io/grpc/examples/helloworld/HelloWorldClient.java`를 엽니다. 다음과 같이 새 메서드를 호출합니다:

```java
public void greet(String name) {
  logger.info("Will try to greet " + name + " ...");
  HelloRequest request = HelloRequest.newBuilder().setName(name).build();
  HelloReply response;
  try {
    response = blockingStub.sayHello(request);
  } catch (StatusRuntimeException e) {
    logger.log(Level.WARNING, "RPC failed: {0}", e.getStatus());
    return;
  }
  logger.info("Greeting: " + response.getMessage());
  // 추가할 코드 - 시작
  try {
    response = blockingStub.sayHelloAgain(request);
  } catch (StatusRuntimeException e) {
    logger.log(Level.WARNING, "RPC failed: {0}", e.getStatus());
    return;
  }
  logger.info("Greeting: " + response.getMessage());
  // 추가할 코드 - 끝
}

```



## 업데이트된 앱 실행

이전과 같이 클라이언트와 서버를 실행합니다. `examples` 디렉터리에서 다음 명령을 실행합니다. 

1.  클라이언트와 서버를 컴파일 합니다.

   ```bash
   $ ./gradlew installDist
   ```

2. 서버를 실행합니다.

   ```bash
   $ ./build/install/examples/bin/hello-world-server
   INFO: Server started, listening on 50051
   ```

3. 다른 터미널에서 클라이언트를 실행합니다. 

   ```bash
   $ ./build/install/examples/bin/hello-world-client
   INFO: Will try to greet world ...
   INFO: Greeting: Hello world
   INFO: Greeting: Hello again world
   ```

   



## 다음에는 무엇을 해야할까요?

* [gRPC 소개](../../../what-is-grpc/introduction-to-grpc.md) 및 [핵심 개념](../../../what-is-grpc/core-concepts-architecture-and-lifecycle.md)에서 gRPC가 작동하는 방식을 알아보세요. 
* [기본 튜토리얼](https://grpc.io/docs/languages/java/basics/)을 통해 작업합니다.
* [API 레퍼런스](https://grpc.io/docs/languages/java/api)를 살펴보세요. 





---

## 진행

* 전제 조건

* 예제 코드 가져오기

  * 커밋을 할 것은 아니여서, 크게 상관은 없는데... `detached HEAD` 상태가 되서

      ```bash
      # 현재 1.42.1 바라보고 있는 상태를 새로 v1.42.1 브랜치를 만들어 이동.
      git switch -c v1.42.1
      
      # 또는 처음부터 마스터로 받은 다음 v1.42.1 태그를 v1.42.1 브랜치로 생성
      $ git clone https://github.com/grpc/grpc-java
      $ git checkout tags/v1.42.1 -b v1.42.1
      ```

  

* 예제 실행

  * 서버 실행

      ```bash
      C:\git-other\grpc-java\examples>.\build\install\examples\bin\hello-world-server
      12월 29, 2021 1:44:01 오전 io.grpc.examples.helloworld.HelloWorldServer start
      정보: Server started, listening on 50051
      ```

  * 클라이언트 실행

      ```bash
      C:\git-other\grpc-java\examples>.\build\install\examples\bin\hello-world-client
      12월 29, 2021 1:45:00 오전 io.grpc.examples.helloworld.HelloWorldClient greet
      정보: Will try to greet world ...
      12월 29, 2021 1:45:01 오전 io.grpc.examples.helloworld.HelloWorldClient greet
      정보: Greeting: Hello world
      C:\git-other\grpc-java\examples>
      ```

      


* gRPC 서비스 업데이트

* 앱 업데이트

  * 서버 업데이트
  * 클라이언트 업데이트

* 업데이트된 앱 실행

  * 서버 실행

    ```bash
    C:\git-other\grpc-java\examples>.\build\install\examples\bin\hello-world-server
    12월 30, 2021 12:14:15 오전 io.grpc.examples.helloworld.HelloWorldServer start
    정보: Server started, listening on 50051
    ```

  * 클라이언트 실행

    ```bash
    C:\git-other\grpc-java\examples>.\build\install\examples\bin\hello-world-client
    12월 30, 2021 12:15:14 오전 io.grpc.examples.helloworld.HelloWorldClient greet
    정보: Will try to greet world ...
    12월 30, 2021 12:15:15 오전 io.grpc.examples.helloworld.HelloWorldClient greet
    정보: Greeting: Hello world
    12월 30, 2021 12:15:15 오전 io.grpc.examples.helloworld.HelloWorldClient greet
    정보: Greeting: Hello again world
    C:\git-other\grpc-java\examples>
    ```

    

* 다음에는 무엇을 해야할까요?



## 한글 윈도우 환경에서 한글 주석 사용시 인코딩 문제

build.gradle이나. gradle.properties에서 따로 인코딩 설정이 지정되어있지 않은데, 이럴 경우 **proto 파일에 한글 주석이 달린 상태**에서는 아래와 같은 오류가 노출되었다. proto 컴파일러가 proto 파일로 자바 소스 파일을 생성하는 과정에서 문제가 생긴 것 같다.  proto 파일을 UTF-8인코딩으로 저장했는데, 현재 환경설정이 MS949여서, 한글 주석이 들어간 부분 처리에 오류가 발생해서 생긴 내용 같다.

```
C:\git-other\grpc-java\examples\build\generated\source\proto\main\grpc\io\grpc\examples\helloworld\GreeterGrpc.java:146: error: unmappable character (0xEB) for encoding x-windows-949
     * ?삉?떎瑜? ?씤?궗留먯쓣 蹂대깂
       ^
C:\git-other\grpc-java\examples\build\generated\source\proto\main\grpc\io\grpc\examples\helloworld\GreeterGrpc.java:146: error: unmappable character (0xEB) for encoding x-windows-949
     * ?삉?떎瑜? ?씤?궗留먯쓣 蹂대깂
...
```



#### 1. example루트의 build.gradle 수정

```groovy
[compileJava, compileTestJava]*.options*.encoding = "UTF-8"
```

#### 2. example 프로젝트 루트에 gradle.properties를 만들고 아래 내용을 넣음

```properties
systemProp.file.encoding=utf-8
```

위의 둘 중 한가지 방법을 사용해서 적용해두면, 빌드시 오류가 발생하지 않았다. 

