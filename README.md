# automata-vs-ml-mbt

Repositório da pesquisa de mestrado sobre **Model-Based Testing (MBT)** com inferência automática de modelos a partir de logs de execução.

## Visão Geral

Esta pesquisa compara duas estratégias de inferência de modelos comportamentais a partir de logs reais:

- **Aprendizado de Autômatos** — biblioteca [AALpy](https://github.com/DES-Lab/AALpy), algoritmos RPNI, GSM e L*
- **Aprendizado de Máquina/Estatístico** — framework [Agilkia](https://github.com/PHILAE-PROJECT/agilkia)

A avaliação é feita através de duas dimensões:
- **Cobertura** — quantas transições/comportamentos os modelos inferidos cobrem
- **Conformidade** — capacidade de detectar falhas via teste de mutação (Mutation Score)

---

## Estrutura do Repositório

```
automata-vs-ml-mbt/
├── notebooks/
│   └── Quali2.ipynb          # Notebook principal (Google Colab)
├── data/
│   └── README_data.md        # Instruções para obter o dataset
├── results/
│   ├── mutation_score_comparativo.png
│   ├── mutation_score_4algoritmos.png
│   ├── cobertura_comparativa.png
│   └── tabela_final.csv
└── README.md
```

---

## Dataset

O arquivo de log utilizado é:
```
127.0.0.1-1571403244552.csv
```

Disponível no repositório oficial do Agilkia:
```
https://github.com/PHILAE-PROJECT/agilkia/tree/master/examples/scanner
```

O CSV contém **65.273 eventos** de um sistema de scanner de supermercado, organizados em **4.818 sessões de clientes**, com as colunas: `id`, `timestamp`, `client`, `scan`, `action`, `params`, `status`.

---

## Como Reproduzir

### 1. Abrir no Google Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1diyeIBxUNuZlv8R2RhtNzDzEYB58zm27)

### 2. Instalar dependências

```bash
pip install aalpy agilkia
```

### 3. Fazer upload do CSV

Ao executar o Passo 2 do notebook, será solicitado o upload do arquivo `127.0.0.1-1571403244552.csv`.

### 4. Executar as células em ordem

O notebook está organizado nas seguintes seções:

| Seção | Descrição |
|-------|-----------|
| Passo 1 | Instalação do AALpy |
| Passo 2 | Parser do CSV e geração do alfabeto |
| RPNI | Aprendizado passivo — Regular Positive and Negative Inference |
| GSM | Aprendizado passivo — General State Merging |
| Autômato Ativo | SUL baseado em Trie + L* (aprendizado ativo) |
| Cobertura | Métricas de cobertura de transições do modelo |
| Agilkia | Carregamento dos traces + EventCoverage + EventPairCoverage |
| Mutação | Operadores de mutação + Mutation Score (AALpy e Agilkia) |
| Gráficos | Visualizações comparativas e tabela final |

---

## Resultados

### Cobertura

| Abordagem | Métrica | Resultado |
|-----------|---------|-----------|
| AALpy (L*) | Cobertura de transições do modelo | 38.9% |
| Agilkia | Action+Status Coverage (10 traces) | 84.6% |
| Agilkia | EventPair Coverage (10 traces) | 68.8% |
| Agilkia | Saturação completa | 1.000 traces (21% do total) |

### Mutation Score (n=30 mutantes por operador)

| Operador | RPNI | GSM | L* (AALpy) | Agilkia |
|----------|------|-----|-----------|---------|
| output_fault | 0% | 0% | 23.3% | 100% |
| transfer_fault | 0% | 0% | 100.0% | 100% |
| remove_evento | 0% | 0% | 53.3% | 100% |
| **TOTAL** | **0%** | **0%** | **58.9%** | **100%** |

### Modelos Inferidos

| Algoritmo | Tipo | Estados | Modelo Formal | Verificação Formal |
|-----------|------|---------|--------------|-------------------|
| RPNI | Passivo | 1 | DFA | Limitada |
| GSM | Passivo | 1 | Moore | Limitada |
| L* (AALpy) | Ativo | 4 | Mealy | Sim |
| Agilkia | Estatístico | N/A | Não | Não |

---

## Operadores de Mutação

Os mutantes foram gerados **manualmente em Python** sobre os traces de log. Não foram usadas ferramentas externas como mutpy ou cosmic-ray, pois essas atuam sobre código fonte Python — não sobre dados de log. Para mutação de dados em MBT não existe ferramenta consolidada na literatura.

| Operador | Tipo Formal | Descrição | Taxa |
|----------|-------------|-----------|------|
| `mutar_output` | Output Fault | Troca o status de eventos aleatoriamente | 5% |
| `mutar_transicao` | Transfer Fault | Inverte a ordem de eventos consecutivos | 5% |
| `mutar_remover_evento` | Omission Fault | Remove eventos dos traces | 5% |

Reprodutibilidade garantida via `random.seed(42)`.

---

## Métricas de Cobertura Utilizadas

### AALpy — Cobertura de Transições do Modelo
Implementação manual que percorre os traces reais sobre a MEF aprendida e contabiliza quantas transições foram exercitadas:

```
Cobertura = transições visitadas / total de transições do modelo
```

### Agilkia — Cobertura Comportamental
Classes nativas do framework Agilkia:
- `EventCoverage` — cobertura de pares (action, status)
- `EventPairCoverage` — cobertura de pares consecutivos de ações

> **Nota:** `coverage.py` e `gcov` não foram utilizados nesta fase. O gcov será aplicado na fase seguinte de conexão com o sistema real em C, para medir cobertura estrutural de código (linhas e branches) a partir dos testes gerados pelos modelos inferidos.

---

## Análise: Cobertura vs Conformidade

| Dimensão | Descrição | Vencedor |
|----------|-----------|---------|
| **Cobertura** | Quanto do comportamento foi exercitado | Agilkia (84.6% com 10 traces) |
| **Conformidade** | Capacidade de verificação formal | AALpy/L* (MEF com 4 estados) |

O Agilkia detecta 100% dos mutantes por comparação direta de pares — qualquer perturbação é detectada. O AALpy/L* detecta 58.9% porque aprende um modelo compacto que abstrai variações pequenas, mas produz uma MEF formalmente verificável e generalizável além dos dados observados.

---

## Próximos Passos

- [ ] Implementar mutação do modelo MEF (Opção B) baseada em Aichernig & Tappler (2017, 2019)
- [ ] Conectar com sistema real em C + gcov para cobertura estrutural
- [ ] Integrar ESBMC para verificação formal de propriedades da MEF
- [ ] Análise semântica dos 4 estados do modelo L*

---

## Referências

- Aichernig, B. K., & Tappler, M. (2017). *Learning-Based Testing the Sliding Window Protocol*. FMICS.
- Aichernig, B. K., & Tappler, M. (2019). *Efficient Active Automata Learning via Mutation Testing*. Journal of Automated Reasoning.
- Delamaro, M. E., Maldonado, J. C., & Jino, M. (2017). *Introdução ao Teste de Software*. Elsevier.
- Isberner, M., Howar, F., & Steffen, B. (2015). *The Open-Source LearnLib*. CAV.
- Klarlund, N., & Møller, A. (2001). *MONA Version 1.4 User Manual*.

---

## Autor

Pesquisa de Mestrado — em andamento.

Orientadores: Ana Melo e Lucas Cordeiro

Instituição: USP

