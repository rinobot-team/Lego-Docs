# Migração do LCD

Esta seção aborda a migração do código relacionado ao LCD do NXT para o EV3.

## Diferenças Principais

No RobotC, o LCD do NXT é acessado através de funções específicas, como `nxtDisplayString()` para exibir texto e `nxtDrawLine()` para desenhar linhas. Essas funções são diretas e utilizam coordenadas simples para posicionamento.

No EV3 temos duas opções, printar na tela pelo serial padrão, ou modificar o LCD. O LCD  é manipulado através de um framebuffer, que é uma representação em memória da tela.

## Printar no Serial Padrão

No EV3, você pode usar a saída padrão para imprimir texto na tela do LCD. Isso é feito utilizando as funções padrão de C++ como `std::cout`. Aqui está um exemplo simples:

```cpp
#include <iostream>
#include "ev3dev.h"
using namespace ev3dev;
int main() {
    std::cout << "Hello, EV3!" << std::endl;
    return 0;
}
```

Essa maneira é mais simples e recomendada para a maioria dos casos, especialmente para depuração e mensagens simples.

## Framebuffer no EV3

O framebuffer é a camada abaixo disso. É o método de mais baixo nível (sem ser o driver do kernel) que o Linux usa para desenhar na tela.

É uma abordagem mais complexa, mas oferece controle total sobre cada pixel na tela.

### O Que é o Framebuffer?

Pense no framebuffer como um grande *mapa de bits* (bitmap) na RAM que representa diretamente cada pixel da tela.

- É um buffer (um pedaço de memória) contínuo.
- Cada célula nesse buffer corresponde a um pixel (ou um conjunto de pixels).
- O hardware de vídeo do EV3 está configurado para, constantemente, ler essa área da memória e exibi-la na tela.
- Se você escrever dados diretamente nessa área da memória, a imagem na tela muda instantaneamente (no próximo ciclo de atualização).

> No EV3, o framebuffer é de 176x128 pixels, com 1 bit por pixel (preto e branco).

### Como Usar o Framebuffer

Para manipular o framebuffer diretamente, você pode abrir o dispositivo `/dev/fb0` e escrever dados nele. Aqui está um exemplo básico de como fazer isso:

```cpp
#include "ev3dev.h"
#include <fcntl.h>
#include <unistd.h>
#include <sys/mman.h>
#include <linux/fb.h>
#include <sys/ioctl.h>
#include <iostream>

bool set_console_bind(bool bind) 
{
    // O arquivo de controle pode ser vtcon0 ou vtcon1.
    // No ev3dev, geralmente é o vtcon0.
    std::ofstream f("/sys/class/vtconsole/vtcon0/bind");
    if (!f.is_open()) {
        // Tenta o vtcon1 como fallback
        f.open("/sys/class/vtconsole/vtcon1/bind");
    }
    if (!f.is_open()) {
        std::cerr << "Erro: Nao foi possivel abrir /sys/class/vtconsole/..." << std::endl;
        return false;
    }
    // Escreve "0" para desvincular ou "1" para vincular
    f << (bind ? "1" : "0");
    f.close();
    return true;
}

int main()
{
    // --- 1. Abrir o Framebuffer ---
    int fb_fd = open("/dev/fb0", O_RDWR);
    if (fb_fd == -1) { /* erro */ }
    if (!set_console_bind(false)) {
        return 1; // Falha
    }
    // --- 2. Obter informações da tela (ESPECIALMENTE o line_length) ---
    fb_fix_screeninfo finfo;
    if (ioctl(fb_fd, FBIOGET_FSCREENINFO, &finfo) == -1) {
        // finfo.line_length nos dirá o "stride" (provavelmente 24 bytes)
        /* erro */
    }
    long screen_size_bytes = finfo.smem_len; // Tamanho total do buffer
    int line_length_bytes = finfo.line_length; // "Stride" (bytes por linha)
    // --- 3. Mapear para a Memória ---
    unsigned char* fb_ptr = (unsigned char*)mmap(
        0, 
        screen_size_bytes, 
        PROT_READ | PROT_WRITE, 
        MAP_SHARED, 
        fb_fd, 
        0
    );
    if (fb_ptr == (void*)-1) { /* erro */ }
    // --- 4. Desenhar! ---
    // Limpar a tela (definir todos os bytes como 0x00 = branco)
    memset(fb_ptr, 0x00, screen_size_bytes);
    // Desenhar um pixel em (x=10, y=5)
    int x = 10;
    int y = 5;
    // Calcular o endereço do byte
    // Pula 'y' linhas (cada uma com 'line_length_bytes')
    // e avança 'x / 8' bytes na linha atual
    unsigned char* byte_ptr = fb_ptr + (y * line_length_bytes) + (x / 8);
    // Calcular o bit dentro desse byte
    // (o bit 0 é o mais à esquerda, ou o mais à direita? 
    // No EV3, é little-endian, então o bit 0 é o da direita)
    int bit_index = x % 8;
    // Ligar o pixel (preto)
    *byte_ptr |= (1 << bit_index);
    // Esperar um pouco para ver
    std::this_thread::sleep_for(std::chrono::seconds(5));
    // --- 5. Limpar ---
    munmap(fb_ptr, screen_size_bytes);
    close(fb_fd);
    // Lembre-se de reativar o console se você o desligou!
    set_console_bind(true);
    return 0;
}
```
