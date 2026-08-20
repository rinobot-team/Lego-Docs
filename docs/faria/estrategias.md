# Estratégias no FARIA

No FARIA, a lógica de tomada de decisão foi completamente isolada da infraestrutura de hardware através do Padrão de Projeto comportamental Strategy (Estratégia).

O objetivo desta arquitetura é permitir que os desenvolvedores foquem puramente na matemática, no controle e na lógica de luta, sem nunca precisarem se preocupar com como um motor gira, como uma thread é travada ou como um arquivo JSON é lido.

**É onde os membros mais vão trabalhar.**

## Isolamento de Domínio

É aqui que o Blackboard entra em ação. As classes de estratégia do FARIA são cegas e surdas para o mundo real. Elas não interagem com a HAL. Em vez disso, elas recebem uma referência para o Blackboard a cada ciclo de execução (chamamos de *tick*, igual no Minecraft).

Toda a inteligência do robô se resume a uma equação simples:

1. Ler os dados do Blackboard (vindos da Thread de Percepção).
2. Processar a lógica de controle e decidir o que fazer.
3. Escrever os comandos resultantes de volta no Blackboard, para a Thread de Ação (ex: velocidade de -100 a 100 para os motores).

## A Anatomia de uma Estratégia

Para que o framework reconheça uma nova estratégia, o desenvolvedor precisa criar uma classe que cumpra rigorosamente um contrato. Este contrato é garantido pela herança da classe base puramente virtual Strategy.

Todo arquivo de estratégia (um `.hpp` dentro de `include/strategies/`) deve conter o seguinte *boilerplate* (código base obrigatório):

- **Herança:** A classe deve herdar publicamente de `Strategy`: `class EstrategiaMuitoFoda : public Strategy`.
- **O Método `execute`:** Devemos sobrescrever o método `void execute(Blackboard& bb) override`. É dentro dele que a lógica roda. Ele é chamado continuamente pela Thread de Ação do framework.
- **O Método `get_name`**: Devemos sobrescrever o método `std::string getName() const override`. Ele retorna uma string com o nome da estratégia para fins de logs, depuração e telemetria.

> Por conveção, todas as estratégias devem ser implementadas em arquivos `.hpp` (C++ Header) e com `Strategy` no final do nome da classe. Exemplo: `AgressivoMalucoStrategy.hpp`.

## Exemplo de Estratégias

Estratégia simples para evitar obstáculos:

```cpp
// Arquivo include/strategies/EvitaObstaculoStrategy.hpp
#pragma once

#include "../Interfaces.hpp"

// A classe herda do contrato base
class EvitaObstaculo : public Strategy {
public:
    // O método execute é onde a lógica opera a cada ciclo
    void execute(Blackboard& bb) override {
        // Passo 1: Ler o sensor usando o alias definido no JSON
        // Utilizamos std::optional pois o hardware pode ter falhado
        auto distOpt = bb.getSensorValue("ultrassomFrente");
        
        // Se o sensor não estiver lendo, assumimos uma distância segura (ex: 999mm)
        int distMm = distOpt.value_or(999);

        // Passo 2: Lógica de controle simples
        if (distMm < 200) {
            // Obstáculo muito perto: Girar no próprio eixo para a direita
            bb.setMotorSpeed("motorEsquerdo", 50);
            bb.setMotorSpeed("motorDireito", -50);
        } else if (distMm < 500) {
            // Obstáculo se aproximando: Reduzir velocidade
            bb.setMotorSpeed("motorEsquerdo", 30);
            bb.setMotorSpeed("motorDireito", 30);
        } else {
            // Caminho livre: Mete marcha
            bb.setMotorSpeed("motorEsquerdo", 100);
            bb.setMotorSpeed("motorDireito", 100);
        }
    }

    //  O método getName identifica a estratégia no sistema
    [[nodiscard]] std::string getName() const override {
        return "EvitaObstaculo";
    }
};
```

## O Strategy Registry

Se você criasse uma nova estratégia, como o FARIA saberia instanciá-la a partir de uma string recebida por um menu ou pela rede, sem criar um bloco de `if / else if` gigantesco?

O desenvolvedor do FARIA é muito foda, então ele criou um mecanismo de registro de estratégias. Trata-se de um mapa (ou Dicionário) que associa uma string a uma função de fábrica (uma *função lambda* que sabe como criar o objeto na memória).

Para acoplar a estratégia recém-criada ao robô, o desenvolvedor só precisa adicioná-la ao Registro no ponto de entrada (`main.cpp`).

Exemplo de registro no `main.cpp`:

```cpp
#include "strategies/EvitaObstaculo.hpp" // Importar o header da nova estratégia

// ... dentro do main() ...

StrategyRegistry registry;

// Registrando a estratégia através de uma lambda factory
registry.registerStrategy("EvitaObstaculo", []() {
    return std::make_unique<EvitaObstaculo>();
});

// Solicitando a criação dinâmica e injetando no robô
std::string chosenStrategy = "EvitaObstaculo";
robot.setStrategy(registry.create(chosenStrategy));
```

> **Importante:** Essa última parte da estratégia será mudada em breve, com a implementação da interface gráfica de seleção de estratégias, que permitirá ao usuário escolher a estratégia desejada sem precisar recompilar o código.