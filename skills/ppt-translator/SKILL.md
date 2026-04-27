---
name: ppt-translator
description: >-
  Amazon Bedrock 기반 PowerPoint(.pptx) 번역. 서식·레이아웃·차트 메타(제목/축 등)를 보존하며
  텍스트를 한국어(ko) 등 대상 언어로 변환. CLI(`python -m ppt_translator.cli`)·SQLite 캐시·용어집·
  원문 언어 자동 감지. PPT/슬라이드 번역, pptx 한국어, Bedrock 프레젠테이션 번역, batch translate,
  dry-run 비용 추정.
---

# PPT Translator (한국어 변환)

## Directory Convention (경로 혼동 방지)

스킬 설치 위치는 두 가지 중 하나다. 실행 전에 먼저 존재 여부를 확인한다.

| 케이스 | 경로 | 비고 |
|--------|------|------|
| **A. 글로벌 스킬** | `~/.claude/skills/ppt-translator` | Claude Code 기본 설치 위치 (대부분의 환경) |
| **B. 프로젝트 번들** | `{WORKING_DIR}/skills/ppt-translator` | `application/` 하위에 동봉된 경우 |

먼저 `ls ~/.claude/skills/ppt-translator/SKILL.md` 또는 B 경로를 확인해 **SKILL_DIR을 고정**한 뒤 그 경로를 모든 명령의 `cwd`로 사용한다. `python -m ppt_translator.cli`가 `ppt_translator` 패키지를 찾으려면 CWD가 `SKILL_DIR`이어야 한다.

## 번들 구성

`SKILL_DIR` 기준의 상대 경로:

| 항목 | 경로 |
|------|------|
| 에이전트 지침 | `SKILL.md` (이 파일) |
| 패키지 | `ppt_translator/` |
| CLI | `python -m ppt_translator.cli` (Click) |
| 의존성 | `requirements.txt` |
| 용어집 예시 | `glossary.yaml` — CWD에 `./glossary.yaml` 이 있으면 자동 탐색 |

