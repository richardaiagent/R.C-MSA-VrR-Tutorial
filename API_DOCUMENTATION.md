# 🔌 AI 채팅·협업 솔루션 API 문서

## 📋 개요
본 문서는 Vert.x MSA 기반 AI 채팅·협업 플랫폼의 REST API 및 WebSocket API 명세를 제공합니다.

---

## 🌐 기본 정보

### Base URLs
- **Development**: `http://localhost:8080/api/v1`
- **Staging**: `https://staging-api.company.com/api/v1`
- **Production**: `https://api.company.com/api/v1`

### 인증 방식
- **Type**: Bearer Token (JWT)
- **Header**: `Authorization: Bearer <token>`
- **Token Expiry**: 1시간 (Refresh Token으로 갱신 가능)

### 공통 응답 형식
```json
{
  "success": true,
  "data": {},
  "message": "Success",
  "timestamp": "2025-05-24T10:30:00Z",
  "traceId": "abc123-def456-ghi789"
}
```

### 에러 응답 형식
```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "사용자를 찾을 수 없습니다.",
    "details": {}
  },
  "timestamp": "2025-05-24T10:30:00Z",
  "traceId": "abc123-def456-ghi789"
}
```

---

## 🔐 인증 API (Auth Service)

### POST /auth/login
사용자 로그인

**Request Body:**
```json
{
  "email": "user@company.com",
  "password": "securePassword123",
  "rememberMe": false
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "tasks": [
      {
        "id": "task123",
        "title": "사용자 인증 API 개발",
        "description": "JWT 기반 사용자 인증 시스템 구현",
        "status": "IN_PROGRESS",
        "priority": "HIGH",
        "assignee": {
          "id": "user123",
          "name": "홍길동",
          "avatar": "https://storage.company.com/avatars/user123.jpg"
        },
        "reporter": {
          "id": "user456",
          "name": "김매니저"
        },
        "project": {
          "id": "proj123",
          "name": "AI 채팅 플랫폼"
        },
        "dueDate": "2025-05-30T23:59:59Z",
        "estimatedHours": 16,
        "actualHours": 8,
        "tags": ["backend", "security", "api"],
        "createdAt": "2025-05-20T09:00:00Z",
        "updatedAt": "2025-05-24T10:15:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "size": 20,
      "total": 1,
      "totalPages": 1
    },
    "summary": {
      "todo": 5,
      "inProgress": 3,
      "done": 12,
      "cancelled": 1
    }
  }
}
```

### POST /tasks
새 업무 생성

**Request Body:**
```json
{
  "title": "데이터베이스 스키마 설계",
  "description": "사용자, 채팅, 업무 관리를 위한 데이터베이스 스키마 설계 및 구현",
  "priority": "HIGH",
  "assigneeId": "user123",
  "projectId": "proj123",
  "dueDate": "2025-06-15T23:59:59Z",
  "estimatedHours": 24,
  "tags": ["database", "design", "backend"],
  "checklist": [
    {
      "title": "ERD 작성",
      "completed": false
    },
    {
      "title": "테이블 생성 스크립트 작성",
      "completed": false
    }
  ]
}
```

