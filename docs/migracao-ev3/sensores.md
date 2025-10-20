# Migração dos Sensores

## Diferenças Principais

Assim como os motores, os sensores no NXT são acessados diretamente através de arrays pré-definidos, como `SensorValue[]`, onde cada índice corresponde a uma porta específica do sensor. Por exemplo, `SensorValue[S1]` acessa o sensor conectado à porta 1.

No EV3, os sensores são representados como objetos de classes específicas, como `ev3dev::sensor::color_sensor` ou `ev3dev::sensor::ultrasonic_sensor`. Cada sensor é instanciado com base na porta à qual está conectado, utilizando a nomenclatura do sistema de arquivos do Linux. A leitura dos valores dos sensores é feita através de métodos específicos dessas classes.

## Tabela de Equivalências

| Ação | NXT | EV3 |
| ------ | ------ | ------ |
| Ler Valor do Sensor Ultrassônico | `SensorType[S2] = sensorSONAR; int dist = SensorValue[S2]; // Em cm` | `ultrasonic_sensor us(INPUT_2); int dist = us.distance_centimeters();` |
| Ler Valor do Sensor de Cor | `SensorType[S3] = sensorCOLORFULL; int color = SensorValue[S3];` | `color_sensor cs(INPUT_3); int color = cs.color();` |
| Configurar Sensor de Toque | `SensorType[S1] = sensorEV3_TOUCH;` | `touch_sensor ts(INPUT_1);` |

## Exemplo de Código

### NXT (RobotC)

```c
task main()
{
  SensorType[S1] = sensorTouch;
  while(SensorValue[S1] == 0)
  {
    // Espera
  }
  // Pressionado
}
```

### Equivalente EV3 (EV3DEV C++)

```cpp
#include "ev3dev.h"

using namespace ev3dev;

int main()
{
  // 1. Instanciar o sensor de toque na porta 1
  touch_sensor ts(INPUT_1);
  // 2. Esperar em um loop enquanto não estiver pressionado
  while( !ts.is_pressed() )
  {
    // É uma boa prática esperar um pouco em loops
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
  } 
  // Pressionado
  return 0;
}
```
