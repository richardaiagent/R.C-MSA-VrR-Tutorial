# 🎨 핵심 기능 명세서 (상세화)

## 1. 실시간 채팅 시스템

### 1.1 WebSocket 기반 실시간 양방향 통신
**기술 구현:**
- **Vert.x WebSocket 서버 구현**
  - `ChatWebSocketHandler.java`: WebSocket 연결 관리
  - 메시지 라우팅 및 브로드캐스팅
  - 연결 풀 관리 (최대 10,000 동시 연결)
  - Heartbeat 및 Keep-alive 구현

**React Native 네이티브 브릿지 연동:**
```typescript
interface WebSocketBridge {
  connect(url: string, token: string): Promise<boolean>;
  sendMessage(message: ChatMessage): void;
  onMessage(callback: (message: ChatMessage) => void): void;
  onConnectionChange(callback: (status: ConnectionStatus) => void): void;
  disconnect(): void;
}

// Android 네이티브 모듈
@ReactModule(name = "WebSocketBridge")
public class WebSocketModule extends ReactContextBaseJavaModule {
  private OkHttpClient client;
  private WebSocket webSocket;
  
  public void connect(String url, String token, Promise promise) {
    Request request = new Request.Builder()
      .url(url)
      .header("Authorization", "Bearer " + token)
      .build();
    
    webSocket = client.newWebSocket(request, new WebSocketListener() {
      @Override
      public void onMessage(WebSocket webSocket, String text) {
        // React Native로 메시지 전달
        sendEventToReactNative("onMessage", text);
      }
    });
  }
}

// iOS Swift 브릿지
@objc(WebSocketBridge)
class WebSocketBridge: NSObject, RCTBridgeModule {
  static func moduleName() -> String! {
    return "WebSocketBridge"
  }
  
  @objc func connect(_ url: String, token: String, resolver: @escaping RCTPromiseResolveBlock, rejecter: @escaping RCTPromiseRejectBlock) {
    let request = URLRequest(url: URL(string: url)!)
    request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
    
    webSocketTask = URLSession.shared.webSocketTask(with: request)
    webSocketTask?.resume()
    
    receiveMessage()
    resolver(true)
  }
}
```

**자동 재연결 및 오프라인 메시지 큐잉:**
- 네트워크 상태 감지 (`@react-native-community/netinfo`)
- 백오프 전략을 사용한 재연결 (1초 → 2초 → 4초 → 최대 30초)
- 오프라인 시 메시지 로컬 큐잉 (`AsyncStorage`)
- 온라인 복구 시 큐된 메시지 자동 전송

### 1.2 멀티미디어 메시지 지원
**지원 형식:**
- **텍스트**: Markdown 지원, 이모지, 멘션(@user)
- **이미지**: JPEG, PNG, WebP (최대 10MB)
- **파일**: PDF, DOC, XLS, ZIP (최대 50MB)
- **음성**: WebM, MP3, M4A (최대 5분)
- **코드 스니펫**: 60개 언어 구문 강조

**업로드 플로우:**
```typescript
interface FileUploadService {
  uploadFile(file: File, messageId: string): Promise<UploadResult>;
  generateThumbnail(file: File): Promise<string>;
  compressImage(file: File): Promise<File>;
  validateFile(file: File): ValidationResult;
}

// 파일 업로드 프로세스
const uploadFlow = async (file: File) => {
  // 1. 파일 검증
  const validation = validateFile(file);
  if (!validation.isValid) throw new Error(validation.error);
  
  // 2. 이미지 압축 (이미지인 경우)
  const compressedFile = file.type.startsWith('image/') 
    ? await compressImage(file) 
    : file;
  
  // 3. 썸네일 생성
  const thumbnail = await generateThumbnail(compressedFile);
  
  // 4. 서버 업로드
  const result = await uploadFile(compressedFile, messageId);
  
  return { ...result, thumbnail };
};
```

