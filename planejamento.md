# Planejamento experimental para adaptação e calibração de aprendizado profundo guiado por física na estimativa da produtividade de soja no Paraná

**Responsável:** Prof. Dr. Fabrício Correia de Oliveira (UTFPR – Campus Santa Helena, GAMBI-TEC)
**Documento:** planejamento interno de pesquisa (versão 1)
**Repositórios relacionados:** `fcoliveira-utfpr/soybean_yield_forecasts`, `fcoliveira-utfpr/aquacrop_ml`

---

## 1. Contexto e justificativa

A estimativa da produtividade de culturas em larga escala depende de dois grupos de modelos com limitações complementares. Os modelos de processo representam a fisiologia da cultura, mas exigem muitos parâmetros e perdem desempenho em escala regional. Os modelos estatísticos e de aprendizado de máquina ajustam bem os dados de treino, mas dependem fortemente da amostra e extrapolam mal para regiões e anos fora da distribuição de treino (KARPATNE et al., 2017; SHEN et al., 2023).

Cao et al. (2026) propuseram um modelo de aprendizado profundo guiado por física para o trigo na China. Nesse modelo, a física do PC model (QIAO et al., 2020; QIAO et al., 2021) foi incorporada à rede de três formas: como camadas físicas (estimativa da GPP por otimalidade, baseada no P-model), como termo de perda física (balanço de massa de carbono do dossel) e como conhecimento prévio sobre a faixa dos parâmetros. Os ganhos foram maiores nos testes de extrapolação, com redução média do RMSE de até 15% no teste com uma província deixada de fora (CAO et al., 2026).

A transferência dessa abordagem para a soja no Brasil esbarra em três limitações. A física do PC model foi construída para trigo e não contém fenologia de soja. O estresse hídrico não é representado de forma explícita, embora o déficit hídrico seja o principal fator de perda da soja no Sul do Brasil. E o desenho de avaliação original não trata a tendência tecnológica da produtividade nem a autocorrelação espacial. Por outro lado, a soja reúne condições favoráveis de dados: máscara anual disponível no MapBiomas, ciclo único de verão bem definido nas séries de índices de vegetação e grande número de registros municipais.

Neste sentido, esta pesquisa propõe adaptar e calibrar o modelo de Cao et al. (2026) para a soja no Paraná e quantificar qual representação física (carbono, água ou ambas) contribui para o desempenho e para a capacidade de extrapolação.

**Hipótese principal:** a incorporação explícita do estresse hídrico como camada física diferenciável aumenta a acurácia e a capacidade de extrapolação do modelo híbrido em relação à física de carbono original, sobretudo em safras com déficit hídrico.

---

## 2. Objetivos

### 2.1 Objetivo geral

Desenvolver, calibrar e validar modelos de aprendizado profundo guiados por física para a estimativa da produtividade municipal de soja no Paraná, com diferentes representações físicas (carbono, água e carbono com água), e comparar seu desempenho com modelos de processo, modelos de aprendizado de máquina e modelos combinados.

### 2.2 Objetivos específicos

- Construir uma base municipal semanal de clima, solo, sensoriamento remoto e fenologia da soja no Paraná para as safras 2000/01 a 2024/25.
- Implementar em PyTorch três núcleos físicos diferenciáveis: (A) GPP por otimalidade (P-model); (B) GPP por otimalidade com fator de estresse hídrico; (C) produtividade da água do AquaCrop.
- Calibrar os parâmetros físicos dinâmicos com restrição de conhecimento prévio derivada da literatura de soja e da calibração do AquaCrop.
- Quantificar o desempenho e a capacidade de extrapolação espacial, temporal e para safras de seca de cada modelo, com validação em blocos espaciais e testes estatísticos entre modelos.
- Estabelecer intervalos de previsão calibrados (predição conformal) para os modelos com melhor desempenho.
- Verificar a consistência física dos modelos híbridos (balanço de carbono, faixa dos parâmetros estimados e padrões de importância das variáveis por fase fenológica).

---

## 3. Hipóteses

