## JUH - 요청, 리소스, SQL, 부하테스트 가벼운 통합 모니터링 


### 모니터링, 부하테스트 하려고, 도대체 몇 개를 더 설치해야 하나요??

서버 지표 수집하려고 Prometheus 추가하고, 그래프 보려고 Grafana 연결하고, 요청 흐름 추적하려고 Zipkin을 설정하고, 느린 SQL 찾으려고 또 다른 도구 붙이고, 부하테스트 결과 편하게 보려고 뭐 또 추가하고

### 모니터링 좀 해보려다가 설정만 하다 하루가 다 가셨나요?

### 이제 그만 추가하세요. JUH 하나면 됩니다.

**JUH는 이 모든 성능 정보를 하나의 대시보드로 통합했습니다.**


![JUH Dashboard](assets/juh_main.png)

의존성 하나로 시작하는 통합 성능 모니터링.
현재 최신 버전 : implementation('io.github.gangjonggeun:juh-spring-boot-starter:0.1.10')


(맨아래 필독 확인해주세요.)
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



