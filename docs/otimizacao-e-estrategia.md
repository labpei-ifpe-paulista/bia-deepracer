# Função de Recompensa, Hiperparâmetros e Algoritmos de Aprendizado

Este guia tem como foco dicas práticas de otimização para a criação de modelos no AWS DeepRacer. Serão abordadas estratégias para construir boas funções de recompensa, escolher hiperparâmetros adequados, interpretar o comportamento dos modelos e decidir entre os algoritmos PPO e SAC.

O objetivo não é apresentar configurações universalmente ideais, mas fornecer princípios que ajudem você a formular hipóteses, conduzir experimentos e identificar quais ajustes realmente melhoram o desempenho do agente.

## Pré-requisito

Este documento é focado em otimização. Recomenda-se a leitura prévia do módulo [Conceitos Fundamentais de Aprendizagem por Reforço](fundamentos-rl.md), pois o entendimento dos conceitos de política, retorno cumulativo, espaço de ação e função de recompensa é importante para aplicar as estratégias apresentadas.

Os procedimentos e as configurações disponíveis podem variar conforme a implementação do DeepRacer utilizada. Consulte a documentação correspondente ao ambiente de treinamento antes de reproduzir os exemplos.

## Otimizando a Função de Recompensa

A função de recompensa é um dos componentes mais importantes do modelo. Ela traduz o objetivo desejado em sinais numéricos utilizados pelo algoritmo durante o treinamento.

Uma função bem definida deve incentivar comportamentos que contribuam para o objetivo final, sem criar conflitos desnecessários entre os diferentes critérios de desempenho.

### Dica 1: Não programe explicitamente as ações do carro

Uma das diretrizes mais importantes é evitar transformar a função de recompensa em uma sequência de instruções que determine antecipadamente cada movimento do veículo.

Por exemplo, em vez de escrever regras que determinem exatamente quando virar à esquerda em cada trecho de uma pista específica, procure definir os comportamentos desejados e os critérios que permitem avaliá-los.

Seu papel é estabelecer incentivos coerentes, como permanecer dentro dos limites da pista, manter uma trajetória controlada e alcançar uma velocidade adequada. O algoritmo utilizará as experiências coletadas durante o treinamento para procurar uma política que maximize as recompensas.

Isso não significa que condicionais (`if`) devam ser evitadas. Elas podem ser úteis para distinguir situações, identificar comportamentos indesejados e aplicar recompensas diferentes conforme as condições do ambiente.

O importante é que essas condições avaliem características relevantes da situação, em vez de apenas reproduzir uma sequência rígida de movimentos.

#### Nota sobre generalização

Uma função excessivamente dependente de posições ou movimentos específicos de determinada pista pode limitar a capacidade do agente de lidar com situações diferentes.

!!! info "Nota conceitual"
    Entretanto, evitar regras rígidas não garante generalização. O desempenho em outras pistas também depende das experiências de treinamento, das observações disponíveis, do espaço de ação e das características do ambiente.

Por isso, avalie o modelo nas condições para as quais ele foi desenvolvido e, quando possível, teste seu comportamento em pistas ou situações diferentes. Não presuma que um modelo bem-sucedido em uma pista apresentará o mesmo desempenho em todas as outras.

### Dica 2: Comece simples e priorize a conclusão da pista

É tentador buscar o menor tempo de volta logo na primeira versão da função de recompensa. Entretanto, otimizar a velocidade antes de estabelecer um comportamento consistente pode tornar o treinamento mais difícil de interpretar.

Se a velocidade receber uma recompensa elevada sem considerar adequadamente o risco de sair da pista, o agente poderá aprender a acelerar em situações nas quais isso prejudica o resultado final.

Por exemplo, um veículo pode obter uma recompensa imediata por alcançar uma velocidade elevada em uma reta, mas sair da pista na curva seguinte. Nesse caso, a função favoreceu um comportamento que parecia vantajoso no curto prazo, mas não contribuiu para completar o percurso.

#### Estratégia recomendada

Divida a otimização em etapas:

