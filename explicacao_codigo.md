# Explicação Detalhada do Código R - Análise de Família de Insumos

## 📚 1. CONFIGURAÇÃO DE PACOTES

```r
library(tidyverse)
library(openxlsx)
library(readxl)
library(dplyr)
library(lubridate)
library(ggplot2)
library(glue)
library(gt)
library(patchwork)
library(gridExtra)
library(grid)
```

**O que faz:** Carrega bibliotecas (pacotes) necessárias para o código funcionar.

**Principais pacotes:**
- `tidyverse` e `dplyr`: Manipulação de dados (filtros, transformações)
- `ggplot2`: Criação de gráficos
- `openxlsx` e `readxl`: Leitura de arquivos Excel
- `lubridate`: Manipulação de datas
- `glue`: Concatenação de strings (textos)
- `patchwork`, `gridExtra`, `grid`: Combinação de gráficos e tabelas

---

## 🎨 2. TEMA CUSTOMIZADO

```r
tema_comparacao <- theme_minimal(base_size = 12) + theme(...)
```

**O que é:** Um objeto que armazena configurações visuais do gráfico.

**O que faz:**
- Define o estilo visual (cores, tamanhos, fontes, margens)
- Baseado no `theme_minimal()` (tema minimalista do ggplot2)
- Customiza títulos, eixos, grid e margens

**Parâmetros importantes:**
- `plot.title`: Configuração do título (tamanho 14, negrito, centralizado)
- `axis.text.x`: Texto do eixo X (estados) - tamanho 9, negrito
- `axis.title.y`: Título do eixo Y - tamanho 11, negrito
- `panel.grid.major.y`: Linhas horizontais do grid (cinza claro)
- `plot.margin`: Margens do gráfico (top, right, bottom, left)

**Onde é usado:** Aplicado na função `plot_comparacao_estados()`

---

## 📊 3. FUNÇÃO: `plot_comparacao_estados()`

### **Assinatura da Função:**
```r
plot_comparacao_estados <- function(data, titulo_familia, ref_mes)
```

### **Parâmetros de Entrada:**
- `data`: DataFrame com os dados filtrados (preços por UF e item)
- `titulo_familia`: Nome da família de insumos (ex: "Aço CA")
- `ref_mes`: Mês de referência formatado (ex: "jan/2022")

### **O que a função faz:**

#### **Passo 1: Define a ordem dos estados**
```r
ordem_ufs <- c("AC","AM","AP","PA","RO","RR","TO", ...)
```
- Cria um vetor com a ordem fixa dos estados agrupados por região
- Norte (7 estados) → Nordeste (9) → Sudeste (4) → Sul (3) → Centro-Oeste (4)

#### **Passo 2: Verifica e ajusta a coluna de região**
```r
if(!"Regiao_Sigla" %in% names(data)) { ... }
```
- Verifica se existe a coluna `Regiao_Sigla`
- Se não existir, tenta encontrar a coluna `Regiao` e converter para siglas
- `case_when()`: Como um "switch case" - converte "Norte" → "N", "Nordeste" → "NE", etc.

#### **Passo 3: Prepara os dados**
```r
data <- data %>% mutate(UF_Ordenada = factor(UF, levels = ordem_ufs))
```
- `mutate()`: Cria ou modifica colunas no DataFrame
- `factor()`: Transforma a coluna UF em fator ordenado (categoria com ordem específica)
- **Por que fator?** Permite que o gráfico mostre os estados na ordem desejada

#### **Passo 4: Define informações das regiões**
```r
regioes <- tibble(
  Regiao = c("Norte","Nordeste","Sudeste","Sul","Centro Oeste"),
  n = c(7,9,4,3,4)
) %>% mutate(...)
```
- `tibble()`: Cria um DataFrame (tabela)
- Calcula posições para os rótulos de região:
  - `start`: Posição inicial de cada região
  - `end`: Posição final
  - `mid`: Posição central (onde o nome da região aparecerá)
- `cumsum()`: Soma cumulativa (ex: 7, 16, 20, 23, 27)

