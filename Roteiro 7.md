# ROTEIRO 7 â€” SENSOR DHT11 INSTALADO POR OTA NO NODEMCU (ESP8266)

No Roteiro 6, a placa passou a receber novos programas pela rede Wi-Fi (OTA) e terminou na versÃ£o **3.0**, com os botÃµes **Ligar** e **Desligar** do LED interno. Agora vamos acrescentar um sensor real ao projeto e **instalar o novo programa Ã  distÃ¢ncia**, sem cabo de dados.

## OBJETIVO

Integrar o sensor **DHT11** (temperatura e umidade) ao projeto do Roteiro 6 e enviar o programa por **OTA**. Ao final, a pÃ¡gina da placa deve mostrar:

- a versÃ£o do firmware;
- a temperatura e a umidade medidas pelo DHT11;
- os botÃµes do LED funcionando como antes.

**Desta vez, o cÃ³digo completo nÃ£o serÃ¡ fornecido.** VocÃª receberÃ¡ as peÃ§as (o exemplo do DHT11 e o projeto do Roteiro 6) e deverÃ¡ juntÃ¡-las. As seÃ§Ãµes abaixo indicam onde cada parte deve ficar e quais cuidados tomar.

## PONTO DE PARTIDA

Antes de comeÃ§ar, confirme:

- [ ] A placa estÃ¡ com a versÃ£o **3.0** do Roteiro 6 e a pÃ¡gina abre no navegador.
- [ ] VocÃª sabe o IP da placa e a senha OTA configurada.
- [ ] O ambiente `ota` do `platformio.ini` aponta para o IP da **sua** placa.

Se a placa perdeu o OTA, regrave o programa do Roteiro 6 por USB, no ambiente `nodemcuv2`, antes de continuar.

## MATERIAIS

| Quantidade | Componente |
| --- | --- |
| 1x | NodeMCU (ESP8266) com o programa do Roteiro 6 |
| 1x | Sensor DHT11 |
| 3x | Jumpers macho-fÃªmea |
| 1x | Cabo USB e fonte USB ou power bank para alimentaÃ§Ã£o |

## 1. MONTAGEM DO CIRCUITO

**Desligue a alimentaÃ§Ã£o da placa antes de conectar os fios.**

| Pino do DHT11 | ConexÃ£o no NodeMCU |
| --- | --- |
| VCC (+) | 3V3 |
| GND (âˆ’) | GND |
| DATA (S / OUT) | D2 |

- MÃ³dulos DHT11 com **3 pinos** jÃ¡ possuem o resistor necessÃ¡rio na linha de dados.
- Se o seu sensor tiver **4 pinos** e vier sem placa, ligue um resistor de **10 kÎ©** entre DATA e VCC e deixe o terceiro pino desconectado.
- Confira a ordem dos pinos impressa no **seu** mÃ³dulo: ela varia entre fabricantes.

O LED interno usa o pino **D4**; o sensor usa o **D2**. NÃ£o hÃ¡ conflito entre eles.

Depois de conferir a montagem, alimente a placa pela fonte USB ou power bank. Como ela continua com a versÃ£o 3.0, ela se reconecta ao Wi-Fi e continua aceitando atualizaÃ§Ãµes OTA.

## 2. AJUSTAR O `platformio.ini`

O DHT11 precisa de duas bibliotecas externas. Acrescente `lib_deps` na seÃ§Ã£o **`[env]`**, que Ã© compartilhada pelos dois ambientes:

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

; Primeiro envio e recuperacao por cabo USB.
[env:nodemcuv2]
upload_protocol = esptool

; Atualizacoes pela rede.
[env:ota]
upload_protocol = espota
upload_port = 192.168.1.100
upload_flags =
    --auth=SenhaOTA123
```

* **DHT sensor library** â†’ faz a leitura do DHT11/DHT22.
* **Adafruit Unified Sensor** â†’ dependÃªncia da biblioteca DHT.

> **AtenÃ§Ã£o:** se `lib_deps` ficar apenas em `[env:nodemcuv2]`, o ambiente `ota` nÃ£o encontrarÃ¡ `DHT.h` e a compilaÃ§Ã£o falharÃ¡.

Mantenha `monitor_speed = 115200`, a mesma velocidade usada em `Serial.begin()` no Roteiro 6.

## 3. AS PEÃ‡AS QUE VOCÃŠ VAI JUNTAR

### PeÃ§a A â€” Exemplo do DHT11 (apenas referÃªncia)

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
    Serial.print(" Â°C | Umidade: ");
    Serial.print(umidade);
    Serial.println(" %");
  }

  delay(2000);
}
```

