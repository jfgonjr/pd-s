[Água que não se perde](#topo)

[Teoria](#teoria)[Protótipo](#prototipo)[Peças 3D](#pecas)[Simulador](#simulador)[Referências](#referencias)

# Como a Internet das Coisas salva milhões de litros de água

Da teoria científica ao protótipo prático de 10 mm: sensores de vazão, válvulas e microcontroladores que trocam a inspeção manual por vigilância contínua.

[Testar o simulador](#simulador) [Ver o protótipo em 3D](#pecas)

Baseado em três artigos científicos e um protótipo open-source. Leitura de cerca de 12 minutos.

Montagem 3D do Controlador de Água 10 mm: corpo de latão com a caixa eletrônica azul-petróleo sobre ele

## O problema: a água que paga imposto de transporte e nunca chega

A água tratada percorre quilômetros de tubos até a torneira. No caminho, parte dela escapa por juntas envelhecidas, obras que danificam a rede e conexões irregulares. Quando essa água sai do sistema sem gerar cobrança, o setor a chama de *água não faturada* (NRW, na sigla em inglês). Purboyo e colegas [\[3\]](#r3) contam que, em discussões com a companhia de água de Bandung (Indonésia), pagamento, furto e vazamento apareceram juntos como ameaça à saúde financeira da empresa.

A escala preocupa. A bibliografia de Ravikiran et al. [\[2\]](#r2) inclui uma reportagem do jornal britânico *The Independent* (2010) cujo título fala em 3,3 bilhões de litros perdidos por dia por vazamentos. E o artigo publicado no GSJ [\[1\]](#r1) resume o ponto central: o problema nem sempre é o tamanho do vazamento, e sim o tempo que se leva para descobri-lo.

Esse é o limite da inspeção manual: alguém precisa ir até lá, olhar e medir. A IoT propõe um ciclo que roda sozinho, 24 horas por dia:

**1. Sentir**Sensores medem vazão, pressão ou umidade.

**2. Decidir**O microcontrolador compara os números com um limite.

**3. Agir**Um atuador corta o fluxo antes que o dano cresça.

**4. Avisar**Uma tela, um app ou um e-mail alertam as pessoas.

## O que dizem os artigos científicos

### Detecção diferencial de vazão: o que entra precisa sair

A ideia de Ravikiran et al. [\[2\]](#r2) cabe numa frase: medir a água na origem, medir de novo no destino e comparar. Os autores colocam um sensor de vazão no tanque que bombeia e outro na casa. Se as leituras coincidem, a tubulação está íntegra. Se a de entrada é maior, a diferença está escapando pelo caminho. O mesmo princípio aparece no trabalho de Rambabu et al. [\[5\]](#r5), com um Arduino Uno que compara os dois sensores e liga um módulo relé.

Vazamento quando Qentrada − Qsaída > limiarExemplo: entram 12 L/min e saem 8,5 L/min. A diferença é 3,5 L/min. Com limiar de 2 L/min, o sistema declara vazamento.

Por que um limiar e não zero? Porque nenhum sensor é perfeito. O artigo da Sensors [\[3\]](#r3) cita trabalhos com sensores de efeito Hall que erraram entre 2,87% e 3,54%. Sem uma margem, o sistema daria alarme falso a cada flutuação. No código de Ravikiran et al., o alerta só dispara quando a diferença passa de 2 unidades de vazão.

O artigo do GSJ [\[1\]](#r1) amplia a receita. Segundo o resumo, o sistema combina sensor de pressão, sensor de vazão e um sensor de água, envia um alerta por e-mail e usa um atuador elétrico para fechar o abastecimento principal até que o responsável intervenha. Em Rambabu et al. [\[5\]](#r5), o relé desliga a bomba quando há vazamento e a religa quando ele termina.

### Retrofit e sensores sem contato

Trocar a tubulação para instalar um medidor novo é caro. Purboyo et al. [\[3\]](#r3) seguem outro caminho: um módulo *retrofit*, que se acopla a um hidrômetro eletromecânico comum sem alterá-lo. Dentro desses hidrômetros há um disco metálico que gira com a passagem da água. Dois sensores indutivo-capacitivos (LC), posicionados a 90° entre si, geram um campo magnético oscilante. Quando o disco passa, ele absorve energia e o sinal enfraquece. Contar esses enfraquecimentos é contar voltas, sem tocar na água e sem peças que se desgastem.

Os números do artigo são concretos. Em 30 repetições de 100 litros, com água entre 25 °C e 30 °C e vazão de 480 a 500 L/h, o erro ficou entre 0% e 2%, com média de 1,33%. A segunda razão para ter dois sensores é evitar leituras falsas pelo efeito de retorno do disco quando a válvula fecha.

No artigo, a válvula solenoide existe para o modelo pré-pago: ela bloqueia a água quando o crédito acaba. A lógica, porém, é a mesma de um corte de emergência: um sensor decide, um atuador fecha. Essa é a ponte entre a teoria e o protótipo que você verá adiante.

### Baixo consumo: anos de bateria com o *deep sleep*

Um medidor longe da tomada depende de bateria. O dispositivo de Purboyo et al. [\[3\]](#r3) usa o microcontrolador STM32L431 e uma bateria de lítio de 3,6 V e 8.500 mAh, dos quais cerca de 6.800 mAh são aproveitáveis. A meta era durar 6 anos (5 exigidos pela regulação local, mais uma folga). Isso permite uma corrente média de no máximo 129 µA.

O truque é dormir quase o tempo todo. No ciclo medido, o aparelho fica 47,97 s em repouso profundo (28,79 µA), liga o Bluetooth por 1,5 s (9.100 µA) e aciona a válvula por 13,2 s (30.290 µA). As atividades caras são raras. Mexa no controle abaixo e veja a conta do artigo funcionar.

Repetições de sono profundo por ciclo 95

Corrente média: µA Autonomia: anos

O artigo conclui que 95 repetições (cerca de 76 minutos de sono entre as atividades) fazem a média encostar nos 129 µA. A conectividade também economiza: o medidor fala por Bluetooth com o celular do usuário, e o celular usa a rede móvel dele para falar com o servidor. Não há gateway nem torre dedicada. A contrapartida, admitida pelos autores, é que os dados não chegam em tempo real.

Os cálculos do simulador de bateria seguem a Equação 2 do artigo, com autodescarga de 10 µA, como na Equação 3.

## Da teoria à bancada: o Controlador de Fluido de 10 mm

O repositório *Controlador de Água 10 mm* [\[4\]](#r4) reúne CAD, firmware e documentação de uma válvula para tubo de 10 mm com **corte automático por vazamento**. Ele mostra, em tamanho de mão (88 × 50 × 62 mm), como as ideias dos artigos viram peças e código.

### Como a válvula funciona

É uma válvula borboleta rotativa. Um disco de 32 mm gira 90° dentro do corpo: a 0° ele bloqueia o fluxo, a 90° fica alinhado e deixa passar. Não há fim de curso. Um ímã na ponta do eixo fica sob um encoder magnético AS5600, que informa o ângulo em tempo real. Se o motor é acionado e o ângulo não muda, o firmware entende que algo travou. Um motor DC com redutor (ou um servo) gira o disco, com alimentação de 12 V e compatibilidade com Arduino e ESP32.

### A lógica de segurança

Ao ligar, a válvula sempre fecha primeiro. Com a válvula aberta, um vazamento confirmado (sensor estável por 2 s) inicia um **pré-alarme de 60 s**, com LED e buzzer intermitentes. Se o vazamento cessa nesse intervalo, a contagem é cancelada. Se persiste, a válvula fecha e fica **bloqueada**, estado gravado na EEPROM, que sobrevive a queda de energia. O rearme exige segurar o botão por 3 s, e só vale se o sensor não acusar mais vazamento.

### O que vem da literatura e o que o protótipo faz hoje

| Capacidade | Na literatura | No repositório |
| --- | --- | --- |
| Detecção | Vazão diferencial entre dois sensores [\[2\]](#r2)[\[5\]](#r5); pressão e sensor de água [\[1\]](#r1) | Sonda de condução e sensor externo de alagamento. Há entrada para sensor de vazão YF-S201 (450 pulsos/L), opcional e desligada por padrão, que detecta fluxo contínuo por 30 min. A comparação entre dois sensores não está implementada. |
| Corte automático | Relé que desliga a bomba [\[5\]](#r5); atuador no abastecimento [\[1\]](#r1); solenoide [\[3\]](#r3) | Válvula borboleta motorizada, em malha fechada pelo encoder. O motor com redutor mantém a posição sem consumir energia. |
| Alerta ao usuário | Aplicativo ou e-mail [\[1\]](#r1)[\[2\]](#r2)[\[5\]](#r5) | LED e buzzer locais. O ESP32 é compatível, mas o envio de alertas pela rede ainda não existe no firmware. |
| Energia | Deep sleep e comunicação enxuta [\[3\]](#r3) | Alimentação por fonte de 12 V. Consumo em repouso não documentado. |

Essa tabela é a parte mais honesta do post: o protótipo implementa bem o *agir* (corte seguro, travamento, rearme) e abre caminho para o *sentir* e o *avisar* da literatura. Como alerta o próprio projeto, ele é um protótipo: deve ser testado em bancada e nunca ser a única proteção contra alagamento.

Como seria ligar as duas pontas? Um segundo sensor YF-S201 a jusante e um trecho de código como o abaixo, que é **ilustrativo** e não faz parte do repositório, trariam a vazão diferencial de Ravikiran et al. [\[2\]](#r2) para o controlador:

```
// ESP32: dois sensores de vazão (pulsos por interrupção)
volatile uint32_t pulsosEntrada = 0, pulsosSaida = 0;
const float PULSOS_POR_LITRO = 450.0;  // valor do YF-S201 no repositório
const float LIMIAR_LPM = 2.0;          // margem para erro dos sensores

void IRAM_ATTR contaEntrada() { pulsosEntrada++; }
void IRAM_ATTR contaSaida()   { pulsosSaida++;   }

void verificaVazamento() {            // chamada a cada 1 s
  noInterrupts();
  uint32_t pe = pulsosEntrada, ps = pulsosSaida;
  pulsosEntrada = pulsosSaida = 0;
  interrupts();

  float qEntrada = pe * 60.0 / PULSOS_POR_LITRO;  // L/min
  float qSaida   = ps * 60.0 / PULSOS_POR_LITRO;

  if (qEntrada - qSaida > LIMIAR_LPM) {
    iniciaPreAlarme();   // reaproveita os 60 s e o bloqueio do firmware
  }
}
```

## O protótipo em 3D

As imagens abaixo foram geradas a partir dos arquivos STL do projeto, o mesmo formato usado para imprimir as peças. As sete partes encaixam em torno de um único eixo vertical, de baixo para cima.

Montagem completa vista em perspectiva

**Montagem completa.** Tubo de Ø10 mm entra pelo corpo; a eletrônica fica protegida na caixa.

Vista explodida com as sete peças empilhadas

**Vista explodida.** De baixo para cima: obturador, corpo, eixo, caixa eletrônica, placa de sensores, junta e tampa.

Corpo de latão da válvula

**Corpo** 88 × 50 × 36 mm. Canal Ø10 mm, câmara Ø32 mm para o disco e furos 4× M4.

Disco obturador

**Obturador** Ø32 × 3 mm. O disco que gira 90° para abrir ou fechar a passagem.

Eixo vertical

**Eixo** Ø6 × 43 mm. Transmite a rotação; a ponta superior leva o ímã lido pelo AS5600.

Caixa eletrônica

**Caixa eletrônica** 68 × 48 × 22 mm. Três passagens Ø7 mm: 12 V, sonda de vazamento e comando ou fluxo.

Placa de sensores

**Placa de sensores** 54 × 34 × 3 mm. O encoder AS5600 fica centrado sobre a ponta do eixo.

Junta de vedação

**Junta** 62 × 42 × 1 mm. Veda a caixa contra umidade.

Tampa da caixa

**Tampa** 68 × 48 × 6 mm. Fecha a caixa, com 4 furos de fixação.

## Simulador: encontre o vazamento

Ajuste a vazão medida na entrada e na saída do trecho monitorado. Se a diferença passar do limiar, o sistema reage. Escolha entre o corte imediato dos artigos [\[2\]](#r2)[\[5\]](#r5) ou o pré-alarme de 60 s do protótipo [\[4\]](#r4).

Sensor 1 (entrada)

Sensor 2 (saída)

Limiar de tolerância

Modo de resposta

Sem vazamento

Diferença\
**0.0** L/min

[ ] Acelerar tempo (10×)

Dica: mantenha S1 maior que S2 por mais que o limiar e observe a válvula girar. Depois de fechada, a válvula só rearma quando a diferença volta ao normal, como no firmware [\[4\]](#r4).

## Pesquisa acadêmica e cultura maker

Os três artigos respondem a perguntas diferentes: *como saber* que há vazamento (E3S e GSJ), *como medir sem desgastar* a infraestrutura existente e *como durar anos* sem tomada (Sensors). O repositório mostra a etapa seguinte: transformar essas respostas em peças que qualquer pessoa pode baixar, imprimir, modificar e testar. O CAD e o firmware abertos reduzem a distância entre ler um artigo e ter algo funcionando na bancada.

O impacto socioambiental é direto. Cada vazamento detectado cedo é água tratada, energia de bombeamento e dinheiro que não se perdem, e menos dano a casas e ruas. A tecnologia sozinha não resolve o problema, e um protótipo não substitui a manutenção da rede. Mas ela torna o vazamento visível, e é isso que muda o tempo de resposta.

## Referências

1. DEVELOPMENT and Optimization of an IoT-Based Smart Water Leakage Detection System in Water Pipelines with Real-Time Response Capability. *Global Scientific Journal (GSJ)*, v. 12, n. 11, nov. 2024. ISSN 2320-9186.
2. RAVIKIRAN, K.; TUKARAM, S.; HEMASAIKIRAN, K.; AFRIDHI; ABHILASH, P. K.; MAARFI, F. Creating a Sustainable Smart Water Leakage Detection System using IoT. *E3S Web of Conferences*, v. 430, art. 01093, 2023. ICMPC 2023. DOI: [10.1051/e3sconf/202343001093](https://doi.org/10.1051/e3sconf/202343001093).
3. PURBOYO, A. K.; FAKHRURROJA, H.; PRAMESTI, D.; CHAIDIR, A. R. Mobile Application Development for Prepaid Water Meter Based on LC Sensor. *Sensors*, v. 24, n. 20, art. 6762, 2024. DOI: [10.3390/s24206762](https://doi.org/10.3390/s24206762).
4. JFGONJR. *Controlador de Fluido 10mm*: válvula borboleta rotativa para tubo de Ø10 mm com corte automático por vazamento. GitHub, \[s.d.\]. Disponível em: [github.com/jfgonjr/controlador-de-fluido-10mm](https://github.com/jfgonjr/controlador-de-fluido-10mm/). Acesso em: 8 out. 2026.
5. RAMBABU, K.; DUBEY, S.; SRINIVAS, K. N.; REDDY, D. N.; ROHITH, C. Smart Water Flow and Pipeline Leakage Detection using IoT and Arduino UNO. In: INTERNATIONAL CONFERENCE ON INTELLIGENT DATA COMMUNICATION TECHNOLOGIES AND INTERNET OF THINGS (IDCIoT), 2., 2024. *Proceedings*... IEEE, 2024. p. 174-180. DOI: [10.1109/IDCIOT59759.2024.10467990](https://doi.org/10.1109/IDCIOT59759.2024.10467990).

Imagens 3D renderizadas a partir dos arquivos STL do projeto. O artigo de E3S Web of Conferences e o da Sensors são de acesso aberto sob licença CC BY 4.0; este texto os parafraseia.