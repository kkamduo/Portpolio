# 소개

## 진행 프로젝트 소개
  ### 1.NIA 빅데이터 구축사업(2022)
  1. 진안 홍삼 데이터 구축사업
       1) 개발 내용
          - 기본적인 영상처리 작업 내용들을 통해 홍삼내부의 특이점들을 도출해냄. (opencv module을 사용함)
          - 추가적으로 YOLOv5를 활용하여 학습을 진행해보았으나 초기에 정확한 특이점 구분 및 라벨링이 이루어지지 않아 난항을 겪었으나 앞서 말한 영상처리들을 통해 조금더 나은 데이터 핸들링을 통해 좋은 결과물들을 도출함.
       2) Trouble Shooting
          - 문제 배경
           : 영상의 전체 픽셀을 이용하여 평균값을 도출하다보니 영상에서 특이점의 크기 및 픽셀수에 따라 편차가 너무 큼.
          - 해결 방법
           : 초기 영상처리를 통한 특이점 도출 부분에서 영상전체의 평균픽셀을 확용하여 작업을 진행했다면 이후에는 픽셀을 국소 부분으로 나누어 평균값을 추출해내 비교하여 조금더 특이점을 쉽게 찾아낼수있었으며 영상의 크기가 컸기에 Thread를 활용하여 동시에 작업들을 진행해 최적화 하였음.
     2. 종자 구분
       1) 개발 내용
          - Threadhold 값을 통한 배경과 씨앗들을 구분해 오토라벨링 툴을 개발함. (opencv module을 사용함)
          - 1차적으로 각 영상마다 평균 픽셀값들을 계산해 Threadholding을 통해 씨앗의 대략적 위치를 도출해낸 이후 각각의 씨앗 객체에 bbox를 부여함. (opencv module을 사용함)
          - 추가적인 bbox의 수정을 위해 bbox class를 생성하였으며 각각의 위치 및 크기를 조정할수있도록 설정하여 오토라벨링의 쉬운 수정과 결함들을 제거하는 방법을 이용함. (PyQt5.GraphcisView를 통해 주로 핸들링함)
       2) Trouble Shooting
          - 문제 배경
            : 각각의 박스들을 이동시키고 크기를 조절하는데 객체에 대한 특성을 부여하지않아 조절 및 수정하기가 힘들었음.
          - 해결 방법
            : 각각의 박스들에 class를 부여하여 따로 조정할수있게하여 구분 및 확인이 편하게 이루어질수있었음. 또한 GUI에 있는 PyQt5.QLabel이나 PyQt5.QGroupBox로는 각 객체의 수정 및 보완에 있어 한계점을 발견해 PyQt5.GraphcisView를 통해 이를 보완 및 해결할 수있었음.
  
  ### 2.실시간 데이터 송수신(2023)
     1. 서버DB 연동 및 실시간 GUI를 활용한 데이터 확인
       1) 개발 내용
          - KT에서 제공하는 서버내의 DB를 활용하여 Arduino센서의 데이터를 서버로 전송. (pymysql 이용)
          - 추가적으로 실시간 데이터 확인 및 데이터 핸들링을 위하여 GUI를 만듬. (실시간 animation을 이용하기 위해 animation.FuncAnimation이라는 함수 사용)
       2) Trouble Shooting
          - 문제 배경
           : 일반적인 PyQt5에서 지원하는 QLable에는 정적인 그래프만 진행 가능하였으며 실시간으로 데이터를 적용하려고하여도 DeadLock에 걸리는 경우들이 발생함.
          - 해결 방법
           : 데이터를 실시간으로 송수신가능한 Thread를 만들고 또한 animation을 진행가능한 plot을 생성하였으며 그또한 Thread를 활용하여 실시간으로 plotting이 가능하도록 진행하였음.
