# Função de Recompensa, Hiperparâmetros e Algoritmos de Aprendizado

Este guia tem como foco dicas práticas de otimização para a criação de modelos no AWS DeepRacer. Serão abordadas as melhores práticas para criar boas Funções de Recompensa, escolher os Hiperparâmetros adequados e decidir entre os algoritmos PPO e SAC.

## Pré-requisito

Este documento é focado estritamente em otimização. Recomenda-se a leitura prévia do módulo Conceitos Fundamentais de Aprendizagem por Reforço, pois o entendimento teórico básico é necessário para a aplicação destas dicas.

## Otimizando a Função de Recompensa

A Função de Recompensa é o coração do seu modelo. Ela define o objetivo que o agente deve alcançar.

### Dica 1: Não programe as ações do carro

Esta é uma das diretrizes mais importantes: não faça o hard code das ações do veículo. Evite escrever condicionais (if) para cada curva específica da pista.

Seu papel é definir os objetivos (ex: "ficar no centro" ou "ir rápido em retas"), e o agente descobrirá sozinho a melhor forma de atingi-los. Se o objetivo for ficar no centro, o agente pode aprender, por conta própria, que precisa começar a virar 1 metro antes da curva.

#### Nota sobre Generalização

Programar explicitamente cada ação impede que o modelo generalize o aprendizado para pistas desconhecidas. A capacidade de generalizar a partir de um conjunto de dados é o principal propósito do aprendizado de máquina.

### Dica 2: Comece simples, foque em completar a pista

É tentador configurar a função para buscar o menor tempo de volta logo na primeira iteração, mas isso é um erro. O foco inicial deve ser exclusivamente completar a pista.

Se a velocidade for muito recompensada no início, o agente pode desenvolver um comportamento "kamikaze": acelerar ao máximo em uma reta para ganhar uma alta recompensa de velocidade, mesmo que isso resulte em sair da pista logo em seguida. O treinamento gastará horas apenas para desaprender esse comportamento.

#### Estratégia recomendada

Limite a velocidade do espaço de ação (seja contínuo ou discreto) com um valor máximo entre 1 e 2 m/s. Quando o agente conseguir completar a pista de forma consistente e controlada, clone o modelo e, então, aumente a velocidade do espaço de ação.

### Dica 3: Junção de Recompensas

Completar a pista rapidamente exige o controle simultâneo de posição, velocidade e ângulo de direção. Essas tarefas são divididas em sub-recompensas, que ao final do código são agrupadas em uma recompensa final. Existem três métodos principais para essa junção:

#### 1. Soma Simples

Permite compensação. O carro pode ser ruim em um aspecto, mas bom em outro.

```python
recompensa_total = recompensa_centro + recompensa_velocidade
```

**Ponto de atenção:** O agente pode focar na recompensa mais fácil e ignorar a outra (ex: andar perfeitamente no centro, mas extremamente devagar).

#### 2. Multiplicação

Exige que todas as condições sejam pelo menos parcialmente atendidas.

```python
recompensa_total = recompensa_centro * recompensa_velocidade
```

**Ponto de atenção:** Se qualquer sub-recompensa for zero (ou próxima disso), a recompensa total será zero. Se o carro parar (recompensa_velocidade = 0), ele não receberá nada, mesmo estando no centro.

#### 3. Exponenciação

Cria um incentivo não linear que recompensa apenas a excelência. Valores médios são reduzidos drasticamente, enquanto valores próximos do ideal (normalmente 1) disparam a pontuação.

**Ponto de atenção:** O aprendizado pode ser muito lento no início devido à escassez de recompensas altas, dificultando a descoberta do caminho certo pelo agente.

### Dica 4: Use Pesos

Sub-recompensas raramente possuem a mesma importância. No início, "ficar na pista" é mais vital que "ser rápido". Utilize multiplicadores para definir a importância de cada variável:

```python
PESO_CENTRO = 0.7
PESO_VELOCIDADE = 0.3

recompensa_centro_ponderada = PESO_CENTRO * recompensa_centro
recompensa_velocidade_ponderada = PESO_VELOCIDADE * recompensa_velocidade

recompensa_total = recompensa_centro_ponderada + recompensa_velocidade_ponderada
```

Essa técnica comunica matematicamente que a centralização tem 70% de prioridade contra 30% da velocidade. Altere esses pesos conforme o treinamento progride e o modelo é clonado.

### Dica 5: Evite recompensar diretamente a velocidade

Recompensar a velocidade de forma crua (recompensa_velocidade = speed) incentiva o comportamento "kamikaze" mencionado na Dica 2.

#### Abordagem Condicional

Dê bônus de velocidade apenas se outras condições estiverem sendo cumpridas (ex: estar em uma reta com o volante alinhado).

#### Abordagem Indireta

Recompense comportamentos que indiretamente exigem que o carro vá mais rápido.

### Dica 6: Bônus por completar a pista

Para resolver o "Problema da Atribuição de Crédito", forneça um bônus significativo quando o agente atingir progress == 100.

O agente toma milhares de decisões por episódio. Sem um bônus final, bater em 95% da pista pode gerar uma recompensa acumulada quase igual a de uma volta completa. O bônus cria um pico de recompensa, sinalizando que aquela sequência específica de ações foi extremamente valiosa.

#### Bônus Dinâmico

Você pode atrelar o tamanho do bônus à quantidade de passos (steps) levados para completar a pista, incentivando a velocidade de forma indireta.

#### Aviso

O bônus deve ser proporcional. Ele serve para coroar uma boa volta, não para ofuscar o restante das recompensas e desestabilizar o algoritmo.

### Dica 7: Penalize Ações Imperdoáveis