1. **Estabeleça um comportamento básico:** incentive o agente a permanecer na pista e completar o percurso.
2. **Avalie a consistência:** verifique se o modelo consegue completar voltas de maneira repetida, e não apenas em uma tentativa isolada.
3. **Introduza melhorias graduais:** depois de obter um comportamento estável, investigue como aumentar a velocidade sem comprometer a conclusão da pista.
4. **Compare os resultados:** avalie o modelo original e a nova versão sob condições equivalentes.

É possível limitar inicialmente a velocidade máxima permitida no espaço de ação para reduzir a complexidade do problema. Valores entre 1 e 2 m/s podem servir como ponto de partida em determinados experimentos, mas não são uma faixa ideal universal: a velocidade apropriada depende da pista, da modalidade, das ações disponíveis e do objetivo do treinamento.

Quando o ambiente permitir, você também poderá clonar um modelo bem-sucedido e utilizar a nova versão como ponto de partida para a próxima etapa. Antes de treinar, confira quais parâmetros e artefatos foram preservados na clonagem.

### Dica 3: Combine as recompensas com cuidado

Completar uma pista de maneira consistente e rápida pode exigir que o agente equilibre diferentes objetivos, como manter uma trajetória adequada, controlar a velocidade e evitar saídas da pista.

Uma estratégia é construir sub-recompensas para aspectos específicos do comportamento e combiná-las em uma recompensa final.

Considere dois exemplos abstratos:

- `recompensa_centro`: representa o incentivo para manter uma posição adequada na pista.
- `recompensa_velocidade`: representa o incentivo para atingir uma velocidade desejada.

Para que os exemplos a seguir sejam comparáveis, considere que as duas sub-recompensas estejam normalizadas entre 0 e 1, em que valores maiores representam resultados mais desejáveis.

Existem diferentes maneiras de combiná-las.

#### 1. Soma simples

A soma permite que uma recompensa compense parcialmente a outra.

```python
recompensa_total = recompensa_centro + recompensa_velocidade
```

**Ponto de atenção:** o agente pode obter uma pontuação razoável mesmo quando apresenta um resultado ruim em um dos objetivos.

Por exemplo, uma recompensa elevada por velocidade pode compensar parcialmente uma recompensa baixa por posicionamento. O modelo pode, assim, receber um sinal aparentemente favorável apesar de apresentar um comportamento indesejado.

#### 2. Multiplicação

A multiplicação faz com que as sub-recompensas influenciem conjuntamente o resultado final.

```python
recompensa_total = recompensa_centro * recompensa_velocidade
```

**Ponto de atenção:** quando uma das sub-recompensas se aproxima de zero, o resultado da multiplicação também se aproxima de zero.

Isso pode ser útil quando se deseja exigir um desempenho razoável em diferentes critérios, mas também pode dificultar o aprendizado quando uma das recompensas permanece muito baixa no início do treinamento.

#### 3. Potenciação

A potenciação pode tornar a recompensa mais seletiva em relação aos valores elevados.

```python
recompensa_total = recompensa_base ** 2
```

Para valores de `recompensa_base` entre 0 e 1, essa transformação reduz proporcionalmente os valores menores e mantém inalterado o valor 1.

**Ponto de atenção:** a transformação também diminui a intensidade de muitas recompensas intermediárias. Se o agente encontrar poucas situações que produzam valores elevados, o sinal de aprendizado poderá ficar menos informativo.

A potenciação não é necessariamente superior aos outros métodos. Sua utilidade depende da escala original da recompensa e do comportamento que se deseja incentivar.

**Recomendação:** comece com uma função simples. Adicione novas sub-recompensas somente quando houver uma hipótese clara sobre o problema que cada uma pretende resolver. Quanto mais componentes forem introduzidos simultaneamente, mais difícil será identificar a origem de uma melhoria ou regressão.

### Dica 4: Utilize pesos para expressar prioridades

Quando diferentes sub-recompensas possuem importâncias distintas, é possível utilizar pesos para controlar sua contribuição para o resultado final.

Por exemplo:

```python
PESO_CENTRO = 0.7
PESO_VELOCIDADE = 0.3

recompensa_centro_ponderada = (
    PESO_CENTRO * recompensa_centro
)

recompensa_velocidade_ponderada = (
    PESO_VELOCIDADE * recompensa_velocidade
)

recompensa_total = (
    recompensa_centro_ponderada
    + recompensa_velocidade_ponderada
)
```

