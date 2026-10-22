## JUH - 요청, 리소스, SQL, 부하테스트 가벼운 통합 모니터링 


### 모니터링, 부하테스트 하려고, 도대체 몇 개를 더 설치해야 하나요??

서버 지표 수집하려고 Prometheus 추가하고, 그래프 보려고 Grafana 연결하고, 요청 흐름 추적하려고 Zipkin을 설정하고, 느린 SQL 찾으려고 또 다른 도구 붙이고, 부하테스트 결과 편하게 보려고 뭐 또 추가하고

### 모니터링 좀 해보려다가 설정만 하다 하루가 다 가셨나요?

### 이제 그만 추가하세요. JUH 하나면 됩니다.

**JUH는 이 모든 성능 정보를 하나의 대시보드로 통합했습니다.**


![JUH Dashboard](assets/juh_main.png)

의존성 하나로 시작하는 통합 성능 모니터링.
현재 최신 버전 : implementation('io.github.gangjonggeun:juh-spring-boot-starter:0.1.10')


맨아래 필독 확인해주세요.
##

물론 각각 훌륭한 도구입니다. 하지만 연결하고 또 창 옮겨가며 확인하고 설정하다가 지쳐 만들어보았습니다.




| 기능 | 확인할 수 있는 정보 |
|---|---|
| 📊 통합 대시보드 | 현재 처리율(RPS), 최대 처리율, P95 응답 시간, 오류율, 응답 시간 추이 그래프, 리소스 사용량 그래프, 느린 요청 및 오류 요청, 쿼리 P95 |
| 🔍 요청 추적 | 최신 요청·느린 요청·오류 요청 조회, 전체 요청 응답 시간, Controller·Service·Repository 등 계층별 실행 시간, 해당 요청 개별 SQL 실행 시간, 실패한 요청의 ERROR이유, 기타 요청 정보 |
| ℹ️ 기타 요청 정보 | 캐시 사용 여부, n+1위험도, 외부 api 사용 등 부가 정보 |
| 🗄️ SQL 분석 | P50·P95·P99 쿼리 실행 시간, 최대·누적·평균 실행 시간, 호출 횟수, 쿼리별 실행 통계, N+1 쿼리 감지 |
| 💻 서버 리소스 | 시스템 CPU, 프로세스 CPU, 힙 메모리, 가비지 컬렉션, 스레드 상태, 데이터베이스 커넥션 풀 상태 |
| ⚡ 부하테스트 | 별도 연동 없이 부하테스트 자동 감지, 테스트별 결과 분리 및 저장, 테스트별 처리율·응답 시간·오류율 분석, 요청·SQL·리소스 지표 연계 분석 |
| 🧩 JUH 자체 리소스 | JUH의 CPU 사용량 추정치, 메모리 사용량 추정치, 큐 포화로 인한 트레이스 드롭 발생 정보 |

##
## JUH 설명
(*이부분은 수정시 삭제. 모든 이미지가 전체적으로 크던데 작게 배치할까 좌우 배치나 표에 배치나 이런방식으로 가능한가?)
![JUH 대쉬보드](assets/juh_dashboard_resource.png)
![JUH 대쉬보드](assets/juh_dashboard_overview.png)
*요청, 리소스, SQL 성능 정보를 한곳에서 확인하세요.*

![JUH 요청 탭](assets/juh_request_1.png)

(*이부분은 수정시 삭제. 아래 두개는 작은 이미지로 좌우에 두고싶음)
![JUH 요청 자세히보기](assets/juh_request_detail_1.png)
![JUH 요청 자세히보기2](assets/juh_request_detail_2.png)

![JUH 쿼리](assets/juh_sql.png)

![JUH 부하테스트](assets/juh_ptest_1.png)
![JUH 부하테스트](assets/juh_ptest_2.png)
![JUH 부하테스트](assets/juh_ptest_3.png)

![JUH가 사용하는 리소스](assets/juh_resource.png)


## 
## 데이터의 정합성

(*이부분은 수정시 삭제. 아래 표로 만들어줘)
 구간별 정합성 평균 1:1 비교표
