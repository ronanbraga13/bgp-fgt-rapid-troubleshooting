# Cenário 01 — BGP não estabelece | Bad Peer AS

**Status:** concluído (20/10/2026)  
**Tipo:** falha simulada de configuração de vizinhança eBGP (**External BGP**)

## 1. Sintoma e escopo

A Matriz perdeu a sessão com o vizinho `2.2.2.1`, enquanto `1.1.1.1` permaneceu estabelecido. O RJ também apresentou a vizinhança correspondente `2.2.2.2` em `Idle`.

| Equipamento | AS local | Peer de teste | AS remoto correto |
|---|---:|---|---:|
| FW_MATRIZ | 65001 | 2.2.2.1 | 65000 |
| FGT_RIODEJANEIRO | 65000 | 2.2.2.2 | 65001 |

No baseline, **ambas as sessões estavam estabelecidas** e recebiam **3 prefixos por neighbor**. Os prefixos `10.10.10.0/24`, `10.10.20.0/24` e `10.10.30.0/24` apareciam instalados pela **VIVO**, no caminho via `1.1.1.1` na Matriz e via `1.1.1.2` no RJ.

![Baseline do BGP na Matriz](../evidencias/01-baseline-matriz-bgp-summary-config.png)

## 2. Falha reproduzida

Alteração intencional no **FW_MATRIZ** (somente lab):

```bash
config router bgp
    config neighbor
        edit "2.2.2.1"
            set remote-as 65099
        next
    end
end
```

O AS real do equipamento remoto permaneceu **65000**; a Matriz passou a esperar **65099**.

Após a mudança:

```text
MATRIZ
1.1.1.1   AS 65000   State/PfxRcd: 3
2.2.2.1   AS 65099   State: Idle

RJ
1.1.1.2   AS 65001   State/PfxRcd: 3
2.2.2.2   AS 65001   State: Idle
```

![Falha reproduzida na Matriz](../evidencias/05-falha-bgp-idle-matriz.png)

## 3. Diagnóstico rápido

### Passo A — Identificar a sessão com falha

```bash
get router info bgp summary
```

**Interpretação:** `Idle` aponta que a sessão BGP não está estabelecida; **não identifica sozinho a causa**.

### Passo B — Consultar o histórico do vizinho

```bash
get router info bgp neighbors 2.2.2.1
```

Trecho relevante:

```text
BGP neighbor is 2.2.2.1, remote AS 65099, local AS 65001
BGP state = Idle
Connections established 1, dropped 1
Last Reset: 00:00:03, due to BGP Notification sent
Notification Error Message: (OPEN Message Error/Bad Peer AS.)
```

**Interpretação:** a rejeição ocorreu durante a negociação da mensagem **OPEN**. `Connections established 1` refere-se à conexão TCP, não ao estado BGP `Established`.

![Detalhes do neighbor indicando Bad Peer AS](../evidencias/07-diagnostico-neighbor-bad-peer-as.png)

### Passo C — Confirmar no debug

Comandos utilizados na **Matriz** para acompanhar a negociação do peer de teste:

```bash
diagnose debug reset
diagnose ip router bgp all enable
diagnose ip router bgp level info
diagnose ip router bgp set-filter neighbor 2.2.2.1
diagnose debug console timestamp enable
diagnose debug enable
```

**Linhas decisivas:**

```text
[DECODE] Open: Bad Remote-AS (65000), expected 65099
%BGP-3-NOTIFICATION: sending to 2.2.2.1 2/2
(OPEN Message Error/Bad Peer AS.)
```

O código de notificação **2/2** significa **OPEN Message Error / Bad Peer AS**. A sessão não prosseguiu até `Established`.

![Debug com os dois registros críticos destacados](../evidencias/08-debug-bgp-bad-peer-as-destacado.png)

**Encerrar debug após a coleta:**

```bash
diagnose debug disable
diagnose debug reset
```

> Use debug em janelas curtas e, de preferência, filtrado por vizinho. A sintaxe e disponibilidade de filtros variam conforme o FortiOS.

## 4. Causa identificada

**`remote-as 65099` configurado na Matriz, divergente do AS real `65000` do vizinho.** A mensagem OPEN foi rejeitada e o FortiGate enviou uma **NOTIFICATION**.

## 5. Correção aplicada

```bash
config router bgp
    config neighbor
        edit "2.2.2.1"
            set remote-as 65000
        next
    end
end
```

![Correção do remote-as na Matriz](../evidencias/09-correcao-remote-as-matriz.png)

## 6. Validação e resultado

```bash
get router info bgp summary
get router info bgp neighbors 2.2.2.1 routes
get router info routing-table bgp
```

**Validação observada:**

```text
Neighbor    AS      Up/Down     State/PfxRcd
1.1.1.1     65000   00:12:02    3
2.2.2.1     65000   00:00:39    3
```

![Sessões restabelecidas](../evidencias/10-validacao-sessoes-bgp-established.png)

O comando de rotas por vizinho mostrou os três prefixos aprendidos por `2.2.2.1`, todos com o marcador `*` (**válidos**), mas **sem `>`**, portanto não selecionados como melhor caminho naquela saída:

```text
* 10.10.10.0/24  2.2.2.1  ... 65000 i
* 10.10.20.0/24  2.2.2.1  ... 65000 i
* 10.10.30.0/24  2.2.2.1  ... 65000 i
Total number of prefixes 3
```

![Três prefixos disponíveis pelo vizinho recuperado](../evidencias/11-validacao-prefixos-neighbor-2-2-2-1.png)

**Resultado final:** sessão BGP recuperada com **3 prefixos recebidos**, mantendo a outra vizinhança operacional durante o teste. **Não foi executado teste de tráfego fim a fim nesta etapa**, portanto a comprovação se limita ao estado das sessões e aos prefixos exibidos.

## 7. Lembrete para incidentes

```text
Neighbor Idle
  → get router info bgp summary
  → get router info bgp neighbors <IP>
  → procurar Last Reset / Notification
  → se necessário, debug BGP filtrado por neighbor
  → comparar local AS / remote-as com configuração real do peer
  → corrigir divergência
  → validar Established + prefixos + tabela de rotas
```

📸 [Todas as evidências, em ordem cronológica](../EVIDENCIAS.md).