전체 옵션: `python -m ppt_translator.cli --help` 및 각 서브커맨드 `--help`.
구조·클래스: `ppt_translator/ppt_handler.py` (`FormattingExtractor`, `TextFrameUpdater`, `PowerPointTranslator`).
upstream 문서: [ppt-translator](https://github.com/daekeun-ml/ppt-translator)

## 데이터 흐름 (수정 시 이 순서 유지)

1. **진입**: `cli.py` (Click).
2. **오케스트레이션**: `ppt_handler.PowerPointTranslator` — 슬라이드·도형·노트·차트 대상, 서식 보존 번역.
3. **LLM**: `translation_engine.py` + `bedrock_client.py`, `config.Config` 의 모델·토큰 한도.
4. **원문**: `language_detection.py` — 자동 감지(옵션); 원문==대상이면 API 생략.
5. **차트**: `chart_handler.py` — 메타 텍스트만 번역, 수치 유지.
6. **후처리**: `post_processing.py` — `PowerPointPostProcessor` 등.
7. **캐시·비용**: `cache.py`, `pricing.py`.
8. **용어·프롬프트**: `glossary.py`, `prompts.py`.

별도 스크립트로 "간단 번역" 파이프라인을 만들지 말고 위 모듈을 확장한다.

## 한국어(ko) 기본값

`ppt_translator/config.py` 의 `Config.DEFAULT_TARGET_LANGUAGE` 는 기본 **`ko`**. 언어 미지정 시 대상은 한국어로 둔다. 한국어 폰트는 `FONT_KOREAN` / `FONT_MAP['ko']`.

## 사전 조건

- Python 3.11+
- AWS 자격 증명 + Bedrock 모델 액세스 (`AWS_REGION`, `BEDROCK_MODEL_ID` 등 — `ppt_translator/config.py`)
- 선택: `SKILL_DIR/.env` (`config` 가 패키지 상위 루트의 `.env` 를 로드)

## 실행 방법

### Claude Code Bash tool (권장)

번역 실행은 **반드시 `dangerouslyDisableSandbox: true`** 로 호출한다. 기본 샌드박스는 두 가지를 모두 막는다:
- Bedrock API 호출의 SigV4 서명이 프록시에서 손상되어 `IncompleteSignatureException` 발생
- `~/Downloads/` 등 허용 목록 밖 경로에 쓰기 불가 (`Operation not permitted`)

사전에 사용자에게 외부 네트워크/쓰기 의도를 알리고 승인받는다.

```bash
# 1) (최초 1회) 의존성 설치 — 이미 설치돼 있으면 생략
python3 -c "import pptx, boto3, click, dotenv, yaml, tenacity, rich" \
  || pip install -r "$SKILL_DIR/requirements.txt" -q

# 2) 번역 — 긴 로그는 tail 로 자른다
cd "$SKILL_DIR" && python3 -m ppt_translator.cli translate \
    INPUT.pptx -t ko --source-language en \
    -o OUTPUT_ko.pptx 2>&1 | tail -60
```

`translate` 는 정상 완료 시 마지막 줄에 `✅ Translation completed: ...` 를 찍고, 캐시 히트율·토큰 수를 요약해준다. `tail` 은 핵심 요약만 볼 수 있게 해준다.

### execute_code / subprocess (대안)

```python
import subprocess, sys, os
SKILL_DIR = "/Users/<user>/.claude/skills/ppt-translator"  # 또는 application/skills/...
subprocess.run([sys.executable, "-m", "ppt_translator.cli",
                "translate", input_file, "-t", "ko", "-o", output_file],
               cwd=SKILL_DIR, capture_output=False, text=True)
```

`capture_output=False` 를 유지해 진행 상황을 스트리밍한다.

## CLI 서브커맨드 & 옵션 레퍼런스

> **공통 주의**: 출력 경로는 `--output`이 아니라 반드시 `-o` / `--output-file` 을 사용한다.

### `translate` — 전체 슬라이드 번역

```bash
python -m ppt_translator.cli translate INPUT_FILE [OPTIONS]
```

| 옵션 | 단축 | 설명 |
|------|------|------|
| `--target-language` | `-t` | 대상 언어 (기본 `ko`) |
| `--output-file` | `-o` | 출력 파일 경로 |
| `--model-id` | `-m` | Bedrock 모델 ID |
| `--source-language` | | 원문 언어 코드 (예: `en`). 생략 시 자동 감지 |
| `--no-detect-source` | | 원문 언어 자동 감지 비활성화 (모델이 컨텍스트로 판단) |
| `--no-polishing` | | 자연어 다듬기(polishing) 비활성화 |
| `--glossary` | `-g` | 용어집 YAML 파일 (기본: CWD의 `./glossary.yaml`) |
| `--cache-backend` | | `sqlite`(기본) / `memory` / `none` |
| `--cache-path` | | SQLite 캐시 경로 |
| `--no-cache` | | 캐시 비활성화 |
| `--dry-run` | | 비용 추정만 (번역·저장 없음) |
| `--no-charts` | | 차트 메타 번역 건너뜀 |

### `translate-slides` — 특정 슬라이드만 번역

```bash
python -m ppt_translator.cli translate-slides INPUT_FILE -s "1,3,5" -t ko -o output.pptx
python -m ppt_translator.cli translate-slides INPUT_FILE -s "2-4"   -t ko -o output.pptx
```

| 옵션 | 단축 | 설명 |
|------|------|------|
| `--slides` | `-s` | 슬라이드 번호. 쉼표 `"1,3,5"` 또는 범위 `"2-4"` **(필수)** |
| (이하 `translate`와 동일) | | |

### `batch-translate` — 폴더 내 전체 .pptx 일괄 번역

```bash
python -m ppt_translator.cli batch-translate INPUT_FOLDER -t ko -o OUTPUT_FOLDER
```

| 옵션 | 단축 | 설명 |
|------|------|------|
| `--target-language` | `-t` | 대상 언어 |
| `--output-folder` | `-o` | 출력 폴더 경로 |
| `--workers` | `-w` | 병렬 처리 워커 수 (기본 4) |
| `--recursive` / `--no-recursive` | `-r` / `-R` | 하위 폴더 재귀 처리 (기본: 활성화) |
| (캐시·용어집·dry-run 등 `translate`와 동일) | | |

### `info` — 슬라이드 정보 미리보기

```bash
python -m ppt_translator.cli info INPUT_FILE
```

## 함정 & 트러블슈팅

실제 실행에서 반복되는 이슈들:

- **Bedrock `IncompleteSignatureException`** — 샌드박스가 SigV4 요청을 손상시킨다. **`dangerouslyDisableSandbox: true`** 로 실행해야 한다. (`pip install`/파일 읽기만 하는 단계에서는 샌드박스를 유지)
- **`PermissionError: ~/Downloads/...`** — 같은 원인. 출력 파일이 `~/Downloads/`, `~/Desktop/` 등 허용 목록 밖이면 샌드박스 비활성화 필요. 또는 `{WORKING_DIR}/artifacts/` 등 쓰기 가능 경로로 출력.
- **`sqlite3.OperationalError: unable to open database file`** — 기본 캐시 경로(`~/.ppt-translator/cache.db`)가 샌드박스에서 못 열린다. `--cache-backend memory` 로 우회하거나 샌드박스를 비활성화한다. memory 캐시는 세션 종료 시 사라지므로 재실행 비용이 늘어남을 유의.
- **원문 언어 감지는 1회 Bedrock 호출** — `language_detection.py` 는 `langdetect` 패키지가 아니라 Bedrock 모델에 한 번 물어본다. 원문 언어를 이미 알면 `--source-language en` 을 명시해 불필요한 호출·비용을 줄인다.
- **로그가 매우 김** — 각 슬라이드마다 INFO 로그 수십 줄. Bash tool에서 `2>&1 | tail -60` 으로 잘라 요약만 확인한다. 실패 시 더 긴 tail 로 재확인.
- **`--output` 플래그 없음** — 반드시 `-o` / `--output-file` 사용.
- **캐시 히트 278/278 등으로 토큰 0** — 이미 같은 원문을 번역했다는 뜻. `--no-cache` 또는 캐시 경로 변경으로 재번역 강제 가능.
- **차트 숫자가 바뀐 것처럼 보일 때** — `chart_handler.py` 는 메타 텍스트(제목/축)만 번역하고 수치는 그대로 둔다. 진짜 숫자가 바뀌었다면 버그이므로 `--no-charts` 로 재현 확인.