계측 항목	코드 실측치 (JSON)	APM 에이전트 측정치	오차	판정
findByUserId()	144.03 ms	143.01 ms	-1.02 ms	일치 (99.3%)
userRepository.findById()	14.59 ms	13.32 ms	-1.27 ms	일치 (89.1%)
refreshTokenRepository.save()	8.40 ms	8.03 ms	-0.37 ms	일치 (%)
userEconomyRepository.findById()	51.57 ms	49.13 ms	-2.43 ms	일치 (%)
Controller 순수 실행	392.11 ms	397.13 ms	+5.02 ms	일치 (%)
SQL 쿼리 개수	4건	4건	0건	100% 일치
총 JDBC 쿼리 실행 시간	-	90 ms (44+1+43+2)


 Service(303ms) vs 커넥션 점유(379ms) 차이가 발생하는 이유
이유: 스프링의 @Transactional 동작 방식 때문입니다.
executeRefreshFlow 비즈니스 로직 자체는 303ms에 끝났지만, 메서드가 종료된 후 스프링의 트랜잭션 관리자가 **DB Commit 및 Dirty Checking(변경 감지)에 따른 UPDATE 쿼리 전송(Query 4: 2ms)**을 처리하고 커넥션을 풀에 반환하기까지 시간이 더 소요됩니다.
따라서 **[커넥션 점유 시간(379ms) > 서비스 메서드 순수 실행 시간(303ms)]**으로 측정된 것은 Spring Data JPA의 실제 물리적 수명 주기와 완전히 부합하는 정확한 데이터입니다.


응답시간 네트워크/직렬화 오버헤드 차이 발생 가능

부하테스트 

 전체 데이터 종합 정합성 대조표 (k6 vs JUH APM)
분류	세부 측정 항목	k6 클라이언트 실측	JUH APM 측정치	오차 및 차이	정합성 판정 및 원인 분석
요청 수	총 요청 수 (Total)	3,951 건	3,951 건	0 건 (0.0%)	100% 일치 (totalRequests.increment())
성공 요청 수	3,470 건	3,470 건	0 건 (0.0%)	100% 일치
실패 요청 수	481 건	481 건	0 건 (0.0%)	100% 일치 (errorCount.increment())
에러율	HTTP 실패율 (Error Rate)	12.17 %	12.17 % (12.174%)	0.00 %	100% 일치 (소수점 셋째 자리까지 일치)
처리량	초당 처리량 (RPS)	87.66 req/s	90.94 req/s	+3.28 req/s	정상 (계산 기준 차이)
• k6: 유예 종료 포함 45.1초 기준 (
3951
/
45.1
=
87.6
3951/45.1=87.6
)
• APM: 순수 요청 인입 43.4초 기준 (
3951
/
43.44
=
90.94
3951/43.44=90.94
)
지연시간	중위값 (P50 / Median)	132.36 ms	126.0 ms	-6.36 ms	95.2% 일치 (클라이언트 소켓 RTT 지연 6ms)
P90	225.65 ms	-	-	k6 전용 지표
P95 (핵심 백분위)	273.16 ms	260.0 ms	-13.16 ms	95.2% 일치 (클라이언트 RTT 13ms 반영)
P99	-	381.0 ms	-	부하 최고조 시점(Pending 5개) 정확 포착
최대 지연 (Max)	1,500.0 ms (1.5s)	1,498.0 ms	-2.0 ms	99.9% 일치 (최악의 아웃라이어 단 1건 완벽 포착)
평균 응답시간 (Avg)	143.81 ms	143.0 ms 추정	-0.81 ms	99.4% 일치

 DB 및 SQL 계층 정합성 대조표 (트랜잭션 무결성 검증)
LoadTestDetectionService의 queryStatistics.recordQuery()가 트레이스 단위로 수집한 SQL 데이터는 Spring Data JPA의 트랜잭션 롤백과 1건의 오차도 없이 일치합니다.
쿼리 번호 및 내용	k6 기대 실행 건수	APM 실제 수집 건수	SQL 오류 건수	정합성 검증 공식
01. insert into users	3,951 건	3,951 건	481 건	k6 실패 수(481)와 정확히 1:1 일치
02. select refresh_tokens	3,470 건 (성공건)	3,470 건	0 건	
3
,
951
−
481
=
3
,
470
3,951−481=3,470
 (성공 트랜잭션만 실행)
