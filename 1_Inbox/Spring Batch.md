tag: #SpringBatch 
### JOB (작업)
* 정의: 배치 처리 과정 전체를 하나의 단위로 묶은 최상위 개념
* Job의 구성요소:
	* [[#JobInstance (작업 인스턴스)]]
	* [[#JobParameters (작업 매개변수)]]
	* [[#JobExecution / StepExecution (실행 상태)|JobExecution (실행 상태)]]
	* [[#Step (단계)]]
* Job을 빌드하는 방법:
	* **JobBuilder 사용**: `new JobBuilder("작업명", jobRepository)`를 통해 직접 객체 생성 (`JobBuilderFactory`는 폐지됨 *(SpringBatch5.0 이후)* )
    * **Step 연결**: 생성된 빌더에 `.start(stepA).next(stepB).build()` 형태로 단계를 순차적으로 연결하여 구성
### Step (단계)
* 정의: Job을 구성하는 독립적인 작업단위
* 관계: Job은 N개의 Step을 가질 수 있음
* `RepeateStatus
	* 메서드 실행을 계속할지 멈출지 결정하는 기준
	* **Chunk 방식**: ItemReader가 null을 반환하면 배치가 알아서 종료하므로 자동으로 제어됨 
	* **Tasklet 방식**: 개발자가 루프를 제어해야 하므로 메서드 반환값으로 명시적으로 알려줘야 함
	* **주요 값**:
		* `CONTINUABLE`
			* 현재 트랜잭션을 커밋하고, 즉시 다음 execute() 메서드를 다시 반복 호출 (짧은 트랜잭션으로 쪼개서 대량 처리할 때 유용).
		* `FINISHED`
			* 작업을 끝내고 다음 Step으로 넘어가거나 Job을 정상 종료함.
### Tasklet vs Chunk (작업 처리 방식)
* 정의: Step을 실행하는 두 가지 핵심 스타일
* Chunk
	* 트랜잭션은 청크 단위로 분리
### JobInstance (작업 인스턴스)
* 정의: Job이 실제 실행될 때 논리적으로 생성되는 고유한 실행 객체
* 동일한 jobName + 동일한 JobParameter = **~={green}JobInstance=~**
### JobExecution / StepExecution (실행 상태)
* 정의: Job과 Step이 실제로 실행된 '시도(Attempt)와 결과 상태를 담은 객체'
### JobParameters (작업 매개변수)
* 정의: 배치 실행할 때 외부에서 주입하는 파라미터 (타입: String, Long, Double, Date)
* ~={yellow}**JobInstance를 구분하는 기준이 됨**=~
### JobRepository (저장소)
* 정의: 배치가 실행되는 모든 메타데이터 (Job 상태, 성공 여부, 실행 시간 등)를 DB에 저장하고 관리하는 관리자역할
* 키워드: `BATCH_JOB_INSTANCE`, `BATCH_JOB_EXECUTION`
### [[JobOperator|JobOperator]] & [[JobOperator|JobLauncher]](실행기)
* 정의: 배치를 구동하고 제어하는 실행 인터페이스
### ItemReader ➔ ItemProcessor ➔ ItemWriter (Chunk 지향)
* 정의: Chnk지향 Step에서 데이터를 가공하는 3단계 핵심 컴포넌트
* **ItemReader**
	* ItemReader는 데이터 소스(DB, 파일 등)에서 데이터를 하나씩 순차적으로 읽어옴
	* 더 이상 읽을 데이터가 없으면 null반환하고 스텝 종료
	* 청크 크기가 10이면 `read()` 메서드 자체는 10번 호출되지만, 내부 페이징/커서 메커니즘 덕분에 실제 물리적인 DB 쿼리가 10번 나가는 것은 아님.
* **ItemProssor**
	* 형태: `O process(I item) throws Exception`
	* 생략 가능 (필수x)
	* 데이터 가공
		* 입력 데이터(`I`)를 원하는 형태(`O`)로 **변환**한다. 예를 들어, 읽어온 원본 데이터를 비즈니스 로직에 맞게 가공하거나, 출력 시스템이 요구하는 형식으로 변환하는 작업
	* 필터링
		* 프로세서가 null을 반환하면 해당 입력 데이터는 Writer로 전달되지 않음
	* 데이터 검증
		* 필터링과 달리 조건에 맞지 않는 데이터를 만나면 예외를 발생
			* ex: 필수 필드 누락, 잘못된 데이터 형식을 발견했을 때, 예외를 던져 배치 잡 중단
		* Skip기능을 사용하면 예외가 발생한 데이터만 건너띄고 배치 작업 계속 진행 가능
* **ItemWriter**
	* 형태: `void write(Chunk<? extends T> chunk) throws Exception;`
	* ItemProssor가 만든 결과물을 받아, 최종 저장/출력
		* DB에 **Insert** / File에 **Write** / 메시지 큐에 **Push**
	* Chunk 단위로 쓴다
		* Chunk 단위로 묶어서 한번에 DB에 **Insert** / File에 **Write** / 메시지 큐에 **Push**
### ExecutionContext(공유 저장소)
* 정의: 배치가 도는 동안 상태를 유지하고, 다른 Step이나 컴포넌트끼리 데이터를 공유하기 위해 사용하는 키-값 저장소