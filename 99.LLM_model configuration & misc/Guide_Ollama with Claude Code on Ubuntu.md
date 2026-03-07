# 🚀 WSL + Ollama + Claude Code 클린 설치 가이드

> Windows에서 AI 개발 환경을 처음부터 끝까지 설정하기

---

## 📋 개요

이 가이드는 Windows 10/11에서 **WSL2 + Ubuntu + Ollama + Claude Code**를 처음 설치하는 사용자를 위한 단계별 가이드입니다.

**소요 시간**: 약 20-30분
**난이도**: 초급 ~ 중급

---

## 🏗️ 시스템 구성도

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              Windows 10/11                              │
├─────────────────────────────────────────────────────────────────────────┤
│                    WSL2 (Windows Subsystem for Linux)                    │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │                      Ubuntu 22.04 LTS                            │  │
│  │                                                                  │  │
│  │   ┌─────────────────────────────────────────────────────────┐  │  │
│  │   │                    Ollama Server                         │  │  │
│  │   │                    Port: 11434                           │  │  │
│  │   │                        │                                   │  │  │
│  │   │       ┌────────────────┼────────────────┐                  │  │  │
│  │   │       │                │                │                  │  │  │
│  │   │  ┌────▼────┐    ┌────▼────┐    ┌────▼────┐             │  │  │
│  │   │  │minimax  │    │  kimi   │    │  glm    │             │  │  │
│  │   │  │ -m2.5   │    │ -k2.5   │    │  -5     │             │  │  │
│  │   │  │:cloud   │    │ :cloud  │    │ :cloud  │             │  │  │
│  │   │  └────┬────┘    └────┬────┘    └────┬────┘             │  │  │
│  │   │       │              │              │                   │  │  │
│  │   │       └──────────────┼──────────────┘                   │  │  │
│  │   └───────────────────────┼──────────────────────────────────┘  │  │
│  │                           │                                      │  │
│  │   ┌───────────────────────▼──────────────────────────────┐   │  │
│  │   │                  Claude Code                            │   │  │
│  │   │              (AI Coding Assistant)                      │   │  │
│  │   │                                                          │   │  │
│  │   │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │   │  │
│  │   │  │  Subagents   │  │ Web Search   │  │ Code Edit    │   │   │  │
│  │   │  └──────────────┘  └──────────────┘  └──────────────┘   │   │  │
│  │   └────────────────────────────────────────────────────────┘   │  │
│  │                                                                  │  │
│  └──────────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────┘

                         사용자 실행 흐름
        ┌───────────────────────────────────────────────────┐
        │                                                   │
        ▼                                                   ▼
┌──────────────┐      ┌──────────────┐      ┌──────────────────────┐
│ ollama launch │  →  │ Ollama Server │  →  │   Claude Code UI     │
│   claude      │      │  :11434      │      │  (minimax/kimi/glm) │
└──────────────┘      └──────────────┘      └──────────────────────┘
```

### 구성 요소 설명

| 레이어 | 구성 요소 | 역할 | 비고 |
|--------|----------|------|------|
| **Windows** | Windows 10/11 | 호스트 OS | WSL2 지원 필요 |
| **가상화** | WSL2 | Linux 커널 | 가상화 없는 경량 VM |
| **OS** | Ubuntu 22.04 | Linux 배포판 | LTS 버전 |
| **LLM Server** | Ollama | 모델 관리/서빙 | Port 11434 |
| **모델** | minimax/glm/kimi | AI 모델 | Cloud 버전 사용 |
| **AI IDE** | Claude Code | 개발 어시스턴트 | Subagent/Web Search |

### 모델 할당 전략

```
┌─────────────────────────────────────────────────────────────┐
│                    모델 역할 분리                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌────────────────┐                                        │
│   │ HAIKU (minimax) │ ◀── 빠른 응답용                       │
│   │ • 코드 스니펫   │     간단한 질문                         │
│   │ • 간단 분석    │     경량 작업                           │
│   └────────────────┘                                        │
│                                                             │
│   ┌────────────────┐                                        │
│   │ SONNET (kimi) │ ◀── 기본 작업용                        │
│   │ • 일반 코딩    │     대화형 개발                        │
│   │ • 리팩토링    │     파일 수정                           │
│   └────────────────┘                                        │
│                                                             │
│   ┌────────────────┐                                        │
│   │ OPUS (glm)    │ ◀── 고성능 작업용                      │
│   │ • 복잡 설계   │     아키텍처 설계                      │
│   │ • 대규모 작업 │     Subagent 활용                      │
│   └────────────────┘                                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## ✅ 사전 요구사항 (Prerequisites)