### 1.3 AI 챗봇 통합 업무 지원
**AI 명령어 시스템:**
- `/ask-hr [질문]`: 인사 관련 질의응답
- `/code-help [요청]`: 코딩 지원 요청
- `/schedule [일정]`: 일정 관리 도움
- `/translate [언어] [텍스트]`: 번역 서비스
- `/summarize [URL/텍스트]`: 문서 요약

**AI 응답 처리 플로우:**
```java
@Component
public class AIIntegrationService {
  
  public CompletableFuture<AIResponse> processAICommand(String command, String userId) {
    return CompletableFuture.supplyAsync(() -> {
      // 1. 명령어 파싱
      AICommand parsedCommand = parseCommand(command);
      
      // 2. 사용자 컨텍스트 조회
      UserContext context = getUserContext(userId);
      
      // 3. 적절한 AI 모델 선택
      AIModel model = selectModel(parsedCommand.getType());
      
      // 4. 프롬프트 생성
      String prompt = buildPrompt(parsedCommand, context);
      
      // 5. AI 모델 호출
      String response = model.generate(prompt);
      
      // 6. 응답 후처리
      return postProcessResponse(response, parsedCommand.getType());
    });
  }
  
  private AIModel selectModel(CommandType type) {
    switch (type) {
      case HR_QUESTION: return llamaModel;
      case CODE_HELP: return codeLlamaModel;
      case DOCUMENT_TASK: return mistralModel;
      default: return defaultModel;
    }
  }
}
```

### 1.4 메시지 암호화 및 보안 기능

**End-to-End 암호화:**
```typescript
interface MessageEncryption {
  encryptMessage(message: string, recipientPublicKey: string): EncryptedMessage;
  decryptMessage(encryptedMessage: EncryptedMessage, privateKey: string): string;
  generateKeyPair(): KeyPair;
  rotateKeys(): Promise<void>;
}

// AES-256 + RSA 하이브리드 암호화
class HybridEncryption implements MessageEncryption {
  async encryptMessage(message: string, recipientPublicKey: string): Promise<EncryptedMessage> {
    // 1. AES-256 키 생성
    const aesKey = crypto.getRandomValues(new Uint8Array(32));
    
    // 2. 메시지 AES 암호화
    const encryptedMessage = await crypto.subtle.encrypt(
      { name: 'AES-GCM', iv: crypto.getRandomValues(new Uint8Array(12)) },
      await crypto.subtle.importKey('raw', aesKey, 'AES-GCM', false, ['encrypt']),
      new TextEncoder().encode(message)
    );
    
    // 3. AES 키 RSA 암호화
    const encryptedKey = await crypto.subtle.encrypt(
      { name: 'RSA-OAEP' },
      await crypto.subtle.importKey('spki', recipientPublicKey, 'RSA-OAEP', false, ['encrypt']),
      aesKey
    );
    
    return {
      encryptedContent: Array.from(new Uint8Array(encryptedMessage)),
      encryptedKey: Array.from(new Uint8Array(encryptedKey)),
      algorithm: 'AES-256-GCM+RSA-OAEP',
      timestamp: Date.now()
    };
  }
}
```

**메시지 만료 시간 설정:**
- 일반 메시지: 영구 보관
- 민감한 메시지: 24시간 후 자동 삭제
- 임시 메시지: 읽은 후 즉시 삭제
- 스케줄 삭제: 지정된 시간에 자동 삭제

### 1.5 대화 히스토리 검색 및 북마크

