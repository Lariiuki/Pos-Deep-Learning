README — Projeto de Deep Learning para Séries Temporais
📋 Visão Geral
Este repositório contém 3 notebooks Jupyter que implementam um pipeline completo e padronizado para previsão de séries temporais usando três abordagens diferentes:

ARIMA — Metodologia estatística clássica (Box & Jenkins)
MLP — Rede neural densa (Multilayer Perceptron)
LSTM — Rede neural recorrente (Long Short-Term Memory)
Cada notebook segue a mesma estrutura experimental, variando apenas os dados (série temporal). Isso permite comparações diretas entre modelos em contextos diferentes.

## 📁 Estrutura de Arquivos

| # | Arquivo | Série | Frequência | Tamanho | Período |
|---|---------|-------|-----------|---------|---------|
| 1 | `notebook_01_airpassengers.ipynb` | AirPassengers — Passageiros aéreos internacionais | Mensal | 144 obs | Jan/1949 – Dez/1960 |
| 2 | `notebook_02_vendas_varejo.ipynb` | Varejo PE — Índice de vendas no varejo em Pernambuco | Mensal | 189 obs | 2010–2025 |
| 3 | `notebook_03_co2.ipynb` | CO₂ — Concentração de CO₂ em Mauna Loa, Hawaii | Semanal | 2284 obs | 1958–2001 |

Características por Série
Dataset 1 — AirPassengers (pequeno, ~144 pontos)
Série multiplicativa com tendência crescente e sazonalidade forte (período=12)
Aplicada: transformação logarítmica para estabilizar variância
Desafiador para: modelos com poucos dados

Dataset 2 — Varejo PE (médio, ~189 pontos)
Série aditiva com leve tendência e sazonalidade moderada
Sem transformações especiais aplicadas
Desafiador para: capturar padrões complexos com dados limitados

Dataset 3 — CO₂ (grande, ~2284 pontos)
Série aditiva com tendência forte e sazonalidade clara (período=52)
Suficiente para deep learning (LSTM)
Desafiador para: modelar tendências de longo prazo

🔄 Pipeline Experimental Padronizado
Cada notebook segue 6 etapas principais:

1️⃣ Carregamento e EDA (Análise Exploratória)
✓ Leitura dos dados (CSV/Excel/API)
✓ Conversão de datetime e indexação temporal
✓ Descrição estatística (mean, std, min, max)
✓ Visualização da série completa
✓ DECOMPOSIÇÃO SAZONAL
  - Tendência
  - Sazonalidade
  - Resíduos
Objetivo: Entender padrões, sazonalidade e possíveis anomalias.

2️⃣ Divisão Temporal (Train / Validation / Test)
Split cronológico (SEM embaralhamento):
├─ Treino:      50% dos dados (início da série)
├─ Validação:   25% dos dados (seleção de hiperparâmetros)
└─ Teste:       25% dos dados (avaliação final)

Por que cronológico?
Simula cenário real: treinar no passado, prever o futuro.

3️⃣ Normalização (MinMaxScaler [0.1, 0.9])
scaler.fit(treino)  # ← Ajusta APENAS no treino
teste_normalizado = scaler.transform(teste)  # ← Reutiliza
Por que [0.1, 0.9]?

Evita extremos [0, 1] (redes neurais lidam melhor com margem)
Estável numericamente

4️⃣ Modelo 1 — ARIMA (Box & Jenkins)
4.1 Teste de Estacionariedade
Teste Augmented Dickey-Fuller (ADF):
├─ p-valor ≤ 0.05  →  Estacionária ✓
└─ p-valor > 0.05  →  NÃO estacionária  ✗  →  Diferençar (d++)

4.2 Análise ACF/PACF
ACF (Autocorrelation Function)  →  estima q (Moving Average)
PACF (Partial ACF)               →  estima p (Autoregressive)

4.3 Grid Search por AIC
Para cada (p, q) em [0, 1, 2, 3]:
  └─ Ajustar ARIMA(p, d, q)
     └─ Calcular AIC
        └─ Manter menor AIC
