# Pipeline Positivador

## Visão Geral

O Positivador é um arquivo coletado na XP com dados de clientes, PL e captação. O pipeline processa esse arquivo seguindo o padrão Bronze → Silver → Gold → Redshift.

---

## Jobs

### `positivador`
Processa arquivos que chegaram diretamente na Bronze.

**Fases:**
1. **Bronze → Silver:** identifica arquivos novos ou modificados e transforma
2. **Silver → Gold:** idem, promove para Gold
3. **Gold → Redshift:** upsert e validação
4. Envia e-mail de resultado ao final

> `report_evaluation` está comentado no job — ver [relatorio.md](relatorio.md).

## Sensores

| Sensor | Intervalo mínimo | Monitora | Dispara |
|---|---|---|---|
| `positivador_sensor` | 90 min | Bronze vs Silver | `positivador` |

---

## Fluxo completo

```
Hub da XP
    │
    ├─ Performance coleta o arquivo
    │
    ▼
SmartCheck
    │
    ▼
 [Bronze]
    │
    ├─ Passa na validação de outlier? ──Não──► [verificacao/positivador/]
    │                                               │
    │                                    Pedro aprova via e-mail
    │                                               │
    │                                    Mover para [verificados/positivador/]
    │                                               │
    │                                    positivador_verificados_sensor apita
    │                                               │
    │◄──────────────────────────────────────────────┘
    │
    ▼
[Silver]
    │
    ▼
[Gold]
    │
    ▼
[Redshift]
```

Ver detalhes da validação de outlier em [outlier.md](outlier.md).

> **Atenção:** a validação de outlier foi **removida temporariamente** do job `positivador`. Arquivos chegam direto do Bronze ao Silver sem passar pela checagem IQR nem pelo fluxo de verificação manual. Ver [outlier.md](outlier.md) e dívida técnica associada.
