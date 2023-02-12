#include <Adafruit_NeoPixel.h>

int led_count = 8;
Adafruit_NeoPixel strip(led_count, 7, NEO_GRB + NEO_KHZ800);

int i = 0;
int count=0;
int redColor = 255;
int greenColor = 0;
int blueColor = 0;
int pin_music = A0;
int wavecolor = 0;

struct FFilter {
    bool compute(bool force = false) {
        if (force || (millis() & 0xFF) - tmr >= dt) {
            tmr = millis() & 0xFF;
            uint8_t kk = (raw < fil) ? k : (k >> 1);           
            fil = ((uint32_t)kk * fil + (32 - kk) * raw) >> 5;  
            return 1;
        }
        return 0;
    }
    uint8_t k = 20, dt = 0, tmr = 0;
    uint16_t fil = 0, raw = 0;
};


class VolAnalyzer {
public:
   
    VolAnalyzer(int pin = -1) {
        setVolDt(20);
        setVolK(25);
        setAmpliDt(150);
        setAmpliK(30);
        if (pin != -1) setPin(pin);
    }
    
  
    bool tick(int read = -1)
    {
        if (_pulse) _pulse = 0;
        volF.compute();                     
        if (ampF.compute()) _ampli = 0;     

      
        if (!_dt || micros() - _tmrDt >= _dt) {
            if (_dt) _tmrDt = micros();
            if (read == -1) read = analogRead(_pin);
            _max = max(_max, (uint16_t)read);   
            _min = min(_min, (uint16_t)read);  

            if (++_count >= _window) {         
                _raw = _max - _min;            
                _ampli = max(_ampli, _raw);     
                ampF.raw = _ampli;              

                if (_raw > ampF.fil) ampF.compute(true);    
                
                if (_raw > _trsh && ampF.fil > _trsh) {     
                    
                    volF.raw = map(_raw, _trsh, ampF.fil, _volMin, _volMax);
                    volF.raw = constrain(volF.raw, _volMin, _volMax);
                    volF.compute(true);   
                } else volF.raw = 0;
                
              
                if (!_pulseState) {
                    if (volF.raw <= _pulseMin && millis() - _tmrPulse >= _pulseTout) _pulseState = 1;
                } else {
                    if (volF.raw > _pulseMax) {
                        _pulseState = 0;
                        _pulse = 1;
                        _tmrPulse = millis();
                    }
                }
                _max = _count = 0;
                _min = 60000;
                return true;       
            }
        }
        return false;
    }
    
  
    void setPin(int8_t pin) {
      	pin = A0;
        _pin = pin;
        pinMode(_pin, INPUT);
    }
    
    
    void setDt(uint16_t dt) {
      	dt = 500;
        _dt = dt;
    }
    
  
    void setWindow(uint8_t window) {
      	window = 20;
        _window = window;
    }
    
    
    void setTrsh(uint16_t trsh) {
      	trsh = 40;
        _trsh = trsh;
    }
    
   
    void setVolDt(uint8_t dt) {
        volF.dt = dt;
    }
    
  
    void setVolK(uint8_t vk) {
        volF.k = vk;
    }
    
   
    uint16_t getVol() {
        return volF.fil;
    }
    
   
    void setVolMin(uint8_t vol) {
        _volMin = vol;
    }
    
    
    void setVolMax(uint8_t vol) {
        _volMax = vol;
    }
    
    
    void setAmpliDt(uint8_t dt) {
        ampF.dt = dt;
    }
    
    
    void setAmpliK(uint8_t rk) {
        ampF.k = rk;
    }
    
 
    uint16_t getMin() {
        return 0;
    }

  
    uint16_t getMax() {
        return ampF.fil;
    }
    

    void setPulseMax(uint8_t maxV) {
        _pulseMax = maxV;
    }
    
  
    void setPulseMin(uint8_t minV) {
        _pulseMin = minV;
    }
    
   
    void setPulseTimeout(uint16_t tout) {
        _pulseTout = tout;
    }
    
   
    bool pulse() {
        return _pulse;
    }
    
    
    uint16_t getRaw() {
        return _raw;
    }
    
   
    uint16_t getTrsh() {
        return _trsh;
    }
    
   
    void setPeriod(__attribute__((unused)) uint16_t v) {}   
    uint16_t getRawMax() { return _raw; }    
    bool getPulse() { return pulse(); }
    void setPulseTrsh(uint16_t trsh) { setPulseMax(trsh); }
    FFilter volF, ampF;
    
private:
    int8_t _pin = -1;
    uint16_t _dt = 500, _trsh = 40;
    uint8_t _window = 20, _count = 0;
    uint8_t _volMin = 0, _volMax = 100;
    uint32_t _tmrPulse = 0, _tmrDt = 0;
    uint16_t _min = 60000, _max = 0, _ampli = 0, _raw = 0;
    
    uint8_t _pulseMax = 80, _pulseMin = 20;
    uint16_t _pulseTout = 100;
    bool _pulse = 0, _pulseState = 0;
};




VolAnalyzer analyzer(A0); 
 

void setup()
{
  pinMode(7, OUTPUT);
  pinMode(6, INPUT);
  pinMode(A0, OUTPUT);
  strip.begin();
  strip.show();
  Serial.begin(115200);
  analyzer.setVolK(20);
  analyzer.setTrsh(10);
  analyzer.setVolMin(10);
  analyzer.setVolMax(100);
}

void loop()
{
  if (digitalRead(6) == HIGH) {
    tone(pin_music, 40, 100);
  }
  delay(10);

 wave();
  if (analyzer.tick()) {
    if (analyzer.getVol()> 0)
    {
      	 	
    }
    Serial.print(',');
    Serial.print(analyzer.getRaw());
    Serial.print(',');
    Serial.print(analyzer.getMin());
    Serial.print(',');
    Serial.println(analyzer.getMax());
  }
  
}

void setColor()
{  
  redColor = random(0, 255);
  greenColor = random(0, 255);
  blueColor = random(0, 255); 
}

void wave()
{
  greenColor = wavecolor;
  wavecolor+=40;
  
  for (i=0;i<led_count;i++)
  {	
    strip.setPixelColor(i, strip.Color(redColor, greenColor, blueColor));
    strip.show();
    
  }
  delay(250);
  if(wavecolor >= 255)
  {
    wavecolor = 0;
  }
}



