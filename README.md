<h1 align="center">💧 Análise de Desempenho de Hidrômetros</h1>

<p align="center">
  <strong>Integração, análise estatística e análise crítica de dados com R</strong><br/>
  Desafio Técnico · 50 registros · Período 2017–2026
</p>

<p align="center">
  <img src="https://img.shields.io/badge/R-4.5.3-276DC3?logo=r&logoColor=white" alt="R 4.5.3"/>
  <img src="https://img.shields.io/badge/RStudio-IDE-75AADB?logo=rstudioide&logoColor=white" alt="RStudio"/>
  <img src="https://img.shields.io/badge/tidyverse-ggplot2%20%7C%20dplyr-1A162D?logo=tidyverse&logoColor=white" alt="tidyverse"/>
  <img src="https://img.shields.io/badge/Dados-Excel%20(.xlsx)-217346?logo=microsoftexcel&logoColor=white" alt="Excel"/>
</p>

---

## 📑 Sumário

1. [Visão geral](#-visão-geral)
2. [Resumo dos resultados](#-resumo-dos-resultados)
3. [Fontes de dados e integração](#-fontes-de-dados-e-integração)
4. [Como reproduzir](#-como-reproduzir)
5. [Estatísticas descritivas](#-estatísticas-descritivas)
6. [Dispersão: Volume × IDM](#-dispersão-volume--idm)
7. [Boxplots](#-boxplots)
8. [Análise crítica](#-análise-crítica)
9. [Limitações](#-limitações)
10. [Conclusão e recomendações](#-conclusão-e-recomendações)

---

## Visão geral

Este repositório apresenta uma análise técnica sobre a base de hidrômetros fornecida em Excel. O trabalho contempla:

- **Integração** de duas bases de dados
- **Análise estatística descritiva** das variáveis principais
- **Análise crítica** do comportamento dos equipamentos em campo

### Variáveis analisadas

| Variável | Descrição |
|---|---|
| **IDM (%)** | Índice de Desempenho Metrológico: expressa a precisão de medição do hidrômetro em relação a um padrão de referência. 100% = medição exata |
| **Volume (m³)** | Volumetria total: soma dos m³ registrados pelo equipamento desde a instalação |

---

## Resumo dos resultados

| Indicador | Resultado |
|---|---|
| Registros consolidados | **50** |
| IDM médio | **98,85%** |
| Hidrômetros com IDM acima de 97,75% | **75%** |
| Correlação de Pearson (Volume × IDM) | **r ≈ −1,00** |
| Modelo linear estimado | `IDM ≈ 101,12 − 0,0033 × Volume` |
| Caso crítico | **A21L060170**, IDM = 88,70% (volume = 3.787 m³) |
| Equipamentos sob atenção preventiva | **6** (IDM entre 93% e 97%) |

---

## Fontes de dados e integração

Dois arquivos `.xlsx` foram disponibilizados:

| Arquivo | Colunas |
|---|---|
| `Arquivo1.xlsx` | `Matrícula`, `HD.Instalado`, `Data de Instalação`, `Volume` |
| `Arquivo2.xlsx` | `Matrícula`, `HD.Instalado`, `IDM` |

### Estratégia de integração

Foi feito um **inner join** com chave composta (`Matrícula` + `HD.Instalado`). Assim, só entram na análise os registros com correspondência exata nas duas bases; entradas sem par são descartadas.

```
Arquivo1 ──┐
           ├── inner join (Matrícula + HD.Instalado) ──▶ Base consolidada (50 registros)
Arquivo2 ──┘
```

### Ferramentas

| Ferramenta | Uso |
|---|---|
| **R 4.5.3** | Linguagem de análise |
| `readxl` | Leitura dos arquivos `.xlsx` |
| `tidyverse` | Manipulação de dados e gráficos |
| `patchwork` | Composição de gráficos |
| **RStudio** | Desenvolvimento e visualização |

---

## Como reproduzir

1. Instale os pacotes:

```r
install.packages(c("tidyverse", "readxl", "patchwork"))
```

2. Coloque `Arquivo1.xlsx` e `Arquivo2.xlsx` na pasta de trabalho e execute o pipeline abaixo (trecho ilustrativo; ajuste nomes de colunas e caminhos conforme o seu script):

```r
library(tidyverse)
library(readxl)
library(patchwork)

a1 <- read_excel("Arquivo1.xlsx")
a2 <- read_excel("Arquivo2.xlsx")

base <- inner_join(a1, a2, by = c("Matrícula", "HD.Instalado"))

# Estatísticas descritivas
base |>
  summarise(across(c(IDM, Volume),
                   list(min = min, media = mean, max = max, dp = sd)))

# Correlação de Pearson
cor.test(base$Volume, base$IDM)

# Dispersão com linha de tendência
ggplot(base, aes(Volume, IDM)) +
  geom_point() +
  geom_smooth(method = "lm")
```

---

## Estatísticas descritivas

Base consolidada com 50 registros únicos:

<div align="center">

| Variável | Mínimo | Média | Máximo | Desvio padrão |
|---|---|---|---|---|
| **IDM (%)** | 88,70 | 98,85 | 101,10 | 2,34 |
| **Volume (m³)** | 5 | 693,46 | 3.787 | 713,98 |

</div>

> O desvio padrão do volume (713,98) é maior que sua média (693,46), sinal de grande dispersão e de assimetria à direita.

---

## Dispersão: Volume × IDM

O gráfico de dispersão mostra a relação entre a volumetria acumulada e o IDM de cada hidrômetro, com linha de tendência.

> **Resultado:** correlação de Pearson **r ≈ −1,00**.
> Quanto maior o volume acumulado, menor o IDM, em uma relação praticamente linear. Cada **100 m³** adicionais estão associados a uma queda de cerca de **0,33 ponto percentual** no IDM.

Esse comportamento é compatível com a premissa técnica de desgaste dos componentes internos ao longo do uso. Veja as [limitações](#-limitações) sobre como interpretar essa relação.

---

## Boxplots

Os boxplots mostram mediana, quartis e valores atípicos das duas variáveis.

> **Resultados:**
> - **IDM:** 75% dos hidrômetros têm IDM acima de **97,75%**, com um único outlier em **88,70%**.
> - **Volume:** distribuição assimétrica à direita, com mediana de **486,5 m³** e outlier em **3.787 m³**.
> - Os dois outliers pertencem ao **mesmo equipamento**.

---

## Análise crítica

### Padrão identificado

Hidrômetros com maior volume acumulado tendem a apresentar menor IDM. É um comportamento esperado do ponto de vista metrológico, pois o desgaste mecânico compromete gradualmente a precisão da medição.

### Equipamentos em destaque

| Matrícula | Hidrômetro | Volume (m³) | IDM (%) | Situação |
|---|---|---|---|---|
| **1203971-3** | **A21L060170** | **3.787** | **88,70** | 🔴 Crítico |
| 1203756-7 | A19L073882 | 2.243 | 93,77 | 🟠 Atenção |
| 1203968-3 | A18L731516 | 2.152 | 94,06 | 🟠 Atenção |
| 1203753-2 | A21L266357 | 1.777 | 95,29 | 🟠 Atenção |
| 1203851-2 | A21L264269 | 1.574 | 95,96 | 🟠 Atenção |
| 1203671-4 | A21L120897 | 1.467 | 96,31 | 🟠 Atenção |
| 1203702-8 | A20L169855 | 1.331 | 96,76 | 🟠 Atenção |

### O caso crítico: A21L060170

O hidrômetro **A21L060170** (matrícula 1203971-3) tem o maior volume acumulado (3.787 m³) e o pior IDM da base (88,70%), caracterizando **submedição expressiva**: o equipamento deixa de registrar cerca de **11,3% do volume real consumido**.

Estimativa de impacto (assumindo IDM = volume medido ÷ volume real):

| Item | Valor |
|---|---|
| Volume registrado | 3.787 m³ |
| Volume real estimado | ≈ 4.269 m³ |
| Volume não medido | **≈ 482 m³** |

> Este equipamento é um outlier estatístico, mas **não é uma inconsistência**: ele se alinha à mesma reta dos demais. Trata-se do ponto mais extremo da tendência, o que reforça a hipótese de degradação por uso.

### Há relação entre volumetria e IDM?

> **Sim.** Com r ≈ −1,00 (r² ≈ 1), praticamente toda a variação do IDM nesta amostra é explicada linearmente pela volumetria acumulada.

---

## ⚠️ Limitações

Estes pontos ajudam a interpretar os resultados com cuidado:

- **Correlação não prova causalidade.** O volume acumulado também reflete o **tempo de uso**; os dados sozinhos não separam desgaste por volume de desgaste por idade. A coluna `Data de Instalação` permitiria testar isso (ex.: IDM × anos em campo, ou regressão com as duas variáveis).
- **Correlação quase perfeita é incomum em dados reais.** Em campo, fatores como qualidade da água, pressão, vazão típica e modelo do medidor costumam adicionar ruído. Vale confirmar a origem dos dados e se há alguma relação construída entre as variáveis.
- **Amostra pequena (n = 50)** e sem indicação de como foi selecionada, o que limita a generalização para toda a frota.
- **Significância estatística:** convém reportar o valor-p e o intervalo de confiança de `cor.test()`, em vez de apenas o coeficiente.
- **IDM acima de 100%** (máximo de 101,10%) indica leve superestimação em alguns equipamentos, caso inverso da submedição, e merece ao menos uma menção na análise.

---

## Conclusão e recomendações

A integração das duas bases foi feita com **inner join** (`Matrícula + HD.Instalado`), resultando em **50 registros consolidados**.

O conjunto tem desempenho globalmente satisfatório:

- **IDM médio:** 98,85%
- **75% dos equipamentos** acima de 97,75%

Foram identificados:

- **1 caso crítico** de submedição (A21L060170, IDM = 88,70%), que requer **intervenção imediata**
- **6 equipamentos** com IDM entre 93% e 97%, que merecem **atenção preventiva**

A relação negativa entre volumetria acumulada e IDM indica que o **envelhecimento dos hidrômetros** é o principal fator de perda de precisão. Isso pode embasar um **programa de substituição preditiva**, por exemplo definindo um limite de volume acumulado para troca (pela equação estimada, um IDM de 95% corresponde a cerca de 1.870 m³), reduzindo perdas comerciais e garantindo a acurácia do faturamento.

### Próximos passos sugeridos

- [ ] Incluir `Data de Instalação` na análise (tempo em campo)
- [ ] Reportar valor-p e intervalo de confiança da correlação
- [ ] Ajustar regressão linear e analisar os resíduos
- [ ] Segmentar por modelo/lote de fabricação do hidrômetro
- [ ] Validar o limite de troca em uma amostra maior

---

<p align="center">
  Desafio Técnico de Análise de Dados · feito com R 📊
</p>
