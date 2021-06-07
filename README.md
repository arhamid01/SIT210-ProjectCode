# SIT210-ProjectCode
# There are two pieces of code below for this project

# This is the dispenser code for the sensing component run by the Raspberry Pi
// This #include statement was automatically added by the Particle IDE.
#include <HC_SR04.h>

// Define pins for HC-SR04, led and buzzer
#define trigPin     D10
#define echoPin     D11
#define LED         D7
#define buzzer      D9     

// cm variable which will be modified 
double cm = 0.0;

// rangefinder variable of type HC_SR04 to use methods of the class with the sensor
HC_SR04 rangefinder = HC_SR04(trigPin, echoPin);

// Initialise all pins being used as input or output and set LED as low
// Create particle variable to keep track in the console that values are being collected
void setup() {
    pinMode(LED,OUTPUT);
    pinMode(trigPin, OUTPUT);
    pinMode(echoPin, INPUT);
    pinMode(buzzer, OUTPUT);
    digitalWrite(LED,LOW);
    Particle.variable("dist",cm);
}

void loop() {
    // Assign the distance being found and used by the sensor to cm variable
    cm = rangefinder.getDistanceCM();
    
    // If distance of hand is below 7cm blink LED, beep buzzer and publish a message to dispenser event
    if (cm <7)
    {
        digitalWrite(LED,HIGH);
        delay(1000);
        digitalWrite(LED,LOW);
        beepBuzzer();
        Particle.publish("dispenser","rightD");
    }
    // If distance of hand is greater than 10cm and below 15cm blink LED, beep buzzer and publish a message to dispenser event
    if (cm >=10 && cm<15)
    {
        digitalWrite(LED,HIGH);
        delay(1000);
        digitalWrite(LED,LOW);
        beepBuzzer();
        Particle.publish("dispenser","leftD");
    }

    delay(1000);
}

// This method activates a buzzer for 0.5 secs then turns it off
void beepBuzzer()
{
    digitalWrite(buzzer,HIGH);
    delay(500);
    digitalWrite(buzzer,LOW);
}

# This code is for the control mechanism run by the Particle Argon
// This #include statement was automatically added by the Particle IDE.
#include <LiquidCrystal_I2C_Spark.h>

// Create lcd variable of the type of library to use its functions
LiquidCrystal_I2C *lcd;

// Two servos are being used and their names are as follows
Servo trayServo;
Servo flapServo;

// Create variables for the servo pins
int trayMotor = A0;
int flapMotor = A1;
    

void setup() {
    // Assign buttonTray function to Particle Cloud
    Particle.function("button",buttonTray);
    // Subscibe to the dispenser event being published by the Pi
    Particle.subscribe("dispenser",myHandler,MY_DEVICES);
    // Initialise position of flap
    activateFlap(90);
    // Initialise position of tray
    activateTray(0);
    // Initialise 16x2 lcd with its i2c address and turn on backlight then clear screen
    lcd = new LiquidCrystal_I2C(0x27, 16, 2);
    lcd->init();
    lcd->backlight();
    lcd->clear();

}

void myHandler(const char *event, const char *data)
{
    // Move the flap to open the right container then drop item in tray and open tray 
    if (strcmp (data,"rightD") == 0)
    {
        activateFlap(180);
        // When flap opened, print message "Item Dispensing"
        lcd->clear();
        lcd->setCursor(0,1);
        lcd->print("Item Dispensing");
        delay(1000);
        activateFlap(90);
        delay(1000);
        
        activateTray(180);
        // When tray opened, print message "Item Ready!"
        lcd->clear();
        lcd->setCursor(0,1);
        lcd->print("Item Ready!");
        
        delay(10000);
        activateTray(0);
        
        lcd->clear();
        
    }
    
    // move the flap to open the left container then drop item in tray and open tray 
    if (strcmp (data,"leftD") == 0)
    {
        activateFlap(0);
        // When flap opened, print message "Item Dispensing"
        lcd->clear();
        lcd->setCursor(0,1);
        lcd->print("Item Dispensing");
        delay(1000);
        activateFlap(90);
        delay(1000);
        
        
        activateTray(180);
        // When tray opened, print message "Item Ready!"
        lcd->clear();
        lcd->setCursor(0,1);
        lcd->print("Item Ready!");
        
        delay(10000);
        activateTray(0);
        
        lcd->clear();
    }
}

void loop() {
    //clear lcd and set writing position to top of screen
    lcd->clear();
    lcd->setCursor(0,0);
    //print the time with the hour, minute and seconds
    lcd->print(Time.hour() < 10? "   0" : "    ");
    lcd->print(Time.hour());
    lcd->print(Time.minute() < 10? ":0": ":");
    lcd->print(Time.minute());
    lcd->print(Time.second()< 10? ":0": ":");
    lcd->print(Time.second());

    delay(1000);
}

// Upon a button being pressed this function will be called through the cloud and open the right container then the tray
int buttonTray(String command)
{
    if (command == "open")
    {
        activateFlap(180);
        // When flap opened, print message "Item Dispensing"
        lcd->clear();
        lcd->setCursor(0,1);
        lcd->print("Item Dispensing");
        delay(1000);
        activateFlap(90);
        delay(1000);
        
        activateTray(180);
        // When tray opened, print message "Item Ready!"
        lcd->clear();
        lcd->setCursor(0,1);
        lcd->print("Item Ready!");
        
        delay(10000);
        activateTray(0);
        
        lcd->clear();
        return 1;
    }
    else return -1;
    
}

// This function activates the servo for the tray and rotates it by the angle given then deactivates the servo motor
void activateTray(int angle) {
    trayServo.attach(trayMotor);
    trayServo.write(angle);
    delay(500);
    trayServo.detach();
}

// This function activates the servo for the flap on containers and rotates it by the angle given then deactivates the servo motor
void activateFlap(int angle) {
    flapServo.attach(flapMotor);
    flapServo.write(angle);
    delay(500);
    flapServo.detach();
}