### 시스템 요구사항

| 항목 | 최소 사양 | 권장 사양 |
|------|----------|----------|
| **OS** | Windows 10 (v2004+) / Windows 11 | Windows 11 최신 버전 |
| **CPU** | x64 아키텍처 | 가상화 지원 CPU |
| **RAM** | 8GB | 16GB 이상 |
| **디스크** | 20GB 여유 공간 | SSD 50GB 이상 |
| **인터넷** | 안정적인 연결 | 필수 |

### Windows 설정 확인

```powershell
# 관리자 권한 PowerShell에서 실행
# Windows 버전 확인 (2004 이상 필요)
winver

# 가상화 지원 확인 (Virtualization: Enabled 여야 함)
systeminfo | findstr /i "hypervisor"
```

---

## 🧹 0단계: 기존 설치 정리 (선택사항)

이미 WSL 또는 관련 도구를 설치한 경우, 깨끗한 상태에서 시작하려면:

```powershell
# ⚠️ 주의: 이 명령어는 기존 WSL 데이터를 모두 삭제합니다
# PowerShell (관리자)에서 실행

# WSL 완전 제거
wsl --shutdown
wsl --unregister Ubuntu-22.04
wsl --unregister Ubuntu
wsl --update

# 선택: WSL 완전 제거
# wsl --uninstall
```

---

## 📌 1단계: WSL2 설치

### 1.1 Windows 기능 활성화

**관리자 권한 PowerShell**에서 순서대로 실행:

```powershell
# 1. WSL 활성화
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 2. 가상 머신 플랫폼 활성화
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 3. Hyper-V 활성화 (선택사항이지만 권장)
dism.exe /online /enable-feature /featurename:Microsoft-Hyper-V /all /norestart

# 4. 시스템 재부팅 (필수!)
Restart-Computer
```

### 1.2 Linux 커널 업데이트

재부팅 후 **관리자 권한 PowerShell**에서:

```powershell
# 방법 1: 자동 설치 (권장)
wsl --install --no-distribution

# 방법 2: 수동 설치
# https://wslstorestorage.blob.core.windows.net/wslblob/wsl2_linux_kernel_update_package_x64.msi
# 위 MSI 파일 다운로드 후 실행
```

### 1.3 WSL2 기본 버전 설정

```powershell
# WSL2를 기본 버전으로 설정
wsl --set-default-version 2

# 확인
wsl --version
```

**예상 출력:**
```
WSL 버전: 2.x.x.x
커널 버전: 5.x.x
WSLg 버전: 1.x.x
MSRDC 버전: 1.x.x
Direct3D 버전: 1.x.x
DXCore 버전: 1.x.x
```

✅ **1단계 완료 확인:** `wsl --version` 명령어가 버전을 출력하면 성공

---

## 📌 2단계: Ubuntu 22.04 LTS 설치

### 2.1 Ubuntu 설치

**PowerShell (일반 권한)**에서:

```powershell
# Ubuntu 22.04 자동 설치
wsl --install -d Ubuntu-22.04

# 또는 Microsoft Store에서 "Ubuntu 22.04 LTS" 설치
```

### 2.2 초기 사용자 설정

설치 완료 후 터미널이 자동으로 열리면:

```bash
# 새 UNIX 사용자명 생성
Enter new UNIX username: forsylph    # 원하는 이름 입력
New password:                       # 비밀번호 입력 (화면에 안 보임)
Retype new password:                # 비밀번호 재확인
```

### 2.3 기본 도구 업데이트