Nesse exemplo, a recompensa relacionada ao posicionamento possui um peso maior do que a recompensa relacionada à velocidade.

Quando as sub-recompensas estão na mesma escala e os pesos somam 1, a combinação funciona como uma média ponderada. Isso facilita interpretar a contribuição relativa de cada componente.

Entretanto, os pesos não garantem que o agente apresentará o comportamento correspondente à prioridade desejada. O efeito depende também da distribuição dos valores retornados por cada sub-recompensa e de como esses incentivos interagem durante o treinamento.

Altere os pesos gradualmente e compare os resultados. Evite modificar vários pesos simultaneamente sem registrar as alterações e suas consequências.

### Dica 5: Evite recompensar a velocidade sem considerar o contexto

Recompensar diretamente a velocidade, por exemplo utilizando `recompensa_velocidade = speed`, pode incentivar o agente a acelerar mesmo quando isso aumenta o risco de sair da pista.

O problema não está em recompensar a velocidade, mas em ignorar as demais consequências desse comportamento.

#### Abordagem condicional

Conceda um bônus de velocidade quando outras condições relevantes também estiverem sendo atendidas.

Por exemplo, a função pode considerar características relacionadas à posição na pista, ao controle da direção e à permanência dentro dos limites do percurso.

Essa abordagem permite que a velocidade seja incentivada de acordo com o contexto, em vez de receber sempre o mesmo tratamento.

Entretanto, condições excessivamente rígidas também podem restringir comportamentos úteis. Os critérios precisam ser avaliados de acordo com o objetivo e com os dados disponíveis.

#### Abordagem indireta

Outra possibilidade é priorizar comportamentos que contribuem para uma corrida eficiente, como completar a pista de forma consistente, evitando recompensar a velocidade de maneira isolada.

Nesse caso, a velocidade ainda pode ser utilizada como critério de otimização, mas seu efeito deve ser analisado em conjunto com o desempenho global do modelo.

**Ponto de atenção:** nenhuma dessas abordagens garante, por si só, uma boa estratégia de direção. Uma função de recompensa precisa ser validada observando o comportamento que o agente realmente aprende.

### Dica 6: Considere recompensar a conclusão da pista

Uma recompensa adicional pela conclusão do percurso pode ajudar a diferenciar uma volta completa de uma tentativa interrompida antes do final.

Durante um episódio, o agente toma diversas decisões. Se a função valorizar apenas comportamentos intermediários, ele poderá acumular recompensas sem completar a tarefa.

Uma recompensa associada à conclusão pode tornar esse objetivo mais explícito.

#### Bônus por conclusão

No DeepRacer, o parâmetro `progress` representa a porcentagem da pista completada. Quando estiver disponível e for apropriado à implementação, ele poderá ser utilizado para identificar a conclusão do percurso.

Por exemplo, a lógica conceitual de um bônus adicional pode ser representada assim:

```python
BONUS_CONCLUSAO = 5.0

if params["progress"] >= 100.0:
    recompensa_total += BONUS_CONCLUSAO
```

Esse trecho é apenas ilustrativo e pressupõe que `recompensa_total` já tenha sido calculada anteriormente. O valor do bônus deve ser definido de acordo com a escala da função utilizada.

#### Bônus dinâmico

Outra possibilidade é relacionar o bônus ao número de passos necessários para concluir a pista. Em determinadas configurações, isso pode favorecer trajetórias que concluem o percurso em menos passos.

Entretanto, a redução no número de passos não representa necessariamente uma redução proporcional no tempo de volta. O resultado também depende da velocidade e da dinâmica do veículo.

#### Aviso

Um bônus excessivamente elevado pode dominar o restante do sinal de recompensa. Isso não garante um aprendizado melhor e pode dificultar a avaliação da contribuição dos outros critérios.

Além disso, uma recompensa terminal não elimina completamente o problema de atribuição de crédito: o agente ainda precisa aprender quais decisões anteriores contribuíram para o resultado final.

