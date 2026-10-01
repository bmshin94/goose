# goose 전수조사 & 활용 전략 (한국어 정리본)

> 작성일: 2026-10-01
> 분석 대상 저장소: **https://github.com/bmshin94/goose**
> 업스트림 원본: **https://github.com/aaif-goose/goose**
> 공식 문서: https://goose-docs.ai
> 공식 유튜브: https://www.youtube.com/@goose-oss
> Discord: https://discord.gg/n8R5VaWDAn

---

## 목차

1. [goose란 무엇인가](#1-goose란-무엇인가)
2. [저장소 전수조사 결과](#2-저장소-전수조사-결과)
3. [핵심 기능 4대장](#3-핵심-기능-4대장)
4. [쉬운 비유로 이해하기](#4-쉬운-비유로-이해하기)
5. [설치 및 사용법](#5-설치-및-사용법)
6. [플러그인? 스킬? MCP?](#6-플러그인-스킬-mcp)
7. [API 토큰이 필요한가](#7-api-토큰이-필요한가)
8. [AI 에이전트 구축에 도움이 되는가](#8-ai-에이전트-구축에-도움이-되는가)
9. [React / PHP 로 만들 수 있는가](#9-react--php-로-만들-수-있는가)
10. [유튜브 강의 제작 가능성](#10-유튜브-강의-제작-가능성)
11. [수익화 아이디어 10선](#11-수익화-아이디어-10선)
12. [추천 로드맵](#12-추천-로드맵)
13. [법적 체크리스트](#13-법적-체크리스트)

---

## 1. goose란 무엇인가

**goose = Rust로 만든 오픈소스 AI 에이전트 프레임워크.**
"내 컴퓨터에서 직접 돌아가는 Claude Code / Cursor 같은 것"을 직접 만들 수 있는 뼈대.

| 항목 | 내용 |
|---|---|
| 원본 저장소 | `aaif-goose/goose` |
| 내 포크 | `bmshin94/goose` |
| 라이선스 | **Apache-2.0** (상업적 사용 가능) |
| 언어 | Rust (백엔드) + TypeScript / React / Electron (데스크톱) |
| 소속 | **Linux Foundation** 산하 Agentic AI Foundation (AAIF) |
| 규모 | 약 **29만 줄** Rust, 15개 크레이트 |
| 최초 개발 | Block (Square / Cash App) |

### 다른 AI 툴과의 결정적 차이

| | Claude Code / Cursor | **goose** |
|---|---|---|
| 소스 | 비공개 | **완전 공개 (Apache-2.0)** |
| 모델 | 고정 | **15개+ 자유 교체 (로컬 포함)** |
| 커스터마이징 | 제한적 | **브랜딩까지 변경해 내 제품으로 배포 가능** |
| 용도 | 주로 코딩 | 코딩 + 리서치 + 글쓰기 + 자동화 + 데이터분석 |
| 수익화 | 불가 | **가능** |

> `CUSTOM_DISTROS.md`(28KB)가 "너만의 goose 배포판 만들기"를 공식적으로 안내한다.
> 즉 **재단이 직접 커스텀 배포를 권장하는 프로젝트**다. 이것이 가장 중요한 포인트.

---

## 2. 저장소 전수조사 결과

### 2.1 최상위 폴더 구조

```
goose/
├── crates/              # Rust 본체 (15개 크레이트)
├── ui/                  # Electron 데스크톱 앱 + ACP 클라이언트
├── documentation/       # Docusaurus 공식 문서 사이트
├── examples/            # MCP 서버 예제, 플러그인 예제
├── workflow_recipes/    # 실전 워크플로우 레시피
├── buzz/                # GitHub 이슈 자동관리 봇 (실전 사례)
├── evals/harbor/        # 성능 평가 하네스
├── oidc-proxy/          # Cloudflare Worker 인증 프록시
├── services/ask-ai-bot/ # AI 봇 서비스
├── goose-self-test.yaml # goose가 자기 자신을 테스트하는 레시피
└── .devcontainer/, flake.nix, Dockerfile  # 개발환경 3종 세트
```

### 2.2 crates/ — Rust 심장부

| 크레이트 | 줄 수 | 역할 |
|---|---:|---|
| **`goose`** | 192,424 | **모든 핵심 로직** (에이전트 루프, 세션, 보안, 레시피, 스킬) |
| `goose-provider-types` | 30,391 | LLM 제공사별 타입 정의 |
| `goose-cli` | 28,424 | 터미널 CLI |
| `goose-providers` | 15,699 | 실제 LLM 연동 구현체 |
| `goose-local-inference` | 10,481 | **로컬 모델 추론** (API 없이 내 PC에서) |
| `goose-mcp` | 6,313 | 기본 내장 MCP 서버 |
| `goose-sdk` / `-types` | 5,994 | **GDK** — Python / Kotlin 바인딩 (UniFFI) |
| `goose-roaming` | 2,865 | **P2P 에이전트 공유** (iroh 기반) |
| `goose-agent` | 2,147 | 에이전트 추상화 |
| `goose-context-management` | 1,156 | 컨텍스트 압축 / 관리 |
| `goose-download-manager` | 690 | 모델 / 바이너리 다운로드 |
| `goose-acp-macros`, `goose-test`, `goose-test-support` | ~840 | 매크로, 테스트 지원 |

### 2.3 crates/goose/src/ — 진짜 핵심

```
agents/                        # 에이전트 루프 (가장 중요)
  ├── agent.rs                 # 레거시 루프
  ├── state_machine/           # 신규 상태머신 (GOOSE_STATE_MACHINE=1)
  ├── subagent_*               # 서브에이전트 (에이전트가 에이전트를 소환)
  ├── tool_execution.rs        # 툴 실행 파이프라인
  ├── tool_confirmation_*      # 사용자 승인 흐름
  ├── large_response_handler.rs# 대용량 툴 결과 처리
  ├── retry.rs                 # 재시도 / 레이트리밋
  ├── tool_schema_normalize.rs # 모델별 툴 스키마 정규화
  ├── extension_manager/       # MCP 익스텐션 로딩
  ├── extension_malware_check.rs # 악성 익스텐션 차단
  └── platform_extensions/     # 내장 툴
       developer, analyze, todo, orchestrator, summarize,
       summon, chatrecall, code_execution, scheduler, apps, tom

recipe/                # YAML 워크플로우 엔진
skills/                # 스킬 시스템
plugins/ + hooks/      # 플러그인 & 훅
security/              # 프롬프트 인젝션 탐지, 유출 감시, 스캐너
permission/            # 권한 제어
session/               # SQLite 대화 저장
scheduler.rs           # Cron 스케줄링
acp/                   # Agent Client Protocol (표준 프로토콜)
gateway/               # Telegram 연동 + 디바이스 페어링
live_voice/ dictation/ # 실시간 음성 대화
goose_apps/            # 에이전트가 만드는 미니 웹앱
otel/ tracing/         # 관측성 (OpenTelemetry)
```

### 2.4 4가지 실행 형태

```
1. 데스크톱 앱  (Electron, Mac/Win/Linux)
2. CLI          (goose run / goose session)
3. API / 서버   (goose serve → 내 앱에 임베드)   ★ 핵심
4. SDK (GDK)    (Python / Kotlin 바인딩)
```
→ 전부 **같은 Rust 엔진**을 공유한다.

### 2.5 지원 LLM 제공사 (`goose-providers` 분석)

Anthropic, OpenAI, Google, Ollama, OpenRouter, Azure AI Foundry, AWS Bedrock,
Databricks (v1 / v2 / AI Gateway), Snowflake, OpenAI-compatible (임의 호환 API),
goose 자체 로컬 추론, Live / Live Voice (실시간 음성)

### 2.6 MCP 익스텐션

- 공식 70개+ 연동 (`documentation/static/servers.json`에 레거시 58개 등재)
- 예: AgentQL, Alby, Apify, Asana, Auto Visualiser, Beads, Blender,
  Browserbase, Chrome DevTools, Cloudinary, Figma, Playwright 등
- **중요:** goose는 자체 MCP 디렉토리를 폐기하고
  [공식 MCP Registry](https://github.com/modelcontextprotocol/registry)로 이전 중
  ([Discussion #10830](https://github.com/aaif-goose/goose/discussions/10830))
  → **MCP 생태계가 지금 막 형성되는 단계 = 선점 기회**

### 2.7 내 포크 현재 상태

```
d24539e Merge pull request #1 from bmshin94/feat/claude-guide
c3a0b34 docs: appended CLAUDE.md persona guide   ← 카리나 페르소나 커밋
1e83e89 feat(sdk): OpenAI provider custom base URL
70fe8cb docs: clarify public crate APIs
d213a3b feat: Live voice conversations in desktop app
```

---

## 3. 핵심 기능 4대장

### 3.1 Recipe (레시피) — YAML 워크플로우

```yaml
version: 1.0.0
title: "Release Change Risk Check"
description: "릴리즈 변경 위험도 리포트 생성"
parameters:
  - key: version
    input_type: string
    requirement: required
instructions: |
  {{recipe_dir}}/release_risk_report.py --version {{version}} -o /tmp/report.md
  결과를 HIGH / MEDIUM / LOW 로 분류해서 보고서를 만들어줘.
```

```bash
goose run --recipe recipe.yaml --params version=1.2.0
```

- 반복 업무를 **템플릿화**해서 재사용
- `goose recipe deeplink` 로 **원클릭 공유 링크** 생성 가능
- 실전 예시: `workflow_recipes/release_risk_check/`, `goose-self-test.yaml`(23KB)

### 3.2 Subagent / Subrecipe — 에이전트 분열

```
          나
           │ "이 프로젝트 전체 분석해줘"
      goose (팀장)
           │ 일을 쪼개서
   ┌───────┼───────┬───────┐
 sub1    sub2    sub3    sub4     ← 병렬 동시 작업
 파일A   파일B   파일C   파일D
   └───────┴───────┴───────┘
           │ 결과 종합
      goose → 최종 보고
```
문서: `documentation/docs/tutorials/subagents.md`,
`documentation/docs/tutorials/subrecipes-in-parallel.md`

### 3.3 Scheduler — 자율 실행

```bash
goose schedule add --cron "0 9 * * 1-5"   # 평일 아침 9시 자동 실행
goose schedule list / sessions / run-now / cron-help
```

### 3.4 Security — 엔터프라이즈급 3중 방어

```
사용자 요청
   ↓
1차 adversary_inspector   → 프롬프트 인젝션 탐지
   ↓
2차 permission            → 툴 권한 검사 / 사용자 승인
   ↓
3차 egress_inspector      → 데이터 유출 감시
   ↓
실행
```
추가: `extension_malware_check.rs` (악성 익스텐션 차단), `scanner.rs`, `patterns.rs`

---

## 4. 쉬운 비유로 이해하기

| 비유 | 설명 |
|---|---|
| **AI 비서 로봇 조립 키트** | 완성품(ChatGPT)이 아니라 키트. 공짜 + 개조 가능 + 판매 가능 |
| **두뇌 소켓** | LLM 15종을 아무거나 꽂는다 |
| **팔다리** | MCP 익스텐션 70개 (손 / 눈 / 귀) |
| **업무 매뉴얼** | Recipe (YAML) — "이럴 땐 이렇게" |
| **알바생 고용** | Subagent — 4명이 나눠서 40분 → 10분 |
| **USB-C 규격** | MCP — 한 번 만들면 모든 AI 호스트에 호환 |
| **공항 보안검사** | Security — 3중 검사 후 실행 |
| **스마트폰 vs 앱** | goose = 스마트폰(플랫폼), MCP/Skill/Plugin = 앱 |

### 일 시키는 3단계

```
1단계 즉흥:  goose run -t "이 폴더 코드 리뷰해줘"
2단계 레시피: goose run --recipe 코드리뷰.yaml --params folder=src
3단계 자동화: goose schedule add --cron "0 9 * * 1-5"
```

---

## 5. 설치 및 사용법

### 5.1 방법 A — 데스크톱 앱 (입문 추천)

https://goose-docs.ai/docs/getting-started/installation
→ Mac / Windows / Linux 지원. 설치 후 API 키 입력만으로 사용.

### 5.2 방법 B — CLI (개발자 추천)

```bash
# 설치 (Mac / Linux)
curl -fsSL https://github.com/aaif-goose/goose/releases/download/stable/download_cli.sh | bash

# 초기 설정 (provider / 모델 / 키 입력)
goose configure

# 사용
goose session                           # 대화형 모드
goose run -t "현재 폴더 구조 설명해줘"      # 단발 실행
goose run -i prompt.txt                 # 파일에서 지시 읽기
```

Windows는 `download_cli.ps1` 사용.

### 5.3 방법 C — 소스 빌드 (개조 목적)

```bash
git clone https://github.com/bmshin94/goose
cd goose
source bin/activate-hermit     # 필수: Rust 툴체인 자동 세팅
cargo build --release

just run-ui                    # 데스크톱 앱 실행
```

개발 루프 (`AGENTS.md` 기준):
```bash
source bin/activate-hermit
# 코드 수정
cargo fmt
cargo build
cargo test -p <crate>
cargo clippy --all-targets -- -D warnings
```

### 5.4 주요 CLI 명령어 전체

| 명령어 | 용도 |
|---|---|
| `goose session` | 대화형 세션 시작 / 재개 |
| `goose run -t "..."` | 단발 실행 (헤드리스) |
| `goose run --recipe x.yaml` | 레시피 실행 |
| `goose recipe list / validate / deeplink / open` | 레시피 관리 |
| `goose configure` | 설정 (provider, 모델, 익스텐션) |
| `goose info -v` | 설정 / 경로 확인 |
| `goose doctor` | 진단 (문제 발생 시 최우선) |
| `goose schedule add --cron "..."` | 크론 스케줄링 |
| `goose serve` | **HTTP / WebSocket 서버 실행** |
| `goose acp` | ACP 에이전트 모드 |
| `goose mcp <name>` | 내장 MCP 서버 단독 실행 |
| `goose plugin install <git-url>` | 플러그인 설치 |
| `goose plugin update` | 플러그인 업데이트 |
| `goose skills list` | 스킬 목록 |
| `goose roam` | P2P 에이전트 공유 (iroh) |
| `goose gateway start / stop / pair / status` | 게이트웨이 (Telegram 등) |
| `goose update` | 본체 업데이트 |

출력 포맷: `--output-format text | json | stream-json` (외부 연동 시 유용)

### 5.5 추천 학습 루트 (7일)

```
1일차  데스크톱 앱 설치 → 대화해보기
2일차  CLI 설치 → goose run -t 로 파일 조작
3일차  레시피 직접 작성 (documentation/docs/guides/recipes/)
4일차  goose-self-test.yaml 읽기 (레시피 교과서)
5일차  소스 빌드 + crates/goose/src/agents/ 분석
6일차  goose serve → React 에서 호출
7일차  CUSTOM_DISTROS.md → 내 브랜드 배포판
```

---

## 6. 플러그인? 스킬? MCP?

**정답: 셋 다 아니다. goose는 "플랫폼 / 호스트"다.**

```
          goose  ← 플랫폼
           │
   ┌───────┼───────┬─────────┐
  MCP    Skills  Plugins  Recipes
 (손발)  (특기)   (훅)    (매뉴얼)
```

| 개념 | goose에서의 위치 | 코드 위치 |
|---|---|---|
| **MCP** | goose가 MCP **클라이언트(호스트)**. 동시에 `goose-mcp`로 **서버도 제공** | `agents/mcp_client.rs`, `crates/goose-mcp/` |
| **Skill** | goose 내부 기능 모듈 (Claude Skills와 동일 개념) | `crates/goose/src/skills/` |
| **Plugin** | 훅으로 goose 동작에 개입. `goose plugin install <git-url>` | `crates/goose/src/plugins/`, `hooks/` |
| **Recipe** | YAML 워크플로우 정의 | `crates/goose/src/recipe/` |
| **ACP** | 외부 클라이언트가 goose를 에이전트로 사용하는 표준 프로토콜 | `crates/goose/src/acp/` |

템플릿:
- MCP 서버 만들기 → `examples/mcp-wiki/` (Python)
- 플러그인 만들기 → `examples/plugins/hello-hooks/`

### Claude Code 와의 관계

- Claude Code = Anthropic 공식, 비공개, Claude 전용
- goose = 오픈소스, 모델 자유, **Claude / ChatGPT / Gemini 구독 계정을 ACP로 연결 가능**
- 문서: `documentation/docs/guides/acp-providers.md`

---

## 7. API 토큰이 필요한가

**아니다. 3가지 선택지가 있다.**

### 선택 1 — 완전 무료 (로컬 모델, 토큰 0원)

```bash
ollama pull qwen2.5-coder:7b
goose configure      # provider: ollama
```
- `goose-local-inference`(10,481줄)로 **goose 자체 추론 엔진**도 제공
- 장점: 무료, 완전 오프라인, 데이터 100% 로컬
- 단점: GPU / RAM 필요 (16GB+ 권장), 성능 제한

### 선택 2 — 기존 구독 재활용 (추천)

README 명시:
> *"Use API keys or your existing **Claude, ChatGPT, or Gemini subscriptions** via ACP"*

- Claude Pro/Max, ChatGPT Plus 구독 중이면 **추가 과금 없이** 연결
- 장점: 추가 비용 0원, 최고 성능 모델
- 단점: Rate limit

### 선택 3 — API 키

```bash
export ANTHROPIC_API_KEY=sk-ant-...
# 또는 OPENAI_API_KEY / GOOGLE_API_KEY / OPENROUTER_API_KEY
goose configure
```

### 기업용 옵션

Azure AI Foundry, AWS Bedrock, Databricks (AI Gateway 포함), Snowflake,
OpenRouter, OpenAI-compatible 엔드포인트

### 비용 절감 4가지 기법

| 기법 | 문서 / 코드 |
|---|---|
| 멀티모델 전략 (쉬운 작업 = 싼 모델) | `guides/multi-model/` |
| 컨텍스트 자동 압축 | `goose-context-management` |
| Tool Shim (툴 미지원 모델도 툴 사용) | `guides/tool-shim.md` |
| Rate limit 핸들링 | `guides/handling-llm-rate-limits-with-goose.md` |

---

## 8. AI 에이전트 구축에 도움이 되는가

**매우 도움된다. 이것이 goose의 최고 가치다.**

### 8.1 에이전트 구축의 진짜 난관 — goose는 이미 해결함

| 난관 | 일반적 지옥 | goose 해결 |
|---|---|---|
| 에이전트 루프 | 무한루프, 툴 호출 꼬임 | `agents/state_machine/` |
| 컨텍스트 초과 | 대화 길어지면 터짐 | `goose-context-management` |
| 모델 교체 | 제공사마다 API 상이 | `goose-providers` 15개 |
| 툴 스키마 | 모델마다 포맷 상이 | `tool_schema_normalize.rs` |
| 재시도 / 레이트리밋 | 429 지옥 | `agents/retry.rs` |
| 세션 저장 | 재시작 시 기억 소실 | SQLite 세션 매니저 |
| 권한 / 승인 | 위험 명령 차단 | `permission/` |
| 프롬프트 인젝션 | 보안 사고 | `security/` |
| 병렬 처리 | 서브에이전트 설계 | `subagent_*` |
| 비용 추적 | 요금 폭탄 | `token_counter.rs` |
| 관측성 | 디버깅 불가 | OpenTelemetry / Langfuse / MLflow / Laminar |
| 대용량 툴 결과 | 컨텍스트 폭파 | `large_response_handler.rs` |

→ 직접 삽질하면 **1~2년** 걸리는 분량이 이미 구현된 **참고 구현체**다.

### 8.2 꼭 읽어야 할 "교과서 파일" 순서

```
1. AGENTS.md                                  # 프로젝트 철학 (10분)
2. crates/goose/src/agents/mod.rs              # 전체 구조 (30분)
3. crates/goose/src/agents/state_machine/      # 모던 에이전트 루프 ★★★
4. crates/goose/src/agents/tool_execution.rs   # 툴 실행 파이프라인 ★★
5. crates/goose/src/agents/subagent_*          # 멀티에이전트 설계 ★★
6. crates/goose/src/security/                  # 에이전트 보안 ★★★
7. crates/goose/src/recipe/                    # 워크플로우 DSL 설계
8. documentation/docs/goose-architecture/       # 공식 아키텍처 문서
```

### 8.3 레어 자료 — "리팩토링 생중계"

`AGENTS.md` 발췌:
> 레거시 에이전트 루프(`agents/agent.rs`)를 상태머신(`agents/state_machine/`)으로
> 교체 중. `GOOSE_STATE_MACHINE=1` 로 활성화. 마이그레이션 완료까지 **양쪽 모두
> 구현 / 테스트**해야 한다.

→ "왜 상태머신으로 전환하는가"를 **실제 코드 두 버전 비교**로 학습 가능.
→ 돈으로 살 수 없는 수준의 자료.

### 8.4 3가지 활용 방식

```
A) 참고용   — 설계만 배우고 직접 구현
B) 임베드용 — goose serve 로 띄워 내 앱 백엔드로 (가장 빠름)
C) 포크용   — 소스 수정해 내 제품으로 (CUSTOM_DISTROS.md)
```

---

## 9. React / PHP 로 만들 수 있는가

**goose 자체 재구현은 비현실적. 그러나 goose를 쓰는 React / PHP 앱은 완전히 가능.**

### 9.1 정답 아키텍처

```
┌────────────────────────────────┐
│  React (또는 PHP) 프론트엔드      │  ← 내가 만드는 부분
│  채팅 UI, 대시보드, 로그인, 결제   │
└──────────────┬─────────────────┘
               │ HTTP / WebSocket
               ↓
┌────────────────────────────────┐
│  $ goose serve                 │  ← 실행만, 코드 0줄
│  (ACP over HTTP + WebSocket)   │
│  + 인증 토큰 지원 (auth.rs)      │
└──────────────┬─────────────────┘
               ↓
       LLM + MCP 익스텐션 70개
```

문서: `documentation/docs/guides/remote-goose-server.md`
코드: `crates/goose/src/acp/transport/` (`auth.rs` 포함)

### 9.2 React (강력 추천)

**레퍼런스가 이미 저장소 안에 있다** — `ui/desktop/` 이 React + TypeScript + Electron.

```
ui/desktop/src/
├── acp/          # goose 서버 통신 클라이언트 (복붙 가능)
├── components/   # 채팅 UI 컴포넌트
├── hooks/        # React 훅
├── recipe/       # 레시피 UI
├── liveVoice/    # 음성 대화 UI
├── i18n/         # 다국어 (한국어 가능)
└── types/        # 로컬 타입

ui/goose-acp-client/   # ACP 클라이언트 라이브러리 (독립 사용 가능)
```

> 주의 (`AGENTS.md` 규칙): `ui/desktop/src/api` 의 생성된 OpenAPI 타입을 import 하지 말고,
> ACP SDK 타입 또는 로컬 `src/types/*` 를 사용한다.

**React 로 만들 수 있는 제품**
- 웹 채팅 UI (팀 공용 AI 에이전트 포털)
- 에이전트 모니터링 대시보드 (세션 / 토큰 / 비용 시각화)
- 레시피 마켓플레이스 (검색 / 공유 / 실행)
- **노코드 레시피 빌더** (드래그앤드롭 → YAML 생성) ← 최고 아이디어
- 사내 AI 관리자 패널 (권한 / 익스텐션 중앙 관리)

### 9.3 PHP (가능하지만 제한적)

```php
// 방법 1: goose serve API 호출 (권장)
$ctx = stream_context_create([ /* headers, body */ ]);
$res = file_get_contents('http://localhost:3000/...', false, $ctx);

// 방법 2: CLI shell 호출 (간단한 작업)
$out  = shell_exec('goose run -t "분석해줘" --output-format json');
$data = json_decode($out, true);
```

**PHP 추천 용도**
- WordPress 플러그인 (AI 글쓰기 / SEO 자동화)
- Laravel 관리자 패널 + 큐로 goose 작업 처리
- 기존 PHP 레거시 시스템에 AI 기능 추가 (SI 사업)

**한계:** 스트리밍(SSE / WebSocket) 처리가 불편, 롱러닝 프로세스 관리 어려움
→ 큐(Redis / Horizon) 필수

### 9.4 비추천

| 시도 | 현실 |
|---|---|
| goose 를 JS 로 재구현 | 29만 줄, 수년 소요 |
| goose 를 PHP 로 재구현 | 성능 / 동시성 부적합 |

### 9.5 추천 스택

```
프론트: React + TypeScript + Vite + Tailwind   (ui/desktop 참고)
통신:   ui/goose-acp-client 재활용
백엔드: goose serve (Rust 바이너리, 작성 코드 0)
배포:   Docker (Dockerfile / BUILDING_DOCKER.md 제공) + Fly.io / Railway
결제:   Stripe / 토스페이먼츠
```

---

## 10. 유튜브 강의 제작 가능성

**가능하며, 현재가 선점 타이밍이다.**

### 10.1 시장 분석

| 지표 | 상태 |
|---|---|
| 한국어 goose 콘텐츠 | **거의 없음** (블루오션) |
| Trendshift 랭킹 등재 | 있음 (글로벌 화제성) |
| Linux Foundation 소속 | 신뢰도 높음 (기업 관심) |
| 공식 문서 품질 | Docusaurus + 튜토리얼 20개+ |
| 공식 유튜브 | `@goose-oss` (영어만) → 한국어 포지션 공백 |
| 검색 수요 | "AI 에이전트 만들기" 상승 중 |

### 10.2 20부작 커리큘럼

**시즌 1: 입문 (1~5화)**

| 화 | 제목 | 후킹 |
|---|---|---|
| 1 | goose가 뭐야? Claude Code 공짜 대안 | "월 20달러 아끼는 방법" |
| 2 | 5분 설치 + 첫 대화 | "진짜 5분 컷" |
| 3 | **API 키 없이 완전 무료로 쓰기** (Ollama) | "토큰 0원 AI 에이전트" |
| 4 | 구독 계정 연결하기 (ACP) | "ChatGPT Plus 재활용" |
| 5 | MCP 익스텐션 70개 구경 | "AI에 손발 달아주기" |

**시즌 2: 실전 자동화 (6~11화)**

| 화 | 제목 |
|---|---|
| 6 | 레시피 입문: 반복 업무를 YAML로 박제 |
| 7 | **매일 아침 9시에 AI가 혼자 일하게 만들기** (스케줄러) |
| 8 | GitHub 이슈 자동 관리 봇 만들기 (`buzz/` 분석) |
| 9 | 서브에이전트 병렬처리로 10배 빠르게 |
| 10 | Telegram으로 AI 조종하기 (`gateway/telegram.rs`) |
| 11 | Playwright 스킬로 웹 자동화 |

**시즌 3: 개발자 심화 (12~16화)**

| 화 | 제목 |
|---|---|
| 12 | 소스 빌드 + 코드 전체 구조 투어 |
| 13 | **에이전트 루프는 어떻게 동작하나** (state_machine 해부) |
| 14 | 나만의 MCP 서버 만들기 (`examples/mcp-wiki`) |
| 15 | 플러그인 & 훅 만들기 (`examples/plugins`) |
| 16 | 에이전트 보안: 프롬프트 인젝션 막기 (`security/`) |

**시즌 4: 수익화 (17~20화)**

| 화 | 제목 |
|---|---|
| 17 | **내 브랜드 AI 제품 만들기** (CUSTOM_DISTROS.md) |
| 18 | React + goose serve 로 웹 AI 서비스 만들기 |
| 19 | Docker 배포 + 클라우드 올리기 |
| 20 | 이걸로 돈 버는 10가지 방법 |

### 10.3 제작 꿀팁

1. **메타 콘텐츠**: `documentation/docs/tutorials/remotion-video-creation.md`
   → goose 가 Remotion 으로 영상을 제작하는 튜토리얼.
   "AI가 자기 소개 영상을 직접 만들었습니다" = 강력한 후킹.
2. **기존 자료 재활용**: `goose-self-test.yaml`(23KB) = 레시피 교재,
   `workflow_recipes/` = 실전 사례, `tutorials/` 20개 = 각 1편씩 = 20편 확보.
3. **멀티 플랫폼**
   ```
   YouTube (메인 강의)
     ├→ 블로그 / 벨로그 (텍스트 SEO)
     ├→ Shorts / 릴스 (30초 꿀팁)
     ├→ GitHub 예제 저장소 → 구독 유도
     └→ 유료 강의 (인플런 / 패스트캠퍼스)
   ```
4. **주의사항**
   - Apache-2.0 이라 강의 / 상업적 활용 합법
   - 코드 노출 시 라이선스 / 출처 표기 권장
   - goose 로고 / 상표는 AAIF 소유 → 내 브랜드로 오인되게 사용 금지
   - 업데이트가 빠르므로 영상에 **버전 명시** 필수

### 10.4 차별화 전략

> 단순 설치 영상은 금방 레드오션이 된다.
> **"goose 코드를 뜯어서 AI 에이전트 아키텍처를 배운다"** 포지션이면
> 경쟁자가 거의 없고, 고급 시청자(개발자)가 모여 **유료 전환율이 높다.**

---

## 11. 수익화 아이디어 10선

```
난이도 ↑
  高  │  ⑧브랜드SaaS   ⑩마켓플레이스
      │  ⑦기업SI       ⑨관측SaaS
  中  │  ④노코드빌더    ⑤버티컬제품
      │  ⑥MCP개발
  低  │  ①강의/콘텐츠   ②레시피템플릿   ③번역/문서
      └──────────────────────────────→ 수익규모
          小            中           大
```

### 1군: 당장 시작 가능 (초기자본 0원)

#### ① 유튜브 + 블로그 + 유료강의 패키지
난이도 ★☆☆☆☆ | 비용 0원 | 예상 월 50~500만원

```
무료 유튜브 20편 (트래픽)
   ↓ 구독자 확보
유료 강의 (인플런 / 클래스101 / 자체)
   ↓ 레퍼런스
1:1 컨설팅 / 기업 교육 (회당 200~500만원)
```
근거: 한국어 콘텐츠 공백 + 개발자 타겟(구매력 높음, 강의 15~30만원 가능)

#### ② 레시피 템플릿 판매 (가장 저평가된 기회)
난이도 ★★☆☆☆ | 비용 0원 | 예상 월 30~300만원

레시피는 YAML 파일 하나 → **디지털 상품화 최적**

| 번들 | 내용 | 가격 |
|---|---|---|
| 개발팀 필수 10종 | 코드리뷰, PR요약, 릴리즈노트, 테스트생성, 보안스캔 | 49,000원 |
| 마케터 팩 | 블로그 자동생성, SEO분석, 경쟁사 모니터링, SNS | 39,000원 |
| 데이터분석 팩 | CSV→리포트, 대시보드 생성, 이상치 탐지 | 59,000원 |
| 1인기업 팩 | 이메일 정리, 계약서 검토, 세금자료 정리 | 29,000원 |
| 취준생 팩 | 이력서 첨삭, 포트폴리오 생성, 면접 준비 | 19,000원 |

근거: `workflow_recipes/release_risk_check/` 가 실제로 그런 상품 형태.
판매 채널: Gumroad, 자체 웹샵, 인플런 자료, 노션 템플릿 마켓.

#### ③ 번역 + 한국어 문서 사이트
난이도 ★☆☆☆☆ | 비용 0원 | 예상 월 20~100만원

`documentation/` 에 Docusaurus 사이트 + `I18N.md` 다국어 구조 완비
→ 한국어 번역 사이트 운영 → SEO 트래픽 → 광고 / 강의 유입 / 제휴
→ 보너스: 번역 PR 업스트림 기여 = **오픈소스 컨트리뷰터 이력**

### 2군: 약간의 개발 필요 (1~3개월)

#### ④ 노코드 레시피 빌더 SaaS (최고 추천)
난이도 ★★★☆☆ | 비용 50~200만원 | 예상 월 100~1,000만원

```
┌──────────────────────────────────────┐
│ 드래그앤드롭 레시피 빌더 (React)        │
│ [트리거]→[파일읽기]→[AI분석]→[슬랙전송]  │
│        ↓ 자동으로 YAML 생성            │
│ [테스트 실행] [배포] [스케줄 설정]       │
└──────────────┬───────────────────────┘
               ↓
         goose serve (백엔드)
```

레시피 YAML 직접 작성은 개발자만 가능 →
**비개발자가 GUI로 AI 자동화를 만들게 하면 시장이 100배 커진다.**
(Zapier / Make 가 수조원 가치인 이유)

| 플랜 | 가격 | 내용 |
|---|---|---|
| Free | 0원 | 레시피 3개, 월 100회 실행 |
| Pro | 월 19,000원 | 무제한 레시피, 스케줄링 |
| Team | 월 99,000원 | 팀 공유, 권한관리, 감사로그 |
| Enterprise | 협의 | 온프레미스, SSO, SLA |

#### ⑤ 버티컬 특화 AI 에이전트 제품
난이도 ★★★☆☆ | 예상 월 200~2,000만원

`CUSTOM_DISTROS.md` 가 공식 지원.

| 제품 | 타겟 | 핵심 기능 | 가격 |
|---|---|---|---|
| 블로그봇 | 블로거 / 마케터 | 키워드→글 생성→워드프레스 발행 | 월 29,000원 |
| 법무도우미 | 중소기업 | 계약서 검토, 리스크 플래그 | 월 99,000원 |
| 의료차트 정리 | 병원 | 진료기록 요약 / 코딩 | 월 199,000원 |
| 쇼핑몰 운영봇 | 셀러 | 상품설명 생성, CS 자동응답, 재고 알림 | 월 49,000원 |
| 건설 문서봇 | 건설사 | 도면 / 견적서 검토 | 월 299,000원 |
| 교사 도우미 | 교사 | 시험문제 생성, 생활기록부 초안 | 월 19,000원 |
| 세무 보조 | 세무사 | 영수증 분류, 신고서 초안 | 월 149,000원 |

```
goose (범용 엔진)
  + 분야별 레시피 10~20개
  + 전용 MCP 익스텐션 (ERP / 차트 / 워드프레스 연동)
  + 일반인용 심플 UI
  + 내 브랜드
  = 완전히 다른 제품
```

> 오픈소스 기반임을 숨기지 말고
> **"오픈소스 기반이라 안전하고 데이터가 외부로 나가지 않습니다"** 를 세일즈 포인트로.
> 특히 법무 / 의료는 보안이 최우선이라 **로컬 모델 지원이 결정적 무기.**

#### ⑥ MCP 익스텐션 개발 & 판매
난이도 ★★★☆☆ | 예상 월 30~500만원

goose 가 공식 MCP Registry 로 이전 중 → **생태계 형성 초기 = 선점 기회**

한국 시장 특화 MCP 공백:

| MCP 아이디어 | 수요 |
|---|---|
| 네이버 (검색 / 블로그 / 카페 / 스마트스토어) | 매우 높음 |
| 카카오 (톡채널 / 맵 / 페이) | 높음 |
| 쿠팡 / 11번가 셀러 API | 높음 |
| 국세청 홈택스 | 높음 |
| 공공데이터포털 | 중간 |
| 더존 / 이카운트 ERP | 기업 고단가 |
| 토스페이먼츠 / 아임포트 | 중간 |

템플릿: `examples/mcp-wiki/` (Python)
수익화: 무료 공개(평판) + 프리미엄 유료 / 기업 커스텀 개발 건당 300~1,000만원

### 3군: 본격 사업 (3개월~)

#### ⑦ 기업 AI 에이전트 구축 SI (현실적으로 가장 큰 수익)
난이도 ★★★★☆ | 예상 프로젝트당 1,000만~1억원

기업의 문제: ChatGPT 쓰고 싶지만 사내 데이터 유출 우려 / 폐쇄망 / 감사·권한 필수

| 기업 요구 | goose 솔루션 |
|---|---|
| 데이터 외부 유출 금지 | 로컬 모델 (`goose-local-inference`) |
| 온프레미스 설치 | Docker / 단일 바이너리 |
| 권한 관리 | `permission/` 세분화 |
| 감사 / 추적 | OpenTelemetry + 세션 DB |
| 보안 검증 | `security/` 인젝션 탐지 |
| 기존 시스템 연동 | MCP 커스텀 개발 |
| SSO 인증 | `oidc-proxy/` 참고 |
| 소스 검증 | 오픈소스 |

```
1단계 AI 도입 진단 컨설팅      →   300~500만원
2단계 PoC (레시피 3~5개 구현)   → 1,000~2,000만원
3단계 본구축 (전사 배포)        → 3,000만~1억원
4단계 유지보수                 →  월 200~500만원  ← 안정적 현금흐름
```

영업 한 줄: *"Linux Foundation 재단 오픈소스 기반이라 벤더 락인이 없고,
데이터가 사내를 벗어나지 않습니다."*

#### ⑧ 브랜드 SaaS (커스텀 배포판)
난이도 ★★★★★ | 예상 월 500만~1억원

```
내 브랜드 AI
├── goose 엔진 (Apache-2.0)
├── 내 로고 / UI / 완전한 한국어 지원
├── 한국 기업용 레시피 50종 내장
├── 네이버 / 카카오 / 더존 MCP 기본 장착
├── 클라우드 동기화 + 팀 협업 (부가가치)
└── 한국어 기술지원  ← 진짜 가치
```
가격: Free / Pro 29,000원 / Team 149,000원 / Enterprise 협의
경쟁우위: Cursor / Claude Code 는 한국어·한국 서비스 연동·온프레미스가 약함
→ **로컬라이제이션이 무기**

#### ⑨ 에이전트 모니터링 / 관측 SaaS
난이도 ★★★★☆ | 예상 월 300~3,000만원

`goose-sdk` 에 Observability Hook 이 이미 설계됨:
```
onRequestStart → onResponseStart → onRequestEnd
(requestId, provider, model, durationMs, usage)
```
+ Langfuse / MLflow / Laminar 튜토리얼 제공

제품: "AI 에이전트 전용 Datadog"
- 토큰 비용 실시간 추적 + 부서별 과금
- 실패 / 재시도 패턴 분석
- 프롬프트 인젝션 시도 알림
- 모델별 성능 / 비용 A/B 비교

근거: 기업이 AI 를 쓰기 시작하면 즉시 "비용 얼마?" 가 문제가 됨
→ 비용 가시화 = 즉각적 ROI = 결제 결정 빠름

#### ⑩ 레시피 마켓플레이스 (플랫폼)
난이도 ★★★★★ | 예상 월 100만~수억 (성공 시)

```
창작자 레시피 업로드 → 사용자 구매 / 구독 → 수수료 20~30%
```
goose 에 딥링크 기능이 이미 존재 (`recipe_deeplink.rs`, `goose recipe deeplink`)
→ **원클릭 설치 인프라가 준비됨**
참고: Zapier 템플릿, GPT Store, Notion 템플릿 마켓이 동일 모델로 성공

---

## 12. 추천 로드맵

```
[1~2개월]  ① 유튜브 + ② 레시피 템플릿
  비용 0원, 리스크 0, 시장 반응 테스트
  목표: 구독 1,000명 / 템플릿 매출 월 50만원
           ↓
[3~5개월]  ④ 노코드 빌더 MVP  또는  ⑤ 버티컬 제품 1개
  유튜브에서 모인 사용자에게 즉시 판매
  목표: 유료 사용자 100명 / 월 200만원
           ↓
[6~12개월] ⑦ 기업 SI (최대 현금) + ⑧ 브랜드 SaaS
  유튜브가 영업 채널로 작동 (기업 문의 유입)
  목표: SI 1건 3,000만원 + SaaS 월 500만원
```

### 핵심 전략: "콘텐츠 → 제품 → 기업" 3단 로켓

```
    유튜브 (무료, 신뢰 구축)
         ↓ 트래픽
    템플릿 / SaaS (소액 다수)
         ↓ 레퍼런스
    기업 SI / 엔터프라이즈 (고액 소수)
```

> 유튜브는 광고수익이 아니라 **영업 파이프라인**으로 작동한다.
> "이 사람이 goose 전문가다" → 기업 문의 → 수천만원 프로젝트.

---

## 13. 법적 체크리스트

| 항목 | 상태 |
|---|---|
| Apache-2.0 상업적 사용 | 가능 |
| 소스 비공개 배포 | 가능 (Copyleft 아님) |
| 라이선스 / NOTICE 파일 포함 | **필수** |
| 변경 사항 명시 | 권장 |
| goose 상표 / 로고 사용 | AAIF 소유 — 내 브랜드로 혼동 유발 금지 |
| 특허 조항 | Apache-2.0 에 특허 라이선스 포함 (안전) |

---

## 부록 A. 기여 워크플로우 (업스트림 PR 시)

`AGENTS.md` 규칙 요약:

- 이슈가 작업의 source of truth.
  [Goose Issues 보드](https://github.com/orgs/aaif-goose/projects/1)에서 Status **Ready** 확인 후 구현
- **Inbox / Needs info / Accepted·design** 상태 이슈는 구현하지 않고 논의부터
- 외부 PR 은 Ready 이슈를 링크하고 verification plan 수행 결과를 설명
- 새 이슈는 `.github/ISSUE_TEMPLATE/` 템플릿 기반 + 이슈 타입 설정
- `documentation/static/servers.json` 에 **신규 서드파티 MCP 서버 추가 불가**
  (공식 MCP Registry 로 이전 중)
- 에이전트 루프 변경 시 **레거시 + 상태머신 양쪽 모두** 구현 / 테스트
- 커밋 전: `cargo fmt` 필수, `cargo clippy --all-targets -- -D warnings` 통과
- 신규 기능 추가 시 `goose-self-test.yaml` 갱신 후
  `goose run --recipe goose-self-test.yaml` 로 검증

## 부록 B. 주요 참고 파일 위치

| 목적 | 경로 |
|---|---|
| 프로젝트 규칙 / 철학 | `AGENTS.md`, `.goosehints` |
| 커스텀 배포판 만들기 | `CUSTOM_DISTROS.md` |
| 기여 가이드 | `CONTRIBUTING.md`, `GOVERNANCE.md` |
| Docker 빌드 | `BUILDING_DOCKER.md`, `Dockerfile` |
| Linux / RISC-V 빌드 | `BUILDING_LINUX.md`, `RISCV_SETUP.md` |
| 다국어 | `I18N.md` |
| 보안 정책 | `SECURITY.md` |
| 빌드 태스크 | `Justfile` |
| 레시피 교재 | `goose-self-test.yaml`, `workflow_recipes/` |
| 실전 자동화 사례 | `buzz/` (GitHub 이슈 관리 봇) |
| MCP 서버 템플릿 | `examples/mcp-wiki/` |
| 플러그인 템플릿 | `examples/plugins/hello-hooks/` |
| 공식 문서 소스 | `documentation/docs/` |

---

## 부록 C. 한 줄 결론

> **goose 는 "AI 에이전트를 만드는 공장"이다.**
> 완성품이 아니라 공장이고, Apache-2.0 이라 공짜로 가져다
> 내 이름을 붙여 팔 수 있다.
> 활용 가치는 ① 학습 교재 ② 업무 자동화 도구 ③ 수익화 재료 세 가지다.

---

*정리: 카리나 (Claude Code)*
*저장소: https://github.com/bmshin94/goose*
