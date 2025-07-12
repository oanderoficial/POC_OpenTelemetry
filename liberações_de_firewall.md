## Liberações no Firewall Interno (On-Premises)

Garantir que o Zabbix Proxy tenha permissão de saída para a internet (AWS/Azure) de forma controlada.

| Origem | Destino | Porta | Protocolo | Motivo |
| --- | --- | --- | --- | --- |
| Zabbix Proxy | VMs na AWS | 10050/TCP | TCP | Coletar dados via Agent2 |
| Zabbix Proxy | VMs na Azure | 10050/TCP | TCP | Coletar dados via Agent2 |
| Zabbix Proxy | Internet geral | 443/TCP | HTTPS | Uso de APIs ou coleta OTLP remota |
| Zabbix Proxy | Zabbix Server | 10051/TCP | TCP | Envio de dados para o Zabbix Server |
| OpenTelemetry | Zabbix Proxy | 4317,4318/TCP | TCP | Receber dados OTLP (coleta externa via OTel SDK) |

Sugestão: criar uma regra de saída controlada, permitindo apenas os IPs públicos das clouds.

---

## Liberação de Firewall / Security Group

| Origem | Destino | Porta | Direção |
| --- | --- | --- | --- |
| Proxy → VMs AWS | 10050/TCP | Outbound |  |
| Proxy → VMs Azure | 10050/TCP | Outbound |  |
| Proxy → Zabbix Server | 10051/TCP | Outbound |  |
| VMs Cloud → Proxy (modo ativo) | 10051/TCP | Inbound |  |
| OpenTelemetry SDK → Proxy | 4317,4318/TCP | Inbound |  |
