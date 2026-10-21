# BGP – FortiGate Rapid Troubleshooting

Guia **operacional e direto** para investigar falhas de BGP (**Border Gateway Protocol**) em FortiGate, utilizando saídas reais de um **laboratório controlado**. O foco é encontrar a causa, aplicar a correção e comprovar o resultado, sem excesso de teoria.

## Diagnóstico rápido

| Objetivo | Comando | O que verificar |
|---|---|---|
| Estado das sessões | `get router info bgp summary` | `State/PfxRcd`, AS remoto, prefixos, `Up/Down` |
| Detalhes do vizinho | `get router info bgp neighbors <IP>` | `BGP state`, `Last Reset`, `Notification Error Message` |
| Prefixos pelo vizinho | `get router info bgp neighbors <IP> routes` | Prefixos disponíveis e indicador `*` (válido), `>` (melhor) |
| Rotas BGP instaladas | `get router info routing-table bgp` | Prefixos, next-hop e interface/túnel |
| Configuração | `show router bgp` | AS local, `remote-as`, vizinhos e redes |

> Uma sessão **Established** não garante, por si só, que os prefixos estejam instalados na tabela de roteamento. Compare os anúncios recebidos com as rotas efetivamente instaladas.

## Casos práticos

| Caso | Sintoma | Causa | Situação |
|---|---|---|---|
| [01 — Neighbor Idle: Bad Peer AS](CENARIOS/01-bgp-bad-peer-as.md) | Vizinhança não estabelece | `remote-as` incorreto | **Concluído** — evidências coletadas |
| [02 — Neighbor Idle: IP incorreto](CENARIOS/02-bgp-neighbor-ip-incorreto.md) | Neighbor configurado com IP divergente do endpoint IPsec | `No such Peer configured` | **Concluído** — aguardando upload das imagens |

## Caso 02 em 30 segundos

**Sintoma:** Matriz com `2.2.2.99` em `Idle`, enquanto a VPN `CLARO_TO_RJ` permanecia ativa.

**Evidência:** `show system interface CLARO_TO_RJ` mostrava `set remote-ip 2.2.2.1`. O sniffer registrou TCP/179 vindo de `2.2.2.1` e encerrado com RST; o debug confirmou:

```text
BGP: [NETWORK] Accept Thread: 2.2.2.1 - No such Peer configured
```

**Correção:** restaurar o IP do neighbor para `2.2.2.1` e manter o AS remoto `65000`. **Resultado:** sessão estabelecida e **3 prefixos recebidos**.

➡️ [Ver investigação completa do Caso 02](CENARIOS/02-bgp-neighbor-ip-incorreto.md)  
📸 [Ver evidências dos cenários](EVIDENCIAS.md)

## Caso 01 em 30 segundos

**Sintoma:** vizinho `2.2.2.1` em `Idle`; vizinho `1.1.1.1` permaneceu estabelecido.

**Evidência:**
```text
[DECODE] Open: Bad Remote-AS (65000), expected 65099
%BGP-3-NOTIFICATION: sending to 2.2.2.1 2/2
(OPEN Message Error/Bad Peer AS.)
```

**Correção:** restaurar `remote-as 65000` no peer `2.2.2.1` da Matriz. **Resultado:** sessão restabelecida, com **3 prefixos recebidos**.

➡️ **[Ver troubleshooting completo do Caso 01](CENARIOS/01-bgp-bad-peer-as.md)**  
📸 **[Ver relação das 11 evidências](EVIDENCIAS.md)**

> **Ambiente de laboratório.** A configuração incorreta de AS foi provocada intencionalmente. Não replique alterações disruptivas em produção sem análise de impacto e janela aprovada. As imagens são adicionadas separadamente à pasta `evidencias/`.