Ações que vão contra o propósito da corrida ou causam reset (is_offtrack, is_crashed, is_reversed) devem ser punidas severamente.
Se o carro não for penalizado por sair da pista, ele entenderá que isso é aceitável após coletar as recompensas de uma reta.

#### Como penalizar

Force o retorno da função a ser um valor extremamente baixo, mas positivo. A prática padrão da comunidade é utilizar 1e-3 (0.001).

#### Por que não usar zero ou negativo?

O algoritmo do DeepRacer é matematicamente otimizado para lidar com recompensas positivas. Valores nulos ou negativos podem desestabilizar os cálculos do gradiente e arruinar o treinamento.

### Dica 8: Normalize suas Recompensas

Mantenha a escala de recompensas consistente e proporcional (exemplo: mantendo o retorno total no intervalo de 0 a 100) para garantir estabilidade:

#### Recompensas Gigantes

(ex: +10.000): Causam hiperfoco. O agente fica vidrado em repetir aquela única ação, desequilibrando o ajuste da rede neural.

#### Punições Severas

Causam congelamento. O agente adquire "medo" de explorar para não sofrer a penalidade, optando por comportamentos excessivamente lentos e seguros.

## Otimizando Hiperparâmetros

[Inserir imagem: Tela de configuração de hiperparâmetros no console do DeepRacer]

Hiperparâmetros são as configurações definidas por você antes do início do processo que controlam a mecânica de aprendizado do algoritmo.

### Dica 9: Hiperparâmetros Não Fazem Milagres

A Função de Recompensa define o Destino (o objetivo). Os Hiperparâmetros definem o Método (como ele aprende). Se a Função de Recompensa estiver apontando para o destino errado, nenhum ajuste de hiperparâmetro fará o carro aprender o comportamento correto.

### Dica 10: Modelo piorando? A culpa geralmente é da recompensa

Quando a curva de recompensa no gráfico começa a cair, a tendência é querer alterar a taxa de aprendizado ou o tamanho do lote. Não faça isso de imediato.
Na maioria das vezes, a degradação ocorre por uma atribuição incorreta de recompensas altas para ações que são prejudiciais no longo prazo. Analise os logs de treinamento antes de mexer nos hiperparâmetros.

### Dica 11: Os Quatro Ajustes Principais

#### Taxa de Aprendizado (Learning Rate)

Define a velocidade com que o modelo atualiza suas regras.

**Alta:** Aprendizado rápido, mas com risco de instabilidade (o gráfico fica muito "serrilhado").

**Baixa:** Estável, mas demorado. Se o gráfico estiver muito ruidoso, reduza o valor (ex: de 0.0003 para 0.0001).

#### Tamanho do Lote (Batch Size)

Quantidade de interações analisadas de uma só vez para fazer uma atualização (siga sempre as potências de 2: 32, 64, 128, 256...).

Lotes maiores trazem estabilidade para o gráfico. Lotes menores são mais rápidos, porém mais ruidosos.

#### Número de Épocas (Number of Epochs)

Quantas vezes o algoritmo revisa os dados do mesmo lote.

**Alta:** Ajuda na memorização do lote. Se o carro vai muito bem no treino e mal na avaliação (overfitting), ele está decorando a pista. Reduzir as épocas ajuda na generalização.

#### Função de Entropia (Entropy)

Controla o balanço entre exploração (testar novas ações) e explotação (usar o que já sabe).

**Aumentar:** Força o carro a ser menos confiante e tentar ações arriscadas (útil se o modelo estagnar em um comportamento seguro e lento).

**Diminuir:** Foca na otimização de uma rota já conhecida.

##### Nota sobre Clonagem

Ao clonar um modelo, a entropia volta ao valor inicial definido no hiperparâmetro, forçando o modelo a "explorar" novamente. Isso explica por que o desempenho de um clone pode parecer pior nas primeiras iterações de treino.

## Algoritmos de Aprendizado: PPO vs. SAC

A escolha do algoritmo define como o agente lida com os dados coletados na pista.

### A Diferença Central: On-Policy vs. Off-Policy

**PPO (On-Policy):** Coleta um lote de experiências recentes, aprende com elas e as descarta. É altamente estável, mas requer um grande volume contínuo de novas experiências.

**SAC (Off-Policy):** Guarda experiências em um Replay Buffer (memória de longo prazo). Aprende utilizando dados recentes misturados com dados antigos. É extremamente eficiente em relação à quantidade de dados, mas pode apresentar instabilidade.

### Quando usar PPO (Proximal Policy Optimization)

É o algoritmo padrão e a escolha ideal para a grande maioria dos casos.

#### Para Iniciantes

O comportamento e os gráficos gerados são mais previsíveis.

#### Time Trial (Ação Discreta)

Excelente para otimizar uma linha de corrida consistente em competições de tomada de tempo.

#### Foco em Estabilidade

Se um modelo rodando SAC estiver oscilando muito, migrar para PPO trará constância.

### Quando usar SAC (Soft Actor-Critic)

Sendo um algoritmo mais moderno e complexo, brilha em cenários específicos.

#### Espaço de Ação Contínuo

O SAC foi desenhado para lidar com variáveis contínuas de forma nativa e eficiente.

#### Ambientes Complexos

Ideal para modalidades de Object Avoidance (desvio de obstáculos) ou Head-to-Head (corridas com outros veículos). A capacidade de reutilizar dados do Replay Buffer permite que ele aprenda estratégias de esquiva muito mais rápido que o PPO.

#### Desestagnar Modelos

Se o PPO atingiu um platô e não melhora de forma alguma, a mecânica de exploração do SAC pode fazer o agente descobrir novas estratégias.

## Próxima Etapa
[Análise de Logs](Análise_de_Logs.md)