### GET /tasks/{taskId}
특정 업무 상세 조회

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "task123",
    "title": "사용자 인증 API 개발",
    "description": "JWT 기반 사용자 인증 시스템 구현",
    "status": "IN_PROGRESS",
    "priority": "HIGH",
    "assignee": {
      "id": "user123",
      "name": "홍길동",
      "email": "user@company.com",
      "avatar": "https://storage.company.com/avatars/user123.jpg"
    },
    "reporter": {
      "id": "user456",
      "name": "김매니저"
    },
    "project": {
      "id": "proj123",
      "name": "AI 채팅 플랫폼",
      "key": "ACP"
    },
    "dueDate": "2025-05-30T23:59:59Z",
    "estimatedHours": 16,
    "actualHours": 8,
    "tags": ["backend", "security", "api"],
    "checklist": [
      {
        "id": "check1",
        "title": "JWT 라이브러리 설정",
        "completed": true,
        "completedAt": "2025-05-22T14:30:00Z"
      },
      {
        "id": "check2",
        "title": "토큰 검증 로직 구현",
        "completed": false
      }
    ],
    "attachments": [
      {
        "id": "att123",
        "name": "API_명세서.pdf",
        "url": "https://storage.company.com/files/att123.pdf",
        "size": 1024000
      }
    ],
    "comments": [
      {
        "id": "comment123",
        "content": "진행 상황 업데이트: JWT 설정 완료",
        "author": {
          "id": "user123",
          "name": "홍길동"
        },
        "createdAt": "2025-05-24T09:30:00Z"
      }
    ],
    "createdAt": "2025-05-20T09:00:00Z",
    "updatedAt": "2025-05-24T10:15:00Z"
  }
}
```

### PUT /tasks/{taskId}
업무 수정

**Request Body:**
```json
{
  "title": "사용자 인증 API 개발 (수정)",
  "description": "JWT 기반 사용자 인증 시스템 구현 - OAuth2 연동 추가",
  "status": "IN_PROGRESS",
  "priority": "HIGH",
  "assigneeId": "user123",
  "dueDate": "2025-06-05T23:59:59Z",
  "estimatedHours": 20,
  "tags": ["backend", "security", "api", "oauth2"]
}
```

### POST /tasks/{taskId}/comments
업무에 댓글 추가

**Request Body:**
```json
{
  "content": "OAuth2 설정 관련 문서를 첨부합니다.",
  "attachments": [
    {
      "name": "OAuth2_설정가이드.pdf",
      "url": "https://storage.company.com/files/temp/oauth_guide.pdf",
      "size": 512000
    }
  ]
}
```

---

## 🔔 알림 API (Notification Service)

### GET /notifications
알림 목록 조회

**Query Parameters:**
- `type` (string): 알림 타입 (CHAT, TASK, SYSTEM, AI)
- `read` (boolean): 읽음 상태
- `page` (integer): 페이지 번호
- `size` (integer): 페이지 크기

**Response:**
```json
{
  "success": true,
  "data": {
    "notifications": [
      {
        "id": "noti123",
        "type": "CHAT",
        "title": "새 메시지 도착",
        "message": "홍길동님이 개발팀 채팅방에 메시지를 보냈습니다.",
        "data": {
          "roomId": "room123",
          "messageId": "msg456",
          "senderId": "user123"
        },
        "read": false,
        "createdAt": "2025-05-24T10:25:00Z"
      },
      {
        "id": "noti124",
        "type": "TASK",
        "title": "업무 마감일 임박",
        "message": "사용자 인증 API 개발 업무의 마감일이 6일 남았습니다.",
        "data": {
          "taskId": "task123",
          "dueDate": "2025-05-30T23:59:59Z"
        },
        "read": true,
        "readAt": "2025-05-24T10:20:00Z",
        "createdAt": "2025-05-24T09:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "size": 20,
      "total": 2,
      "totalPages": 1
    },
    "unreadCount": 1
  }
}
```

### PUT /notifications/{notificationId}/read
알림 읽음 처리

**Response:**
```json
{
  "success": true,
  "message": "알림이 읽음 처리되었습니다."
}
```

### PUT /notifications/read-all
모든 알림 읽음 처리

**Response:**
```json
{
  "success": true,
  "message": "모든 알림이 읽음 처리되었습니다.",
  "data": {
    "updatedCount": 5
  }
}
```

### POST /notifications/subscribe
푸시 알림 구독

**Request Body:**
```json
{
  "endpoint": "https://fcm.googleapis.com/fcm/send/...",
  "keys": {
    "auth": "auth-key",
    "p256dh": "p256dh-key"
  },
  "deviceType": "WEB",
  "deviceInfo": {
    "browser": "Chrome",
    "os": "Windows"
  }
}
```

---

## 📁 파일 API (File Service)

### POST /files/upload
파일 업로드

**Request:** Multipart Form Data
- `file`: 업로드할 파일
- `folder`: 저장 폴더 (optional)
- `description`: 파일 설명 (optional)

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "file123",
    "name": "document.pdf",
    "originalName": "프로젝트_문서.pdf",
    "size": 2048576,
    "mimeType": "application/pdf",
    "url": "https://storage.company.com/files/file123.pdf",
    "thumbnailUrl": "https://storage.company.com/thumbnails/file123.jpg",
    "folder": "documents",
    "uploadedBy": "user123",
    "createdAt": "2025-05-24T10:30:00Z"
  }
}
```

### GET /files/{fileId}
파일 정보 조회

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "file123",
    "name": "document.pdf",
    "originalName": "프로젝트_문서.pdf",
    "size": 2048576,
    "mimeType": "application/pdf",
    "url": "https://storage.company.com/files/file123.pdf",
    "downloadUrl": "https://storage.company.com/download/file123",
    "uploadedBy": {
      "id": "user123",
      "name": "홍길동"
    },
    "createdAt": "2025-05-24T10:30:00Z"
  }
}
```

### DELETE /files/{fileId}
파일 삭제

**Response:**
```json
{
  "success": true,
  "message": "파일이 삭제되었습니다."
}
```

---

## 🔌 WebSocket API

### 연결 엔드포인트
```
ws://localhost:8082/ws/chat?token=<jwt_token>
```

### 메시지 타입

#### 1. 채팅 메시지 전송
```json
{
  "type": "SEND_MESSAGE",
  "data": {
    "roomId": "room123",
    "content": "안녕하세요!",
    "messageType": "TEXT"
  }
}
```

#### 2. 채팅방 입장
```json
{
  "type": "JOIN_ROOM",
  "data": {
    "roomId": "room123"
  }
}
```

#### 3. 채팅방 퇴장
```json
{
  "type": "LEAVE_ROOM",
  "data": {
    "roomId": "room123"
  }
}
```

#### 4. 타이핑 상태 전송
```json
{
  "type": "TYPING",
  "data": {
    "roomId": "room123",
    "isTyping": true
  }
}
```

### 서버에서 클라이언트로 보내는 메시지

#### 1. 새 메시지 수신
```json
{
  "type": "NEW_MESSAGE",
  "data": {
    "id": "msg123",
    "roomId": "room123",
    "content": "안녕하세요!",
    "messageType": "TEXT",
    "sender": {
      "id": "user456",
      "name": "김개발",
      "avatar": "https://storage.company.com/avatars/user456.jpg"
    },
    "timestamp": "2025-05-24T10:30:00Z"
  }
}
```

#### 2. 사용자 상태 변경
```json
{
  "type": "USER_STATUS",
  "data": {
    "userId": "user123",
    "status": "ONLINE",
    "lastSeen": "2025-05-24T10:30:00Z"
  }
}
```

#### 3. 타이핑 상태 수신
```json
{
  "type": "USER_TYPING",
  "data": {
    "roomId": "room123",
    "userId": "user456",
    "userName": "김개발",
    "isTyping": true
  }
}
```

---

## 📊 상태 코드 및 에러 코드

### HTTP 상태 코드
- `200` OK - 요청 성공
- `201` Created - 리소스 생성 성공
- `400` Bad Request - 잘못된 요청
- `401` Unauthorized - 인증 실패
- `403` Forbidden - 권한 없음
- `404` Not Found - 리소스 없음
- `409` Conflict - 리소스 충돌
- `429` Too Many Requests - 요청 한도 초과
- `500` Internal Server Error - 서버 오류

### 커스텀 에러 코드
| 코드 | 설명 |
|------|------|
| `AUTH_INVALID_TOKEN` | 유효하지 않은 토큰 |
| `AUTH_TOKEN_EXPIRED` | 토큰 만료 |
| `AUTH_INSUFFICIENT_PERMISSIONS` | 권한 부족 |
| `USER_NOT_FOUND` | 사용자 없음 |
| `USER_ALREADY_EXISTS` | 이미 존재하는 사용자 |
| `CHAT_ROOM_NOT_FOUND` | 채팅방 없음 |
| `CHAT_ACCESS_DENIED` | 채팅방 접근 거부 |
| `TASK_NOT_FOUND` | 업무 없음 |
| `TASK_INVALID_STATUS` | 유효하지 않은 업무 상태 |
| `AI_MODEL_NOT_AVAILABLE` | AI 모델 사용 불가 |
| `AI_REQUEST_FAILED` | AI 요청 실패 |
| `FILE_TOO_LARGE` | 파일 크기 초과 |
| `FILE_TYPE_NOT_SUPPORTED` | 지원하지 않는 파일 형식 |
| `RATE_LIMIT_EXCEEDED` | 요청 한도 초과 |

---

## 🔄 Rate Limiting

### 제한 규칙
- **일반 API**: 사용자당 분당 100회
- **AI API**: 사용자당 분당 10회
- **파일 업로드**: 사용자당 시간당 50회
- **WebSocket 메시지**: 사용자당 초당 5회

### Rate Limit 헤더
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1632150000
```

