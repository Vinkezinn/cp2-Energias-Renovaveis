# Projeto de Machine Learning: Energias Renováveis

## Objetivo
Este repositório contém a avaliação final envolvendo o consumo de APIs públicas e a aplicação de Machine Learning para duas tarefas no setor de energias renováveis:
1. **Classificação:** Prever a fonte de energia de empreendimentos (ANEEL) baseando-se em potência e localização.
2. **Regressão:** Estimar a radiação solar horária (Open-Meteo) em Petrolina (PE) usando variáveis atmosféricas.

## Origem dos Dados
* **Tarefa 1:** [SIGA - ANEEL](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel)
* **Tarefa 2:** [API Histórica Open-Meteo](https://open-meteo.com/en/docs/historical-weather-api) (Período: 01/04/2025 a 30/06/2025).

## Como Executar
1. Clone este repositório.
2. Certifique-se de que possui as bibliotecas listadas nos imports (`pandas`, `scikit-learn`, `matplotlib`, `seaborn`).
3. Mantenha os ficheiros `.csv` no mesmo diretório do notebook.
4. Execute o ficheiro `Notebook_Energias.ipynb` célula a célula.

## Conclusões Principais
* **Classificação:** A Regressão Logística obteve o menor desempenho (Accuracy de ~82%), enquanto modelos baseados em árvores (Random Forest) ou vizinhos (KNN) superaram os 96%. A limitação do modelo reside na ausência de contexto topográfico.
* **Regressão:** A Regressão Linear teve baixa capacidade de capturar a não-linearidade do ciclo solar (R² de ~0.35). O Random Forest Regressor destacou-se (R² de ~0.84 e menor MAE), evidenciando a importância da variável temporal (`hora`) conjugada com a cobertura de nuvens.
