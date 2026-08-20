# Configuração dos Robôs

No LEGO sempre tivemos um problema imenso: pelo fato de que todos os robôs são diferentes, cada um com seus sensores e motores, o código de controle precisa ser adaptado para cada robô. Isso significa que, se você tem 10 robôs diferentes, você precisa manter 10 cópias do mesmo código, uma para cada robô. Isso é um pesadelo de manutenção e por isso nosso Github é uma zona.

Para eliminar essa fragilidade, o FARIA adota uma arquitetura de Configuração Declarativa baseada no conceito de *Data-Driven Design*.

## Configuração Declarativa

A Engine do framework é projetada para ser uma "folha em branco" (ou *tabula rasa* como diria John Locke). O robô nasce sem saber quantos sensores possui, onde eles estão conectados ou quais são os seus motores.

A "psiquê" e a topologia física do robô são definidas inteiramente em um arquivo de texto externo no formato JSON. A responsabilidade de ler esse arquivo, traduzir o texto e popular a classe Robot com os periféricos corretos pertence exclusivamente à classe `ConfigParser`. Ela atua como uma Fábrica Dinâmica, instanciando as classes da HAL e injetando as dependências na Engine em tempo de execução.

## Apelidos (Aliases)

A maior vantagem da configuração declarativa é a criação de um dicionário entre o físico e o lógico. O ConfigParser lê o JSON, concatena o prefixo exigido pelo sistema operacional do EV3 (transformando `"B" em "outB"` por exemplo), inicializa a classe de hardware da HAL e a cadastra na Engine com o carimbo do *alias*.

A partir desse momento, a Estratégia C++ desenvolvida por nós perde completamente a noção de onde o cabo está ligado. Ela interage apenas com o Blackboard utilizando as chaves lógicas ("motorEsquerdo"). Se, no dia do campeonato, o cabo for movido para a porta "C", basta a equipe editar uma letra no arquivo de configuração de texto, sem precisar recompilar ou tocar em uma única linha de C++.

## Template de Configuração

```json
{
    "robot_name": "",
    "sensors": [
        {"type": "", "port": "", "alias": ""},
    ],
    "motors": [
        {"type": "", "port": "", "alias": ""},
    ],
    "strategies": [
        {"name": "", "class": ""},
    ]
}
```

## Exemplo de Configuração

```json
{
    "robot_name": "Robogilson",
    "sensors": [
        {"type": "ultrasonic", "port": "1", "alias": "ultrassom1"},
        {"type": "ultrasonic", "port": "2", "alias": "ultrassom2"},
        {"type": "color", "port": "3", "alias": "cor"}
    ],
    "motors": [
        {"type": "large", "port": "A", "alias": "motorEsquerdo"},
        {"type": "large", "port": "B", "alias": "motorDireito"}
    ],
    "strategies": [
        {"name": "AgressivoMaluco", "class": "AgressivoMalucoStrategy"},
        {"name": "ViraMortal", "class": "ViraMortalStrategy"}
    ]
}
```