---

## 📝 버전 관리

### API 버전 전략
- **URL 경로 버전**: `/api/v1/`, `/api/v2/`
- **하위 호환성**: 이전 버전 1년간 지원
- **Deprecation**: 6개월 전 사전 공지

### 변경 사항 알림
- **Major 변경**: 사전 공지 및 마이그레이션 가이드 제공
- **Minor 변경**: API 문서 업데이트
- **Patch 변경**: 버그 수정, 즉시 적용

---

## 🧪 테스트

### API 테스트 환경
- **Postman Collection**: [다운로드 링크]
- **Swagger UI**: `http://localhost:8080/swagger-ui`
- **테스트 계정**: 
  - Email: `test@company.com`
  - Password: `test123`

### 샘플 코드

#### JavaScript (Axios)
```javascript
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:8080/api/v1',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  }
});

// 사용자 프로필 조회
const getUserProfile = async () => {
  try {
    const response = await api.get('/users/profile');
    return response.data;
  } catch (error) {
    console.error('API Error:', error.response.data);
  }
};
```

#### Java (OkHttp)
```java
OkHttpClient client = new OkHttpClient();

Request request = new Request.Builder()
    .url("http://localhost:8080/api/v1/users/profile")
    .addHeader("Authorization", "Bearer " + token)
    .build();

Response response = client.newCall(request).execute();
String responseBody = response.body().string();
```