#### **Passo 5: Cria o gráfico base**
```r
grafico <- ggplot(data, aes(x = UF_Ordenada, y = Preco_Unitario, ...)) +
  geom_line() + geom_point() + labs(...) + scale_y_continuous(...)
```

**Funções do ggplot2:**
- `ggplot()`: Inicializa o gráfico
- `aes()`: Define aesthetics (mapeamento visual): x=estados, y=preços, cor=item, forma=item
- `geom_line()`: Adiciona linhas conectando os pontos
- `geom_point()`: Adiciona pontos nos dados
- `labs()`: Define títulos e rótulos
- `scale_y_continuous()`: Configura o eixo Y
  - `limits`: Define limites do eixo (0 até 150% do preço máximo)
  - `breaks`: Define onde aparecem os números (de 2 em 2)
  - `labels`: Formata números (vírgula decimal, ponto milhar)
- `scale_color_manual()`: Define cores para cada item manualmente
- `scale_shape_manual()`: Define formas dos pontos (círculo=16, quadrado=15, etc.)

#### **Passo 6: Adiciona linhas verticais separadoras**
```r
vlines_pos <- regioes$end[-nrow(regioes)] + 0.5
grafico <- grafico + geom_vline(xintercept = vlines_pos, ...)
```
- Calcula posições após o último estado de cada região
- `geom_vline()`: Adiciona linha vertical
- `-nrow(regioes)`: Remove a última região (não precisa de linha após a última)

#### **Passo 7: Adiciona rótulos de região**
```r
grafico <- grafico + annotation_custom(grob = textGrob(...), ...)
```
- `textGrob()`: Cria um objeto de texto (da biblioteca `grid`)
- `annotation_custom()`: Adiciona anotação customizada no gráfico
- Parâmetros:
  - `xmin/xmax`: Posição horizontal (centralizada na região)
  - `ymin/ymax`: Posição vertical (valor negativo = abaixo do eixo)
  - `gp = gpar()`: Parâmetros gráficos (tamanho da fonte, negrito)

#### **Passo 8: Finaliza o gráfico**
```r
grafico <- grafico + coord_cartesian(ylim = c(0, ...), clip = "off")
```
- `coord_cartesian()`: Define sistema de coordenadas
- `clip = "off"`: Permite que elementos apareçam fora da área do gráfico (rótulos de região)

### **Retorno:**
- **Objeto:** Um gráfico ggplot2
- **Onde vai:** Retorna para a variável `grafico` na função `gera_analise_familia()`

---

## 📋 4. FUNÇÃO: `gera_tabela_estatisticas()`

### **Assinatura:**
```r
gera_tabela_estatisticas <- function(df_data)
```

### **Parâmetro:**
- `df_data`: DataFrame com dados filtrados

### **O que faz:**

#### **Passo 1: Mapeia descrições e símbolos**
```r
df_mapa <- df_data %>% select(ITEM_Lider) %>% distinct() %>% mutate(...)
```
- `select()`: Seleciona apenas a coluna ITEM_Lider
- `distinct()`: Remove duplicatas (retorna apenas valores únicos)
- `mutate()`: Adiciona colunas com descrições e símbolos Unicode
  - `\u25CF`: Círculo preenchido (●)
  - `\u25A0`: Quadrado preenchido (■)
  - `\u25B2`: Triângulo (▲)
  - `\u2716`: X (✖)

#### **Passo 2: Calcula estatísticas**
```r
df_estat <- df_data %>% group_by(ITEM_Lider) %>% summarise(...)
```
- `group_by()`: Agrupa dados por item
- `summarise()`: Calcula estatísticas por grupo
  - `mean()`: Média dos preços
  - `min()`: Preço mínimo
  - `max()`: Preço máximo
- `mutate()`: Calcula amplitude: (Máx - Mín) / Mín

#### **Passo 3: Identifica o líder**
```r
item_lider <- df_estat %>% filter(Media == min(Media)) %>% pull(ITEM_Lider)
```
- `filter()`: Filtra o item com menor média
- `pull()`: Extrai o valor como vetor (não como DataFrame)
- `first()`: Pega o primeiro elemento

