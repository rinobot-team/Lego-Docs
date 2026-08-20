# Camada de Abstração de Hardware (HAL)

A HAL (Hardware Abstraction Layer) funciona como uma fronteira entre o núcleo lógico do framework e o mundo físico. O objetivo desta camada é garantir que a Engine e as Estratégias nunca precisem saber que estão rodando em um LEGO EV3 ou qual biblioteca de baixo nível está sendo utilizada para acionar os pinos do processador.

## Inversão de Dependência

Todo o acesso ao hardware é regido pelo Princípio da Inversão de Dependência (a letra "D" dos Princípios SOLID). O framework consome apenas classes abstratas puramente virtuais definidas no arquivo `Interfaces.hpp`). Quando estudarem Orientação a Objetos esses termos todos que usei vão fazer mais sentido, eu prometo.

De forma resumida, o FARIA não sabe o que é a biblioteca `ev3dev`. Ela apenas sabe que existe um objeto do tipo `Motor` que possui os métodos `setSpeed()` e `getEncoderValue()`. Isso torna o framework imune a trocas de hardware. Se no futuro a equipe decidir migrar o robô do EV3 para um Lego Spike com outros motores, basta criar uma nova classe `SpikeMotor` que herde de `Motor`. Nenhuma linha da Engine central ou das Estratégias precisará ser reescrita. Isso também permite a criação de hardwares "mockados" (falsos) para rodar simulações do robô diretamente no PC, se alguém tiver coragem de fazer.

## Metaprogramação e Templates

A API C++ nativa do `ev3dev` possui classes distintas para motores grandes (`ev3dev::large_motor`) e motores médios (`ev3dev::medium_motor`), pois eles possuem características mecânicas diferentes no nível do driver. Em vez de escrevermos uma implementação de HAL separada para cada tipo de motor Lego, criamos um template genérico. O compilador C++ se encarrega de gerar as classes concretas estaticamente durante o build (instanciando `Ev3Motor<ev3dev::large_motor>` ou `Ev3Motor<ev3dev::medium_motor>`).

Como a resolução dos templates ocorre em tempo de compilação, o framework suporta múltiplos tipos de motores com zero custo adicional de processamento em tempo de execução.

## Tradução Semântica

A HAL também é responsável por normalizar as unidades de medida para que a Estratégia trabalhe sempre com números limpos e fáceis de raciocinar.

Enquanto o driver de baixo nível do EV3 pode exigir comandos de velocidade em "tiques de tacômetro por segundo", ou outra unidade alienígena, a HAL traduz a intenção do desenvolvedor. A estratégia envia um percentual de potência (de -100 a 100) para o Blackboard, e a HAL converte esse percentual na tensão ou limite de velocidade correto para o hardware específico antes de enviar o comando.