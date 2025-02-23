### WebSocket Test HTML 파일 설명 (README)

---

## 📌 개요
이 HTML 파일은 **WebSocket 기반의 STOMP 프로토콜**을 이용하여 **채팅 기능을 테스트**하는 웹 페이지입니다.  
Spring Boot 백엔드와 WebSocket을 통해 연결되며, **SockJS** 및 **STOMP.js** 라이브러리를 사용하여 WebSocket 통신을 수행합니다.

---

## 📂 프로젝트 구조
- `index.html` → WebSocket 연결 및 채팅 기능을 테스트할 수 있는 프론트엔드 UI
- `SockJS & STOMP.js` → WebSocket 클라이언트 라이브러리

---

## 📌 주요 기능
1. **JWT 토큰을 이용한 WebSocket 연결**
2. **채팅방 구독 (Subscribe)**
3. **채팅 메시지 전송 (Send)**
4. **연결 상태 및 로그 확인**
5. **자동 재연결 기능 추가** (연결이 끊어질 경우 일정 시간 후 자동 재연결 시도)
6. **JWT 토큰 유효성 검사 및 만료 시간 경고 추가**
7. **로그 메시지 유형(성공, 경고, 오류) 추가**

---

## 🛠️ 사용된 기술
- HTML / JavaScript (Vanilla JS)
- [SockJS](https://github.com/sockjs/sockjs-client) (`sockjs.min.js`)
- [STOMP.js](https://github.com/stomp-js/stompjs) (`stomp.min.js`)
- WebSocket / STOMP 기반 통신
- JWT (JSON Web Token) 인증

---

## 🚀 실행 방법
### 1️⃣ 서버 실행
먼저, Spring Boot에서 WebSocket 서버(`ws://localhost:8080/ws`)가 실행 중이어야 합니다.

### 2️⃣ WebSocket 연결
1. **JWT 토큰 입력**  
   - "Token (without 'Bearer '):" 입력란에 **JWT 토큰을 입력**합니다.  
   - `Bearer ` 접두어 없이 입력해야 합니다.

2. **Connect 버튼 클릭**  
   - WebSocket 서버와 연결을 시도합니다.
   - 연결 성공 시 `"Connected: ..." 메시지`가 표시됩니다.

3. **Disconnect 버튼 클릭**  
   - WebSocket 연결을 종료합니다.

---

## 💬 채팅 기능 사용 방법
### 1️⃣ 채팅방 구독
1. **Room ID 입력**  
   - `채팅방 ID(Room ID)`를 입력합니다.
   - 예) `123`

2. **Subscribe Chat 버튼 클릭**  
   - 해당 채팅방을 구독하여 메시지를 받을 수 있습니다.
   - 서버에서 채팅 메시지가 도착하면 로그에 출력됩니다.

### 2️⃣ 채팅 메시지 전송
1. **Message 입력**  
   - 보낼 메시지를 입력합니다.

2. **Send Message 버튼 클릭**  
   - 입력한 메시지를 지정된 채팅방으로 전송합니다.
   - 전송 성공 시 `"Sent message to room ..."` 메시지가 표시됩니다.

---

## 📌 주요 코드 설명

### ✅ 1. JWT 토큰 검증 및 만료 시간 경고
```js
function validateToken(token) {
    if (!token) {
        return { isValid: false, message: 'Token is required' };
    }
    const parts = token.split('.');
    if (parts.length !== 3) {
        return { isValid: false, message: 'Invalid JWT format' };
    }
    try {
        const payload = JSON.parse(atob(parts[1]));
        const now = Date.now() / 1000;
        
        if (!payload.exp) {
            return { isValid: false, message: 'Token has no expiration' };
        }
        if (payload.exp < now) {
            return { isValid: false, message: 'Token has expired' };
        }
        if (payload.exp - now < 600) {
            return { isValid: true, warning: `Token will expire in ${Math.floor((payload.exp - now) / 60)} minutes` };
        }
        return { isValid: true, message: 'Token is valid', expiresIn: Math.floor((payload.exp - now) / 60) };
    } catch (e) {
        return { isValid: false, message: 'Invalid token format: ' + e.message };
    }
}
```
- JWT 토큰을 검증하고 만료 시간을 확인하여 **10분 이내에 만료될 경우 경고 메시지**를 표시합니다.

### ✅ 2. 자동 재연결 기능
```js
const RECONNECT_INTERVAL = 5000; // 5초
let reconnectTimer = null;
function handleDisconnect() {
    stompClient = null;
    socket = null;
    clearReconnectTimer();
    reconnectTimer = setTimeout(() => {
        log('connection', 'Attempting to reconnect...', 'warning');
        connect();
    }, RECONNECT_INTERVAL);
}
```
- 연결이 끊어지면 **5초 후 자동으로 재연결**을 시도합니다.

### ✅ 3. 로그 메시지 유형 추가
```js
function log(area, message, type = '') {
    const log = document.getElementById(area + 'Log');
    const time = new Date().toLocaleTimeString('ko-KR', {
        hour12: false,
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit',
        fractionalSecondDigits: 3
    });
    log.innerHTML += `<div class="${type}">[${time}] ${message}</div>`;
    log.scrollTop = log.scrollHeight;
}
```
- 로그에 `success`, `warning`, `error` 등의 유형을 추가하여 시각적으로 구분할 수 있도록 개선하였습니다.

---

## 🛠️ 개선 사항
1. **자동 재연결 기능 추가**  
   - 연결이 끊어질 경우 자동으로 재연결을 시도합니다.

2. **JWT 토큰 만료 검사 추가**  
   - 만료 10분 전이면 경고 메시지를 표시합니다.

3. **로그 메시지 개선**  
   - `성공 (success)`, `경고 (warning)`, `오류 (error)` 타입별 로그 메시지 출력 기능 추가

---

## 🏁 결론
이 HTML 파일은 **WebSocket + STOMP + JWT 인증**을 기반으로 채팅 기능을 테스트하는 클라이언트 코드입니다.  
기존 버전 대비 **JWT 검증 강화, 자동 재연결 기능 추가, 로그 메시지 개선** 등의 기능이 추가되었습니다. 이를 활용하여 WebSocket 기반의 채팅 서버와 클라이언트를 쉽게 테스트할 수 있습니다. 🚀