Métrica: AIC (Akaike Information Criterion) penaliza complexidade.

4.4 Diagnóstico de Resíduos
✓ Série temporal de resíduos (deve ser ~zero)
✓ ACF dos resíduos (sem correlação)
✓ Q-Q Plot (normalidade)
✓ Ljung-Box (teste de ruído branco)

4.5 Previsão One-Step-Ahead
Treino:  fitted_values do modelo ajustado
Teste:   janela expansiva (expanding window)
         └─ A cada passo: re-ajustar ARIMA + prever 1 passo
            └─ Adicionar valor real ao histórico
               └─ Próximo passo
Resultado: 1 previsão por passo de tempo (foco em curto prazo).

5️⃣ Modelo 2 — MLP (Multilayer Perceptron)
5.1 Janelamento Deslizante (Sliding Window)
Série:  [s₀, s₁, s₂, s₃, ..., sₙ]
Janela=3:
  ├─ X[0] = [s₀, s₁, s₂],      y[0] = s₃
  ├─ X[1] = [s₁, s₂, s₃],      y[1] = s₄
  └─ X[2] = [s₂, s₃, s₄],      y[2] = s₅

Tamanho de janela:
  ├─ AirPassengers: 12 (período sazonal mensal)
  ├─ Varejo PE:     12 (período sazonal mensal)
  └─ CO₂:           26 (meio período sazonal semanal)

5.2 Random Search (Seleção Validação)
Grid de hiperparâmetros:
├─ hidden_layer_sizes: [(50,), (100,), (50,50), (100,50), (100,100)]
├─ activation:         ['relu', 'tanh']
└─ learning_rate:      [0.001, 0.01]

Para cada config:
  ├─ Treinar 3 seeds diferentes
  ├─ Calcular MSE no conjunto de validação (média das 3)
  └─ Guardar melhor config

5.3 Avaliação
Métricas em [Treino, Validação, Teste]:
├─ MSE:  mean_squared_error
└─ MAPE: mean_absolute_percentage_error

6️⃣ Modelo 3 — LSTM (Long Short-Term Memory)
6.1 Reshape para 3D
MLP:   [batch_size, features]           → [64, 12]
LSTM:  [batch_size, timesteps, features] → [64, 12, 1]

6.2 Busca de Hiperparâmetros LSTM
Configs:
├─ units:  [32, 64, 128]
├─ layers: [1, 2]
├─ dropout: [0.1, 0.2, 0.3]
└─ batch_size: [16, 32]

Treino:
├─ Otimizador: Adam
├─ Loss: MSE
├─ Early Stopping: monitor=val_loss, patience=10
└─ Epochs: até 100 (parado antecipadamente)

6.3 Avaliação
Mesmas métricas que MLP

📊 Comparação Final dos Modelos
Cada notebook produz uma tabela comparativa:

COMPARAÇÃO DOS MODELOS — [Dataset]
╔═════════╦═════════════╦════════════╦═════════╦═════════════╗
║ Modelo  ║ MSE Treino  ║ MAPE Treino║ MSE Test║  MAPE Teste ║
╠═════════╬═════════════╬════════════╬═════════╬═════════════╣
║ ARIMA   ║   0.00523   ║   1.15%    ║ 0.00891 ║   2.34%     ║
║ MLP     ║   0.00421   ║   0.98%    ║ 0.00756 ║   1.89%     ║
║ LSTM    ║   0.00389   ║   0.87%    ║ 0.00712 ║   1.76%     ║
╚═════════╩═════════════╩════════════╩═════════╩═════════════╝

Visualizações Comparativas
1.Curvas de Previsão (gráfico de linha)
Original         (preto, contínuo)
Previsão ARIMA   (vermelho, tracejado)
Previsão MLP     (laranja, tracejado)
Previsão LSTM    (roxo, tracejado)

2.Barplot de MSE (comparação direta no teste)
└─ Quem tem a barra mais curta vence

3.Barplot de MAPE (erro percentual médio no teste)
└─ Métrica mais intuitiva para negócios