```bash
# 패키지 목록 업데이트
sudo apt update

# 시스템 업그레이드
sudo apt upgrade -y

# 필수 도구 설치
sudo apt install -y curl wget git build-essential apt-transport-https ca-certificates software-properties-common

# Git 기본 설정
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### 2.4 WSL 메모리 최적화 (권장)

**Windows**에서 `C:\Users\사용자명\.wslconfig` 파일 생성:

```ini
[wsl2]
memory=8GB
processors=4
swap=2GB
localhostForwarding=true
```

```powershell
# WSL 재시작으로 적용
wsl --shutdown
```

✅ **2단계 완료 확인:** `lsb_release -a`로 Ubuntu 22.04 출력 확인

---

## 📌 3단계: Ollama 설치

### 3.1 Ollama 설치

**Ubuntu 터미널**에서:

```bash
# 공식 설치 스크립트 실행
curl -fsSL https://ollama.com/install.sh | sh

# 설치 확인
which ollama
ollama --version
```

### 3.2 Ollama 서비스 설정

```bash
# Ollama 서비스 파일 생성 (systemd)
sudo tee /etc/systemd/system/ollama.service > /dev/null <<EOF
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/local/bin/ollama serve
User=$USER
Group=$USER
Restart=always
RestartSec=3
Environment="HOME=$HOME"

[Install]
WantedBy=default.target
EOF

# 서비스 활성화 및 시작
sudo systemctl daemon-reload
sudo systemctl enable ollama
sudo systemctl start ollama

# 상태 확인
sudo systemctl status ollama
```

### 3.3 필수 모델 다운로드

```bash
# Claude Code 연동에 필요한 모델들을 미리 다운로드

# HAIKU 용 (빠른 응답)
ollama pull minimax-m2.5:cloud

# SONNET 용 (균형 잡힌 성능)
ollama pull kimi-k2.5:cloud

# OPUS 용 (고성능)
ollama pull glm-5:cloud

# 설치된 모델 목록 확인
ollama list
```

**예상 출력:**
```
NAME                    ID              SIZE      MODIFIED
minimax-m2.5:cloud      c0d5751c800f    337 B     2 minutes ago
glm-5:cloud             c313cd065935    323 B     1 minute ago
kimi-k2.5:cloud         6d1c3246c608    340 B     30 seconds ago
```

✅ **3단계 완료 확인:** `ollama list`에 3개 모델이 모두 표시되면 성공

---

## 📌 4단계: Claude Code 설치

### 4.1 Claude Code CLI 설치

```bash
# 공식 설치 스크립트 실행
curl -fsSL https://claude.ai/install.sh | bash

# 설치 확인
which claude
claude --version
```

### 4.2 Claude Code 환경변수 설정 (Ollama 연동)

```bash
# ~/.bashrc에 Ollama 연동 설정 추가
cat >> ~/.bashrc << 'EOF'

# ============================================
# Claude Code + Ollama Configuration
# ============================================

# Ollama 서버 URL
export ANTHROPIC_BASE_URL="http://127.0.0.1:11434"

# Ollama 인증 토큰
export ANTHROPIC_AUTH_TOKEN="ollama"

# 작업 유형별 모델 설정
# HAIKU: 빠른 응답용 (경량 작업, 코드 스니펫)
export ANTHROPIC_DEFAULT_HAIKU_MODEL="minimax-m2.5:cloud"

# SONNET: 균형 잡힌 성능 (일반 코딩, 리팩토링)
export ANTHROPIC_DEFAULT_SONNET_MODEL="kimi-k2.5:cloud"

# OPUS: 고성능용 (복잡한 설계, 대규모 작업)
export ANTHROPIC_DEFAULT_OPUS_MODEL="glm-5:cloud"

# ⭐ SUBAGENT: 별도 모델 지정 (필수)
# Subagent가 부모 세션과 다른 모델을 사용하도록 설정
# 반드시 glm-5:cloud로 설정해야 병렬 분석 가능
export CLAUDE_CODE_SUBAGENT_MODEL="glm-5:cloud"

# PATH에 Claude Code 추가 (설치 시 자동 추가되나, 재설치용)
export PATH="$HOME/.local/bin:$PATH"

