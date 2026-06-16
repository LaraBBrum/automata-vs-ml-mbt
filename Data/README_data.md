# Dataset

O arquivo de log não está incluído neste repositório por questões de tamanho.

## Como obter

1. Acesse o repositório oficial do Agilkia:
   ```
   https://github.com/PHILAE-PROJECT/agilkia
   ```

2. Navegue até:
   ```
   examples/scanner/127.0.0.1-1571403244552.csv
   ```

3. Faça o download do arquivo e coloque-o na pasta `data/` ou faça upload diretamente no Google Colab ao executar o Passo 2 do notebook.

## Descrição do Dataset

| Propriedade | Valor |
|-------------|-------|
| Total de eventos | 65.273 |
| Total de sessões (clientes) | 4.818 |
| Período | Outubro de 2019 |
| Sistema | Scanner de supermercado |
| Formato | CSV |

## Colunas

| Coluna | Descrição |
|--------|-----------|
| `id` | Identificador do evento |
| `timestamp` | Timestamp da requisição (removido no pré-processamento) |
| `client` | Identificador da sessão do cliente |
| `scan` | Identificador do scan |
| `action` | Ação executada (scanner, debloquer, transmission, etc.) |
| `params` | Parâmetros da ação (código de barras, valor, etc.) |
| `status` | Código de retorno (0=ok, -2=erro, ?=void, decimal=valor) |

## Pré-processamento Aplicado

1. Remoção da coluna `timestamp`
2. Semântização dos parâmetros: códigos numéricos → `(produto)`, `(caixa)`, `(valor/qtd)`
3. Semântização do status: `0→ok`, `-2→erro`, `?→void`, decimais→`valor`
4. Geração de eventos compostos: `action + param_semântico` (ex: `scanner(produto)`)
5. Agrupamento por cliente → 4.818 traces de sessão
