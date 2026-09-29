# FAROS 전수조사 분석 및 활용 전략 (한국어 정리본)

> 작성: Claude Code 세션 대화 정리
> 작성일: 2026-09-29
> 대상 저장소: <https://github.com/bmshin94/FAROS>
> 원본(업스트림) 저장소: <https://github.com/OpenNSWM-Lab/FAROS>

---

## 목차

1. [프로젝트 정체](#1-프로젝트-정체)
2. [폴더 전수조사 결과](#2-폴더-전수조사-결과)
3. [핵심 아키텍처](#3-핵심-아키텍처)
4. [쉬운 설명 (비유편)](#4-쉬운-설명-비유편)
5. [설치 및 사용법](#5-설치-및-사용법)
6. [Q&A 정리](#6-qa-정리)
7. [수익화 아이디어](#7-수익화-아이디어)
8. [법적 주의사항](#8-법적-주의사항)
9. [참고 링크](#9-참고-링크)

---

## 1. 프로젝트 정체

**FAROS (智塔 · Foundation AutoResearch Operating System)**

- 한 줄 정의: **연구 주제 선정부터 감사 가능한 과학적 증거까지 이어지는 협업형 AI Scientist 시스템**
- 버전: `1.1.0-rc1` (Release Candidate)
- 테스트 배지: 백엔드 685 passed / 프론트엔드 41 passed
  (README 본문 검증 baseline은 백엔드 644 / 프론트 35로 병기됨)
- 성격: **독립 실행형 풀스택 웹 애플리케이션 + 자체 플러그인 플랫폼**
- 핵심 주장: *"FAROS is not a loose chain of LLM calls."*
  (단순 LLM 호출 체이닝이 아니라, 문헌 증거에서 출발해 `PlanPackage` 계약으로 모듈을 잇고
  ReviewX와 실제 실험 결과, 인간 검토로 폐루프를 닫는 구조)

### 연구 폐루프 (Research Loop)

```
연구 관심사
  → Idea (문헌 검색 · 증거 게이트 · 후보 아이디어)
  → PlanPackage (가설/변수/단계/승인 기준 · 인간 승인)
  → Code (실험 코드 생성 · 샌드박스 실행)
  → Experiment (지표 수집 · 그래프)
  → Paper (아웃라인 · 섹션 작성 · LaTeX/PDF)
  → ReviewX (주장-증거-측정값 일치성 감사)
  → 인간 최종 승인 또는 증거 기반 수정 (→ Idea로 회귀)
```

---

## 2. 폴더 전수조사 결과

```
FAROS/
├── README.md                     # 중/영 이중언어, Mermaid 다이어그램 포함
├── CLAUDE.md                     # 프로젝트 지침 (페르소나 설정)
├── FAROS.png                     # 배너 이미지
├── assets/readme/                # 파이프라인 스크린샷
├── backend/                      # Python / FastAPI (핵심)
│   ├── app/faros/                # 기반 런타임 (基座)
│   │   ├── blueprints/ml_paper/  # 워크플로우 설계도 (blueprint.json)
│   │   ├── profiles/             # faros_llm / faros_hybrid 프로파일
│   │   ├── skills/               # 9종 스킬 패키지 (skill.json + README)
│   │   ├── capabilities/adapters # idea_refinement / experiment /
│   │   │                         # paper_drafting / reviewer_simulation
│   │   ├── providers/            # llm / tool / execution / human
│   │   ├── registry/             # 패키지 라이프사이클, trust, compatibility
│   │   ├── runtime/              # orchestrator, state_store, event_log,
│   │   │                         # artifact_store, graph_builder, agent_executor
│   │   ├── memory/               # research_memory
│   │   ├── verification/         # 룰 엔진, preflight/blueprint/profile 검증기
│   │   ├── models/               # Blueprint/Profile/Agent/Skill/Provider 등 타입
│   │   └── api/faros_api.py      # FAROS 안정 API 면
│   ├── app/modules/              # 도메인 5모듈
│   │   ├── idea/    (19 파일)    # bfts_search, deep_reading,
│   │   │                         # evidence_relevance, reasoning_kg, seed_coach
│   │   ├── code/    (19 파일)    # execution_assessment, experiment_evidence_service
│   │   ├── paper/   (8 파일)     # service, skills/outline
│   │   ├── review/  (30 파일)    # ReviewX 엔진 — 가장 큰 모듈
│   │   └── platform/(14 파일)    # providers/runs/plan_packages/templates API
│   ├── app/code/                 # 샌드박스 · 평가 · 파이프라인
│   │   ├── sandbox/              # docker_backend, subprocess_backend, pool, trace
│   │   ├── eval/                 # static_eval, dynamic_eval, scoring
│   │   ├── context/              # repo_scanner, chunker, retriever, context_pack
│   │   └── pipeline/             # pipeline_runner, prompts
│   ├── app/api/v1/               # 레거시 호환 API (신규 확장 금지 구역)
│   ├── app/core/settings.py      # 프로바이더 설정 + Fernet 암호화 저장
│   ├── alembic/                  # DB 마이그레이션
│   ├── templates/latex/          # ICML / NeurIPS / ICLR / ACL /
│   │                             # challenge_cup / generic 템플릿
│   ├── experiments/              # reviewx_scifact / oscillator / reliability /
│   │                             # multidomain / visual_cem
│   └── tests/                    # 55+ 테스트 파일
├── frontend/                     # React 18 + TypeScript 워크벤치 (31 페이지)
├── experiments/                  # ReviewX 벤치마크 + 사람 평가 도구
│   ├── reviewx_eval/cem_bench/
│   └── reviewx_annotation_web/
├── docs/                         # DEVELOPER_GUIDE, FAROS_TODO(Phase A~D) 등
├── deploy/                       # Ubuntu + systemd + Caddy 배포 가이드
└── scripts/                      # 실행/검증/릴리즈 스크립트 13개
```

### 기술 스택

**백엔드** (`backend/requirements.txt`)

| 분류 | 패키지 |
|---|---|
| 웹 | `fastapi==0.109.0`, `uvicorn[standard]==0.27.0`, `starlette<0.36` |
| 데이터 | `sqlmodel`, `sqlalchemy>=2.0`, `alembic`, `pydantic==2.5.3` |
| LLM | `litellm==1.82.0`, `openai>=2.8.0` |
| 과학계산 | `numpy`, `scipy`, `matplotlib` |
| 문서 | `fpdf2` |
| 실행/보안 | `docker>=7.0`, `cryptography>=43` |

**프론트엔드** (`frontend/package.json`, 내부 이름 `llm-scientist-platform` v5.15.2)

| 분류 | 패키지 |
|---|---|
| 코어 | React 18, TypeScript 5, Vite 5, Tailwind 3 |
| 상태/데이터 | `@tanstack/react-query`, `react-table`, `react-virtual` |
| 시각화 | `recharts`, `@antv/g6`, `@antv/g6-extension-3d` |
| UI | Radix UI, `lucide-react`, `class-variance-authority` |
| 테스트 | Vitest, Testing Library, Playwright |

---

## 3. 핵심 아키텍처

### 3.1 4겹 추상화: Blueprint + Capability + Profile + Provider

| 개념 | 파일 위치 | 역할 |
|---|---|---|
| **Blueprint** | `app/faros/blueprints/ml_paper/blueprint.json` | 워크플로우 설계도 (노드 + 엣지 DAG) |
| **Capability** | `app/faros/capabilities/adapters/*.py` | 각 단계의 실행 어댑터 |
| **Agent / Skill** | `app/faros/skills/*/skill.json` | 실행 주체와 세부 기술 |
| **Profile** | `app/faros/profiles/faros_llm/profile.json` | 모델·프로바이더 바인딩, 메모리/검증 정책 |
| **Provider** | `app/faros/providers/*.py` | `llm` / `tool` / `execution` / `human` |

> 핵심 이점: **워크플로우 정의(JSON)와 실행 코드(Python)가 완전히 분리**되어,
> 코드 수정 없이 JSON만 바꿔서 모델 교체·워크플로우 추가가 가능하다.

### 3.2 `ml_paper` 블루프린트 (실제 내용)

| 노드 | capability | agent | skills |
|---|---|---|---|
| `idea` | `idea_refinement` | `researcher` | `literature-grounding`, `idea-analysis` |
| `experiment` | `experiment` | `experimenter` | `experiment-scaffold`, `artifact-packaging` |
| `paper` | `paper_drafting` | `writer` | `paper-outline`, `section-drafting`, `latex-assembly` |
| `review` | `reviewer_simulation` | `reviewer` | `review-critique`, `consistency-audit` |

엣지: `idea → experiment → paper → review`
지원 학회: `icml`, `neurips`, `iclr`, `acl`, `generic`
최종 산출물 계약: `paper_pdf`, `latex_zip`, `review_report`

### 3.3 품질 게이트 (`verification_rules`)

```json
[
  {"capability": "idea_refinement",     "requires": ["ideaSessionId", "selectedCandidateId", "researchDossier"]},
  {"capability": "experiment",          "requires": ["projectId", "experimentId", "executionAssessment", "experimentEvidence"]},
  {"capability": "paper_drafting",      "requires": ["paperId", "paperStatus"]},
  {"capability": "reviewer_simulation", "requires": ["reviewId", "reviewStatus"], "packs": ["review_quality"]}
]
```

LLM이 무엇을 출력하든 필수 산출물이 없으면 다음 단계로 진입하지 못한다.
**말로 때우는 것을 구조적으로 불가능하게 만든 설계.**

### 3.4 ReviewX (가장 차별화된 부분)

`backend/app/modules/review/` — 30개 파일로 구성된 최대 모듈.

| 단계 | 파일 | 역할 |
|---|---|---|
| 1 | `claim_extractor.py` | 논문에서 검증 가능한 주장 추출 |
| 2 | `evidence_graph.py` | 주장 ↔ 인용 증거 매핑 |
| 3 | `mismatch_scorer.py` | 주장 / 증거 / 실제 측정값 불일치 점수화 |
| 4 | `effect_statistics.py` | 효과 크기 통계 검증 |
| 5 | `revision_planner.py`, `plan_delta.py` | 실행 가능한 수정 계획 생성 |
| 6 | `human_signoff.py`, `audit_chain.py` | 인간 최종 승인 + 감사 체인 |

부가: `guardrails.py`, `risk_analyzer.py`, `visual_evidence.py`,
`competition_evidence.py`, `experiment_feedback.py`, `reviewer_auth.py`

### 3.5 외부 연동

**문헌 검색 소스** (`backend/app/services/search_service.py`)

| 소스 | 엔드포인트 | 키 필요 |
|---|---|---|
| Semantic Scholar | `api.semanticscholar.org/graph/v1` | 선택 (rate limit 완화) |
| arXiv | `export.arxiv.org/api/query` | 불필요 |
| OpenAlex | `api.openalex.org/works` | 불필요 (polite pool 권장) |
| Crossref | `api.crossref.org/works` | 불필요 |

**LLM 프로바이더** (`backend/app/core/settings.py`, 기본 Base URL)

| 프로바이더 | Base URL |
|---|---|
| moonshot / kimi | `https://api.moonshot.cn/v1` |
| openai | `https://api.openai.com/v1` |
| anthropic / claude | `https://api.anthropic.com/v1` |
| deepseek | `https://api.deepseek.com/v1` |
| zhipu / bigmodel | `https://open.bigmodel.cn/api/paas/v4` |
| **qwen (권장)** | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| minimax | `https://api.minimaxi.com/anthropic` |

코드상 기본값은 `moonshot`, README 권장 설정은 `qwen`.

### 3.6 샌드박스 (`backend/app/code/sandbox/`)

- `docker_backend.py` — 프로덕션 격리 실행
- `subprocess_backend.py` — 로컬 개발용 (기본값 `SANDBOX_DEFAULT_BACKEND=subprocess`)
- `pool.py` — 동시 실행 풀 (기본 `SANDBOX_MAX_CONCURRENT=4`)
- `trace.py` — 실행 추적

### 3.7 패키지 라이프사이클

지원 패키지 4종: `blueprint`, `agent`, `skill`, `verifier`
지원 동작: `validate` / `install` / `refresh` / `uninstall` / `rollback` / `audit` / `trust` / `compatibility`

실제로 `backend/data/faros/packages/backups/skill/api-rollback-skill/` 에
타임스탬프별 백업 스냅샷이 10개 이상 축적되어 있어 롤백이 동작함을 확인.

---

## 4. 쉬운 설명 (비유편)

### 4.1 AI 직원 4명이 일하는 회사

| 직원 | 실제 agent id | 하는 일 |
|---|---|---|
| 연구원 | `researcher` | 논문 찾고 아이디어 생성 |
| 실험자 | `experimenter` | 코드 작성 및 실행 |
| 작가 | `writer` | 논문 작성 + LaTeX 조판 |
| 심사위원 | `reviewer` | 주장-증거 일치성 감사 |
| **사람** | (human provider) | 최종 승인 · 서명 |

### 4.2 라면집 비유로 보는 4겹 구조

| FAROS 용어 | 비유 | 설명 |
|---|---|---|
| Blueprint | 레시피 | "물 끓이기 → 면 → 스프 → 계란" 순서표 |
| Capability | 조리 동작 | "끓이기", "볶기" 같은 추상 기능 |
| Skill | 세부 기술 | "계란 예쁘게 풀기" 노하우 |
| Agent | 요리사 | 실제 수행 주체 |
| Profile | 가게 설정표 | "우리 집은 인덕션 사용" |
| Provider | 실제 장비 | Qwen? GPT? Claude? |

레시피는 그대로 두고 장비만 교체할 수 있다는 것이 핵심.
`profile.json` 의 `"provider": "qwen"` 을 `"deepseek"` 로 바꾸면 코드 수정 없이 전환된다.

### 4.3 품질 게이트가 막는 것

```
일반 LLM : "저희 방법이 기존 대비 15% 성능 향상을 달성했습니다."
            → 실험을 돌리지 않았어도 이렇게 씀 (환각)

FAROS    : experimentEvidence 없음 → paper 단계 진입 차단
```

### 4.4 ReviewX 가 하는 검사

```
주장 추출 : "우리 방법이 baseline 대비 15% 향상"
증거 확인 : 인용한 [Kim et al. 2024]에 해당 내용 존재?  → 없음
실험 대조 : 실제 실행 로그의 수치는?                    → 7.2%
판정      : MISMATCH (과장)
수정안    : "15% → 7.2% 정정" 또는 "추가 실험 3회 필요"
```

---

## 5. 설치 및 사용법

### 5.1 요구사항

| 항목 | 요구 |
|---|---|
| Python | 3.11+ (필수) |
| Node.js | 18+ (필수) |
| Docker | 로컬 개발 선택 / 정식 Code·Experiment 샌드박스 필수 |
| LaTeX | `latexmk` + XeLaTeX + `ctex` + CJK 폰트 (없으면 fallback PDF만) |
| LLM Key | 최소 1개 (Qwen 권장) |

### 5.2 백엔드

```bash
git clone https://github.com/bmshin94/FAROS.git
cd FAROS/backend

python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt

uvicorn app.main:app --host 127.0.0.1 --port 8005 --reload
```

### 5.3 프론트엔드

```bash
cd FAROS/frontend
npm ci
VITE_API_BASE_URL=http://127.0.0.1:8005 npm run dev
```

- 웹 UI: `http://127.0.0.1:5176`
- API 문서: `http://127.0.0.1:8005/api/docs`

### 5.4 LLM 설정

권장: UI의 `설정 / LLM Provider` 에서 계정별 API Key 등록 (Fernet 암호화 저장).

환경변수 방식:

```bash
export ACTIVE_PROVIDER_NAME=qwen
export QWEN_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
export QWEN_API_KEY=your_api_key
```

프로덕션에서는 반드시 아래를 설정:

```bash
export FAROS_CREDENTIAL_KEY=<Fernet 키>   # 자격증명 암호화 키
# 신뢰된 리버스 프록시가 X-Faros-User 헤더 주입 → 사용자별 격리
```

> 실제 API Key 를 Git 에 커밋하지 말 것.

### 5.5 검증

```bash
cd backend && ./.venv/bin/pytest -q
cd ../frontend && npm run test -- --run && npm run build

# 배포 사전 점검
./scripts/check_deployment_dependencies.sh --role local
```

---

## 6. Q&A 정리

### Q1. 플러그인 / 스킬 / MCP 중 무엇인가?

**셋 다 아님. 독립 실행형 풀스택 웹 애플리케이션이다.**

| 질문 | 답 | 근거 |
|---|---|---|
| Claude 플러그인? | 아니오 | `.claude-plugin/` 없음 |
| Claude Skill? | 아니오 | `SKILL.md` 없음. `skill.json` 은 FAROS 자체 포맷 |
| MCP 서버? | 아니오 | MCP 관련 코드 없음 |
| 독립 웹앱? | 예 | FastAPI + React + DB |
| 플러그인 플랫폼? | 예 | 자체 4종 패키지 시스템 보유 |

혼동 원인: FAROS **자신이** blueprint/agent/skill/verifier 패키지 생태계를 가지고 있기 때문.
비유하자면 "VS Code 확장"이 아니라 "VS Code 그 자체"에 가깝다.

### Q2. API 토큰이 필요한가?

**필수.** FAROS 는 모델을 내장하지 않은 오케스트레이터다.

- 필수: LLM Provider 키 1개 이상 (Qwen / OpenAI / Anthropic / DeepSeek / Moonshot / Zhipu / MiniMax)
- 선택: 문헌 검색 API (arXiv·Crossref 는 키 불필요, Semantic Scholar 는 키 있으면 유리)
- 보안: `FAROS_CREDENTIAL_KEY` (Fernet), `X-Faros-User` 사용자 격리, UI 마스킹

비용 주의: Idea → Plan → Code → Paper → Review 전체 사이클은 LLM 호출이 수십~수백 회 발생한다.
초기 테스트는 저가 모델(DeepSeek, Qwen 등)로 시작할 것.

### Q3. 왜 GitHub 에서 주목받는가?

1. **타이밍** — "AI Scientist" 가 현재 AI 커뮤니티 최대 관심사
2. **완성도** — 800+ 파일, 685 테스트, 실동작 프론트엔드 31 페이지
3. **README 퀄리티** — 이중언어, Mermaid 다이어그램, 비교표, 스크린샷, Star History
4. **기술적 차별점** — `PlanPackage` 타입 계약으로 모듈 간 자유 텍스트 전달 제거
5. **ReviewX 유니크함** — 주장·증거·측정값 일치성 감사는 경쟁 프로젝트에 거의 없음
6. **중국 AI 커뮤니티 + Qwen 생태계** — 중국어 문서가 1급 시민
7. **대회 트랙** — `challenge_cup`, `competition_evidence` 등 대회 출품 흔적

단, GitHub 스타는 "북마크"에 가깝고 실사용자 수와 직결되지 않는다.
README 스스로도 *"release candidate for competition validation and research prototyping,
not an unsupervised replacement for researchers"* 라고 한계를 명시한다.

### Q4. 로컬 에이전트 구축에 도움이 되는가?

**"그대로 쓰기"보다 "패턴 학습"으로 매우 유용하다.**

훔쳐올 가치가 있는 패턴:

| 패턴 | 위치 | 가치 |
|---|---|---|
| Blueprint-Capability 분리 | `blueprints/*.json` + `capabilities/adapters/` | ★★★★★ |
| Provider 추상화 (human 포함) | `providers/base.py` | ★★★★★ |
| 검증 게이트 | `blueprint.json` `verification_rules` | ★★★★★ |
| 복구 가능한 장기 작업 | `runtime/state_store.py`, `event_log.py` | ★★★★ |
| 구조화 메모리 정책 | `profile.json` `memory_policy` | ★★★★ |
| 샌드박스 이중 백엔드 | `code/sandbox/` | ★★★★ |

`memory_policy` 예시:

```json
{
  "scope_strategy": "node",
  "max_history_entries": 32,
  "compaction_mode": "summary_only",
  "volatile_prefixes": ["tmp_", "draft_", "scratch_"],
  "retained_scopes": ["run"]
}
```

개발자 가이드는 "메모리를 큰 딕셔너리 하나로 퇴화시키지 말 것"을 명시적으로 금지한다.

부담 요소: 무거운 스택, 논문 도메인 특화, 라이선스 불명, 중국어 핵심 문서, RC 단계 내부 API 불안정.

추천 독해 순서:
`faros/models/*.py` → `runtime/orchestrator.py` → `providers/base.py`
→ `verification/rules.py` → `docs/DEVELOPER_GUIDE.md` §4~§5

### Q5. React 나 PHP 로 만들 수 있는가?

**React — 이미 React 로 구현되어 있다.** 프론트엔드 전체가 React 18 + TypeScript 이므로
UI 개조·한국어화는 바로 가능하다.

**PHP — 가능하지만 전면 이식은 비권장.**

| 기능 | PHP 적합도 | 비고 |
|---|---|---|
| REST API / DB / 인증 / 결제 | 좋음 | Laravel + Eloquent + Cashier |
| 백그라운드 잡 | 보통 | Laravel Queue + Horizon |
| 과학 계산 (numpy/scipy/matplotlib) | 매우 나쁨 | 대체제 사실상 없음 |
| 코드 샌드박스 | 나쁨 | 실행 대상이 Python 코드 |
| LLM 스트리밍 (SSE) | 나쁨 | 장기 연결 취약 |
| ML 생태계 (litellm, tiktoken) | 나쁨 | 부재 |

**권장 하이브리드 아키텍처:**

```
React + TypeScript        (프론트엔드)
        ↓ REST / SSE
PHP (Laravel)             비즈니스 레이어 — 회원·인증·결제·과금·관리자
        ↓ 내부 API
Python (FastAPI)          AI 레이어 — LLM 오케스트레이션·문헌검색·샌드박스·그래프·PDF
```

PHP 는 "수익 레이어", Python 은 "AI 엔진"으로 역할을 분리하는 것이 현실적이다.
PHP 단독으로 간다면 샌드박스·과학계산을 제외한
"문헌 검색 + 리뷰 리포트" 수준의 경량 버전은 충분히 구현 가능하다.

---

## 7. 수익화 아이디어

### 요약 비교표

| # | 아이디어 | 난이도 | 초기비용 | 수익화 속도 | 잠재수익 | 리스크 | 평점 |
|---|---|---|---|---|---|---|---|
| 1 | AI 팩트체커 (ReviewX 독립화) | 중 | 중 | 중간 | 매우 높음 | 낮음 | ★★★★★ |
| 2 | 도메인 문서 자동화 SaaS | 중상 | 중 | 중간 | 매우 높음 | 중 | ★★★★★ |
| 3 | 교육 콘텐츠 / 강의 | 낮음 | 매우 낮음 | 매우 빠름 | 중 | 낮음 | ★★★★★ |
| 4 | 컨설팅 / 구축 대행 | 낮음 | 낮음 | 빠름 | 높음 | 낮음 | ★★★★ |
| 5 | 오픈코어 SaaS | 매우 높음 | 높음 | 매우 느림 | 매우 높음 | 높음 | ★★★ |
| 6 | 한국 시장 로컬라이징 | 중 | 중 | 중간 | 중 | 중 | ★★★★ |

### 7.1 AI 팩트체커 — ReviewX 독립 상품화 (최우선 추천)

FAROS 6개 모듈 중 ReviewX 가 가장 유니크하고, 가장 작고, 가장 범용적이다.

**제품 흐름**

```
입력: 문서(PDF/DOCX/MD) + 근거 자료(첨부/URL/DB)
 1) 주장 추출      — 검증 가능한 명제 N개
 2) 근거 매핑      — 각 주장 ↔ 근거 위치 연결
 3) 일치성 판정    — 지지 / 반박 / 근거없음 / 과장
 4) 리포트 생성    — 신뢰도 점수 + 문제 구간 하이라이트 + 수정 제안
출력: 웹 대시보드 + PDF 리포트 + API
```

**타겟 시장 (논문 시장보다 지불 의사가 높음)**

| 시장 | 페인포인트 | 지불 의사 |
|---|---|---|
| 금융 IR / 애널리스트 | 리포트 수치 오류 = 금융사고 | 매우 높음 |
| 로펌 | 판례 인용 오류 = 소송 리스크 | 매우 높음 |
| 제약 / 임상 | 규제 문서 정확성 = 승인 여부 | 매우 높음 |
| 언론사 | 오보 = 신뢰 붕괴 | 높음 |
| 컨설팅 | 제안서 근거 검증 | 높음 |
| 대학 | 학위논문 인용 검증 | 중간 |

**수익 모델 (예시)**

| 플랜 | 가격 | 대상 |
|---|---|---|
| Free | 월 5문서 | 유입 |
| Pro | 29,000원/월 · 100문서 | 개인/프리랜서 |
| Team | 149,000원/월 · 500문서 + 협업 | 중소기업 |
| Enterprise | 별도 견적 (온프레미스 + SLA) | 대기업 (실제 수익원) |
| API | 문서당 500~2,000원 | 개발자 / SI |

**스택:** React + TS / Laravel(결제·회원) / FastAPI(AI 엔진) + pgvector 또는 Qdrant
**MVP 기간:** 2~3개월 (1인 가능)

### 7.2 도메인 특화 문서 자동화 SaaS

FAROS 파이프라인(Idea → Plan → Draft → Review)을 논문이 아닌 문서에 이식.

| 도메인 | 시장 | 경쟁 | 진입난이도 | 평점 |
|---|---|---|---|---|
| 정부 R&D 과제 제안서 | 큼 | 낮음 | 중간 | ★★★★★ |
| 특허 명세서 | 큼 | 중 | 높음 | ★★★★ |
| ESG 보고서 | 성장중 | 낮음 | 중간 | ★★★★ |
| 학위논문 서포트 | 중간 | 중 | 낮음 | ★★★★ |
| 임상시험 보고서 | 매우 큼 | 높음 | 매우 높음 | ★★★ |
| 시장조사 리포트 | 중간 | 높음 | 낮음 | ★★★ |

**최우선 후보: 정부 R&D 과제 제안서**

- 한국 시장만 연 수만 건 (NTIS, IRIS)
- 양식이 완전히 정형화되어 AI 적합도 높음
- 과제 수주 시 수억 원 규모 → 지불 의사 압도적
- 고객: 중소기업 기업부설연구소, 대학 산학협력단, 컨설팅펌
- 경쟁자 희소

```
입력: 사업공고문 PDF + 회사 소개 + 보유 기술
 1) 공고 요건 파싱 → 평가지표 추출
 2) 선행 과제 검색(NTIS) → 중복성 회피 + 차별점 도출
 3) 목표/내용/추진체계/일정/예산 자동 생성
 4) 평가위원 시뮬레이션 → 감점 요인 사전 진단
출력: 제안서 초안(HWP/DOCX) + 예상 점수 리포트
```

수익 모델: 과제당 30만원~ / 연간 구독 300만원~ / 성공보수형

### 7.3 교육 콘텐츠 (가장 빠른 현금화)

"AI 멀티에이전트 시스템 아키텍처"는 수요 대비 양질의 교육 자료가 희소하다.

| 포맷 | 가격대 | 제작기간 | 수익 잠재력 |
|---|---|---|---|
| 블로그 / 기술 아티클 | 무료 | 1주 | 간접 (브랜딩) |
| 유튜브 시리즈 | 무료 | 1개월 | 광고 + 유입 |
| 인프런 / 유데미 강의 | 8~15만원 | 2~3개월 | 좋음 |
| 전자책 / 기술서적 | 2.5~4만원 | 3개월 | 괜찮음 |
| 기업 출강 | 일 100만원~ | 준비 1개월 | 최고 |
| 부트캠프 모듈 | 계약별 | 2개월 | 좋음 |

**커리큘럼 초안**

1. 왜 단일 LLM 호출로는 부족한가 (환각·컨텍스트 한계·검증 불가)
2. 워크플로우를 데이터로 분리하기 (Blueprint 패턴, DAG 설계)
3. Provider 추상화 — 모델 종속성 탈출 (LLM/Tool/Execution/Human)
4. 품질 게이트 — LLM 출력 강제 검증 (`verification_rules`)
5. 상태·메모리·복구 (state_store / event_log / checkpoint / 메모리 scope)
6. 안전한 코드 실행 (Docker 샌드박스, 리소스 제한)
7. 실전 — 나만의 리서치 에이전트 구축

주의: 강의 자료도 "코드 복붙"이 아니라 "패턴 설명 + 직접 작성한 예제코드"로 구성하고,
원작자 크레딧을 명시할 것.

### 7.4 컨설팅 / 구축 대행

| 서비스 | 단가 | 기간 |
|---|---|---|
| AI 자동화 진단 워크숍 | 200~500만원 | 1~2주 |
| PoC 구축 | 1,000~3,000만원 | 1~2개월 |
| 본 시스템 구축 | 5,000만원~ | 3~6개월 |
| 운영 / 유지보수 | 월 200만원~ | 지속 |

타겟: 기업 R&D 조직, 연구소, 대학 산학협력단, 로펌, 제약사
전략: 7.3(콘텐츠)으로 전문가 포지셔닝 → 7.4(컨설팅)로 수익화하는 순서가 정석.

### 7.5 오픈코어 SaaS

```
오픈소스 (MIT/Apache-2.0)  : 코어 엔진, 셀프호스팅 → 커뮤니티 확보
클라우드 유료 (Managed)    : 설치 불필요, 팀 협업, SSO, 감사로그, SLA, 온프레미스 지원
```

선례: n8n, Supabase, Dify, LangSmith
현실: 커뮤니티 형성에 최소 1~2년 필요. 장기전 각오가 있을 때만 권장.

### 7.6 한국 시장 로컬라이징

FAROS 는 중국어/영어 중심이고 Qwen 특화라 한국 시장이 비어 있다.

| 포인트 | 내용 |
|---|---|
| 한국 문헌 DB | KCI, RISS, DBpia, ScienceON 연동 |
| 한국 과제 시스템 | NTIS, IRIS 연동 |
| 한글 문서 포맷 | **HWP / HWPX 생성** (한국 시장 필수, 해외 툴 미지원) |
| 국산 LLM | HyperCLOVA X, A.X, Solar |
| 학회 템플릿 | 한국정보과학회, 대한전자공학회 등 |
| 대학 규정 | 학위논문 양식, 표절 검사 연동 |

### 7.7 권장 로드맵

```
1~2개월차 : 교육 콘텐츠 시작 (블로그 → 유튜브)       — 비용 0, 시장 반응 테스트
3~5개월차 : AI 팩트체커 MVP 개발                      — React + FastAPI
6개월차~  : 컨설팅 문의 대응 시작                      — 콘텐츠가 영업 역할
1년차~    : 도메인 SaaS 또는 한국 로컬라이징 본격화
```

**핵심 원칙 3가지**

1. 작게 시작한다 — FAROS 전체가 아니라 가장 유니크한 조각(ReviewX) 하나만
2. 논문 시장이 아니라 기업 시장을 노린다 — 지불 의사가 다르다
3. 코드 복붙이 아니라 패턴 학습으로 간다 — 라이선스 리스크 원천 차단

---

## 8. 법적 주의사항

**FAROS 저장소에 LICENSE 파일이 존재하지 않는다.** (루트 및 하위 경로 전수 확인 완료)

| 항목 | 해석 |
|---|---|
| LICENSE 부재 | 기본값 = All Rights Reserved (저작권자 모든 권리 보유) |
| 상업적 이용 | 명시적 허가 없이는 법적 위험 |
| 재배포 / 파생물 배포 | 동일하게 위험 |
| 아이디어 · 아키텍처 패턴 참고 | 자유 (아이디어에는 저작권이 미치지 않음) |

**권장 경로**

| 경로 | 안전도 | 설명 |
|---|---|---|
| A. 원작자에게 라이선스 문의 | 안전 | GitHub Issue 로 상업적 이용 가능 라이선스 추가 계획 문의 |
| B. 패턴만 학습해 직접 새로 작성 | 가장 안전 (권장) | 아이디어 참고, 코드는 자체 작성 |
| C. 코드 복사 후 상업 서비스 | 위험 | 권장하지 않음 |

추가 고려사항:

- `backend/templates/latex/` 의 학회 스타일 파일(`.sty`, `.bst`)은 각 학회의 자체 라이선스를 따른다.
- 문헌 검색 API(Semantic Scholar, arXiv, OpenAlex, Crossref)는 각각의 이용약관과
  rate limit 정책을 준수해야 한다. 대량 수집·재배포 시 별도 확인 필요.
- LLM 프로바이더별 이용약관(생성물의 상업적 이용 범위)도 개별 확인이 필요하다.

> 본 문서의 법적 내용은 일반적인 정보 정리이며 법률 자문이 아니다.
> 실제 상업화 전에는 전문가 검토를 받을 것.

---

## 9. 참고 링크

### 저장소

- 이 저장소: <https://github.com/bmshin94/FAROS>
- 원본(업스트림): <https://github.com/OpenNSWM-Lab/FAROS>
- Star History: <https://star-history.com/#OpenNSWM-Lab/FAROS&Date>

### 저장소 내 주요 문서

| 문서 | 경로 |
|---|---|
| 프로젝트 개요 | `README.md` |
| 개발자 가이드 | `docs/DEVELOPER_GUIDE.md` |
| 후속 개발 로드맵 (Phase A~D) | `docs/FAROS_TODO.md` |
| 문서 총람 (중국어) | `docs/FAROS_docs_overview_zh.md` |
| Idea → Plan 하위 전달 가이드 | `docs/idea-plan-downstream-handoff-guide.md` |
| 논문 스킬 파이프라인 참조 | `docs/paper_skill_pipeline_reference_zh.md` |
| 배포 가이드 | `deploy/README.md` |
| ReviewX 평가 프레임워크 | `experiments/reviewx_eval/README.md` |
| SciFact 폐루프 실험 | `backend/experiments/reviewx_scifact/README.md` |

### 코드 독해 추천 순서 (에이전트 아키텍처 학습용)

1. `backend/app/faros/models/*.py` — 데이터 모델 설계
2. `backend/app/faros/blueprints/ml_paper/blueprint.json` — 워크플로우 정의
3. `backend/app/faros/profiles/faros_llm/profile.json` — 바인딩 및 정책
4. `backend/app/faros/runtime/orchestrator.py` — 스케줄링
5. `backend/app/faros/providers/base.py` — 프로바이더 추상화
6. `backend/app/faros/verification/rules.py` — 검증 엔진
7. `backend/app/modules/review/` — ReviewX 구현
8. `docs/DEVELOPER_GUIDE.md` §4~§5 — 안정면 / 내부면 경계 철학

### 외부 참고

- Semantic Scholar API: <https://api.semanticscholar.org/graph/v1>
- arXiv API: <https://export.arxiv.org/api/query>
- OpenAlex API: <https://api.openalex.org/works>
- Crossref API: <https://api.crossref.org/works>
- Qwen (DashScope): <https://dashscope.aliyuncs.com/compatible-mode/v1>