Utilize o bônus como parte da estratégia de recompensa, e não como substituto de uma função coerente para o restante do percurso.

### Dica 7: Penalize comportamentos que comprometem o objetivo

Comportamentos como sair da pista ou colidir com obstáculos podem impedir a conclusão do percurso e devem ser tratados de maneira coerente com o objetivo do treinamento.

Se o agente receber recompensas elevadas por acelerar ou manter determinada trajetória, mas não sofrer consequências adequadas quando sai da pista, a função poderá incentivar comportamentos incompatíveis com uma corrida consistente.

#### Como penalizar

Uma abordagem utilizada em exemplos de funções de recompensa do DeepRacer é retornar um valor muito baixo, mas positivo, para determinadas situações indesejadas.

Por exemplo:

```python
def reward_function(params):
    if not params["all_wheels_on_track"]:
        return 1e-3

    return 1.0
```

Nesse exemplo, o modelo recebe uma recompensa baixa quando todas as rodas não estão dentro da pista e uma recompensa maior quando o veículo permanece dentro dos limites.

O valor `1e-3`, equivalente a `0.001`, é uma escolha possível, não um valor obrigatório para todas as situações.

Dependendo do problema, uma função pode utilizar outras penalizações ou recompensas. O importante é considerar a escala dos valores, as características do algoritmo e a consequência que se deseja representar.

#### Por que não tratar zero ou valores negativos como universalmente proibidos?

Recompensas nulas ou negativas não são, por si só, matematicamente inválidas em Aprendizagem por Reforço.

Sua adequação depende da implementação, do algoritmo e da forma como os valores influenciam o treinamento. Portanto, não é correto afirmar que qualquer recompensa nula ou negativa necessariamente desestabilizará o modelo.

Ao utilizar exemplos ou recomendações desenvolvidos para uma implementação específica do DeepRacer, siga as convenções documentadas para aquele ambiente e verifique os resultados.

### Dica 8: Mantenha uma escala de recompensas coerente

A escala das recompensas influencia o sinal utilizado pelo algoritmo durante o treinamento.

Se uma sub-recompensa apresentar valores muito maiores do que os demais componentes, ela poderá dominar a combinação final e fazer com que outros objetivos tenham pouca influência no resultado.

Por isso, procure manter os componentes em escalas compatíveis quando eles forem combinados diretamente.

#### Recompensas desproporcionais

Imagine uma função em que a recompensa por velocidade seja centenas ou milhares de vezes maior do que a recompensa por permanecer na pista.

Nesse caso, a soma poderá favorecer a velocidade mesmo quando o veículo apresentar um comportamento inadequado em relação ao percurso.

O problema não é existir um valor absoluto específico, mas a relação entre os componentes e seus efeitos sobre o objetivo final.

#### Penalizações desproporcionais

Uma penalização muito forte pode dominar os demais incentivos. Dependendo da função e do treinamento, isso poderá dificultar a descoberta de estratégias úteis ou favorecer um comportamento excessivamente conservador.

Entretanto, não existe uma relação universal segundo a qual penalizações severas sempre tornam o modelo lento ou incapaz de explorar. O resultado precisa ser verificado experimentalmente.

#### Como avaliar a escala

- Analise os valores retornados por cada sub-recompensa.
- Verifique a contribuição de cada componente para a recompensa final.
- Evite combinar diretamente valores que estejam em escalas muito diferentes sem considerar suas proporções.
- Compare o comportamento do agente antes e depois de alterações na função.

Manter recompensas coerentes ajuda a interpretar o treinamento, mas não garante estabilidade por si só. A escala adequada depende do algoritmo, da implementação e das características do problema.

## Otimizando Hiperparâmetros

Os hiperparâmetros são configurações que influenciam a forma como o algoritmo aprende. Diferentemente dos parâmetros da rede neural ajustados durante o treinamento, eles são definidos pelo usuário conforme as opções disponíveis no ambiente.

A função de recompensa define o objetivo que o agente tentará alcançar. Os hiperparâmetros influenciam o processo utilizado para aprender uma política que maximize esse objetivo.

Essa distinção ajuda a evitar ajustes sem uma hipótese clara sobre o problema observado.