**Redis Vector Search 기반 의미론적 검색:**
```python
import redis
from sentence_transformers import SentenceTransformer

class MessageSearchService:
    def __init__(self):
        self.redis_client = redis.Redis(host='localhost', port=6379, decode_responses=True)
        self.encoder = SentenceTransformer('all-MiniLM-L6-v2')
        
    async def index_message(self, message_id: str, content: str, metadata: dict):
        # 메시지 임베딩 생성
        embedding = self.encoder.encode(content).tolist()
        
        # Redis에 벡터 저장
        self.redis_client.hset(
            f"message:{message_id}",
            mapping={
                "content": content,
                "vector": json.dumps(embedding),
                "timestamp": metadata["timestamp"],
                "user_id": metadata["user_id"],
                "chat_room_id": metadata["chat_room_id"]
            }
        )
        
    async def search_messages(self, query: str, user_id: str, limit: int = 10):
        # 쿼리 임베딩 생성
        query_embedding = self.encoder.encode(query).tolist()
        
        # 벡터 유사도 검색
        results = self.redis_client.execute_command(
            'FT.SEARCH', 'message_index',
            f'(@user_id:{user_id})=>[KNN {limit} @vector $query_vec AS score]',
            'PARAMS', 2, 'query_vec', json.dumps(query_embedding),
            'SORTBY', 'score', 'ASC',
            'RETURN', 3, 'content', 'timestamp', 'score'
        )
        
        return self.parse_search_results(results)
```

## 2. 지능형 업무 관리 (인사 업무 특화)

### 2.1 AI 기반 업무 분류 및 우선순위 자동 설정

**휴가 신청 자동 처리:**
```java
@Service
public class LeaveRequestService {
    
    public LeaveRequestResult processLeaveRequest(LeaveRequest request) {
        // 1. AI 기반 자동 승인 규칙 검증
        AIValidationResult validation = aiService.validateLeaveRequest(request);
        
        if (validation.isAutoApprovable()) {
            // 자동 승인 조건:
            // - 연차 잔여일수 충분
            // - 팀 내 동시 휴가자 3명 이하
            // - 프로젝트 마일스톤과 충돌 없음
            // - 과거 승인률 90% 이상
            return approveAutomatically(request, validation.getScore());
        }
        
        // 2. 매니저 승인 필요한 경우 우선순위 설정
        Priority priority = calculatePriority(request, validation);
        return sendForApproval(request, priority);
    }
    
    private Priority calculatePriority(LeaveRequest request, AIValidationResult validation) {
        int score = 0;
        
        // 요청 시급성 (당일 요청: +50, 1주일 전: +20, 1개월 전: +0)
        score += calculateUrgencyScore(request.getRequestDate(), request.getStartDate());
        
        // 팀 업무 부하 (높음: +30, 보통: +10, 낮음: +0)
        score += getTeamWorkloadScore(request.getTeamId());
        
        // 직원 신뢰도 (승인률 기반: 90%+: +0, 70-90%: +20, 70%-: +40)
        score += getEmployeeReliabilityScore(request.getEmployeeId());
        
        return Priority.fromScore(score);
    }
}
```

**출장 신청 예산 및 일정 충돌 검사:**
```java
@Service
public class BusinessTripService {
    
    public TripValidationResult validateBusinessTrip(BusinessTripRequest request) {
        CompletableFuture<BudgetValidation> budgetCheck = validateBudget(request);
        CompletableFuture<ScheduleValidation> scheduleCheck = validateSchedule(request);
        CompletableFuture<PolicyValidation> policyCheck = validatePolicy(request);
        
        return CompletableFuture.allOf(budgetCheck, scheduleCheck, policyCheck)
            .thenApply(v -> {
                TripValidationResult result = new TripValidationResult();
                result.setBudgetValidation(budgetCheck.join());
                result.setScheduleValidation(scheduleCheck.join());
                result.setPolicyValidation(policyCheck.join());
                
                // AI 추천 사항 생성
                result.setRecommendations(generateAIRecommendations(result));
                
                return result;
            }).join();
    }
    
    private CompletableFuture<BudgetValidation> validateBudget(BusinessTripRequest request) {
        return CompletableFuture.supplyAsync(() -> {
            // 1. 부서별 출장 예산 조회
            DepartmentBudget budget = budgetRepository.findByDepartmentAndYear(
                request.getDepartmentId(), 
                Year.now().getValue()
            );
            
            // 2. 기 사용 예산 계산
            BigDecimal usedBudget = calculateUsedBudget(request.getDepartmentId());
            BigDecimal remainingBudget = budget.getTotalBudget().subtract(usedBudget);
            
            // 3. AI 기반 비용 예측
            BigDecimal estimatedCost = aiService.estimateTripCost(request);
            
            BudgetValidation validation = new BudgetValidation();
            validation.setRemainingBudget(remainingBudget);
            validation.setEstimatedCost(estimatedCost);
            validation.setApproved(remainingBudget.compareTo(estimatedCost) >= 0);
            
            if (!validation.isApproved()) {
                validation.setSuggestions(aiService.generateCostReductionSuggestions(request));
            }
            
            return validation;
        });
    }
}
```

