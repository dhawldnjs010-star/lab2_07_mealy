# 실험 전 레포트: LAB2-07 Mealy 상태 머신

작성자: 상혁 (2025440084) / 작성일: 2026-09-20 / 소스 커밋: `8eebadc` / workspace: `LAB1.code-workspace` (템플릿 v2.0.1) / OS: `Windows 11 Home 10.0.26200` / Python: `Python 3.14.7` / 시뮬레이터: Icarus Verilog `Icarus Verilog version 12.0 (devel) (s20150603-1539-g2693dd32b)`

> 이 레포트는 VS Code(Icarus) 시뮬레이션까지의 사전 검증이다. Vivado GUI와 실물 보드 결과는 실험 후 레포트([post](../post/post_report.md))에서 다룬다. 시각은 clk 상승 에지(5, 15, 25, … ns)의 1 ns 뒤, 즉 TB가 비교하는 시각으로 적었다.

## 목적과 예상 동작

입력이 1일 때 상태가 S0↔S1로 토글되는 Mealy 상태 머신을 설계한다. 출력이 현재 상태와 현재 입력에 조합적으로 의존하므로 클록 사이에서도 바뀐다는 점을 파형으로 확인한다.

### 포트 (`mealy_toggle.v`의 `mealy_toggle`)

| 포트 | 방향 | 비트 폭 | 설명 |
|---|---|---|---|
| clk | in | 1 | 상승 에지 기준 클록 |
| rst | in | 1 | 동기 active-high 리셋(상태만 초기화) |
| enable | in | 1 | 상태 갱신 조건. 조합 출력은 끄지 않음 |
| bit_in | in | 1 | 입력. 1이면 토글 대상, 출력에도 직접 영향 |
| state | out (reg) | 1 | 현재 상태(0=S0, 1=S1) |
| value | out (wire) | 2 | 조합 출력 = !bit_in ? 00 : (state ? 01 : 10) |

최상위(`lab2_mealy.v`, `lab2_mealy`): `enable = press`, `bit_in = switches[7]`(SW1), `led = {5'b00000, state, value}`.

### 동작 규칙과 경계 입력

- 규칙: 상태: rst이면 S0, enable && bit_in인 에지에서 state 반전. 출력: (S0, 입력0)→00, (S0, 입력1)→10, (S1, 입력0)→00, (S1, 입력1)→01.
- 정상·경계: 클록 사이에 bit_in이 바뀌면 value가 즉시 바뀐다. enable=0이어도 상태만 멈추고 출력은 입력에 따라 변한다. 입력 1인 채 리셋 에지를 지나면 state=0, value=10.
- 시간 기준: TB 클록은 10 ns 주기(상승 에지 5, 15, 25, … ns)이고 입력은 에지 1 ns 뒤에 바꾼다. 레지스터는 다음 상승 에지에서 갱신된다. 실제 보드 클록(1 kHz)은 시뮬레이션의 10 ns와 별개다.

## 소스와 테스트벤치

- 설계 top: `lab2_mealy` / 시뮬레이션 top: `tb_mealy_toggle`
- 소스: [`src/mealy_toggle.v`](../../src/mealy_toggle.v), [`src/input_frontend.v`](../../src/input_frontend.v), [`src/lab2_mealy.v`](../../src/lab2_mealy.v)
- 테스트벤치: [`sim/tb_mealy_toggle.sv`](../../sim/tb_mealy_toggle.sv)
- 제약: [`constraints/lab2_mealy.xdc`](../../constraints/lab2_mealy.xdc)
- 설정: [`simulation.json`](../../simulation.json) (sources 3개, testbench `sim/tb_mealy_toggle.sv`, simulation_top `tb_mealy_toggle`)

