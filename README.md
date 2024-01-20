#include <Adafruit_Fingerprint.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Servo.h>
#include <RTClib.h>
#include <Keypad.h>

#define SERVO_PIN 10

RTC_DS1307 rtc;
Servo myServo;
LiquidCrystal_I2C lcd(0x27, 16, 2);   // Địa chỉ I2C của LCD: 0x27, Kích thước LCD: 16x2
SoftwareSerial mySerial(12, 13);   //Khai báo chân servo: 12=TX  , 13=RX

Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial);

const int ROW_NUM    = 4;    
const int COLUMN_NUM = 4;

char keys[ROW_NUM][COLUMN_NUM] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};

byte pin_rows[ROW_NUM] = {5, 4, 3, 2};    //Kết nối sơ đồ chân của hàng trong bàn phím
byte pin_column[COLUMN_NUM] = {9, 8, 7, 6};    //Kết nối sơ đồ chân của cột trong bàn phím

Keypad keypad = Keypad(makeKeymap(keys), pin_rows, pin_column, ROW_NUM, COLUMN_NUM);
char *correctPassword = "1234";

enum ProgramState {   
  MAIN_PROGRAM,
  OTHER_PROGRAM
};

ProgramState currentProgramState = MAIN_PROGRAM;

void setup() {
        Serial.begin(9600);
  //Khởi tạo RTC
        rtc.begin();
        rtc.isrunning();
        
   //Khởi tạo LCD
        lcd.begin(16, 2);
        lcd.init();
        lcd.backlight();
        lcd.setCursor(0,0);
        lcd.print("Enter Password: ");

        DateTime now = rtc.now();
        lcd.setCursor(10, 1);
        lcd.print(now.hour(), DEC);
        lcd.print(':');
        lcd.print(now.minute(), DEC);

  // Khởi tạo cảm biến vân tay
  finger.begin(57600);

  if (finger.verifyPassword()) {
    Serial.println("Found fingerprint sensor!");   //Đã tìm thấy cảm biến vân tay
  } else {
    Serial.println("Did not find fingerprint sensor :(");   //Không tìm thấy cảm biến vân tay
    while (1) { delay(1); }
  }

  // Mở cổng Serial để điều khiển servo
  myServo.attach(SERVO_PIN);
  myServo.write(0);    //Ban đầu servo ở trạng thái đóng
}

void loop() {
  switch (currentProgramState) {
    case MAIN_PROGRAM:
      runMainProgram();
      break;
    case OTHER_PROGRAM:
      runOtherProgram();
      break;   } }


void runMainProgram() {
     getFingerprintID();
     delay(50);

  static String enteredPassword = "";
  char key = keypad.getKey();
      if (key) {
      if (key == 'B' || key == 'b') {
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("Setup Time...");
        delay(500);
        currentProgramState = OTHER_PROGRAM;  // Chuyển sang chương trình khác
    } else {
        enteredPassword += key;
        lcd.setCursor(0, 1);
        lcd.print(enteredPassword);

      if (enteredPassword.length() == strlen(correctPassword)) {
      if (enteredPassword == correctPassword) {
        lcd.clear();
        lcd.setCursor(5, 0);
        lcd.print("Opened!");
      DateTime now = rtc.now();
        lcd.setCursor(0, 1);
        lcd.print(" Time:");
        lcd.print(now.hour(), DEC);
        lcd.print(':');
        lcd.print(now.minute(), DEC);
        lcd.print(':');
        lcd.print(now.second(), DEC);

        myServo.write(0);  // Đưa servo về góc 0 độ
        delay(500);
        myServo.write(180);  // Đưa servo về góc 180 độ
        delay(5000);
        myServo.write(0);  // Đưa servo về góc 0 độ
        
        lcd.clear();
        lcd.print("Enter Password: ");
        enteredPassword = ""; // Xóa mật khẩu đã nhập
      //DateTime now = rtc.now();
        lcd.setCursor(10, 1);
        lcd.print(now.hour(), DEC);
        lcd.print(':');
        lcd.print(now.minute(), DEC);   } 
        
      else {
        lcd.clear();
        lcd.setCursor(3, 0);
        lcd.print("Incorrect!");  //Không chính xác
      DateTime now = rtc.now();
        lcd.setCursor(0, 1);
        lcd.print(" Time:");
        lcd.print(now.hour(), DEC);
        lcd.print(':');
        lcd.print(now.minute(), DEC);
        lcd.print(':');
        lcd.print(now.second(), DEC);
        delay(5000);
        
        lcd.clear();
        lcd.setCursor(3, 0);
        lcd.print("Try Again!");   //Thử lại
      //DateTime now = rtc.now();
        lcd.setCursor(0, 1);
        lcd.print(" Time:");
        lcd.print(now.hour(), DEC);
        lcd.print(':');
        lcd.print(now.minute(), DEC);
        lcd.print(':');
        lcd.print(now.second(), DEC);
        delay(5000);
        
        lcd.setCursor(1, 0);
        lcd.clear();
        lcd.print("Enter Password: ");
        enteredPassword = ""; // Xóa mật khẩu đã nhập
      //DateTime now = rtc.now();
        lcd.setCursor(10, 1);
        lcd.print(now.hour(), DEC);
        lcd.print(':');
        lcd.print(now.minute(), DEC);  }  }  }  }  }

