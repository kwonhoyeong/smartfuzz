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

Solidity 소스코드 또는 Foundry/Hardhat 프로젝트를 입력받아, 정적 분석 → 동적 퍼징으로 이어지는 하이브리드 파이프라인을 실행하고, 취약 후보와 Medusa 실패 시퀀스 기반 재현 정보를 리포트로 출력하는 CLI 도구 구현

**세부 목표**

| # | 목표 | 성공 기준 |
|---|---|---|
| 1 | 정적 분석 파이프라인 구현 | Slither API로 취약 후보 함수·상태 변수·호출 관계 JSON 추출 |
| 2 | 퍼징 자동화 | 정적 분석 결과를 Medusa 설정 및 Solidity property/assertion harness로 변환하여 퍼징 실행 |
| 3 | 파이프라인 통합 | 단일 CLI 명령으로 전체 파이프라인 실행 |
| 4 | 최종 검증 | Damn Vulnerable DeFi v4.1.0 태그를 고정하고 3개 이상 챌린지에서 실패 시퀀스 또는 취약 후보 재현 |
| 5 | 커버리지 향상 측정 | 동일 timeout/testLimit 조건에서 기본 Medusa 실행 대비 하이브리드 harness 실행의 coverage 변화 기록 |
| 6 | FP 감소 측정 | 라벨링된 소규모 benchmark에서 Slither 후보 중 Medusa로 재현된 항목과 미재현 항목 분리 |
| 7 | 탐지 시간 측정 | 후보 추출·harness 생성·퍼징 실행 단계별 소요 시간 기록 |

### 2-2. 범위 (Scope)

**In-Scope**
- Solidity 소스코드(`.sol`) 입력 분석
- Foundry/Hardhat 프로젝트 단위 분석
- Reentrancy, Access Control, Unchecked External Call, Integer Overflow/Underflow 후보 탐지
- Medusa 설정 파일 및 property/assertion harness 자동 생성
- 취약점 심각도 분류 (Critical / High / Medium / Low)
- Markdown / JSON 형식 리포트 자동 생성

**Out-of-Scope**
- 실제 온체인 컨트랙트 자동 공격 실행
- EVM 바이트코드만 있는 컨트랙트의 정밀 분석
- 범용 exploit/PoC 트랜잭션 자동 생성
- 완전 자동 invariant 생성 (MVP에서는 템플릿 기반 초안 생성 및 수동 보정 지원)
- 멀티체인 대응 (EVM 체인 한정)
- GUI 인터페이스

### 2-3. 시스템 아키텍처

```
┌─────────────────────────────────────────────────────┐
│                      입력 레이어                        │
│   .sol 파일 / Foundry 프로젝트 / Hardhat 프로젝트        │
└──────────────────────┬──────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│                   Stage 1 : 정적 분석                  │
│  · Slither Python API — AST/CFG 분석                  │
│  · 내장 detector + 커스텀 휴리스틱 기반 후보 추출        │
│  · 함수 selector, 인자 타입, 상태 변수 write/read 추출   │
└──────────────────────┬──────────────────────────────┘
                       ↓ (취약 후보 함수 목록 + 힌트 JSON)
                         fuzz target 변환:
                         targetFunctionSignatures /
                         property/assertion harness /
                         선행 호출 순서 휴리스틱
┌─────────────────────────────────────────────────────┐
│                   Stage 2 : 동적 퍼징                  │
│  · Medusa — 정적 분석 힌트 기반 설정/harness 실행         │
│  · Medusa 내장 TestChain 환경에서 실행                   │
│  · invariant/property 템플릿 초안 생성:                  │
│    잔액 변화, 권한 없는 상태 변경, arithmetic panic 등    │
│  · 커버리지 기반 변이(mutation)로 취약 경로 탐색           │
└──────────────────────┬──────────────────────────────┘
                       ↓ (실패 테스트 + 재현 call sequence)
┌─────────────────────────────────────────────────────┐
│                   Stage 3 : 리포팅                     │
│  · 취약점 분류 및 심각도 평가                             │
│  · 재현 call sequence 및 실행 조건 출력                  │
│  · Markdown / JSON 리포트 생성 (Jinja2)                │
└─────────────────────────────────────────────────────┘
```

### 2-4. MVP 가정 및 측정 기준

- 입력 프로젝트는 로컬에서 컴파일 가능한 Solidity 프로젝트로 제한한다.
- Medusa 내부 corpus 포맷을 직접 수정하지 않고, 공개 CLI 설정과 생성된 harness를 우선 사용한다.
- 자동 생성된 invariant/property는 초안이며, 프로젝트별 의미 보정이 필요할 수 있음을 리포트에 명시한다.
- coverage와 FP 감소는 일반적 성능 주장보다, 고정된 benchmark·timeout·testLimit 조건에서 재현 가능한 실험 결과로 기록한다.

---

## 3. 추진 일정

### 3-1. 전체 일정 (총 5주)

