# SmartFuzz
> 정적 분석-동적 퍼징 하이브리드 기반 스마트 컨트랙트 취약점 자동 탐지 시스템

---

## 📋 프로젝트 개요

| 항목 | 내용 |
|---|---|
| 프로젝트명 | SmartFuzz |
| 부제 | 정적 분석-동적 퍼징 하이브리드 기반 스마트 컨트랙트 취약점 자동 탐지 시스템 |
| 분류 | 개인 프로젝트 / Web3 보안 |
| 추진 기간 | 총 5주 |
| 작성일 | 2026.06 |

---

## 1. 배경 및 주제 선정 이유

### 1-1. 시장 현황 및 문제 인식

Web3 생태계의 급성장과 함께 스마트 컨트랙트를 대상으로 한 해킹 피해가 지속적으로 증가하고 있다. Chainalysis 보고서 기준, 2023년 한 해 DeFi 프로토콜에서 발생한 해킹 피해액은 약 **11억 달러** 규모에 달하며, 피해의 상당수는 컨트랙트 로직 취약점에 기인한다.

이러한 배경에서 스마트 컨트랙트 보안 감사(Audit) 수요는 폭발적으로 증가했으나, 현재 현장에서 사용되는 분석 도구들은 구조적 한계를 가지고 있다.

### 1-2. 기존 도구의 한계 분석

**① 정적 분석 도구 (Slither, Mythril 등)**

정적 분석 도구는 소스코드를 AST(Abstract Syntax Tree) 및 CFG(Control Flow Graph)로 파싱한 뒤, 자체 중간 표현인 SlithIR(SSA 기반 3-주소 코드)로 변환하여 취약 패턴을 탐지한다. 분석 속도가 빠르고 전체 코드베이스를 커버할 수 있다는 장점이 있으나, 실행 컨텍스트 없이 패턴만 매칭하므로 **False Positive 비율이 높고**, 탐지된 취약점이 실제로 익스플로잇 가능한지 검증하지 못한다.

**② 동적 퍼징 도구 (Echidna, Medusa 등)**

동적 퍼징 도구는 실제 EVM 환경에서 컨트랙트를 실행하며 취약점을 탐지한다. 실제 트리거 가능성을 검증할 수 있어 False Positive가 낮다는 장점이 있으나, 랜덤 또는 커버리지 기반 시드에 의존하므로 **복잡한 상태 전이가 필요한 취약 경로에 도달하지 못하는 경우가 많고**, 분석 시간이 길다.

**③ 도구 통합의 부재**

두 방식을 결합하는 **통합 파이프라인이 업계 표준으로 정착되어 있지 않으며**, 보안 감사 엔지니어가 수동으로 각 도구를 운용하고 결과를 해석해야 하는 비효율이 존재한다.

### 1-3. 주제 선정 이유

상기 문제를 해결하기 위해, 정적 분석의 결과를 동적 퍼저의 시드(seed)로 활용하는 **하이브리드 자동 분석 파이프라인**을 구현하고자 한다. 이를 통해 각 도구의 단점을 상호 보완하고, 실무 수준의 Web3 보안 엔지니어링 역량을 체득하는 것을 목표로 한다.

---

## 2. 프로젝트 상세

### 2-1. 목표

**최종 목표**

Solidity 소스코드 및 EVM 바이트코드를 입력받아, 정적 분석 → 동적 퍼징으로 이어지는 하이브리드 파이프라인을 통해 스마트 컨트랙트 취약점을 자동 탐지하고, PoC(Proof of Concept) 트랜잭션까지 자동 생성하는 CLI 도구 구현

**세부 목표**

| # | 목표 | 성공 기준 |
|---|---|---|
| 1 | 정적 분석 파이프라인 구현 | Slither API로 취약 후보 함수 JSON 추출 |
| 2 | 퍼징 자동화 | 정적 분석 결과를 Medusa 시드로 주입하여 퍼징 실행 |
| 3 | 파이프라인 통합 | 단일 CLI 명령으로 전체 파이프라인 실행 |
| 4 | 최종 검증 | Damn Vulnerable DeFi v3 대상 취약점 탐지 성공 — Reentrancy(Rewarder), Access Control(NaiveReceiver), Unchecked Return(TrusterLenderPool) 3개 챌린지 검증 |
| 5 | 커버리지 향상 측정 | 정적 분석 단독 시드 대비 하이브리드 시드로 branch coverage 향상 확인 |
| 6 | FP 감소 측정 | Slither 단독 실행 대비 퍼징 검증 후 False Positive 비율 감소 확인 |
| 7 | 탐지 시간 측정 | 수동 감사 대비 초기 후보 추출 소요 시간 기록 |

