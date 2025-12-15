# Rmd_Korean_template
R markdown에서 한국어 문서를 효율적으로 생성하기 위한 환경. Noto Sans/Serif KR의 OTF 파일이 설치되었음을 전제함. MacOS/Win에서 동작하는 것을 확인함

## uhjinrmd 템플릿 설치법 (Obsidian 정리본)

### 개념 요약
- **uhjinrmd**는 R Markdown **템플릿을 담은 R 패키지**이다.
- 한 번 설치하면 프로젝트 경로와 무관하게  
  **RStudio → File → New File → R Markdown → From Template**에서 항상 사용 가능하다.
- 템플릿 수정은 **소스 폴더**, 사용은 **설치된 패키지** 기준이다.

---

### 공통 전제
- 소스 폴더 구조는 아래를 만족해야 한다.
  - `DESCRIPTION`
  - `NAMESPACE`
  - `inst/rmarkdown/templates/kor_pdfknit/`
    - `template.yaml`
    - `skeleton/skeleton.Rmd`

---

### Windows 설치

#### R 라이브러리 위치
- `C:/Users/사용자/AppData/Local/R/win-library/4.4`

#### 설치 절차
- RStudio 콘솔에서 uhjinrmd 소스 폴더로 이동
  - `setwd("C:/경로/uhjinrmd")`
- 설치 실행
  - `devtools::install()`
  - (대체 가능) `install.packages(".", repos = NULL, type = "source")`
- 성공 기준
  - 콘솔에 `* DONE (uhjinrmd)` 출력

#### 사용
- RStudio 메뉴에서  
  **File → New File → R Markdown → From Template**
- `kor_pdfknit` 또는 “한글 PDF 작성 템플릿” 선택

---

### macOS 설치

#### R 라이브러리 위치 (Frameworks 기반)
- `/Library/Frameworks/R.framework/Versions/4.4-arm64/Resources/library`

#### 설치 절차
- RStudio 콘솔에서 uhjinrmd 소스 폴더로 이동
  - `setwd("~/Documents/R/templates/uhjinrmd")` (예시)
- 설치 실행
  - `install.packages(".", repos = NULL, type = "source")`
- 성공 기준
  - 콘솔에 `* DONE (uhjinrmd)` 출력
- **중요**
  - 설치 후 **RStudio 앱을 완전히 종료 후 재실행** (템플릿 캐시 갱신)

#### 사용
- RStudio 메뉴에서  
  **File → New File → R Markdown → From Template**
- `kor_pdfknit` 또는 “한글 PDF 작성 템플릿” 선택

---

### 템플릿 수정 및 재설치
- 수정 위치
  - `inst/rmarkdown/templates/kor_pdfknit/`
- 수정 후 반드시 재설치
  - Windows: `devtools::install()`
  - macOS: `install.packages(".", repos = NULL, type = "source")`
- 필요 시 RStudio 재시작

---

### 프로젝트 폴더 관리 원칙
- 설치 완료 후 **프로젝트 폴더는 삭제 가능**
- 단, **소스 백업용 폴더 1곳은 유지 권장** (Git 관리 추천)

---

### 한 줄 요약
- **Windows는 devtools, macOS는 install.packages**로 설치  
- 설치되면 **From Template**에서 즉시 사용 가능
