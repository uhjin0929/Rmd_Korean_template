## Windows에서 Noto 폰트 강제 삭제 가이드

### 왜 TTF/OTF 혼재가 문제를 키우는가
#### 핵심은 “형식”보다 “중복/혼재”다
- TTF와 OTF는 모두 OpenType 폰트 컨테이너다. “OTF가 무조건 옳고 TTF가 문제” 같은 단정은 맞지 않다.
- 실제로 문제를 만드는 패턴은 보통 아래 3개가 겹칠 때다.
  - 시스템 폰트(C:\Windows\Fonts) + 사용자 폰트(%LOCALAPPDATA%\Microsoft\Windows\Fonts)에 같은 패밀리가 이중 설치됨
  - 정적 폰트(여러 웨이트 OTF/TTF) + 가변 폰트(VF.ttf)가 동시에 존재/등록됨
  - 앱마다 글꼴 선택/매핑 로직이 달라서(특히 VF) “Regular/Bold/Light 매핑”이 꼬이거나 글꼴 목록 중복이 발생함
- 그래서 정답은 “OTF만 써라”가 아니라:
  - (1) 한 패밀리를 “한 장소(시스템 or 사용자) + 한 세트(정적 or 가변)”로 통일
  - (2) 특히 문제가 잦다면 VF.ttf(가변)를 제거하고 정적 세트만 남기는 쪽이 재현성/호환성에 유리한 경우가 많다

#### 언제 “OTF로 통일”이 실용적인가
- 가변 폰트(VF.ttf)를 쓰기 싫고, 웨이트별 파일로 고정해서 앱/출력 재현성을 높이고 싶을 때
- 디자인/출판/프린트 파이프라인에서 특정 정적 세트(종종 OTF)를 표준으로 쓰는 경우
- 결론: “TTF라서 문제”가 아니라 “혼재(중복 설치 + VF 포함) 상태가 문제”다

---

### 강제 삭제의 정답 전략
- Windows가 폰트를 점유(fontdrvhost 등)하면 정상 부팅 상태에서 파일 삭제가 막힐 수 있다.
- 이 경우 가장 확실한 방식은:
  1) (부팅 후) 레지스트리에서 Noto 등록 제거
  2) (복구환경 WinRE)에서 오프라인으로 파일 강제 삭제
  3) (부팅 후) 폰트 캐시 정리
- WinRE에서는 PIN이 안 먹는 경우가 흔하지만, 이 루트는 PIN과 무관하게 진행 가능(오프라인 환경)

---

### 1) (정상 부팅) 레지스트리에서 Noto 등록 제거
- 관리자 PowerShell에서 실행
- 아래는 “Noto 전체” 제거. KR만 원하면 패턴을 noto.*kr 로 좁히거나, findstr 결과를 보고 골라 삭제

```powershell
# (관리자 PowerShell)
# 1) HKLM/HKCU의 Fonts 등록값에서 'noto' 포함 항목 제거

$PATTERN = "noto"

$regPaths = @(
  "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Fonts",
  "HKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Fonts"
)

foreach ($p in $regPaths) {
  $lines = reg query $p 2>$null | Select-String -Pattern $PATTERN -CaseSensitive:$false
  foreach ($l in $lines) {
    # REG_SZ / REG_EXPAND_SZ / REG_MULTI_SZ 등 어떤 타입이든 'REG_...' 토큰 기준으로 값 이름을 안전하게 분리
    $name = ($l.Line -split '\s+REG_\w+\s+')[0].Trim()
    if ($name) { reg delete $p /v "$name" /f | Out-Null }
  }
}

# 확인(출력이 0줄이면 OK)
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Fonts" | findstr /i noto
reg query "HKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Fonts" | findstr /i noto
```

---

### 2) WinRE(복구환경)에서 오프라인 파일 삭제
#### WinRE 진입
- 로그인 화면에서 Shift 누른 상태로 “다시 시작”
- 또는(가능하면) 관리자 PowerShell에서:
```powershell
shutdown /r /o /t 0
```

#### 메뉴 경로
- 문제 해결(Troubleshoot) → 고급 옵션(Advanced options) → 명령 프롬프트(Command Prompt)

#### 2-1) Windows 드라이브 문자 확인
- WinRE에서는 Windows가 C:가 아닐 수 있다.
```bat
dir C:\Windows
dir D:\Windows
```
- Windows 폴더가 보이는 드라이브가 진짜 Windows 드라이브다(이하 W로 표기)

#### 2-2) 사용자 폴더 이름 확인
```bat
dir W:\Users
```
- 여기서 실제 사용자 폴더명을 확인(예: uus03, Dsd 사람마다 다름). 이하 U로 표기

#### 2-3) 사용자 폰트 삭제(이중 설치 제거에 중요)
```bat
del /f /q W:\Users\U\AppData\Local\Microsoft\Windows\Fonts\*Noto*
```
- 성공해도 아무 메시지가 안 나오는 게 정상(/q)
- “지정된 경로를 찾을 수 없습니다”면 W 또는 U가 다른 것(위의 dir로 재확인)

#### 2-4) 시스템 폰트 삭제
```bat
del /f /q W:\Windows\Fonts\*Noto*
```

---

### 3) (선택) WinRE에서 레지스트리 오프라인 재확인/청소
- 파일을 지웠는데 글꼴 이름이 남는 “잔상”을 싹 끊고 싶을 때
```bat
reg load HKLM\OFFSOFT W:\Windows\System32\Config\SOFTWARE
reg query "HKLM\OFFSOFT\Microsoft\Windows NT\CurrentVersion\Fonts" | findstr /i noto

:: 남아있으면 값 이름을 그대로 복사해 삭제
:: reg delete "HKLM\OFFSOFT\Microsoft\Windows NT\CurrentVersion\Fonts" /v "Noto Sans KR (TrueType)" /f

reg unload HKLM\OFFSOFT
```

---

### 4) (부팅 후) 폰트 캐시 정리
- 관리자 PowerShell
```powershell
Stop-Service FontCache -Force -ErrorAction SilentlyContinue
Stop-Service "FontCache3.0.0.0" -Force -ErrorAction SilentlyContinue

Remove-Item -Force -ErrorAction SilentlyContinue `
  "$env:WINDIR\ServiceProfiles\LocalService\AppData\Local\FontCache\*"

Remove-Item -Force -ErrorAction SilentlyContinue `
  "$env:WINDIR\System32\FNTCACHE.DAT"

Start-Service FontCache -ErrorAction SilentlyContinue
Start-Service "FontCache3.0.0.0" -ErrorAction SilentlyContinue

Restart-Computer
```

---

### 5) 최종 확인(부팅 후)
```powershell
Get-ChildItem "$env:WINDIR\Fonts" -Filter "*Noto*"
Get-ChildItem "$env:LOCALAPPDATA\Microsoft\Windows\Fonts" -Filter "*Noto*"

reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Fonts" | findstr /i noto
reg query "HKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Fonts" | findstr /i noto
```

- 위 4개가 모두 “무출력”이면 완전 제거 완료

---

### 운영 팁: 재발(재설치) 방지 방향
- Noto가 다시 깔리면 보통 다음 트리거가 원인인 경우가 많다:
  - Chrome/Google 관련 패키지
  - Android Studio/SDK
  - Adobe 계열
  - TeXLive/폰트 패키지
- “Noto를 완전 차단”할지, “KR만 차단”할지에 따라 후처리가 달라진다.
