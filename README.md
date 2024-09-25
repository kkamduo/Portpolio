# 소개

## 진행 프로젝트 소개
  ## 1.NIA 빅데이터 구축사업
   ### 1. 진안 홍삼 데이터 구축사업
    1) 개발 내용
      - X-ray이미지를 이용한 기본적인 영상처리 작업 내용들을 통해 홍삼내부의 특이점들을 도출해냄. 
      (opencv module을 사용함)
      - 추가적으로 YOLOv5를 활용하여 학습을 진행해보았으나 초기에 정확한 특이점 구분 및 라벨링이 이루어지지 
      않아 난항을 겪었으나 앞서 말한 영상처리들을 통해 조금더 나은 데이터 핸들링을 통해 좋은 결과물들을 도출함.
    2) Trouble Shooting
      - 문제 배경
        : 영상의 전체 픽셀을 이용하여 평균값을 도출하다보니 영상마다 특이점의 크기 및 픽셀수에 따라 편차가 너무 큼.
      - 해결 방법
        : 초기 영상처리를 통한 특이점 도출 부분에서 영상전체의 평균픽셀을 확용하여 작업을 진행했다면 이후에는 픽셀을 국소 부분으로 나누어 
        평균값을 추출해내 비교하여 조금더 특이점을 쉽게 찾아낼수있었으며 영상의 크기가 컸기에 Thread를 활용하여 동시에 작업들을 진행해 최적화함.
           
  
  #### 국소 부위 추출 CODE
    for img_range in range(0,1960,1):
      new_img_line = np.array([])
      img_gray_line = img_gray[0:1080,img_range]

      img_gray_line_sort = np.array([each for each in img_gray_line if (50< each <230)])
      img_gray_mean = img_gray_line_sort.mean()

      if img_gray_mean > 0 :
          sort_min = min(img_gray_line_sort)
          sort_min_index = np.argmin(img_gray_line_sort)
          sort_max = max(img_gray_line_sort)
          sort_max_index = np.argmax(img_gray_line_sort)
          index_range = abs(sort_max_index - sort_min_index)
  
          test_a = (img_gray_mean - sort_min)
          test_b = (sort_max - img_gray_mean)
          test_c = int((test_a + test_b)/2)
  
          for line in img_gray_line:
              if line > test_c or line < 50 or line > 230:
                  line = 0
              else:
                  line = 255
              new_img_line = np.append(new_img_line, line)
              new_img_line = np.array([new_img_line])
  
          if test_img.size == 0:
              test_img = new_img_line
          else:
              test_img = np.append(test_img, new_img_line, axis = 0)
      else :
          if test_img.size == 0:
              test_img = np.zeros((1,1080))
          else:
              test_img = np.append(test_img , np.zeros((1,1080)), axis = 0)

    test_img = np.uint8(test_img.T)

  ### 2. 종자 구분
    1) 개발 내용
      - Threadhold 값을 통한 배경과 씨앗들을 구분해 오토라벨링 툴을 개발함. (opencv module을 사용함)
      - 1차적으로 각 영상마다 평균 픽셀값들을 계산해 Threadholding을 통해 씨앗의 대략적 위치를 도출해낸 이후 각각의 씨앗 객체에 bbox를 부여함. 
      (opencv module을 사용함)
      - 추가적인 bbox의 수정을 위해 bbox class를 생성하였으며 각각의 위치 및 크기를 조정할수있도록 설정하여 오토라벨링의 쉬운 수정과 결함들을 
      제거하는 방법을 이용함. (PyQt5.GraphcisView를 통해 주로 핸들링함)
    2) Trouble Shooting
      - 문제 배경
        : 각각의 박스들을 이동시키고 크기를 조절하는데 객체에 대한 특성을 부여하지않아 조절 및 수정하기가 힘들었음.
      - 해결 방법
        : 각각의 박스들에 class를 부여하여 따로 조정할수있게하여 구분 및 확인이 편하게 이루어질수있었음. 또한 GUI에 있는 PyQt5.QLabel이나 
        PyQt5.QGroupBox로는 각 객체의 수정 및 보완에 있어 한계점을 발견해 PyQt5.GraphcisView를 통해 이를 보완 및 해결할 수있었음.

  #### GrapchisItemGroup 객체 CODE
    class Testset(QGraphicsItemGroup):
      global pop_num_list
  
      #GraphicsItem 생성시 기본 Default값 지정
      def __init__(self,i,x,y,pix):
          super().__init__()
          self.cnt = i
          self.pix = pix
          self.bb = QGraphicsRectItem()
          self.pos_x = x
          self.pos_y = y
          self.bb.setRect(self.pos_x-(self.pix), self.pos_y-(self.pix), 2*self.pix, 2*self.pix)
          self.hover_flag = False
  
          #초기 좌표에 따라 박스 팬 색상설정
          if (self.pos_x-self.pix > 0 and self.pos_y-self.pix > 0 and self.pos_x+self.pix < 1200 and self.pos_y+self.pix < 900): 
              self.bb.setPen(Qt.green)
          else : self.bb.setPen(Qt.red)
  
          #좌측상단 박스 라벨 번호 추가
          self.lab = QGraphicsTextItem(str(i))
          self.lab.setPos(self.pos_x-50, self.pos_y-50)
          self.fontt = self.lab.font()
          self.fontt.setPointSize(20)
          self.lab.setFont(self.fontt)
  
          #GraphicsItem 그룹 지정 (라벨 + 박스)
          self.addToGroup(self.bb)
          self.addToGroup(self.lab)
  
          #GraphicsItem 선택,이동,HoverEvent 지정
          self.setAcceptHoverEvents(True)
          self.setFlag(QGraphicsItem.ItemIsMovable)
          self.setFlag(QGraphicsItem.ItemIsSelectable)
          self.box_pos = (self.pos_x, self.pos_y)
  
      #GraphicsItem 마우스 이벤트 지정
      def mousePressEvent(self, event):
          if event.buttons() == Qt.RightButton:
              ui.img_view_scene.removeItem(self)
              pop_num_list.append(self.cnt)
              pop_num_list.sort()
              ui.img_view_scene.boxes[self.cnt-1] = None
  
          if event.buttons() == Qt.LeftButton:
              self.hover_flag = True
              
      #GraphicsItem Hover 이벤트 지정 (박스이동)
      def hoverMoveEvent(self, event):
          if self.hover_flag :
              # print(self.pos_x,self.pos_y)
              # print(self.pos_x + self.pos().x(),self.pos_y + self.pos().y())
              self.box_pos = (self.pos_x + self.pos().x(),self.pos_y + self.pos().y())
              # if (event.scenePos().x()-self.pix > 0 and event.scenePos().y()-self.pix > 0 
              #     and event.scenePos().y()+self.pix < 900 and event.scenePos().x()+self.pix < 1200): 
              if (self.box_pos[0]-self.pix > 0 and self.box_pos[1]-self.pix > 0 
                  and self.box_pos[1]+self.pix < 900 and self.box_pos[0]+self.pix < 1200): 
                  self.bb.setPen(Qt.green)
              else : 
                  self.bb.setPen(Qt.red)
                  
              self.scene().clearSelection()
  
          self.hover_flag = False

  #### GUI 내 실시간 수정 가능 Label 객체 CODE
    class graphicsScene(QtWidgets.QGraphicsScene):
      #라벨 생성시 기본 Default값 지정
      def __init__ (self, i, pix, parent=None):
          super(graphicsScene, self).__init__ (parent)
          self.i = i
          self.x = 0
          self.y = 0
          self.boxes = []
          self.test_num = 0
          self.flag = False
          self.pix = pix
  
      #라벨 마우스 이벤트 지정
      def mousePressEvent(self, event) :
          #박스 생성
          if event.buttons() == Qt.LeftButton and self.flag:
              if pop_num_list == [] :
                  self.x = event.scenePos().x()
                  self.y = event.scenePos().y()
  
                  self.test = Testset(self.i, self.x, self.y, self.pix)
                  self.boxes.append(self.test)
                  self.addItem(self.test)
  
                  self.i += 1
  
          #박스 제거
              elif pop_num_list != []:
                  self.x = event.scenePos().x()
                  self.y = event.scenePos().y()
                  self.test_num = pop_num_list.pop(0)
  
                  self.test = Testset(self.test_num, self.x, self.y, self.pix)
                  self.boxes[self.test_num-1] =  self.test
                  self.addItem(self.test)
  
      #Ctrl 키에 대한 플래그 지정
      def keyPressEvent(self, event):
          if event.key() == Qt.Key_Control :
              self.flag = True
  
      def keyReleaseEvent(self, event):
          self.flag = False
  
   ## 2.개인 연습 및 개인 프로젝트
   ### 1. 서버DB 연동 및 실시간 GUI를 활용한 데이터 확인
    1) 개발 내용
      - KT에서 제공하는 서버내의 DB를 활용하여 Arduino센서의 데이터를 서버로 전송. (pymysql 이용)
      - 추가적으로 실시간 데이터 확인 및 데이터 핸들링을 위하여 GUI를 만듬. 
      (실시간 animation을 이용하기 위해 animation.FuncAnimation이라는 함수 사용)
      - 윈도우 환경이 아닌 CLI환경(LINUX)에서도 실시간 데이터를 활용할수 있도록 채팅소켓을 열어 활용 가능하도록 추가함. (socketserver modulue 활용)
      - 추가적으로 데이터 송수신 및 외부 접속시도들을 파악하기위해 Wireshark라는 app도 이용해봄. (보안파트 지식 부족)
    2) Trouble Shooting
      - 문제 배경
        : 일반적인 PyQt5에서 지원하는 QLabel에는 정적인 그래프만 진행 가능하였으며 실시간으로 데이터를 적용하려고하여도 DeadLock에 걸리는 경우들이 발생함.
      - 해결 방법
        : 데이터를 실시간으로 송수신가능한 Thread를 만들고 또한 animation을 진행가능한 plot을 생성하였으며 그또한 Thread를 활용하여 
        실시간으로 plotting이 가능하도록 진행하였음.

   #### 실시간 Arduino 송수신 Thread CODE
    class WorkerThread(QThread):
    # 데이터를 보내기 위한 시그널 정의
    update_data = pyqtSignal(float, float)

    def __init__(self):
        QThread.__init__(self)
        try :
            self.arduino = serial.Serial('COM5', 9600)
        except:
            pass

    def run(self):
        while True:
            y = self.arduino.readline()
            y = y.decode()[:-2]
            y = re.split(" ", y)
            print(y)

            try:
                tmp = float(y[1])
                hum = float(y[3])
                print(tmp, hum)

                # 시그널 발생
                self.update_data.emit(tmp, hum)

            except:
                pass

    def cleanup(self):
        self.arduino.close()

   #### 실시간 MYSQL 송수신 Thread CODE
    class WorkerThread2(QThread):
      # 데이터를 보내기 위한 시그널 정의
      update_data2 = pyqtSignal(float, float)
  
      def __init__(self):
          QThread.__init__(self)
          self.flag = True
  
      def run(self):
          while self.flag:
              self.con = ps.connect(host = str("IP"),
                                port = PORT,
                                user = str("ID"),
                                password = str("PASSWORD"),
                                db = str("test"),
                                charset = 'utf8')
              
              data_table = pd.read_sql("select * from 0404test", self.con)
              data_col = tuple(data_table.columns)
  
              data_fil = []

              for each_col in data_col:
                  data_fil.append([each_col,data_table[each_col]])
  
              try:
                  tmp2 = float(list(data_fil[1][1])[-1])
                  hum2 = float(list(data_fil[2][1])[-1])
                  print(tmp2, hum2)
  
                  # 시그널 발생
                  self.update_data2.emit(tmp2, hum2)
  
              except:
                  pass
              time.sleep(1)
  
      def stop(self):
          self.flag = False
          self.con.close()
          self.quit()
          print("종료")
          self.wait(3000)
  
   ### 2. ROI 지정 후 물체추적
    1) 개발 내용
      - YOLOv5를 베이스로 하는 Deepsort모델을 활용하여 GUI와 합쳐 실시간 Object Tracking 및 Roi영역 지정 후 Roi내만 추적하는 기능들을 추가
      또한 Object Tracking과 Roi 지정을 실시간으로 작동하여야했기에 각각의 기능에 Thread를 활용함.
    2) Trouble Shooting
      - 문제 배경
        : 딥러닝을 활용하는 GUI이기에 실시간 처리속도가 현저히 느려졌으며 영상의 크기 및 객체수에 따라 기능 차이가 심함.
      - 해결 방법
        : 현재 하드웨어 업그레이드를 제외한 방법을 찾는 중이나 해결하지못함. 다른 모델들을 통해 개선할수 있는지 확인하기 위해 CNN모델 외 
        Transformer 모델 등을 활용하였으나 하드웨어 자원이 부족해 진행중지.

   #### 각 Object의 bbox의 좌표를 추출해 ROI내에 포함되는지 확인
     if len(outputs[i]) > 0:
      for j, (output, conf) in enumerate(zip(outputs[i], confs)):
        bboxes = output[0:4]
        id = output[4]
        cls = output[5]
        is_counterable = True
    
      ##중심좌표 추가 및 오브젝트가 area에 있는지 없는지 판단##
        cx = int(output[0]+((output[2] - output[0])/2))
        cy = int(output[1]+((output[3] - output[1])/2))

        bbox_pos = [cx,cy]
        bbox_pos_G = Point(cx,cy)
        
        global cnt, data

        if poly !=[] and bbox_pos_G.within(poly):
          if id not in data:
            cnt += 1
            data.append(id)

          else :
            if id in data :
              cnt -= 1
              data.remove(id)

        cv2.circle(im0, bbox_pos,5, color=(255,255,0), thickness = -1)
           
   ### 3. 딥러닝 모델따라 구현해보기 및 사용해보기
    1) 개발 내용
      - pytorch와 tensorflow 이용해 CNN을 직접 구현해보고 학습 및 성능 테스트를 진행해봄.
      - 활성화 함수 변경 및 hyper parameter들을 변경해가며 성능 및 결과들을 비교해봄.
      - YOLO 시리즈, VIT시리즈, DINO시리즈, DETR, BEIT 등 들을 활용해봄.
    2) Trouble Shooting
      - 문제 배경
        : 여러가지 모델들을 사용할때 각 모델들에 사용되는 모듈들의 버전들이 충돌하는 경우가 발생.
      - 해결 방법
        : Window WSL2 와 Docker을 활용하여 각각의 모델에 적용시 버전 및 개발환경을 고정시켜 충돌하는 경우를 사전 제거함.

   ### 4. 개인 데이터 송수신앱
    1) 개발 내용
      - 핸드폰 android환경을 통해 촬영하거나 얻은 데이터 또는 rasberrypi를 이용해 얻은 데이터를 실시간으로 상호작용하여 
      데이터 확인 및 연동하는 작업을 수행하는 app을 제작.
      - android환경 구축은 kotlins언어를 활용하였으며 메인서버는 rasberrypi에 두어 핸드폰과는 bluetooth를 활용하여 연결함.
    2) Trouble Shooting
      - 문제 배경
        : bluetooth 연결당시 rasberrypi 가 가지고있는 여러가지 포트들을 이해하지 못하고 bluetooth 연결방식에 대해 제대로 이해하지 
        못해 두 기기간의 통신에 문제를 겪음.
      - 해결 방법
        : rasberrypi에 직접 서버를 가지는 것보다 인터넷 망 내에 서버를 구축하여 그 안에서 서로 상호작용하도록 하면 속도 및 연결 문제를 
        쉽게 해결할 수 있을 것으로 보임.

  #### 주요 Kotlin 블루투스 연결 코드 및 데이터 송신 코드
    // 블루투스 연결파트
    private fun connectBluetooth(device: BluetoothDevice) {
        try {
            bluetoothSocket = device.createRfcommSocketToServiceRecord(SERVICE_UUID)
            bluetoothSocket?.connect()
            Log.d(TAG, "Bluetooth connected. Device is $deviceAddress")
            textView.text = "Bluetooth connected."

            outputStream = bluetoothSocket!!.outputStream
            inputStream = bluetoothSocket!!.inputStream

            // 데이터 수신을 위한 스레드 시작
            Thread {
                val buffer = ByteArray(1024)
                var bytes: Int
                try {
                    while (true) {
                        bytes = inputStream.read(buffer)
                        if (bytes == -1) {
                            Log.d(TAG, "End of stream reached. Closing connection.")
                            break
                        }
                        val incomingMessage = String(buffer, 0, bytes)
                        Log.d(TAG, "InputStream: $incomingMessage")
                        runOnUiThread {
                            // 수신된 데이터를 UI에 표시하거나 필요한 작업 수행
                        }
                    }
                } catch (e: IOException) {
                    Log.e(TAG, "Error reading from InputStream. Closing connection.", e)
                } finally {
                    disconnectBluetooth()
                }
            }.start()

        } catch (e: IOException) {
            Log.e(TAG, "Error connecting to Bluetooth device.", e)
            textView.text = "Error connecting to Bluetooth device."
        }
    }
    
    // 블루투스 메시지 송신파트
    private fun sendBluetoothMessage(message: String) {
        try {
            outputStream.write(message.toByteArray())
            Log.d(TAG, "Message sent: $message")
            editText.text.clear()
        } catch (e: IOException) {
            Log.e(TAG, "Error sending message.", e)
            textView.text = "Error sending message."
        }
    }
    
   ### 5. 엑셀데이터 가시화 및 관리 GUI
    1) 개발 내용
      - 여러 엑셀파일들에 들어가있는 내용들을 각 항목별 및 세부사항을 취합하여 가시성 및 관리성을 높이는 GUI를 개발.
      (Pandas 활용)
    2) Trouble Shooting
      - 문제 배경
        : 직접 엑셀파일을 집어 넣어야해주는 방법이 다소 귀찮고 복잡함.
      - 해결 방법
        : 차후 실질적인 관리 프로그램과 연동하여 서버에 직접 핸들링한다면 가시성과 관리성 둘다 높은 효율을 나타낼 것으로 생각됨.

   ####

   ### 6. 개인 작업 서버 구축
    1) 개발 내용
      - 미사용 노트북을 기반으로 홈서버를 구축하여 실행해봄. LINUX환경으로 Port Fowarding을 활용하여  [모뎀 -> 노트북 -> DOCKER -> 서버] 
      순으로 연결하도록해 간단한 I/O 용도로까지 구축해봄.
      - 추후 그래픽카드 등을 활용하여 Colab과 같은 개발 환경을 구축하는 것을 목표로 하고있음.
    2) Trouble Shooting
      : 보안 및 내구성 등 이 취약하여 조금더 보안공부에 필요성이 중요하게 느껴지며 하드웨어 및 공간 등 자원여건이 부족해 추가적인 구축을 이루지못함.

   ### 7. Google Map API
    1) 개발 내용
      - 핸드폰으로 촬영한 사진과 구글맵 api를 이용하여 사진이 있는 디렉토리를 gui를 통해 지정했을때 해당 사진이 촬영된 위치를 구글맵 위에 표시하도록 기능을 구현함.
    2) Trouble Shooting
      : 작업공유 및 범용성이 떨어져 추후 개인 작업서버를 이용한 app으로 사용하게 된다면 활용도가 높아질 것으로 예상됨.

  #### Google Map API주소 코드
    # Google API Private Key
        api_key = "API KEY"
        
        # 주소 역추적
        address = reverse_geocoding(fin_a,fin_b, api_key)
        
        # Google Map 설명란 추가
        description = f'Image: <a href="{test_img}">{test_img}</a>'
        
        # 각 지점 추가
        pnt = kml.newpoint(name=str(num), coords=[(fin_b, fin_a)])
        pnt.description = description
        
        all_data.append((address, (fin_a,fin_b), img_time, test_img))

   ### 8. RF모델을 활용한 예측
    1) 개발 내용
      - RF모델을 이용한 침강속도 예측을 진행해봄. 신뢰도 0.85 정도가 나오는걸로 결론이됨.
    2) Trouble Shooting
      : 실제 학습데이터량이 현저히 부족하여 충분한 학습데이터량을 가지고 진행한다면 충분히 좋은 신뢰도가 나타날 것으로 예상됨.
