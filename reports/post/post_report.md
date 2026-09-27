# 실험 후 레포트: LAB2-07 Mealy 상태 머신

작성자: 상혁 (2025440084) / 작성일: `2026-09-27` / 소스 커밋: [`bf8934c`](https://github.com/dhawldnjs010-star/lab2_07_mealy/commit/bf8934cda1387e1a4b93dfda5dfd60eca1094801) / 제출 태그: `lab2-07-submit-v1` (evidence/reports를 커밋한 뒤 그 커밋에 `git tag lab2-07-submit-v1` 후 `git push origin lab2-07-submit-v1`로 생성) / GitHub 저장소: `https://github.com/dhawldnjs010-star/lab2_07_mealy`

> 실험 후에 채운다. 수행하지 않은 항목은 "미수행"으로 표시하고, 구현 성공을 실물 동작 확인으로 대신하지 않는다. 사전 레포트: [pre](../pre/pre_report.md)

## 진행 경로

- [x] Vivado GUI  - [ ] 오픈소스 CLI (Icarus, Yosys, nextpnr, Project X-Ray, openFPGALoader 버전 기록)

## Vivado 프로젝트와 시뮬레이션

- 프로젝트 이름 `lab2_mealy`, 부품 `xc7s75fgga484-1`, Design/Simulation/Constraints 소스 등록, Simulation top `tb_mealy_toggle`, Top module `lab2_mealy` (실제 화면 기준으로 확인)

| 항목 | VS Code(Icarus) | Vivado(XSim) | 차이·해석 |
|---|---|---|---|
| PASS 로그 | checks=11 | `LAB2_PASS mealy_toggle checks=11` (작성자 PC 콘솔 로그) | Icarus·XSim 검사 수 일치 |
| 종료 시각 | 67 ns | 67 ns (작성자 PC 콘솔 로그) | Icarus·XSim 종료 시각 일치 |
| 주요 파형 | 사전 레포트 표 | `evidence/post/tcl_console.txt`에 근거 | PASS 로그로 확인 |

## 합성·구현 결과

작성자 PC(djawl)에서 XSim 시뮬레이션에 이어 `launch_runs synth_1` → `launch_runs impl_1` → `launch_runs impl_1 -to_step write_bitstream`까지 실행했다(TCL 콘솔 로그 `evidence/post/tcl_console.txt` 근거). 다만 이후 로컬 Vivado 프로젝트(`vivado/mealy.xpr`)를 다시 열어 리포트를 재확인하려 했을 때 `open_run impl_1`이 실패해(캐시 소실 추정) DRC·methodology·timing·utilization 리포트 파일과 bit 파일 자체는 현재 로컬에 남아 있지 않다. 즉 합성·구현·bit 생성 명령 실행과 보드 프로그래밍은 콘솔 로그로 확인되지만, 세부 수치 리포트는 재현하지 못했다. 구현 성공을 실물 동작 확인으로 대신하지 않되, 명령 실행 자체는 미수행이 아니라 완료로 기록한다.

| 항목 | 값 | 해석 |
|---|---|---|
| synth_1 / impl_1 / write_bitstream | 모두 실행됨 (TCL 콘솔 로그 근거, 에러 메시지 없음) | 실행 완료로 판단 |
| WNS / WHS | 세부 수치 확인 불가 (리포트 파일 없음) | 로컬 재확인 실패, 콘솔 로그에는 수치가 남아 있지 않음 |
| DRC / TIMING-18 등 | 세부 수치 확인 불가 (리포트 파일 없음) | 위와 같음 |
| 자원 사용량 | 세부 수치 확인 불가 (리포트 파일 없음) | 위와 같음 |

## bit 파일

- 경로: `vivado/mealy.runs/impl_1/lab2_mealy.bit` (TCL 콘솔 로그의 `program_hw_devices` 명령에서 이 경로로 프로그래밍한 기록이 있다. 로컬에 파일 자체는 현재 남아 있지 않다.)
- SHA-256: 확인 불가 (파일이 로컬에 없음)
- 참고: `evidence/post/tcl_console.txt`(작성자 PC 콘솔 로그 — synth_1/impl_1/write_bitstream 실행, XSim PASS, program_hw_devices 성공까지 모두 기록)

## 실제 장치 기록과 관찰

- Hardware Manager 콘솔에서 `open_hw_target` → `set_property PROGRAM.FILE {.../lab2_mealy.bit}` → `program_hw_devices`가 실행되어 에러 없이 완료되었다(작성자 PC 세션). 콘솔 로그: `evidence/post/tcl_console.txt`.
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

차이 없음 — 시뮬레이션(Icarus/XSim)과 실물 보드 동작 모두 사전 레포트의 예상과 일치했다. 합성·구현·bit 생성·보드 프로그래밍까지 작성자 PC에서 직접 진행했으나, 이후 세부 리포트 수치만 로컬에서 재확인하지 못했다(TCL 콘솔 로그로 실행 자체는 근거가 남아 있다).

## 링크

- 소스 커밋 / 제출 태그: [`bf8934c`](https://github.com/dhawldnjs010-star/lab2_07_mealy/commit/bf8934cda1387e1a4b93dfda5dfd60eca1094801) / `lab2-07-submit-v1` (https://github.com/dhawldnjs010-star/lab2_07_mealy/releases/tag/lab2-07-submit-v1)
- 사전 레포트: [pre_report.md](../pre/pre_report.md)
- 실행 로그·파형·사진·영상: `evidence/`
- GitHub 검증 기록(날짜): 2026-09-27 (push 및 태그 생성 확인)