### 2.2 스마트 일정 관리 및 미팅 스케줄링

**참석자 가용시간 자동 분석:**
```typescript
interface SchedulingService {
  findOptimalMeetingTime(
    participants: string[],
    duration: number,
    preferredTimeRange: TimeRange,
    constraints: MeetingConstraints
  ): Promise<MeetingTimeSlot[]>;
}

class AISchedulingService implements SchedulingService {
  async findOptimalMeetingTime(
    participants: string[],
    duration: number,
    preferredTimeRange: TimeRange,
    constraints: MeetingConstraints
  ): Promise<MeetingTimeSlot[]> {
    
    // 1. 모든 참석자의 캘린더 정보 조회
    const calendars = await Promise.all(
      participants.map(id => this.calendarService.getCalendar(id, preferredTimeRange))
    );
    
    // 2. AI 기반 최적 시간 분석
    const analysis = await this.aiService.analyzeSchedulingContext({
      participants,
      calendars,
      duration,
      constraints,
      historicalMeetingData: await this.getHistoricalMeetingData(participants)
    });
    
    // 3. 충돌 없는 시간대 필터링
    const availableSlots = this.findAvailableTimeSlots(calendars, duration);
    
    // 4. AI 점수 기반 정렬
    const scoredSlots = availableSlots.map(slot => ({
      ...slot,
      score: this.calculateSlotScore(slot, analysis)
    })).sort((a, b) => b.score - a.score);
    
    return scoredSlots.slice(0, 5); // 상위 5개 추천
  }
  
  private calculateSlotScore(slot: TimeSlot, analysis: SchedulingAnalysis): number {
    let score = 100;
    
    // 선호 시간대 가중치 (오전 10-12시, 오후 2-4시 선호)
    if (this.isPreferredTime(slot.startTime)) score += 20;
    
    // 참석자 생산성 시간대 분석
    score += analysis.participantProductivityScore[slot.startTime.getHours()] || 0;
    
    // 회의실 접근성 (참석자 사무실과의 거리)
    score += analysis.locationConvenienceScore;
    
    // 과거 미팅 성공률 (같은 시간대)
    score += analysis.historicalSuccessRate[slot.startTime.getHours()] || 0;
    
    // 점심시간/퇴근시간 회피
    if (this.isInconvenientTime(slot.startTime)) score -= 30;
    
    return score;
  }
}
```

**회의실 예약 최적화:**
```java
@Service
public class MeetingRoomOptimizer {
    
    public RoomRecommendation recommendOptimalRoom(MeetingRequest request) {
        List<MeetingRoom> availableRooms = findAvailableRooms(
            request.getStartTime(), 
            request.getEndTime()
        );
        
        return availableRooms.stream()
            .map(room -> calculateRoomScore(room, request))
            .max(Comparator.comparing(RoomRecommendation::getScore))
            .orElseThrow(() -> new NoAvailableRoomException());
    }
    
    private RoomRecommendation calculateRoomScore(MeetingRoom room, MeetingRequest request) {
        double score = 0.0;
        
        // 1. 수용 인원 적합성 (정원의 70-90%가 최적)
        double capacityRatio = (double) request.getParticipantCount() / room.getCapacity();
        if (capacityRatio >= 0.7 && capacityRatio <= 0.9) {
            score += 25;
        } else if (capacityRatio < 0.7) {
            score += 15; // 여유 공간
        } else {
            score -= 10; // 수용 인원 초과 위험
        }
        
        // 2. 참석자와의 거리 최적화
        double avgDistance = calculateAverageDistance(room, request.getParticipants());
        score += Math.max(0, 20 - avgDistance); // 거리 20m당 -1점
        
        // 3. 필요 장비 보유 여부
        Set<Equipment> requiredEquipment = request.getRequiredEquipment();
        Set<Equipment> availableEquipment = room.getAvailableEquipment();
        if (availableEquipment.containsAll(requiredEquipment)) {
            score += 15;
        }
        
        // 4. 회의실 예약 이력 기반 선호도
        score += getRoomPreferenceScore(room, request.getOrganizer());
        
        // 5. AI 기반 회의 성공률 예측
        score += aiService.predictMeetingSuccessRate(room, request);
        
        return new RoomRecommendation(room, score);
    }
}
```