03. insert into messages	3,470 건	3,470 건	0 건	100% 일치
04. insert into refresh_tokens	3,470 건	3,470 건	0 건	100% 일치
05. select oil_0...	3,470 건	3,470 건	0 건	100% 일치
06. select users (중복 조회)	7,902 건	7,902 건	0 건	
3
,
951
×
2
=
7
,
902
3,951×2=7,902
 (요청당 2회 무조건 실행)
07~09. 부가 정보 INSERT 3종	3,470 건씩	3,470 건씩	0 건	100% 일치
10. insert user_card_items	10,410 건	10,410 건	0 건	
3
,
470
×
3
장
=
10
,
410
3,470×3장=10,410

=>네트워크 왕복 시간(RTT)과 커넥션 연결 오버헤드가 포함되어 약 6~10ms 더 길게 나올 수 있다고 알려주자



## 필독
1. 제 운영서버 요청이 많이 없어 실제 운영단계 테스트가 많이 부족합니다. 로컬 테스트를 통해 확인해본결과로는 overview(개요)에서 약간의 딜레이가 발생하는건 확인 신속히 고치겠다.

2. 네트워크 왕복 시간(RTT)과 커넥션 연결 오버헤드가 포함되어 약 6~10ms 더 길게 나올 수 있음

## 설정
1. 로그인 및 로컬환경(부하테스트를 자동 감지) 설정
security설정	기본값	  설명
enabled	false	JUH 대시보드 및 API 보안 활성화 여부 true시 로그인 활성
username	admin	대시보드 관리자 아이디
password	juh_admin_pw	대시보드 관리자 비밀번호
local	false	엔드포인트 기반 자동 부하 테스트 감지 활성화 여부 (로컬에서 부하테스트시 true)

load-test설정 항목	기본값	의미
idle-timeout-seconds	15초	요청이 없을 때 테스트 세션을 종료하는 대기 시간
min-requests	250	일반 테스트의 최소 요청 수
min-rps	10.0	일반 테스트의 최소 평균 초당 요청 수
soak-min-requests	1000	Soak 테스트의 최소 요청 수
soak-min-duration-seconds	60초	Soak 테스트의 최소 지속 시간
spike-min-requests	50	Spike 테스트의 최소 요청 수
spike-min-rps	50.0	Spike 테스트의 최소 평균 초당 요청 수
explicit-min-requests	5	테스트 이름 헤더로 명시한 테스트의 최소 요청 수



yml
```yaml
juh:
  security:
    # JUH true시 로그인 활성화 
    enabled: false
    # 대시보드 관리자 계정
    username: admin
    password: juh_admin_pw
    
    # 로컬 환경의 자동 부하 테스트 감지 활성화 여부(로컬로 부하시 true)
    local: false

  load-test:
    # 요청이 없을 때 테스트 세션 종료까지 대기하는 시간(초)
    idle-timeout-seconds: 15

    # 일반 부하 테스트 감지 기준
    min-requests: 250
    min-rps: 10.0

    # 장시간 지속되는 부하 테스트(Soak Test) 감지 기준
    soak-min-requests: 1000
    soak-min-duration-seconds: 60

    # 짧고 강한 부하 테스트(Spike Test) 감지 기준
    spike-min-requests: 50
    spike-min-rps: 50.0

    # X-JUH-Test-Name 헤더로 명시한 테스트의 최소 요청 수
    explicit-min-requests: 5
```


properties
```properties
# ===== JUH Security =====
# JUH true시 로그인 활성화
juh.security.enabled=false
# 대시보드 관리자 계정
juh.security.username=admin
juh.security.password=juh_admin_pw
# 로컬 환경의 자동 부하 테스트 감지 활성화 여부(로컬로 부하시 true)
juh.security.local=false

# ===== JUH Load Test Detection =====
# 요청이 없을 때 테스트 세션 종료까지 대기하는 시간(초)
juh.load-test.idle-timeout-seconds=15

# 일반 부하 테스트 감지 기준
juh.load-test.min-requests=250
juh.load-test.min-rps=10.0

# 장시간 지속되는 부하 테스트(Soak Test) 감지 기준
juh.load-test.soak-min-requests=1000
juh.load-test.soak-min-duration-seconds=60

# 짧고 강한 부하 테스트(Spike Test) 감지 기준
juh.load-test.spike-min-requests=50
juh.load-test.spike-min-rps=50.0

# X-JUH-Test-Name 헤더로 명시한 테스트의 최소 요청 수
juh.load-test.explicit-min-requests=5
```



