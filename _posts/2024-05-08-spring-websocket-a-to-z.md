---
title: "Spring+WebSocket A to Z"
date: 2024-05-08 10:00:00 +0900
categories: [Spring]
tags: [Spring, WebSocket, STOMP, 실시간통신, 채팅, 양방향통신]
---

# Spring+WebSocket A to Z

## Why?

현재 백엔드에서 Spring Boot를 이용해서 서비스를 개발하고 있습니다.
현재 이 서비스에서는 유저간의 소통을 위해 채팅 기능을 제공하고자
하는데요, 유저 간의 채팅은 현재 서비스에 접속된 유저 간에 실시간으로
이뤄져야 하기 때문에 Websocket을 이용한 방식을 자주 사용한다고 하여
선정하게 되었습니다

## 인터넷 통신 계층과 통신 방법

다음 그림은 인터넷 프로토콜의 계층과 통신 방법입니다.

![인터넷 프로토콜 계층](/assets/img/posts/2024-05-08-spring-websocket-a-to-z/socket_1.webp)

위 그림을 조금 더 쉽게 생각하면 Ethernet → IP → TCP → HTTP 로 생각할 수 있고, 인터넷 통신이 위와 같은 계층을 거쳐 이뤄진다는 것을 알 수 있다.

즉 우리가 사용하는 웹 애플리케이션은 HTTP 레이어에서 동작하고
이 레이어에서 사용하는 라이브러리를 통해 OS 계층인 TCP, IP 레이어로
또 이 패킷은 네트워크 인터페이스로 전달하여 인터넷을 통해 다른 서버로 메시지(data)를 전달하는 것입니다.

우리가 사용하는 WebSocket은 이 애플리케이션 레이어에서 Http 통신
프로토콜 특징으로 인해 나타나는 불편함을 해결하기 위해 User Layer(application layer)에서 사용하는 통신 방법이라고 볼 수 있다.

## Web Socket VS TCP/IP Socket

제가 웹소켓을 공부하게 되면서 의문점을 갖게 된 것 중 하나가 바로
"Web Socket과 System programming에서의 Socket(TCP/IP)이 다른 가?"
였습니다.

저는 학교에서 Linux System Programming에 대해서 공부하면서
Socket에 대해서도 배우게 되었는데, OS를 배우시면서 한번 쯤 공부해보셨을 수도 있다고 생각합니다.

![TCP/IP Socket 동작 방식](/assets/img/posts/2024-05-08-spring-websocket-a-to-z/socket_2.webp)

위 그림이 System Programming에서 Socket이 어떤 방식으로 동작되는 것인지 나타낸 것입니다. 우선 Server에서 Socket이라는 시스템 콜로 파일 디스크립터를 만들고, bind 시스템 콜을 호출하여 이름을 정합니다. 이후 Server에서는 Client의 요청을 기다리다가 Client의 connect 요청이 이뤄지면 복사본 소켓을 만들고(accept) 이 소켓에 Client가 연결하게 됩니다.

이런 구조로 이뤄지는 게 저희가 OS나 System Programming을 공부하면서 배우게 되는 Socket입니다. 이렇게 프로그램을 완성하고 나면, 저희가 생각하는 것처럼 양방향으로 통신을 할 수 있게 됩니다.

이렇다보니, WebSocket과 굉장히 유사합니다. 그래서 저는 굳이 WebSocket이라고 지칭하는 지 이해가 되지 않았고 그러다보니 위에 인터넷 프로토콜
계층 그림에서 application layer에 Socket Library를 통해 Message를 보낸다는 것도 이해가 되지 않았습니다.

교수님 답변을 토대로 제가 재해석해서 설명해보자면 open() System call과
C Library를 이용한 fopen()을 비교해봤을 때, open()은 저희가 직접적으로 System call을 사용하는 것이고 fopen()은 C 표준 Library의 함수를 통해
추가적인 이점을 얻으면서 open() System call을 호출하는 것입니다.
즉, User Level에서 부가적인 이점과 편의성을 갖고 사용하기 위해 미리 작성된 Library의 함수를 통해 fopen()을 호출하는 것처럼 WebSocket도 Framework 개발자들이 만들어 놓은 Socket Library를 사용해서 Socket 기능을 사용할 수 있게 되는 것입니다. 그래서 개념적으로 유사하면서도 약간의 차이점을 갖고 있다고 생각하는 것이 바람직할 것 같습니다.

## HTTP 통신의 특징

우선 HTTP의 특징에 대해서 간단하게 알아봅시다.

### 단방향(request — response 구조)

요청에 대해 응답을 보내는 단방향적인 통신 구조를 가집니다.
따라서 소켓처럼 계속 connection이 유지되는 실시간 통신을 할 수 없습니다.

### 비연결성(Connectionless)

연결성을 유지하지 않아 서버가 클라이언트를 식별할 수 없는 무상태(stateless)

연결을 유지하기 위한 리소스를 줄이면 더 많은 연결을 할 수 있으나
동일한 클라이언트의 모든 요청에 대해, 매번 새로운 연결을 시도/해제의 과정을 거쳐야하므로(서버는 클라이언트를 기억하고 있지 않으므로) 연결/해제에 대한 오버헤드가 발생한다는 단점이 있습니다.

### 헤더의 비중이 크다.

실시간 성으로 많은 데이터를 주고 받는 경우 cost가 많이 듭니다.

## WebSocket의 특징