| Código | Hipótese | Experimentos que testam |
|---|---|---|
| H1 | Os modelos híbridos reduzem o erro em relação à LSTM-CNN pura nos testes de extrapolação espacial e temporal | E2, E3 |
| H2 | O núcleo com água (B ou C) supera o núcleo apenas de carbono (A) em safras de seca | E4 |
| H3 | A vantagem dos híbridos aumenta quando o número de amostras de treino diminui | E7 |
| H4 | Os parâmetros físicos estimados variam de forma coerente com o déficit hídrico e com a latitude, dentro das faixas da literatura | E9 |

---

## 4. Área de estudo e recorte

- **Fase A:** estado do Paraná, todos os municípios com área de soja acima do limite mínimo definido na etapa (iii). A divisão em mesorregiões geográficas do IBGE (10 mesorregiões) será usada nos testes espaciais.
- **Fase B (expansão):** Mato Grosso do Sul, Rio Grande do Sul, Goiás e Mato Grosso, para testar a extrapolação climática entre zonas Cfa, Cfb, Aw e Am (ALVARES et al., 2013).
- **Período:** safras 2000/01 a 2024/25 (25 safras), condicionado à disponibilidade de MODIS a partir de 2000.

---

## 5. Dados

| Componente | Fonte | Resolução | Uso |
|---|---|---|---|
| Produtividade, área plantada e colhida | IBGE/PAM, tabela SIDRA 1612 | Municipal, anual | Variável resposta |
| Máscara de soja | MapBiomas, classe soja (coleção vigente) | 30 m, anual | Agregação municipal e ponderação |
| Precipitação, temperatura, umidade, radiação, vento | Xavier et al. (BR-DWGD) e ERA5-Land | 0,1° a 0,25°, diário | Entradas climáticas e balanço hídrico |
| Dados de estação | INMET (asset `projects/fcoliveira/assets/inmet_pr_diario`) e NASA/POWER | Pontual, diário | Validação e correção de viés das grades |
| ETo | Penman-Monteith FAO-56 (ALLEN et al., 1998) | Diário | Balanço hídrico e núcleo C |
| Solo | SoilGrids, MapBiomas Solo (textura), PronaSolos | 30 a 250 m | Capacidade de água disponível (CAD), variáveis de solo |
| NDVI e EVI | MODIS MOD13Q1 / MYD13Q1 | 250 m, 16 dias | Fenologia e entrada da rede |
| FPAR e LAI | MODIS MOD15A2H / MYD15A2H | 500 m, 8 dias | Perda de dossel observado |
| CO₂ | NOAA (Mauna Loa) | Anual | Entrada do P-model |
| Índice ENSO | NOAA ONI | Mensal | Covariável e estratificação das safras |
| Relevo | SRTM | 30 m | Altitude e declividade |

---

## 6. Procedimentos de preparação dos dados

**(i) Agregação espacial.** As variáveis serão agregadas por município apenas sobre os pixels de soja da máscara do MapBiomas do respectivo ano. Para o FPAR e o LAI do MODIS (500 m), a média será ponderada pela fração de soja dentro de cada pixel, e pixels com fração inferior a [INSERIR LIMIAR, ex.: 0,5] serão descartados.

**(ii) Fenologia.** As datas de semeadura aparente (início de estação, SOS), pico (POS) e fim de estação (EOS) serão extraídas das séries de EVI suavizadas por Savitzky–Golay (CHEN et al., 2004). Será usado o método de limiar de amplitude:

$$
EVI_{lim} = EVI_{min} + f \,(EVI_{max} - EVI_{min}) \qquad \text{(eq. 1)}
$$

em que: $EVI_{lim}$ é o valor de limiar (adimensional); $EVI_{min}$ e $EVI_{max}$ são os valores mínimo e máximo do ciclo (adimensional); $f$ é a fração de amplitude, igual a 0,2 para SOS e EOS e testada entre 0,1 e 0,5 na análise de sensibilidade. As datas serão comparadas com as janelas do ZARC para verificação de plausibilidade.

**(iii) Filtros de registros.** Serão mantidos os municípios-safra com área de soja acima de [INSERIR LIMIAR, ex.: 1.000 ha] e com pelo menos [INSERIR N] semanas válidas de dados. **Não será aplicado filtro de produtividade baixa**, diferentemente de Cao et al. (2026), que descartaram valores abaixo de 1 t ha⁻¹. Valores baixos em safras de seca são reais e constituem justamente o objeto do teste E4. Serão removidos apenas registros com inconsistência evidente (área colhida maior que a plantada, valores repetidos idênticos por mais de [INSERIR N] safras consecutivas).

