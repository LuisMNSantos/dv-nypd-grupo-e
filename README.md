# Visualização de Dados 2026/27 — Projeto NYPD (Grupo E)

Projeto da UC de **Visualização de Dados** (M:DS, FCUP, 2026/27). Usa os dados de queixas da polícia de Nova Iorque (**NYPD Complaint Data**) para produzir visualizações que ajudem o **Mayor de Nova Iorque e o chefe da NYPD** a reduzir o número de incidentes e a tornar a cidade mais segura.

> O público-alvo tem pouco tempo e não é da área de ciência de dados. Cada figura deve mostrar **um padrão** e a **decisão que esse padrão suporta** (onde e quando reforçar patrulhas, que tipos de local precisam de prevenção, que infrações estão a crescer e onde).

## Regras importantes

- Tudo em **R**, com as ferramentas da UC: `ggplot2`, `dplyr`, `plotly` e `Shiny`.
- O projeto cresce por fases: exploração (ggplot2) → hipóteses → interação (plotly) → dashboard (Shiny).
- **Não** usar código de outros grupos. Citar todas as fontes externas.
- **IA generativa**: só para (1) melhorar texto escrito pelo grupo, (2) melhorar ou corrigir o código de uma figura já feita pelo grupo e (3) manipulação de dados (dplyr/tidyr). **Não** pode criar figuras de raiz, explorar o dataset, formular perguntas ou hipóteses, nem interpretar resultados. O uso tem de ser declarado todas as semanas no *progress check*.
- Todas as legendas das figuras citam a fonte: *"Data: NYPD Complaint Data, NYC Open Data (complaints reported from January 2020 to June 2026)"*.

## Calendário e entregas

Todas as sextas-feiras até às **23:30** (23:30–23:59 conta como atraso; depois não é aceite): **ZIP da pasta do projeto** (cumulativo, **sem o ficheiro de dados**) + **progress check** no Moodle.

| Semana | Prazo (sexta) | Entrega |
|---|---|---|
| 1 | 30 out | ZIP 1 |
| 2 | 6 nov | **C1 — Perguntas e exploração**: perguntas de investigação, exploração em ggplot2 (scripts + figuras) e proposta **PPA** (PDF, máx. 4 páginas) |
| 3 | 13 nov | ZIP 3 |
| 4 | 20 nov | **C2 — Hipóteses**: cada hipótese com a figura que a suporta ou refuta; hipóteses descartadas e porquê |
| 5 | 27 nov | **C3 — Interação**: figuras interativas em `plotly`, cada uma com uma frase sobre o que a interação acrescenta |
| 6 | 4 dez | ZIP 6 |
| 7 | 11 dez | **C4 — Dashboard e artigo**: app Shiny, artigo **PD** (formato IEEE, máx. 6 páginas, PDF), slides/screenshots e todo o código |
| — | **14 dez** | **Apresentação final**: 12 min (com demo ao vivo do Shiny) + 5 min de discussão. O limite é estrito. |

**Avaliação (60 % da nota da UC):** PPA 10 pts · Artigo PD 25 pts · Apresentação 25 pts.

Os nomes das figuras devem manter-se de semana para semana (uma figura melhorada mantém o nome; só figuras novas têm nomes novos).

## Os dados

Ficheiro `data/nypd_2020_2026.rds` (não está no repositório; descarregar do Moodle e colocar em `data/`).

- **3 386 974 queixas × 39 colunas**, reportadas entre 1 jan 2020 e 30 jun 2026 (uma linha = uma queixa).
- Fonte: NYC Open Data, *NYPD Complaint Data Historic* (qgea-i56i) e *Current YTD* (5uac-w243).
- Colunas por tema:
  - **Quando**: `reported_on`, `occurred_on`, `occurred_at`, `hour`, `wday`, `report_year`, `old_occurrence` (colunas derivadas, a usar em vez das de texto)
  - **Onde**: `boro_nm`, `addr_pct_cd`, `patrol_boro`, `latitude`/`longitude`, `prem_typ_desc` (tipo de local), `loc_of_occur_desc`, `parks_nm`, `station_name`, …
  - **O quê**: `law_cat_cd` (felony/misdemeanor/violation), `ofns_desc`, `pd_desc`, `crm_atpt_cptd_cd`, `juris_desc`
  - **Quem**: `susp_*` e `vic_*` (idade, raça, sexo)

Cuidados (ver o dicionário de dados e o feedback do C1):

- Valores em falta aparecem como `NA` **e** como o texto literal `"(null)"` ou `UNKNOWN` — contar os dois.
- 2026 está incompleto (só até junho): para análises sazonais usar só anos completos (2020–2025).
- `old_occurrence == TRUE` (16 630 linhas) mistura participações tardias com datas impossíveis.
- Horas redondas (`00:00`, `12:00`) podem ser horas desconhecidas.
- Em `vic_sex`, `D` = empresa/organização e `E` = Estado de Nova Iorque (não são pessoas).
- A raça é a registada pela polícia, não autoidentificação.
- São contagens de queixas, não taxas: sem dados de população não se fala de risco.

