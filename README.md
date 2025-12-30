# FPGA 이용한 실시간 동적 표적 추적 및 LFSR 기반 보안 영상 통신 시스템 구현

---

## 프로젝트 개요

본 프로젝트 OV7670 카메라 모듈과 VGA를 이용하여 실시간 동적 표적 추적 및 LFSR 기반의 보안 영상 통신 시스템을 구현하는 것입니다.

---

## 개발 환경

  - 하드웨어 : Basys3, OV7670 카메라 모듈
  - 사용 언어 : SystemVerilog
  - 개발 도구 : Vivado, Visual Studio Code

---

## System Architecture

### 1. 전체 블록 다이어그램

<img width="2014" height="750" alt="image" src="https://github.com/user-attachments/assets/2fa8cf8b-e0e4-4c13-bfeb-a5185adc818f" />
<img width="670" height="524" alt="image" src="https://github.com/user-attachments/assets/275b4707-b793-4009-b039-485b68effe20" />

### 2. 주요 모듈

  - SCCB Master : SDA/SCL 신호를 이용해 카메라를 제어하는 모듈
  - Buffer : 카메라에서 들어오는 한 프레임을 메모리에 저장
  - Motion Compare : RGB to Grayscale 변환 및 Frame Difference 기반 모션 감지
  - Motion Cordinate :  활성 픽셀 누적 연상을 통한 표적 중심 좌표 산출
  - Motion Display : 좌표 기반 표적 추적 박스 오버레이 및 영상 합성
  - LSFR : Master에서 원본 픽셀 데이터 마다 XOR 암호키 생성, Slave에서 생성하는 암호키와 동일한 타이밍에 생성

### 3. 동작 방식

<img width="960" height="540" alt="image" src="https://github.com/user-attachments/assets/b9922a7e-bad8-4894-bc7d-a2381d45da2e" />

---

## 트러블슈팅 및 고찰

### 1. 암호키 초기화 동기 실패와 신호 무결성(Metastability) 해결

- 문제상황 : Reset 버튼을 눌렀을 경우 암호화 키가 동시에 Reset되지 않는 문제 발생
- 문제원인 : 10ns의 Rising 신호가 PCLK의 상승 엣지에 falling 했을 시 Metastability(CDC) 문제가 발생하였음
- 해결방안 : 싱크로나이저를 적용하고, 리셋 신호를 40ns이상으로 유지하여 안정성을 확보
- 성과 : 암·복호화 모듈을 동시에 Reset하였음

### 2. 동적 표적 추적 시 잔상(Ghosting) 제거

- 문제상황 : 움직이는 물체가 없을 때에도 LOCK ON 마크가 남아있는 문제 발생
- 문제원인 : 중심 좌표 값이 유지되어 발생하는 문제임을 확인
- 해결방안 : motion_vali(프레임 단위) 플래그를 추가하여 움직임이 없을 때는 좌표값을 초기화함
- 성과 : 움직이는 물체가 없을 경우 LOCK ON 마크가 사라지고 원본 카메라 영상만 출력되는 것을 확인

### 3. 고찰

   1. 단순한 기능 구현보다 타이밍적인 부분을 수정하는 것이 어렵다는 것을 느꼈습니다.
   2. 팀단위의 작업에서 서로간의 소통, 협업이 중요하다는 것을 느꼈습니다.