**(iv) Passo temporal.** As séries serão agregadas em passos semanais, da semeadura aparente até a maturação, e preenchidas com valor de máscara até um comprimento fixo de [INSERIR N, ex.: 22] semanas, com a máscara informada à rede para não contaminar a LSTM.

**(v) Tendência tecnológica.** Três tratamentos serão comparados:

- T1: remoção prévia da tendência por município (linear ou LOESS), com modelagem da anomalia;
- T2: tendência incluída como termo aditivo fora da rede, ajustado conjuntamente;
- T3: ano-safra como variável de entrada, como referência do que ocorre sem tratamento explícito.

A métrica principal de habilidade será calculada sobre as anomalias destendenciadas, para evitar que a tendência infle r e R².

**(vi) Balanço hídrico do solo.** Será calculado diariamente e agregado por semana, conforme FAO-56 (ALLEN et al., 1998):

$$
D_{r,i} = \min\!\left[\,CAD,\; \max\!\left(0,\; D_{r,i-1} - P_{ef,i} + ET_{c,i}\right)\right] \qquad \text{(eq. 2)}
$$

$$
CAD = 1000\,(\theta_{CC} - \theta_{PMP})\,Z_r \qquad \text{(eq. 3)}
$$

$$
K_s =
\begin{cases}
1, & D_{r,i} \le p \cdot CAD \\[4pt]
\dfrac{CAD - D_{r,i}}{(1-p)\,CAD}, & D_{r,i} > p \cdot CAD
\end{cases}
\qquad \text{(eq. 4)}
$$

em que: $D_{r,i}$ é a depleção da água na zona radicular no dia $i$ (mm); $P_{ef,i}$ é a precipitação efetiva (mm); $ET_{c,i}$ é a evapotranspiração da cultura (mm); $CAD$ é a capacidade de água disponível (mm); $\theta_{CC}$ e $\theta_{PMP}$ são as umidades volumétricas na capacidade de campo e no ponto de murcha permanente (m³ m⁻³); $Z_r$ é a profundidade efetiva das raízes (m); $p$ é a fração de água facilmente disponível (adimensional); $K_s$ é o coeficiente de estresse hídrico (adimensional).

---

## 7. Núcleos físicos diferenciáveis

Os três núcleos serão implementados em PyTorch, de modo que o gradiente atravesse a física e os parâmetros sejam estimados de forma endógena, como em Cao et al. (2026).

### 7.1 Núcleo A, GPP por otimalidade (física de Cao et al., 2026)

A GPP é obtida pelo modelo de eficiência de uso da luz com LUE prevista por otimalidade (WANG et al., 2017; STOCKER et al., 2020):

$$
GPP = LUE \cdot FPAR \cdot PAR \qquad \text{(eq. 5)}
$$

$$
\chi = \frac{\Gamma^*}{c_a} + \left(1 - \frac{\Gamma^*}{c_a}\right)\frac{\xi}{\xi + \sqrt{D}},
\qquad
\xi = \sqrt{\frac{\beta\,(K + \Gamma^*)}{1{,}6\,\eta^*}} \qquad \text{(eq. 6)}
$$

$$
m = \frac{\chi c_a - \Gamma^*}{\chi c_a + 2\Gamma^*},
\qquad
LUE = \varphi_0\, m \sqrt{1 - \left(\frac{c^*}{m}\right)^{2/3}} \qquad \text{(eq. 7)}
$$

em que: $GPP$ é a produção primária bruta (g C m⁻² semana⁻¹); $LUE$ é a eficiência de uso da luz (g C mol⁻¹ fóton); $FPAR$ é a fração da radiação fotossinteticamente ativa absorvida (adimensional); $PAR$ é a radiação fotossinteticamente ativa (mol fóton m⁻² semana⁻¹); $\chi$ é a razão entre as concentrações interna e externa de CO₂ (adimensional); $\Gamma^*$ é o ponto de compensação fotorrespiratório (Pa); $c_a$ é a pressão parcial de CO₂ ambiente (Pa); $D$ é o déficit de pressão de vapor (Pa); $K$ é o coeficiente de Michaelis-Menten efetivo da Rubisco (Pa); $\eta^*$ é a viscosidade relativa da água (adimensional); $\beta$ é a razão de custos entre carboxilação e transpiração (adimensional); $\varphi_0$ é a eficiência quântica intrínseca (mol mol⁻¹, convertida para g C); $c^*$ é o parâmetro de custo do transporte de elétrons (adimensional, 0,41).