#### **Passo 4: Formata a tabela**
```r
tabela_final_df <- df_estat %>% left_join(...) %>% mutate(...)
```
- `left_join()`: Une (merge) dois DataFrames pela coluna comum
- Formata números:
  - `format()`: Formata com 2 casas decimais, vírgula decimal
  - `paste0()`: Concatena texto (ex: "45%")
- `select()`: Seleciona e renomeia colunas finais

#### **Passo 5: Cria a tabela como objeto gráfico**
```r
tabela_grob <- tableGrob(tabela_final_df, ...)
```
- `tableGrob()`: Converte DataFrame em objeto gráfico (grob)
- `ttheme_minimal()`: Define tema da tabela
  - `core`: Estilo do corpo (células de dados)
  - `colhead`: Estilo do cabeçalho
  - `fg_params`: Parâmetros do texto (cor, fonte)
  - `bg_params`: Parâmetros do fundo (cor, bordas)

#### **Passo 6: Adiciona título**
```r
titulo_tabela <- textGrob("Preços e Amplitudes dos Insumos", ...)
tabela_final <- arrangeGrob(titulo_tabela, tabela_grob, heights = c(0.1, 0.9))
```
- `textGrob()`: Cria título como objeto gráfico
- `arrangeGrob()`: Organiza múltiplos grobs verticalmente
- `heights`: Proporção de espaço (10% título, 90% tabela)

### **Retorno:**
- **Objeto:** Um grob (objeto gráfico combinado: título + tabela)
- **Onde vai:** Retorna para `tabela_grob` na função `gera_analise_familia()`

---

## 🎯 5. FUNÇÃO PRINCIPAL: `gera_analise_familia()`

### **Assinatura:**
```r
gera_analise_familia <- function(
  caminho_codigos_familia,
  caminho_serie_historica,
  ref_atual
)
```

### **Parâmetros:**
- `caminho_codigos_familia`: Caminho do arquivo Excel com códigos das famílias
- `caminho_serie_historica`: Caminho do arquivo Excel com série histórica de preços
- `ref_atual`: Mês de referência (ex: "jan-22")

### **Fluxo de execução:**

#### **Etapa 1: Carrega dados**
```r
CodigosFamilia <- read.xlsx(caminho_codigos_familia, ...)
SerieHistorica <- read.xlsx(caminho_serie_historica, ...)
```
- `read.xlsx()`: Lê arquivo Excel e cria DataFrame
- `colNames = TRUE`: Primeira linha é cabeçalho
- `startRow = 1`: Começa da linha 1

#### **Etapa 2: Identifica família e insumos**
```r
familia_alvo <- CodigosFamilia %>% select(Família) %>% distinct() %>% slice_head(n = 1) %>% pull(Família)
```
- Pipeline (`%>%`): Encadeia operações
- `slice_head(n=1)`: Pega primeira linha
- Extrai lista de códigos (insumos) da família

#### **Etapa 3: Pré-processamento**
```r
df <- SerieHistorica %>% select(-tail(names(.), 8))
```
- `tail(names(.), 8)`: Pega últimas 8 colunas
- `select(-...)`: Remove essas colunas (estatísticas agregadas)

```r
df <- df %>% mutate(across(11:ncol(.), as.numeric))
```
- `across()`: Aplica função em múltiplas colunas
- `11:ncol(.)`: Da coluna 11 até a última
- `as.numeric()`: Converte para número

```r
df <- df %>% pivot_longer(cols = 11:ncol(.), ...)
```
- `pivot_longer()`: Transforma dados de largo para longo
- Converte colunas de meses em linhas (uma coluna "Mes_Str" e uma "Preco_Unitario")

```r
mutate(Mes_Data = as.Date(paste0("01-", Mes_Str), format = "%d-%b-%y"))
```
- `paste0()`: Concatena strings ("01-" + "jan-22" = "01-jan-22")
- `as.Date()`: Converte texto para data
- `format = "%d-%b-%y"`: Define formato (dia-mês-ano)

