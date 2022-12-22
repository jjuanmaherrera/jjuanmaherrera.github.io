La siguiente guía sirve de referencia para poder desplegar **vSphere with Tanzu** en un ambiente productivo. HAProxy es un Balanceador de carga soportado por VMware, y en esta arquitectura, será desplegado con tres NICs: Management, Workload y Frontend para separar el tráfico.
A grandes rasgos los pasos a seguir son los siguientes:
- Completar el direccionamiento IP.
- Desplegar HAProxy
- Habilitar vSphere with Tanzu

## Pre requisitos
- Cluster con DRS y HA activado
- Switch Virtual estandard o distribuido
- Tres subnets configuradas con sus respectivas VLAN en el Switch Virtual
- OVA HAProxy - [Link de descarga]([https://](https://github.com/haproxytech/vmware-haproxy#download))
- Storage compartido - *No puede ser local*
- Content Library

## HAProxy
Antes de desplegar el OVA debemos definir ciertas configuraciones.
> En esta guía no voy a mostrar como desplegar el OVA, pero con las siguientes anotaciones vas a poder completarlo sin problemas.

### 1. Configuración del Appliance

| Parámetro | Valor | Descripción |
|---|---|---|
| Root Password | *root-password* | Password inicial del usuario root. |

### 2. Configuración de Red

| Parámetro | Valor | Descripción |
|---|---|---|
| Host Name | haproxy.home.lab  | El nombre FQDN a asignar a la VM de HAProxy  |
| DNS       | 172.16.26.5       | Lista de servidores DNS separados por coma. **Usar dos servidores DNS como mínimo.**  |
| IP Management | 172.16.27.2/24    | La dirección IP en formato CIDR para la red de Management. |
| Gateway Management    | 172.16.27.1 | Gatewat de la red de Management |
| IP Workload   | 172.16.28.2/24 | La dirección IP en formato CIDR para la red de Workload |
| Gateway Workload  | 172.16.28.1 |  Gatewat de la red de Workload  |
| IP Frontend  | 172.16.29.2/24 |  La dirección IP en formato CIDR para la red de Frontend. |
| Gateway Frontend  | 172.16.29.1 |  Gatewat de la red de Workload  |

### 3. Configuración de Balanceo de Carga

| Parámetro | Valor | Descripción |
|---|---|---|
| Rango de IP del Balanceador de carga | 172.16.29.16/28 | Rango de direcciones IP en formato CIDR válido. **El rango seleccionado no debe solaparse con la IP configurada en el paso anterior para la red de Frontend (en mi caso 172.16.29.2/24)** |
| Puerto de API | 5556 (Default) | El puerto en el que la API de balanceo de carga escucha |
| Usuario ID HAProxy | admin | El usuario para autenticarse a la API del Balanceador de carga. |
| Password HAProxy | *password*    | La password para autenticarse a la API del Balanceador de carga. | 

*IPs disponibles para la subnet 172.16.29.16/28 es desde 172.16.29.17 - 172.16.29.30*

## Habilitar vSphere with Tanzu
### Storage Policy
https://www.youtube.com/watch?v=TUB4rgSX528&list=PLymLY4xJSThplrNcSTTaa9iFU016J7gaI&index=2

### Content Library
https://www.youtube.com/watch?v=10QviDkW8ds&list=PLymLY4xJSThplrNcSTTaa9iFU016J7gaI&index=3

### Balanceo de Carga
![Configuración de Balanceo de Carga - HAProxy](/assets/img/configurar-vsphere-with-tanzu-con-haproxy/vsphere-with-tanzu-load-balancer.webp "Configurar Load Balancer en vSphere with Tanzu").

Certificado
https://github.com/vsphere-tmm/QSG-Setup-Deploy

#### Management

| Port Group        | IP Inicial    | Máscara       |   Gateway     |
| -----------       | -----------   | -----------   | -----------   | 
| pg-tmgmt-vlan27   | 172.16.27.10  | 255.255.255.0 | 172.16.27.1   |

La **Dirección IP inicial** debe tener cinco IPs consecutivas reservadas que no sean utilizadas.

#### Workload

| Port Group        | IP Inicial                    | Máscara       |   Gateway     |
| -----------       | -----------                   | -----------   | -----------   | 
| pg-tmgmt-vlan28   | 172.16.28.17 - 172.16.28.30   | 255.255.255.0 | 172.16.27.1   |
