# CLAUDE.md — Servo Ear (서보 소리 분석기, auto-tuning)

클라우드 Claude Code 세션용 안내서. 이 PC의 메모·시험 파일은 클라우드에 없으니, 여기 적힌 것과 저장소 코드만 믿고 작업할 것.

## 누구를 위한 앱인가
- 미쓰비시 MR-J4 서보 튜닝을 돕는 웹앱. 휴대폰 마이크(소리)·가속도 센서(진동)로 운전 소리를 재서 공진·헌팅을 판정하고 노치 필터(PB13/PB15 등)·응답성(PA09) 값을 추천.
- 화면은 MR Configurator2 풍 (파라미터 표 PA08/PA09/PB01/PB06~PB16, 체크한 값 = 녹음 이름).
- 라이브: https://s95722462-cell.github.io/auto-tuning/ (main 에 push = GitHub Pages 배포)
- 별개 앱인 '코골이 노트(noise-filter)'와 헷갈리지 말 것.
- 사용자는 한국인 비개발자(설비·PLC 담당). **답은 한국어, 쉬운 말로.** 장황한 화면 글을 싫어함.
- **새 기능·큰 변경은 기획안을 먼저 보여 주고 승인받은 뒤 만들 것.** 작은 수정·버그는 바로 해도 됨.

## 구조
- `index.html` 하나 (CSS + IIFE 스크립트). `manifest.webmanifest`, `sw.js`, `icons/`. 이 저장소가 원본.
- **push 할 때마다 `sw.js` 의 `CACHE` 번호를 올릴 것** (지금 `servoear-v9` → 다음 `servoear-v10`).
- 외부 라이브러리: cdnjs jszip 3.10.1 (ZIP 공유), qrcodejs 1.0.0. 새 것도 cdnjs 정확한 버전만. CDN 없으면 안내.
- 저장: localStorage 접두사 `sear_` (`ls.get/ls.set`, 프로젝트별 키 `bg_/vbg_/vals_/memo_/ref_<id>`), 측정은 IndexedDB `servo-ear` (`takes`), IDB 없으면 메모리.

## 주요 기능과 코드 위치
- 소리: AnalyserNode FFT 실시간 스펙트럼, 배경 소음, 측정 평균 `specAvg`(쿵·건드림 프레임 제외), 스펙트로그램.
- 판정은 저장하지 않고 `evalTake(t)` 로 매번 계산 (기준 운전 REF → 정지 배경 BG 순), `judge()`. 50/60Hz 배수 = 전원 험, 배음은 노치 후보에서 제외.
- 진동: DeviceMotion (iOS `requestPermission`), `vibAnalyze`, `lowpass2` 로 3Hz 아래(왕복 동작)와 3~30Hz(헌팅) 분리, 첨도 높으면 덜컥·충격.
- 마이크 멈춤 대비: `wakeAudio`(터치·화면 복귀), 입력 멈춤 감시 `micDead` → 그 측정은 소리 판정 제외.
- 프로젝트(장비·축별) `PROJS`, 🚗 기준 운전 `bRef`, 겹쳐 보기 `bCmp`, A4 보고서, CSV `makeCsv`(BOM), 📤 공유(navigator.share files).
- 📥 MRC2 파라미터 불러오기 `parseParams`: 실제 MR Configurator2 1.170C 형식 — CSV 머리 `No.,Abbr.,Name,Value,Unit,Setting range` (일본어 値·設定範囲, 한국어 값·설정 범위), 16진 `0001`, 값 없음 `----`; `.prm2` = XML `<Parameters>` 안에 같은 줄(머리 없음). euc-kr/sjis.
- 🔗 휴대폰→PC 실시간: ntfy 주제 `servoear-<코드>`, PC 는 `?link=<코드>` QR. **코드 = 비밀** — 실제 코드를 커밋·로그에 남기지 말 것.
- 화면 흔들림 금지: 그래프·숫자줄은 고정 칸(grid·nowrap·말줄임), 캔버스 `fitCv`(DPR). 경고문·녹음 이름·자릿수가 바뀌어도 아래 창이 밀리면 안 됨.

## 안전 (꼭 지킬 것)
- 앱은 모터를 움직이지 않고, 앞으로도 움직이게 만들지 말 것 (PLC·앰프 통신으로 운전 명령 보내기 금지).
- 추천은 "참고값". 튜닝 운전은 사용자가 장비 앞에서 비상정지 준비 후 직접. 응답성을 한 번에 크게 올리라거나 보호 기능을 끄라는 안내를 하지 말 것. 화면 문구도 이 원칙대로.

## 규칙
- 화면 글은 짧게. 계측기 느낌 테마(`:root` 어두운 기본 + `prefers-color-scheme: light`), 선 아이콘 도구 막대.
- 휴대폰: 누르는 칸 ≥44px, 입력칸 ≥16px, `touch-action: manipulation`, 320~430px 가로 넘침 없음, 결과는 카드, 대화상자는 아래 시트.
- 파일 이름은 영문만: `ServoEar_Result_<날짜_시각>.csv`, `ServoEar_Vib_…` (아이폰 한글 파일명 깨짐).
- 비밀·개인정보·고객 장비 데이터를 저장소에 넣지 말 것.

## 시험 (클라우드에서)
- 사용자 PC의 시험 스크립트는 저장소에 없음. 필요하면 `tests/` 를 만들어 작게 넣을 것.
- 헤드리스 Chromium + Playwright/CDP, 소리 음소거. 가짜 마이크 `--use-fake-device-for-media-stream --use-file-for-fake-audio-capture=<합성 wav>` (조용 → 1240Hz 공진 → 70Hz 헌팅 등 파이썬으로 생성, 큰 파일 커밋 금지).
- 진동은 합성 `DeviceMotionEvent` 60Hz 로 dispatch. 헤드리스에서 타이머가 1초 1번으로 느려지지 않게 CDP `Emulation.setFocusEmulationEnabled` 켤 것.
- 측정 결과 읽기 전에 기록 버튼(bRec) disabled 가 풀릴 때까지 기다릴 것.
- 중계는 로컬 가짜 ntfy(POST 저장 + `/sse`)로; 실제 ntfy.sh 는 가끔만.
- CSV 저장 클릭 시험은 내려받기 폴더를 임시 폴더로 지정할 것 (사용자 Downloads 에 쌓인 적 있음).
- 확인할 것: JS 오류 0, 조용/공진/헌팅 판정, MRC2 CSV·prm2 읽기, 화면 흔들림 0px(측정 중 요소 크기 변화), 320/390px 넘침, 다크/라이트.
- 실제 장비·휴대폰 확인은 사용자가 현장에서.

## Git
- 커밋 메시지는 한국어로 무엇이 바뀌었는지.
- 시작 전 `git pull --rebase`, 강제 push 금지. main 에 push = 배포.
- push 뒤 라이브 확인: `https://s95722462-cell.github.io/auto-tuning/sw.js` 의 CACHE 가 새 번호인지, JS 오류 없는지.
