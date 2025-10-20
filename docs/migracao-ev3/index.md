# Migração para o EV3

Esta seção aborda o processo de migração do código e configuração para o robô baseado no EV3, destacando as principais diferenças e adaptações necessárias em relação ao NXT.

Pontos importantes a serem considerados durante a migração:

- Certifique-se de seguir as instruções de configuração do EV3 na seção [Setup do EV3](../config-projeto/setup-ev3.md).
- O SDK para o EV3 é diferente do NXT, então você precisará adaptar seu código para usar as bibliotecas e APIs específicas do EV3. Enquanto o NXT usa uma API procedural baseada em C, o EV3 é 100% baseado em C++11 e orientado a objetos.
- O RobotC roda como um firmware no NXT, enquanto o EV3DEV C++ roda como um programa de usuário no EV3, que possui um sistema operacional Linux completo (baseado em Debian). Isso afeta como a interação com o hardware e o sistema é realizada: O hardware é exposto como arquivos na pasta `/sys/class`, e a comunicação com sensores e atuadores é feita através da leitura e escrita nesses arquivos.

Essa documentação ainda está em desenvolvimento e ainda não cobre todos os aspectos da migração, até o momento há essas interfaces:

- [Motores](./motores.md).
- [Sensores](./sensores.md).
- [LCD](./lcd.md).