### 2-2. 범위 (Scope)

**In-Scope**
- Solidity 소스코드(`.sol`) 입력 분석
- EVM 바이트코드 입력 분석 (소스 없는 배포 컨트랙트 대응)
- Reentrancy, Integer Overflow/Underflow (unchecked 블록, 인라인 어셈블리, 다운캐스팅 대상 한정 — Solidity 0.8.x 기본 built-in 보호 범위 외), Access Control, Unchecked External Call 탐지
- 취약점 심각도 분류 (Critical / High / Medium / Low)
- Markdown / JSON 형식 리포트 자동 생성

**Out-of-Scope**
- 실제 온체인 컨트랙트 자동 공격 실행
- 멀티체인 대응 (EVM 체인 한정)
- GUI 인터페이스

### 2-3. 시스템 아키텍처

```
┌─────────────────────────────────────────────────────┐
│                      입력 레이어                        │
│   .sol 파일 / 컨트랙트 주소 / EVM 바이트코드             │
└──────────────────────┬──────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│                   Stage 1 : 정적 분석                  │
│  · Slither Python API — AST/CFG 분석                  │
│  · 취약 함수 후보 추출 → 입력 제약조건 힌트 생성           │
│  · 바이트코드 입력 시 pyevmasm 디스어셈블 후 분석          │
└──────────────────────┬──────────────────────────────┘
                       ↓ (취약 후보 함수 목록 + 힌트 JSON)
                         corpus seed 변환: 함수 selector → method 필드 /
                         인자 타입 힌트 → arguments 초기값 /
                         선행 호출 순서 → CallSequenceElement 배열 순서
┌─────────────────────────────────────────────────────┐
│                   Stage 2 : 동적 퍼징                  │
│  · Medusa — 정적 분석 힌트 기반 시드 corpus 주입          │
│  · Medusa 내장 TestChain (medusa-geth 기반) 환경에서 실행 │
│  · invariant 자동 생성 (취약 유형별 템플릿 기반):          │
│    Reentrancy → 잔액 불변 조건                          │
│    Access Control → 권한 없는 주소 상태 변경 불변 조건    │
│    Integer Overflow → 연산 전후 값 범위 불변 조건         │
│  · 커버리지 기반 변이(mutation)로 취약 경로 탐색           │
└──────────────────────┬──────────────────────────────┘
                       ↓ (트리거된 취약점 + 재현 트랜잭션)
┌─────────────────────────────────────────────────────┐
│                   Stage 3 : 리포팅                     │
│  · 취약점 분류 및 심각도 평가                             │
│  · PoC 트랜잭션 시퀀스 자동 출력                          │
│  · Markdown / JSON 리포트 생성 (Jinja2)                │
└─────────────────────────────────────────────────────┘
```

---

## 3. 추진 일정

### 3-1. 전체 일정 (총 5주)

| Phase | 주요 과제 | 산출물 | 기간 |
|---|---|---|---|
| Phase 1 | Slither 정적 분석 연동 | 취약 후보 추출기 모듈 | 1주 |
| Phase 2 | Medusa 퍼징 자동화 | 시드 주입 및 퍼징 실행 모듈 | 2주 |
| Phase 3 | 파이프라인 통합 및 CLI 제작 | 통합 CLI 도구 | 1주 |
| Phase 4 | 리포팅 및 최종 검증 | 리포트 생성기, 검증 결과 | 1주 |

### 3-2. 세부 일정

**Phase 1 — Slither 정적 분석 연동 (1주)**
- Slither Python API 환경 구성
- 커스텀 Detector 작성 (타겟 취약점 패턴 정의)
- 취약 함수 후보 JSON 추출기 구현
- 산출물: `static_analyzer.py`

