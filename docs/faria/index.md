# Migração para o EV3 e FARIA

Esta seção aborda o processo de migração do código e configuração para o robô baseado no EV3, destacando as principais diferenças e adaptações necessárias em relação ao NXT.

Pontos importantes a serem considerados durante a migração:

- Certifique-se de seguir as instruções de configuração do EV3 na seção [Setup do EV3](../config-projeto/setup-ev3.md).
- O SDK para o EV3 é diferente do NXT, então você precisará adaptar seu código para usar as bibliotecas e APIs específicas do EV3. Enquanto o NXT usa uma API procedural baseada em C, o EV3 é 100% baseado em C++11 e orientado a objetos.
- O RobotC roda como um firmware no NXT, enquanto o EV3DEV C++ roda como um programa de usuário no EV3, que possui um sistema operacional Linux completo (baseado em Debian). Isso afeta como a interação com o hardware e o sistema é realizada: O hardware é exposto como arquivos na pasta `/sys/class`, e a comunicação com sensores e atuadores é feita através da leitura e escrita nesses arquivos.

Essa documentação ainda está em desenvolvimento e ainda não cobre todos os aspectos da migração, até o momento há essas interfaces:

- [Motores](./motores.md).
- [Sensores](./sensores.md).
- [LCD](./lcd.md).

## FARIA

O FARIA é o framework de controle que está sendo desenvolvido para o EV3, e tem como objetivo fornecer uma interface semelhante ao que tínhamos no NXT, mas adaptada para as características do EV3 e do EV3DEV C++. Ele ainda está em desenvolvimento, mas a ideia é que ele seja uma camada de abstração sobre o EV3DEV C++, facilitando a migração do código e a implementação de novas funcionalidades como parametrização das configurações dos robôs, multithreading e melhores padrões de código.

O FARIA é um projeto open-source, e a documentação está sendo construída à medida que o desenvolvimento avança. Para acompanhar o progresso do FARIA, você pode acessar o [repositório oficial no GitHub](https://github.com/rinobot-team/FARIA).

Para informações sobre a proposta de arquitetura do FARIA, veja a seção [Arquitetura do FARIA](./arquitetura.md).