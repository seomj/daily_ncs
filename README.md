# 🚀 Gemini-Based NCS Automation Pipeline

Google Gemini API와 GitHub Actions를 결합하여 매일 아침 학습 콘텐츠를 자동 생성하고 레포지토리에 커밋하는 **데이터 파이프라인**입니다.

## 🏗️ System Architecture

1. **Trigger**: GitHub Actions `schedule` (cron) 및 `workflow_dispatch` (수동 실행)
2. **Environment**: `ubuntu-latest` 가상 머신에서 Python 3.11 런타임 구성
3. **Inference**: `google-genai` SDK를 통해 `prompt.txt` 기반의 문제/정답 데이터 생성
4. **Processing**: Python 스크립트를 이용한 마크다운 파싱 및 파일 시스템 분리 저장
5. **Deployment**: `github-actions[bot]`을 이용한 자동 `git commit` 및 `push`

## 🚀 Quick Start

본 파이프라인을 자신의 레포지토리에서 활성화하는 순서입니다.

### 1. 레포지토리 구성
- 본 저장소를 `Fork`하거나, 새 저장소를 만들어 아래 파일들을 업로드합니다.
  - `.github/workflows/ncs_automation.yaml`
  - `prompt.txt` (본인의 학습 목표에 맞게 수정 가능)

### 2. API Key 등록 (필수)
- [Google AI Studio](https://aistudio.google.com/)에서 발급받은 키를 GitHub `Settings` > `Secrets` > `Actions`에 `GEMINI_API_KEY`라는 이름으로 등록합니다.

### 3. 워크플로우 활성화 및 수동 실행
- 리포지토리 상단의 **`Actions`** 탭을 클릭합니다.
- 왼쪽 목록에서 `Daily NCS Problem Generator`를 선택합니다.
- 오른쪽의 **`Run workflow`** 버튼을 눌러 즉시 첫 번째 문제를 생성해 봅니다.
  - *참고: 처음 실행 시 파일 생성 및 커밋 권한(403 에러)이 발생한다면, `Settings` > `Actions` > `General`에서 `Workflow permissions`를 `Read and write permissions`로 변경하세요.*

### 4. 결과 확인
- 실행이 완료되면 메인 페이지의 `problems/`와 `answers/` 폴더에 날짜별로 생성된 마크다운 파일이 업로드된 것을 확인할 수 있습니다.


## 🔑 Setup: API Key & Secrets

본 프로젝트를 정상적으로 실행하려면 Google AI Studio에서 발급받은 API 키를 GitHub Secrets에 등록해야 합니다.

1. **API 키 발급**: [Google AI Studio](https://aistudio.google.com/)에서 `Get API key` 클릭 후 키 생성.
2. **GitHub Secrets 등록**:
   - 본 리포지토리의 `Settings` > `Secrets and variables` > `Actions` 이동.
   - `New repository secret` 클릭.
   - **Name**: `GEMINI_API_KEY`
   - **Secret**: 발급받은 API 키 붙여넣기.

## 🛠️ Configuration Guide (`ncs_automation.yaml`)

워크플로우 파일(`ncs_automation.yaml`)을 본인의 환경에 맞게 수정하는 방법입니다.

### 1. 실행 시간 변경 (Cron)
기본 설정은 한국 시간 기준 매일 오전 8시(`0 23 * * *` UTC)입니다. 시간을 바꾸려면 `on.schedule.cron` 값을 수정하세요.
- 예: 오전 7시 실행 -> `- cron: '0 22 * * *'`

### 2. 모델 설정 및 리전 우회
현재 `gemini-2.5-flash-lite`를 사용 중입니다. 아래와 같이 모델명을 변경할 수 있습니다.
```python
# python 스크립트 내부
response = client.models.generate_content(
    model='gemini-2.5-flash', 
    contents=user_prompt
)
```

### 3. 권한 설정 (중요)
봇이 파일(문제/정답)을 직접 커밋하기 위해 아래 권한이 반드시 포함되어야 합니다.
``` YAML
permissions:
  contents: write
  ```

## 📅 Stop Logic

본 파이프라인은 자동 종료 로직이 포함되어 있지 않습니다.

훈련 종료 후 스케줄러 자체를 완전히 멈추려면 아래 과정을 수행하세요.

- `Actions` 탭 -> `Daily NCS Problem Generator` -> `...` 메뉴 -> `Disable workflow` 클릭

## 📂 Directory Structure

```text
.
├── .github/workflows/
│   └── ncs_automation.yml  # 자동화 핵심 설정
├── problems/               # 생성된 문제 파일 (.md)
├── answers/                # 생성된 정답 파일 (.md)
├── prompt.txt              # LLM 모델용 시스템 프롬프트
└── README.md               # 시스템 아키텍처 및 가이드
```