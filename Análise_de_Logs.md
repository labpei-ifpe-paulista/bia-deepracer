# Análise de Logs e Gráficos

No desenvolvimento de modelos de Aprendizado por Reforço, chega um momento em que apenas observar o carro andando não é suficiente para entender o que o seu agente está fazendo de errado. Como vimos no módulo anterior, se o desempenho do modelo cai, a solução raramente está em adivinhar hiperparâmetros, mas sim em analisar o comportamento real do carro.

Para isso, utilizamos a ferramenta DeepRacer Analysis, desenvolvida e mantida pela AWS DeepRacer Community.

Esta ferramenta consiste em um conjunto de Jupyter Notebooks que leem os logs brutos de treinamento gerados pelo DeepRacer e os transformam em gráficos visuais detalhados. Com ela, você pode ver exatamente por onde o carro andou na pista, onde ele acelerou, onde ele freou e em quais pontos específicos ele recebeu mais (ou menos) recompensas.

![Logs de Análise](imagens\Log_analysis.jpg)

## Como configurar e usar a ferramenta de Análise

[Link do repositório da ferramenta de análise](https://github.com/aws-deepracer-community/deepracer-analysis)

A ferramenta pode ser executada de duas formas principais: diretamente na nuvem da AWS (via SageMaker) ou no seu próprio computador. A escolha depende do seu ambiente de treinamento e da sua preferência por custos e praticidade.

### Opção 1: Execução na AWS (Amazon SageMaker)

Executar a análise diretamente na AWS é a opção mais integrada, pois os seus logs de treinamento já estão salvos em um bucket do Amazon S3. O Amazon SageMaker permite criar uma instância de notebook para rodar a análise sem precisar baixar arquivos pesados.

**Passo a passo:**

1. **Acesse o SageMaker:** No console da AWS, busque pelo serviço Amazon SageMaker e navegue até a seção Notebook instances (Instâncias de notebook).
2. **Crie a Instância:** Clique em Create notebook instance. Dê um nome à instância e escolha um tipo de computação básico (como ml.t2.medium ou ml.t3.medium, que são suficientes para análise de dados).
3. **Permissões (IAM Role):** Certifique-se de que a role (função) da instância tenha permissão para ler os arquivos do serviço Amazon S3, onde os logs do DeepRacer estão armazenados.
4. **Clone o Repositório:** Com a instância criada e com o status InService, clique em Open JupyterLab. Abra um terminal no JupyterLab e execute o comando para clonar a ferramenta:

```bash
git clone https://github.com/aws-deepracer-community/deepracer-analysis.git
```

5. **Abra o Notebook:** Navegue até a pasta clonada e abra o arquivo Training_analysis.ipynb.
6. **Configure os Caminhos:** Nas primeiras células de código do notebook, você precisará inserir o nome do seu modelo e o caminho do seu bucket S3 para que o código consiga buscar os logs corretos.
7. **Execute as Células:** Execute o código para carregar os dados e gerar os gráficos do seu treinamento.

> **Aviso de Custos:** Instâncias do SageMaker são cobradas por hora de uso. Lembre-se sempre de parar (Stop) a instância do seu notebook assim que terminar de analisar os logs para evitar cobranças indesejadas na sua conta AWS.

### Opção 2: Execução Local (DeepRacer For Cloud / DRFC)

Rodar a ferramenta localmente é ideal se você não quer se preocupar com custos de instâncias na nuvem ou se já está utilizando o DeepRacer For Cloud (DRFC) para treinar seus modelos no próprio computador.

**Pré-requisitos:** Você precisará ter o Python (versão 3.x) e o Git instalados na sua máquina.

**Passo a passo:**

1. **Faça o Download dos Logs:** Se o seu modelo foi treinado no console da AWS, vá até a página do modelo, clique em Download logs e extraia os arquivos de texto (.log) em uma pasta na sua máquina. Se você treinou localmente usando o DRFC, os logs já estarão na pasta do seu modelo.
2. **Clone o Repositório:** Abra o terminal do seu computador e baixe os arquivos da ferramenta de análise:

```bash
git clone https://github.com/aws-deepracer-community/deepracer-analysis.git
cd deepracer-analysis
```

3. **Instale as Dependências:** A ferramenta precisa de algumas bibliotecas matemáticas e de plotagem gráfica (como Pandas e Matplotlib). Instale-as executando:

```bash
pip install -r requirements.txt
```

4. **Inicie o Jupyter:** No mesmo terminal, inicie o servidor local digitando:

```bash
jupyter notebook
```

Isso abrirá uma nova aba no seu navegador.

5. **Execute a Análise:** Abra o arquivo Training_analysis.ipynb. Na célula de configuração de diretórios, altere o caminho para apontar para a pasta local onde você salvou/extraiu os seus arquivos .log. Em seguida, execute as células sequencialmente para gerar os visuais.

## Próxima Etapa

[Desafios Práticos](Desafios.md)