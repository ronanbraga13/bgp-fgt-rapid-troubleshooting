# Cenário 02 — BGP Neighbor Idle | Endereço IP incorreto

**Data:** 21/10/2026  
**Ambiente:** FortiGate Matriz ↔ RJ, sessões eBGP (External Border Gateway Protocol) sobre VPN IPsec  
**Resultado:** **resolvido** — sessão restabelecida, 3 prefixos recebidos

## Sintoma

A Matriz apresentava o neighbor **2.2.2.99** em `Idle`, com `MsgRcvd=0`, `MsgSent=0` e sem prefixos recebidos. O peer **1.1.1.1** continuava estabelecido; as redes `10.10.10.0/24`, `10.10.20.0/24` e `10.10.30.0/24` permaneciam instaladas via `VIVO_TO_RJ`.

No baseline, os dois neighbors eram `1.1.1.1` e `2.2.2.1`, ambos no AS remoto **65000**, com **3 prefixos recebidos por neighbor**.

![Baseline BGP](../evidencias/lab02/00-baseline-bgp-matriz.png)

> **Falha simulada:** no FortiGate Matriz, a vizinhança `2.2.2.1` foi removida e substituída por `2.2.2.99`, mantendo o `remote-as 65000`. Alteração restrita ao lab; preservado o segundo caminho BGP pela VIVO.

## Investigação operacional

### 1. Conferir vizinhanças e rotas

```bash
get router info bgp summary
get router info routing-table bgp
```

**Encontrado:** `2.2.2.99` em `Idle`; ausência de troca de mensagens; três rotas BGP continuavam instaladas via `1.1.1.1 / VIVO_TO_RJ`.

![Neighbor Idle e rotas preservadas](../evidencias/lab02/01-neighbor-idle-rotas-preservadas.png)

### 2. Validar VPN IPsec

```bash
get vpn ipsec tunnel summary
```

As VPNs **CLARO_TO_RJ** e **VIVO_TO_RJ** apareciam com seletores `1/1` ativos. Havia contadores de tráfego. Isso **não comprova conectividade BGP**, mas afasta a hipótese de ambos os túneis estarem simplesmente desativados.

![Status IPsec na GUI](../evidencias/lab02/02-vpn-ipsec-status-gui-matriz.png)

![Resumo IPsec via CLI](../evidencias/lab02/03-vpn-ipsec-tunnel-summary-matriz.png)

### 3. Consultar rota e testar o IP configurado no neighbor

```bash
get router info routing-table details 2.2.2.99
execute ping-options source 2.2.2.2
execute ping 2.2.2.99
```

A consulta ao destino `2.2.2.99` encontrou apenas a **rota padrão `0.0.0.0/0` via `192.168.239.2`, interface `mgmt`**. O ping retornou **5 enviados, 0 recebidos, 100% de perda**.

**Interpretação:** não havia rota mais específica para o neighbor configurado. A ausência de resposta ICMP, isoladamente, **não prova** IP incorreto nem bloqueio de TCP/179.

![Rota default via mgmt e ping sem resposta](../evidencias/lab02/04-rota-default-mgmt-ping-sem-resposta.png)

### 4. Conferir o endereçamento da VPN — identificação antecipada da causa

```bash
show system interface CLARO_TO_RJ
```

Trecho relevante:

```text
set ip 2.2.2.2 255.255.255.255
set remote-ip 2.2.2.1 255.255.255.252
```

A interface do túnel aponta para o endpoint remoto **2.2.2.1**, mas o BGP estava configurado com **2.2.2.99**.

**⚠️ Ponto de atenção:** **a divergência já podia ser identificada nesta etapa**. Não era necessário sniffer ou debug para perceber a inconsistência entre o endereço do túnel e a vizinhança configurada. Os passos seguintes foram realizados **para confirmar o comportamento do TCP e o motivo da rejeição pelo processo BGP**, e não para descobrir novamente a divergência.

![Interface IPsec: endereço remoto correto](../evidencias/lab02/05-interface-claro-ip-remoto-correto.png)

