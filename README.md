# 🚘 UsUaly (유즐리) — Co-Ride Wrapped
> 🚀 **웹 브라우저 즉시 시연 데모 (Live HMI & Virtual Console)**: [https://haklip.github.io/UsUaly/UsUaly_V1.html](https://haklip.github.io/UsUaly/UsUaly_V1.html)
> *(별도 설치 없이 위 링크를 클릭하면 브라우저에서 좌석 센서·도어·DTW 궤적 매칭 및 보딩패스 생성을 즉시 테스트하실 수 있습니다.)*

> **“평범한 우리의 일상이 특별한 기록이 된다 (US + USUALLY)”**
>
> [**현대자동차그룹 차량용 앱 개발 공모전 (Code your App into Pleos App Market)**](https://appdevchallenge.auto-hmg.io/) **출품작**
>
> Pleos Connect SDK 기반 무자각(Zero-Touch) 동승자 자동 식별 및 연간 모빌리티 회고·소셜 라이프로그 서비스

UsUaly(유즐리)는 기존 커넥티드카 앱이 연비·전비·총 주행거리 등 운전자 1인의 기계적 수치에만 머물렀던 한계를 넘어, **Pleos Connect SDK의 실내 좌석(Seat)·도어(Door)·주행거리계(Odometer)·내비게이션(NaviHelper) 센서 데이터와 클라우드 2단계 시공간·궤적(Trajectory) 매칭 엔진**을 결합하여 “운전자가 아무런 조작을 하지 않아도(Zero-Touch) 지난 1년간 내 옆자리를 가장 오래 지킨 소중한 사람들과의 거리($\Delta D$)와 시간($\Delta T$)을 자동으로 기록·시각화하고, 오늘의 운행 리캡·동승 마일리지 양도·추억의 코스 재주행으로 연결하는 인카(In-Car) 소셜 라이프로그 서비스”입니다.

> **🔒 심사 유의사항 (블라인드 심사 준수)**
>
> 본 저장소는 [현대자동차그룹 차량용 앱 개발 공모전](https://appdevchallenge.auto-hmg.io/) 심사 규정에 따라 참가자의 실명, 소속(학교/학과), 연락처 등 개인 식별 정보를 일절 포함하지 않은 블라인드 기술 문서 및 프로토타입 저장소입니다.

## ✨ 1. 기획 배경 및 해결하고자 하는 문제 (Why UsUaly?)

1. **운전자 1인·기계적 수치에만 머무른 기존 모빌리티 기록의 한계**

   * 자동차는 집과 일터를 제외하고 가족·연인·동료·친구와 가장 깊은 대화를 나누는 사적 공간이지만, 기존 차량 리포트는 기계적 주행 지표만 남길 뿐 \*“지난 1년 동안 누구와 어디를 얼마나 함께 달렸는가”\*라는 관계의 맥락은 시동이 꺼지는 순간 휘발됩니다.

2. **수동 기록의 피로감 및 주행 안전(Driver Distraction) 문제 해결**

   * 탑승할 때마다 화면을 터치해 동승자를 수동 입력하는 방식은 운전 주의를 분산시키고 며칠 내로 기록을 포기하게 만듭니다. 스포티파이 연말 결산(Wrapped)처럼 감동을 주려면 사용자가 의식하지 않아도 1년간 자동으로 쌓이는 **Zero-Touch 로깅 파이프라인**이 필수적입니다.

3. **왜 스마트폰 단독 앱이 아닌 ‘Pleos Connect 차량용 네이티브 앱’이어야 하는가?**

   * **물리적 동승 확증**: 스마트폰 단독 앱은 카페에 마주 앉아 있는지, 대중교통을 탔는지, 내 차에 탔는지 구분할 수 없습니다. 반면 Pleos Connect 앱은 `Seat` 착석 센서와 `Door` 개폐 센서를 통해 **내 차의 어느 좌석에 실제 탑승했는지 물리적으로 100% 확증**합니다.

   * **블루투스(BLE) 한계 및 에뮬레이터 제약 극복**: 최신 iOS·Android의 BLE MAC 주소 무작위화(Randomization) 보안 정책과 에뮬레이터의 외부 블루투스 동글 미지원 제약을 극복하기 위해, **차량 착석 트리거 + 스마트폰 저전력 모션(`IN_VEHICLE`) 전환 + 클라우드 2단계 궤적 검증** 아키텍처를 설계했습니다.

## 🏗️ 2. 전체 시스템 아키텍처 (3-Tier System Architecture)

전체 시스템은 **① In-Car Client (Pleos Connect AAOS 네이티브 앱 & 웹 시연 시뮬레이터)**, **② Cloud Match & Session Server (시공간·궤적 매칭 및 마일리지 원장 서버)**, ③ Companion Mobile Client (동승자 스마트폰 앱/PWA)의 3계층으로 동작합니다.

```
flowchart TB
    subgraph InCar["1. In-Car Client (Pleos Connect AAOS App · API 28+)"]
        direction TB
        SeatDoor["Vehicle SDK (v2.3.2)<br/>Seat · SeatBelt · Door Listener"]
        Odo["Vehicle SDK (v2.3.2)<br/>Odometer (ΔD, ΔT 차분 산출)"]
        FusedLoc["FusedLocation SDK (v1.1.2)<br/>1차(0m) · 2차(200m) 좌표 추출"]
        Navi["NaviHelper SDK (v3.0.5)<br/>getDestinationInfo / requestRouteWithWaypoints"]
        Gleo["Gleo AI Interface (assets/schema.json)<br/>3 Action Calls & TTS v3.3.1 브리핑"]
        RoomDB[("Room SQLite 로컬 캐시<br/>(터널·지하 음영지역 오프라인 큐)")]
        UI["Adaptive HMI & Digital Twin<br/>1-Window(DO 방현) / 2-Window(와이드 리포트)"]
    end

    subgraph Cloud["2. Cloud Match & Session Server (Backend)"]
        direction TB
        TrajEngine["2단계 시공간·궤적 교차 검증 엔진<br/>(PostGIS · Redis Session Store)"]
        PrivacyClip["양 끝단 500m(15%) 폴리라인 절삭기<br/>& 인카 MediaSession 플리 매칭"]
        MileageLedger["동승 마일리지 원장 (Mileage Ledger)<br/>동시 적립 · 원터치 양도 · 바우처 교환"]
        WrappedAgg["연간 리포트 집계 엔진<br/>(Dual Crown · 페르소나 군집화 · QR 티켓)"]
    end

    subgraph Mobile["3. Companion Mobile Client (iOS / Android / PWA)"]
        direction TB
        QROnboard["최초 1회 QR 메이트 연결<br/>& 개인정보 수집 동의 (Opt-In)"]
        MotionAPI["OS 저전력 모션 감지 API<br/>(ActivityRecognition IN_VEHICLE)"]
        InstaShare["9:16 인스타 스토리 & 카카오톡 공유<br/>(모빌리티 보딩패스 & 오늘의 운행 리캡)"]
    end

    SeatDoor -->|"Occupied + Belt 검증"| FusedLoc
    Odo -->|"좌석별 독립 D_start 스냅샷"| TrajEngine
    FusedLoc -->|"T=0s & ΔD≥200m 좌표"| TrajEngine
    MotionAPI -->|"IN_VEHICLE 위치 체크인"| TrajEngine
    RoomDB <-->|"네트워크 복구 시 배치 동기화"| TrajEngine
    TrajEngine -->|"매칭 확정 & 재회 브리핑"| UI
    Gleo <-->|"Broadcast / Activity 호출"| UI
    Navi <-->|"목적지 POI / 추억의 코스 재주행"| UI
    PrivacyClip -->|"안심 경로 + 플레이리스트"| InstaShare
    MileageLedger -->|"동시 적립 & 운전자 양도"| UI
    WrappedAgg -->|"QR Share Ticket"| InstaShare

```

### 계층별 세부 기술 스택

| 계층 구분 | 구성 요소 및 개발 환경 | 핵심 역할 및 처리 데이터 | 
| ----- | ----- | ----- | 
| **In-Car Client (차량용 앱)** | Android Automotive OS(AAOS) 기반 Kotlin 네이티브 앱 (`Min SDK: Android API 28`, `Gradle 8.0+`) 및 본선 라이브 시연용 통합 웹 HMI 에뮬레이터 | `Vehicle SDK`, `FusedLocation SDK`, `NaviHelper SDK` 연동, 실시간 2D 실내 디지털 트윈 렌더링, 좌석별 순수 주행 거리·시간 차분($\Delta D, \Delta T$) 독립 계산, `CarUxRestrictions` 주행 안전 다크 방현 모드 제어 | 
| **Cloud Backend (매칭·집계 서버)** | REST API & WebSocket 실시간 세션 서버 + 시공간 매칭 엔진 (`PostgreSQL / PostGIS`, `Redis`) | 차량 착석 이벤트와 메이트 기기의 차량 이동(`IN_VEHICLE`) 이벤트 간 2단계 궤적(탑승 시점 및 200m 주행 후 좌표) 교차 검증, 양 끝단 500m(15%) 프라이버시 폴리라인 절삭, 동승 마일리지 동시 적립·양도 원장 관리 | 
| **Companion Client (모바일 앱/웹)** | 운전자·동승자 스마트폰 컴패니언 앱 (iOS / Android) 및 공유용 모바일 웹(PWA) | 최초 1회 QR 코드 기반 메이트 연결 및 개인정보 동의(Opt-In), OS 저전력 모션 API(`ActivityTransition`) 기반 탑승 체크인 전송, 9:16 인스타그램 스토리 규격 보딩패스·오늘의 리캡 카드 공유 | 

## 🔄 3. 클라우드 2단계 시공간·궤적 동승자 자동 매칭 알고리즘

모바일 OS의 백그라운드 위치 조회 제한을 우회하고, 배웅 나온 친구나 좌석 위에 올려둔 가방이 동승자로 잘못 기록되는 오탐지를 완벽히 차단하기 위해 **‘양방향 이벤트 체크인 + 2단계 이동 궤적 검증 + 엣지 케이스 방어’** 파이프라인을 구현했습니다.

```
sequenceDiagram
    autonumber
    participant Car as 🚘 차량 (Pleos Vehicle/FusedLocation SDK)
    participant Server as ☁️ 클라우드 시공간 매칭 서버 (PostGIS/Redis)
    participant Mate as 📱 동승 메이트 스마트폰 (Companion App)

    Note over Car,Mate: [사전 온보딩] 최초 1회 초대 QR 스캔으로 메이트 등록 및 위치 공유 동의 완료
    Car->>Car: Door 개폐 및 SeatOccupancyListener(Occupied) 감지
    Car->>Car: checkSeatBeltBuckledCapabilities() 교차 검증 (가방·짐 오탐지 차단)
    Car->>Car: 좌석별 기준 주행거리 D_start[seat] 및 T_start[seat] 독립 스냅샷 기록
    Car->>Server: 1차 탑승 이벤트 전송 (Lat_car_0, Lng_car_0, T=0s, SeatArea)
    Mate->>Mate: OS 저전력 ActivityRecognition (IN_VEHICLE 진입) 감지
    Mate->>Server: 가벼운 탑승 체크인 패킷 전송 (User_ID, Lat_mate_0, Lng_mate_0, Timestamp)
    Server->>Server: [1차 검증] 거리 Δd ≤ 50m & 시간차 |Δt| ≤ 120s 후보군 필터링 (운전자 본인 ID 제외)
    Car->>Car: 차량 주행 시작 → DistanceDrivenListener ΔD ≥ 200m 도달 감지
    Car->>Server: 2차 주행 궤적 좌표 전송 (Lat_car_1, Lng_car_1)
    Mate->>Server: 이동 중 좌표 업데이트 (Lat_mate_1, Lng_mate_1)
    Server->>Server: [2차 검증] 200m 주행 벡터·속도 일치 확인 (배웅 지인 오탐지 완벽 차단)
    Server-->>Car: 동승 세션(Co-Ride Session) 최종 확정 + 재회 브리핑 & 맞춤 앰비언트 테마 반환
    Note over Car,Server: 주행 중 좌석별 순수 차분 ΔD_seat = D_current - D_start[seat] 실시간 누적 (터널 진입 시 Room SQLite 로컬 큐잉)
    Car->>Car: 목적지 도착 · 기어 P 전환 및 좌석 Empty 감지 (휴게소 정차 시 3분 디바운싱 유지)
    Car->>Server: 오늘의 운행 종료(Session Finalization) 요청
    Server-->>Car: 양 끝단 500m(15%) 절삭 경로 + 인카 플레이리스트 + 운전자·동승자 마일리지 동시 적립

```

### 🛡️ 핵심 엣지 케이스(Edge Case) 방어 설계

1. **좌석 위 무거운 가방·짐 오탐지 차단**: `checkSeatBeltBuckledCapabilities()`를 사전 호출하여 안전벨트 센서를 지원하는 좌석에서는 `Occupied(착석) + SeatBelt Buckled(안전벨트 체결)`이 동시에 충족될 때만 동승자 매칭을 트리거합니다 (`Occupied && !SeatBelt` 상태는 즉시 ‘가방/짐’으로 판정해 세션에서 제외).

2. **중간 픽업·하차 시 좌석별 독립 차분(**$\Delta D$**) 연산**: 운전자가 혼자 달리다가 동승자를 중간에 태우거나 2열에 추가 동승자가 합류할 때, 각 좌석(`1R`, `2L`, `2R`)별로 탑승 순간의 Odometer 스냅샷 `seatStartOdo[seat]`를 개별 저장하여 $\Delta D_{\text{seat}} = D_{\text{current}} - D_{\text{start}}[\text{seat}]$ 수식으로 각 동승자가 실제로 함께 달린 순수 거리만 정확히 발라냅니다.

3. **휴게소 임시 하차 시 3분(180초) 디바운싱**: 정차(`Parking`) 상태에서 도어가 열리고 좌석이 `Empty`가 되더라도 즉시 세션을 끊지 않고 **180초 디바운싱 타이머**를 가동하여, 휴게소·편의점 잠깐 정차 후 재탑승 시 단일 트립 세션이 끊기지 않고 자연스럽게 이어집니다.

4. **터널·지하주차장 네트워크 단절 시 Room SQLite 오프라인 큐**: 네트워크 단절 시 UI 프리징 없이 로컬 Room SQLite 트랜잭션 큐에 주행 차분 데이터를 1차 캐싱하고, 온라인 복구 시 클라우드 서버로 일괄 배치(Batch) 동기화합니다.

5. **시동 OFF(Suspend) 진입 시 리스너 일괄 해제**: 차량 시동이 꺼져 Suspend 모드로 진입할 때 `unregisterSeatOccupancy()`, `unregisterDistanceDriven()`, `FusedLocation.release()`를 일괄 호출하여 백그라운드 메모리 누수와 암전류 소모를 방지합니다.

## 🔌 4. Pleos Connect SDK 도메인별 활용 및 퍼미션 명세

하드웨어에 직접 접근하지 않고 [현대자동차그룹 차량용 앱 개발 공모전](https://appdevchallenge.auto-hmg.io/)이 제공하는 표준 Pleos Connect SDK 및 `checkXXXCapabilities` 사전 가용성 검증 원칙을 100% 준수합니다.

| 활용 SDK 모듈 | 도메인 / 클래스 | 호출 API·리스너 및 필수 퍼미션 | 서비스 내 활용 목적 및 상세 구현 로직 | 
| ----- | ----- | ----- | ----- | 
| **Vehicle SDK (`v2.3.2`)** | `Seat` & `Door` | `checkSeatOccupancyCapabilities()`, `getCurrentSeatOccupancy()`, `SeatOccupancyListener.onSeatOccupancyUpdated()`, `unregisterSeatOccupancy()`, `checkSeatBeltBuckledCapabilities()`, `DoorPositionListener` | `SeatArea`(`ROW_1_RIGHT`, `ROW_2_LEFT`, `ROW_2_RIGHT`)별 착석 상태(`Empty`/`Occupied`)와 안전벨트 체결, 도어 개폐를 감지하여 **실시간 2D 실내 디지털 트윈**에 렌더링하고 동승 세션 시작·종료 트리거로 활용 | 
| **Vehicle SDK (`v2.3.2`)** | `Odometer` | `getCurrentDistanceDriven()`, `getTripCurrentDrivingDistance()`, `DistanceDrivenListener`, `checkTripDrivingTimeCapabilities()`, `DrivingTimeListener.onDrivingTimeUpdated()` | 좌석별 탑승 시점 기준 주행거리(`D_start[seat]`)와 현재 주행거리(`D_current`)의 차분($\Delta D$) 및 순수 동승 시간($\Delta T$)을 독립 산출하고, 1km당 10P의 동승 마일리지를 운전자·동승자에게 동시 적립 | 
| **FusedLocation SDK (`v1.1.2`)** | `FusedLocation` | `isSupported()`, `getLastKnownLocation()`, `registerFusedLocationCallback()`, `showLocationAccessBlockedPopup()`, `FusedLocationListener.onChanged()` / `onError()` | 탑승 시점(0m)과 200m 주행 후 좌표를 추출해 클라우드 2단계 궤적 검증의 기준점으로 전송하고, 내비 미사용 주행 시 하차 지점 역지오코딩(2순위 태깅) 및 위치 차단 표준 안내 팝업 표시 | 
| **NaviHelper SDK (`v3.0.5`)** | `NaviHelper` | 퍼미션: `pleos.car.permission.NAVI_ROUTE`, `NAVI_ROUTE_SEARCH` / 메서드: `getDestinationInfo()`, `requestRouteWithWaypoints(RouteWithWaypointsRequest)` | 1순위로 순정 내비게이션 목적지 POI 명칭을 수집하고, 리포트에서 **\[그날의 코스로 길안내 시작\]** 실행 시 `RouteInfo`와 `RequestWaypointInfo`를 담아 순정 내비게이션 경유지 길안내 즉시 호출 | 
| **Gleo Interface & Voice** | `assets/schema.json`, `TextToSpeech (v3.3.1)`, `SpeechToText` | `actionCalls` 최대 3개 대문자 함수(`GET_TOP_CORIDER_STATS`, `TAG_CURRENT_PASSENGER`, `START_MEMORY_ROUTE`) 및 Callback Schema | 주행 중 화면 전환 없이 백그라운드 브로드캐스트(`component: "service"`)로 동승 통계를 LLM 음성으로 브리핑하거나 미등록 동승자를 음성 태깅하고, 추억의 코스 길안내(`component: "activity"`) 실행 | 
| **AAOS Standard API** | `CarUxRestrictionsManager` | `registerListener()`, `onUxRestrictionsChanged()` | 기어 `D(Driving)` 주행 제한 활성 시 즉시 주행 안전 다크 방현(Glare-Free) 고대비 카드(명암비 ≥ 7:1, 56px 대형 터치 타겟)로 전환하고 2-Window 리포트 탭 진입을 하드웨어 수준에서 잠금 | 

### Gleo AI `assets/schema.json` 등록 규격 (최대 3개 함수 정책 준수)

```
{
  "appVersion": "1.0.0",
  "actionCalls": [
    {
      "name": "GET_TOP_CORIDER_STATS",
      "component": "service",
      "description": "주행 중 화면 전환 없이 올해 최장 거리·최장 시간 동승자 통계를 백그라운드 브로드캐스트로 조회하여 TTS 음성 브리핑"
    },
    {
      "name": "TAG_CURRENT_PASSENGER",
      "component": "service",
      "description": "현재 착석한 미등록 동승자를 음성 명령으로 즉시 태깅하여 동승 세션에 매핑"
    },
    {
      "name": "START_MEMORY_ROUTE",
      "component": "activity",
      "description": "지정한 동승자와의 최장 로드트립 경유지 코스를 NaviHelper.requestRouteWithWaypoints()로 즉시 길안내 실행"
    }
  ]
}

```

## 🎨 5. 핵심 기능 및 UI/UX 상세 명세

### ① 실시간 2D 실내 디지털 트윈 & 감성 재회 브리핑 (In-Car Live Experience)

* **실시간 차량 실내 디지털 트윈 (Digital Twin)**: 좌석(`1R`, `2L`, `2R`) 착석 여부, 안전벨트 체결 상태, 도어 개폐 애니메이션, 그리고 좌석별 독립 주행거리($\Delta D$) 카운팅을 탑다운(Top-Down) 2D 차량 그래픽으로 직관적으로 시각화합니다.

* **‘N일 만의 재회’ 메모리 브리핑 & 앰비언트/공조 동기화**: 오랜만에 탑승한 동승자(예: 45일 만에 탑승한 단짝 친구)가 매칭되면 \*"박영희 님과 45일 만의 반가운 재회네요! 마지막 동승은 9월 28일 해운대 드라이브(68km)였습니다"\*라는 감성 토스트와 음성 브리핑을 제공하고, 해당 동승자와의 추억 테마에 맞춘 실내 앰비언트 라이트(오션 시안 / 선셋 코랄 / 포레스트 민트 등)와 선호 공조 온도를 자동 동기화합니다.

### ② 오늘의 운행 종료 리캡 카드 (프라이버시 폴리라인 절삭 + 인카 플리)

* **양 끝단 500m(15%) 프라이버시 폴리라인 절삭(Clipping)**: 목적지 도착 후 운전자가 \[🛑 오늘의 운행 종료\]를 누르면 오늘의 주행 경로와 키로수가 생성됩니다. 이때 **\[🔒 출발·도착지 감추기\]** 보호 필터가 작동하여 출발지(자택)와 도착지(숙소 등) 반경 500m(단거리는 앞뒤 15%) 폴리라인 세그먼트를 물리적으로 절삭함으로써 SNS에 경로를 공유해도 개인 사생활 위치가 노출되지 않습니다.

* **함께 들었던 인카 플레이리스트(BGM) 아카이빙**: 주행 구간별로 차량 미디어 세션에서 재생된 노래 목록과 재생 지점을 자동 기록하며, 터치 시 인카 오디오 샘플을 미리 들어볼 수 있습니다.

### ③ 동승 마일리지 동시 적립 · 원터치 양도 · 제휴처 바우처 교환

* **운전자·동승자 동시 적립 (`1 km = 10 P`)**: 동승 주행 거리($\Delta D$)에 비례해 운전자와 동승자 양측의 지갑에 마일리지가 동시에 쌓입니다.

* **운전자에게 오늘의 마일리지 원터치 양도**: 동승자는 기름값·운전 수고에 대한 고마움의 표시로 **\[오늘 마일리지 양도\]** 버튼을 눌러 그날 적립된 포인트를 운전자에게 즉시 선물할 수 있습니다.

* **실생활 모빌리티 제휴처 사용**: 누적된 마일리지는 차량 화면에서 즉시 **현대오일뱅크 주유/세차 쿠폰(3,000P)**, **현대 E-pit 전기차 충전 크레딧(2,000P)**, **인카 카페이 드라이브스루 커피 할인권(1,500P)**, **현대 블루핸즈 정비 바우처(5,000P)** 핀번호로 교환할 수 있습니다.

### ④ 유즐리 연간 모빌리티 리포트 (Co-Ride Wrapped — 4대 스토리 카드 & 9:16 인스타 스토리 프리뷰)

* **Dual Crown Award & 페르소나 배지**: 최장 거리($\Sigma \Delta D$) 및 최장 시간($\Sigma \Delta T$) 베스트 동승자 선정과 직관적 비유 카피(*"서울-부산 2.3회 왕복 거리"*, *"영화 16편 분량의 대화"*), 주행 시간대 군집화 기반 페르소나 배지 부여.

* **실물 9:16 인스타그램 스토리 & 카카오톡 공유 프리뷰 모달**: QR 코드를 스캔하거나 공유 버튼을 누르면 실제 스마트폰 9:16 비율의 ‘연간 모빌리티 보딩패스’와 **‘오늘의 운행 종료 리캡 카드’** 실물 그래픽 프리뷰 모달이 즉시 열립니다.

## 🖥️ 6. 본선 오프라인 라이브 시연 (Emulator & Virtual Console) 실행 방법

본 저장소에 포함된 통합 프로토타입(`Usualy_V1.html`)은 외부 블루투스 동글이 없는 **Pleos Connect Emulator + Virtual Console (v2.1.0+)** 오프라인 발표장 환경에서 100% 실시간 시연이 가능하도록 제작되었습니다.

### 빠른 실행 및 테스트 시나리오

1. 브라우저에서 `Usualy_V1.html`을 실행합니다.

2. **원클릭 자동 시연**: 우측 상단의 **\[2차 본선 시연 시나리오 자동 실행\]** 버튼을 클릭하면 아래 3단계가 자동으로 연속 시연됩니다.

   * **Step A (조수석 탑승 & 45일 만의 재회)**: 조수석(`ROW_1_RIGHT`)이 `Occupied`로 전환되며 ‘45일 만의 재회’ 브리핑과 함께 선셋 코랄 앰비언트 조명이 동기화되고, 기어 `D` 주행 안전 다크 방현 뷰가 활성화됩니다.

   * **Step B (2열 중간 합류 & 좌석별 독립** $\Delta D$ **누적)**: 주행 중 2열 좌측(`ROW_2_LEFT`)에 동기가 중간 합류하여 각 좌석의 탑승 시점(`D_start`)부터 독립된 주행거리 차분($\Delta D$)과 동승 마일리지가 실시간 누적됩니다.

   * **Step C (운행 종료 · 500m 폴리라인 절삭 · 마일리지 양도)**: 기어 `P` 전환과 함께 **\[🏁 오늘의 운행 종료 & 마일리지\]** 화면이 열리며, 양 끝단 500m 프라이버시 절삭 경로, 함께 들은 플레이리스트, 동승 마일리지 운전자 양도 및 현대오일뱅크 쿠폰 교환을 즉시 시연할 수 있습니다.

3. **수동 엣지 케이스 검증**:

   * 좌측 **2D 차량 실내 디지털 트윈**의 좌석을 직접 클릭하거나 우측 Virtual Console 드롭다운에서 `가방/짐 놓기 (벨트 OFF)`를 선택해 안전벨트 미체결 짐 오탐지 차단을 확인합니다.

   * **`[☕ 휴게소 임시하차 (3분 디바운싱)]`** 버튼을 눌러 180초 카운트다운 동안 세션이 유지되는지 확인합니다.

   * 상단 **`[📱 9:16 인스타 스토리 실물 프리뷰]`** 버튼을 눌러 연간 보딩패스와 오늘의 운행 리캡 카드의 실제 SNS 공유 화면을 확인합니다.

***"※ 아래 디렉토리 구조는 본선 최종 구현을 위한 타깃 아키텍처이며, 1차 서류 심사에서는 심사위원분들께서 별도 환경 구축 없이 웹에서 즉시 전체 기능과 Virtual Console을 테스트하실 수 있도록 단일 통합 시뮬레이터(Usualy_V1.html)를 제공합니다."***
## 📂 7. 저장소 디렉토리 구조 (Repository Structure)

```
UsUaly/
├── README.md                        # 프로젝트 개요, 3-Tier 시스템 아키텍처 및 SDK 명세서
├── Usualy_V1.html         # 인카 HMI(1-Win/2-Win/디지털 트윈) + Virtual Console 통합 시연 프로토타입
├── app-in-car/                      # Pleos Connect AAOS Kotlin 네이티브 클라이언트 모듈
│   ├── src/main/assets/schema.json  # Gleo AI Interface 3대 액션 함수 선언 스키마
│   ├── src/main/AndroidManifest.xml # pleos.car.permission.NAVI_ROUTE 퍼미션 및 Landscape Activity 선언
│   └── src/main/kotlin/...          # Seat/Door/Odometer/FusedLocation/NaviHelper SDK 래퍼 및 Room 캐시
├── server-trajectory-match/         # 클라우드 2단계 시공간·궤적 교차 검증 & 마일리지 원장 서버
└── companion-web-pwa/               # 모바일 QR 온보딩 및 9:16 인스타 스토리 보딩패스 렌더러

```
