//----------------- Cell Cycler -----------------
// Version: 2.2
// Author: Chase Zielinski ---- zchase@pdx.edu
// License: GPL-3.0
// Tested on Arduino Uno R3
//
//
//
// Verify test variables before use.

//---------------- Test Variables ----------------
const int n_or_f = 1;                     // number of cycles, enter 0 for test until fail
const float voltUpperLim =      4.2;      // charge to this vaoltage
const float voltLowerLim =      3.2;      // discharge to this voltage
const int waitAfterCharge =     300;      // seconds of wait time after charge cycle
const int waitAfterDischarge =  300;      // seconds of wait time after discharge cycle
const int thermalUpperLim =     80;       // test fail limit in deg C
const int thermalLowerLim =     25;       // set additional wait criterea in deg C, set to 9999 for no thermal wait
const float captureRate =       0.2;      // data collection rate in Hz
const int failMode =            0;        // 0 = hard stop on fail, 1 = cycle reset on fail
const float resistorVal =       1;        // enter resistor value in Ohms for energy calculation

//------ Calculate data read delay based on desired collection frequency
const uint32_t timeDelay =  1000 / captureRate; // ms

//------ set pins ------
const int thermistorV1 = 1;               // set pin number for thermistor V+
const int thermistorG1 = 3;               // set pin number for thermistor ground
const int button1Pin = 5;                 // set pin number for start button read pin
const int relayChargeGND = 11;
const int relayChargeVPP = 12;
const int relayCharge = 13;                // set pin number for relay in line with cell charger
const int relayDischarge = 7;             // set pin number for relay in line with discharge circuit
const int relayDischargeGND = 8;
const int relayDischargeVPP = 9;

//------  initialize loop variables ------
int     nLoops =            1;            //  loop tracking variable
uint32_t tStart =           millis();     //
uint32_t tEnd =             millis();     //
uint32_t tDelta =           0;            //
int state =                 0;            //  state variable for wait-running-end finite state machine
int cycleState =            0;            //  state variable for charge-wait-discharge-wait finite state machine
uint32_t cycleTimer =       0;            //  timer for wait cycles
uint32_t loopTime =         0;            //  timer for data collection

//------ set read variables ------
int voltageRead = 0;
double voltage = 0.0;
uint32_t voltageTimer = 10;
int voltageSum = 0;
int nVoltageRead = 0;
double therm = 0;

void setup() {
  Serial.begin(9600);                                                 //  initialize serial output

  //------  set arduino pinmodes and starting state
  pinMode(thermistorV1, OUTPUT); digitalWrite(thermistorV1, HIGH);    //  thermistor V++, set to HIGH = 5V
  pinMode(thermistorG1, OUTPUT); digitalWrite(thermistorG1, LOW);     //  thermistor V--, set to LOW = ground
  pinMode(relayChargeGND, OUTPUT); digitalWrite(relayChargeGND, LOW);
  pinMode(relayChargeVPP, OUTPUT); digitalWrite(relayChargeVPP, HIGH);
  pinMode(relayDischargeGND, OUTPUT); digitalWrite(relayDischargeGND, LOW);
  pinMode(relayDischargeVPP, OUTPUT); digitalWrite(relayDischargeVPP, HIGH);
  pinMode(button1Pin, INPUT); digitalWrite(button1Pin, HIGH);         //  start button pin input, set to HIGH = internal pull-up resistor
  pinMode(A0, INPUT);                                                 //  initialize analog read
  pinMode(A1, INPUT);                                                 //  initialize analog read
  pinMode(relayCharge, OUTPUT); digitalWrite(relayCharge, LOW);       //  initialize charge circuit relay, set to LOW = open circuit
  pinMode(relayDischarge, OUTPUT); digitalWrite(relayDischarge, LOW); //  initialize discharge circuit relay, set to LOW = open circuit
  analogReference(INTERNAL);                                          //  Switch to Internal 1.1V Reference

}

