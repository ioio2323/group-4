# MDP2-4

Incheon Electronic Meister High School MDP Project

## 💻작품 소개

자동으로 침수 방지를 해주는 지하차도 바리게이트

## 🕑개발 기간

2023년 8월 4일 ~

## 👩‍👦‍👦구성원

길수빈 : 하드웨어, 센서 개발 담당

서지혜 : 소프트웨어, openCV, 카메라 개발 담당

정민웅 : 하드웨어, 모터 개발 담당

## 📚자료 조사

### 하드웨어(센서)    

#### <아두이노>
HC-SR04 초음파센서 사용법


HC-SR04 모듈을 동작하기 위해서는 Trig 핀에 10uS 길이의 펄스를 넣어주어야 한다. 펄스가 들어오면 HC-SR04 모듈은 초음파를 발생시키고, 초음파가 돌아오기 까지의 시간을 측정하게 된다.


초음파가 돌아오면 ECHO 핀으로 펄스를 출력하는데 이 펄스의 길이는 초음파가 돌아오는데 걸린 시간에 비레하게 된다. 따라서 이 펄스의 길이를 측정하여 길이를 측정할 수 있다.


-VCC는 5V에 연결


-Trig에 펄스를 인가하기 위해 8번에 연결


-Echo로 출력된 펄스를 입력받기 위해서 9번에 연결


-GND는 GND에 연결

### <뉴클레오>

#### 배선

1.VCC는 5V에 연결


2.Trig에는 펄스를 인가하기 위해 PA5에 GPIO Output으로 설정	


3.Echo로 출력된 펄스를 입력받기 위해 타이머3의 채널1 input capture를 사용하기위해PA6으로 설정


4.GND는 GND에 연결

#### Cube MX

1.TIM3은 APB1 클락소스(90MHz)를 공급받는다. 그리고 Prescaler 값을 90-1 로 해주면 1us(마이크로초) 마다 CNT가 오른다. 그리고 TIM3은 16비트 카운터라서 ARR은 최대값인 0xffff-1로 해준다. UART3 설정도 켠다
  

### 하드웨어(모터)

#### <아두이노>
SG90 서보모터 사용법


갈색 GND


빨간색 5V


주황색 제어할 핀

아두이노에서 #include<Servo.h> 라는 명령어를 넣어서 아두이노에 내장되어 있는 Servo 라이브러리르 불러오고
Servo myservo;  를 써서 서보 모터를 제어할 객체 만들기.

    
    void setup() {
         myservo.attach(제어할 핀);
    }
    

    void loop(){
          myservo.write(제어할 각도);
    }
    

### 소프트웨어(openCV, 카메라)

• 라즈베리파이에 openCV, tensorflow 설치 코드

• 라즈베리파이와 누클레오 보드 시리얼 통신 연결

#### 라즈베리파이 코드

    import cv2
    import tensorflow as tf
    import RPi.GPIO as GPIO
    import numpy as np
    import serial
    
    ser = serial.Serial('/dev/serial0', 115200)
    ser.close()
    ser.open()
    
    model_path = '/home/pi/MDP2-4/keras_model.h5'
    model = tf.keras.models.load_model(model_path, compile=False)
    
    cap = cv2.VideoCapture(0)
    
    # 원하는 해상도 설정
    desired_width = 640  # 원하는 가로 해상도
    desired_height = 480  # 원하는 세로 해상도
    
    # 카메라 속성 설정
    cap.set(cv2.CAP_PROP_FRAME_WIDTH, desired_width)
    cap.set(cv2.CAP_PROP_FRAME_HEIGHT, desired_height)
    
    frame_count = 0
    frame_interval = 10  # 5프레임 간격으로 처리
    
    while True:
        ret, frame = cap.read()
        frame_count += 1
    
        if frame_count % frame_interval == 0:
            if ret:
                # 이미지 크기 조정
                resized_frame = cv2.resize(frame, (224, 224))
    
                # 모델 추론을 위한 전처리
                input_data = np.expand_dims(resized_frame, axis=0)
    
                # 모델 추론 수행
                prediction = model.predict(input_data)[0]
                print(prediction)
    
                flipped_frame = cv2.flip(frame, 0)  # 원본 프레임 뒤집기
    
                if prediction[0] > 0.7:
                    cv2.putText(flipped_frame, 'True', (50, 50), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0), 2)
                    data_to_send = "a"
                    ser.write(data_to_send.encode())
                    print("True")
    
                if prediction[1] > 0.7:
                    cv2.putText(flipped_frame, 'False', (50, 50), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0), 2)
                    data_to_send = "b"
                    ser.write(data_to_send.encode())
                    print("False")
    
                # 변환된 프레임을 화면에 표시
                cv2.imshow('Teachable Machine Model', flipped_frame)
    
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break
    
    cap.release()
    cv2.destroyAllWindows()