---

## 📚 추가 리소스

- **OpenAPI 스펙**: `/openapi.json`
- **Postman Collection**: `/docs/postman-collection.json`
- **SDK**: 
  - JavaScript/TypeScript: `@company/chat-api-sdk`
  - Java: `com.company:chat-api-client`
- **지원**: `api-support@company.com` {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4...",
    "expiresIn": 3600,
    "user": {
      "id": "user123",
      "email": "user@company.com",
      "name": "홍길동",
      "role": "USER",
      "avatar": "https://storage.company.com/avatars/user123.jpg"
    }
  }
}
```

### POST /auth/refresh
토큰 갱신

**Request Body:**
```json
{
  "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4..."
}
```

### POST /auth/logout
로그아웃

**Headers:**
```
Authorization: Bearer <token>
```

### POST /auth/register
사용자 회원가입

**Request Body:**
```json
{
  "email": "newuser@company.com",
  "password": "securePassword123",
  "name": "신규사용자",
  "department": "개발팀",
  "phone": "010-1234-5678"
}
```

---

## 👤 사용자 API (User Service)

### GET /users/profile
현재 사용자 프로필 조회

**Headers:**
```
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "user123",
    "email": "user@company.com",
    "name": "홍길동",
    "department": "개발팀",
    "position": "시니어 개발자",
    "phone": "010-1234-5678",
    "avatar": "https://storage.company.com/avatars/user123.jpg",
    "status": "ACTIVE",
    "lastLoginAt": "2025-05-24T09:30:00Z",
    "createdAt": "2025-01-15T10:00:00Z",
    "preferences": {
      "language": "ko",
      "timezone": "Asia/Seoul",
      "notifications": {
        "chat": true,
        "task": true,
        "email": false
      }
    }
  }
}
```

### PUT /users/profile
사용자 프로필 수정

**Request Body:**
```json
{
  "name": "홍길동",
  "department": "개발팀",
  "position": "시니어 개발자",
  "phone": "010-1234-5678",
  "preferences": {
    "language": "ko",
    "timezone": "Asia/Seoul",
    "notifications": {
      "chat": true,
      "task": true,
      "email": false
    }
  }
}
```

### GET /users/search
사용자 검색

**Query Parameters:**
- `q` (string): 검색 키워드
- `department` (string): 부서 필터
- `page` (integer): 페이지 번호 (기본값: 1)
- `size` (integer): 페이지 크기 (기본값: 20)

**Example:** `GET /users/search?q=홍길동&department=개발팀&page=1&size=10`

**Response:**
```json
{
  "success": true,
  "data": {
    "users": [
      {
        "id": "user123",
        "name": "홍길동",
        "email": "user@company.com",
        "department": "개발팀",
        "position": "시니어 개발자",
        "avatar": "https://storage.company.com/avatars/user123.jpg",
        "status": "ONLINE"
      }
    ],
    "pagination": {
      "page": 1,
      "size": 10,
      "total": 1,
      "totalPages": 1
    }
  }
}
```

---

## 💬 채팅 API (Chat Service)

### GET /chat/rooms
채팅방 목록 조회

**Query Parameters:**
- `type` (string): 채팅방 타입 (DIRECT, GROUP, AI_BOT)
- `page` (integer): 페이지 번호
- `size` (integer): 페이지 크기

**Response:**
```json
{
  "success": true,
  "data": {
    "rooms": [
      {
        "id": "room123",
        "name": "개발팀 채팅방",
        "type": "GROUP",
        "description": "개발팀 공용 채팅방",
        "memberCount": 15,
        "lastMessage": {
          "id": "msg456",
          "content": "오늘 회의 일정 확인 부탁드립니다.",
          "sender": {
            "id": "user789",
            "name": "김개발"
          },
          "timestamp": "2025-05-24T10:25:00Z"
        },
        "unreadCount": 3,
        "createdAt": "2025-05-20T14:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "size": 20,
      "total": 5,
      "totalPages": 1
    }
  }
}
```

### POST /chat/rooms
새 채팅방 생성

**Request Body:**
```json
{
  "name": "새 프로젝트 팀",
  "type": "GROUP",
  "description": "새 프로젝트 논의용 채팅방",
  "members": ["user123", "user456", "user789"],
  "isPrivate": false
}
```

### GET /chat/rooms/{roomId}/messages
채팅방 메시지 조회

**Path Parameters:**
- `roomId` (string): 채팅방 ID

**Query Parameters:**
- `before` (string): 이전 메시지 ID (페이징용)
- `limit` (integer): 조회할 메시지 수 (기본값: 50, 최대: 100)

**Response:**
```json
{
  "success": true,
  "data": {
    "messages": [
      {
        "id": "msg123",
        "content": "안녕하세요! 새로운 프로젝트에 대해 논의해보겠습니다.",
        "type": "TEXT",
        "sender": {
          "id": "user123",
          "name": "홍길동",
          "avatar": "https://storage.company.com/avatars/user123.jpg"
        },
        "timestamp": "2025-05-24T10:20:00Z",
        "edited": false,
        "reactions": [
          {
            "emoji": "👍",
            "count": 3,
            "users": ["user456", "user789", "user101"]
          }
        ],
        "attachments": []
      },
      {
        "id": "msg124",
        "content": "",
        "type": "FILE",
        "sender": {
          "id": "user456",
          "name": "김개발"
        },
        "timestamp": "2025-05-24T10:22:00Z",
        "attachments": [
          {
            "id": "file123",
            "name": "프로젝트_계획서.pdf",
            "size": 2048576,
            "mimeType": "application/pdf",
            "url": "https://storage.company.com/files/file123.pdf"
          }
        ]
      }
    ],
    "hasMore": true,
    "nextCursor": "msg122"
  }
}
```

### POST /chat/rooms/{roomId}/messages
메시지 전송

**Request Body:**
```json
{
  "content": "안녕하세요! 메시지입니다.",
  "type": "TEXT",
  "replyTo": "msg123",
  "attachments": [
    {
      "name": "document.pdf",
      "size": 1024000,
      "mimeType": "application/pdf",
      "url": "https://storage.company.com/files/temp/doc123.pdf"
    }
  ]
}
```

---

## 🤖 AI API (AI Service)

### POST /ai/chat
AI와 채팅

**Request Body:**
```json
{
  "message": "Java Spring Boot와 Vert.x의 차이점을 설명해주세요.",
  "context": {
    "conversationId": "conv123",
    "previousMessages": [
      {
        "role": "user",
        "content": "웹 프레임워크에 대해 알고 싶어요."
      },
      {
        "role": "assistant",
        "content": "웹 프레임워크는 웹 애플리케이션 개발을 위한 소프트웨어 플랫폼입니다..."
      }
    ]
  },
  "options": {
    "model": "llama2",
    "temperature": 0.7,
    "maxTokens": 1000,
    "useKnowledgeBase": true
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "response": "Java Spring Boot와 Eclipse Vert.x는 모두 자바 기반의 웹 프레임워크이지만 몇 가지 중요한 차이점이 있습니다...",
    "conversationId": "conv123",
    "model": "llama2",
    "usage": {
      "promptTokens": 150,
      "completionTokens": 300,
      "totalTokens": 450
    },
    "sources": [
      {
        "title": "Vert.x 공식 문서",
        "url": "https://vertx.io/docs/",
        "relevance": 0.95
      }
    ],
    "timestamp": "2025-05-24T10:30:00Z"
  }
}
```

### POST /ai/code-review
코드 리뷰 요청

**Request Body:**
```json
{
  "code": "public class UserService {\n    private UserRepository userRepository;\n    \n    public User findById(String id) {\n        return userRepository.findById(id);\n    }\n}",
  "language": "java",
  "reviewType": "SECURITY",
  "context": "Spring Boot 프로젝트의 서비스 클래스입니다."
}
```

### GET /ai/models
사용 가능한 AI 모델 목록

**Response:**
```json
{
  "success": true,
  "data": {
    "models": [
      {
        "name": "llama2",
        "displayName": "Llama 2 7B",
        "description": "Meta의 대화형 AI 모델",
        "size": "7B",
        "capabilities": ["chat", "completion", "translation"],
        "status": "READY",
        "lastUpdated": "2025-05-20T10:00:00Z"
      },
      {
        "name": "codellama",
        "displayName": "Code Llama 7B",
        "description": "코드 생성 특화 모델",
        "size": "7B",
        "capabilities": ["code-generation", "code-review", "debug"],
        "status": "READY",
        "lastUpdated": "2025-05-20T10:00:00Z"
      }
    ]
  }
}
```

---

## 📋 업무 관리 API (Task Service)

### GET /tasks
업무 목록 조회

**Query Parameters:**
- `status` (string): 업무 상태 (TODO, IN_PROGRESS, DONE, CANCELLED)
- `assignee` (string): 담당자 ID
- `priority` (string): 우선순위 (LOW, MEDIUM, HIGH, URGENT)
- `project` (string): 프로젝트 ID
- `page` (integer): 페이지 번호
- `size` (integer): 페이지 크기

**Response:**
```json
{
  "success": true,
  "data":

}
###  어둠의 서막이다. 꽁으로 먹을려고 했니