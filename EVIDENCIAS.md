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

---

# Caso 02 — Neighbor com IP incorreto

**Registro:** 21/10/2026 — arquivos PNG para envio na pasta `evidencias/lab02/`.

| Nº | Evidência | Descrição |
|---:|---|---|
| 00 | `00-baseline-bgp-matriz.png` | Sessões normais antes da simulação |
| 01 | `01-neighbor-idle-rotas-preservadas.png` | Neighbor incorreto em Idle e rotas pela VIVO |
| 02 | `02-vpn-ipsec-status-gui-matriz.png` | Estado das VPNs pela interface gráfica |
| 03 | `03-vpn-ipsec-tunnel-summary-matriz.png` | Seletores IPsec UP e contadores via CLI |
| 04 | `04-rota-default-mgmt-ping-sem-resposta.png` | Rota default via mgmt e ICMP sem resposta |
| **05** | **`05-interface-claro-ip-remoto-correto.png`** | **Ponto em que a divergência já podia ser identificada** |
| 06 | `06-sniffer-tcp179-rj-inicia-matriz-envia-rst.png` | Handshake TCP, dados e RST enviados pela Matriz |
| **07** | **`07-debug-bgp-no-such-peer-destacado.png`** | **Debug com `No such Peer configured` destacado** |
| 08 | `08-correcao-neighbor-gui-ip-as.png` | Configuração corrigida na GUI |
| **09** | **`09-validacao-bgp-established-tres-prefixos-destacado.png`** | **Sessão recuperada e três prefixos destacados** |

### 00 — Baseline
![Baseline BGP](evidencias/lab02/00-baseline-bgp-matriz.png)

### 01 — Neighbor em Idle
![Neighbor Idle e rotas](evidencias/lab02/01-neighbor-idle-rotas-preservadas.png)

### 02 — VPNs na GUI
![Status IPsec GUI](evidencias/lab02/02-vpn-ipsec-status-gui-matriz.png)

### 03 — VPNs pela CLI
![Status IPsec CLI](evidencias/lab02/03-vpn-ipsec-tunnel-summary-matriz.png)

### 04 — Roteamento e conectividade
![Roteamento e ping](evidencias/lab02/04-rota-default-mgmt-ping-sem-resposta.png)

### 05 — Identificação da divergência na VPN
![IP remoto da VPN](evidencias/lab02/05-interface-claro-ip-remoto-correto.png)

### 06 — Sniffer TCP/179
![Sniffer TCP](evidencias/lab02/06-sniffer-tcp179-rj-inicia-matriz-envia-rst.png)

### 07 — Debug BGP destacado
![No such peer configured](evidencias/lab02/07-debug-bgp-no-such-peer-destacado.png)

### 08 — Correção
![Neighbor correto](evidencias/lab02/08-correcao-neighbor-gui-ip-as.png)

### 09 — Resultado final
![Resumo e rotas após correção](evidencias/lab02/09-validacao-bgp-established-tres-prefixos-destacado.png)

[← Voltar ao índice](README.md) · [Análise do Caso 02](CENARIOS/02-bgp-neighbor-ip-incorreto.md)
