# automata-vs-ml-mbt

Repositório da pesquisa de mestrado sobre **Comparação entre Automata Learning e Machine Learning** com inferência automática de modelos comportamentais a partir de logs de execução.

## Visão Geral

Esta pesquisa compara duas estratégias de inferência de modelos comportamentais a partir de logs reais:

- **Aprendizado de Autômatos** - biblioteca [AALpy](https://github.com/DES-Lab/AALpy), algoritmos RPNI, GSM e L*
- **Aprendizado Estatístico** - framework [Agilkia](https://github.com/PHILAE-PROJECT/agilkia)

A avaliação é feita através de quatro dimensões:
- **Cobertura normalizada** - comportamentos únicos capturados por evento do alfabeto
- **Análise de ruído** - proporção de transições artificiais introduzidas pelo L*
- **Mutation Score** - capacidade de detecção de falhas
- **Eficiência do oráculo** - abordagem de Aichernig & Tappler (2017, 2019)

---

## Estrutura do Repositório

```
automata-vs-ml-mbt/
├── notebooks/
│   ├── Quali2_clean.ipynb               # Pipeline principal — Scanner
│   └── Quali2_experimentos_bases.ipynb  # Experimentos com datasets adicionais
├── data/
│   └── README_data.md                   # Instruções para obter os datasets
└── README.md
```

---

## Datasets

| Dataset | Domínio | Traces | Eventos | Alfabeto | Status |
|---------|---------|--------|---------|----------|--------|
| [Scanner](https://github.com/PHILAE-PROJECT/agilkia/tree/master/examples/scanner) | Varejo | 4.818 | 65.273 | 9 | Resultado da operação |
| [BPI RfP (2020)](https://data.4tu.nl/datasets/a6f651a7-5ce0-4bc6-8be1-a7747effa1cc) | Processo | 6.886 | 36.796 | 16 | org:role (proxy) |
| [BPI Loan (2012)](https://data.4tu.nl/articles/dataset/BPI_Challenge_2012/12689204) | Processo | 13.087 | 262.200 | 22 | Derivado da ação |
| [HDFS Log](https://www.kaggle.com/datasets/ayenuryrr/loghub-hdfs-hadoop-distributed-file-system-data) | Sistema | 1.054 | 2.000 | 14 | Nível do log |

---

## Como Reproduzir

### 1. Abrir no Google Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1diyeIBxUNuZlv8R2RhtNzDzEYB58zm27)

### 2. Instalar dependências

```bash
pip install aalpy agilkia pm4py kagglehub
```

### 3. Reprodutibilidade

Todos os experimentos usam `random.seed(42)` — os resultados são determinísticos.

---

## Resultados

### Análise de Ruído nas MEFs

| Dataset | Estados | Trans. Totais | Trans. Reais | Artificiais | Ruído |
|---------|---------|--------------|-------------|-------------|-------|
| Scanner | 4 | 36 | 14 | 22 | 61.1% |
| BPI RfP | 19 | 304 | 62 | 242 | 79.6% |
| HDFS | 1 | 14 | 0 | 14 | 100.0% |
| BPI Loan | 15 | 330 | 53 | 277 | 83.9% |

> O L* introduz transições artificiais para garantir a completude do modelo. Esse ruído cresce com a complexidade do processo e inflaciona o denominador da cobertura bruta do AALpy.

### Cobertura Normalizada (comportamentos por evento do alfabeto)

| Dataset | AALpy/ev | Agilkia/ev | Razão |
|---------|----------|------------|-------|
| Scanner | 1.56 | 1.44 | 1.08× |
| BPI RfP | 3.88 | 1.19 | 3.26× |
| HDFS | 0.00 | 1.00 | 0.00× |
| BPI Loan | 2.41 | 1.00 | 2.41× |

> Em domínios estruturados, AALpy captura de 1.08× a 3.26× mais comportamentos por evento. No HDFS (domínio não estruturado), AALpy falha completamente enquanto Agilkia mantém cobertura.

### Mutation Score — Scanner (n=30 mutantes por operador)

| Operador | RPNI | GSM | L* (AALpy) | Agilkia |
|----------|------|-----|-----------|---------|
| output_fault | 0% | 0% | 23.3% | 100% |
| transfer_fault | 0% | 0% | 100.0% | 100% |
| remove_evento | 0% | 0% | 53.3% | 100% |
| **TOTAL** | **0%** | **0%** | **58.9%** | **100%** |

### Oráculo de Aichernig — Scanner

| Métrica | Oráculo Aleatório | Oráculo Aichernig |
|---------|------------------|-------------------|
| MQ — Equivalência | 2.000 | 575 |
| Redução de queries | — | **71%** |

---

## Principais Achados

1. **Ruído sistemático:** O L* cria transições artificiais que inflacionam o denominador da cobertura. Esse ruído varia de 61% (Scanner) a 100% (HDFS).

2. **Comparação justa:** A normalização por evento do alfabeto é o único denominador comum entre os dois modelos, permitindo comparação válida.

3. **Domínios estruturados:** AALpy captura mais comportamentos únicos por evento (razão 1.08× a 3.26×) graças ao contexto de estados.

4. **Domínios não estruturados:** Agilkia é mais robusto — no HDFS, AALpy falha completamente enquanto Agilkia captura 1.00 comportamento por evento.

5. **Aichernig:** Sequências distinguidoras geradas por mutação da MEF reduzem as queries de equivalência em 71% sem perda de qualidade do modelo.

---

## Referências

- Aichernig, B. K., & Tappler, M. (2017). *Learning from Failing: Mutation Testing in Active Automata Learning*.
- Aichernig, B. K., & Tappler, M. (2019). *Learning a Behavior Model of Hybrid Systems*. ICTSS.
- Utting, M., et al. (2020). *Identifying and Generating Missing Tests using Machine Learning on Execution Traces*. AITest.
- Muškardin, E., et al. (2022). *AALpy: an active automata learning library*. ISSE.
- Angluin, D. (1987). *Learning regular sets from queries and counterexamples*. Information and Computation.

---

## Autor

Pesquisa de Mestrado — IME/USP  
Orientadores: Ana C. V. de Melo e Lucas C. Cordeiro