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


SG90 서보모터 사용법


갈색 GND


빨간색 5V


주황색 제어할 핀

아두이노에서 #include<Servo.h> 라는 명령어를 넣어서 아두이노에 내장되어 있는 Servo 라이브러리르 불러오고
Servo myservo;  를 써서 서보 모터를 제어할 객체 만들기.

    `
    void setup() {
         myservo.attach(제어할 핀);
    }
    

    void loop(){
          myservo.write(제어할 각도);
    }
    `

HC-SR04 초음파센서 사용법


HC-SR04 모듈을 동작하기 위해서는 Trig 핀에 10uS 길이의 펄스를 넣어주어야 한다. 펄스가 들어오면 HC-SR04 모듈은 초음파를 발생시키고, 초음파가 돌아오기 까지의 시간을 측정하게 된다.


초음파가 돌아오면 ECHO 핀으로 펄스를 출력하는데 이 펄스의 길이는 초음파가 돌아오는데 걸린 시간에 비레하게 된다. 따라서 이 펄스의 길이를 측정하여 길이를 측정할 수 있다.


-VCC는 5V에 연결


-Trig에 펄스를 인가하기 위해 8번에 연결


-Echo로 출력된 펄스를 입력받기 위해서 9번에 연결


-GND는 GND에 연결


### 하드웨어(모터)

### 소프트웨어(openCV, 카메라)

• 라즈베리파이에 openCV, tensorflow 설치 코드

    
    ~ $ git clone https://github.com/EdjeElectronics/TensorFlow-Lite-Object-Detection-on-Android-and-Raspberry-Pi.git
   


   
   ~ $ cd tflite1~/tflite1 $ sudo pip3 install virtualenv~/tflite1 $ python3 -m venv tflite1-env~/tflite1 $ source tflite1-env/bin/activate(tflite1-env)~/tflite1 $ bash get_pi_requirements.sh
   
   
• 사물 인식 기계학습 모델 설치 사이트
   
   <https://seo-dh-elec.tistory.com/32>
   
• 파이 카메라 연결 코드

`
    (tflite1-env)~/tflite1 $ python3 TFLite_detection_webcam.py --modeldir=TFLite_model
`