### Dica 9: Hiperparâmetros não fazem milagres

A função de recompensa e os hiperparâmetros possuem papéis diferentes.

Se a função de recompensa incentiva um comportamento indesejado, alterar a taxa de aprendizado ou o tamanho do lote não corrigirá necessariamente o problema.

Por exemplo, se o modelo recebe incentivos desproporcionais para acelerar, ele pode continuar buscando velocidade mesmo após diversas alterações nos parâmetros de treinamento.

Antes de ajustar os hiperparâmetros, procure responder às seguintes perguntas:

- A função de recompensa representa adequadamente o objetivo?
- O espaço de ação permite executar os comportamentos desejados?
- Os resultados observados correspondem às recompensas atribuídas?
- Existem evidências de instabilidade ou apenas oscilações naturais do treinamento?

Essa investigação ajuda a separar problemas relacionados à modelagem da recompensa daqueles associados ao processo de otimização.

### Dica 10: O modelo piorou? Investigue antes de modificar os parâmetros

Quando a curva de recompensa começa a cair, é comum alterar imediatamente a taxa de aprendizado, o tamanho do lote ou outros hiperparâmetros.

Entretanto, uma queda na recompensa não possui uma causa única. Ela pode estar relacionada à função de recompensa, às atualizações da política, à exploração, à variabilidade entre episódios ou a outras características da configuração.

#### Estratégia de diagnóstico

Antes de realizar alterações, siga uma sequência de investigação:

1. **Examine a curva de recompensa:** verifique se a queda é pontual ou se representa uma tendência persistente.
2. **Avalie o comportamento do veículo:** observe se o agente está saindo da pista, apresentando movimentos instáveis ou deixando de progredir.
3. **Analise os logs:** procure evidências que ajudem a identificar em quais situações o desempenho se deteriora.
4. **Revise a função de recompensa:** confira se o sinal recebido pelo agente corresponde ao comportamento desejado.
5. **Investigue os hiperparâmetros:** se os dados indicarem um problema no processo de otimização, escolha um ajuste compatível com a hipótese formulada.

!!! tip "Dica de experimentação"
    Evite alterar várias configurações de uma vez. Quando múltiplos parâmetros são modificados simultaneamente, fica mais difícil identificar qual alteração contribuiu para a melhoria ou piora do resultado.

### Dica 11: Conheça os principais hiperparâmetros

Os hiperparâmetros disponíveis variam conforme o algoritmo e a implementação do DeepRacer utilizada. Na solução oficial DeepRacer on AWS, PPO e SAC possuem conjuntos de configurações diferentes.

A seguir estão alguns dos parâmetros mais relevantes para compreender o processo de treinamento.

#### Taxa de aprendizado (*Learning Rate*)

A taxa de aprendizado controla a magnitude dos ajustes realizados nos parâmetros do modelo durante a otimização.

**Valor alto:** pode produzir atualizações excessivas e tornar o treinamento instável.

**Valor baixo:** pode resultar em atualizações menores e retardar a aprendizagem.

O efeito depende do algoritmo e das demais configurações. Na interface documentada da solução oficial, a taxa de aprendizado é apresentada entre os parâmetros de SAC; a disponibilidade desse ajuste para outros ambientes depende da implementação.

Quando o parâmetro estiver disponível, compare valores de forma controlada e verifique se existe uma alteração consistente no comportamento ou nas métricas de avaliação.

#### Tamanho do lote (*Batch Size*)

O tamanho do lote determina quantas experiências são utilizadas em uma atualização do treinamento.

Lotes maiores podem produzir estimativas menos ruidosas em determinadas condições, mas também podem exigir mais memória e processamento. Lotes menores podem permitir atualizações mais frequentes, sem garantir necessariamente uma aprendizagem melhor ou mais rápida.

Não existe uma regra universal segundo a qual se devam utilizar exclusivamente potências de 2. Valores disponíveis e adequados dependem das opções da implementação e das condições do experimento.

#### Número de épocas (*Number of Epochs*)

Nas configurações do PPO que disponibilizam esse parâmetro, o número de épocas indica quantas vezes o algoritmo utiliza o mesmo conjunto de experiências durante as atualizações.