A implementação de referência do `pyrealm` será usada para conferir a versão diferenciável. O balanço de carbono do dossel entra como perda física, conforme Cao et al. (2026):

$$
FPAR = 1 - e^{-k \cdot LAI},
\qquad
LAI_{dem} = \frac{\eta_L \sum GPP}{LMA} \qquad \text{(eq. 8)}
$$

em que: $k$ é o coeficiente de extinção da luz (adimensional); $LAI$ é o índice de área foliar previsto pela rede (m² m⁻²); $LAI_{dem}$ é o LAI sustentado pela alocação de carbono (m² m⁻²); $\eta_L$ é a fração da GPP alocada às folhas (adimensional); $LMA$ é a massa foliar específica (g C m⁻²).

### 7.2 Núcleo B, GPP com estresse hídrico

Igual ao núcleo A, com a GPP multiplicada pelo coeficiente de estresse da eq. 4, calculado de forma diferenciável (função suave em vez de função por partes):

$$
GPP_B = K_s^{\,\gamma} \cdot LUE \cdot FPAR \cdot PAR \qquad \text{(eq. 9)}
$$

em que: $\gamma$ é o expoente de sensibilidade ao estresse, parâmetro dinâmico estimado pela rede (adimensional).

### 7.3 Núcleo C, produtividade da água (AquaCrop)

Baseado nas equações centrais do AquaCrop (STEDUTO et al., 2009; RAES et al., 2009):

$$
Tr_t = K_{s,t}\; CC^*_t\; K_{cb,x}\; ET_{o,t} \qquad \text{(eq. 10)}
$$

$$
B = WP^* \sum_t \frac{Tr_t}{ET_{o,t}},
\qquad
Y_{fis} = HI \cdot B \qquad \text{(eq. 11)}
$$

em que: $Tr_t$ é a transpiração na semana $t$ (mm); $CC^*_t$ é a cobertura do dossel ajustada (adimensional), prevista pela LSTM e restringida pelo FPAR observado; $K_{cb,x}$ é o coeficiente máximo de transpiração da cultura (adimensional); $ET_{o,t}$ é a evapotranspiração de referência (mm); $B$ é a biomassa acima do solo (t ha⁻¹); $WP^*$ é a produtividade da água normalizada (g m⁻²); $HI$ é o índice de colheita (adimensional); $Y_{fis}$ é a produtividade estimada pela camada física (t ha⁻¹).

### 7.4 Parâmetros dinâmicos e conhecimento prévio

Os parâmetros $\beta$, $\varphi_0$, $\eta_L$, $LMA$, $\gamma$, $WP^*$, $HI$ e $p$ serão previstos por uma sub-rede (camadas densas) a partir de variáveis climáticas da safra e da latitude. As faixas e os valores iniciais virão da literatura de soja e da calibração do AquaCrop-OSPy (KELLY; FOSTER, 2021) no Paraná, e não dos valores padrão do trigo.

| Parâmetro | Valor inicial | Faixa | Fonte |
|---|---|---|---|
| $WP^*$ (g m⁻²) | [INSERIR] | [INSERIR] | Calibração AquaCrop soja |
| $HI$ | [INSERIR] | [INSERIR] | Literatura nacional |
| $p$ | [INSERIR] | [INSERIR] | FAO-56 |
| $\beta$ | [INSERIR] | [INSERIR] | Stocker et al. (2020) |
| $LMA$, $\eta_L$ | [INSERIR] | [INSERIR] | Literatura de soja |

### 7.5 Função de perda

$$
\mathcal{L} = \mathrm{MSE}(Y, \hat{Y})
+ \lambda_1\, \mathrm{MSE}(FPAR_{obs}, \widehat{FPAR})
+ \lambda_2\, \mathcal{L}_{fis}
+ \lambda_3 \sum_k \left(\frac{\theta_k - \theta_k^0}{\theta_k^0}\right)^2 \qquad \text{(eq. 12)}
$$

