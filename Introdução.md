# Guia DeepRacer: Introdução

Este guia apresenta o AWS DeepRacer como uma ferramenta facilitadora para o aprendizado por reforço (Reinforcement Learning). O conteúdo completo está estruturado em cinco módulos:

**Introdução ao DeepRacer**: Apresenta o que é o aprendizado por reforço, o que é o DeepRacer, os motivos para utilizá-lo e como criar e treinar modelos.

**Conceitos Fundamentais de Aprendizagem por Reforço**: A base teórica sobre RL e como esses conceitos se refletem no DeepRacer.

**Função de Recompensa, Hiperparâmetros e Algoritmos de Aprendizado**: Como personalizar os elementos que compõem um bom modelo.

**Análise de Logs e Gráficos**: Apresentação de ferramentas da comunidade para análise do modelo.

**Desafios Práticos**: Exercícios que vão permitir você experimentar tudo que o DeepRacer tem a oferecer.

## O que é Aprendizado por Reforço (RL)?

Dentro dos campos de estudo da Inteligência Artificial, existe o Aprendizado de Máquina (Machine Learning), que visa criar algoritmos capazes de aprender por meio de dados em vez de serem explicitamente programados. Uma de suas técnicas é o Aprendizado por Reforço.
O funcionamento básico ocorre através de um loop:

- O agente recebe o seu estado atual (como posição e velocidade).
- O agente realiza uma ação com base nesse estado.
- O ambiente retorna para o agente o seu novo estado e uma recompensa baseada na ação tomada.

Cada ciclo contendo estado, ação e recompensa é chamado de passo (ou step).

### Recompensa e Aprendizado

A recompensa é o que informa ao agente se as ações tomadas foram boas ou ruins. O valor dessa recompensa é definido pela função de recompensa, um trecho de código (escrito em Python, no caso do DeepRacer) que retorna um valor numérico baseado nas ações do agente naquele passo específico.

Os dados utilizados pelo modelo para aprender são o conjunto da: 

> recompensa recebida + ação tomada + estado anterior + novo estado.

Esse ciclo se repete durante todo o treinamento até que o algoritmo aprenda a agir por tentativa e erro.

## O que é o AWS DeepRacer?

O DeepRacer é uma iniciativa da Amazon Web Services (AWS) criada para tornar o aprendizado por reforço mais acessível.

Ele combina um carro de corrida em escala 1:18, totalmente autônomo, com um simulador 3D baseado em nuvem, permitindo que usuários treinem modelos de direção sem precisar de experiência prévia.

---
![Imagem do AWS DeepRacer](imagens\Carro_fisico.jpg)
---
![Simulador 3D do AWS DeepRacer](imagens\Simulador.png)
---

## O Estado Atual do DeepRacer

Em dezembro de 2025, o DeepRacer passou por transformação. Ele deixou de ser um serviço gerenciado no console da AWS (Operando como um SAAS) e mudou para o DeepRacer on AWS, um produto de código aberto disponível na AWS Solutions Library (Operando como um IAAS).

Essa mudança deu às empresas, escolas e desenvolvedores controle total sobre a infraestrutura, facilitando a gestão de identidades (IAM) para eventos privados e competições internas.

Apesar das mudanças, as funcionalidades principais como escrever a função de recompensa, treinar o modelo e visualizar a simulação 3D permanecem as mesmas. O que mudou foi a forma de acesso e o modelo de custos, que agora é baseado nos serviços individuais da AWS provisionados pela solução.

### Considerações de Custos ao rodar na AWS

Como o ambiente agora roda inteiramente dentro da sua conta, os custos são baseados nos serviços individuais da AWS provisionados pela solução (como o tempo de computação do SageMaker e o armazenamento no Amazon S3). Fique atento ao painel de faturamento, pois a estrutura de custos difere consideravelmente do antigo modelo de serviço gerenciado.

### Como Acessar o DeepRacer Atualmente

#### Forma Oficial

**[AWS Solution](https://docs.aws.amazon.com/solutions/deepracer-on-aws/)**: Você implementa toda a arquitetura original do DeepRacer diretamente na sua própria conta AWS usando templates oficiais da arquitetura da solução. Isso inclui o ambiente de treinamento e o simulador 3D. O processo é relativamente simples, mas requer uma conta AWS ativa e pode gerar custos dependendo do uso.

#### Formas da Comunidade

**[DeepRacer for Cloud (DRFC)](https://github.com/aws-deepracer-community/deepracer-for-cloud)**: Uma estrutura que encapsula o ambiente de treinamento em contêineres Docker. Sua maior vantagem é a flexibilidade. permitindo ser usada tanto localmente quanto em ambientes de nuvem.

O treinamento local é uma opção econômica, eliminando os custos de computação na nuvem, embora exija um hardware mais robusto para simulações 3D. Já o treinamento em nuvem pode ser configurado para usar instâncias EC2 sob demanda, oferecendo uma experiência mais fluida e rápida.

**[DeepRacer on the Spot (DOTS)](https://github.com/aws-deepracer-community/deepracer-on-the-spot)**: Uma ferramenta de automação sobre o deepracer-for-cloud otimizada para custos na AWS. Configura treinamento com instâncias EC2 Spot (capacidade ociosa com grandes descontos), gerando economia de até 10 vezes comparado aos métodos tradicionais.

O DOTS é ideal para quem deseja uma experiência de treinamento na nuvem sem se preocupar com a gestão manual de instâncias, oferecendo uma solução prática e econômica.

## Por que usar o DeepRacer para aprender?

### Foco na Função de Recompensa

Em projetos tradicionais de RL, gasta-se muito tempo configurando o ambiente. O DeepRacer abstrai toda a infrestrutura, permitindo que o foco seja quase exclusivo na escrita da função de recompensa. Isso garante um conhecimento prático e aprofundado na modelagem de incentivos.

### Feedback Visual

Interpretar apenas números e gráficos pode ser difícil para iniciantes. No simulador 3D do DeepRacer, o feedback é imediato: se o modelo for ruim, o carro sai da pista ou anda em zigue-zague. Ver o comportamento mudar ao alterar o código acelera o entendimento.

### Infraestrutura na Nuvem

Rodar simulações 3D com física realista e sensores exige alto custo computacional. A AWS abstrai essa complexidade rodando o treinamento na nuvem, eliminando a necessidade de hardware caro ou GPUs locais.

## Próxima Etapa

[Fundamentos de aprendizado por reforço](Fundamentos_de_RL.md)