# ============================================
# Claude Code 모델 전환 함수
# ============================================

# HAIKU 모델로 Claude Code 실행 (minimax-m2.5:cloud)
claude-haiku() {
    ANTHROPIC_MODEL=claude-haiku-4-5 claude "$@"
}

# SONNET 모델로 Claude Code 실행 (kimi-k2.5:cloud)
claude-sonnet() {
    ANTHROPIC_MODEL=claude-sonnet-4-6 claude "$@"
}

# OPUS 모델로 Claude Code 실행 (glm-5:cloud)
claude-opus() {
    ANTHROPIC_MODEL=claude-opus-4-6 claude "$@"
}

# 이전 세션을 유지하면서 모델 전환
alias haiku='ANTHROPIC_MODEL=claude-haiku-4-5 claude --continue'
alias sonnet='ANTHROPIC_MODEL=claude-sonnet-4-6 claude --continue'
alias opus='ANTHROPIC_MODEL=claude-opus-4-6 claude --continue'

# 모델 정보 확인 함수
claude-model() {
    echo "=== Claude Code 모델 설정 ==="
    echo "HAIKU:  $ANTHROPIC_DEFAULT_HAIKU_MODEL"
    echo "SONNET: $ANTHROPIC_DEFAULT_SONNET_MODEL"
    echo "OPUS:   $ANTHROPIC_DEFAULT_OPUS_MODEL"
    echo "SUBAGENT: $CLAUDE_CODE_SUBAGENT_MODEL"
}
EOF

# 설정 적용
source ~/.bashrc

# 환경변수 확인
env | grep ANTHROPIC
```

**예상 출력:**
```
ANTHROPIC_BASE_URL=http://127.0.0.1:11434
ANTHROPIC_AUTH_TOKEN=ollama
ANTHROPIC_DEFAULT_HAIKU_MODEL=minimax-m2.5:cloud
ANTHROPIC_DEFAULT_SONNET_MODEL=kimi-k2.5:cloud
ANTHROPIC_DEFAULT_OPUS_MODEL=glm-5:cloud
CLAUDE_CODE_SUBAGENT_MODEL=glm-5:cloud
```

### 4.3 모델 전환 함수 및 Alias

설정한 함수들을 사용하면 모델을 쉽게 전환할 수 있습니다:

```bash
# 모델별로 Claude Code 새로 실행
claude-haiku    # HAIKU (minimax-m2.5:cloud)
claude-sonnet   # SONNET (kimi-k2.5:cloud)
claude-opus     # OPUS (glm-5:cloud)

# 현재 세션 유지하며 모델 전환
haiku   # HAIKU로 전환
sonnet  # SONNET으로 전환 (기본값)
opus    # OPUS로 전환

# 모델 설정 확인
claude-model
```

**출력 예시:**
```
=== Claude Code 모델 설정 ===
HAIKU:  minimax-m2.5:cloud
SONNET: kimi-k2.5:cloud
OPUS:   glm-5:cloud
SUBAGENT: glm-5:cloud
```

✅ **4단계 완료 확인:** `claude --version`이 버전을 출력하고, 환경변수가 설정되면 성공

---

## 📌 5단계: 통합 테스트 및 실행

### 5.1 최종 통합 테스트

```bash
# 1. Ollama 서비스 상태 확인
sudo systemctl status ollama --no-pager

# 2. Ollama API 테스트
curl -s http://localhost:11434/api/tags | head -20

# 3. 설치된 모델 확인
ollama list

# 4. Claude Code 버전 확인
claude --version

# 5. 환경변수 최종 확인
echo "=== 환경 구성 ==="
echo "Ollama URL: $ANTHROPIC_BASE_URL"
echo "Auth Token: $ANTHROPIC_AUTH_TOKEN"
echo "HAIKU Model: $ANTHROPIC_DEFAULT_HAIKU_MODEL"
echo "SONNET Model: $ANTHROPIC_DEFAULT_SONNET_MODEL"
echo "OPUS Model: $ANTHROPIC_DEFAULT_OPUS_MODEL"
echo "SUBAGENT Model: $CLAUDE_CODE_SUBAGENT_MODEL"
```

### 5.2 Claude Code 실행

**중요**: 새 터미널 세션에서 실행하세요 (VS Code 터미널 또는 새 WSL 창):

```bash
# 방법 1: 기본 모델로 실행 (SONNET: kimi-k2.5 사용)
ollama launch claude