#### **Etapa 4: Filtra dados finais**
```r
df_final <- df %>% filter(ITEM %in% insumos_alvo, Mes_Data == ref_data, ...)
```
- `filter()`: Filtra linhas que atendem condições
- `%in%`: Verifica se está contido no vetor
- `!(UF %in% c(...))`: Exclui UFs agregadas (MÁX, MÍN, MÉD)

```r
rename(Regiao_Sigla = Região)
```
- `rename()`: Renomeia coluna (novo_nome = nome_antigo)

#### **Etapa 5: Gera visualizações**
```r
grafico <- plot_comparacao_estados(df_final, familia_alvo, ref_mes_formatado)
tabela_grob <- gera_tabela_estatisticas(df_final)
```
- Chama as funções criadas anteriormente
- Armazena os resultados

#### **Etapa 6: Combina e salva**
```r
layout <- grafico / wrap_elements(tabela_grob) + plot_layout(heights = c(0.75, 0.25))
```
- `wrap_elements()`: Converte grob em elemento patchwork
- `/`: Operador do patchwork - empilha verticalmente
- `plot_layout()`: Define proporções (75% gráfico, 25% tabela)

```r
ggsave(caminho_saida, layout, width = 15, height = 10, dpi = 300)
```
- `ggsave()`: Salva gráfico em arquivo
- Parâmetros:
  - `width`, `height`: Dimensões em polegadas
  - `dpi`: Resolução (dots per inch)

### **Retorno:**
- **Objeto:** Layout combinado (gráfico + tabela)
- **Arquivo:** PNG salvo em disco
- **Onde vai:** Para a variável `resultado_analise` no bloco de execução

---

## 🚀 6. BLOCO DE EXECUÇÃO

```r
resultado_analise <- gera_analise_familia(
  caminho_codigos_familia = "C:/Users/.../CódigosFamilia.xlsx",
  caminho_serie_historica = "C:/Users/.../Série Histórica.xlsx",
  ref_atual = "jan-22"
)
print(resultado_analise)
```

**O que faz:**
1. Chama a função principal com os parâmetros
2. Armazena resultado em `resultado_analise`
3. `print()`: Exibe o gráfico na tela do RStudio

---

## 📊 FLUXO COMPLETO DE DADOS

```
Arquivos Excel
    ↓
gera_analise_familia() → Lê e processa dados
    ↓
df_final (DataFrame filtrado)
    ↓
    ├→ plot_comparacao_estados() → grafico (objeto ggplot)
    └→ gera_tabela_estatisticas() → tabela_grob (objeto grob)
    ↓
layout (patchwork: grafico + tabela)
    ↓
ggsave() → Arquivo PNG salvo
    ↓
return(layout) → Visualização no RStudio
```

---

## 🔑 CONCEITOS-CHAVE

### **Pipeline (%>%)**
Encadeia operações sequenciais. Equivalente a:
```r
# Com pipeline
df %>% filter(x > 5) %>% mutate(y = x * 2)

# Sem pipeline (aninhado)
mutate(filter(df, x > 5), y = x * 2)
```

### **Factor (Fator)**
Tipo de dado categórico com níveis ordenados. Usado para controlar ordem em gráficos.

### **Grob (Graphical Object)**
Objeto gráfico de baixo nível da biblioteca `grid`. Permite criar elementos customizados.

### **DataFrame vs Tibble**
- DataFrame: Estrutura básica do R (tabela de dados)
- Tibble: Versão moderna do tidyverse (mais consistente)

### **Aesthetic Mapping (aes)**
Define como dados são mapeados para elementos visuais (x, y, cor, forma, tamanho)

---

## 💡 RESUMO

Este código:
1. **Lê** dados de arquivos Excel
2. **Processa** e filtra para uma família específica
3. **Cria** um gráfico comparativo entre estados e regiões
4. **Gera** uma tabela de estatísticas
5. **Combina** gráfico e tabela
6. **Salva** como arquivo PNG de alta resolução

É um exemplo de **análise de dados end-to-end** em R, desde a importação até a visualização final.