#### STM32 코드

##### 센서값과 라즈베리파이 신호를 받아서 모터를 돌리는 코드

    /* USER CODE BEGIN 0 */
    char rx_data[1] = "b";
    void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
    {
      if(huart == &huart1){
        if(!strncmp("a", rx_data, 1)){
           HAL_UART_Transmit(&huart3, (uint8_t *)"0\n", sizeof("0\n")-1, 10);
            htim4.Instance->CCR1 = 1500;
          }
       }
       HAL_UART_Receive_IT(&huart1, (uint8_t *)rx_data, 2);
    }
    /* USER CODE END 0 */
    
      /* USER CODE BEGIN 2 */
      HAL_TIM_IC_Start_IT(&htim3, TIM_CHANNEL_1);
      HAL_TIM_IC_Start_IT(&htim4, TIM_CHANNEL_1);
      HAL_UART_Receive_IT(&huart1, (uint8_t*)rx_data, 2);
    
      char out1[20];
      char out2[20];
      /* USER CODE END 2 */
    
      /* USER CODE BEGIN WHILE */
      while (1)
      {
          HCSR04_Read();
          if(!strncmp("b", rx_data, 1)){
               if (Distance <= 6){
                  HAL_UART_Transmit(&huart1, (uint8_t *)out1, strlen(out1), 10);
                  HAL_UART_Transmit(&huart3, (uint8_t *)"1\n", sizeof("1\n")-1, 10);
                  htim4.Instance->CCR1 = 2500;
               }
               else{
                  HAL_UART_Transmit(&huart1, (uint8_t *)out2, strlen(out2), 10);
                  HAL_UART_Transmit(&huart3, (uint8_t *)"0\n", sizeof("0\n")-1, 10);
                  htim4.Instance->CCR1 = 1500;
               }
         }
                  HAL_Delay(1000);
        /* USER CODE END WHILE */


##### 보드 통신으로 신호를 받아서 모터를 돌리는 코드

    /* USER CODE BEGIN 0 */
    char rx_data[2];
    /* USER CODE END 0 */
    
      /* USER CODE BEGIN 2 */
      HAL_TIM_IC_Start_IT(&htim3, TIM_CHANNEL_1);
      HAL_TIM_IC_Start_IT(&htim4, TIM_CHANNEL_1);
    
      //HAL_UART_Receive_IT(&huart3, (uint8_t *)rx_data, 1);
      HAL_TIM_PWM_Start(&htim4, TIM_CHANNEL_1);
      /* USER CODE END 2 */
    
      /* USER CODE BEGIN WHILE */
      while (1)
      {
         HAL_UART_Receive(&huart3, (uint8_t *)rx_data, sizeof(rx_data)-1, 10);
         if(!strncmp("1", rx_data, 1)) {
            HAL_UART_Transmit(&huart2, (uint8_t *)"1\n", sizeof("1\n"), 10);
            htim4.Instance->CCR1 = 500;
         } else if(!strncmp("0", rx_data, 1)) {
            HAL_UART_Transmit(&huart2, (uint8_t *)"0\n", sizeof("0\n"), 10);
            htim4.Instance->CCR1 = 1500;
         }
         HAL_Delay(1000);
    
        /* USER CODE END WHILE */
















