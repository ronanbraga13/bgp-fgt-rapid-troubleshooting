# Evidências — Caso 01: Bad Peer AS

Imagens coletadas em **20/10/2026**. Os nomes seguem a ordem de execução e devem ser enviados **sem renomear** para a pasta `evidencias/`.



| Nº | Arquivo | Comprovação |
|---:|---|---|
| 01 | `01-baseline-matriz-bgp-summary-config.png` | Sessões estabelecidas e configuração BGP na Matriz |
| 02 | `02-baseline-rj-bgp-summary-config.png` | Sessões estabelecidas e configuração BGP no RJ |
| 03 | `03-baseline-rj-rotas-bgp.png` | Rotas instaladas na tabela BGP do RJ |
| 04 | `04-baseline-matriz-rotas-bgp.png` | Rotas instaladas na tabela BGP da Matriz |
| 05 | `05-falha-bgp-idle-matriz.png` | Peer de teste em Idle na Matriz |
| 06 | `06-falha-bgp-idle-rj.png` | Peer correspondente em Idle no RJ |
| 07 | `07-diagnostico-neighbor-bad-peer-as.png` | Last Reset e Bad Peer AS no detalhe do neighbor |
| **08** | **`08-debug-bgp-bad-peer-as-destacado.png`** | **Debug com as duas mensagens de erro destacadas** |
| 09 | `09-correcao-remote-as-matriz.png` | AS remoto restaurado |
| 10 | `10-validacao-sessoes-bgp-established.png` | Ambos os peers recuperados com três prefixos |
| 11 | `11-validacao-prefixos-neighbor-2-2-2-1.png` | Prefixos recebidos pelo peer restaurado |

## Galeria cronológica

### 01 — Baseline Matriz
![Baseline Matriz](evidencias/01-baseline-matriz-bgp-summary-config.png)

### 02 — Baseline RJ
![Baseline RJ](evidencias/02-baseline-rj-bgp-summary-config.png)

### 03 — Rotas BGP RJ
![Tabela BGP RJ](evidencias/03-baseline-rj-rotas-bgp.png)

### 04 — Rotas BGP Matriz
![Tabela BGP Matriz](evidencias/04-baseline-matriz-rotas-bgp.png)

### 05 — Sessão em Idle na Matriz
![Neighbor Idle Matriz](evidencias/05-falha-bgp-idle-matriz.png)

### 06 — Sessão em Idle no RJ
![Neighbor Idle RJ](evidencias/06-falha-bgp-idle-rj.png)

### 07 — Erro no detalhe do vizinho
![BGP neighbor Bad Peer AS](evidencias/07-diagnostico-neighbor-bad-peer-as.png)

### 08 — Debug: erro destacado
![BGP debug Bad Peer AS marcado](evidencias/08-debug-bgp-bad-peer-as-destacado.png)

### 09 — Correção
![Restaurando remote-as](evidencias/09-correcao-remote-as-matriz.png)

### 10 — Sessões recuperadas
![Sessões BGP restabelecidas](evidencias/10-validacao-sessoes-bgp-established.png)

### 11 — Prefixos do neighbor recuperado
![Prefixos recebidos pelo neighbor](evidencias/11-validacao-prefixos-neighbor-2-2-2-1.png)

[← Voltar ao guia](README.md) · [Análise do Caso 01](CENARIOS/01-bgp-bad-peer-as.md)
