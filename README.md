
# Contador Binário com LEDs - ESP32
## Autor
Guilherme Pedro de Oliveira Souza N°USP 11801178

Este projeto foi desenvolvido para implementar um **contador binário** utilizando **LEDs** no **ESP32**, controlado por **FreeRTOS**. A implementação consiste em um contador que vai de **0 a 31**, representando o número binariamente através dos LEDs. Os LEDs acendem conforme o valor binário do contador, representando cada bit do número.

## Objetivo
O objetivo principal deste projeto foi criar um contador binário utilizando **LEDs**, onde cada LED representa uma potência de 2. O contador é implementado de forma que o número binário é mostrado progressivamente nos LEDs, de 0 a 31.

Além disso, fiz uma tentativa de implementar o mesmo contador utilizando **displays de 7 segmentos** para exibir o número de forma decimal. No entanto, devido a limitações de tempo na plataforma de simulação (**Wokwi**), não consegui concluir a implementação com os displays. Para isso, incluí uma imagem explicando o problema de **overtime**.

## Estrutura do Projeto

1. **Contador Binário com LEDs**:
    - **LED 1** representa $2^0$ (1),
    - **LED 2** representa $2^1$ (2),
    - **LED 3** representa $2^2$ (4),
    - **LED 4** representa $2^3$ (8),
    - **LED 5** representa $2^4$ (16).
   
    O contador vai de **0 a 31** e o valor binário é representado pelos LEDs acesos.

2. **Tentativa com Displays de 7 Segmentos**:
    - A ideia era utilizar dois **displays de 7 segmentos** para exibir o número de 0 a 31 em formato decimal.
    - Devido a limitações de tempo na plataforma, não conseguimos concluir essa implementação a tempo.

## Implementação do Código

### Contador Binário com LEDs
A primeira parte do código implementa a contagem binária, onde cada LED é aceso conforme o valor binário do contador. O **FreeRTOS** foi utilizado para gerenciar as tarefas de contagem e controle dos LEDs em núcleos diferentes do ESP32.

#### Código do Contador Binário com LEDs:

```cpp
#include <Arduino.h>

#define LED_PIN_1 2   // LED 1 -> 2^0 (1)
#define LED_PIN_2 4   // LED 2 -> 2^1 (2)
#define LED_PIN_3 5   // LED 3 -> 2^2 (4)
#define LED_PIN_4 18  // LED 4 -> 2^3 (8)
#define LED_PIN_5 19  // LED 5 -> 2^4 (16)

int counter = 0;  // Contador de 0 a 31

// Task 1 - Contagem binária (alta prioridade, núcleo 1)
void countTask(void *parameter) {
  while (1) {
    counter = (counter + 1) % 32;  // Incrementa o contador e garante que ele vai de 0 a 31
    Serial.println(counter);  // Exibe o contador no monitor serial
    delay(1000);  // Delay de 1 segundo
  }
}

// Task 2 - Controle dos LEDs (baixa prioridade, núcleo 0)
void ledTask(void *parameter) {
  while (1) {
    // Apaga todos os LEDs inicialmente
    digitalWrite(LED_PIN_1, LOW);
    digitalWrite(LED_PIN_2, LOW);
    digitalWrite(LED_PIN_3, LOW);
    digitalWrite(LED_PIN_4, LOW);
    digitalWrite(LED_PIN_5, LOW);

    // Controle dos LEDs de acordo com o valor binário do contador
    if (counter & 1) digitalWrite(LED_PIN_1, HIGH);  // LED 1 (2^0)
    if (counter & 2) digitalWrite(LED_PIN_2, HIGH);  // LED 2 (2^1)
    if (counter & 4) digitalWrite(LED_PIN_3, HIGH);  // LED 3 (2^2)
    if (counter & 8) digitalWrite(LED_PIN_4, HIGH);  // LED 4 (2^3)
    if (counter & 16) digitalWrite(LED_PIN_5, HIGH); // LED 5 (2^4)

    delay(100);  // Delay de 100 ms para atualização dos LEDs
  }
}

void setup() {
  Serial.begin(115200);

  // Configura os pinos dos LEDs como saída
  pinMode(LED_PIN_1, OUTPUT);
  pinMode(LED_PIN_2, OUTPUT);
  pinMode(LED_PIN_3, OUTPUT);
  pinMode(LED_PIN_4, OUTPUT);
  pinMode(LED_PIN_5, OUTPUT);

  // Criação das tasks no FreeRTOS
  xTaskCreatePinnedToCore(countTask, "Count Task", 10000, NULL, 1, NULL, 1);  // Task 1 no núcleo 1 (alta prioridade)
  xTaskCreatePinnedToCore(ledTask, "LED Task", 10000, NULL, 1, NULL, 0);      // Task 2 no núcleo 0 (baixa prioridade)
}

void loop() {
  // O loop não é utilizado, pois as tasks são gerenciadas pelo FreeRTOS
}
```

