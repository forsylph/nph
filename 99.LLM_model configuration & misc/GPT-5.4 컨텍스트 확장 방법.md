# you asked

그리고 codex에게 전달할 프롬프트 만들어줘

---

# chatgpt response

좋습니다.  
사용자님 **WSL Ubuntu + Codex** 기준으로 바로 쓸 수 있게 **복붙용 `config.toml` 예시**와, 그다음 **Codex에 그대로 넣을 프롬프트**까지 같이 드리겠습니다.

OpenAI 공식 문서 기준으로 Codex는 `~/.codex/config.toml` 또는 프로젝트별 `.codex/config.toml`을 읽고, `model_context_window`와 `model_auto_compact_token_limit` 같은 키를 지원합니다. GPT-5.4는 최대 **1M 토큰 컨텍스트 윈도우**를 지원하며, **272K 초과 입력부터는 장문 컨텍스트 과금 규칙**이 적용됩니다. ([OpenAI Developers](https://developers.openai.com/codex/config-reference/?utm_source=chatgpt.com))
## 1) WSL에서 넣을 `config.toml` 예시

아래 파일을 만드시면 됩니다.

```toml
# ~/.codex/config.toml

model = "gpt-5.4"

# GPT-5.4의 긴 컨텍스트 사용
model_context_window = 1050000

# 너무 길어지기 전에 자동 압축 시작
model_auto_compact_token_limit = 900000

# 선택: 추론 강도
model_reasoning_effort = "medium"

# 선택: 승인/샌드박스 기본값
approval_policy = "on-request"
sandbox_mode = "workspace-write"

# 선택: 프로젝트 루트 신뢰
projects = { "/home/forsylph/workspace" = { trust_level = "trusted" } }
```

이 설정은 Codex 설정 레퍼런스와 샘플 설정 문서에 있는 키 이름 형식에 맞춘 예시입니다. `config.toml`은 전역 경로 `~/.codex/config.toml`에 둘 수 있고, 프로젝트 안의 `.codex/config.toml`로도 덮어쓸 수 있습니다. ([OpenAI Developers](https://developers.openai.com/codex/config-reference/?utm_source=chatgpt.com))
## 2) 사용자님 환경 기준 추천값

사용자님처럼 **코드베이스 분석 / RAG / 긴 문맥 유지보수 작업**이 목적이면 처음에는 이렇게 권장합니다.

```toml
model = "gpt-5.4"
model_context_window = 1050000
model_auto_compact_token_limit = 850000
model_reasoning_effort = "medium"
approval_policy = "on-request"
sandbox_mode = "workspace-write"
```

이유는 간단합니다.

- `1050000`: GPT-5.4의 긴 컨텍스트 상한 활용 ([OpenAI Developers](https://developers.openai.com/api/docs/models/gpt-5.4?utm_source=chatgpt.com))- `850000~900000`: 너무 늦게 압축하면 세션이 무거워질 수 있어서 여유를 두는 편이 안정적
- `medium`: 코드 탐색/설계 설명에는 보통 균형이 좋음

## 3) WSL에서 실제 적용 명령어

```bash
mkdir -p ~/.codex
nano ~/.codex/config.toml
```

붙여넣고 저장한 뒤 Codex를 다시 실행하면 됩니다. Codex는 이 설정 파일을 읽어 동작합니다. ([OpenAI Developers](https://developers.openai.com/codex/config-reference/?utm_source=chatgpt.com))
## 4) 일회성 테스트만 할 때

파일 수정 없이 한 번만 시험하려면 CLI에서 `--config`로 덮어쓸 수 있습니다. 공식 고급 설정 문서에서 이 방식이 지원됩니다. ([OpenAI Developers](https://developers.openai.com/codex/config-reference/?utm_source=chatgpt.com))
```bash
codex \
  --config 'model="gpt-5.4"' \
  --config 'model_context_window=1050000' \
  --config 'model_auto_compact_token_limit=900000'
```

## 5) 비용 주의

GPT-5.4는 1M 컨텍스트를 지원하지만, **입력이 272K를 넘으면 더 높은 장문 컨텍스트 요금 규칙**이 적용됩니다. 그래서 항상 1M을 꽉 채워 쓰는 것보다, 정말 필요할 때만 긴 컨텍스트를 쓰는 게 좋습니다. ([OpenAI Developers](https://developers.openai.com/api/docs/models/gpt-5.4?utm_source=chatgpt.com))
---

# Codex에게 전달할 프롬프트 예시

아래는 사용자님 용도에 맞춰 바로 쓸 수 있게 만든 프롬프트입니다.

## A. 설정 파일 자동 점검 + 수정 요청 프롬프트

```text
내 WSL Ubuntu 환경에서 Codex 설정을 점검해줘.

목표:
- GPT-5.4를 사용
- 긴 컨텍스트(1M 근처)를 사용할 수 있도록 설정
- 자동 compact가 너무 늦지 않게 설정
- 기존 설정이 있으면 백업 후 최소 변경만 적용

작업 지시:
1. ~/.codex/config.toml 파일 존재 여부를 확인해.
2. 없으면 새로 생성해.
3. 있으면 현재 내용을 먼저 보여주고, 필요한 최소 수정안만 제안해.
4. 아래 값을 반영해:
   - model = "gpt-5.4"
   - model_context_window = 1050000
   - model_auto_compact_token_limit = 900000
   - model_reasoning_effort = "medium"
5. 적용 후 최종 config.toml 전체 내용을 보여줘.
6. 위험한 변경은 하지 말고, 덮어쓰기 전에 백업 파일을 만들어.
7. 마지막에 이 설정이 실제로 반영되었는지 확인하는 실행 명령도 알려줘.

답변 형식:
- 현재 상태
- 변경 전 백업 경로
- 변경 내용
- 최종 파일 내용
- 검증 명령
```

## B. 프로젝트별 설정까지 해달라는 프롬프트

```text
내 프로젝트 작업용으로 Codex 설정을 최적화해줘.

환경:
- WSL Ubuntu
- 작업 폴더: ~/workspace
- 주 용도: 대형 코드베이스 분석, RAG, 문서/코드 동시 탐색

해야 할 일:
1. 전역 설정 ~/.codex/config.toml 과 프로젝트별 설정 후보 .codex/config.toml 중 어떤 방식이 더 적절한지 판단해.
2. 가능하면 전역 설정은 보수적으로 두고, 프로젝트별 설정으로 긴 컨텍스트를 적용해.
3. 다음 값을 기준으로 설정안을 작성해:
   - model = "gpt-5.4"
   - model_context_window = 1050000
   - model_auto_compact_token_limit = 850000~900000 범위에서 적절값 선택
   - model_reasoning_effort = "medium"
4. 왜 그 값을 선택했는지 짧게 설명해.
5. 최종적으로 내가 복붙할 수 있게 완성된 TOML 파일을 출력해.

주의:
- 임의 키를 만들지 말고 실제 Codex 설정 키만 사용해.
- 설명은 짧고, 결과물은 바로 적용 가능하게 작성해.
```

## C. 사용자님 스타일에 맞춘 강한 작업 프롬프트
병원 유지보수/RAG/대형 소스 분석 쪽에 맞춘 버전입니다.

```text
너는 내 로컬 개발 파트너다.

내 환경은 WSL Ubuntu이고, Codex 설정을 GPT-5.4 긴 컨텍스트 중심으로 최적화하려고 한다.
목표는 대형 코드베이스와 문서를 한 세션에서 길게 다루는 것이다.

우선 할 일:
1. 현재 Codex 설정 파일 위치와 존재 여부 확인
2. 현재 설정 백업
3. GPT-5.4 긴 컨텍스트용 설정 반영
4. 적용 결과 검증

반영할 기본값:
- model = "gpt-5.4"
- model_context_window = 1050000
- model_auto_compact_token_limit = 900000
- model_reasoning_effort = "medium"

제약:
- 불필요한 변경 금지
- 기존 사용자 설정 최대한 보존
- 바뀌는 항목만 명확히 설명
- 최종 결과는 완성된 config.toml 형태로 출력

추가:
- 긴 컨텍스트 사용 시 비용/성능상 주의점도 3줄 이내로 요약
- 마지막에 실행 검증 명령 2개를 제시
```

---

원하시면 제가 다음 답변에서 바로  
**사용자님 홈 경로(`/home/forsylph`) 기준으로 완성된 최종 `config.toml` 한 벌**을 딱 맞게 써드리겠습니다.