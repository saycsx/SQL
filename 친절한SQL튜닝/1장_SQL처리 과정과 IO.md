# 1장 SQL 처리 과정과 I/O

## 1.1장 SQL 파싱과 최적화
#### SQL 최적화 파싱
> SQL 파싱 : SQL을 전달받으면, 가장 먼저 SQL 파서가 파싱을 진행
  - 파싱 트리 생성 : SQL문을 이루는 개별 구성요소를 분석해서 파싱 트리 생성
  - Syntax 체크 : 문법적 오류 체크 ex) 누락된 키워드 및 사용할 수 없는 키워드
  - Semantic 체크 : 의미상 오류가 없는지 체크 ex) 존재하지 않는 테이블, 컬럼, 권한
  a. SQL 최적화 : SQL 옵티마이저는 미리 수집한 시스템 및 오브젝트 통계정보를 바탕으로 다양한 실행경로를 생성해서 비교한 후, 가장 효율적인 하나를 선택 --> 데이터 베이스 성능을 결정하는 가장 핵심적인 엔진 (옵티마이저가 이 역할을 맡음)
   - 일반적으로 처리 과정을 세세하게 설명할 목적이 아니면 일반적으로 이 둘을 구분할 필요는 없음
  b. 로우 소스 생성 : SQL 옵티마이저가 선택한 실행경로를 실제 실행 가능한 코드 또는 프로시저 형태로 포맷팅하는 단계(로우 소스 생성기가 이 역할을 맡음)
> SQL 옵티마이저
> 옵티마이저 최적화 단계. 
  
  a. 전달받은 쿼리를 수행하는데 후보군이 될만한 실행계획을 찾음   
  b. 데이터 딕셔너리에 미리 수집해 둔 오브젝트 통계 및 시스템 통계정보를 이용해 각 실행계획 및 예상비용을 산정  
  c. 최저 비용을 나타내는 실행계획을 선택
> 실행계획과 비용
  - SQL 옵티마이저는 자동차 네비게이션과 비슷(경로 요약,모의 주행)
  - DBMS에 SQL 실행경로 미리보기 기능이 있음. 이것이 실행 계획
  - 실행 계획에 표시되는 Cost도 어디까지나 예상치. 실행경로를 선택하기 위해 옵티마이저가 여러 통계정보를 활용해서 계산해 낸 값
  - ** 비용 : 쿼리를 수행하는 동안 발생할 것으로 예상하는 I/O 횟수 또는 예상 소요시간을 표현한 값
> 옵티마이저 힌트
  - 힌트
    - SQL 옵티마이저는 대부분 좋은 선택을 하지만, 완벽하지 않음
    - 옵티마이저 힌트를 이용해 데이터 액세스 경로를 바꿀 수 있음
    - /*+ INDEX(A 고객_PK) */
> 주의사항
  - 힌트 안에 인자를 나열할 땐 ,(콤마)를 사용할 수 있지만, 힌트와 힌트사이에는 불가
  - 스키마명을 명시하면 안됨
  - From 절에서 AS 사용 시 AS명으로 사용해야 함
  - 기왕에 힌트를 쓸 거면, 빈틈없이 기술해야 함


## 1.3장 데이터 저장 구조 및 I/O 메커니즘

> ### 1.35 논리적 I/O vs 물리적 I/O


#### DB 버퍼캐시
  - 데이터 캐시라고 할 수 있음
  - 어렵게 읽은 데이터 블록을 캐싱해 둠으로써 같은 블록에 대한 반복적인 I/O Call을 줄일 수 있음
  - 서버 프로세스와 데이터 파일 사이에는 버퍼 캐시가 있으므로 데이터 블록을 읽을 땐 항상 버퍼캐시부터 탐색
  - 버퍼캐시는 공유메모리 영역이므로 같은 블록을 읽는 다른 프로세스도 득을 봄
  - 버퍼캐시 사이즈 확인 : show sga
#### 논리적 I/O vs 물리적 I/O
  - 논리적 I/O : SQL문을 처리하는 과정에서 메모리 버퍼캐시에서 발생한 총 블록 I/O
  - 물리적 I/O : 버퍼캐시에서 블록을 찾지 못해 디스크에서 발생한 총 블록 I/O
  ** 메모리 I/O는 전기적 신호인데 반해, 디스크 I/O는 물리적 작용이므로 보통 10,000배 쯤 느림