| Phase | 주요 과제 | 산출물 | 기간 |
|---|---|---|---|
| Phase 1 | Slither 정적 분석 연동 | 취약 후보 추출기 모듈 | 1주 |
| Phase 2 | Medusa 퍼징 자동화 | 설정/harness 생성 및 퍼징 실행 모듈 | 2주 |
| Phase 3 | 파이프라인 통합 및 CLI 제작 | 통합 CLI 도구 | 1주 |
| Phase 4 | 리포팅 및 최종 검증 | 리포트 생성기, 검증 결과 | 1주 |

### 3-2. 세부 일정

**Phase 1 — Slither 정적 분석 연동 (1주)**
- Slither Python API 환경 구성
- 내장 detector 결과 파싱 및 커스텀 휴리스틱 작성
- 취약 함수 후보·상태 변수 read/write·호출 관계 JSON 추출기 구현
- 산출물: `static_analyzer.py`

**Phase 2 — Medusa 퍼징 자동화 (2주)**
- Medusa CLI 설정 구조 및 실행 결과 포맷 분석
- 정적 분석 결과 → Medusa `targetFunctionSignatures` 및 실행 설정 변환 로직 구현
- 취약 유형별 property/assertion harness 템플릿 초안 생성 로직 구현
- Medusa TestChain 환경 실행 및 실패 call sequence 수집 스크립트 작성
- 산출물: `fuzzing_engine.py`, `seed_injector.py`, `invariant_generator.py`

**Phase 3 — 파이프라인 통합 및 CLI (1주)**
- Stage 1 → 2 자동 연결 오케스트레이터 구현
- Foundry/Hardhat 프로젝트 컴파일 경로 및 artifact 탐색 구현
- CLI 인터페이스 제작 (`smartfuzz analyze <target>`)
- 산출물: `orchestrator.py`, `cli.py`

**Phase 4 — 리포팅 & 최종 검증 (1주)**
- 취약점 심각도 평가 로직 구현
- Jinja2 기반 Markdown / JSON 리포트 생성기 구현
- Damn Vulnerable DeFi v4.1.0 고정 태그 대상 end-to-end 검증
- 기본 Medusa 실행 대비 하이브리드 harness 실행 결과 비교
- 산출물: `reporter.py`, 최종 검증 리포트

---

## 4. 활용 기술 스택

| 레이어 | 기술 | 버전 | 선택 이유 |
|---|---|---|---|
| 정적 분석 | Slither | 0.11.x | Python API 지원, 커스텀 분석 작성 가능 |
| 동적 퍼징 | Medusa | 1.5.x 고정 | coverage-guided fuzzing, CLI 실행, 자체 TestChain 지원 |
| 프로젝트 빌드 | Foundry / Hardhat | stable | 실전 Solidity 프로젝트 컴파일 및 artifact 확보 |
| 오케스트레이션 | Python | 3.10+ | Slither 연동 및 전체 파이프라인 제어 |
| 리포팅 | Jinja2 | 3.x | 템플릿 기반 유연한 리포트 생성 |

---

## 5. 진행 현황

| 항목 | 상태 | 비고 |
|---|---|---|
| 주제 선정 | ✅ 완료 | |
| 요구사항 정의 | ✅ 완료 | MVP 기준 In/Out-Scope 확정 |
| 시스템 아키텍처 설계 | ✅ 완료 | 3-Stage 파이프라인 확정 |
| 기술 스택 확정 | ✅ 완료 | Slither + Medusa + Foundry/Hardhat 조합 채택 |
| 추진 일정 수립 | ✅ 완료 | 5주 MVP 로드맵 확정 |
| Phase 1 착수 | 🔲 예정 | Slither API 연동 |
| Phase 2 착수 | 🔲 예정 | Medusa CLI 설정/결과 포맷 분석 선행 필요 |
| Phase 3 착수 | 🔲 예정 | |
| Phase 4 착수 | 🔲 예정 | |

---

## 6. 기대 효과

**기술적 기대 효과**
- 정적 분석 힌트를 활용한 퍼징 타겟 집중 및 취약 경로 탐색 효율 개선
- 수동 감사 대비 초기 취약점 후보 추출 시간 단축
- Slither 후보와 Medusa 재현 결과를 결합하여 후보 triage 효율 개선

**학습/성장 기대 효과**
- Slither 내부 분석 엔진 구조 및 커스텀 Detector 작성 역량 확보
- Medusa 퍼저 소스 수준 이해 및 커스터마이징 경험
- 실전 수준의 Web3 보안 도구 설계 및 구현 역량 체득

---

## 7. 리스크 관리

| 리스크 | 발생 가능성 | 대응 방안 |
|---|---|---|
| Medusa Go API 변경으로 인한 커스텀 어려움 | 중 | 내부 API 직접 의존 대신 CLI 설정과 harness 생성 우선 |
| Slither 정적 분석 결과의 낮은 품질 (과도한 FP) | 중 | 내장 detector 결과 필터링 및 커스텀 휴리스틱으로 정밀도 제어 |
| 자동 invariant 생성의 의미적 한계 | 높음 | 템플릿 초안 생성 + 수동 보정 가능한 harness 구조 제공 |
| 5주 일정 내 파이프라인 통합 미완 | 중 | bytecode-only 분석과 리포터 고급 기능 제외 |
| EVM 바이트코드 분석 정확도 한계 | 높음 | MVP 제외, 후속 연구 과제로 분리 |

---

## License

MIT License
