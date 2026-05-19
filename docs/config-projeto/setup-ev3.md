# Setup do EV3

## Cartão SD

O EV3 utiliza um cartão SD para armazenar o sistema operacional e os arquivos do usuário. Para configurar o cartão SD, siga os passos abaixo:

1. Baixe a imagem do SO EV3DEV a partir do [repositório oficial](https://www.ev3dev.org/downloads/).
2. Use uma ferramenta como o [Etcher](https://www.balena.io/etcher/) para gravar a imagem no cartão SD. Baixe até a versão 17 do Etcher, pois versões mais recentes podem apresentar problemas.
3. Insira o cartão SD no computador com um adaptador de cartão SD, inicialize o Etcher, selecione a imagem baixada e o cartão SD como destino, e clique em `Flash!` (**Isso vai apagar todo o conteúdo do cartão SD**).
4. Após a gravação, insira o cartão SD no EV3 e ligue o dispositivo. O EV3 deve iniciar com o sistema operacional EV3DEV.

## Configuração de Rede via Bluetooth

Para conectar o EV3 a uma rede Wi-Fi, é preciso conectar via bluetooth à um computador ou smartphone e habilitar o *bluetooth tethering*:

### Ubuntu (16.04)

> No PC na sala há uma maquina virtual com Ubuntu 16.04 pronta para uso.

1. No EV3, vá em `Brickman > Wireless and Networks > Bluetooth > Visibile` e habilite a visibilidade.
2. No computador, abra o Blueman e no menu `View > Local Services` habilite a opção `Network Access Point` e a opção de `PAN` (Personal Area Network).
3. No Blueman, clique em `Search` para procurar o EV3, depois clique em `Pair` (ícone de chaves) para emparelhar o EV3 com o computador. Cliquei na estrela para salvar o EV3 como confiável.
4. No EV3, aceite o emparelhamento e cheque o código PIN (que aparece no computador).
5. Agora, no EV3, você deve ter a opção `Connect to Network` habilitada no menu de Bluetooth. Clique nela para conectar o EV3 à rede do computador.
6. Se tudo der certo, o EV3 deve ter o estado de rede alterado para `Online`.

### Android

1. No EV3, vá em `Brickman > Wireless and Networks > Bluetooth > Visibile` e habilite a visibilidade.
2. No smartphone, ative o Bluetooth e emparelhe com o EV3.
3. No smartphone, ative o `Bluetooth Tethering` (em `Configurações > Rede e Internet > Hotspot e tethering`, ou semelhante).
4. Confirme o emparelhamento no EV3, checando o código PIN.
5. Agora, no EV3, você deve ter a opção `Connect to Network` habilitada no menu de Bluetooth. Clique nela para conectar o EV3 à rede do smartphone.
6. Se tudo der certo, o EV3 deve ter o estado de rede alterado para `Online`.

> Existe uma forma de conectar via windows, mas não conseguimos fazer funcionar durante os testes.

## Configuração de Rede via Cabo USB

Também é possível conectar o EV3 à rede via cabo USB. Esse é o método indicado para **Windows 10**, mas ainda não foi testado. Para isso, siga os passos abaixo:

1. Conecte o EV3 ao computador usando um cabo USB.
2. Vá em `Dispositivos e Impressoras` no Windows e localize o EV3, ele deve aparecer com o nome 'Remote NDIS Compatible Device'.
3. Espere o Windows instalar os drivers necessários.
4. Após a instalação, clique com o botão direito no EV3 em `Dispositivos e Impressoras` e selecione `Configurações de Rede`.
5. Veja qual tipo de acesso à rede está disponível (geralmente `Internet - Network 5` ou semelhante).
6. Vá em `Alterar as Configurações do Adaptador` no painel lateral esquerdo e Renomeie a conexão do EV3 para algo fácil de identificar, como `EV3 USB`.
7. Para compartilhar a conexão de internet do computador com o EV3, clique com o botão direito na conexão de internet ativa (Wi-Fi ou Ethernet) e selecione `Propriedades`.
8. Vá até a aba `Compartilhamento` e marque a opção `Permitir que outros usuários da rede se conectem pela conexão deste computador à Internet`.
9. No menu suspenso abaixo, selecione a conexão do EV3 que você renomeou anteriormente (`EV3 USB`).
10. Clique em `OK` para salvar as configurações.
11. No EV3, vá em `Brickman > Wireless and Networks > USB` e selecione `Connect to Network`.
12. Se tudo der certo, o EV3 deve ter o estado de rede alterado para `Online`.

## Acesso via SSH

Após conectar o EV3 à rede, você pode acessar o dispositivo via SSH. O endereço IP do EV3 pode ser encontrado no menu `Brickman > Wireless and Networks > Wi-Fi > Status`.

Use o seguinte comando no terminal do seu computador para acessar o EV3 via SSH:

```bash
ssh robot@10.42.0.212
```

Também é possível enviar arquivos para o EV3 usando SCP:

```bash
scp <ARQUIVO> robot@10.42.0.212:/home/robot/
```

> Para a primeira conexão, a senha padrão é `maker`. Se já está configurado provavelmente colocamos **aquela** senha da Rino 👀.

## Rodando um Programa

Para rodar um programa no EV3, você pode transferir o arquivo do programa para o EV3 via SCP e depois executá-lo via SSH. Por exemplo:

```bash
scp meu_programa.py robot@10.42.0.212:/home/robot/
ssh robot@10.42.0.212 'python3 /home/robot/meu_programa.py'
```

Para programas C/C++, é aconselhável compilar no computador host e transferir o binário para o EV3, garantindo que todas as dependências estejam satisfeitas:

```bash
scp meu_programa robot@10.42.0.212:/home/robot/
ssh robot@10.42.0.212 '/home/robot/meu_programa'
```

Também é possível rodar programas pela interface do Brickman, navegando até o arquivo do programa e selecionando-o para execução.

## Erros Comuns

### Conexão não estabelecida

Se você estiver tendo problemas para estabelecer uma conexão com o EV3, certifique-se de que:

- O EV3 está ligado, ele descarrega bem rápido.
- O dispositivo está dentro do alcance do Bluetooth, caso esteja tentando wireless.
- As credenciais de acesso estão corretas.
- O firewall não está bloqueando a conexão, o windows defender é bem chato as vezes.
- Está usando o cabo USB correto em caso de ligação wired, alguns Micro-USBs são charge-only e não passam dados.

### EV3 reconhece a conexão mas o PC não (Linux)

Este problema foi recorrente na fase de testes com o Linux e é causado por uma incompatiblidade entre o NetworkManager padrão dos Linux modernos e o ConnMan utilizado pelo EV3.

No seu PC rode este comando para identificar o dispositivo do EV3:

```bash
ip link
```

Você verá interfaces comuns como `lo`, `eno1` (cabo) ou `wlan0` (wi-fi). Procure por uma interface adicional que geralmente começa com `enp` seguido de `u` (ex: `enp0s20f0u4`) ou `usb0`. Anote este nome.

Depois rode:

```bash
#Criar perfil da rede: 
nmcli connection add type ethernet ifname <Nome_da_Interface> con-name "EV3_USB"

# Mudar o IPv4 para compartilhado: 
nmcli connection modify "EV3_USB" ipv4.method shared

# Subir a interface: 
nmcli connection up "EV3_USB"
```

Isto deve funcionar para a maioria dos Linux atuais (Foi testado só no EndeavourOS).
