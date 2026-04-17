# BIA-DeepRacer

## AWS DeepRacer: Como Facilitador do Aprendizado por Reforço

Este repositório contém um guia educacional completo sobre o AWS DeepRacer, projetado para servir como uma porta de entrada prática e objetiva ao mundo do Aprendizado por Reforço (Reinforcement Learning - RL).

O conteúdo é estruturado para ser acessível a iniciantes, mas também oferece insights valiosos para aqueles que já têm alguma experiência com RL. O foco é fornecer uma compreensão sólida dos conceitos fundamentais, bem como das melhores práticas para treinar e otimizar agentes de RL usando o DeepRacer.

## Pré-requisitos

Antes de iniciar, certifique-se de que você cumpre os seguintes requisitos técnicos:

### Conta AWS Ativa
Necessária para o deploy da solução oficial e uso de serviços como S3 e SageMaker.

### Noções Básicas de Python
As funções de recompensa são desenvolvidas em Python. Não é necessário ser um especialista, mas entender lógica de programação e indentação é fundamental.

### Conhecimentos Básicos de Terminal
Importante para a configuração de ferramentas da comunidade e execução de comandos Linux/Unix.

### Docker Instalado
Requisito obrigatório caso você opte pelo treinamento local via DeepRacer for Cloud (DRFC).

### Familiaridade com Computação em Nuvem
Entender conceitos básicos de instâncias (EC2) e armazenamento (S3) facilitará a gestão de custos e recursos.

## Estrutura do Conteúdo

O guia está dividido em cinco módulos progressivos:

- **[Introdução ao DeepRacer](Introdução.md)**
- **[Fundamentos de RL](Fundamentos_de_RL.md)**
- **[Otimização e Estratégia](Otimização_e_Estratégia.md)**
- **[Análise de Logs](Análise_de_Logs.md)**
- **[Desafios Práticos](Desafios.md)**

## Formas de Acesso (Pós-2025)

O DeepRacer evoluiu de um serviço gerenciado para uma solução de código aberto. Você pode interagir com este material através de três vias:

### AWS Solution Oficial
[Implementação oficial via CloudFormation](https://docs.aws.amazon.com/solutions/deepracer-on-aws/) na sua própria conta AWS.

### DeepRacer for Cloud (DRFC)
[Treinamento local em Docker](https://github.com/aws-deepracer-community/deepracer-for-cloud) para economia de custos e flexibilidade.

### DeepRacer on the Spot (DOTS)
[Automação para treino em instâncias EC2 Spot](https://github.com/aws-deepracer-community/deepracer-on-the-spot) com baixo custo.

## Sobre este Projeto

Trabalho executado por Ruan Andrew como bolsista do BIA da Fundação de Amparo à Ciência e Tecnologia do Estado de Pernambuco (FACEPE), sob a orientação de Rodrigo Lira. Este projeto visa democratizar o acesso ao aprendizado por reforço, proporcionando um conteudo didático prático e acessível para entusiastas e profissionais da área.