### 5. Capturar TCP/179

```bash
diagnose sniffer packet any 'tcp port 179 and (host 2.2.2.99 or host 2.2.2.1)' 4 0 l
```

Exemplo das linhas observadas na Matriz:

```text
CLARO_TO_RJ in  2.2.2.1.2472 -> 2.2.2.2.179: syn
CLARO_TO_RJ out 2.2.2.2.179  -> 2.2.2.1.2472: syn ack
CLARO_TO_RJ in  2.2.2.1.2472 -> 2.2.2.2.179: ack
CLARO_TO_RJ in  2.2.2.1.2472 -> 2.2.2.2.179: psh
CLARO_TO_RJ out 2.2.2.2.179  -> 2.2.2.1.2472: rst
```

**Interpretação:** a Matriz **recebia** tentativas do RJ originadas em `2.2.2.1`; o handshake TCP era concluído, mas a Matriz encerrava a conexão com **RST**. A captura **não mostrou** SYN destinado a `2.2.2.99` durante a janela observada.

![Sniffer TCP/179: recebimento e RST](../evidencias/lab02/06-sniffer-tcp179-rj-inicia-matriz-envia-rst.png)

### 6. Confirmar pelo debug BGP

```bash
diagnose debug reset
diagnose ip router bgp all enable
diagnose ip router bgp level info
diagnose debug console timestamp enable
diagnose debug enable
```

**Evidência decisiva:**

```text
BGP: [NETWORK] Accept Thread: Incoming conn from host 2.2.2.1 (FD=28 VRF=0)
BGP: [NETWORK] Accept Thread: 2.2.2.1 - No such Peer configured
```

O erro foi registrado novamente em outra tentativa. **Causa confirmada:** a Matriz recebia uma conexão BGP do endereço real `2.2.2.1`, mas não possuía esse IP configurado como neighbor.

![Debug destacado: No such Peer configured](../evidencias/lab02/07-debug-bgp-no-such-peer-destacado.png)

Ao concluir a coleta, desative o debug:

```bash
diagnose debug disable
diagnose debug reset
execute ping-options reset
```

## Correção aplicada

O neighbor foi restaurado pela **interface gráfica** (GUI) para o IP **2.2.2.1**, com AS remoto **65000**, conforme captura. Em ambientes que utilizam autenticação BGP, preservar também a configuração de autenticação correspondente ao peer.

![GUI: neighbor 2.2.2.1 e AS 65000](../evidencias/lab02/08-correcao-neighbor-gui-ip-as.png)

## Validação final

```bash
get router info bgp summary
get router info bgp neighbors 2.2.2.1 routes
```

O comando `summary` apresentou os dois peers estabelecidos e **3 prefixos por neighbor**:

```text
Neighbor  AS     Up/Down    State/PfxRcd
1.1.1.1   65000  00:36:34   3
2.2.2.1   65000  00:01:27   3
```

O vizinho recuperado disponibilizou:

```text
* 10.10.10.0/24 via 2.2.2.1 ... 65000 i
* 10.10.20.0/24 via 2.2.2.1 ... 65000 i
* 10.10.30.0/24 via 2.2.2.1 ... 65000 i
Total number of prefixes 3
```

O marcador `*` indica **rota BGP válida**, mas a ausência de `>` nessa saída indica que esses caminhos **não eram os melhores selecionados** naquele instante. Não foi documentado teste de tráfego ICMP após a correção; a validação final comprova o restabelecimento da sessão e o recebimento dos três prefixos.

![Validação final destacada](../evidencias/lab02/09-validacao-bgp-established-tres-prefixos-destacado.png)

## Resumo em uma linha

**BGP Idle → IPsec UP → consulta de rota caiu na default/mgmt → ping sem resposta → `remote-ip` da VPN divergente do neighbor → TCP/179 do peer real chega e recebe RST → debug: `No such Peer configured` → corrigido IP do neighbor → sessão Established e 3 prefixos recebidos.**

📸 [Todas as evidências deste caso](../EVIDENCIAS.md#caso-02--neighbor-com-ip-incorreto)
