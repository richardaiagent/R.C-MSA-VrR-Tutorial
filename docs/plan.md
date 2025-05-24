 # 프로젝트 개요

 ### 목표
Windows 11 + Docker Desktop + VS Code 환경에서 Vert.x 기반 MSA와 React 프론트엔드를 활용한 클라우드,AI 채팅·협업 솔루션 구축

### 기술 스택
- **Backend**: Java 11 + Eclipse Vert.x 4.x
- **Frontend**: React 18 + TypeScript
- **Database**: PostgreSQL + Redis
- **Container**: Docker + Docker Compose
- **Proxy**: Nginx
- **Message Queue**: Vert.x EventBus + Redis Pub/Sub
- **Monitoring**: Prometheus + Grafana

## 🎯 프로젝트 범위

### 핵심 기능
1. **사용자 관리** - 로그인, 회원가입, 프로필 관리
2. **실시간 채팅** - WebSocket 기반 메시징
3. **과제 관리** - 업무 할당, 진행률 추적
4. **주간보고** - 보고서 작성 및 제출
5. **알림 시스템** - 실시간 푸시 알림

### MSA 서비스 구성
```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  Gateway        │  │  User Service   │  │  Chat Service   │
│  (Vert.x)       │  │  (Vert.x)       │  │  (Vert.x)       │
│  Port: 8080     │  │  Port: 8081     │  │  Port: 8082     │
└─────────────────┘  └─────────────────┘  └─────────────────┘
         │                     │                     │
         └─────────────────────┼─────────────────────┘
                               │
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  Task Service   │  │ Notification    │  │  React Frontend │
│  (Vert.x)       │  │  Service        │  │  Port: 3000     │
│  Port: 8083     │  │  (Vert.x)       │  │                 │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

---