### 2.3 업무 진행률 추적 및 리포팅

**개인별 업무 대시보드:**
```typescript
interface ProductivityDashboard {
  generatePersonalDashboard(userId: string, period: DateRange): Promise<Dashboard>;
  calculateProductivityMetrics(userId: string): Promise<ProductivityMetrics>;
  generateInsights(metrics: ProductivityMetrics): Promise<Insight[]>;
}

class AIProductivityAnalyzer implements ProductivityDashboard {
  async generatePersonalDashboard(userId: string, period: DateRange): Promise<Dashboard> {
    const [tasks, meetings, communications] = await Promise.all([
      this.taskService.getUserTasks(userId, period),
      this.calendarService.getUserMeetings(userId, period),
      this.chatService.getUserCommunications(userId, period)
    ]);
    
    const metrics = await this.calculateProductivityMetrics(userId);
    const insights = await this.generateInsights(metrics);
    
    return {
      overview: {
        tasksCompleted: tasks.filter(t => t.status === 'COMPLETED').length,
        tasksInProgress: tasks.filter(t => t.status === 'IN_PROGRESS').length,
        averageTaskCompletionTime: this.calculateAverageCompletionTime(tasks),
        productivityScore: metrics.overallScore
      },
      timeDistribution: {
        focusTime: metrics.focusTime,
        meetingTime: metrics.meetingTime,
        communicationTime: metrics.communicationTime,
        breakTime: metrics.breakTime
      },
      performance: {
        weeklyTrend: metrics.weeklyProductivityTrend,
        peakProductivityHours: metrics.peakHours,
        accomplishments: this.extractAccomplishments(tasks),
        improvements: insights.filter(i => i.type === 'IMPROVEMENT')
      },
      aiInsights: insights
    };
  }
  
  async calculateProductivityMetrics(userId: string): Promise<ProductivityMetrics> {
    const rawData = await this.collectProductivityData(userId);
    
    // AI 모델을 통한 생산성 분석
    const analysis = await this.aiService.analyzeProductivity({
      taskCompletionData: rawData.tasks,
      timeTrackingData: rawData.timeTracking,
      communicationPatterns: rawData.communications,
      calendarData: rawData.calendar
    });
    
    return {
      overallScore: analysis.overallProductivityScore,
      focusTime: analysis.deepWorkHours,
      meetingTime: analysis.meetingHours,
      communicationTime: analysis.communicationHours,
      breakTime: analysis.breakHours,
      weeklyProductivityTrend: analysis.weeklyTrend,
      peakHours: analysis.mostProductiveHours,
      burnoutRisk: analysis.burnoutRiskScore
    };
  }
}
```

