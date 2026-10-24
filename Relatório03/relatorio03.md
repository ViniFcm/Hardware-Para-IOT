# Relatório 03 – Atividade HIoT_11

## 1. Objetivo da prática

- Utilizar novamente o sensor ultrassônico HC-SR04 com a ESP32;
- Determinar a distância de um objeto posicionado à frente do sensor;
- Utilizar interrupção para cronometrar com maior precisão o intervalo entre a subida e a descida do sinal ECHO;
- Converter o tempo de propagação do sinal em distância, em centímetros;
- Informar a distância medida textualmente pela porta serial.

---

## 2. Materiais utilizados

- Placa ESP32;
- Sensor ultrassônico HC-SR04;
- Protoboard;
- Resistor de 1,1 kΩ;
- Resistor de 2 kΩ;
- Jumpers;
- Cabo USB para alimentação e programação;
- Computador com Arduino IDE ou ambiente de desenvolvimento equivalente.

---

## 3. Montagem do circuito

O sensor HC-SR04 foi conectado à ESP32 conforme a lógica do código:

- `trigPin = 27`;
- `echoPin = 25`;
- alimentação e aterramento comuns entre o sensor e a ESP32.

Como a saída ECHO do HC-SR04 pode atingir 5 V, foi utilizado um divisor de tensão antes da entrada da ESP32. Os resistores de 1,1 kΩ e 2 kΩ foram ligados em série entre o ECHO e o GND, nessa ordem. O ponto entre os dois resistores foi conectado ao GPIO 25 da ESP32, reduzindo a tensão para um nível adequado à entrada da placa.

O pino TRIG foi conectado diretamente ao GPIO 27. Durante a leitura, a ESP32 envia um pulso de 10 microssegundos pelo TRIG. O tempo em que o ECHO permanece em nível alto é medido por interrupção, permitindo calcular a distância do objeto.

### Imagens da montagem

![Montagem do circuito](WhatsApp%20Image%202026-10-23%20at%2019.56.44.jpeg)

![Vista da montagem na protoboard](WhatsApp%20Image%202026-10-23%20at%2019.56.44%20%281%29.jpeg)

![Detalhe do sensor ultrassônico](WhatsApp%20Image%202026-10-23%20at%2019.56.44%20%282%29.jpeg)

![Montagem completa](WhatsApp%20Image%202026-10-23%20at%2019.56.44%20%283%29.jpeg)

---

## 4. Funcionamento do programa

Foi utilizada uma interrupção configurada para atuar nas duas mudanças de estado do pino ECHO (`CHANGE`):

1. Na subida do sinal ECHO, o instante inicial é registrado com `micros()`;
2. Na descida do sinal ECHO, o instante final é registrado;
3. A diferença entre os dois instantes representa a duração do pulso recebido;
4. A duração é convertida em distância usando a velocidade do som;
5. O resultado é enviado ao monitor serial no formato `Distância: valor cm`.

Caso não seja recebida uma resposta em até 50 milissegundos, o programa informa um erro de leitura por timeout.

---

## 5. Código desenvolvido

Arquivo principal da prática:

[Código da atividade HIoT_11](codigo-atividade-HIoT__11.ino)

```cpp
const int trigPin = 27;
const int echoPin = 25;

#define SOUND_SPEED 0.034 // cm/us

volatile unsigned long tempoInicio = 0;
volatile unsigned long tempoFim = 0;
volatile bool novaLeituraDisponivel = false;

void IRAM_ATTR echoISR() {
  if (digitalRead(echoPin) == HIGH) {
    tempoInicio = micros();
  } else {
    tempoFim = micros();
    novaLeituraDisponivel = true;
  }
}

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  digitalWrite(trigPin, LOW);
  Serial.begin(115200);

  attachInterrupt(digitalPinToInterrupt(echoPin), echoISR, CHANGE);
}

void loop() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  unsigned long tempoEspera = millis();
  while (!novaLeituraDisponivel && (millis() - tempoEspera < 50)) {
  }

  if (novaLeituraDisponivel) {
    noInterrupts();
    unsigned long duracao = tempoFim - tempoInicio;
    novaLeituraDisponivel = false;
    interrupts();

    float distanciaCM = (duracao * SOUND_SPEED) / 2.0;

    Serial.print("Distância: ");
    Serial.print(distanciaCM);
    Serial.println(" cm");
  } else {
    Serial.println("Distância: Erro de leitura (Timeout)");
  }

  delay(1000);
}
```

---

## 6. Resultado

Após a montagem e a gravação do programa na ESP32, o sensor realizou leituras sucessivas da distância do objeto. Os valores foram exibidos no monitor serial a cada segundo, acompanhados da unidade `cm`.

A utilização da interrupção permitiu registrar os instantes de subida e descida do ECHO diretamente, evitando o uso de uma função de bloqueio para cronometrar o pulso. Quando não houve retorno do sensor dentro do tempo limite, foi exibida a mensagem `Distância: Erro de leitura (Timeout)`.

---

## 7. Conclusão

A atividade possibilitou aplicar interrupções na ESP32 para realizar a medição do tempo de resposta do sensor HC-SR04. O uso de `micros()` nas bordas do sinal ECHO tornou possível calcular a duração do pulso e, consequentemente, a distância do objeto. O divisor de tensão também protegeu a entrada da ESP32 contra o nível de 5 V fornecido pelo sensor.

---

## 8. Arquivos relacionados

- [PDF da atividade](HIoT_11.pdf)
- [Código da prática](codigo-atividade-HIoT__11.ino)
- [Imagem da montagem 1](WhatsApp%20Image%202026-10-23%20at%2019.56.44.jpeg)
- [Imagem da montagem 2](WhatsApp%20Image%202026-10-23%20at%2019.56.44%20%281%29.jpeg)
- [Imagem da montagem 3](WhatsApp%20Image%202026-10-23%20at%2019.56.44%20%282%29.jpeg)
- [Imagem da montagem 4](WhatsApp%20Image%202026-10-23%20at%2019.56.44%20%283%29.jpeg)
