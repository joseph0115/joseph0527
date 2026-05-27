import cv2
import serial
import time

# 1. 設定與 Arduino 的連線
try:
    # 'COM3' 請改成你 Arduino 的連接埠編號SS
    ser = serial.Serial('COM3', 9600, timeout=1)
    time.sleep(2)  # 等待連線穩定
    print("成功連結 Arduino！")
except Exception as e:
    print(f"無法連結 Arduino: {e}")
    ser = None

# 2. 開啟 USB 鏡頭
cap = cv2.VideoCapture(1)  # 0通常是筆電鏡頭，1是外接USB鏡頭

if not cap.isOpened():
    print("找不到鏡頭，請檢查 index (0 或 1)")
    exit()

print("程式啟動：按下 'g' 亮綠燈，按下 'r' 亮紅燈，按下 'q' 離開")

while True:
    ret, frame = cap.read()
    if not ret:
        break

    # 在視窗畫面上顯示一些資訊
    cv2.putText(frame, "Status: System Ready", (10, 30),
                cv2.FONT_HERSHEY_SIMPLEX, 0.7, (255, 255, 255), 2)

    cv2.imshow('Parking System Monitor', frame)

    # 讀取鍵盤輸入
    key = cv2.waitKey(1) & 0xFF

    if ser:
        if key == ord('g'):  # 按下 g 傳送 'G'
            ser.write(b'G')
            print("Python 傳送：亮綠燈 (G)")
        elif key == ord('r'):  # 按下 r 傳送 'R'
            ser.write(b'R')
            print("Python 傳送：亮紅燈 (R)")

    # 按下 'q' 鍵退出程式
    if key == ord('q'):
        break

# 釋放資源
cap.release()
cv2.destroyAllWindows()
if ser:
    ser.close()
`Arduino 程式碼`
const int redLed = 4;
const int greenLed = 3;

void setup() {
  Serial.begin(9600);
  pinMode(redLed, OUTPUT);
  pinMode(greenLed, OUTPUT);
  // 初始狀態亮紅燈
  digitalWrite(redLed, HIGH);
  digitalWrite(greenLed, LOW);
}

void loop() {
  if (Serial.available() > 0) {
    char cmd = Serial.read();
    
    if (cmd == 'G') {
      digitalWrite(greenLed, HIGH);
      digitalWrite(redLed, LOW);
    } 
    else if (cmd == 'R') {
      digitalWrite(greenLed, LOW);
      digitalWrite(redLed, HIGH);
    }
  }
}