**Phase 2 — Medusa 퍼징 자동화 (2주)**
- Medusa 소스 구조 분석 및 커스텀 포인트 파악
- 정적 분석 결과 → Medusa corpus seed 변환 로직 구현 (Slither 추출 함수 selector → corpus method 필드, 인자 타입 힌트 → arguments 초기값, 선행 호출 순서 → CallSequenceElement 배열)
- 취약 유형별 Invariant 템플릿 자동 생성 로직 구현 (Reentrancy → 잔액 불변 조건, Access Control → 권한 불변 조건, Integer Overflow → 값 범위 불변 조건), SlithIR 상태 변수·함수 시그니처 기반 `.sol` 파일 자동 출력
- Medusa TestChain 환경 구성 및 자동 실행 스크립트 작성
- 산출물: `fuzzing_engine.py`, `seed_injector.py`, `invariant_generator.py`

**Phase 3 — 파이프라인 통합 및 CLI (1주)**
- Stage 1 → 2 자동 연결 오케스트레이터 구현
- EVM 바이트코드 fallback 경로 구현 (pyevmasm 연동)
- CLI 인터페이스 제작 (`smartfuzz analyze <target>`)
- 산출물: `orchestrator.py`, `cli.py`

**Phase 4 — 리포팅 & 최종 검증 (1주)**
- 취약점 심각도 평가 로직 구현
- Jinja2 기반 Markdown / JSON 리포트 생성기 구현
- Damn Vulnerable DeFi v3 대상 end-to-end 검증 (Reentrancy/Rewarder, Access Control/NaiveReceiver, Unchecked Return/TrusterLenderPool 3개 챌린지)
- 산출물: `reporter.py`, 최종 검증 리포트

---

## 4. 활용 기술 스택

| 레이어 | 기술 | 버전 | 선택 이유 |
|---|---|---|---|
| 정적 분석 | Slither | 0.10.x | Python API 지원, 커스텀 Detector 작성 가능 |
| 동적 퍼징 | Medusa | latest | Go 기반으로 소스 수정 용이, API 친화적 리포팅; go-ethereum 기반 자체 EVM(TestChain) 내장, 외부 EVM 환경 불필요 |
| 바이트코드 분석 | pyevmasm | 0.2.x | EVM 바이트코드 디스어셈블, 소스 없는 컨트랙트 대응 |
| 오케스트레이션 | Python | 3.10+ | Slither 연동 및 전체 파이프라인 제어 |
| 리포팅 | Jinja2 | 3.x | 템플릿 기반 유연한 리포트 생성 |

---

## 5. 진행 현황

| 항목 | 상태 | 비고 |
|---|---|---|
| 주제 선정 | ✅ 완료 | |
| 요구사항 정의 | ✅ 완료 | In/Out-Scope 확정 |
| 시스템 아키텍처 설계 | ✅ 완료 | 3-Stage 파이프라인 확정 |
| 기술 스택 확정 | ✅ 완료 | Slither + Medusa 조합 채택 |
| 추진 일정 수립 | ✅ 완료 | 5주 로드맵 확정 |
| Phase 1 착수 | 🔲 예정 | Slither API 연동 |
| Phase 2 착수 | 🔲 예정 | Medusa 소스 구조 분석 선행 필요 |
| Phase 3 착수 | 🔲 예정 | |
| Phase 4 착수 | 🔲 예정 | |

---

## 6. 기대 효과

**기술적 기대 효과**
- 정적 분석 단독 대비 퍼저 커버리지 향상 (취약 경로 탐색 효율 증가)
- 수동 감사 대비 초기 취약점 후보 추출 시간 단축
- EVM 바이트코드 레벨 분석 지원으로 소스 미공개 컨트랙트 대응 가능

**학습/성장 기대 효과**
- Slither 내부 분석 엔진 구조 및 커스텀 Detector 작성 역량 확보
- Medusa 퍼저 소스 수준 이해 및 커스터마이징 경험
- 실전 수준의 Web3 보안 도구 설계 및 구현 역량 체득

---

## 7. 리스크 관리

| 리스크 | 발생 가능성 | 대응 방안 |
|---|---|---|
| Medusa API 변경으로 인한 커스텀 어려움 | 중 | Echidna로 퍼저 교체 검토 |
| Slither 정적 분석 결과의 낮은 품질 (과도한 FP) | 중 | 커스텀 Detector 직접 작성으로 정밀도 제어 |
| 5주 일정 내 파이프라인 통합 미완 | 저 | Phase 4 범위 축소 (리포터 기능 간소화) |
| EVM 바이트코드 분석 정확도 한계 | 중 | 소스코드 입력 경로 우선 구현, 바이트코드는 추후 개선 |

---

## License

MIT License
