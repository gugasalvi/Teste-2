# ROTEIRO 7 — SENSOR DHT11 INSTALADO POR OTA NO NODEMCU (ESP8266)

No Roteiro 6, a placa passou a receber novos programas pela rede Wi-Fi (OTA) e terminou na versão **3.0**, com os botões **Ligar** e **Desligar** do LED interno. Agora vamos acrescentar um sensor real ao projeto e **instalar o novo programa à distância**, sem cabo de dados.

## OBJETIVO

Integrar o sensor **DHT11** (temperatura e umidade) ao projeto do Roteiro 6 e enviar o programa por **OTA**. Ao final, a página da placa deve mostrar:

- a versão do firmware;
- a temperatura e a umidade medidas pelo DHT11;
- os botões do LED funcionando como antes.

**Desta vez, o código completo não será fornecido.** Você receberá as peças (o exemplo do DHT11 e o projeto do Roteiro 6) e deverá juntá-las. As seções abaixo indicam onde cada parte deve ficar e quais cuidados tomar.

## PONTO DE PARTIDA

Antes de começar, confirme:

- [ ] A placa está com a versão **3.0** do Roteiro 6 e a página abre no navegador.
- [ ] Você sabe o IP da placa e a senha OTA configurada.
- [ ] O ambiente `ota` do `platformio.ini` aponta para o IP da **sua** placa.

Se a placa perdeu o OTA, regrave o programa do Roteiro 6 por USB, no ambiente `nodemcuv2`, antes de continuar.

## MATERIAIS

| Quantidade | Componente |
| --- | --- |
| 1x | NodeMCU (ESP8266) com o programa do Roteiro 6 |
| 1x | Sensor DHT11 |
| 3x | Jumpers macho-fêmea |
| 1x | Cabo USB e fonte USB ou power bank para alimentação |

## 1. MONTAGEM DO CIRCUITO

**Desligue a alimentação da placa antes de conectar os fios.**

| Pino do DHT11 | Conexão no NodeMCU |
| --- | --- |
| VCC (+) | 3V3 |
| GND (−) | GND |
| DATA (S / OUT) | D2 |

- Módulos DHT11 com **3 pinos** já possuem o resistor necessário na linha de dados.
- Se o seu sensor tiver **4 pinos** e vier sem placa, ligue um resistor de **10 kΩ** entre DATA e VCC e deixe o terceiro pino desconectado.
- Confira a ordem dos pinos impressa no **seu** módulo: ela varia entre fabricantes.

O LED interno usa o pino **D4**; o sensor usa o **D2**. Não há conflito entre eles.

Depois de conferir a montagem, alimente a placa pela fonte USB ou power bank. Como ela continua com a versão 3.0, ela se reconecta ao Wi-Fi e continua aceitando atualizações OTA.

## 2. AJUSTAR O `platformio.ini`

O DHT11 precisa de duas bibliotecas externas. Acrescente `lib_deps` na seção **`[env]`**, que é compartilhada pelos dois ambientes:

```ini
[platformio]
default_envs = nodemcuv2

[env]
platform = espressif8266
board = nodemcuv2
framework = arduino
monitor_speed = 115200
lib_deps =
    adafruit/DHT sensor library @ ^1.4.6
    adafruit/Adafruit Unified Sensor @ ^1.1.14

; Primeiro envio e recuperação por cabo USB.
[env:nodemcuv2]
upload_protocol = esptool

; Atualizações pela rede.
[env:ota]
upload_protocol = espota
upload_port = 192.168.1.100
upload_flags =
    --auth=SenhaOTA123
```

* **DHT sensor library** → faz a leitura do DHT11/DHT22.
* **Adafruit Unified Sensor** → dependência da biblioteca DHT.

> **Atenção:** se `lib_deps` ficar apenas em `[env:nodemcuv2]`, o ambiente `ota` não encontrará `DHT.h` e a compilação falhará.

Mantenha `monitor_speed = 115200`, a mesma velocidade usada em `Serial.begin()` no Roteiro 6.

## 3. AS PEÇAS QUE VOCÊ VAI JUNTAR

### Peça A — Exemplo do DHT11 (apenas referência)

