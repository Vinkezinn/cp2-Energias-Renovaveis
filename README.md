# CP2 — APIs, energias renováveis e aprendizado de máquina

## Objetivo
Consultar duas APIs públicas do setor de energias renováveis e resolver duas tarefas independentes de aprendizado de máquina, comparando **três algoritmos em cada uma**:

1. **Classificação:** prever a fonte de um empreendimento (Solar, Eólica ou Hidráulica) a partir de potência outorgada e localização.
2. **Regressão:** estimar a radiação solar horária em Petrolina (PE) a partir de variáveis atmosféricas e da hora do dia.

## Origem e período dos dados
| Tarefa | Fonte | Período / observações |
|---|---|---|
| 1 | [SIGA — ANEEL](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel) (API CKAN, sem token) | Cadastro atual de empreendimentos; 3.876 linhas válidas (1.200 Solar, 1.200 Eólica, 1.476 Hidráulica). As quantidades vêm do `limit=1200` da consulta e **não** refletem a matriz energética brasileira. |
| 2 | [API histórica Open-Meteo](https://open-meteo.com/en/docs/historical-weather-api) (sem token) | Petrolina (−9,39; −40,50), **01/04/2025 a 30/06/2025**, fuso `America/Recife`, horas de 7h a 17h; 1.001 linhas. Dados estimados por modelo/reanálise, não medidos. |

## Arquivos
| Arquivo | Conteúdo |
|---|---|
| `CP2_Energias_Renovaveis.ipynb` | Notebook completo, já executado, com análise, modelos, métricas, gráficos e interpretação |
| `aneel_classificacao_orange.csv`, `meteo_regressao_orange.csv` | CSVs gerados pelas APIs |
| `meteo_treino_orange.csv`, `meteo_teste_orange.csv` | Divisão temporal 80/20 (800 / 201 linhas) da Tarefa 2, a mesma usada no notebook, disponível para reproduzir a comparação em outras ferramentas |
| `Fluxos_para_Classificacao_e_Regressao.ows` | Fluxo do Orange com as duas tarefas (material complementar; a avaliação principal está no notebook) |

## Como executar
1. Clone o repositório.
2. Instale as dependências: `pip install pandas numpy matplotlib scikit-learn jupyter`.
3. Abra `CP2_Energias_Renovaveis.ipynb` e execute as células **na ordem**. Os dois CSVs na mesma pasta são lidos diretamente; se estiverem ausentes, a seção 0 os gera consultando as APIs (precisa de internet, nenhum token é necessário).

## Configuração de avaliação
- **Classificação:** divisão 80% treino / 20% teste, **estratificada**, `random_state=42`; padronização (`StandardScaler`) dentro de um `Pipeline`, ou seja, ajustada só no treino, para Regressão Logística e KNN. Métricas por classe com média **macro**.
- **Regressão:** **primeiras 80% das horas** para treino (01/04 a 12/06) e **últimas 20%** para teste (12/06 a 30/06), sem embaralhar. Entradas: temperatura, umidade, nuvens, vento e hora (`data_hora` e `radiacao_w_m2` fora de `X`).

## Resultados

### Tarefa 1 — Classificação (conjunto de teste, 776 linhas)
| Algoritmo | Accuracy | Precision (macro) | Recall (macro) | F1 (macro) |
|---|---|---|---|---|
| Regressão Logística | 0,825 | 0,828 | 0,821 | 0,820 |
| KNN (k=5) | 0,965 | 0,966 | 0,964 | 0,965 |
| Random Forest (200 árvores) | **0,976** | **0,977** | **0,974** | **0,975** |

### Tarefa 2 — Regressão (conjunto de teste, 201 horas)
| Algoritmo | MAE (W/m²) | MSE ((W/m²)²) | R² |
|---|---|---|---|
| Regressão Linear | 145,2 | 30.034 | 0,360 |
| Árvore de Decisão (profundidade 6) | 90,9 | 15.127 | 0,678 |
| Random Forest (200 árvores) | **66,3** | **7.251** | **0,845** |

## Conclusões

**Classificação.** O Random Forest foi o melhor (Accuracy ≈ 97,6%), seguido de perto pelo KNN; a Regressão Logística ficou em ≈ 82% por usar fronteiras lineares em um problema não linear. A maior confusão da Regressão Logística é Solar prevista como Eólica; nos outros dois modelos, o erro mais frequente envolve Solar e Hidráulica. Limitações: a potência outorgada não é energia gerada e o cadastro mistura empreendimentos em fases diferentes; há coordenadas ausentes (valor 0) e muitas usinas solares com potência de 1 kW ou menos (provável valor padrão); e, como parques eólicos e complexos hidráulicos têm várias unidades muito próximas, a divisão aleatória favorece KNN e Random Forest, então a acurácia provavelmente é otimista para locais novos. Remover as 28 linhas duplicadas quase não alterou os resultados.

**Regressão.** O Random Forest foi o melhor (R² ≈ 0,85; erro médio de ≈ 66 W/m² para uma radiação média de ≈ 470 W/m²). A Regressão Linear (R² ≈ 0,36) não representa o ciclo diário, que tem forma de sino, e chega a prever radiação negativa. A **hora** é a variável mais importante (cerca de metade da importância no Random Forest), pois define a altura do sol e o teto de radiação; nuvens, umidade e temperatura modulam esse valor. Estimar radiação **não equivale a prever geração elétrica**, que depende também de área e eficiência dos painéis, temperatura de operação, inclinação, sombreamento, perdas no inversor e na rede e cortes de operação. Os dados são de reanálise, de um único local e de três meses, e o teste cobre só o fim de junho.