Um número maior de épocas pode aumentar o aproveitamento dos dados coletados, mas também pode favorecer atualizações excessivas em determinadas configurações.

Reduzir as épocas não garante maior generalização. Se houver suspeita de *overfitting* ou de degradação da política, examine as métricas, o comportamento do agente e os demais parâmetros antes de decidir pela alteração.

Na solução oficial DeepRacer on AWS, esse parâmetro faz parte das configurações de PPO.

#### Número de episódios entre atualizações da política

Na configuração oficial de PPO, é possível definir quantos episódios de experiência são coletados antes de uma atualização da política.

Esse parâmetro influencia a quantidade de experiências reunidas para a atualização. Seu efeito depende também do tamanho dos episódios, do custo de coleta de dados e das demais configurações de treinamento.

Não deve ser confundido com o número de épocas: um parâmetro determina quantas experiências são coletadas antes da atualização, enquanto o outro determina quantas vezes o conjunto coletado é utilizado durante o processo de atualização.

#### Entropia e exploração

A entropia está relacionada à diversidade das ações selecionadas pela política.

No PPO, a entropia participa do processo de otimização como um mecanismo de regularização que incentiva a exploração. Uma política com maior entropia tende a distribuir suas escolhas entre diferentes ações, enquanto uma política com menor entropia tende a concentrar suas escolhas em ações mais prováveis.

No SAC, a exploração está incorporada ao objetivo de maximização da entropia. O parâmetro `SAC alpha` controla a intensidade do incentivo associado à exploração na implementação oficial.

**Ponto de atenção:** os mecanismos não são idênticos. O parâmetro `SAC alpha` não deve ser interpretado como se tivesse necessariamente o mesmo efeito de um coeficiente de entropia do PPO.

Além disso, maior exploração não é sempre melhor. Ela pode ajudar o agente a descobrir novas estratégias, mas também pode aumentar a variabilidade do comportamento durante o treinamento.

#### Fator de desconto (*Discount Factor*)

O fator de desconto, representado por $\gamma$, determina a importância atribuída às recompensas futuras no cálculo do retorno cumulativo.

Valores maiores dão relativamente mais importância a recompensas distantes, enquanto valores menores favorecem recompensas mais imediatas.

O parâmetro influencia o horizonte temporal considerado pelo agente, mas seu efeito sobre o desempenho depende do problema e das demais configurações.

Na solução oficial DeepRacer on AWS, o fator de desconto está disponível entre as configurações de SAC.

#### Função de perda (*Loss Type*)

O tipo de função de perda define como determinados erros de previsão são quantificados durante o treinamento.

Na solução oficial DeepRacer on AWS, a configuração de SAC permite escolher entre Huber loss e erro quadrático médio (*Mean Squared Error*).

A Huber loss trata erros pequenos de forma quadrática e reduz a influência quadrática de erros grandes, enquanto o erro quadrático médio penaliza os erros de acordo com seu quadrado.

A escolha influencia a forma como os erros contribuem para a otimização, mas não existe uma opção universalmente superior para todos os experimentos. É necessário avaliar o comportamento do treinamento e a adequação à configuração utilizada.

#### Clonagem de modelos

A clonagem pode ser útil quando você deseja desenvolver uma nova versão a partir de um modelo existente, evitando começar sempre do zero.

Uma estratégia prática é manter um modelo de referência e criar novas versões para investigar alterações específicas, como a função de recompensa ou o espaço de ação.

!!! warning "Atenção aos requisitos do ambiente"
    Entretanto, não presuma que todos os estados internos do treinamento ou todos os hiperparâmetros serão preservados da mesma maneira em qualquer implementação. Confira quais elementos foram copiados e revise a configuração antes de iniciar o próximo experimento.

A queda inicial no desempenho de um modelo clonado também não possui uma causa universal. Ela pode estar associada às novas configurações, às atualizações da política ou à variabilidade do treinamento. Investigue os dados antes de concluir que a clonagem ou determinado parâmetro foi responsável pelo resultado.

