---
layout: post
title:  "Como Fazer Previsão de Demanda com Ajustamento Sazonal em Python"
date:   2025-08-29 12:00:52 -0300
categories: jekyll update
---
Prever a demanda futura é um dos desafios mais críticos para qualquer negócio. Quando as vendas ou a produção seguem padrões que se repetem ao longo do ano — como mais vendas de sorvete no verão ou de casacos no inverno — estamos lidando com a **sazonalidade**.

Neste tutorial, vamos explorar o **método de ajustamento sazonal**, uma técnica poderosa para prever a demanda em cenários como este. Faremos isso de forma prática, passo a passo, utilizando Python e a biblioteca NumPy.

**O que vamos fazer:**

1.  Analisar uma série histórica de produção trimestral.
2.  Calcular os fatores de sazonalidade para entender os picos e vales de demanda.
3.  Utilizar uma regressão linear simples para prever a demanda total do próximo ano.
4.  Ajustar essa previsão total para cada trimestre, aplicando os fatores sazonais.

### **Passo 1: Entender o Problema e os Dados**

Imagine que temos os dados de produção de uma empresa ao longo de três anos, divididos por trimestre.

| | Trimestre 1 | Trimestre 2 | Trimestre 3 | Trimestre 4 |
| :--- | :--- | :--- | :--- | :--- |
| **Ano 1** | 60 | 890 | 250 | 120 |
| **Ano 2** | 80 | 1600 | 290 | 150 |
| **Ano 3** | 130 | 2300 | 500 | 260 |

Ao observar a tabela, notamos um padrão claro: a produção (e, consequentemente, a demanda) é muito maior nos trimestres 2 e 3. O nosso objetivo é **prever a demanda para cada trimestre do Ano 4**.
a
### **Passo 2: Configuração do Ambiente em Python**

Para começar, vamos precisar apenas da biblioteca NumPy, que é fundamental para cálculos numéricos em Python. Se ainda não a tiver instalada, pode fazê-lo com um simples comando no seu terminal:

```bash
pip install numpy
```

Agora, vamos abrir o nosso editor de código ou Jupyter Notebook e começar a programar.

1.  **Importar a biblioteca e definir os dados:**

    ```python
    import numpy as np

    # Dados de produção trimestral para os anos 1, 2 e 3
    dados = np.array([
    [60, 890, 250, 120],  # Ano 1
    [80, 1600, 290, 150],  # Ano 2
    [130, 2300, 500, 260]  # Ano 3
    ])

    # Anos correspondentes aos dados
    anos = np.array([1, 2, 3])
    ```

### **Passo 3: Calcular os Totais e Médias Anuais**

O primeiro passo da nossa análise é calcular o total de produção e a média trimestral para cada ano.

```python
# Calcular os totais anuais (soma das linhas)
totais_anuais = np.sum(dados, axis=1)
print(f"Totais anuais: {totais_anuais}")
# Saída esperada: Totais anuais: [1320 2120 3190]

# Calcular as médias anuais
medias_anuais = np.mean(dados, axis=1)
print(f"Médias trimestrais anuais: {medias_anuais}")
# Saída esperada: Médias trimestrais anuais: [330.  530.  797.5]
```

### **Passo 4: Encontrar os Coeficientes de Sazonalidade**

Esta é a parte mais importante do método\! O coeficiente de sazonalidade nos diz o quanto um trimestre específico desvia da média daquele ano.

Calculamo-lo dividindo a produção de cada trimestre pela média do seu respectivo ano.

```python
# Criar uma matriz vazia para guardar os coeficientes
coeficientes_sazonalidade = np.zeros_like(dados, dtype=float)

# Calcular os coeficientes
for i in range(len(anos)):
    for j in range(4): # 4 trimestres
        coeficientes_sazonalidade[i, j] = dados[i, j] / medias_anuais[i]

print("Coeficientes de Sazonalidade por Ano e Trimestre:")
print(coeficientes_sazonalidade)
```