> **NÃ£o envie este exemplo para a placa.** Ele nÃ£o tem OTA nem servidor web. Se for gravado, a placa deixa de aceitar atualizaÃ§Ãµes pela rede e vocÃª precisarÃ¡ do cabo USB para recuperÃ¡-la.

### PeÃ§a B â€” O `src/main.cpp` do Roteiro 6

Ã‰ a base do trabalho. Copie o projeto do Roteiro 6 para uma nova pasta **Roteiro 7** ou continue no mesmo projeto. **Tudo o que existe nele deve continuar funcionando**, em especial `ArduinoOTA.begin()`, `ArduinoOTA.handle()` e `server.handleClient()`.

## 4. MAPA DA INTEGRAÃ‡ÃƒO

Use a tabela para decidir onde cada trecho da PeÃ§a A entra na PeÃ§a B:

| Trecho da PeÃ§a A | Onde colocar na PeÃ§a B |
| --- | --- |
| `#include "DHT.h"` | Junto aos outros `#include`, no inÃ­cio do arquivo. |
| `#define DHTPIN`, `#define DHTTYPE` e `DHT dht(...)` | Junto Ã s variÃ¡veis globais (`ssid`, `versao`, `server`â€¦). |
| `Serial.begin(9600)` | **NÃ£o copie.** O Roteiro 6 jÃ¡ usa `Serial.begin(115200)`. |
| `dht.begin()` | Dentro de `setup()`. |
| Leitura de temperatura e umidade | Dentro de `loop()`, **sem `delay(2000)`** (veja o passo 5). |
| ExibiÃ§Ã£o dos valores | Dentro de `handleRoot()`, montando o HTML da pÃ¡gina. |

Altere tambÃ©m:

- `versao` para `"4.0"`;
- o tÃ­tulo da pÃ¡gina para algo como `Roteiro 7 - DHT11 + OTA`.

## 5. O CUIDADO PRINCIPAL: NÃƒO USAR `delay(2000)`

O `loop()` do Roteiro 6 precisa executar muitas vezes por segundo:

```cpp
void loop() {
  ArduinoOTA.handle();
  server.handleClient();
}
```

Se vocÃª colocar `delay(2000)` ali, a placa passa a maior parte do tempo parada. A pÃ¡gina fica lenta, os botÃµes demoram a responder e o envio OTA pode falhar por tempo esgotado.

O DHT11 sÃ³ deve ser lido a cada **2 segundos**, no mÃ­nimo. Para respeitar esse intervalo sem travar o `loop()`, use `millis()`, que informa hÃ¡ quantos milissegundos a placa estÃ¡ ligada:

```cpp
const unsigned long intervaloLeitura = 2000; // ms
unsigned long ultimaLeitura = 0;

void loop() {
  ArduinoOTA.handle();
  server.handleClient();

  if (millis() - ultimaLeitura >= intervaloLeitura) {
    ultimaLeitura = millis();
    // Leia o sensor aqui e guarde os valores em variÃ¡veis globais.
  }
}
```

Guarde a Ãºltima temperatura e a Ãºltima umidade em **variÃ¡veis globais**. Assim, `handleRoot()` apenas exibe os valores guardados, sem precisar ler o sensor a cada acesso Ã  pÃ¡gina.

### Dicas para a pÃ¡gina

- Inicialize as variÃ¡veis globais com `NAN` (por exemplo, `float temperatura = NAN;`). Assim, antes da primeira leitura vÃ¡lida, `isnan()` indica que ainda nÃ£o hÃ¡ dados.
- Se a leitura falhar (`isnan`), mostre na pÃ¡gina uma mensagem como **"Erro ao ler o sensor DHT11"** em vez de `nan`.
- Para acrescentar um nÃºmero a uma `String`, use `html += String(temperatura, 1);` (uma casa decimal).
- Para a pÃ¡gina recarregar sozinha a cada 5 segundos, acrescente dentro do `<head>`:

```cpp
html += "<meta http-equiv='refresh' content='5'>";
```

## 6. ENVIAR POR OTA â€” A ATUALIZAÃ‡ÃƒO Ã€ DISTÃ‚NCIA

1. Salve os arquivos.
2. Compile primeiro: **Project Tasks â†’ ota â†’ General â†’ Build**. Corrija os erros antes de enviar.
3. Envie: **Project Tasks â†’ ota â†’ General â†’ Upload**.

Pelo terminal do PlatformIO:

```powershell
pio run -e ota
pio run -e ota -t upload
```

