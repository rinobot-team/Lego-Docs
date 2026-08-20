# Infraestrutura e Ferramental (Tooling)

O FARIA foi arquitetado para operar em sistemas embarcados com restrições de hardware e sistema operacional. A infraestrutura base visa garantir previsibilidade: o código deve ser agnóstico à máquina de quem o programa, e o binário gerado deve ser imutável e autossuficiente quando executado no LEGO EV3.

## Ambiente de Desenvolvimento Padronizado

Para evitar a situação de "funciona na minha máquina", o ecossistema de desenvolvimento foi ancorado em **Linux**, por ser o sistema operacional superior, de pessoas com caráter.

**O Problema do Windows:** Sistemas Windows utilizam quebras de linha diferentes (`CRLF` em vez de `LF`) e possuem sistemas de arquivos *case-insensitive* (onde `Motor.hpp` e `motor.hpp` são tratados como iguais). Isso frequentemente causa quebras silenciosas ou falhas de compilação quando o código é enviado para o Linux embarcado do EV3.

**A Solução (WSL):** Desenvolvedores no Windows devem utilizar o [WSL](https://docs.microsoft.com/pt-br/windows/wsl/). Os scripts em Python do framework foram projetados para detectar o ambiente e realizar a tradução de caminhos de forma transparente (ex: convertendo `C:\Users\...` para `/mnt/c/Users/...` antes de invocar o compilador).

## Cross-Compilation Estática Hermética (Falei bonito né?)

O EV3 roda em um processador ARM9 (ARM926EJ-S) e utiliza uma imagem Linux baseada no Debian antigo, o que significa que suas bibliotecas dinâmicas do sistema (como a `glibc` e a `libstdc++`) estão defasadas.

Para contornar essa limitação e usar C++ moderno (C++17) com segurança, aplicamos o conceito de **Cross-Compilation Estática**:

- **Toolchain Específica:** O projeto baixa e utiliza uma *toolchain* isolada (GCC 7.3.0 `armv5-eabi`), que compila o código na arquitetura x86 do PC do desenvolvedor, mas gera instruções em linguagem de máquina específicas para a arquitetura ARM do EV3.
- **Linkagem Estática Total:** Utilizamos a flag `-static` para empacotar todas as dependências matemáticas e de sistema dentro de um único arquivo executável (o binário não dependerá de nada que está instalado no robô).

## Orquestração, Build e Deploy CLI

Toda a complexidade de compilar e transferir o código é abstraída por uma suíte de scripts Python unificada em uma única Command Line Interface (CLI): o `deployer.py`.

Abaixo estão os módulos internos que compõem o pipeline:

- `setup.py` (Inicialização): Faz o download atômico da toolchain de cross-compilação `armv5-eabi`, extrai e valida os caminhos do compilador g++ localmente na pasta `toolchain/`.
- `build.py` (Compilação): Inspeciona a árvore do diretório `src/`, agrupa todos os artefatos `.cpp`, lê as flags do global.json e invoca a toolchain isolada.
- `upload.py` (Deploy): Conecta-se ao robô via rede (SSH); espelha a configuração enviando o arquivo `.json` do robô alvo para o diretório `/home/robot/config/`; Envia o binário e aplica privilégios do Linux (`chmod +x`).

### Workflow do CLI

```bash
# Configurar o ambiente do zero (rodar apenas na primeira vez)
./deployer.py setup

# Compilar todo o projeto visando um robô específico
./deployer.py build robo_exemplo

# Enviar o binário compilado para o hardware
./deployer.py upload robo_exemplo --ip 192.168.10.1

# Compilar e fazer upload em uma única etapa
./deployer.py all robo_exemplo --ip 192.168.10.1
```

> É normal que, ao fazer o upload, o robô peça a senha do usuário duas vezes, uma para o SSH e a outra para o `chmod +x`.