em que: $Y$ e $\hat{Y}$ são as produtividades observada e prevista (t ha⁻¹); $FPAR_{obs}$ e $\widehat{FPAR}$ são o FPAR do MODIS e o calculado a partir do LAI ou da cobertura previstos; $\mathcal{L}_{fis}$ é a perda física (balanço de carbono, eq. 8, nos núcleos A e B; consistência entre biomassa por transpiração e por dossel no núcleo C); $\theta_k$ e $\theta_k^0$ são o valor estimado e o valor de referência do parâmetro $k$; $\lambda_1$, $\lambda_2$ e $\lambda_3$ são pesos ajustados na validação.

### 7.6 Módulo de alocação

A saída física ($GPP$ acumulada ou $Y_{fis}$), juntamente com $K_s$, a cobertura do dossel, as variáveis de solo, relevo e índices de vegetação, alimenta uma LSTM-CNN com camadas densas finais, seguindo a arquitetura de Cao et al. (2026).

---

## 8. Modelos comparados

| Código | Modelo | Grupo | Observação |
|---|---|---|---|
| M1 | AquaCrop-OSPy calibrado | Processo | Referência física, simulação por município |
| M2 | DSSAT/CROPGRO-Soybean | Processo | Opcional, apenas em municípios-piloto |
| M3 | Modelo do `soybean_yield_forecasts` | Aprendizado de máquina | Baseline próprio |
| M4 | LSTM-CNN pura | Aprendizado profundo | Equivalente ao "conventional deep learning" de Cao et al. (2026) |
| M5 | LSTM-CNN com saídas do M1 como entrada | Combinado | Equivalente ao "conventional combined" de Cao et al. (2026) |
| H-A | Híbrido com núcleo A | Guiado por física | Replicação adaptada de Cao et al. (2026) |
| H-B | Híbrido com núcleo B | Guiado por física | Carbono com água |
| H-C | Híbrido com núcleo C | Guiado por física | Água (AquaCrop diferenciável) |

Todos os modelos de aprendizado serão treinados com as mesmas entradas, partições e orçamento de ajuste de hiperparâmetros, para garantir uma comparação justa.

---

## 9. Experimentos

### E1, desempenho geral com blocos espaciais

Validação cruzada em 5 blocos espaciais, formados por agrupamento de municípios contíguos (ROBERTS et al., 2017), em substituição à divisão aleatória usada por Cao et al. (2026). Em cada dobra, os blocos restantes serão divididos em treino e validação (70:30). Uma divisão aleatória também será rodada apenas para quantificar o quanto ela infla o desempenho.

### E2, extrapolação espacial

Deixar uma mesorregião de fora, com 10 testes. Na Fase B, deixar um estado de fora, com o Paraná sempre no treino.

### E3, extrapolação temporal

Origem móvel: treino com as safras até o ano $t$ e teste na safra $t+1$, para $t$ de [INSERIR, ex.: 2012/13] a 2023/24. Será rodado também o teste deixando uma safra de fora, para comparação direta com Cao et al. (2026). Os dois testes serão executados com os tratamentos de tendência T1, T2 e T3.

### E4, safras de seca

As safras serão classificadas pela anomalia de produtividade destendenciada e pelo déficit hídrico acumulado (SPI ou ISNA no período reprodutivo). Safras com seca severa no estado serão retiradas do treino e usadas como teste. As safras candidatas (por exemplo, 2008/09, 2011/12 e 2021/22) serão confirmadas pelos próprios dados antes do experimento. A métrica principal será o erro sobre as anomalias negativas.

### E5, ablações

Para cada híbrido serão removidos, um de cada vez:

- a camada física (mantendo a LSTM-CNN);
- a perda física;
- a perda de FPAR observado;
- os parâmetros dinâmicos (substituídos por parâmetros fixos calibrados);
- a perda de conhecimento prévio;
- os blocos CNN.

As ablações serão avaliadas no E2, que é o teste mais exigente, como em Cao et al. (2026).

### E6, sensibilidade às fontes de dados