🎓 Roteiro para Apresentação
Slide 1 — Título & Contexto
Deep Learning para Séries Temporais
├─ 3 Datasets públicos
├─ 3 Modelos (ARIMA, MLP, LSTM)
└─ Comparação sistemática

Slide 2 — Séries Utilizadas
┌─────────────────────────────────────────────────────────┐
│        Dataset      │ Tamanho │ Frequência │ Período    │
├─────────────────────┼─────────┼────────────┼────────────┤
│ AirPassengers       │ 144 obs │  Mensal    │ 12 anos    │
│ Varejo PE           │ 189 obs │  Mensal    │ 15 anos    │
│ CO₂ Atmosférico     │ 2284 obs │ Semanal   │ 43 anos    │
└─────────────────────────────────────────────────────────┘

Variação: Pequeno (144) → Médio (189) → Grande (2284)

Slide 3 — Pipeline Único (EDA → Split → Normalização)
1. Carregamento + EDA
   └─ Decomposição, tendência, sazonalidade

2. Divisão Temporal (50/25/25)
   └─ Treino | Validação | Teste

3. Normalização [0.1, 0.9]
   └─ Evita extremos, mantém escala

Slide 4 — ARIMA (Metodologia)
Passo 1: Teste ADF
  └─ "A série é estacionária?"

Passo 2: ACF/PACF
  └─ "Quantos parâmetros p, d, q?"

Passo 3: Grid Search
  └─ "Qual combinação minimiza AIC?"

Passo 4: Diagnóstico
  └─ "Os resíduos são ruído branco?"

Passo 5: Previsão
  └─ One-step-ahead com janela expansiva

Slide 5 — MLP (Metodologia)
Passo 1: Janelamento Deslizante
  └─ Transformar série em (X, y) supervisionados

Passo 2: Random Search
  └─ Grid de arquiteturas + hiperparâmetros
  └─ Validar em conjunto de validação

Passo 3: Avaliar em Teste
  └─ MSE, MAPE

Slide 6 — LSTM (Metodologia)
Mesmo que MLP, mas:
├─ Reshape para 3D [batch, timesteps, features]
├─ Camadas LSTM (recorrentes)
├─ Early Stopping
└─ Captura dependências de longo prazo

Slide 7 — Comparação: Tabela
(Mostrar tabela consolidada)
Melhor modelo por MSE em TESTE

Slide 8 — Comparação: Gráficos
Mostrar:
1. Curvas de previsão (qual segue melhor o real?)
2. Barplot MSE (qual é mais preciso?)
3. Barplot MAPE (qual tem menor erro percentual?)

Slide 9 — Insights & Conclusões
✓ ARIMA: Rápido, interpretável, bom baseline
✓ MLP:   Melhor que ARIMA em dados pequenos
✓ LSTM:  Melhor geral, especialmente em séries longas

Quando usar cada um?
├─ ARIMA:  dados com padrão regular + poucos dados
├─ MLP:    dados com múltiplos fatores + dados médios
└─ LSTM:   dados com dependências longas + dados grandes

📚 Referências Teóricas
ARIMA
Box, G. E., & Jenkins, G. M. (1970). Time series analysis forecasting and control.
Akaike, H. (1974). A new look at the statistical model identification.
MLP & LSTM
Goodfellow, I., Bengio, Y., & Courville, A. (2016). Deep learning.
Hochreiter, S., & Schmidhuber, J. (1997). Long short-term memory.
Graves, A. (2012). Supervised sequence labelling with recurrent neural networks.

💡 Dicas para Apresentação
Comece com os dados: Mostre gráficos das séries. "Qual é mais fácil de prever?"
Explique por que split temporal: "Não embaralhamos porque tempo importa."
ARIMA é o baseline: "É rápido, não precisa GPU, bom para comparação."
MLP vs LSTM: "Ambas são redes neurais, mas LSTM 'lembra' do passado."
Métrica MAPE é mais intuitiva: "5% de erro é melhor que 0.00523 de MSE" 😊
Contexto de negócio: "Para varejo, prever com 2% de erro é bom?"