| 파일 | 역할 |
|---|---|
| `src/mealy_toggle.v` | 핵심 동작을 담은 코어 `mealy_toggle`. TB가 이 모듈을 직접 검사한다. |
| `src/input_frontend.v` | 버튼·스위치를 클록에 맞추는 입력 회로. 리셋 2단 해제 동기화, 버튼·스위치 2단 동기화 플립플롭, STABLE_CYCLES=20(1 kHz에서 20 ms) 안정 확인 뒤 한 클록짜리 `press` 펄스를 만든다. |
| `src/lab2_mealy.v` | 보드 top. 프런트엔드와 코어를 연결하고 LED로 출력한다. |
| `sim/tb_mealy_toggle.sv` | 입력 자극, 기대값 계산, 자동 비교(`check`), PASS/FAIL 출력, VCD 생성, watchdog. |
| `constraints/lab2_mealy.xdc` | 핀 번호·전압과 1 kHz 클록 정의. Icarus는 XDC를 읽지 않으므로 이 사전 시뮬레이션에는 사용되지 않는다. |
| `simulation.json` | VS Code 시뮬레이션 작업이 읽는 소스 목록·테스트벤치·시뮬레이션 top. |

### 테스트벤치 동작

- 자극 순서: 리셋 후 bit_in을 클록 사이(에지 1 ns 뒤)에 바꿔 즉시 출력 변화를 검사 → enable=0/1에 따른 전이 → S1에서 입력 0/1 변화 → 전이 두 번 → 입력 1인 채 rst=1 → 입력 0으로 출력 0 확인.
- 검사 횟수: 11 (reset input zero, S0 input changes between clocks, disabled state still has Mealy output, transition to S1, S1 input zero immediately, input zero holds S1, S1 pre-edge value, transition to S0, toggle again, reset state with input one, clear combinational output)
- 종료·watchdog: 마지막 검사 뒤 `finish` task가 `LAB2_PASS mealy_toggle checks=N`을 출력하고 `$finish`한다. 별도로 100000 ns(100 µs) 뒤에 `watchdog timeout`으로 `$fatal` 처리한다. 예상 종료 시각은 67 ns이다.
- 이 TB는 코어 `mealy_toggle`만 시험한다. 입력 동기화·디바운스와 실제 핀·타이밍이 통과했다는 뜻은 아니다.

### XDC 설명

`lab2_mealy.xdc`은 포트 이름을 `lab2_mealy.v`과 맞춰 핀을 지정한다. 모든 I/O는 `LVCMOS33`이고 `create_clock -name trainer_1khz -period 1000000.000 [get_ports clk]`로 주 클록을 1 kHz(주기 1,000,000 ns)로 정의하며 `set_false_path -from [get_ports {rst button sw[*]}]`로 비동기 입력을 타이밍 경로에서 제외한다.

| 포트 | 핀 | 보드 대응 |
|---|---|---|
| clk | B6 | 1 kHz 주 클록 |
| rst | K4 | 리셋 (active-high) |
| button | N8 | 스텝 버튼 |
| sw[7:0] | U4(sw[0]), V4, W1, W4, T1, U2, W3, Y1(sw[7]) | DIPSW8..DIPSW1 (DIPSW1..8 = sw[7]..sw[0]) |
| led[7:0] | N5(led[0]), M1, M3, M7, N7, M2, M4, L4(led[7]) | LED0..LED7 |

## VS Code 실행 과정

1. File → New Window → File → Open Workspace from File...로 `LAB1.code-workspace`를 연다. 확장(slang, VaporView, vscode-pdf)을 설치한다.
2. RTL·TB·XDC·`simulation.json`을 직접 입력하고 File → Save All.
3. Terminal → Run Task... → `01 Check tools`로 Git·Python·iverilog·vvp 버전을 확인한다.
4. `02 Simulate`를 실행해 `LAB2_PASS`와 종료 시각을 확인한다.
5. `03 Open waveform`으로 `build/sim/wave.vcd`를 VaporView로 연다. 신호: clk, rst, enable, bit_in, state, value[1:0] (value는 2진수, 6~16 ns와 26~37 ns 확대).

