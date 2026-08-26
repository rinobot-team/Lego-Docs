# Arquitetura e Design da Interface de Usuário (TUI)

A interface de usuário do FARIA foi projetada sob o paradigma de **Text User Interface (TUI)**, operando diretamente no terminal físico (TTY0) do EV3. Tentei fazer uma UX parecida com a do NXT, acho que está bem funcional.

Tudo está definido em `include/UI.hpp` e só deve ser chamado em `main.cpp`.

## Restrições de Hardware

O EV3 possui um display LCD monocromático passivo com resolução de **178x128 pixels**.

### Ajuste de Kernel

O console do Linux nativamente tenta utilizar uma fonte microscópica (4x6) para maximizar o terminal. Para garantir a legibilidade em ambiente de competição, a classe UI injeta um comando de sistema (`setfont Lat15-Terminus14 -C /dev/tty0`) no momento de sua instanciação quue aumenta a fonte.

### Grid

O uso da fonte Terminus restringe o terminal a uma matriz rígida de aproximadamente 22 colunas por 8 linhas. O layout foi matematicamente projetado para não exceder esse limite de 22 caracteres por linha, evitando quebras de linha indesejadas (line-wrap) que quebrariam a interface.

## Renderização e ANSI Escape Codes

A TUI não utiliza nenhuma bilbioteca gráfica. Toda a renderização é feita via manipulação direta do fluxo de saída padrão (`std::cout`) utilizando sequências de escape ANSI. O autor original foi idiota o suficiente pra fazer isso do zero!

- **Atualização de Frame:** O método `clearScreen()` utiliza a sequência `\033[2J\033[H` para limpar o display e retornar o cursor à posição inicial.

- **Feedback Visual:** A seleção de itens (o cursor do usuário) é destacada invertendo as cores do terminal através do código ANSI `\033[7m` (Highlight) e resetada com `\033[0m`.