# 방법 2: 특정 모델 지정
ollama launch claude --model minimax-m2.5:cloud  # HAIKU 용
ollama launch claude --model kimi-k2.5:cloud     # SONNET 용
ollama launch claude --model glm-5:cloud         # OPUS 용

# 방법 3: 프롬프트와 함께 실행
ollama launch claude --model kimi-k2.5:cloud -- -p "Hello, Claude!"
```

✅ **5단계 완료 확인:** Claude Code 대화형 인터페이스가 열리면 성공

---

## ✅ 설치 완료 확인 목록

### 필수 설치 항목

- [ ] **WSL2 설치**: `wsl --version` 출력 확인
- [ ] **Ubuntu 22.04**: `lsb_release -a`에서 22.04 확인
- [ ] **Ollama**: `ollama --version` 출력 확인
- [ ] **Ollama 서비스**: `sudo systemctl status ollama`에서 active 확인
- [ ] **모델 다운로드**: `ollama list`에 3개 모델 확인
- [ ] **Claude Code**: `claude --version` 출력 확인
- [ ] **환경변수**: `env | grep ANTHROPIC`에 5개 변수 확인
- [ ] **실행 테스트**: `ollama launch claude --model kimi-k2.5:cloud` 동작 확인

---

## 🔧 문제 해결 (Troubleshooting)

### T1. WSL2 설치 실패

**증상**: "The attempted operation is not supported" 오류

**해결**:
```powershell
# BIOS에서 가상화 활성화 확인
# Windows 기능 켜기/끄기:
# - Linux용 Windows 하위 시스템 ✓
# - 가상 머신 플랫폼 ✓
# - Hyper-V ✓ (선택)

# BIOS 설정 (재부팅 시 F2/Del 키):
# Virtualization Technology (VT-x/AMD-V) → Enabled
```

### T2. Ollama 서비스 시작 실패

**증상**: `systemctl status ollama`에서 failed

**해결**:
```bash
# 로그 확인
sudo journalctl -u ollama -n 50

# 수동 실행 테스트
ollama serve

# 포트 충돌 확인
sudo lsof -i :11434
sudo kill -9 $(sudo lsof -t -i:11434)
```

### T3. 모델 다운로드 실패

**증상**: "pulling manifest"에서 멈춤

**해결**:
```bash
# 네트워크 확인
ping ollama.com

# DNS 캐시 클리어
sudo systemd-resolve --flush-caches

# 수동 다운로드 재시도
ollama pull kimi-k2.5:cloud --insecure
```

### T4. Claude Code "중첩 세션" 오류

**증상**: "Claude Code cannot be launched inside another Claude Code session"

**해결**:
```bash
# 현재 세션 종료 후 새 터미널에서 실행
exit

# 또는 VS Code의 새 터미널에서 실행
# WSL: 새 창에서 실행
```

### T5. 환경변수 미적용

**증상**: `env | grep ANTHROPIC`에 변수 없음

**해결**:
```bash
# 수동 적용
source ~/.bashrc

# 또는 새 WSL 세션 시작
wsl --terminate Ubuntu-22.04
wsl
```

---

## 🚀 빠른 시작 (Quick Start)

### 설치 후 첫 실행

```bash
# 1. WSL 시작 (Windows 터미널 또는 VS Code)
wsl

# 2. Ollama 서비스 확인 (자동 시작되지 않았을 경우)
sudo systemctl start ollama

# 3. Claude Code 실행
ollama launch claude --model kimi-k2.5:cloud

# 4. 종료 방법
# Claude Code 내부에서: /exit 또는 Ctrl+D
```

### 유용한 명령어 모음

```bash
# 모델 목록
ollama list

