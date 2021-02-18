# Датчик угла поворота AS5600:

 

![&#x41C;&#x430;&#x433;&#x43D;&#x438;&#x442;&#x43D;&#x44B;&#x439; &#x434;&#x430;&#x442;&#x447;&#x438;&#x43A; AS5600](.gitbook/assets/screenshot-from-2021-02-02-11-51-11.png)

[Описание датчика](https://ams.com/documents/20143/36005/AS5600_DS000365_5-00.pdf)

[Библиотека работы с датчиком](https://github.com/kanestoboi/AS5600) 

Покупал [тут](https://aliexpress.ru/item/AS5600-magnetic-encoder-magnetic-induction-angle-measurement-sensor-module-12bit-high-precision/4000551682522.html?spm=a2g0s.9042311.0.0.264d33ed0SdIOG&_ga=2.229159834.721664696.1612051115-530682806.1605135245&_gac=1.220963690.1610796936.CjwKCAiAuoqABhAsEiwAdSkVVCUs5nTX80VVDpcnxLvB3kZpXs2zuaedRuR_P9vpGu__abLf1sHESBoCMw4QAvD_BwE&sku_id=10000002869296963), правдо есть одна особенность, **адрес I2c менять нельзя** на устройстве, и магниты в комплекте намагничены **в осевом направлении**.

## Схема подключения к arduino nano:

 

![](.gitbook/assets/photo5244773462558487057.jpg)

## Код получения данных угла:

```text
#include <AS5600.h>
#include <MedianFilterLib2.h>

static MedianFilter2<long> medianFilter(3);
AS5600 encoder;

long revolutions = 0;   // number of revolutions the encoder has made
double position, position_filtred, last_position_filtred  = 0;    // the calculated value the encoder is at
double output;          // raw value from AS5600
long lastOutput;        // last output from AS5600
double derivative;
bool _init = false;
double max_derivative = 0;

void setup() {
  Serial.begin(115200);

  output = encoder.getPosition();
  lastOutput = output;
  position = output;
}

void loop() {
  output = encoder.getPosition();           // get the raw value of the encoder                      
  
  if ((lastOutput - output) > 2047 )        // check if a full rotation has been made
    revolutions++;
  if ((lastOutput - output) < -2047 )
    revolutions--;

  position = (revolutions * 4096 + output)/4096. * 360;   // calculate the position the the encoder is at based off of the number of revolutions
  position_filtred = medianFilter.AddValue(position);

  if (_init) {
    double derivative = abs(position_filtred  - last_position_filtred);
//    if (derivative > max_derivative) {
      max_derivative = derivative;
//    }
  } else {
    _init = true;
  }

  last_position_filtred = position_filtred;
  Serial.println(position_filtred);

  lastOutput = output;                      // save the last raw value for the next loop 
}

```



