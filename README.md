#include <AFMotor.h>    // Подключаем библиотеку AFMotor  
 
// Создание объектов моторов  
AF_DCMotor motor1(1);  // Мотор 1 на порту 1 (Правый передний)  
AF_DCMotor motor2(2);  // Мотор 2 на порту 2 (Левый передний)  
AF_DCMotor motor3(3);  // Мотор 3 на порту 3 (Левый задний)  
AF_DCMotor motor4(4);  // Мотор 4 на порту 4 (Правый задний)  
 
// Константы  
const int NORMAL_SPEED = 160;   // Обычная скорость моторов (0-160)  
const int left_SPEED = 127;     //Влево
const int MAX_SPEED = 255;      // Максимальная скорость моторов (0-255)  
const long FORWARD_TIME = 3000;  // Время движения на 1 метр (в миллисекундах)  
const long TURN_TIME = 1800;     // Время поворота на 180 градусов (в миллисекундах)  
 
// Пин для ультразвукового датчика  
const int TRIG_PIN = A4;  // Пин для отправки сигнала  
const int ECHO_PIN = A5;  // Пин для получения отраженного сигнала  
const int OBSTACLE_DISTANCE = 25;  // Расстояние до препятствия в сантиметрах, при котором робот остановится  
 
// Функция для измерения расстояния  
int getDistance() {  
  // Отправляем импульс на TRIG_PIN  
  digitalWrite(TRIG_PIN, LOW);  
  delayMicroseconds(2);  
  digitalWrite(TRIG_PIN, HIGH);  
  delayMicroseconds(10);  
  digitalWrite(TRIG_PIN, LOW);  
 
  // Измеряем время отражения сигнала  
  long duration = pulseIn(ECHO_PIN, HIGH);  
 
  // Преобразуем время в расстояние (в сантиметрах)  
  int distance = duration * 0.034 / 2;  
  return distance;  
}  
 
// Функция для движения вперед (с проверкой препятствий)  
void goForward(long time) {  
  unsigned long startTime = millis(); // Запоминаем время начала движения  
 
  while (millis() - startTime < time) {  
    // Измеряем расстояние до препятствия  
    int distance = getDistance();  
    Serial.print("Distance: ");  
    Serial.println(distance);  
 
    // Если расстояние меньше порога, останавливаем робота и ждем, пока препятствие не исчезнет 
    if (distance < OBSTACLE_DISTANCE) {  
      Serial.println("Obstacle detected! Stopping...");  
      stopMotors();  
      while (getDistance() < OBSTACLE_DISTANCE) { 
        delay(100); // Ждем, пока препятствие не исчезнет 
      } 
      Serial.println("Obstacle cleared! Resuming..."); 
      startTime = millis(); // Сбрасываем время начала движения 
    }  
 
    // Двигаемся вперед  
    motor1.setSpeed(NORMAL_SPEED);  
    motor2.setSpeed(left_SPEED);  
    motor3.setSpeed(left_SPEED);  
    motor4.setSpeed(NORMAL_SPEED);  
    motor1.run(FORWARD);  
    motor2.run(FORWARD);     
    motor3.run(FORWARD);  
    motor4.run(FORWARD);  
 
    // Небольшая задержка для стабильности работы датчика  
    delay(50);  
  }  
 
  // Останавливаем моторы после завершения движения  
  stopMotors();  
}  
 
// Функция для танкового разворота (поворот на месте)  
void turnAround(long time) {  
  // Правая сторона (моторы 1 и 4) вращается назад с максимальной скоростью  
  motor1.setSpeed(MAX_SPEED);  
  motor4.setSpeed(MAX_SPEED);  
  motor1.run(BACKWARD);  
  motor4.run(BACKWARD);  
 
  // Левая сторона (моторы 2 и 3) вращается вперед с максимальной скоростью  
  motor2.setSpeed(MAX_SPEED);  
  motor3.setSpeed(MAX_SPEED);  
  motor2.run(FORWARD);  
  motor3.run(FORWARD);  
 
  delay(time);  
  stopMotors(); // Останавливаем моторы после поворота  
}  
 
// Функция для остановки  
void stopMotors() {  
  motor1.run(RELEASE);  
  motor2.run(RELEASE);  
  motor3.run(RELEASE);  
  motor4.run(RELEASE);  
}  
 
void setup() {  
  Serial.begin(9600);  
  pinMode(TRIG_PIN, OUTPUT);  
  pinMode(ECHO_PIN, INPUT);  
}  
 
void loop() {  
  // Движение вперед на 1 метр  
  goForward(FORWARD_TIME);  
 
  // Поворот на 180 градусов  
  turnAround(TURN_TIME);  
 
  // Движение вперед на 1 метр (обратно)  
  goForward(FORWARD_TIME);  
 
  // Поворот на 180 градусов (для следующего цикла)  
  turnAround(TURN_TIME);  
}
