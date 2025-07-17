# Instalação do Agente OpenTelemetry nas Aplicações

Este guia descreve como instalar e configurar o agente OpenTelemetry nas aplicações hospedadas na AWS e Azure, para envio de métricas ao OpenTelemetry Collector instalado no Zabbix Proxy (on-premises).

## 1. Pré-requisitos

- Aplicações devem rodar em ambientes que suportem injeção de agente (ex: Java, .NET, Python, Node.js)
- A VM deve permitir saída para a porta **4317 (gRPC)** ou **4318 (HTTP)** do OpenTelemetry Collector on-prem

##  2. Java - Auto Instrumentação com o Agente OTEL

### 2.1. Baixe o agente

```bash
wget https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v2.17.1/opentelemetry-javaagent.jar
```

### 2.3 Rodar a aplicação com o agente ativado

```bash
java -javaagent:/opt/otel/opentelemetry-javaagent.jar \
     -Dotel.service.name=nome-da-aplicacao \
     -Dotel.exporter.otlp.endpoint=http://<IP_DO_PROXY>:4317 \
     -jar sua-aplicacao.jar
```

Altere:
 * nome-da-aplicacao para um nome único da aplicação
 * <IP_DO_PROXY> para o IP ou hostname do Proxy onde está o Collector

## 3. .NET - Auto Instrumentação

```bash
dotnet add package OpenTelemetry.Instrumentation.AspNetCore
dotnet add package OpenTelemetry.Exporter.OpenTelemetryProtocol
```

Configure no Program.cs:

```csharp
services.AddOpenTelemetry()
    .WithMetrics(builder => builder
        .AddAspNetCoreInstrumentation()
        .AddOtlpExporter(opt => {
            opt.Endpoint = new Uri("http://<IP_DO_PROXY>:4317");
        }));

```

## 4. Python - Exportador OTEL

```bash
pip install opentelemetry-api \
            opentelemetry-sdk \
            opentelemetry-exporter-otlp \
            opentelemetry-instrumentation

opentelemetry-instrument \
    --traces_exporter=otlp \
    --metrics_exporter=otlp \
    --exporter_otlp_endpoint=http://<IP_DO_PROXY>:4317 \
    python app.py
```

##  5. Node.js - Auto Instrumentação

```bash

npm install @opentelemetry/api \
             @opentelemetry/sdk-node \
             @opentelemetry/exporter-trace-otlp-grpc
```

```js
// otel.js
const { NodeSDK } = require('@opentelemetry/sdk-node');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-grpc');

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter({
    url: 'http://<IP_DO_PROXY>:4317',
  }),
  serviceName: 'nome-da-aplicacao',
});

sdk.start();
```

E inicie a aplicação com:

``` bash
node -r ./otel.js app.js
```

## Checklist pós-instalação

 * Liberação de firewall da aplicação para o IP do Collector (porta 4317 ou 4318)
 * Métricas aparecendo no log ou integradas no backend do Zabbix (via zabbix_sender, Prometheus scrape ou outra bridge)
 * Identificação correta por otel.service.name
