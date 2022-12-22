# Como configurar vSphere with Tanzu con HAProxy

## Pre requisitos
- Cluster con DRS y HA activado.
- Tres subnets
- Switch virtual estandard o distribuido
- OVA HAProxy - [Link de descarga]([https://](https://github.com/haproxytech/vmware-haproxy#download))
- Storage compartido
- Content Library

## HAProxy
Antes de desplegar el OVA debemos definir ciertas configuraciones.
> En esta guía no voy a mostrar como desplegar el OVA, pero con las siguientes anotaciones vas a poder completarlo sin problemas.

### 1. Configuración
| Detalle       | Password      |
| -----------   | -----------   |
| Root Password | *password*    |

### 2. Configuración de Red
Registros DNS para la red de Management:
| Detalle       | Nombre            |
| -----------   | -----------       |
| FQDN          | haproxy.home.lab  |
| Server DNS    | 172.16.26.5       |

Dirección IP fija para cada servicio; Management, Workload, Frontend en formato CIDR:

| Detalle       | Port Group            | CIDR              | GATEWAY       |
| -----------   | -----------           | -----------       | -----------   |
| Mangement     | pg-tmgmt-vlan27       | 172.16.27.2/24    | 172.16.27.1   |
| Workload      | pg-tworkload-vlan28   | 172.16.28.2/24    | 172.16.28.1   |
| Frontend      | pg-tvip-vlan29        | 172.16.29.2/24    | 172.16.29.1   |

![Configuración de red - HAProxy](/assets/img/configurar-vsphere-with-tanzu-con-haproxy/haproxy-network-config.webp "Paso 2 - Configuración de Red").


### 3. Configuración de Balanceo de Carga

Rango de IPs destinadas al servicio de balanceo de carga (Frontend Network) en formato CIDR. **Este rango no puede contener la IP fija destinada a Frontend en el punto anterior.**

| Detalle       | CIDR                  |
| -----------   | -----------           |
| Frontend      | 172.16.29.16/28       |
*IPs disponibles desde 172.16.29.17 - 172.16.29.30*

Usuario y password que podrá conectarse a la API.

| Detalle       | Valor         |
| -----------   | -----------   |
| User          | admin         |
| Password      | *password*    |
| API Port      | 5556          |

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
