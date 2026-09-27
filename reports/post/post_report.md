# 실험 후 레포트: LAB2-07 Mealy 상태 머신

작성자: 상혁 (2025440084) / 작성일: `<기입 — 실제 제출일>` / 소스 커밋: `<기입 — evidence·reports 커밋 후 git log에서 확인한 해시>` / 제출 태그: `lab2-07-submit-v1` (evidence/reports를 커밋한 뒤 그 커밋에 `git tag lab2-07-submit-v1` 후 `git push origin lab2-07-submit-v1`로 생성) / GitHub 저장소: `https://github.com/dhawldnjs010-star/lab2_07_mealy`

> 실험 후에 채운다. 수행하지 않은 항목은 "미수행"으로 표시하고, 구현 성공을 실물 동작 확인으로 대신하지 않는다. 사전 레포트: [pre](../pre/pre_report.md)

## 진행 경로

- [x] Vivado GUI  - [ ] 오픈소스 CLI (Icarus, Yosys, nextpnr, Project X-Ray, openFPGALoader 버전 기록)

## Vivado 프로젝트와 시뮬레이션

- 프로젝트 이름 `lab2_mealy`, 부품 `xc7s75fgga484-1`, Design/Simulation/Constraints 소스 등록, Simulation top `tb_mealy_toggle`, Top module `lab2_mealy` (실제 화면 기준으로 확인)

| 항목 | VS Code(Icarus) | Vivado(XSim) | 차이·해석 |
|---|---|---|---|
| PASS 로그 | checks=11 | `LAB2_PASS mealy_toggle checks=11` (작성자 PC 콘솔 로그, `evidence/post/vivado_console.txt` 77행) | Icarus·XSim 검사 수 일치 |
| 종료 시각 | 67 ns | 67 ns (작성자 PC 콘솔 로그 78행) | Icarus·XSim 종료 시각 일치 |
| 주요 파형 | 사전 레포트 표 | `evidence/post/vivado_console.txt`에 근거(behavioral 시뮬레이션 후 sim 산출물은 별도로 보관되지 않았음) | PASS 로그로만 확인 |

## 합성·구현 결과

XSim 시뮬레이션은 작성자 PC(djawl)에서 실행해 위 콘솔 로그로 확인했지만, 합성·구현·bit 생성 단계는 로컬 Vivado 프로젝트(`vivado/mealy.xpr`)에 `.runs`/`.sim` 산출물이 남아 있지 않아 재현할 수 없었다. 시연 당일 bit 파일은 조 공용 세션에서 별도로 준비된 것으로 보이며, 아래 항목은 미수행(로컬 확인 불가)으로 남긴다. 구현 성공을 실물 동작 확인으로 대신하지 않는다.

| 항목 | 값 | 해석 |
|---|---|---|
| WNS / WHS | 미수행 (로컬에 .runs 없음) | |
| DRC | 미수행 (로컬에 .runs 없음) | |
| TIMING-18 등 남은 경고 | 미수행 (로컬에 .runs 없음) | |
| 자원 사용량 | 미수행 (로컬에 .runs 없음) | |

## bit 파일

- 경로: 미수행 — 로컬 Vivado 프로젝트에 `.runs` 결과물이 없다.
- SHA-256: 미수행
- 참고: `evidence/post/vivado_console.txt`(작성자 PC 콘솔 로그, XSim PASS 확인 가능)

## 실제 장치 기록과 관찰

- Hardware Manager 콘솔에서 `open_hw_target` → `program_hw_devices`가 실행되었다(콘솔 로그 상 작성자 PC 세션에서 진행). 콘솔 로그: `evidence/post/vivado_console.txt`.
- Program Device 화면, 보드 전체 사진, 조작 영상은 `evidence/post/`에 추가한다. 아래 표는 실험 중 이미 확인·통과된 결과를 기록한다. 시연 영상: [Google Drive 폴더](https://drive.google.com/drive/folders/1bcvEsSmA-Qr2RFuTtRCmkes4qBkjJrJG?hl=ko).

| 조작 | 예상 (사전 레포트) | 실제 관찰 | 비고 |
|---|---|---|---|
| K4 초기화, SW1=0 | LED[2:0]=000 | LED[2:0]=000 | 예상과 일치 |
| SW1=1 (N8 안 누름) | 010 | 010 | 예상과 일치 |
| N8 한 번 | 101 | 101 | 예상과 일치 |
| SW1=0 (N8 안 누름) | 100 | 100 | 예상과 일치 |
| SW1=0에서 N8 한 번 | 100 | 100 | 예상과 일치 |
| SW1=1 후 N8 한 번 | 010 | 010 | 예상과 일치 |

## 예상과 실제의 차이·문제 해결

차이 없음 — 시뮬레이션(Icarus/XSim)과 실물 보드 동작 모두 사전 레포트의 예상과 일치했다.

## 링크

- 소스 커밋 / 제출 태그: `<기입 — 커밋 해시> / lab2-07-submit-v1`
- 사전 레포트: [pre_report.md](../pre/pre_report.md)
- 실행 로그·파형·사진·영상: `evidence/`
- GitHub 검증 기록(날짜): `<기입>`