**팀별 생산성 분석:**
```java
@Service
public class TeamAnalyticsService {
    
    public TeamProductivityReport generateTeamReport(String teamId, LocalDate startDate, LocalDate endDate) {
        List<TeamMember> members = teamRepository.findMembersByTeamId(teamId);
        
        // 개별 구성원 생산성 데이터 수집
        Map<String, ProductivityMetrics> memberMetrics = members.stream()
            .collect(Collectors.toMap(
                TeamMember::getUserId,
                member -> productivityService.calculateMetrics(member.getUserId(), startDate, endDate)
            ));
        
        // AI 기반 팀 동역학 분석
        TeamDynamicsAnalysis dynamics = aiService.analyzeTeamDynamics(TeamAnalysisRequest.builder()
            .teamId(teamId)
            .memberMetrics(memberMetrics)
            .communicationPatterns(getCommunicationPatterns(teamId, startDate, endDate))
            .collaborationData(getCollaborationData(teamId, startDate, endDate))
            .build());
        
        return TeamProductivityReport.builder()
            .teamId(teamId)
            .period(DateRange.of(startDate, endDate))
            .overallProductivity(calculateTeamProductivityScore(memberMetrics))
            .memberPerformance(memberMetrics)
            .teamDynamics(dynamics)
            .insights(generateTeamInsights(dynamics))
            .recommendations(generateImprovementRecommendations(dynamics))
            .build();
    }
    
    private List<Insight> generateTeamInsights(TeamDynamicsAnalysis dynamics) {
        List<Insight> insights = new ArrayList<>();
        
        // 커뮤니케이션 패턴 분석
        if (dynamics.getCommunicationBalance() < 0.6) {
            insights.add(Insight.builder()
                .type(InsightType.COMMUNICATION)
                .severity(Severity.MEDIUM)
                .title("커뮤니케이션 불균형 감지")
                .description("팀 내 일부 구성원이 과도하게 많은 커뮤니케이션을 담당하고 있습니다.")
                .recommendation("업무 분담을 재조정하고 정기적인 팀 미팅을 통해 소통을 개선하세요.")
                .build());
        }
        
        // 업무 부하 분석
        if (dynamics.getWorkloadVariance() > 0.8) {
            insights.add(Insight.builder()
                .type(InsightType.WORKLOAD)
                .severity(Severity.HIGH)
                .title("업무 부하 불균형")
                .description("팀 구성원 간 업무 부하 차이가 큽니다.")
                .recommendation("업무 재배분을 통해 균형을 맞추고, 과부하 구성원의 번아웃을 방지하세요.")
                .build());
        }
        
        // 협업 효율성 분석
        if (dynamics.getCollaborationEfficiency() > 0.8) {
            insights.add(Insight.builder()
                .type(InsightType.COLLABORATION)
                .severity(Severity.POSITIVE)
                .title("우수한 팀 협업")
                .description("팀의 협업 효율성이 매우 높습니다.")
                .recommendation("현재의 협업 방식을 다른 팀에도 전파해보세요.")
                .build());
        }
        
        return insights;
    }
}
```

## 3. 코딩 어시스턴트 (개발 업무 특화)

### 3.1 React/React Native 컴포넌트 생성

**UI 목업을 코드로 자동 변환:**
```java
@Service
public class ComponentGeneratorService {
    
    public ComponentGenerationResult generateReactComponent(ComponentRequest request) {
        // 1. UI 명세 분석
        UISpecification spec = parseUISpecification(request.getDescription());
        
        // 2. AI 기반 컴포넌트 구조 설계
        ComponentStructure structure = aiService.designComponentStructure(
            ComponentDesignRequest.builder()
                .uiSpec(spec)
                .framework(request.getFramework()) // React or React Native
                .styling(request.getStylingPreference()) // CSS, Styled-Components, Tailwind
                .stateManagement(request.getStateManagement()) // useState, Redux, Context
                .build()
        );
        
        // 3. TypeScript 인터페이스 생성
        String interfaces = generateTypeScriptInterfaces(structure);
        
        // 4. 컴포넌트 코드 생성
        String componentCode = generateComponentCode(structure, request);
        
        // 5. 스타일 코드 생성
        String styleCode = generateStyleCode(structure, request.getStylingPreference());
        
        // 6. 테스트 코드 생성
        String testCode = generateTestCode(structure, request);
        
        return ComponentGenerationResult.builder()
            .componentCode(componentCode)
            .interfaceCode(interfaces)
            .styleCode(styleCode)
            .testCode(testCode)
            .usage(generateUsageExample(structure))
            .build();
    }
    
    private String generateComponentCode(ComponentStructure structure, ComponentRequest request) {
        StringBuilder code = new StringBuilder();
        
        // Import 문 생성
        code.append(generateImports(structure, request.getFramework()));
        
        // Interface 정의
        code.append(generateInterface(structure));
        
        // 컴포넌트 함수 시작
        code.append(String.format("const %s: React.FC<%sProps> = ({\n", 
            structure.getComponentName(), 
            structure.getComponentName()));
        
        // Props 구조분해할당
        code.append(generatePropsDestructuring(structure.getProps()));
        
        // State 관리 코드
        if (!structure.getStateVariables().isEmpty()) {
            code.append(generateStateManagement(structure.getStateVariables(), request.getStateManagement()));
        }
        
        // Effect Hooks
        if (!structure.getEffects().isEmpty()) {
            code.append(generateEffectHooks(structure.getEffects()));
        }
        
        // 이벤트 핸들러
        if (!structure.getEventHandlers().isEmpty()) {
            code.append(generateEventHandlers(structure.getEventHandlers()));
        }
        
        // JSX 반환
        code.append("  return (\n");
        code.append(generateJSX(structure.getJsxStructure(), request.getFramework()));
        code.append("  );\n");
        
        // 컴포넌트 함수 종료 및 export
        code.append("};\n\n");
        code.append(String.format("export default %s;\n", structure.getComponentName()));
        
        return code.toString();
    }
}
```

