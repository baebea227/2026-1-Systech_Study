## 1. 실습 개요

- 차세대 스토리지 기술인 ZNS(Zoned Namespace) SSD의 핵심 구조 및 동작 원리를 이해하고, 기존 NVMe SSD와의 성능, 상태 전이, 수명(WAF) 차이를 직접 비교 및 분석합니다.

## 2. 실습 환경

- **운영체제 (OS):** Linux Kernel 5.18.0
- **가상화 환경:** QEMU

**[주요 실습 도구]**

- **fio:** 스토리지 I/O 성능 측정 및 워크로드 인가 테스트를 수행합니다.
- **nvme-cli:** NVMe 디바이스를 제어하고 SMART 로그 및 상태 정보를 확인합니다.
- **blkzone:** 스토리지의 Zone 관리 명령어를 수행하고 세부 상태를 모니터링합니다.

## **3. 단계별 실습 상세 계획**

### **Lab 1 — 쓰기 제약 및 기본 성능 비교**

- **Step 1.1 [순차 쓰기 성능 비교]:** ZNS 및 Conventional SSD에 동일한 조건의 순차 쓰기 워크로드를 인가하여 처리량(BW, Bandwidth)과 평균 지연 시간(Latency)을 측정하고 비교합니다.
- **Step 1.2 [랜덤 쓰기 예외 처리 확인]:** ZNS 스토리지에 `zonemode=none` 옵션을 주어 랜덤 쓰기를 시도합니다. 이때 발생하는 Write Pointer 위반 오류 로그를 확인하고, 정상적으로 랜덤 쓰기를 처리하는 Conventional SSD의 동작과 대조하여 분석합니다.

### **Lab 2 — Zone 관리 상태 전이**

- **Step 2.1 [Zone 상태 조회 및 자동 전이]:** `blkzone report` 명령어를 사용하여 전체 Zone의 현재 상태(EMPTY, Implicit Open, Explicit Open, Closed, Full)와 Write Pointer 위치를 파악합니다. 이후 명시적인 Open 명령 없이 쓰기를 수행하여, Zone이 `Implicit Open (io)` 상태로 자동 전이되는 과정을 확인합니다.
- **Step 2.2 [명시적 수명주기 제어]:** `zone_lifecycle.sh` 스크립트를 구동하여 특정 Zone(예: Zone 1)에 대해 `Explicit Open` ➔ `Write` ➔ `Close` ➔ `Finish` 단계를 순서대로 실행하며, 각 단계마다 Write Pointer가 어떻게 이동하고 고정되는지 관찰합니다.
- **Step 2.3 [Zone 초기화]:** `zone_reset.sh` 스크립트를 사용하여 용량이 가득 찬 `FULL` 상태의 Zone을 Reset 합니다. Write Pointer가 초기화되고 Zone의 상태가 다시 `EMPTY`로 성공적으로 복구되는지 검증합니다.

### **Lab 3 — F2FS Host-visible WAF 측정과 한계**

- **Step 3.1 [Conv/Zoned F2FS 환경 준비]:** `cleanup.sh`와 `setup.sh conv|zoned`를 사용하여 3 GB `null_blk` 장치를 Conventional SSD 또는 Zoned F2FS 실험용 장치로 순차 구성하고 F2FS를 포맷·마운트합니다.
- **Step 3.2 [GC 압박 워크로드 인가]:** `bench.sh conv|zoned`를 실행하여 적응형 fill, fragment, 측정 write를 수행하고 `/sys/block/nullb*/stat`의 `write_sectors` 변화량과 `fio` logical write bytes를 함께 수집합니다.
- **Step 3.3 [proxy WAF와 한계 분석]:** host-visible WAF를 `block-layer submitted write bytes / fio logical write bytes`로 계산하고, `null_blk` 환경에서는 실제 flash/NAND 기록량을 볼 수 없으므로 proxy WAF와 device-internal WAF를 구분해 해석합니다.

### **Lab 4 — ZUFS 논문 스터디**

- **[스토리지 패러다임 변화 분석]:** 기존 스마트폰에 쓰이는 CUFS(Conventional UFS)가 겪는 L2P(Logical-to-Physical) 매핑 테이블 SRAM 오버헤드 문제를 분석하고, ZUFS가 순차 쓰기 제약을 통해 1 TB 기준 매핑 크기를 약 1 GB에서 약 8 KB 수준으로 줄이는 원리를 학습합니다.
- **[크로스 레이어 최적화 토론]:** 모바일 환경의 3대 과제(제한된 SRAM, 쓰기 순서 보장, GC 오버헤드 완화)를 해결하기 위해 제안된 5계층(Android ➔ F2FS ➔ 블록 레이어 ➔ 드라이버 ➔ 장치 펌웨어) 협력 아키텍처를 분석합니다.
- **[실무 성능 평가 및 리뷰]:** 실제 ZUFS가 탑재된 기기에서 단편화(Fragmentation)가 심화될 때 CUFS 대비 쓰기 처리량이 2배 이상 우수하게 유지되는 원리를 살펴보고, 수명 및 게임 로딩 속도 등 사용자 체감 성능에 미치는 영향을 종합적으로 평가합니다.

## **4. 기대 효과**

- ZNS 스토리지의 순차 쓰기 제약 및 이로 인한 시스템 설계의 차이를 이해할 수 있습니다.
- 스토리지 내부의 Zone 상태 및 Write Pointer 메커니즘을 디버깅하고 제어하는 방법을 습득합니다.
- 가비지 컬렉션(GC)이 스토리지 성능 및 WAF에 미치는 영향을 데이터 기반으로 입증할 수 있습니다.
- ZUFS의 크로스 레이어(Cross-Layer) 최적화를 통해 모바일 스토리지의 자원 제약 극복 및 체감 성능 향상을 종합적으로 이해합니다.

## 5. 참고자료

https://www.usenix.org/conference/fast26/presentation/kim-jungae
