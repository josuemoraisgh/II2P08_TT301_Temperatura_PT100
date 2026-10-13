# Instrumentação Industrial II
## Aula Prática 08: Medição de Temperatura com Pt100 e Transmissor TT301 — Configuração e Calibração

Esta prática apresenta a base teórica e a execução, no simulador **LasecSimul**, da medição de temperatura de um tanque aquecido com uma termorresistência **Pt100** ligada a **4 fios** ao transmissor **Smar TT301 (HART)**, configurado e calibrado pelo **PACTware**.

O material segue a ordem de trabalho de uma planta real, com foco em:

- funcionamento do Pt100 e a curva de Callendar–Van Dusen da **IEC 60751**;
- ligação a 2, 3 e 4 fios e o erro causado pela resistência dos cabos;
- configuração do TT301: sensor, ligação, faixa, unidade, amortecimento, burnout e display;
- calibração do **transmissor** com simulador de resistência e do **sensor** com bloco seco e termômetro padrão;
- **trim de leitura** (zero e ganho), **trim de corrente** e calibração com referência;
- teste de malha, teste de burnout, teste como deixado e registro de calibração.

---

## Índice

1. [Introdução](#1-introdução)
2. [A termorresistência Pt100](#2-a-termorresistência-pt100)
3. [Ligação a 2, 3 e 4 fios](#3-ligação-a-2-3-e-4-fios)
4. [O transmissor TT301](#4-o-transmissor-tt301)
5. [A planta no LasecSimul](#5-a-planta-no-lasecsimul)
6. [Arquivos da prática](#6-arquivos-da-prática)
7. [Como executar](#7-como-executar)
8. [Conexão com o PACTware](#8-conexão-com-o-pactware)
9. [Configuração do TT301](#9-configuração-do-tt301)
10. [Calibração](#10-calibração)
11. [Bancada virtual no LasecSimul](#11-bancada-virtual-no-lasecsimul)
12. [Sugestão de atividade experimental](#12-sugestão-de-atividade-experimental)
13. [Conclusão](#13-conclusão)
14. [Referências](#14-referências)
15. [Apêndice: lista de imagens](#15-apêndice-lista-de-imagens)

---

## 1. Introdução

A temperatura é a variável mais medida na indústria. Entre os sensores de contato, a **termorresistência de platina (Pt100)** é a preferida quando se precisa de exatidão e estabilidade entre −200 °C e 850 °C. Ela é usada em:

- tanques, reatores e trocadores de calor;
- linhas de vapor, óleo térmico e água gelada;
- processos alimentícios e farmacêuticos;
- referência de laboratórios de calibração.

O Pt100 não gera sinal: ele muda de resistência com a temperatura. Quem mede essa resistência, lineariza a curva e transmite a temperatura em 4–20 mA com HART é o **transmissor de temperatura**, aqui o **TT301** da Smar, normalmente montado no cabeçote do sensor ou perto dele.

**Figura 1. Termorresistência industrial com poço e cabeçote**

![Figura 1](assets/fig01_pt100_industrial.jpg)

> **Imagem a inserir:** foto de uma termorresistência Pt100 industrial completa: bainha metálica, poço termométrico com rosca ou flange, cabeçote de ligação aberto mostrando os 4 fios (dois vermelhos e dois brancos) e, se possível, um TT301 montado ao lado ou no cabeçote.

---

## 2. A termorresistência Pt100

### 2.1 Princípio

A resistência elétrica da platina aumenta com a temperatura de forma muito repetitiva. O **Pt100** tem **100 Ω a 0 °C** (R0 = 100 Ω). Perto da temperatura ambiente, a sensibilidade é de cerca de **0,385 Ω/°C**.

### 2.2 Equação de Callendar–Van Dusen (IEC 60751)

Para t ≥ 0 °C:

$$
R(t) = R_0\left(1 + A\,t + B\,t^2\right)
$$

Para t < 0 °C, soma-se o termo cúbico:

$$
R(t) = R_0\left[1 + A\,t + B\,t^2 + C\,(t-100)\,t^3\right]
$$

Com os coeficientes da IEC 60751:

- `A = 3,9083 × 10⁻³ °C⁻¹`;
- `B = −5,775 × 10⁻⁷ °C⁻²`;
- `C = −4,183 × 10⁻¹² °C⁻⁴`.

O coeficiente médio entre 0 e 100 °C é:

$$
\alpha = \frac{R_{100} - R_0}{100\,R_0} = 0{,}00385055\ \text{°C}^{-1}
$$

Para t ≥ 0 °C, a temperatura pode ser obtida da resistência invertendo a equação:

$$
t = \frac{-A + \sqrt{A^2 - 4B\left(1 - \dfrac{R}{R_0}\right)}}{2B}
$$

| t (°C) | 0 | 20 | 25 | 50 | 75 | 80 | 100 |
|---|---:|---:|---:|---:|---:|---:|---:|
| R (Ω) | 100,000 | 107,794 | 109,735 | 119,397 | 128,987 | 130,897 | 138,506 |

### 2.3 Classes de tolerância

A IEC 60751 define a tolerância do elemento em função da temperatura `t` (em °C):

| Classe | Tolerância | A 0 °C | A 50 °C | A 100 °C |
|---|---|---:|---:|---:|
| AA | ±(0,10 + 0,0017·\|t\|) °C | ±0,10 | ±0,18 | ±0,27 |
| A | ±(0,15 + 0,002·\|t\|) °C | ±0,15 | ±0,25 | ±0,35 |
| B | ±(0,30 + 0,005·\|t\|) °C | ±0,30 | ±0,55 | ±0,80 |

Um Pt100 classe B pode errar até 0,8 °C a 100 °C sem estar defeituoso. Quando se precisa de menos que isso, o sensor é **calibrado** e os coeficientes dele são gravados no transmissor (seção 10.7).

### 2.4 Autoaquecimento e tempo de resposta

- A corrente de medição aquece o elemento (**autoaquecimento**). Por isso os transmissores usam correntes baixas, de fração de mA.
- O **poço termométrico** protege o sensor, mas atrasa a medida: a resposta é aproximadamente de primeira ordem, com constante de tempo de segundos a minutos.

**Figura 2. Curva resistência × temperatura do Pt100**

![Figura 2](assets/fig02_curva_pt100.png)

> **Imagem a inserir:** gráfico da resistência do Pt100 (eixo vertical, Ω) em função da temperatura (eixo horizontal, de −200 a 850 °C), calculado pela equação de Callendar–Van Dusen. Destaque com pontos os valores da tabela da seção 2.2 e, num detalhe ampliado de 0 a 100 °C, a reta de inclinação 0,385 Ω/°C.

---

## 3. Ligação a 2, 3 e 4 fios

O transmissor injeta uma corrente `I` no sensor e mede a tensão. A resistência de cada fio do cabo (`R_f`) pode entrar ou não na medida, conforme a ligação. Os bornes do TT301 são numerados de 1 a 4.

### 3.1 Dois fios

Bornes 1 e 2 juntos (ponte) e 3 e 4 juntos. A corrente passa pelos dois fios, e a tensão medida inclui a queda neles:

$$
V = (R_{Pt100} + 2R_f)\,I
$$

O erro de temperatura é aproximadamente:

$$
\Delta t \approx \frac{2R_f}{0{,}385\ \Omega/\text{°C}}
$$

Com 0,5 Ω por fio, o erro é de **+2,6 °C**; com 2 Ω por fio, de **+10,3 °C**.

### 3.2 Três fios

Bornes 1 e 2 juntos. Uma extremidade do sensor vai ao borne 2 (com o 1), e a outra vai, por dois fios, aos bornes 3 e 4. O borne 3 não leva corrente. O transmissor mede `V2` (entre 2 e 3) e `V1` (entre 3 e 4) e subtrai:

$$
V_2 - V_1 = (R_{Pt100} + R_f)\,I - R_f\,I = R_{Pt100}\,I
$$

A compensação só é perfeita se os fios tiverem **a mesma resistência**: mesmo material, bitola e comprimento.

### 3.3 Quatro fios

A corrente sai pelo borne 1 e volta pelo 4. Os bornes 2 e 3 têm alta impedância e só medem a tensão no próprio elemento:

$$
V = R_{Pt100}\,I
$$

A resistência dos fios não influi. É a ligação de maior exatidão.

### 3.4 Código de cores

Pela IEC 60751, os fios de uma extremidade do elemento são **vermelhos** e os da outra são **brancos**. Em um Pt100 a 4 fios: vermelho e vermelho em uma ponta, branco e branco na outra.

**Figura 3. Ligações do sensor nos bornes do TT301**

![Figura 3](assets/fig03_ligacoes_2_3_4_fios.png)

> **Imagem a inserir:** desenho dos bornes 1, 2, 3 e 4 do TT301 com as três ligações lado a lado (baseado na Figura 1.9 do manual): 2 fios com pontes 1–2 e 3–4; 3 fios com ponte 1–2; 4 fios com um fio em cada borne. Pinte os fios nas cores da IEC 60751 (vermelho e branco).

**Figura 4. Por que a ligação a 4 fios elimina os cabos**

![Figura 4](assets/fig04_esquema_4_fios.png)

> **Imagem a inserir:** esquema elétrico baseado na Figura 2.5 do manual: fonte de corrente `I` saindo pelo borne 1, as quatro resistências de fio `R`, o elemento RTD e o voltímetro de alta impedância entre os bornes 2 e 3. Marque com setas o caminho da corrente e indique que nos fios 2 e 3 não passa corrente.

---

## 4. O transmissor TT301

### 4.1 Bornes

- **+ e −**: alimentação e saída 4–20 mA (12 a 45 Vcc, a dois fios).
- **1 a 4**: sensor. O manual adverte: nunca ligue a alimentação nesses bornes.
- **TEST**: para medir a corrente sem abrir a malha.
- **COMM**: para ligar o configurador HART.

### 4.2 Sensores aceitos

- RTD: Pt50, Pt100, Pt500 e Pt1000 (IEC), Pt100 (JIS, GOST, MIL), Ni120 e Cu10, entre outros;
- termopares B, E, J, K, N, R, S, T, L e U, com compensação de junta fria;
- resistência (Ω) e tensão (mV);
- ligações a 2, 3 e 4 fios, diferencial, backup, média, máximo e mínimo.

### 4.3 Cadeia de sinal

```text
bornes 1-4 → entrada (Ω) → trim de leitura → linearização do sensor → faixa LRV/URV → % → saída → trim de corrente → 4-20 mA
```

- O **trim de leitura** (zero e ganho) corrige a medida de resistência em relação a um padrão.
- A **faixa** (LRV/URV) só define que temperatura corresponde a 4 e a 20 mA.
- O **trim de corrente** corrige o conversor da saída.

### 4.4 Burnout

Se o sensor romper ou sair da faixa, a saída vai ao **burnout**: **21 mA (High)** ou **3,6 mA (Low)**, conforme a configuração. O display indica a falha.

**Figura 5. Bornes do TT301**

![Figura 5](assets/fig05_tt301_bornes.jpg)

> **Imagem a inserir:** foto do TT301 com a tampa do lado dos terminais aberta, mostrando os bornes de alimentação (+ e −), os bornes do sensor numerados de 1 a 4 e os terminais TEST e COMM. Identifique cada grupo com setas.

**Figura 6. Cadeia de sinal do TT301**

![Figura 6](assets/fig06_cadeia_sinal_tt301.png)

> **Imagem a inserir:** diagrama de blocos simplificado (baseado na Figura 2.2 do manual) com: entrada, trim de leitura, linearização do sensor, calibração (LRV/URV), saída e trim de corrente. Marque onde agem o trim de leitura, a faixa e o trim de corrente, e mostre a saída digital HART saindo depois da linearização.

---

## 5. A planta no LasecSimul

O projeto [`tanque_aquecido_tt301.lsproj`](tanque_aquecido_tt301.lsproj) reproduz um tanque de água com uma resistência de aquecimento e um Pt100 em poço termométrico, ligado por um cabo de 4 fios aos bornes 1 a 4 do TT301. A malha 4–20 mA tem fonte de 24 V, resistor de 250 Ω, amperímetro, modem HART e osciloscópio. A planta vem **incorporada ao projeto**. É preciso o LasecSimul **0.0.84 ou mais novo**: o TT301 a 4 fios foi introduzido nessa versão.

**Figura 7. Projeto completo no LasecSimul**

![Figura 7](assets/fig07_projeto_completo.png)

> **Imagem a inserir:** captura do LasecSimul com o projeto `tanque_aquecido_tt301.lsproj` rodando (selo "Running"): o slider "Aquecimento (%)", a planta do tanque aquecido, os quatro fios (vermelho, vermelho, branco, branco) chegando aos bornes 1–4 do TT301, a fonte de 24 V, o resistor de 250 Ω, o amperímetro, o modem HART e o osciloscópio. Use o botão "ajustar tudo à tela".

### 5.1 O que a planta calcula

- **Balanço de energia:** `m·c·dT/dt = P − UA·(T − T_amb)`, com `P = P_max·u/100`. A 100 °C a água ferve e a temperatura para de subir.
- **Poço termométrico:** `dT_s/dt = (T − T_s)/τ`.
- **Pt100:** `R = R0·(1 + A·t + B·t²)` com `t = T_s + desvio do sensor`.
- **Elemento e fios do cabo:** são resistências de verdade no circuito. O TT301 mede a resistência pelos bornes, como o aparelho real.

### 5.2 Parâmetros (Propriedades exportadas)

| Parâmetro | Padrão |
|---|---|
| Volume de água no tanque | 10 L |
| Potência máxima do aquecedor (100 %) | 6 kW |
| Coeficiente de perdas para o ambiente UA | 80 W/K |
| Temperatura ambiente | 25 °C |
| Constante de tempo do poço termométrico | 15 s |
| R0 do sensor | 100 Ω |
| Resistência de cada fio do cabo | 0,5 Ω |
| Desvio do sensor (erro do Pt100) | 0 °C |
| Sensor rompido | Não |

### 5.3 Valores esperados

- Constante de tempo térmica: `m·c/UA = 523 s ≈ 8,7 min`.
- Temperatura de regime: `T = T_amb + P/UA`.

| Aquecimento | Potência | Temperatura de regime |
|---:|---:|---:|
| 25 % | 1,5 kW | 43,8 °C |
| 50 % | 3,0 kW | 62,5 °C |
| 75 % | 4,5 kW | 81,3 °C |
| 100 % | 6,0 kW | 100 °C (ferve) |

Com 100 %, a água vai de 25 °C a 80 °C em cerca de **11,5 min** de simulação.

**Figura 8. Detalhe da planta**

![Figura 8](assets/fig08_planta.png)

> **Imagem a inserir:** captura ampliada só da planta, com aquecimento em 50 % há alguns minutos: tanque com água, serpentina de aquecimento no fundo, lâmpada "Aquecendo" acesa, poço termométrico (TE-101) com o Pt100 na ponta, os quatro fios coloridos, o mostrador radial TI-101, a barra de temperatura e os displays (temperatura da água, temperatura no sensor, resistência do Pt100, potência, perdas e constante de tempo).

**Figura 9. Propriedades exportadas da planta**

![Figura 9](assets/fig09_propriedades_exportadas.png)

> **Imagem a inserir:** painel **Properties** com a planta selecionada, rolado até "Propriedades exportadas", mostrando os campos de volume, potência, UA, temperatura ambiente, τ do poço, R0, resistência de cada fio, desvio do sensor e a caixa "Sensor rompido".

---

## 6. Arquivos da prática

- [`tanque_aquecido_tt301.lsproj`](tanque_aquecido_tt301.lsproj): projeto do LasecSimul com a planta incorporada e o TT301;
- [`assets/`](assets): imagens deste roteiro.

---

## 7. Como executar

1. Instale ou atualize a extensão **LasecSimul** no VS Code (0.0.84 ou mais nova).
2. Abra esta pasta no VS Code e dê dois cliques em `tanque_aquecido_tt301.lsproj`.
3. Clique em **Run** (▶).
4. Selecione o slider **"Aquecimento (%)"** e altere o **Valor do sinal** no painel Properties.
5. Acompanhe a temperatura da água e a do sensor, a corrente no amperímetro e o display do TT301.
6. Para mudar a instalação (fios, desvio, sensor rompido), selecione a planta e use **Propriedades exportadas**.

Observações importantes:

- a planta é lenta como uma real (constante de tempo de 8,7 min); para testes rápidos, reduza o volume de água;
- o TT301 vem com a faixa de fábrica (0 a 800 °C): a corrente quase não muda até a faixa ser configurada (seção 9);
- o poço atrasa o sensor em relação à água: espere estabilizar antes de comparar valores.

---

## 8. Conexão com o PACTware

1. Selecione o **Modem HART** e confira em Properties: **Ligação com o PC** = Serial (COM / CNC) e **Porta serial** = `CNCB0`.
2. No **com0com**, confira o par `CNCB0` ↔ `COM20`.
3. No PACTware, em **Ver → Catálogo de instrumentos**, atualize o catálogo e adicione **HART Communication**.
4. Configure o driver HART na porta **COM20** e desmarque **Multimaster and Burst mode support**.
5. Clique com o botão direito no driver → **Adicionar instrumento** → **TT301** (Smar).
6. Com a simulação rodando, conecte o TT301 (ícones verdes) e abra **OnLine Parameterize**.

O DTM do TT301 organiza as telas em **OnLine Parameterize → Blocks**: **Identification**, **Configuration** (com **Sensor** e **PID**), **Calibration** e **General**. Há ainda **Observe** (monitoração) e **Diagnosis**.

**Figura 10. TT301 conectado no PACTware**

![Figura 10](assets/fig10_pactware_conectado.png)

> **Imagem a inserir:** tela do PACTware com a árvore do projeto (HART Communication em COM20 → TT301) com ícones verdes e a tela **Identification** aberta à direita: Manufacturer SMAR, Device Type TT301, Tag, Descriptor e Message.

---

## 9. Configuração do TT301

Faça uma alteração por vez, aplique (**Write**) e confira no simulador.

### 9.1 Identificação

Em **Identification**: **Tag** `TT-101` (8 caracteres), **Descriptor** `TEMP TANQUE` (16) e **Message** com a data e o responsável (32).

### 9.2 Sensor e ligação

Em **Configuration → Sensor**: **Sensor Type = RTD**, **Sensor = Pt 100 IEC** e **Connection = Four wires**.

**Figura 11. Configuração do sensor**

![Figura 11](assets/fig11_dtm_sensor.png)

> **Imagem a inserir:** tela **Configuration → Sensor** do DTM com Sensor Type RTD, Sensor Pt 100 IEC e Connection Four wires selecionados, antes de aplicar.

### 9.3 Faixa, unidade e amortecimento

Em **Calibration**: **PV Unit = degC**, **LRV = 0**, **URV = 100** e **Damping = 2 s**. Confira os limites somente de leitura: **LRL = −200 °C**, **URL = 850 °C** e **Min Span = 10 °C** (Pt100 IEC).

Com a faixa de 0 a 100 °C, 25 °C correspondem a 8 mA.

**Figura 12. Faixa e amortecimento**

![Figura 12](assets/fig12_dtm_calibration.png)

> **Imagem a inserir:** tela **Calibration** do DTM com PV Unit degC, URL 850, URV 100, LRV 0, LRL −200, Min Span 10 e Damping 2, e o quadro "To Applied Input" com os botões LO− e HI+ visível.

### 9.4 Burnout, proteção de escrita e display

Em **Configuration**:

- **Burnout = High** (21 mA) ou **Low** (3,6 mA): escolha o lado que leva o processo à condição segura. Num aquecimento, temperatura alta deve desligar o aquecedor, então **High** é a escolha comum.
- **W.Protect** = read/write durante a configuração.
- **LCD: First Variable = Temp. PV** e **Second Variable = PV%** (ou corrente).

**Figura 13. Configuração geral e display**

![Figura 13](assets/fig13_dtm_configuration.png)

> **Imagem a inserir:** tela **Configuration** do DTM com W.Protect read/write, Burnout High, Function Linear e o quadro LCD com First Variable e Second Variable.

**Figura 14. Display do TT301**

![Figura 14](assets/fig14_lcd_tt301.png)

> **Imagem a inserir:** duas capturas ampliadas do display do TT301 no simulador, com a água estabilizada (por exemplo, a 62,5 °C com aquecimento em 50 %): uma com a temperatura em °C e outra, 3 s depois, com a segunda variável. Monte lado a lado.

---

## 10. Calibração

### 10.1 O que se calibra

Em uma malha de temperatura há dois instrumentos com erros diferentes:

- o **sensor** (Pt100): erro de tolerância da classe, deriva com o uso, choque térmico;
- o **transmissor** (TT301): erro na medida de resistência e no conversor de saída.

O erro do sensor costuma ser a maior parcela. Por isso os dois são calibrados **separadamente**:

- o transmissor com um **simulador de resistência** (década ou calibrador de processo) no lugar do sensor;
- o sensor com uma **fonte de temperatura** (bloco seco ou banho) e um **termômetro padrão**.

Uma verificação da **malha completa** (sensor + transmissor + indicação no sistema de controle) fecha o trabalho.

### 10.2 Teste como encontrado

Antes de qualquer ajuste, registre o estado do instrumento:

1. Salve no PACTware a configuração como encontrada.
2. Anote tag, sensor, ligação, faixa, amortecimento e burnout.
3. Faça o teste do transmissor (seção 10.3) sem mudar nada.

Critério de aceitação de exemplo: **±0,2 % do span** para o transmissor (±0,2 °C numa faixa de 0 a 100 °C).

### 10.3 Calibração do transmissor com simulador de resistência

1. Desligue o Pt100 dos bornes 1–4 e ligue no lugar dele a **década** ou o **calibrador de processo** em modo resistência (ou simulação de RTD Pt100), na **mesma ligação configurada** (4 fios).
2. Ligue o multímetro de precisão em série na malha (ou nos terminais TEST).
3. Aplique as resistências da tabela, subindo e depois descendo, e espere estabilizar em cada ponto.
4. Anote a temperatura lida no PACTware (PV) e a corrente.

| Ponto | t (°C) | R aplicada (Ω) | I esperada (mA) | PV lida ↑ | I lida ↑ | PV lida ↓ | I lida ↓ |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 % | 0 | 100,000 | 4,000 | | | | |
| 25 % | 25 | 109,735 | 8,000 | | | | |
| 50 % | 50 | 119,397 | 12,000 | | | | |
| 75 % | 75 | 128,987 | 16,000 | | | | |
| 100 % | 100 | 138,506 | 20,000 | | | | |

Erro da leitura digital e da saída em % do span:

$$
e_{PV}(\%) = \frac{PV - t}{URV - LRV}\cdot 100
\qquad
e_{mA}(\%) = \frac{I - I_{esp}}{16}\cdot 100
$$

**Figura 15. Simulador de resistência nos bornes do TT301**

![Figura 15](assets/fig15_simulador_resistencia.jpg)

> **Imagem a inserir:** foto da bancada com a década de resistência (ou o calibrador de processo em modo RTD) ligada a 4 fios nos bornes 1–4 do TT301, o multímetro em série na malha e o PACTware aberto no notebook mostrando a PV. O valor da década deve estar legível (por exemplo, 119,397 Ω = 50 °C).

### 10.4 Trim de leitura (zero e ganho)

Se a PV não confere com a resistência aplicada, ajuste a medida de resistência:

1. Aplique a resistência do **ponto inferior** (100,000 Ω) e, em **Calibration → Trim**, faça o **Zero Trim** informando o valor aplicado.
2. Aplique a resistência do **ponto superior** (138,506 Ω) e faça o **Gain Trim**.
3. Confira de novo os cinco pontos.
4. Se algo der errado, o **Factory Trim** restaura o ajuste de fábrica.

O manual pede um padrão de resistência ou mV com **exatidão melhor que 0,02 %**.

### 10.5 Trim de corrente

Se a PV confere mas a corrente não, ajuste a saída:

1. Coloque a malha de controle em **manual**: durante o trim a corrente não representa o processo.
2. Em **Trim → Current Trim**, o TT301 coloca a saída em 4 mA. Leia o multímetro e informe o valor lido.
3. Repita para 20 mA.

### 10.6 Calibração com referência (LO− / HI+)

No quadro **To Applied Input** da tela **Calibration**:

- com a entrada no valor que deve ser 4 mA, clique em **LO−**: o LRV passa a ser o valor medido, e o span é mantido;
- com a entrada no valor de 20 mA, clique em **HI+**: o URV passa a ser o valor medido.

Use quando a referência é o próprio processo, por exemplo um banho na temperatura do ponto inferior. Isso **não** corrige o sensor: só muda a faixa.

### 10.7 Calibração do sensor e casamento com o transmissor

1. Retire o Pt100 do poço e coloque-o no **bloco seco** junto com o **termômetro padrão**. A imersão deve ser de pelo menos 15 vezes o diâmetro do sensor, mais o comprimento do elemento.
2. Em cada ponto (por exemplo 0, 50 e 100 °C), espere a estabilização e anote a temperatura do padrão e a resistência do Pt100 (medida a 4 fios).
3. Compare com a tabela da IEC 60751 e com a tolerância da classe do sensor.
4. Para medir com incerteza menor que a da classe, calcule os coeficientes de Callendar–Van Dusen do sensor (R0, A e B) e grave-os no TT301 pela função **Callendar Van Dusen**. Isso é o **casamento sensor–transmissor**.

**Figura 16. Calibração do Pt100 em bloco seco**

![Figura 16](assets/fig16_bloco_seco.jpg)

> **Imagem a inserir:** foto do bloco seco (calibrador de temperatura) com o termômetro padrão e o Pt100 inseridos lado a lado no mesmo inserto, na mesma profundidade, e o indicador do padrão mostrando a temperatura do ponto.

### 10.8 Teste de burnout e teste de malha

1. **Burnout:** desligue um fio do sensor (ou use "Sensor rompido" na planta). A saída deve ir a 21 mA (High) ou 3,6 mA (Low), e o display deve indicar a falha.
2. **Teste de malha:** force 4, 12 e 20 mA pelo DTM e confira a indicação no sistema de controle.

### 10.9 Teste como deixado e registro

Repita o teste da seção 10.3 na faixa final e preencha o registro com:

- identificação (tag, sensor, ligação, faixa);
- padrões usados e validade;
- tabelas como encontrado e como deixado;
- ajustes feitos;
- resultado, responsável e data.

**Figura 17. Gráfico de erro como encontrado e como deixado**

![Figura 17](assets/fig17_grafico_erro.png)

> **Imagem a inserir:** gráfico com os pontos de 0 a 100 % no eixo horizontal e o erro em °C (ou % do span) no vertical, com as séries subida e descida como encontrado e como deixado e as linhas de tolerância (±0,2 % do span).

---

## 11. Bancada virtual no LasecSimul

O projeto permite ensaiar toda a prática:

| Situação real | Como reproduzir no LasecSimul |
|---|---|
| Cabo longo | Propriedade "Resistência de cada fio do cabo" (por exemplo 2 Ω) |
| Ligação a 2 fios | Desligue os fios 2 e 3 da planta e ligue pontes 1–2 e 3–4 nos bornes do TT301 |
| Ligação a 3 fios | Desligue o fio 2 e ligue uma ponte 1–2 nos bornes do TT301 |
| Sensor com deriva | Propriedade "Desvio do sensor" (por exemplo +0,8 °C) |
| Sensor rompido | Propriedade "Sensor rompido" = Sim |
| Simulador de resistência | Desligue a planta e ligue um **Resistor** (Passivos) nos bornes 1–4, com os valores da tabela da seção 10.3 |
| Bloco seco | Aquecimento constante e espera até a estabilização; a "temperatura da água" faz o papel do padrão |

A eletrônica simulada do TT301 é ideal: o simulador mostra os erros da instalação (fios, sensor, faixa), e os trims seguem os mesmos passos do DTM.

**Figura 18. TT301 ligado a 2 fios no simulador**

![Figura 18](assets/fig18_ligacao_2_fios_simulador.png)

> **Imagem a inserir:** captura ampliada do TT301 no LasecSimul com a ligação a 2 fios: só os fios 1 e 4 da planta chegando aos bornes, as pontes 1–2 e 3–4 desenhadas e, ao lado, o display da planta com a temperatura no sensor e o display do TT301 mostrando a leitura maior (erro dos fios).

---

## 12. Sugestão de atividade experimental

### Parte A: comissionamento

1. Conecte o PACTware e salve a configuração como encontrada.
2. Configure o TT301 como na seção 9 (TT-101, Pt100 IEC a 4 fios, 0 a 100 °C, damping 2 s, burnout High).
3. Com aquecimento em 0 %, confira 25 °C e 8,00 mA.

### Parte B: ligação a 2, 3 e 4 fios

1. Na planta, ajuste a resistência de cada fio para **2 Ω**.
2. Com a água estabilizada, registre a leitura do TT301 a 4 fios, a 3 fios (com a conexão 3 fios configurada) e a 2 fios (com a conexão 2 fios configurada).
3. Compare com a temperatura no sensor e com o erro teórico `2R_f/0,385`.

| Ligação | Configuração no DTM | Temperatura no sensor (°C) | Leitura do TT301 (°C) | Erro (°C) | Erro teórico (°C) |
|---|---|---:|---:|---:|---:|
| 4 fios | Four wires | | | | 0 |
| 3 fios | Three wires | | | | 0 |
| 2 fios | Two wires | | | | +10,3 |

### Parte C: calibração do transmissor

1. Substitua a planta por um resistor nos bornes 1–4 (simulador de resistência).
2. Faça o teste de cinco pontos de ida e volta da seção 10.3 e calcule os erros.

### Parte D: sensor com deriva

1. Volte a ligar a planta e ajuste o "Desvio do sensor" para **+0,8 °C**.
2. Compare a leitura do TT301 com a temperatura da água estabilizada (que faz o papel do padrão).
3. O erro está dentro da tolerância de um Pt100 classe A? E classe B?
4. Discuta: o trim do transmissor deve ser usado para esconder o erro do sensor?

### Parte E: tempo de resposta do poço

1. Com a água estabilizada, aplique um degrau de 0 para 100 % no aquecimento.
2. Registre a temperatura da água e a do sensor a cada 10 s durante 2 min.
3. Estime a constante de tempo do poço (tempo para o sensor atingir 63 % da diferença) e compare com os 15 s configurados.

### Parte F: burnout e malha

1. Ative "Sensor rompido" e registre a corrente e o display.
2. Mude o burnout para Low e repita.
3. Faça o teste de malha em 4, 12 e 20 mA.

### Questões para o relatório

1. Por que a ligação a 3 fios só compensa os cabos quando eles têm a mesma resistência?
2. Qual a diferença entre o trim de leitura e a calibração com referência (LO−/HI+)?
3. Por que o sensor e o transmissor devem ser calibrados separadamente?
4. Para que serve gravar os coeficientes de Callendar–Van Dusen do sensor no TT301?
5. Num aquecimento, por que o burnout High costuma ser a escolha segura?

Essa atividade conecta a física do sensor, a instalação elétrica e a configuração e calibração de um transmissor HART.

---

## 13. Conclusão

A prática mostra, de forma aplicada, a cadeia completa de uma medição de temperatura industrial:

- o aquecimento muda a temperatura da água, e o poço transmite essa temperatura ao Pt100 com atraso;
- o Pt100 muda de resistência conforme a IEC 60751;
- a ligação a 4 fios elimina os cabos da medida;
- o TT301 mede a resistência, lineariza e transmite a temperatura em 4–20 mA com HART;
- a calibração separa o erro do sensor e o do transmissor, e os trims corrigem cada um no lugar certo.

Com isso, o roteiro serve tanto como material de estudo quanto como apoio para a aula prática de instrumentação.

---

## 14. Referências

- Apostila de Instrumentação Industrial II - UFU
- Smar. *TT301 — Manual de Instrução, Operação e Manutenção* (seção 1: instalação e ligação do sensor; seção 2: RTD e conexões a 2, 3 e 4 fios; seção 3: configuração, Trim e calibração).
- IEC 60751. *Industrial platinum resistance thermometers and platinum temperature sensors.*
- EURAMET cg-13. *Calibration of Temperature Block Calibrators.*
- ABNT NBR ISO/IEC 17025. *Requisitos gerais para a competência de laboratórios de ensaio e calibração.*
- REOTEMP. [Making Sense of RTD Tolerance Classes](https://reotemp.com/2025/06/24/rtd-tolerance-classes-explained).
- All About Circuits. [Modeling RTD characteristics: alpha parameter and Callendar–Van Dusen equation](https://www.allaboutcircuits.com/technical-articles/modeling-resistance-temperature-detector--characteristics-using-alpha-parameter-callendar-van-dusen-equation-iec-751/).
- Fluke. [Selecting a dry-block calibrator](https://www.fluke.com/en-gb/learn/blog/calibration/selecting-dry-block-calibrator).
- Fluke. [Calculadora PT100](https://fluke.com/pt-br/saiba-mais/ferramentas-calculadoras/pt100-calculator).
- ABCM CONEM 2010. [Calibração de Pt100 em bloco seco](https://www.abcm.org.br/anais/conem/2010/PDF/CON10-0431.pdf).
- [LasecSimul — LASEC/UFU](https://github.com/LASEC-UFU/LasecSimul)

---

## 15. Apêndice: lista de imagens

Todas as imagens ficam na pasta [`assets/`](assets), com os nomes exatos da tabela. Cada figura do texto já traz, no quadro "Imagem a inserir", o que ela precisa mostrar. Este apêndice reúne como produzir cada uma.

### 15.1 Como produzir as imagens

- **Capturas do LasecSimul:** VS Code maximizado (1920 × 1080), barra lateral fechada (`Ctrl+B`), escala do Windows em 100 %, simulação rodando e valores estabilizados. Use **Ajustar tudo** ou **Ajustar seleção** e a Ferramenta de Captura (`Win+Shift+S`). Salve em PNG.
- **Capturas do PACTware:** janela maximizada, TT301 conectado (ícones verdes). Capture antes de aplicar quando a figura mostra valores digitados.
- **Fotos:** luz difusa, sem flash direto nos displays, câmera paralela ao instrumento, valores legíveis. Salve em JPG.
- **Desenhos e gráficos:** diagrams.net (draw.io), Inkscape, PowerPoint, planilha ou Python, fundo branco, PNG com cerca de 1600 px de largura.
- **Anotações:** setas e números em vermelho, explicados na legenda. Nos desenhos de ligação, use as cores da IEC 60751: fios **vermelhos** numa ponta do elemento e **brancos** na outra.

### 15.2 Figuras

| Figura | Arquivo | Tipo | Como obter | O que destacar |
|---:|---|---|---|---|
| 1 | `fig01_pt100_industrial.jpg` | Foto | Foto de um Pt100 industrial do laboratório com o cabeçote aberto. | Bainha, poço, cabeçote e os 4 fios (2 vermelhos, 2 brancos). |
| 2 | `fig02_curva_pt100.png` | Gráfico | Planilha ou Python com a equação de Callendar–Van Dusen de −200 a 850 °C. | Pontos da tabela da seção 2.2 e o detalhe de 0 a 100 °C com 0,385 Ω/°C. |
| 3 | `fig03_ligacoes_2_3_4_fios.png` | Desenho | Redesenhe a Figura 1.9 do manual só com as ligações de RTD a 2, 3 e 4 fios. | Pontes 1–2 e 3–4 (2 fios), ponte 1–2 (3 fios) e as cores dos fios. |
| 4 | `fig04_esquema_4_fios.png` | Desenho | Redesenhe a Figura 2.5 do manual. | Caminho da corrente pelos bornes 1 e 4 e a medição de alta impedância entre 2 e 3. |
| 5 | `fig05_tt301_bornes.jpg` | Foto | Foto do TT301 do laboratório com a tampa dos terminais aberta. | Bornes + e −, bornes 1 a 4 do sensor, TEST e COMM. |
| 6 | `fig06_cadeia_sinal_tt301.png` | Desenho | Diagrama de blocos simplificado a partir da Figura 2.2 do manual. | Onde agem o trim de leitura, a faixa (LRV/URV) e o trim de corrente. |
| 7 | `fig07_projeto_completo.png` | Captura LasecSimul | Abra `tanque_aquecido_tt301.lsproj`, clique em **Run**, aquecimento em 50 %, espere alguns segundos e clique em **Ajustar tudo**. | Slider, planta, os 4 fios até o TT301, fonte, resistor, amperímetro, modem e osciloscópio. |
| 8 | `fig08_planta.png` | Captura LasecSimul | Com 50 % de aquecimento por alguns minutos, selecione a planta, **Ajustar seleção** e recolha o painel Properties. | Lâmpada "Aquecendo", poço com o Pt100, mostrador TI-101, barra de temperatura e displays. |
| 9 | `fig09_propriedades_exportadas.png` | Captura LasecSimul | Selecione a planta e abra **Properties → Propriedades exportadas**. | Os nove parâmetros, com a caixa "Sensor rompido" aberta. |
| 10 | `fig10_pactware_conectado.png` | Captura PACTware | Conecte o TT301 e abra **Identification**. | Ícones verdes, SMAR, TT301, Tag, Descriptor e Message. |
| 11 | `fig11_dtm_sensor.png` | Captura PACTware | **Configuration → Sensor** com os valores escolhidos, antes de aplicar. | RTD, Pt 100 IEC e Four wires. |
| 12 | `fig12_dtm_calibration.png` | Captura PACTware | Tela **Calibration** com a faixa digitada. | degC, LRV 0, URV 100, Damping 2, LRL/URL, Min Span e os botões LO−/HI+. |
| 13 | `fig13_dtm_configuration.png` | Captura PACTware | Tela **Configuration**. | W.Protect, Burnout High e o quadro LCD. |
| 14 | `fig14_lcd_tt301.png` | Captura LasecSimul (montagem) | Água estabilizada; aproxime o display do TT301 (zoom de cerca de 300 %) e capture as duas páginas com 3 s de intervalo. | Temperatura em °C e a segunda variável. |
| 15 | `fig15_simulador_resistencia.jpg` | Foto | Foto da bancada com a década ou o calibrador de processo nos bornes 1–4 e o multímetro na malha. | Valor da década legível e a PV correspondente no notebook. |
| 16 | `fig16_bloco_seco.jpg` | Foto | Foto do bloco seco com o termômetro padrão e o Pt100 no mesmo inserto. | Mesma profundidade de imersão e a temperatura do padrão no indicador. |
| 17 | `fig17_grafico_erro.png` | Gráfico | Planilha ou Python com as tabelas da seção 10.3 (como encontrado e como deixado). | Quatro séries e as linhas de tolerância de ±0,2 % do span. |
| 18 | `fig18_ligacao_2_fios_simulador.png` | Captura LasecSimul | Na Parte B, com a ligação a 2 fios montada e os fios de 2 Ω, aproxime a região do TT301. | As pontes 1–2 e 3–4 e a diferença entre a temperatura no sensor e a leitura do TT301. |