# 모델 삭제
ollama rm 모델명:태그

# Claude Code 도움말
claude --help

# Ollama 서비스 재시작
sudo systemctl restart ollama

# 전체 업데이트
sudo apt update && sudo apt upgrade -y
curl -fsSL https://ollama.com/install.sh | sh
curl -fsSL https://claude.ai/install.sh | bash
```

---

## 📝 모델 선택 가이드

| 모델 | 역할 | 사용 시나리오 |
|------|------|--------------|
| **minimax-m2.5:cloud** | HAIKU | 빠른 응답, 코드 스니펫, 간단한 질문 |
| **kimi-k2.5:cloud** | SONNET | 일반 코딩, 리팩토링, 문서화 (기본값) |
| **glm-5:cloud** | OPUS | 복잡한 설계, 아키텍처, 대규모 작업 |

### 모델 매핑 상세

| Claude 명칭 | Ollama 모델 | 환경변수 | 함수명 |
|-------------|-------------|----------|--------|
| `claude-haiku-4-5` | `minimax-m2.5:cloud` | `ANTHROPIC_DEFAULT_HAIKU_MODEL` | `claude-haiku()` |
| `claude-sonnet-4-6` | `kimi-k2.5:cloud` | `ANTHROPIC_DEFAULT_SONNET_MODEL` | `claude-sonnet()` |
| `claude-opus-4-6` | `glm-5:cloud` | `ANTHROPIC_DEFAULT_OPUS_MODEL` | `claude-opus()` |

### 모델 선택 예시

```bash
# 기본 작업 (SONNET) - 권장 기본값
ollama launch claude

# 빠른 응답이 필요한 작업 (HAIKU)
ollama launch claude --model minimax-m2.5:cloud

# 복잡한 분석/설계 (OPUS)
ollama launch claude --model glm-5:cloud

# 세션 유지하며 모델 변경
# (Claude Code 내부에서 /exit 후)
sonnet  # SONNET으로 재시작
opus    # OPUS로 재시작
```

**Subagent 사용 예시:**

```bash
# 1. 기본 모델로 Claude Code 실행 (SONNET: kimi-k2.5)
ollama launch claude --model kimi-k2.5:cloud

# 2. 대화창에서 subagent spawn 명령 입력:
# 복잡한 아키텍처 분석을 OPUS 모델에 위임
spawn subagent with OPUS model to analyze the codebase architecture

# 3. 여러 subagent를 병렬로 생성하여 각각 다른 영역 탐색
spawn subagents to explore auth flow, payment integration, and notification system

# 4. Web Search와 함께 사용
spawn subagents to research Postgres 18 release notes, audit queries for deprecated patterns, and create migration tasks

# 5. 경쟁사 분석 등 복잡한 리서치
spawn subagents to research how our top 3 competitors price their API tiers, compare against our current pricing, and draft recommendations
```

**Subagent 동작 방식:**

| 명령어 | 설명 |
|--------|------|
| `spawn subagent with OPUS model` | 환경변수의 OPUS 모델(glm-5:cloud)로 subagent 생성 |
| `spawn subagents to explore X, Y, and Z` | 복수의 subagent를 병렬로 생성하여 각각 탐색 |
| `spawn subagents to research...` | Web Search와 함께 사용 가능 |

**예시 흐름:**

```
# 1. 터미널에서 Claude Code 실행 (SONNET)
$ ollama launch claude --model kimi-k2.5:cloud

# 2. Claude Code 대화창에서 입력:
> spawn subagent with OPUS model to analyze database schema

# 3. Claude Code 동작:
#    - OPUS 모델(glm-5:cloud)로 subagent 생성
#    - Subagent가 schema 분석 수행
#    - 결과 반환: "Found 3 tables: users, orders, products..."

# 4. 추가 subagent 생성 (병렬):
> spawn subagents to explore auth, payment, notification modules

# 5. 3개의 subagent가 병렬로 작업 후 결과 통합 반환
```

**Subagent 실행 흐름도:**

```
사용자 입력
    ↓
