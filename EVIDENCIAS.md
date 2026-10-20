# Evidências — Caso 01: Bad Peer AS

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
