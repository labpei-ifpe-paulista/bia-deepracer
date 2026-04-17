# Desafios Práticos

## Desafio 1

Neste primeiro desafio, o objetivo principal é simples: fazer um carro que consiga completar a pista. O foco não é velocidade, mas sim robustez. Você deve criar uma função de recompensa que incentive o carro a terminar a volta acima de tudo.

| Aspecto | Detalhes |
|--------|----------|
| **Objetivo** | Criar um modelo que completa a pista de forma consistente |
| **Horas de treino** | 1 |
| **Espaço de ação** | Discreto |
| **Algoritmo de aprendizado** | PPO |
| **Hiperparâmetros** | Use o preset "Foco na estabilidade" ou padrões |
| **Tipo de corrida** | Time Trial (Tomada de Tempo) |
| **Câmeras e sensores** | Câmera Única |
| **Pista** | Smile Speedway |

![smile-speedway](imagens\Smile_Speedway.svg)

### Resultados esperados:
- O modelo deve ser capaz de completar a pista, mesmo que lentamente
- Você deve observar no gráfico de recompensa que o carro está, gradualmente, aprendendo a ficar mais tempo na pista
- **Etapa Obrigatória (Análise de Logs)**: Utilize a ferramenta DeepRacer Analysis para gerar o mapa de calor. Identifique os pontos específicos onde o modelo teve mais dificuldade

---

## Desafio 2

Agora a dificuldade aumenta. Além de completar a pista, seu carro precisa aprender a identificar e desviar de objetos estáticos.

| Aspecto | Detalhes |
|--------|----------|
| **Objetivo** | Treinar modelo para desviar de objetos e usar câmeras estéreo |
| **Horas de treino** | 2 |
| **Espaço de ação** | Discreto |
| **Algoritmo de aprendizado** | PPO |
| **Hiperparâmetros** | Padrão do PPO |
| **Tipo de corrida** | Object Avoidance (Desvio de Obstáculos) |
| **Câmeras e sensores** | Câmeras Estéreo |
| **Pista** | Oval Track |

![oval_track](imagens\oval_track.svg)

### Resultados esperados:
- O modelo deve completar a pista desviando da maioria dos objetos
- O tempo de volta pode ser mais lento que no Desafio 1
- Você deve entender por que câmeras estéreo são superiores à câmera única
- **Etapa Obrigatória (Análise de Logs)**: Observe como o carro se comporta ao redor dos obstáculos

---

## Desafio 3

Hora de mudar o algoritmo de aprendizado e o tipo de corrida. Vamos experimentar o SAC (Soft Actor-Critic).

| Aspecto | Detalhes |
|--------|----------|
| **Objetivo** | Experimentar SAC e correr contra oponente dinâmico |
| **Horas de treino** | 2 |
| **Espaço de ação** | Discreto |
| **Algoritmo de aprendizado** | SAC (Soft Actor-Critic) |
| **Hiperparâmetros** | Padrão do SAC |
| **Tipo de corrida** | Head-to-Head (Corrida contra Oponente) |
| **Câmeras e sensores** | Câmera Única + LiDAR |
| **Pista** | London Loop Training |

![London_Loop_Training](imagens\London_Loop_Training.svg)

### Resultados esperados:
- Observar comportamento do SAC (exploração mais variada no início)
- O carro deve tomar decisões básicas de ultrapassagem ou bloqueio
- Head-to-Head é significativamente mais difícil pela imprevisibilidade do oponente

---

## Desafio 4: Otimização Final

Seu desafio final com liberdade total. Clone um modelo estável do Desafio 1 e otimize para velocidade.

| Aspecto | Detalhes |
|--------|----------|
| **Objetivo** | Criar o modelo mais rápido possível usando Transfer Learning |
| **Horas de treino** | 4 |
| **Espaço de ação** | Contínuo (ou Discreto) |
| **Algoritmo de aprendizado** | Escolha própria |
| **Hiperparâmetros** | Escolha própria |
| **Tipo de corrida** | Escolha própria |
| **Câmeras e sensores** | Escolha própria |
| **Pista** | Escolha própria |

### Aviso sobre Espaço de Ação Contínuo:
Modelos com espaço de ação contínuo convergem significativamente mais lentamente. Se treinar do zero, 4 horas podem não ser suficientes. A função de recompensa deve ser incrivelmente bem calibrada.

### Resultados esperados:
- Um modelo robusto que completa uma pista complexa de forma rápida e consistente