| Variante | Comparação |
|---|---|
| Clima | Xavier contra ERA5-Land contra NASA/POWER |
| FPAR | MODIS ponderado pela fração de soja contra sem perda de FPAR |
| Fenologia | Limiar $f$ de 0,1 a 0,5 contra calendário fixo do ZARC |
| Solo | CAD do SoilGrids contra MapBiomas Solo |

### E7, tamanho amostral

Curvas de aprendizado com frações de 10%, 25%, 50%, 75% e 100% dos municípios de treino, mantendo fixos os blocos de teste. Esse experimento testa H3, a afirmação de Cao et al. (2026) de que a física regulariza a rede quando há poucos dados.

### E8, incerteza

Intervalos de previsão por predição conformal com calibração por blocos espaciais, nos níveis nominais de 80% e 95%. A avaliação considerará a cobertura empírica e a largura média dos intervalos, por mesorregião e para as safras de seca.

### E9, verificação física

- Diferença relativa entre a demanda e a oferta de carbono do dossel nos núcleos A e B, com e sem a perda física.
- Distribuição dos parâmetros dinâmicos estimados, comparada com as faixas da literatura, e sua relação com $K_s$ acumulado e com a latitude.
- SHAP por semana relativa à semeadura, para verificar se a importância das variáveis se concentra nas fases reprodutivas (LUNDBERG; LEE, 2017).
- Correlação entre a GPP simulada e produtos independentes (por exemplo, SIF ou GPP de satélite) nos pixels de soja.

### E10, aplicação (opcional)

Cenários de data de semeadura com o melhor híbrido, integrados ao framework multicritério da soja no Paraná.

---

## 10. Métricas e análise estatística

| Métrica | Uso |
|---|---|
| r, R² | Associação, calculados também sobre as anomalias destendenciadas |
| RMSE, NRMSE, MAE, viés | Magnitude do erro |
| d de Willmott (WILLMOTT, 1981), NSE | Concordância |
| Taxa de acerto do sinal da anomalia | Habilidade em safras atípicas |
| Cobertura e largura dos intervalos | Incerteza (E8) |

As diferenças entre modelos serão testadas por bootstrap pareado por bloco espacial e pelo teste de Diebold-Mariano nos testes temporais (DIEBOLD; MARIANO, 1995), com correção de Holm para comparações múltiplas. Os resultados serão apresentados com intervalos de confiança, e não apenas com médias.

---

## 11. Implementação

- **Extração e agregação:** Google Earth Engine, com assets próprios.
- **Modelagem:** Python 3, PyTorch, `pyrealm` para conferência do P-model, AquaCrop-OSPy para M1 e calibração.
- **Execução:** Google Colab com GPU; Cao et al. (2026) relataram cerca de 2 min e 2 GB por teste, o que é compatível.
- **Reprodutibilidade:** sementes fixas, registro de configurações e código e dados agregados em repositório público no GitHub (`fcoliveira-utfpr`).

---

## 12. Riscos e mitigação

| Risco | Efeito | Mitigação |
|---|---|---|
| Código de Cao et al. (2026) não disponível | Reimplementação pode divergir | Solicitar o código aos autores; documentar as diferenças |
| Parâmetros físicos não identificáveis | Parâmetros absorvem erro da rede | Faixas da literatura, perda de conhecimento prévio, E9 |
| Erro da produtividade do IBGE | Limite inferior do erro alcançável | Filtros de consistência; métricas sobre anomalias |
| FPAR contaminado por outras coberturas | Perda de dossel enviesada | Ponderação pela fração de soja; variante sem FPAR (E6) |
| Ganho pequeno dos híbridos | Resultado pouco expressivo | Comparação entre núcleos responde à pergunta mesmo com ganho pequeno |
| Tendência tecnológica | Habilidade inflada | Tratamentos T1–T3 e métricas sobre anomalias |

---

## 13. Cronograma

| Atividade | T1 | T2 | T3 | T4 | T5 | T6 |
|---|---|---|---|---|---|---|
| (i) Base de dados e fenologia no GEE | X | X | | | | |
| (ii) Balanço hídrico e calibração do AquaCrop-OSPy | X | X | | | | |
| (iii) Baselines M3, M4 e M5 | | X | X | | | |
| (iv) Núcleos A, B e C em PyTorch | | X | X | | | |
| (v) Experimentos E1 a E5 | | | X | X | | |
| (vi) Experimentos E6 a E9 | | | | X | X | |
| (vii) Redação do artigo da Fase A | | | | | X | X |
| (viii) Fase B (expansão estadual) | | | | | X | X |