Tentativa com Displays de 7 Segmentos

A segunda parte do código era destinada a displays de 7 segmentos, onde os números de 0 a 31 seriam exibidos utilizando dois displays: um para as unidades e outro para as dezenas.

Código da Tentativa com Displays de 7 Segmentos (não concluído):

```cpp
#include <Arduino.h>

// Definindo pinos para os segmentos dos displays
#define SEG_A 13
#define SEG_B 12
#define SEG_C 14
#define SEG_D 27
#define SEG_E 26
#define SEG_F 25
#define SEG_G 33
#define DP 32  // Pino de ponto decimal (não usado)

// Segmentos do Display para números de 0 a 9 (em binário)
const byte digitSegments[] = {
  B1111110,  // 0
  B0110000,  // 1
  B1101101,  // 2
  B1111001,  // 3
  B0110011,  // 4
  B1011011,  // 5
  B1011111,  // 6
  B1110000,  // 7
  B1111111,  // 8
  B1111011   // 9
};

// Função para mostrar um número no display de 7 segmentos
void displayNumber(int number) {
  byte tens = number / 10;  // Dezena
  byte ones = number % 10;  // Unidade

  // Exibe as dezenas
  byte segmentsTens = digitSegments[tens];
  digitalWrite(SEG_A, (segmentsTens & B1000000) ? HIGH : LOW);
  digitalWrite(SEG_B, (segmentsTens & B0100000) ? HIGH : LOW);
  digitalWrite(SEG_C, (segmentsTens & B0010000) ? HIGH : LOW);
  digitalWrite(SEG_D, (segmentsTens & B0001000) ? HIGH : LOW);
  digitalWrite(SEG_E, (segmentsTens & B0000100) ? HIGH : LOW);
  digitalWrite(SEG_F, (segmentsTens & B0000010) ? HIGH : LOW);
  digitalWrite(SEG_G, (segmentsTens & B0000001) ? HIGH : LOW);

  delay(5);  // Delay de 5 ms para dar tempo para o display

  // Exibe as unidades
  byte segmentsOnes = digitSegments[ones];
  digitalWrite(SEG_A, (segmentsOnes & B1000000) ? HIGH : LOW);
  digitalWrite(SEG_B, (segmentsOnes & B0100000) ? HIGH : LOW);
  digitalWrite(SEG_C, (segmentsOnes & B0010000) ? HIGH : LOW);
  digitalWrite(SEG_D, (segmentsOnes & B0001000) ? HIGH : LOW);
  digitalWrite(SEG_E, (segmentsOnes & B0000100) ? HIGH : LOW);
  digitalWrite(SEG_F, (segmentsOnes & B0000010) ? HIGH : LOW);
  digitalWrite(SEG_G, (segmentsOnes & B0000001) ? HIGH : LOW);

  delay(5);  // Delay de 5 ms para dar tempo para o display
}
```

## Desafios e Limitações

Durante a implementação com os displays de 7 segmentos, encontrei limitações de tempo na plataforma Wokwi, o que resultou em uma implementação sem a visualização adicional. A plataforma apresentou "overtime".

<img width="1872" height="931" alt="tentativacomovertime" src="https://github.com/user-attachments/assets/e7a511f1-6ff1-41e2-a811-1973cd2211ea" />

## Vídeo do Contador com LEDs Funcionando

https://youtu.be/Vo1K6kM6LD8

## Conclusão

Neste projeto, consegui implementar um contador binário utilizando LEDs com FreeRTOS no ESP32, onde os LEDs representam os valores binários de 0 a 31. A tentativa com os displays de 7 segmentos foi iniciada, mas não foi concluída devido a limitações de tempo na plataforma de simulação Wokwi.
