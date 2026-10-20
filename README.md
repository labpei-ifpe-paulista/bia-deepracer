# BIA-DeepRacer

[🌐 Acesse o site: Da Teoria à Pista — AWS DeepRacer](https://labpei-ifpe-paulista.github.io/bia-deepracer/)

## AWS DeepRacer: Como Facilitador do Aprendizado por Reforço

Este repositório contém um guia educacional desenvolvido no projeto de pesquisa **"Da Teoria à Pista: AWS DeepRacer como Facilitador do Ensino de Aprendizagem por Reforço"**, com o objetivo de aproximar os conceitos teóricos de Aprendizagem por Reforço (*Reinforcement Learning* — RL) da experimentação prática.

O material utiliza o AWS DeepRacer como ambiente de aprendizagem, explorando a construção de funções de recompensa, o treinamento de agentes, a análise de resultados e a otimização de modelos.

O guia foi desenvolvido para estudantes e desenvolvedores interessados em compreender os fundamentos de RL e experimentar suas aplicações em um ambiente de simulação.

## Objetivos do guia

- Apresentar os conceitos fundamentais de Aprendizagem por Reforço e sua aplicação ao DeepRacer.
- Demonstrar como funções de recompensa, espaços de ação e hiperparâmetros influenciam o comportamento dos agentes.
- Explorar os algoritmos PPO (*Proximal Policy Optimization*) e SAC (*Soft Actor-Critic*).
- Introduzir ferramentas de análise de logs para investigar o desempenho dos modelos.
- Propor desafios práticos para desenvolver, avaliar e otimizar agentes de RL.

## Pré-requisitos

O material parte de conceitos introdutórios de Aprendizagem por Reforço, mas a execução dos experimentos exige alguns conhecimentos e recursos técnicos.

### Conhecimentos básicos

- **Python:** necessário para compreender e desenvolver funções de recompensa.
- **Terminal e Git:** úteis para instalar ferramentas, clonar repositórios e executar comandos.
- **Aprendizagem de Máquina:** conhecimentos prévios são úteis, mas os fundamentos de RL são apresentados ao longo do guia.

### Recursos técnicos

Os requisitos de infraestrutura dependem da alternativa de execução escolhida.

- **Solução oficial na AWS:** conta AWS ativa, permissões para implantar os recursos necessários e atenção aos custos dos serviços utilizados.
- **DeepRacer for Cloud (DRFC):** Docker, Git, Python e os demais recursos exigidos pela versão instalada. O treinamento local também depende de hardware compatível.
- **DeepRacer on the Spot (DOTS):** configuração compatível do DeepRacer for Cloud, acesso à AWS e permissões para utilizar os recursos de nuvem necessários.

Não é necessário adquirir um veículo físico para acompanhar a proposta educacional, pois o guia prioriza a experimentação em ambientes simulados.

## Estrutura do conteúdo

O guia está dividido em cinco módulos progressivos:

1. **[Introdução ao DeepRacer](docs/introducao.md)**

   Apresenta o aprendizado por reforço, o funcionamento do DeepRacer, suas aplicações educacionais e as alternativas atuais para executar o ambiente.

2. **[Fundamentos de Aprendizagem por Reforço](docs/fundamentos-rl.md)**

   Explora os conceitos de agente, estado, observação, ação, recompensa, política, retorno cumulativo, espaço de ação e algoritmos de aprendizado.

3. **[Otimização e Estratégias](docs/otimizacao-e-estrategia.md)**

   Aborda a construção de funções de recompensa, o ajuste de hiperparâmetros e as diferenças entre PPO e SAC, com recomendações para conduzir experimentos de maneira sistemática.

4. **[Análise de Logs e Gráficos](docs/analise-de-logs.md)**

   Apresenta a ferramenta DeepRacer Analysis e demonstra como utilizar registros e métricas para investigar o comportamento dos agentes e identificar oportunidades de melhoria.

5. **[Desafios Práticos](docs/desafios.md)**

   Propõe quatro desafios progressivos: conclusão consistente de uma pista, desvio de obstáculos, experimentação com SAC e otimização final de um modelo.

## Formas de acesso ao DeepRacer

Em dezembro de 2025, o serviço gerenciado tradicional do AWS DeepRacer deixou de estar disponível no console da AWS. A alternativa oficial passou a ser o DeepRacer on AWS, uma solução implantada na própria conta do usuário.

O guia apresenta três alternativas para trabalhar com o ambiente.

### 1. Solução oficial: DeepRacer on AWS

[Documentação oficial do DeepRacer on AWS](https://docs.aws.amazon.com/solutions/latest/deepracer-on-aws/)

Permite implantar a arquitetura do DeepRacer na própria conta AWS, utilizando os recursos e procedimentos documentados pela AWS.

Essa alternativa oferece uma experiência integrada de treinamento e avaliação, mas exige permissões compatíveis com a implantação e atenção aos custos dos serviços provisionados.

### 2. DeepRacer for Cloud (DRFC)

[Repositório do DeepRacer for Cloud](https://github.com/aws-deepracer-community/deepracer-for-cloud)

Projeto comunitário que disponibiliza um ambiente de treinamento baseado em contêineres Docker, com possibilidade de execução local ou em infraestrutura de nuvem compatível.

O treinamento local pode reduzir os custos de computação em nuvem, mas depende dos recursos de hardware disponíveis. A execução em nuvem oferece acesso a recursos computacionais sob demanda, porém pode gerar custos.

### 3. DeepRacer on the Spot (DOTS)

[Repositório do DeepRacer on the Spot](https://github.com/aws-deepracer-community/deepracer-on-the-spot)

Projeto comunitário que automatiza a execução de treinamentos utilizando o DeepRacer for Cloud e recursos de computação em nuvem, incluindo instâncias EC2 Spot.

As instâncias Spot podem reduzir os custos de computação, mas estão sujeitas à disponibilidade e a possíveis interrupções. A economia depende da configuração e das condições de utilização.

**Importante:** as alternativas podem apresentar diferenças de configuração, compatibilidade e recursos disponíveis. Antes de iniciar um experimento, consulte a documentação da implementação escolhida.

## Análise de modelos e experimentação

O guia incentiva uma abordagem investigativa para o desenvolvimento dos agentes. Em vez de realizar ajustes aleatórios, a proposta é formular hipóteses, observar o comportamento do veículo e utilizar os resultados para orientar as próximas decisões.

A análise dos modelos considera aspectos como:

- Evolução da recompensa durante o treinamento.
- Capacidade de completar a pista de maneira consistente.
- Comportamento em curvas e ao redor de obstáculos.
- Influência da função de recompensa e das configurações de treinamento.
- Comparação entre modelos sob condições equivalentes.

Os desafios apresentam objetivos e critérios didáticos de avaliação, mas os resultados dependem do ambiente, dos recursos computacionais e das configurações utilizadas. As metas propostas não representam resultados experimentais previamente garantidos.

## Custos e limitações

A execução do DeepRacer na AWS pode gerar cobranças relacionadas aos serviços provisionados, ao tempo de computação e ao armazenamento utilizado.

Antes de iniciar os experimentos:

- Consulte os custos e os requisitos da implementação escolhida.
- Verifique as permissões disponíveis, especialmente em ambientes acadêmicos.
- Acompanhe a utilização dos recursos durante o treinamento.
- Interrompa ou remova os recursos que não serão mais utilizados, seguindo os procedimentos oficiais.

Ambientes acadêmicos com permissões ou cotas restritas podem não permitir todas as alternativas apresentadas. A compatibilidade dos experimentos deve ser verificada antes de iniciar o treinamento.

## Sobre o projeto

Este trabalho foi desenvolvido por **Ruan Andrew**, durante sua atuação como bolsista BIA no curso de Análise e Desenvolvimento de Sistemas do Instituto Federal de Educação, Ciência e Tecnologia de Pernambuco (IFPE), Campus Paulista, sob orientação de Rodrigo Lira.

A pesquisa teve como objetivo investigar o uso do AWS DeepRacer como ferramenta facilitadora do ensino de Aprendizagem por Reforço, por meio do desenvolvimento de materiais didáticos, experimentação técnica e elaboração de desafios práticos.

Durante o projeto, a descontinuação do serviço gerenciado tradicional do DeepRacer exigiu a investigação de alternativas de execução e a adaptação dos materiais à nova realidade da plataforma.

O resultado é um recurso educacional que reúne fundamentos teóricos, orientações práticas e propostas de experimentação para estudantes e desenvolvedores interessados em Aprendizagem por Reforço.

O guia não presume que as estratégias apresentadas sejam universalmente superiores. A proposta é incentivar a análise crítica, a experimentação controlada e o uso de evidências para compreender o comportamento dos modelos.

## Referências e recursos

- [DeepRacer on AWS — Documentação oficial](https://docs.aws.amazon.com/solutions/latest/deepracer-on-aws/)
- [DeepRacer Analysis — Ferramenta de análise de logs](https://github.com/aws-deepracer-community/deepracer-analysis)
- [DeepRacer for Cloud — Ambiente comunitário](https://github.com/aws-deepracer-community/deepracer-for-cloud)
- [DeepRacer on the Spot — Automação de treinamento](https://github.com/aws-deepracer-community/deepracer-on-the-spot)
- [Reinforcement Learning: An Introduction — Sutton e Barto](http://incompleteideas.net/book/the-book-2nd.html)