void loop() {
  //------ start loop timer ------
  tStart = millis();                              //  should be at the top to capture the entire loop time

  //------ wait for start button ------           //  starting state, after setup waits for button press
  if (state == 0) {
    if (digitalRead(button1Pin) == LOW) {
      delay(50);                                  //  de-bounce the button read
      if (digitalRead(button1Pin) == LOW) {
        state = 1;                                //  advance to running state
        digitalWrite(relayCharge, HIGH);          //  turn on charge relay
      }
    }
  }

  if (state == 1)
  {
    //------- read variables
    therm = analogRead(A0);                       //  thermistor read
    therm = therm * 1100 / (1024 * 10);           //  using manufacturer formula
    voltageRead = analogRead(A1);                 //  voltage read
    voltage = voltageRead * (5.0 / 1023.0);       //  using arduino voltage read example

    if (loopTime > timeDelay) {                   //  print data at desired rate
      loopTime = 0;                               //  reset loopTime
      Serial.print("DATA,TIME,");                 //  export data, start line
      Serial.print(nLoops);                       //  prints currentcharge-discharge cycle
      Serial.print(",");                          //  comma diliniation
      Serial.print(therm);
      Serial.print(",");                          //  comma diliniation
      Serial.print(voltage);

    }

    //-- limit test                               //  test battery is within thermal limit, force
   // if (therm > thermalUpperLim) {
   //   state = 2;
   //   digitalWrite(relayCharge, LOW);             //  open both relays on test exit
   //   digitalWrite(relayDischarge, LOW);
   // }

    //-- charge                                   //  charge sub-cycle
    if (cycleState == 0)                          
    { if (voltage > voltUpperLim) {               //  check for charge limit
        cycleState = 1;                           //  progress cycle if charge is complete
        digitalWrite(relayCharge, LOW);           //  open charge relay
      }

    }

    //-- wait after charge
    if (cycleState == 1)
    {
      if (thermalLowerLim == 9999) {              //  check for wait settings
        if (cycleTimer > waitAfterCharge * 1000) {  
          cycleState = 2;                         //  progress cycle state after enough time has lapsed
        }
      }
      else {
        if (cycleTimer > waitAfterCharge * 1000 && therm < thermalLowerLim) {
          cycleState = 2;                         //  progress cycle state after enough time has lapsed and battery is below specified lower limit
        }
      }
      cycleTimer = cycleTimer + tDelta;           //  increment elasped time

      if (cycleState == 2) {                      //  after state has been progressed, close discharge relay and reset timer
        cycleTimer = 0; digitalWrite(relayDischarge, HIGH);
      }
    }

    //-- discharge                                //  discharge sub-cycle
    if (cycleState == 2)
    { if (voltage < voltLowerLim) {               //  check for discharge battery lower voltage limit
        cycleState = 3;                           //  progress cycle when discharge is done
        digitalWrite(relayDischarge, LOW);        //  open discharge relay
      }

    }

    //-- wait after discharge
    if (cycleState == 3)
    {
      if (thermalLowerLim == 9999) {              //  check for wait settings
        if (cycleTimer > waitAfterDischarge * 1000) {
          cycleState = 0;                         //  press cycle state after enough time has lapsed
        }
      }
      else {
        if (cycleTimer > waitAfterCharge * 1000 && therm < thermalLowerLim) {
          cycleState = 0;                         //  progress cycle state after enough time has lapsed and battery is below specified lower limit
        }
      }

      cycleTimer = cycleTimer + tDelta;           //  increment elasped time

      if (cycleState == 0) {
        nLoops++;                                 //  increment number of elapsed discharge cycles
        cycleTimer = 0;                           //  reset cycle timer
        digitalWrite(relayCharge, HIGH);          //  close charge circuit
        if(nLoops > n_or_f && n_or_f > 0){        //  check for discharge cycles limit or no limit, quit if done
          state = 2; digitalWrite(relayCharge, LOW);}   
      }
    }

  }
  //-- hard stop state                            //  empty state for fail or test complete, reset arduino to leave
  if (state == 2) {
  }


  //------ update timer ------                    //  timer update, need to be at the end to capture full loop time
  if (state == 1) {
    tEnd = millis();
    tDelta = tEnd - tStart;
    loopTime = loopTime + tDelta;
  }
}
