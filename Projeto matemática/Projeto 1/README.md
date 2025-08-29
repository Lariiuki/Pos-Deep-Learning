## Explicação do Projeto 🔎

Grupo: Larissa Akemi Iuki, Marcos Vinicuis Suguino, Ana Vitória da Silva Santos, Luan Fernandes de França, Vinicius da Silva Ferreira Lima

**Objetivo:** O trabalho consiste em projetar e implementar uma rede neural artificial a partir do exemplo exemplo4.py.
Foram feitas modificações na arquitetura, nas funções de ativação e na base de dados para observar como esses fatores influenciam o desempenho do modelo. No notebook, foi realizado um experimento de classificação utilizando redes neurais para separar dados não linearmente separáveis, gerados pela função make_circles do scikit-learn. 

## Código exemplo

Código original (exemplo4.py) utilizava o dataset 'make_moons', implementava uma rede neural manual com 1 camada escondida com 2 neurônios, com função de ativação **Sigmoide** e algoritmo de retropropagação + gradiente descendente. Também implementava uma versão em Keras, como comparação.

## Código modificado

As modificações realizadas foi a mudança do dataset utilizado para 'make_circles', que gera pontos em círculos concêntricos, com ruído, para simular um problema de classificação binária. Na rede manual aumentamos a camada escondida de 2 para 4 neurônios e a função de ativação que utilizamos é a **tanh**. Na rede com keras definimos 2 camadas escondidas de 8 neurônios cada, com função de ativação **ReLU** e com camada de saída **Sigmoide**. O treinamento foi feito via gradiente descendente, ajustando os pesos para minimizar o erro quadrático. A acurácia da rede foi avaliada antes e depois do treinamento, mostrando a evolução do aprendizado. O modelo final apresentou uma acurácia significativamente melhor, mostrando que as modificações na arquitetura e nas funções de ativação tiveram impacto positivo.

## Conclusão

No início do treinamento, a perda era muito alta, com valor de `0 [14.72242375]`. À medida que o modelo é treinado, a perda diminui progressivamente, atingindo `0.26997605` após 4.000 iterações. Esse comportamento indica que o modelo está aprendendo, pois a função de perda apresenta uma tendência clara de decrescimento.

A acurácia final alcançou 100%, o que significa que o modelo classificou **todas as amostras corretamente**. Além disso, a perda final é extremamente baixa (`0.0147`), confirmando que as previsões do modelo estão muito próximas dos valores reais.