┌───────────────────────────────────────┐
│  메인 Claude Code 세션 (SONNET)        │
│  ┌─────────────────────────────────┐ │
│  │ spawn subagent with OPUS model  │ │
│  │           ↓                     │ │
│  │  ┌──────────────────────┐        │ │
│  │  │ Subagent (OPUS)     │        │ │
│  │  │ ┌─────────────────┐│        │ │
│  │  │ │ 파일 탐색       ││        │ │
│  │  │ │ 코드 분석       ││        │ │
│  │  │ │ 리서치 수행     ││        │ │
│  │  │ └─────────────────┘│        │ │
│  │  │ 결과 → 메인 세션  │        │ │
│  │  └──────────────────────┘        │ │
│  └─────────────────────────────────┘ │
└───────────────────────────────────────┘
    ↓
사용자에게 결과 표시
```

**주의:** Subagent는 Claude Code 대화창 **내부**에서만 동작합니다. 터미널에서 직접 `claude` 명령어를 실행하면 중첩 세션 오류가 발생합니다!

---

## 📚 참고 자료

| 리소스 | 링크 |
|--------|------|
| **Claude Code Quickstart** | https://code.claude.com/docs/en/quickstart |
| WSL 공식 문서 | https://docs.microsoft.com/ko-kr/windows/wsl/ |
| Ollama 공식 사이트 | https://ollama.com |
| Ollama GitHub | https://github.com/ollama/ollama |
| Claude Code 문서 | https://docs.anthropic.com/ |

---

## 🎉 완료!

모든 설치가 완료되었습니다. 다음 명령어로 Claude Code를 시작하세요:

```bash
ollama launch claude --model kimi-k2.5:cloud
```

**시스템 정보:**
- OS: Ubuntu 22.04 LTS (WSL2)
- Ollama: 최신 버전
- Claude Code: 최신 버전
- Models: minimax-m2.5, glm-5, kimi-k2.5

**지원 기능:**
- ✅ Subagent (자동 트리거)
- ✅ Web Search
- ✅ 코드 편집
- ✅ 파일 탐색

---

## 📌 Appendix A: 현재 구성 요약

### 환경변수 설정 (~/.bashrc)

```bash
# Ollama 서버 연결
export ANTHROPIC_BASE_URL="http://127.0.0.1:11434"
export ANTHROPIC_AUTH_TOKEN="ollama"

# 모델 매핑
export ANTHROPIC_DEFAULT_HAIKU_MODEL="minimax-m2.5:cloud"
export ANTHROPIC_DEFAULT_SONNET_MODEL="kimi-k2.5:cloud"
export ANTHROPIC_DEFAULT_OPUS_MODEL="glm-5:cloud"
export CLAUDE_CODE_SUBAGENT_MODEL="glm-5:cloud"
```

### 사용 가능한 명령어

| 명령어 | 설명 |
|--------|------|
| `ollama launch claude` | 기본 모델(SONNET)로 실행 |
| `ollama launch claude --model {모델명}` | 특정 모델로 실행 |
| `claude-haiku` | HAIKU로 새 세션 시작 |
| `claude-sonnet` | SONNET으로 새 세션 시작 |
| `claude-opus` | OPUS로 새 세션 시작 |
| `haiku` | 현재 세션을 HAIKU로 전환 |
| `sonnet` | 현재 세션을 SONNET으로 전환 |
| `opus` | 현재 세션을 OPUS로 전환 |
| `claude-model` | 현재 모델 설정 표시 |
| `ollama list` | 설치된 모델 목록 |

### Claude Code Memory 파일

Claude Code는 다음 위치에 프로젝트별 메모리를 저장합니다:

```
~/.claude/projects/{project_name}/memory/
├── MEMORY.md              # 프로젝트 개요
├── claude_ollama_config.md # Ollama 설정 상세
├── setup_prompts.md       # 설정용 프롬프트 모음
└── github_ssh_config.md   # GitHub SSH 설정
```

### .gitignore 권장 설정

```gitignore
# Backup files
*.bak
*.bak.*
*~
*.swp
*.tmp

# OS generated files
.DS_Store
Thumbs.db

# IDE files
.idea/
.vscode/
```
