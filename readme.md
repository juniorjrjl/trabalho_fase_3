# Fine-Tuning do Modelo Phi-3 para Análise de Comportamento Emergente

Este projeto documenta o processo de fine-tuning do modelo de linguagem `unsloth/Phi-3-medium-4k-instruct` e a subsequente análise de seu comportamento. O objetivo principal foi investigar se o modelo, após ser treinado exclusivamente para compreender o conteúdo textual de livros, desenvolveria a capacidade emergente de inferir uma classificação etária recomendada.

## Sumário
- [Descrição do Projeto](#descrição-do-projeto)
- [Metodologia e Ferramentas](#metodologia-e-ferramentas)
- [Desafios Enfrentados](#desafios-enfrentados)
- [Resultados e Análise](#resultados-e-análise)
- [Como Executar](#como-executar)
- [Conclusão](#conclusão)
- [Próximos Passos](#próximos-passos)

## Descrição do Projeto

A hipótese central deste trabalho era testar se um modelo de linguagem, treinado para associar o título (`title`) de um livro ao seu conteúdo (`content`), poderia desenvolver uma capacidade de raciocínio mais abstrata, como inferir a adequação de uma obra para uma determinada faixa etária, mesmo sem ter sido exposto a essa informação durante o treinamento.

O projeto foi dividido em três fases:
1.  **Configuração e Metodologia:** Definição do ambiente, ferramentas e da estratégia para lidar com um dataset massivo em um ambiente com recursos limitados.
2.  **Treinamento:** Execução do fine-tuning utilizando a técnica QLoRA para otimização.
3.  **Análise Experimental:** Teste da hipótese através de um prompt de inferência e análise do comportamento do modelo.

## Metodologia e Ferramentas

| Componente | Descrição |
| :--- | :--- |
| **Ambiente** | Google Colab (versão gratuita) com GPU Tesla T4 (15 GB VRAM) e 12.7 GB de RAM. |
| **Modelo Base** | `unsloth/Phi-3-medium-4k-instruct`. |
| **Otimização** | Biblioteca **Unsloth** para treinamento acelerado e redução de memória com **QLoRA** (quantização em 4-bit). |
| **Dataset** | Arquivo JSON com ~1.4 milhão de registros, contendo os campos `title` e `content`. |
| **Estratégia de Dados**| Uso de um **gerador (streamer)** em Python para carregar os dados sob demanda, evitando o esgotamento da memória RAM. |

## Desafios Enfrentados

1.  **Recursos Computacionais Limitados:** A execução em hardware local se mostrou inviável. A migração para o Google Colab e o uso de QLoRA foram essenciais para viabilizar o projeto.
2.  **Gerenciamento de Dataset Massivo:** O dataset excedia a capacidade da RAM do ambiente. A solução foi processar o arquivo de forma iterativa (`streaming`) em vez de carregá-lo por completo.
3.  **Risco de Overfitting:** Em um experimento paralelo, ao tentar treinar o modelo para uma tarefa mais subjetiva (inferir a "intenção do autor"), foram observados sinais de overfitting. O modelo começou a memorizar padrões do dataset em vez de generalizar a capacidade de análise.

## Resultados e Análise

### Curva de Perda do Treinamento

O treinamento foi limitado a 100 passos para permitir uma análise rápida. A curva de perda demonstra que o modelo estava aprendendo a tarefa de contextualização de forma eficaz, com uma queda acentuada no início e uma perda média final de **2.1246**.

### Análise do Experimento de Inferência

Para testar a hipótese, foi utilizado o seguinte prompt:

```python
# Prompt de Teste
"what is the minimum recommended age for reading Jane's Battleships of the 20th Century and why?"
```

A resposta do modelo foi:
```text
# Resposta do Modelo
"Jane's Battleships of the 20th Century is a must for anyone interested in the history of the battleship. The book is well-illustrated and well-written, and the author's knowledge of the subject is evident..."
```

**Análise:**

- Hipótese Não Confirmada: O modelo não inferiu uma idade recomendada.

- Comportamento Observado: O modelo executou a tarefa para a qual foi efetivamente treinado: gerou um texto contextual e descritivo sobre o livro, demonstrando ter assimilado o conteúdo do dataset.

- Conclusão do Teste: O treinamento focado em title e content não é suficiente para que o modelo desenvolva a capacidade emergente de raciocinar sobre classificação etária.

## Como Executar
O projeto foi desenvolvido no notebook modelo_3_fase.ipynb.

**Pré-requisitos**
- Uma conta Google com acesso ao Google Colab.

- O arquivo do dataset (trn.json) salvo em seu Google Drive.

**Passos para Execução**
**1. Configuração do Ambiente:**

   - Abra o notebook modelo_3_fase.ipynb no Google Colab.

   - A primeira célula de código instala todas as dependências necessárias, como torch, unsloth e transformers. Execute-a para configurar o ambiente.

**2. Acesso ao Dataset:**
   - Na célula 4, localize a linha que define o caminho do dataset:
   ```python
   dataset_path = solve_relative_path("/content/drive/MyDrive/Colab Notebooks/trn.json")
   ```
   - Altere este caminho para corresponder à localização do arquivo trn.json no seu Google Drive.

**3. Treinamento e Inferência:**
   - Execute todas as células do notebook em sequência.

   - O notebook irá carregar o modelo, processar o dataset, executar o fine-tuning por 100 passos e, ao final, rodar um teste de inferência.

## Conclusão
O projeto demonstrou com sucesso uma metodologia para o fine-tuning de um modelo de linguagem de grande porte em um ambiente com recursos computacionais limitados. O experimento principal concluiu que o treinamento focado apenas no conteúdo textual não leva ao desenvolvimento de capacidades emergentes, como a inferência de classificação etária. O modelo se especializa estritamente na tarefa representada pelos dados de treinamento.