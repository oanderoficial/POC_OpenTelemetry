## Planejamento de Infraestrutura

Provisionar uma VM (Linux) on-premises para a configuração do novo Proxy Zabbix.

## Instalação do Zabbix Proxy

**Sistema operacional recomendado:** Ubuntu 22.04

**Configuração do Proxy**
Editar o arquivo `/etc/zabbix/zabbix_proxy.conf` com os seguintes parâmetros:

| Parâmetro | Valor |
| --- | --- |
| Server | IP do seu Zabbix Server |
| Hostname | Nome único para esse Proxy (ex: Proxy-Cloud) |
| DBName | Nome do banco |
| DBUser | Usuário do banco |
| DBPassword | Senha do banco |
| LogFile | Caminho para o log |


## Passos: 

## 1. Instalar o Zabbix Proxy (Linux)

### Para Ubuntu/Debian:

```bash
sudo apt update
sudo apt install zabbix-proxy-mysql zabbix-sql-scripts zabbix-agent2
```

### Para RHEL/CentOS/Rocky:

```bash
sudo yum install zabbix-proxy-mysql zabbix-sql-scripts zabbix-agent2
```

##  2. Configurar o Zabbix Proxy

Editar o arquivo de configuração:
```bash
sudo nano /etc/zabbix/zabbix_proxy.conf
```
### Parâmetros principais:

```ini
Server=<IP_DO_ZABBIX_SERVER>
Hostname=Proxy-Cloud
DBName=zabbix_proxy
DBUser=zabbix
DBPassword=zabbix123
LogFile=/var/log/zabbix/zabbix_proxy.log
Timeout=30

```
Iniciar e habilitar o serviço:

```bash
sudo systemctl enable zabbix-proxy
sudo systemctl start zabbix-proxy
```

## 3. Liberar Firewall (Local e na Cloud)

### No firewall interno:
* Permitir saída do Proxy para IPs públicos das VMs na AWS/Azure na porta 10050/TCP
* Permitir saída do Proxy para o Zabbix Server na porta 10051/TCP

### Na cloud (Security Group / NSG):
* Permitir entrada na VM da Cloud (porta 10050/TCP) a partir do IP público do Proxy

## 4. Instalar o Zabbix Agent 2 nas VMs (AWS e Azure)

### Ubuntu/Debian:

```bash
sudo apt install zabbix-agent2
```
### RHEL/CentOS:

```bash
sudo yum install zabbix-agent2
```

### Editar configuração:

```bash
sudo nano /etc/zabbix/zabbix_agent2.conf
```

```ini
Server=<IP_DO_PROXY>
ServerActive=<IP_DO_PROXY>
Hostname=vm-cloud01
```
### Iniciar e habilitar:

```bash
sudo systemctl enable zabbix-agent2
sudo systemctl start zabbix-agent2
```

## 5. Criar o Proxy no Zabbix Server

1. Acesse o frontend do Zabbix Server
2. Vá em Administration → Proxies → Create proxy
3. Defina:
* Hostname: Proxy-Cloud
* Mode: Active (recomendado)
* Vincule esse proxy aos hosts da cloud

## 6. Instalar o OpenTelemetry Collector no Proxy

### Baixar binário oficial:

```bash
wget https://github.com/open-telemetry/opentelemetry-collector-releases/releases/download/v0.130.0/otelcol-contrib_0.130.0_linux_amd64.deb
sudo dpkg -i otelcol-contrib_0.130.0_linux_amd64.deb
```

### Caminhos padrão após instalação via .deb

| Item            | Caminho                            |
| --------------- | ---------------------------------- |
| Binário         | `/usr/bin/otelcol-contrib`         |
| Configuração    | `/etc/otelcol-contrib/config.yaml` |
| Serviço systemd | `otelcol-contrib.service`          |


## Exemplo básico de configuração (/etc/otelcol-contrib/config.yaml)

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

exporters:
  logging:

service:
  pipelines:
    metrics:
      receivers: [otlp]
      exporters: [logging]
```

## Criar serviço systemd (/etc/systemd/system/otel-collector.service):

```ini
[Unit]
Description=OpenTelemetry Collector
After=network.target

[Service]
ExecStart=/usr/local/bin/otelcol --config /etc/otel-collector/config.yaml
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### Iniciar o serviço:

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable otel-collector
sudo systemctl start otel-collector
```

## 7. Testar OpenTelemetry Collector

Use uma aplicação de teste ou ferramenta como otel-cli para enviar métricas.

```bash
otel-cli --endpoint http://localhost:4318 --name test --metrics
```
Verifique os logs com:

```bash
journalctl -u otel-collector -f
```