```cpp
#include "DHT.h"

#define DHTPIN D2
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(9600);
  dht.begin();
  Serial.println("Sensor DHT11 iniciado!");
}

void loop() {
  float temperatura = dht.readTemperature();
  float umidade = dht.readHumidity();

  if (isnan(temperatura) || isnan(umidade)) {
    Serial.println("Erro ao ler o sensor DHT11!");
  } else {
    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.print(" °C | Umidade: ");
    Serial.print(umidade);
    Serial.println(" %");
  }

  delay(2000);
}
```

> **Não envie este exemplo para a placa.** Ele não tem OTA nem servidor web. Se for gravado, a placa deixa de aceitar atualizações pela rede e você precisará do cabo USB para recuperá-la.

### Peça B — O `src/main.cpp` do Roteiro 6

É a base do trabalho. Copie o projeto do Roteiro 6 para uma nova pasta **Roteiro 7** ou continue no mesmo projeto. **Tudo o que existe nele deve continuar funcionando**, em especial `ArduinoOTA.begin()`, `ArduinoOTA.handle()` e `server.handleClient()`.

## 4. MAPA DA INTEGRAÇÃO

Use a tabela para decidir onde cada trecho da Peça A entra na Peça B:

| Trecho da Peça A | Onde colocar na Peça B |
| --- | --- |
| `#include "DHT.h"` | Junto aos outros `#include`, no início do arquivo. |
| `#define DHTPIN`, `#define DHTTYPE` e `DHT dht(...)` | Junto às variáveis globais (`ssid`, `versao`, `server`…). |
| `Serial.begin(9600)` | **Não copie.** O Roteiro 6 já usa `Serial.begin(115200)`. |
| `dht.begin()` | Dentro de `setup()`. |
| Leitura de temperatura e umidade | Dentro de `loop()`, **sem `delay(2000)`** (veja o passo 5). |
| Exibição dos valores | Dentro de `handleRoot()`, montando o HTML da página. |

Altere também:

- `versao` para `"4.0"`;
- o título da página para algo como `Roteiro 7 - DHT11 + OTA`.

## 5. O CUIDADO PRINCIPAL: NÃO USAR `delay(2000)`

O `loop()` do Roteiro 6 precisa executar muitas vezes por segundo:

```cpp
void loop() {
  ArduinoOTA.handle();
  server.handleClient();
}
```

Se você colocar `delay(2000)` ali, a placa passa a maior parte do tempo parada. A página fica lenta, os botões demoram a responder e o envio OTA pode falhar por tempo esgotado.

O DHT11 só deve ser lido a cada **2 segundos**, no mínimo. Para respeitar esse intervalo sem travar o `loop()`, use `millis()`, que informa há quantos milissegundos a placa está ligada:

```cpp
const unsigned long intervaloLeitura = 2000; // ms
unsigned long ultimaLeitura = 0;

void loop() {
  ArduinoOTA.handle();
  server.handleClient();

  if (millis() - ultimaLeitura >= intervaloLeitura) {
    ultimaLeitura = millis();
    // Leia o sensor aqui e guarde os valores em variáveis globais.
  }
}
```

Guarde a última temperatura e a última umidade em **variáveis globais**. Assim, `handleRoot()` apenas exibe os valores guardados, sem precisar ler o sensor a cada acesso à página.

### Dicas para a página

- Inicialize as variáveis globais com `NAN` (por exemplo, `float temperatura = NAN;`). Assim, antes da primeira leitura válida, `isnan()` indica que ainda não há dados.
- Se a leitura falhar (`isnan`), mostre na página uma mensagem como **"Erro ao ler o sensor DHT11"** em vez de `nan`.
- Para acrescentar um número a uma `String`, use `html += String(temperatura, 1);` (uma casa decimal).
- Para a página recarregar sozinha a cada 5 segundos, acrescente dentro do `<head>`:

```cpp
html += "<meta http-equiv='refresh' content='5'>";
```

## 6. ENVIAR POR OTA — A ATUALIZAÇÃO À DISTÂNCIA

