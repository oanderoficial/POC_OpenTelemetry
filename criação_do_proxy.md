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