정상 실행 로그(본인 로그로 교체하고 `evidence/pre/`에 복사):

```text
LAB2_PASS mealy_toggle checks=11
sim/tb_mealy_toggle.sv:19: $finish called at 67000 (1ps)
```

- 본인 실행 로그: [`../../evidence/pre/lab2_07_normal.log`](../../evidence/pre/lab2_07_normal.log)
- VCD: [`../../evidence/pre/lab2_07_wave_normal.vcd`](../../evidence/pre/lab2_07_wave_normal.vcd)
- 파형 캡처(VaporView): `evidence/pre/lab2_07_wave_full.png`(전체 Zoom Fit), `evidence/pre/lab2_07_wave_zoom.png`(상승 에지 확대)
- 오류: 첫 실행에서 발생한 오류가 있으면 첫 오류 → 수정 → 재실행 로그 순서로 기록한다. (없으면 "없음", Python 실행 경로를 고쳤다면 그 내용 기입)

### 사전 파형 해석

| 시간 구간 | 입력 | 예상 | 실제 파형 | 해석 |
|---|---|---|---|---|
| 6 ns | rst=1, bit_in=0 | state=0, value=00 | state=0, value=00 | 리셋, 입력 0이므로 출력 00. |
| 7 ns | bit_in=1 (에지 전 1 ns 뒤, 클록 사이) | state=0, value=10 | state=0, value=10 | 경계: 에지가 없어도 입력이 바뀌자마자 value가 10(Mealy 출력). |
| 16 ns | enable=0, 에지 15 ns | state=0, value=10 | state=0, value=10 | enable=0이라 상태 유지, 출력은 입력 기준으로 계속 10. |
| 26 ns | enable=1, 에지 25 ns | state=1, value=01 | state=1, value=01 | enable && bit_in인 에지에서 S1로 전이. 같은 입력 1이라도 출력은 01로 바뀜. |
| 27 ns | bit_in=0 (클록 사이) | state=1, value=00 | state=1, value=00 | S1에서도 입력 0이면 즉시 00. |
| 36 ns | 에지 35 ns (입력 0) | state=1, value=00 | state=1, value=00 | 입력 0이라 전이하지 않고 S1 유지. |
| 37 ns | bit_in=1 (클록 사이) | state=1, value=01 | state=1, value=01 | S1에서 입력 1이면 에지 전에도 01. |
| 46 ns | 에지 45 ns | state=0, value=10 | state=0, value=10 | S1→S0 전이. 출력은 (S0, 1)=10. |
| 56 ns | 에지 55 ns | state=1, value=01 | state=1, value=01 | 다시 토글. |
| 66 ns | rst=1, bit_in=1 · 에지 65 ns | state=0, value=10 | state=0, value=10 | 리셋은 state만 0으로. assign value에는 rst가 없어 (S0, 1)=10. |
| 67 ns | bit_in=0 (클록 사이) | state=0, value=00 | state=0, value=00 | 입력을 0으로 내리면 출력이 즉시 00. |

표의 값은 상승 에지 직후 안정된 값이다. `LAB2_PASS`만 적지 않고 각 행에서 입력, 이전 상태, 다음 상태를 비교한다. 파형 캡처에서 위 시각을 확대해 본인 화면으로 확인한다.

## 코드 수정·실패·복구 실험

- 변경: 입력 1일 때 두 상태의 출력을 서로 바꾼다(`(state ? 2'b01 : 2'b10)` → `(state ? 2'b10 : 2'b01)`).
- 변경한 파일과 위치: `src/mealy_toggle.v` 12행
- 테스트벤치 기대값은 바꾸지 않는다.

```diff
-    (state ? 2'b01 : 2'b10)
+    (state ? 2'b10 : 2'b01)
```