Referência: [Criação de modelos no DeepRacer on AWS](https://docs.aws.amazon.com/solutions/latest/deepracer-on-aws/create-a-model.html).

## Algoritmos de Aprendizado: PPO vs. SAC

A escolha do algoritmo influencia a maneira como as experiências são utilizadas durante o treinamento.

No DeepRacer on AWS, os algoritmos PPO e SAC possuem características diferentes. Compreender essas diferenças ajuda a selecionar experimentos adequados ao espaço de ação e aos objetivos do projeto.

### A diferença central: On-Policy vs. Off-Policy

**PPO (*Proximal Policy Optimization*) — On-Policy**

O PPO utiliza experiências coletadas pela política atual para orientar suas atualizações. Em linhas gerais, o agente coleta experiências, utiliza esses dados para atualizar a política e, depois, coleta novas experiências com a política atualizada.

Essa abordagem tende a exigir a coleta contínua de novos dados. O PPO também utiliza um mecanismo de otimização que procura limitar alterações excessivas na política durante as atualizações.

**SAC (*Soft Actor-Critic*) — Off-Policy**

O SAC pode reutilizar experiências coletadas por políticas anteriores. Os dados são armazenados em um *replay buffer*, que permite utilizá-los em múltiplas atualizações.

O algoritmo também incorpora a entropia ao objetivo de otimização, incentivando a exploração de diferentes ações.

A reutilização de experiências pode aumentar a eficiência na utilização dos dados, mas não garante melhor desempenho ou menor tempo de treinamento em qualquer problema.

### Quando usar PPO (Proximal Policy Optimization)

O PPO é um ponto de partida conveniente para diversos experimentos de DeepRacer, especialmente quando o objetivo é compreender os fundamentos do treinamento e estabelecer uma referência de desempenho.

#### Para iniciantes

O PPO é útil para estudar a relação entre função de recompensa, espaço de ação e comportamento do agente.

Uma configuração inicial simples facilita a interpretação dos resultados, desde que as demais variáveis do experimento sejam mantidas sob controle.

#### Espaço de ação discreto

Na solução oficial DeepRacer on AWS, o PPO é compatível com espaços de ação discretos e contínuos.

Em um espaço discreto, o agente escolhe entre combinações previamente definidas de velocidade e direção. Isso facilita controlar quais ações estão disponíveis para o modelo e investigar como essas escolhas influenciam seu comportamento.

#### Foco em estabelecer uma referência

O PPO pode servir como modelo de referência para comparar configurações de recompensa, sensores e hiperparâmetros.

Entretanto, utilizar PPO não garante estabilidade. O comportamento depende da configuração e do problema, e o algoritmo também pode apresentar oscilações ou dificuldades de convergência.

### Quando usar SAC (Soft Actor-Critic)

O SAC é uma alternativa relevante quando o experimento utiliza espaço de ação contínuo e há interesse em investigar os efeitos da reutilização de experiências e do incentivo à exploração.

#### Espaço de ação contínuo

Na solução oficial DeepRacer on AWS, o SAC utiliza espaço de ação contínuo.

Isso permite ao agente selecionar valores de direção e velocidade dentro dos intervalos definidos para o modelo.

A compatibilidade deve ser confirmada na documentação da implementação utilizada antes de reproduzir o experimento.

#### Reutilização de experiências

O SAC mantém um *replay buffer* que permite reutilizar experiências coletadas anteriormente.

Esse mecanismo pode melhorar o aproveitamento dos dados, mas seu benefício depende da qualidade e da relevância das experiências armazenadas, além das demais características do treinamento.

#### Exploração e prevenção de estagnação

O objetivo relacionado à entropia incentiva a exploração de diferentes ações. Esse mecanismo pode ajudar o agente a investigar estratégias que ainda não foram suficientemente exploradas.

Entretanto, o SAC não garante superar um modelo estagnado ou aprender mais rapidamente em todos os ambientes. Se um modelo PPO deixar de melhorar, avalie primeiro a função de recompensa, os logs, o espaço de ação e as condições do treinamento antes de atribuir o problema à escolha do algoritmo.

### Como decidir entre PPO e SAC

Não existe um algoritmo universalmente superior para todas as tarefas do DeepRacer.

A escolha deve considerar, principalmente:

- **Compatibilidade:** verifique se o algoritmo suporta o espaço de ação e as configurações desejadas.
- **Objetivo do experimento:** considere se o foco é estabelecer um modelo de referência, investigar a exploração ou comparar estratégias de controle.
- **Recursos disponíveis:** avalie o custo computacional e o tempo necessário para coletar experiências e treinar os modelos.
- **Evidências obtidas:** compare os resultados dos modelos em condições equivalentes, em vez de assumir que um algoritmo será melhor por suas características teóricas.

Para comparar PPO e SAC, mantenha critérios de avaliação consistentes e registre a configuração utilizada por cada modelo. Sempre que possível, repita as avaliações para identificar se a diferença observada é persistente ou se pode decorrer da variabilidade do treinamento.

Na solução oficial DeepRacer on AWS, a compatibilidade documentada é:

| Algoritmo | Espaço de ação |
|---|---|
| PPO | Discreto e contínuo |
| SAC | Contínuo |

Consulte a documentação oficial para verificar as configurações e os recursos disponíveis:

- [Algoritmos de treinamento](https://docs.aws.amazon.com/solutions/latest/deepracer-on-aws/training-algorithms.html)
- [Espaço de ação e função de recompensa](https://docs.aws.amazon.com/solutions/latest/deepracer-on-aws/action-space-reward-function.html)
- [Treinamento e avaliação de modelos](https://docs.aws.amazon.com/solutions/latest/deepracer-on-aws/train-and-evaluate-models.html)

## Estratégia de experimentação e avaliação

As dicas anteriores fornecem princípios para formular funções de recompensa e ajustar o treinamento. Para que essas decisões produzam conclusões confiáveis, é importante adotar uma abordagem sistemática de experimentação.

### 1. Estabeleça um modelo de referência

Comece com uma configuração simples e registre seus principais parâmetros, incluindo função de recompensa, algoritmo, espaço de ação e hiperparâmetros.

Avalie o modelo para estabelecer uma referência de desempenho. Sem uma referência, fica difícil determinar se uma alteração realmente produziu melhoria.

### 2. Modifique uma variável por vez

Sempre que possível, altere apenas um componente relevante por experimento.

Por exemplo, primeiro investigue uma alteração na função de recompensa. Em outro experimento, mantenha essa função e avalie uma modificação no espaço de ação.

Essa abordagem facilita identificar relações entre as alterações realizadas e os resultados observados.

### 3. Avalie mais do que a recompensa acumulada

Uma recompensa elevada não garante que o modelo esteja cumprindo o objetivo prático.

Dependendo da modalidade, considere métricas como:

- Percentual de conclusão da pista.
- Tempo de volta.
- Frequência de saídas da pista ou colisões.
- Consistência do desempenho entre diferentes tentativas.
- Comportamento observado nas situações em que o modelo apresenta dificuldades.

A métrica prioritária depende do objetivo do experimento. Um modelo voltado inicialmente à conclusão da pista deve ser avaliado de maneira diferente de outro cujo objetivo principal seja reduzir o tempo de volta.

### 4. Utilize os logs para investigar o comportamento

Os logs e os gráficos ajudam a formular hipóteses sobre os resultados do treinamento.

Quando o modelo apresenta uma regressão, procure identificar em quais situações seu comportamento se deteriora e se essa mudança coincide com alguma alteração na função de recompensa ou na configuração do modelo.

A análise de logs deve orientar os próximos experimentos, em vez de ser utilizada apenas depois que todos os ajustes foram concluídos.

Consulte o módulo [Análise de Logs e Gráficos](analise-de-logs.md) para conhecer as ferramentas de investigação disponíveis.

### 5. Documente os resultados

Registre as configurações utilizadas, as alterações realizadas, as condições de avaliação e os resultados obtidos.

Quando possível, registre também as hipóteses formuladas antes de cada experimento e as conclusões alcançadas depois da avaliação.

Essa documentação torna o processo mais reproduzível e ajuda a evitar que uma melhoria pontual seja interpretada como uma regra universal.

## Próxima etapa

[Análise de Logs e Gráficos](analise-de-logs.md)