T = trimestre.

---

## 14. Produtos esperados

- Artigo da Fase A em periódico internacional (candidatos: *Agricultural and Forest Meteorology*, *Computers and Electronics in Agriculture*, *European Journal of Agronomy*, *Field Crops Research*).
- Base municipal semanal de soja no Paraná e código no GitHub.
- Orientações de iniciação científica, TCC ou mestrado vinculadas aos experimentos.
- Inserção como subprojeto da Fase II do projeto de recursos hídricos no oeste do Paraná.

---

## Referências

ALLEN, R. G.; PEREIRA, L. S.; RAES, D.; SMITH, M. **Crop evapotranspiration**: guidelines for computing crop water requirements. Rome: FAO, 1998. (FAO Irrigation and Drainage Paper, 56).

ALVARES, C. A. et al. Köppen's climate classification map for Brazil. **Meteorologische Zeitschrift**, v. 22, n. 6, p. 711-728, 2013.

CAO, B. et al. Physics-guided deep learning for crop yield estimation. **European Journal of Agronomy**, v. 172, 127850, 2026. DOI: 10.1016/j.eja.2025.127850.

CHEN, J. et al. A simple method for reconstructing a high-quality NDVI time-series data set based on the Savitzky–Golay filter. **Remote Sensing of Environment**, v. 91, p. 332-344, 2004.

DIEBOLD, F. X.; MARIANO, R. S. Comparing predictive accuracy. **Journal of Business & Economic Statistics**, v. 13, n. 3, p. 253-263, 1995.

KARPATNE, A. et al. Theory-guided data science: a new paradigm for scientific discovery from data. **IEEE Transactions on Knowledge and Data Engineering**, v. 29, p. 2318-2331, 2017.

KELLY, T. D.; FOSTER, T. AquaCrop-OSPy: bridging the gap between research and practice in crop-water modeling. **Agricultural Water Management**, v. 254, 106976, 2021.

LUNDBERG, S. M.; LEE, S.-I. A unified approach to interpreting model predictions. In: **Advances in Neural Information Processing Systems**, 2017.

QIAO, S. et al. Extending a first-principles primary production model to predict wheat yields. **Agricultural and Forest Meteorology**, v. 287, 107932, 2020. DOI: 10.1016/j.agrformet.2020.107932.

QIAO, S. et al. Optimality-based modelling of climate impacts on global potential wheat yield. **Environmental Research Letters**, v. 16, 114013, 2021. DOI: 10.1088/1748-9326/ac2e38.

RAES, D. et al. AquaCrop — the FAO crop model to simulate yield response to water: II. Main algorithms and software description. **Agronomy Journal**, v. 101, p. 438-447, 2009.

ROBERTS, D. R. et al. Cross-validation strategies for data with temporal, spatial, hierarchical, or phylogenetic structure. **Ecography**, v. 40, p. 913-929, 2017.

SHEN, C. et al. Differentiable modelling to unify machine learning and physical models for geosciences. **Nature Reviews Earth & Environment**, v. 4, p. 552-567, 2023. DOI: 10.1038/s43017-023-00450-9.

STEDUTO, P. et al. AquaCrop — the FAO crop model to simulate yield response to water: I. Concepts and underlying principles. **Agronomy Journal**, v. 101, p. 426-437, 2009.

STOCKER, B. D. et al. P-model v1.0: an optimality-based light use efficiency model for simulating ecosystem gross primary production. **Geoscientific Model Development**, v. 13, p. 1545-1581, 2020. DOI: 10.5194/gmd-13-1545-2020.

WANG, H. et al. Towards a universal model for carbon dioxide uptake by plants. **Nature Plants**, v. 3, p. 734-741, 2017. DOI: 10.1038/s41477-017-0006-8.

WILLMOTT, C. J. On the validation of models. **Physical Geography**, v. 2, n. 2, p. 184-194, 1981.

> Observação: as referências devem ser conferidas (volume, páginas e DOI) antes do uso em texto submetido.