실행 전 계산: 7 ns에 bit_in=1이 되면 S0(state=0)의 정상 출력은 10이라 {state,value}=010이다. 변경 회로는 01을 내어 001이 되고, TB의 `3'b010` 검사가 7 ns에 실패한다. 에지가 없는 시점의 조합 출력이 처음 검사되는 시각이라 가장 먼저 걸린다.

| 단계 | 소스 커밋 또는 해시 | 실행 폴더·로그 링크 | 입력·기대값·실제값 | 해석 |
|---|---|---|---|---|
| 정상 코드 | `<커밋/해시 기입>` | [normal.log](../../evidence/pre/lab2_07_normal.log) | 7 ns 기대 value=10, 실제 value=10. `LAB2_PASS mealy_toggle checks=11` | 모든 검사 통과, 67 ns 종료. |
| 지정한 RTL 변경 | `<커밋/해시 기입>` | [mod.log](../../evidence/pre/lab2_07_mod.log) | 7 ns 기대 value=10, 실제 value=01. `LAB2_FAIL S0 input changes between clocks time=7000`, `FATAL: sim/tb_mealy_toggle.sv:12: check failed` | `S0 input changes between clocks` 검사가 변경을 발견했다(로그의 time은 ps 단위, 7000 ps = 7 ns). |
| 원래 코드로 복구 | `<커밋/해시 기입>` | [recover.log](../../evidence/pre/lab2_07_recover.log) | 복구 후 전체 검사 재실행. `LAB2_PASS mealy_toggle checks=11`, `$finish called at 67000 (1ps)` | PASS와 종료 시각이 정상 실행과 같고 새 VCD를 확인한다. |

- 첫 실패 이후에는 `$fatal`로 시뮬레이션이 끝나므로 뒤의 검사는 실행되지 않는다. 변경 전후 파형은 각각 별도 폴더에 보관한다.
- 문법 오류를 경험했다면 오류 위치로 이동한 화면, 원인, 수정 내용과 재실행 로그도 이 절에 연결한다.

## 보드 실험 계획

- 부품·프로젝트: Vivado 2026.1, RTL Project `lab2_mealy`, 부품 `xc7s75fgga484-1`(정확히 -1). Design Sources: `mealy_toggle.v`, `input_frontend.v`, `lab2_mealy.v`(Copy sources 해제). Simulation Sources: `tb_mealy_toggle.sv`(Set as Top: `tb_mealy_toggle`). Constraints: `lab2_mealy.xdc`. Project Summary의 Top module name은 `lab2_mealy`이다.
- 장비 클록·제약: Combo II-DLD S75 주 클록 B6을 1 kHz로 맞추고 XDC의 `trainer_1khz`(1,000,000 ns)와 일치시킨다.
- 입력: DIPSW1=`sw[7]`=bit_in, N8=press(=enable), K4=리셋. 주 클록은 1 kHz.
- 출력: LED[2]=state, LED[1:0]=value, LED[7:3]=0.

| 조작 | 예상 LED / 동작 |
|---|---|
| K4 초기화, SW1=0 | LED[2:0]=000 |
| SW1=1 (N8 안 누름) | 010 |
| N8 한 번 | 101 |
| SW1=0 (N8 안 누름) | 100 |
| SW1=0에서 N8 한 번 | 100 |
| SW1=1 후 N8 한 번 | 010 |

버튼 없이 출력이 바뀌지만 실제 SW1은 동기화 회로를 거치므로 물리 스위치 변화와 지연 없이 같아지지는 않는다.

촬영할 장면: 보드 전체(배선·입력·출력이 함께 보이는 사진)와 위 표의 조작별 LED 상태 사진·영상. 예상되는 차이: 버튼·스위치는 동기화·안정 확인 지연이 있고, 빠른 신호는 눈으로 구분되지 않을 수 있다.

이 단계에서는 Vivado GUI와 실물 보드의 결과를 수행한 것처럼 기록하지 않는다. 합성·구현·bit 생성과 실제 장치 기록은 실험 후 레포트에서 다룬다.