void runOtherProgram() {
      DateTime now = rtc.now();
        lcd.setCursor(0, 0);
        lcd.print("Time: ");
        lcd.print(now.hour(), DEC);
        lcd.print(':');
        lcd.print(now.minute(), DEC);
        lcd.print(':');
        lcd.print(now.second(), DEC);

    char key = keypad.getKey();
    if (key == '*') {
        lcd.clear();
        lcd.print("Enter new hour:");
        lcd.setCursor(0, 1);
        int newHour = readKeypadNumber();
        lcd.clear();
        lcd.print("Enter new minute:");
        lcd.setCursor(0, 1);
        int newMinute = readKeypadNumber();
        rtc.adjust(DateTime(now.year(), now.month(), now.day(), newHour, newMinute, 0));
        lcd.clear();
        lcd.print("Time updated!");
        delay(2000);
        lcd.clear();

       //char key = keypad.getKey();
  
       //Code chương trình khác ở đây...
}

      if (key == 'C' || key == 'c') {
          currentProgramState = MAIN_PROGRAM;  // Quay lại chương trình chính
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("Enter Password: ");
      DateTime now = rtc.now();
        lcd.setCursor(10, 1);
        lcd.print(now.hour(), DEC);
        lcd.print(':');
        lcd.print(now.minute(), DEC);  } }

    int readKeypadNumber() {
    int number = 0;
    char key = keypad.getKey();
    while (key != '#') {
    if (key >= '0' && key <= '9') {
        number = number * 10 + (key - '0');
        lcd.print(key);
       }
        key = keypad.getKey();
       }
  return number;
}


// Hàm xử lý vân tay và nhận phím ở đây...