**TypeScript 인터페이스 자동 생성:**
```typescript
interface ComponentProps {
  // AI가 분석한 필수 props
  title: string;
  data: DataItem[];
  onItemSelect?: (item: DataItem) => void;
  loading?: boolean;
  error?: string;
  
  // 스타일 관련 props
  className?: string;
  style?: React.CSSProperties;
  theme?: 'light' | 'dark';
  
  // 접근성 props
  'aria-label'?: string;
  'data-testid'?: string;
}

interface DataItem {
  id: string | number;
  name: string;
  description?: string;
  imageUrl?: string;
  metadata?: Record<string, any>;
}

// AI가 생성한 상태 타입
interface ComponentState {
  selectedItems: DataItem[];
  searchQuery: string;
  sortOrder: 'asc' | 'desc';
  filterOptions: FilterOption[];
}

// 이벤트 핸들러 타입
interface ComponentHandlers {
  handleSearch: (query: string) => void;
  handleSort: (order: 'asc' | 'desc') => void;
  handleFilter: (options: FilterOption[]) => void;
  handleReset: () => void;
}
```

### 3.2 단위 화면 템플릿 자동 생성

**CRUD 화면 스캐폴딩:**
```java
@Service
public class CRUDScaffoldingService {
    
    public ScaffoldingResult generateCRUDScreens(CRUDRequest request) {
        EntityModel entity = parseEntityModel(request.getEntityDefinition());
        
        // 1. 목록 화면 생성
        ComponentCode listScreen = generateListScreen(entity, request);
        
        // 2. 상세 화면 생성
        ComponentCode detailScreen = generateDetailScreen(entity, request);
        
        // 3. 생성/편집 화면 생성
        ComponentCode formScreen = generateFormScreen(entity, request);
        
        // 4. API 서비스 코드 생성
        String apiService = generateApiService(entity, request);
        
        // 5. 상태 관리 코드 생성 (Redux/Context)
        String stateManagement = generateStateManagement(entity, request);
        
        // 6. 라우팅 설정 생성
        String routing = generateRouting(entity, request);
        
        return ScaffoldingResult.builder()
            .listScreen(listScreen)
            .detailScreen(detailScreen)
            .formScreen(formScreen)
            .apiService(apiService)
            .stateManagement(stateManagement)
            .routing(routing)
            .build();
    }
    
    private ComponentCode generateListScreen(EntityModel entity, CRUDRequest request) {
        return ComponentCode.builder()
            .fileName(entity.getName() + "ListScreen.tsx")
            .code(aiService.generateCode(CodeGenerationRequest.builder()
                .template("crud-list-screen")
                .entity(entity)
                .framework(request.getFramework())
                .styling(request.getStyling())
                .features(Arrays.asList(
                    "pagination",
                    "search",
                    "sorting",
                    "filtering",
                    "bulk-actions"
                ))
                .build()))
            .build();
    }
}
```