Leitura:

```r
library(dplyr)
nypd <- as.data.frame(readRDS("data/nypd_2020_2026.rds"))
nrow(nypd)  # deve dar 3386974
```

## Estado atual do projeto

### Checkpoint 1 (PPA) — 11/55

A proposta explorava quatro tipos de local (ruas, residências, lojas *chain store*, espaços públicos), mas **não tinha hipóteses**. Pontos principais do feedback (ver `docs/feedback/checkpoint1/`):

- Faltavam hipóteses, transformações dos dados descritas e o que o Mayor/NYPD poderiam fazer.
- Figuras 3 e 4 não foram feitas em ggplot2 nem tinham código no ZIP (−8 pts); 4 de 10 figuras ilegíveis em papel (−4,8 pts).
- Os `"(null)"` não foram contados como valores em falta; o gráfico sazonal somava meses de 2020 a 2026.
- Horas de trabalho declaradas muito abaixo do mínimo.
- Pistas de hipóteses sugeridas pelo docente:
  1. A subida de *petit larceny* nas residências em dezembro corresponde a furtos de encomendas.
  2. Cada tipo de espaço público tem a sua ofensa dominante e a sua hora de pico (comparar em %).
  3. Nas *chain stores*, as queixas com vítima pessoa crescem ao mesmo ritmo que os furtos contra a empresa.

### Análise atual: crime de rua e vítimas (`R/trabalho_vd.ipynb`)

**Pergunta central:** Quem são as vítimas de crime de rua em Nova Iorque, e isto muda ao longo do tempo e pela cidade?

- **Hipótese descartada:** os crimes de rua estão distribuídos de forma igual entre grupos de vítimas e locais (qualquer resultado a responde e, sem população, não se interpretam contagens).
- **Hipótese testada:** os homens são a maioria das vítimas em todas as infrações de rua → **rejeitada**: são ~6 em cada 10 vítimas no total e a maioria em 7 das 8 infrações principais, mas em *Harassment 2* as mulheres são 55 %.

| Figura | Conteúdo |
|---|---|
| `fig/figura_rua_1_sexo_vitimas.png` | Vítimas homens vs. mulheres (barras) |
| `fig/figura_rua_2_infracoes.png` | % homens/mulheres nas 8 infrações mais frequentes (barras 100 %) |
| `fig/figura_rua_3_suspeitos_harassment2.png` | *Harassment 2*: sexo do suspeito por sexo da vítima |
| `fig/figura_rua_4_distritos_harassment2.png` | *Harassment 2*: queixas por distrito e sexo da vítima (lollipop) |
| `fig/figura_rua_5…8_*.png` | *Harassment 2* com suspeito homem: raça de vítimas e suspeitos por distrito (mapas de calor, opcionais) |

> As imagens em `fig/` foram extraídas dos outputs guardados no notebook (resolução de ecrã). Ao correr o notebook, o `ggsave()` gera as versões a 300 dpi com os mesmos nomes.

### Por fazer

- [ ] **C3 (27 nov):** versões interativas em `plotly` das figuras escolhidas + uma frase por figura sobre o que a interação acrescenta.
- [ ] Juntar a proposta PPA (PDF) em `docs/`.
- [ ] **C4 (11 dez):** app Shiny em `app/` e artigo PD (IEEE, PDF) em `docs/`.
- [ ] Slides/screenshots da apresentação de 14 dez.

## Estrutura do repositório

```
.
├── R/                      scripts e notebooks (.R, .Rmd, .qmd, .ipynb)
│   └── trabalho_vd.ipynb   análise de crime de rua (kernel R, feito no Colab)
├── fig/                    figuras produzidas pelos scripts
├── app/                    app Shiny (a partir do checkpoint 4)
├── docs/
│   ├── enunciado/          enunciado, dicionário de dados, protocolo e resumo de prazos
│   └── feedback/checkpoint1/  feedback e classificação do PPA (fotografias)
├── data/                   nypd_2020_2026.rds — NÃO vai para o Git nem para o ZIP
└── README.md
```

## Como correr

O notebook foi escrito para o **Google Colab** (kernel R) e lê o ficheiro de dados e grava as figuras na pasta de trabalho. Para correr localmente:

1. Instalar os pacotes: `install.packages(c("dplyr", "ggplot2", "scales", "lubridate", "plotly", "shiny"))`.
2. Colocar `nypd_2020_2026.rds` em `data/`.
3. No notebook, mudar o caminho de leitura para `"../data/nypd_2020_2026.rds"` e os `ggsave()` para `"../fig/…"` (ou, no Colab, carregar o `.rds` para a pasta de ficheiros, como até agora).

## Preparar o ZIP semanal

Comprimir a pasta do projeto **sem** a pasta `data/`, por exemplo a partir do Git (só inclui ficheiros versionados):

```bash
git archive --format=zip -o entrega_semana_N.zip HEAD
```