앞서 HTTP 통신의 특징을 살펴 봤습니다. 요약하자면, 단방향 통신,
비연결성, 헤더가 HTTP 통신 특징이라는 것입니다.

![HTTP 폴링 방식](/assets/img/posts/2024-05-08-spring-websocket-a-to-z/socket_3.webp)

상대적으로 빈도가 높은 데이터 가져오기에 대한 HTTP 연결의 전통적인
패턴은 폴링으로, 클라이언트가 주기적으로 새 서버 데이터를 요청하는 방식입니다.
이 방식은 대기 시간을 계속해서 발생시킬 수 있고, HTTP 헤더 크기 등으로 인한 오버헤드를 발생시키기 때문에 WebSocket을 이용하는 것에 비해
비효율적입니다.

WebSocket을 이용하는 것이 왜 더 좋은 지 WebSocket 통신 방식을 보며 장점을 알아봅시다.

![WebSocket 통신 방식](/assets/img/posts/2024-05-08-spring-websocket-a-to-z/socket_4.webp)

### 양방향 통신

WebSocket은 클라이언트와 서버 간에 양방향 통신을 지원합니다. 연결된 양쪽에서 언제든지 메시지를 보낼 수 있기 때문에, 데이터를 빠르게 주고받아야 할 때 WebSocket 연결이 유용한 이유입니다.

### 짧은 대기 시간

WebSocket 연결을 사용하면 데이터가 사용 가능한 즉시 전송됩니다. 전통적인 HTTP 연결 방식을 이용한 Polling과는 달리 클라이언트는 계속 요청할 필요가 없습니다. 결과적으로 오버헤드와 네트워크 트래픽이 최소화되어 대기 시간이 훨씬 단축됩니다.

### 지속적인 연결

전통적인 HTTP 연결에서는 클라이언트가 요청을 하고 서버가 응답을 하면 연결을 닫습니다. 그래서 클라이언트는 더 많은 데이터를 받기 위해서는 새 연결을 해야합니다. 하지만 WebSocket 연결을 하면 연결을 닫지 않아서 단일 연결만으로도 지속적으로 연결할 수 있습니다.

## Spring에서의 구현(STOMP)

그렇다면 이제 Spring에서 채팅 기능을 어떻게 구현하는지 알아봅시다.
Spring에서는 메시지 전송을 효율적으로 하기 위한 프로토콜인 STOMP(Simple Text Oriented Messaging Protocol)를 지원합니다.
이를 통해 따로 WebSocketHandler를 복잡하게 구현할 필요가 없고, @MessageMapping과 같은 어노테이션을 사용하여, 메시지 발행 시
엔드 포인트를 별도로 분리하여 관리할 수 있습니다.

Stomp를 사용한다면 일반 WebSocket을 사용해서 session으로 방을 분류하던 방식에서 특정 엔드포인트로 분류하는 방식으로 간편하게 바뀌었다는
장점도 있습니다. (그렇다고 Session이 완전 없는 건 아님)

STOMP의 몇가지 특징에 대해 알아보고 코드로 넘어가자.
STOMP에서는 메시지를 송, 수신에 대한 처리를 명확하게 하기 위해 pub/sub(발행/구독) 구조를 사용하고 있다.

![STOMP Pub/Sub 구조](/assets/img/posts/2024-05-08-spring-websocket-a-to-z/socket_5.webp)

pub/sub 구조란 Publisher와 Subscriber Publisher가 메시지를 발행하여 메시지 브로커에서 전달하고

해당 channel을 구독한 Subscriber에게 메시지 브로커가 메시지를 전달하는 구조를 말합니다.

(서버를 메시지 브로커로 생각하면 된다.)

### 기타

![Spring Message Broker](/assets/img/posts/2024-05-08-spring-websocket-a-to-z/socket_6.webp)

Spring Boot에서는 기본적으로 Message Broker를 제공하며 이 Message Broker는 Spring Boot 서버의 내부 메모리에 존재합니다. 현재는 일단 구현하는 것이 목표이고, 간단하게 만드는 것이기에 외부 메시지 브로커를 사용하지는 않았다.

하지만 Spring 자체적인 Broker는 메시지 유실 가능성이 있고, 모니터링이 쉽지 않다. 또 많은 양의 데이터를 처리하기에는 적합하지 않아서 실제 사용자가 있는 서비스를 개발하기 위해서는 외부 메시지 브로커를 사용해야 할 것이다(Redis나 RabbitMQ).

## Spring STOMP 구현

### Entity — Message

```java
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
public class Message {
    private Long senderId;
    private Long channelId;
    private String content;
    
    public enum MessageType{
        ENTER, TALK
        //,LEAVE
        ;
    }
    private MessageType messageType;
}
```

### WebSocketConfig

```java
@Configuration
@RequiredArgsConstructor
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws")
                .setAllowedOriginPatterns("*");
                //.withSockJS()
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/sub");
        registry.setApplicationDestinationPrefixes("/pub");
    }
}
```

### Controller

```java
@Slf4j
@RequiredArgsConstructor
@Controller
public class ChatController {

    @Autowired
    private SimpMessageSendingOperations simpleMessageSendingOperations;

    @MessageMapping("/message/{roomId}")
    public void send(Message message) {
        if (message.MessageType.ENTER.equals(message.getType())){
            message.setContent(chat.getSenderId() + "님이 입장하셨습니다.");
        }
        simpleMessageSendingOperations.convertAndSend("/sub/chat/room/" + message.getChannelId(), message);
    }
}
```