**폼 유효성 검증 로직 생성:**
```typescript
// AI가 생성한 검증 스키마
interface ValidationSchema {
  [fieldName: string]: ValidationRule[];
}

interface ValidationRule {
  type: 'required' | 'minLength' | 'maxLength' | 'pattern' | 'custom';
  value?: any;
  message: string;
  validator?: (value: any) => boolean;
}

// 자동 생성된 검증 함수
export const validateUserForm = (data: UserFormData): ValidationResult => {
  const errors: ValidationError[] = [];
  
  // 이름 검증
  if (!data.name?.trim()) {
    errors.push({
      field: 'name',
      message: '이름은 필수 입력 항목입니다.',
      type: 'required'
    });
  } else if (data.name.length < 2) {
    errors.push({
      field: 'name',
      message: '이름은 최소 2자 이상이어야 합니다.',
      type: 'minLength'
    });
  }
  
  // 이메일 검증
  if (!data.email?.trim()) {
    errors.push({
      field: 'email',
      message: '이메일은 필수 입력 항목입니다.',
      type: 'required'
    });
  } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(data.email)) {
    errors.push({
      field: 'email',
      message: '올바른 이메일 형식이 아닙니다.',
      type: 'pattern'
    });
  }
  
  // 비밀번호 검증
  if (!data.password) {
    errors.push({
      field: 'password',
      message: '비밀번호는 필수 입력 항목입니다.',
      type: 'required'
    });
  } else if (data.password.length < 8) {
    errors.push({
      field: 'password',
      message: '비밀번호는 최소 8자 이상이어야 합니다.',
      type: 'minLength'
    });
  } else if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/.test(data.password)) {
    errors.push({
      field: 'password',
      message: '비밀번호는 대소문자, 숫자, 특수문자를 포함해야 합니다.',
      type: 'pattern'
    });
  }
  
  return {
    isValid: errors.length === 0,
    errors,
    validFields: Object.keys(data).filter(field => 
      !errors.some(error => error.field === field)
    )
  };
};

// 실시간 검증 훅
export const useFormValidation = <T>(
  initialData: T,
  validationSchema: ValidationSchema
) => {
  const [data, setData] = useState<T>(initialData);
  const [errors, setErrors] = useState<ValidationError[]>([]);
  const [touched, setTouched] = useState<Set<string>>(new Set());
  
  const validateField = useCallback((fieldName: string, value: any) => {
    const rules = validationSchema[fieldName] || [];
    const fieldErrors: ValidationError[] = [];
    
    for (const rule of rules) {
      if (!validateRule(value, rule)) {
        fieldErrors.push({
          field: fieldName,
          message: rule.message,
          type: rule.type
        });
      }
    }
    
    setErrors(prev => [
      ...prev.filter(error => error.field !== fieldName),
      ...fieldErrors
    ]);
    
    return fieldErrors.length === 0;
  }, [validationSchema]);
  
  const updateField = useCallback((fieldName: string, value: any) => {
    setData(prev => ({ ...prev, [fieldName]: value }));
    
    if (touched.has(fieldName)) {
      validateField(fieldName, value);
    }
  }, [touched, validateField]);
  
  const markFieldTouched = useCallback((fieldName: string) => {
    setTouched(prev => new Set([...prev, fieldName]));
    validateField(fieldName, data[fieldName as keyof T]);
  }, [data, validateField]);
  
  return {
    data,
    errors,
    touched,
    updateField,
    markFieldTouched,
    isValid: errors.length === 0,
    validate: () => {
      const allErrors: ValidationError[] = [];
      Object.keys(validationSchema).forEach(fieldName => {
        const fieldErrors = validateRule(data[fieldName as keyof T], validationSchema[fieldName]);
        allErrors.push(...fieldErrors);
      });
      setErrors(allErrors);
      return allErrors.length === 0;
    }
  };
};
```