Agora, para obter um fator de ajuste único para cada trimestre, calculamos a média dos coeficientes de cada coluna.

```python
# Calcular a média dos coeficientes para cada trimestre (média das colunas)
coef_sazonais_medios = np.mean(coeficientes_sazonalidade, axis=0)

print(f"\nCoeficientes Sazonais Médios (Fatores de Ajuste):")
print(coef_sazonais_medios)
# Saída esperada: [0.16525699 2.86661672 0.64390161 0.32422468]
```

Estes quatro números são os nossos **fatores de ajuste**. Um valor maior que 1 (como no T2 e T3) indica uma demanda acima da média, e um valor menor que 1 (T1 e T4) indica uma demanda abaixo da média.

### **Passo 5: Prever a Demanda Total para o Ano 4**

Observando os totais anuais (`[1320 2120 3190]`), vemos uma tendência de crescimento. Vamos usar uma **regressão linear** para projetar qual será o total para o Ano 4.

A função `np.polyfit()` faz este trabalho para nós, encontrando os coeficientes da reta (`y = ax + b`) que melhor se ajusta aos nossos dados.

```python
# Ajustar uma reta aos totais anuais
# O '1' indica que queremos um polinómio de grau 1 (uma reta)
coeficientes_reta = np.polyfit(anos, totais_anuais, 1)

# A função de previsão (a nossa reta)
previsao_func = np.poly1d(coeficientes_reta)

# Prever o total para o Ano 4
ano_a_prever = 4
total_previsto_ano4 = previsao_func(ano_a_prever)

print(f"\nA equação da reta de tendência é: y = {coeficientes_reta[0]:.2f}x + {coeficientes_reta[1]:.2f}")
# Saída esperada: A equação da reta de tendência é: y = 935.00x + 340.00
print(f"A demanda total prevista para o Ano 4 é: {total_previsto_ano4:.2f}")
# Saída esperada: A demanda total prevista para o Ano 4 é: 4080.00
```

### **Passo 6: Ajustar a Previsão com os Fatores Sazonais**

Temos a previsão total para o Ano 4 (4080 unidades), mas como ela se distribui pelos trimestres?

1.  Primeiro, calculamos a demanda trimestral base, como se não houvesse sazonalidade:

    ```python
    demanda_trimestral_base_ano4 = total_previsto_ano4 / 4
    print(f"\nDemanda trimestral base (sem ajuste) para o Ano 4: {demanda_trimestral_base_ano4:.2f}")
    # Saída esperada: Demanda trimestral base (sem ajuste) para o Ano 4: 1020.00
    ```

2.  Finalmente, multiplicamos esta demanda base pelos nossos fatores de ajuste sazonais:

    ```python
    # Aplicar os fatores sazonais para obter a previsão final
    previsao_ajustada_ano4 = demanda_trimestral_base_ano4 * coef_sazonais_medios

    print("\n--- PREVISÃO FINAL PARA O ANO 4 ---")
    for i, previsao in enumerate(previsao_ajustada_ano4):
        print(f"Trimestre {i+1}: {previsao:.2f} unidades")
    ```

**Resultado Final Esperado:**

```
--- PREVISÃO FINAL PARA O ANO 4 ---
Trimestre 1: 168.56 unidades
Trimestre 2: 2923.95 unidades
Trimestre 3: 656.78 unidades
Trimestre 4: 330.71 unidades
```

Como podemos ver, a nossa previsão final reflete perfeitamente o padrão sazonal observado nos dados históricos, com uma demanda muito maior nos trimestres 2 e 3.

### **Conclusão**

Acabamos de implementar um método de previsão de demanda robusto. Com apenas algumas linhas de Python, conseguimos capturar a tendência de crescimento e os padrões sazonais para gerar uma previsão para o futuro. Este método pode ser aplicado a diversos tipos de dados, desde vendas mensais a produção diária.