#### 버퍼캐시 히트율
  - 버퍼캐시 효율을 측정하는 전통적인 지표
  - BCHR = (캐시에서 곧바로 찾은 블록 수 / 총 읽은 블록수)x100   
         = ((논리적I/O - 물리적 I/O) x 100
  - 읽은 전체 블록 중 물리적인 디스크I/O를 수반하지 않고 곧바로 메모리에서 찾은 비율을 나타냄    
  ** 온라인 트랜잭션을 주로 처리하는 앱에서는 평균 99% 히트율을 달성해야 함
  - 물리적 I/O = 논리적I/O x (100-BCHR) 이므로 결과적으로 논리적 I/O를 줄여야 함

> ### 1.36 Single Block I/O vs Multiblock I/O
  - 데이터 모두를 캐시에 적재할 수 없음. 비용,기술적 한계로 일부만 캐시에 적재해서 읽을 수 있음
#### Single Block I/O
  - 한번에 한 블록씩 요청해서 메모리에 적재하는 방식
  - 인덱스를 이용할 때 기본적으로 인덱스와 테이블 블록 모두 Single Block I/O 방식 이용
#### Multi Block I/O
  - 특정 블록을 읽으려고 I/O Call 할 때, 디스크 상에 그 블록과 인접한 블록들을 한꺼번에 읽어 캐시에 미리 적재하는 기능
  - "인접한" 블록은 같은 익스텐트에 속한 블록(익스텐트 경계를 넘지 못함)
  - 테이블을 Full Scan할 때, Multiblock I/O 단위를 크게 설정하면, 프로세스가 잠자는 횟수를 줄이고 성능을 높일 수 있음
  - DBMS 블록 사이즈가 얼마건 간에 OS 단에서는 보통 1MB 단위로 I/O를 수행한다.(OS마다 다름)


> ### 1.37 Table Full Scan vs Index Range Scan
  - Table Full Scan은 피해야한다는 많은 개발자의 인식과 달리 인덱스가 SQL 성능을 떨어뜨리는 경우도 상당히 많음


#### Table Full Scan
  - 테이블에 속한 블록 전체를 읽어서 사용자가 원하는 데이터를 찾는 방식
  - 시퀀셜 액세스와 Multibloc I/O 방식으로 디스크 블록을 읽음
  - 한번에 많은 데이터를 처리하는 집계용 SQL과 배치 프로그램에 유리
  - 상당수가 Table Full Scan으로 유도하면 성능이 좋아짐. 조인을 포함한 SQL이면, 조인 메소드로 해시 조인을 선택해주면 됨(테이블 스캔이 항상 나쁜게 아님)

#### Index Range Scan
  - 인덱스에서 일정량을 스캔하면서 얻은 ROWID로 테이블 레코드를 찾아가는 방식
  - 랜덤 액세스와 Single I/O 방식으로 디스크 블록을 읽음
  - 캐시에서 블록을 찾지 못하면 레코드 하나를 읽기위해 매번 잠을 자는 I/O 메커니즘
  - 큰 테이블에서 소량 데이터를 검색할 떄는 반드시 인덱스를 이용


> ### 1.38 캐시 탐색 메커니즘
  - Direct Path I/O를 제외한 모든 블록 I/O는 메모리 버퍼캐시를 경유 함
  - 버퍼캐시에서 블록을 찾을 때 해시 알고리즘으로 버퍼 헤더를 찾고, 거기서 얻은 포인터로 버퍼 블록을 액세스하는 방식을 사용


#### 해시 구조의 특징
  - 같은 입력 값은 항상 동일한 해시 체인(=버킷)에 연결
  - 다른 입력 값이 동일한 해시 체인에 연결될 수 있음
  - 해시 체인 내에서는 정렬이 보장되지 않음

#### 메모리 공유자원에 대한 액세스 직렬화
  - 버퍼캐시에 캐싱된 버퍼블록은 모두 공유자원. 누구나 접근 가능
  - 버퍼 블록에 두 개 이상 프로세스가 "동시에"접근하면 블록 정합성에 문제가 생길 수 있음
  - 따라서, 내부에선 한 프로세스씩 순차적으로 접근하도록 구현해야 하며, 이를 위해 직렬화(줄 세우기)메커니즘이 필요
  - 이런 줄서기가 가능하도록 지원하는 메커니즘이 "래치"
  - SGA(SystemGlobalArea)를 구성하는 서브 캐시마다 별도의 래치가 존재
  - 래치에 의한 경합이 생길 수 있기 때문에 캐시 I/O도 생각만큼 빠르지 않을 수 있음
  - 캐시버퍼 체인뿐만 아니라 버퍼블록 자체에도 직렬화 메커니즘이 존재. 바로 "버퍼 Lock"
  - 직렬화 메커니즘에 의한 캐시 경합을 줄이려면 결국 SQL 튜닝을 통해 쿼리 일량(논리적 I/O) 자체를 줄여야 함

#### 캐시버퍼 체인 래치
  - 해시 체인을 스캔하는 동안 다른 프로세스가 체인 구조를 변경하는 일을 막기 위해 해시 체인 래치가 존재
  - 체인 앞쪽에 자물쇠가 있고, 자물쇠를 열 수 있는 키를 획득한 프로세스만이 체인으로 진입할 수 있음

#### 버퍼 Lock
  - 읽고자하는 블록을 찾았으면 캐시버퍼 체인 래치를 곧바로 해제. 그래야 래치가 풀리길 기다리던 프로세스들이 작업을 재개할 수 있음
  - 래치를 해제한 상태로 버퍼블록 데이터를 읽고 쓰는 도중에 후행 프로세스가 하필 같은 블록에 접근해서 데이터를 읽고 쓴다면 데이터 정합성에 문제가 생길 수 있는데, 이를 방지하기 위해 오라클은 버퍼 Lock을 사용
  - 캐시버퍼 체인 래치를 해제하기 전에 버퍼 헤더에 Lock을 설정함으로써 버퍼블록 자체에 대한 직렬화 문제를 해결    
  ** 같은 로우에서 로우 Lock을 설정하는 행위도 블록을 변경하는 작업. 로우 Lock을 설정하는 순간 다른 프로세스에서 해당 블록을 읽는다면 문제가 생김. 그래서 버퍼 Lock이 필요 