Ao terminar, aguarde a placa reiniciar e acesse `http://IP_DA_PLACA`.

**Resultado esperado:**

- a pÃ¡gina mostra a versÃ£o **4.0**;
- temperatura e umidade aparecem e mudam ao segurar o sensor entre os dedos ou soprar sobre ele;
- os botÃµes **Ligar** e **Desligar** continuam funcionando.

Como a placa nÃ£o estÃ¡ ligada ao computador por cabo de dados, o Monitor Serial nÃ£o mostra nada. **A pÃ¡gina web Ã© a forma de conferir o sensor.**

## 7. SEGUNDA ATUALIZAÃ‡ÃƒO: PROVAR QUE O OTA CONTINUA FUNCIONANDO

Um programa com `delay()` longo pode atÃ© receber a versÃ£o 4.0, mas falhar na atualizaÃ§Ã£o seguinte. Por isso, faÃ§a mais um envio pela rede com uma melhoria:

- Altere `versao` para `"4.1"`.
- Mostre um aviso na pÃ¡gina quando a temperatura passar de **30 Â°C**. Por exemplo: `<p style='color:red;'>Temperatura alta!</p>`.

Envie novamente pelo ambiente `ota` e confira a versÃ£o **4.1** na pÃ¡gina.

## 8. SE ALGO NÃƒO FUNCIONAR

| SituaÃ§Ã£o | O que verificar |
| --- | --- |
| `DHT.h: No such file or directory` | `lib_deps` deve estar em `[env]` e nÃ£o apenas em `[env:nodemcuv2]`. |
| A pÃ¡gina mostra `nan` ou o erro de leitura | Confira VCC, GND e DATA no pino **D2**. Verifique `DHTTYPE DHT11` e o resistor de 10 kÎ© em sensores de 4 pinos. |
| Os valores nunca mudam | Confirme que a leitura em `loop()` atualiza as variÃ¡veis **globais**, e nÃ£o variÃ¡veis locais com o mesmo nome. |
| A pÃ¡gina e os botÃµes ficaram lentos | HÃ¡ um `delay()` longo no `loop()`. Substitua-o pela lÃ³gica com `millis()`. |
| O OTA da versÃ£o 4.1 falhou, mas o da 4.0 funcionou | A versÃ£o 4.0 estÃ¡ bloqueando o `loop()` ou deixou de chamar `ArduinoOTA.handle()`. Corrija e regrave por USB no ambiente `nodemcuv2`. |
| Falha de autenticaÃ§Ã£o ou o OTA nÃ£o conecta | Revise o IP em `upload_port` e se `--auth` corresponde Ã  senha da placa. Veja a tabela do Roteiro 6. |
| A placa perdeu o OTA | Regrave por USB, no ambiente `nodemcuv2`, uma versÃ£o com OTA. |

## ATIVIDADE

1. Monte o DHT11 com a placa desligada.
2. Ajuste o `platformio.ini` com as bibliotecas do sensor.
3. Integre o DHT11 ao programa do Roteiro 6 seguindo o mapa da integraÃ§Ã£o.
4. Envie a versÃ£o **4.0** por OTA e confira temperatura, umidade e LED na pÃ¡gina.
5. Envie a versÃ£o **4.1**, com o aviso de temperatura alta, tambÃ©m por OTA.

**Mostre:** o `main.cpp`, o `platformio.ini` sem credenciais reais, capturas da pÃ¡gina nas versÃµes 4.0 e 4.1 e do terminal indicando sucesso nos envios OTA. Remova SSID e senhas das capturas antes de compartilhar.

**Responda:**

- Por que o exemplo da PeÃ§a A nÃ£o pode ser enviado diretamente para a placa?
- Por que `delay(2000)` atrapalha o OTA e o servidor web, e como o `millis()` resolve isso?
- Por que as bibliotecas devem ficar na seÃ§Ã£o `[env]` do `platformio.ini`?
- Por que Ã© melhor guardar a leitura em variÃ¡veis globais do que ler o sensor dentro de `handleRoot()`?
- Em que situaÃ§Ã£o real seria Ãºtil instalar ou atualizar o programa de um sensor sem acesso fÃ­sico Ã  placa?

## DESAFIO EXTRA

Crie a rota `/dados`, que responde apenas com os valores em formato JSON:

```json
{"versao":"4.1","temperatura":25.0,"umidade":60.0}
```

Dica: use `server.on("/dados", handleDados);` e `server.send(200, "application/json", json);`. Essa rota serÃ¡ Ãºtil para enviar as mediÃ§Ãµes a outros sistemas, como um painel ou um servidor MQTT.