1. Salve os arquivos.
2. Compile primeiro: **Project Tasks → ota → General → Build**. Corrija os erros antes de enviar.
3. Envie: **Project Tasks → ota → General → Upload**.

Pelo terminal do PlatformIO:

```powershell
pio run -e ota
pio run -e ota -t upload
```

Ao terminar, aguarde a placa reiniciar e acesse `http://IP_DA_PLACA`.

**Resultado esperado:**

- a página mostra a versão **4.0**;
- temperatura e umidade aparecem e mudam ao segurar o sensor entre os dedos ou soprar sobre ele;
- os botões **Ligar** e **Desligar** continuam funcionando.

Como a placa não está ligada ao computador por cabo de dados, o Monitor Serial não mostra nada. **A página web é a forma de conferir o sensor.**

## 7. SEGUNDA ATUALIZAÇÃO: PROVAR QUE O OTA CONTINUA FUNCIONANDO

Um programa com `delay()` longo pode até receber a versão 4.0, mas falhar na atualização seguinte. Por isso, faça mais um envio pela rede com uma melhoria:

- Altere `versao` para `"4.1"`.
- Mostre um aviso na página quando a temperatura passar de **30 °C**. Por exemplo: `<p style='color:red;'>Temperatura alta!</p>`.

Envie novamente pelo ambiente `ota` e confira a versão **4.1** na página.

## 8. SE ALGO NÃO FUNCIONAR

| Situação | O que verificar |
| --- | --- |
| `DHT.h: No such file or directory` | `lib_deps` deve estar em `[env]` e não apenas em `[env:nodemcuv2]`. |
| A página mostra `nan` ou o erro de leitura | Confira VCC, GND e DATA no pino **D2**. Verifique `DHTTYPE DHT11` e o resistor de 10 kΩ em sensores de 4 pinos. |
| Os valores nunca mudam | Confirme que a leitura em `loop()` atualiza as variáveis **globais**, e não variáveis locais com o mesmo nome. |
| A página e os botões ficaram lentos | Há um `delay()` longo no `loop()`. Substitua-o pela lógica com `millis()`. |
| O OTA da versão 4.1 falhou, mas o da 4.0 funcionou | A versão 4.0 está bloqueando o `loop()` ou deixou de chamar `ArduinoOTA.handle()`. Corrija e regrave por USB no ambiente `nodemcuv2`. |
| Falha de autenticação ou o OTA não conecta | Revise o IP em `upload_port` e se `--auth` corresponde à senha da placa. Veja a tabela do Roteiro 6. |
| A placa perdeu o OTA | Regrave por USB, no ambiente `nodemcuv2`, uma versão com OTA. |

## ATIVIDADE

1. Monte o DHT11 com a placa desligada.
2. Ajuste o `platformio.ini` com as bibliotecas do sensor.
3. Integre o DHT11 ao programa do Roteiro 6 seguindo o mapa da integração.
4. Envie a versão **4.0** por OTA e confira temperatura, umidade e LED na página.
5. Envie a versão **4.1**, com o aviso de temperatura alta, também por OTA.

**Mostre:** o `main.cpp`, o `platformio.ini` sem credenciais reais, capturas da página nas versões 4.0 e 4.1 e do terminal indicando sucesso nos envios OTA. Remova SSID e senhas das capturas antes de compartilhar.

**Responda:**

- Por que o exemplo da Peça A não pode ser enviado diretamente para a placa?
- Por que `delay(2000)` atrapalha o OTA e o servidor web, e como o `millis()` resolve isso?
- Por que as bibliotecas devem ficar na seção `[env]` do `platformio.ini`?
- Por que é melhor guardar a leitura em variáveis globais do que ler o sensor dentro de `handleRoot()`?
- Em que situação real seria útil instalar ou atualizar o programa de um sensor sem acesso físico à placa?

## DESAFIO EXTRA

Crie a rota `/dados`, que responde apenas com os valores em formato JSON:

```json
{"versao":"4.1","temperatura":25.0,"umidade":60.0}
```

Dica: use `server.on("/dados", handleDados);` e `server.send(200, "application/json", json);`. Essa rota será útil para enviar as medições a outros sistemas, como um painel ou um servidor MQTT.