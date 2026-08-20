# Arquitetura de Núcleo e Engine

A Engine é o núcleo (a galera mais nerd chama de *kernel*) do FARIA. A sua função é orquestrar as threads de execução, gerenciar a comunicação com os sensores e atuadores, e fornecer uma interface de alto nível para o desenvolvimento das estratégias.

## O Padrão Blackboard

Em vez de permitir que a lógica de controle (Estratégia) acesse diretamente os sensores e motores, implementamos o padrão arquitetural Blackboard. Trata-se de um espaço de memória central e thread-safe onde os dados do mundo físico são depositados e consumidos.

Imagine que duas pessoas precisam trabalhar no mesmo projeto, mas não podem se comunicar diretamente. Uma solução é criar um quadro-negro (blackboard) onde ambas podem escrever seus estados atuais. É isso que as threads do FARIA usam.

### Detalhes para Nerds

**Complexidade:** O armazenamento interno utiliza `std::unordered_map`, mapeando chaves de texto (os aliases definidos no JSON, como "motorEsquerdo") para valores inteiros. Isso garante que a busca por dados seja feita em tempo constante, `O(1)`.

**Concorrência Segura:** Todo acesso ao Blackboard (leitura ou escrita) é protegido por um `std::mutex` operando em conjunto com `std::lock_guard`. Para explicar melhor, voltando ao exemplo das duas pessoas: Pode ser que ambas escrevam ao mesmo tempo, resultando numa informação corrompida. O que o mutex o lock guard fazem é garantir que isso não ocorra. É como se só tivesse uma caneta, e as duas pessoas tem de revezar ela, garantindo que nunca haja escrita simultânea. Na computação chamamos esse erro de *condição de corrida*.

> *Nota para evitar retrabalho:* Originalmente, consideramos o uso de `std::shared_mutex` (Readers-Writer Lock) para otimizar múltiplas leituras simultâneas. Mas, devido a um bug conhecido na implementação de variáveis de condição estáticas no compilador GCC 7.3.0 (__concurrence_broadcast_error), deu muita merda.

**Prevenção de Falhas:** Os métodos de leitura (como `getSensorValue`) não retornam tipos primitivos brutos. Eles retornam um `std::optional<int>`. Se uma estratégia tentar ler um sensor chamado "cor1", mas o cabo estiver desconectado ou o nome no JSON estiver errado, o Blackboard retorna `std::nullopt` em vez de um falso 0. Isso permite que a estratégia trate a falha graciosamente em vez de tomar decisões baseadas em dados fantasmas.

## Multithreading

A classe `Robot` divide o fluxo de execução do hardware em duas threads concorrentes e isoladas. Isso garante que o robô seja reativo e que chamadas de hardware lentas não congelem a tomada de decisão.

O que são threads? São linhas de execução independentes que podem rodar em paralelo (ao mesmo tempo). No caso do FARIA, temos:

- **Thread de Percepção (Read-Only do Mundo Real):** Seu único trabalho é iterar infinitamente sobre a lista de sensores e os encoders dos motores. Ela puxa os dados físicos via HAL (Hardware Abstraction Layer) e escreve os valores atualizados no Blackboard.
- **Thread de Ação (Write-Only para o Mundo Real):** Esta thread é o motor de execução. A cada ciclo, ela lê o estado atualizado do Blackboard, injeta esse estado na Estratégia ativa (que roda sua matemática de controle) e coleta as respostas de velocidade. Em seguida, ela varre o Blackboard pegando as novas velocidades e envia os comandos reais de potência para os motores.

## Gestão de Ciclo de Vida e Segurança (RAII)

Na robótica, um bug no código pode resultar em um robô desgovernado fisicamente. A classe `Robot` aplica o idioma C++ RAII (Resource Acquisition Is Initialization).

> **Nota do Chaves**: Recomendo muito o livro [C++ Moderno e Eficaz](https://www.amazon.com.br/moderno-eficaz-formas-específicas-aprimorar/dp/8550800031) do Scott Meyers. Ele explica muito bem conceitos atuais de C++ como RAII, smart pointers, move semantics, etc.

Algumas das garantias que o RAII nos dá:

- **Atomicidade:** O estado de execução do robô é mantido em uma variável `std::atomic<bool> m_running`. Ela garante que comandos de start ou stop, seja via teclado, rede ou interface gráfica, alterem o estado instantaneamente e de forma segura entre as threads.
- **Destruição Segura:** Quando o programa é encerrado (ou abortado), o destrutor `~Robot()` é invocado. Ele chama o método `stop()`, sinalizando as flags atômicas para falso e aplicando `join()` nas threads de percepção e ação. Isso garante que o programa só devolva o controle ao sistema operacional depois que o loop de ação tiver finalizado sua iteração atual e cortado a energia dos motores. Evitando que o robô continue andando desgovernado após o programa encerrar.