uint8_t getFingerprintID() {
  uint8_t p = finger.getImage();   //Lấy hình ảnh ngón tay
  switch (p) {
    case FINGERPRINT_OK:
      Serial.println("Image taken");   //Hình ảnh được chụp
      break;
    case FINGERPRINT_NOFINGER:
      Serial.println("No finger detected");   //Không phát hiện ngón tay
      return p;
    case FINGERPRINT_PACKETRECIEVEERR:
      Serial.println("Communication error");   //Lỗi giao tiếp
      return p;
    case FINGERPRINT_IMAGEFAIL:
      Serial.println("Imaging error");   ////Lỗi hình ảnh
      return p;
    default:
      Serial.println("Unknown error");   //Lỗi không xác định
      return p;   }

    p = finger.image2Tz();   //Chuyển hình ảnh ngón tay sang Template
    switch (p) {
    case FINGERPRINT_OK:
      Serial.println("Image converted");   //Nhận dạng thành công
      break;
    case FINGERPRINT_IMAGEMESS:
      Serial.println("Image too messy");   //Hình ảnh quá lộn xộn
      return p;
    case FINGERPRINT_PACKETRECIEVEERR:
      Serial.println("Communication error");   //Lỗi giao tiếp
      return p;
    case FINGERPRINT_FEATUREFAIL:    //Lỗi tính năng
      Serial.println("Could not find fingerprint features");   //Không có vân tay nào được tìm thấy
      return p;
    case FINGERPRINT_INVALIDIMAGE:   //Lỗi hình ảnh không hợp lệ 
      Serial.println("Could not find fingerprint features");   //Không có vân tay nào được tìm thấy
      return p;
    default:
      Serial.println("Unknown error");   //Lỗi không xác định
      return p;   }

// OK converted!
  p = finger.fingerSearch();   //Lấy hình ảnh ngón tay
  if (p == FINGERPRINT_OK) {
    Serial.println("Found a print match!");    //Tìm thấy một bản in phù hợp
        lcd.clear();
        lcd.setCursor(5,0);
        lcd.print("Opened!");
     DateTime now = rtc.now();
        lcd.setCursor(0, 1);
        lcd.print(" Time:");
        lcd.print(now.hour(), DEC);
        lcd.print(':');
        lcd.print(now.minute(), DEC);
        lcd.print(':');
        lcd.print(now.second(), DEC);

        myServo.write(0);  // Đưa servo về góc 0 độ
        delay(500);
        myServo.write(180);  // Đưa servo về góc 180 độ
        delay(5000);
        myServo.write(0);  // Đưa servo về góc 0 độ
        
        lcd.clear();
        lcd.setCursor(0,0);
        lcd.print("Enter Password: ");
     //DateTime now = rtc.now();
        lcd.setCursor(10, 1);
        lcd.print(now.hour(), DEC);
        lcd.print(':');
        lcd.print(now.minute(), DEC);

  } else if (p == FINGERPRINT_PACKETRECIEVEERR) {    //Bộ nhận gói vân tay
    Serial.println("Communication error");   //Lỗi giao tiếp
    return p;
  } else if (p == FINGERPRINT_NOTFOUND) {
    Serial.println("Did not find a match");   //Không tìm thay vân tay phù hợp
        lcd.clear();
        lcd.setCursor(3,0);
        lcd.print("Incorrect!");   //Không chính xác
      DateTime now = rtc.now();
        lcd.setCursor(0, 1);
        lcd.print(" Time:");
        lcd.print(now.hour(), DEC);
        lcd.print(':');
        lcd.print(now.minute(), DEC);
        lcd.print(':');
        lcd.print(now.second(), DEC);
        delay(5000);
        
        lcd.clear();
        lcd.setCursor(3,0);
        lcd.print("Try Again!");   //Thử lại
     //DateTime now = rtc.now();
        lcd.setCursor(0, 1);
        lcd.print(" Time:");
        lcd.print(now.hour(), DEC);
        lcd.print(':');
        lcd.print(now.minute(), DEC);
        lcd.print(':');
        lcd.print(now.second(), DEC);
        delay(5000);
        
        lcd.clear();
        lcd.setCursor(0,0);
        lcd.print("Enter Password: ");
      //DateTime now = rtc.now();
        lcd.setCursor(10, 1);
        lcd.print(now.hour(), DEC);
        lcd.print(':');
        lcd.print(now.minute(), DEC);
        
    return p;
  } else {
    Serial.println("Unknown error");   //Lỗi không xác định
    return p;
  }

  // found a match!
  Serial.print("Found ID #"); Serial.print(finger.fingerID);
  Serial.print(" with confidence of "); Serial.println(finger.confidence);

  return finger.fingerID;
  
}
  // returns -1 if failed, otherwise returns ID #    //trả về -1 nếu thất bại, nếu không trả về ID #
  int getFingerprintIDez() {
  uint8_t 
     p = finger.getImage();   //Lấy hình ảnh ngón tay
     if (p != FINGERPRINT_OK)  return -1;   

     p = finger.image2Tz();   //Chuyển hình ảnh ngón tay sang Template
     if (p != FINGERPRINT_OK)  return -1;

     p = finger.fingerFastSearch();   //Tìm kiếm nhanh ngón tay
     if (p != FINGERPRINT_OK)  return -1;

  // found a match!
  Serial.print("Found ID #"); Serial.print(finger.fingerID);
  Serial.print(" with confidence of "); Serial.println(finger.confidence);
  return finger.fingerID;
}
