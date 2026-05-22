# Migração dos Motores

Esta seção aborda a migração do código relacionado aos motores do NXT para o EV3.

## Diferenças Principais

No RobotC, os motores são acessados diretamente através de arrays pré-definidos, como `motor[]`, onde cada índice corresponde a uma porta específica do motor. Por exemplo, `motor[motorA]` acessa o motor conectado à porta A. 

Já no EV3, os motores são representados como objetos de classes específicas, como `ev3dev::motor::large_motor` ou `ev3dev::motor::medium_motor`. Cada motor é instanciado com base na porta à qual está conectado, utilizando a nomenclatura do sistema de arquivos do Linux.
Além disso, o motor não é controlado diretamente por valores numéricos, mas sim métodos que definem comportamentos específicos, como velocidade, posição, etc.

## Tabela de Equivalências

| Ação | NXT | EV3 |
| ------ | ------ | ------ |
| Girar Continuamente | `motor[motorA] = 50;` | `large_motor motorA("outA"); motorA.set_speed_sp(500); motorA.run_forever();` |
| Parar Motor | `motor[motorA] = 0;` | `motorA.stop();` |
| Ler Enconder | `long valor = nMotorEncoder[motorA];` | `int valor = motorA.position();` |
| Redefinir Encoder | `nMotorEncoder[motorA] = 0;` | `motorA.set_position(0);` |

## Exemplo de Código

### NXT (RobotC)

```c
task main() {
    // Girar motor A a 50% de potência
    motor[motorA] = 50;
    wait1Msec(2000); // Espera por 2 segundos
    // Parar o motor
    motor[motorA] = 0;
    // Ler o valor do encoder
    long valor = nMotorEncoder[motorA];
    // Redefinir o encoder
    nMotorEncoder[motorA] = 0;
}
```

### Equivalente EV3 (EV3DEV C++)

```cpp
#include "ev3dev.h"
#include <thread>
#include <chrono>

using namespace ev3dev;

int main() {
  // 1. Instanciar o motor na porta B
  motor motor_b(OUTPUT_B);
  // 2. Definir velocidade (ex: 450 graus/seg)
  motor_b.set_speed_sp(450);
  // 3. Ligar o motor
  motor_b.run_forever();
  // 4. Esperar 2 segundos (usando C++ moderno)
  std::this_thread::sleep_for(std::chrono::seconds(2));
  // 5. Parar o motor
  motor_b.stop();
  return 0;
}
```
