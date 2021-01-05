---
title: Limiter le trafic de sortie dans Azure Kubernetes Service (AKS)
description: Découvrez les ports et adresses requis pour contrôler le trafic de sortie dans Azure Kubernetes Service (ACS)
services: container-service
ms.topic: article
ms.author: jpalma
ms.date: 11/09/2020
author: palma21
ms.openlocfilehash: a1d045e66771026d2b4cf7ad44fd6943d2d407f4
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94701600"
---
# <a name="control-egress-traffic-for-cluster-nodes-in-azure-kubernetes-service-aks"></a>Contrôler le trafic de sortie pour les nœuds de cluster dans Azure Kubernetes Service (AKS)

Cet article donne les informations nécessaires pour sécuriser le trafic sortant d’Azure Kubernetes Service (AKS). Il comporte les exigences en matière de cluster d’un déploiement AKS de base, ainsi des exigences supplémentaires concernant les extensions et les fonctionnalités facultatives. [Un exemple de configuration de ces exigences avec le Pare-feu Azure est fourni à la fin](#restrict-egress-traffic-using-azure-firewall). Toutefois, vous pouvez appliquer ces informations à n’importe quelle méthode ou appliance de restriction sortante.

## <a name="background"></a>Arrière-plan

Les clusters AKS sont déployés sur un réseau virtuel. Ce réseau peut être géré (créé par AKS) ou personnalisé (préconfiguré au préalable par l’utilisateur). Dans les deux cas, le cluster comporte des dépendances **sortantes** vis-à-vis de services situés en dehors de ce réseau virtuel (le service ne présente aucune dépendance entrante).

Pour la gestion et à des fins opérationnelles, les nœuds d’un cluster AKS doivent accéder à certains ports et noms de domaine complet (FQDN). Ces points de terminaison sont nécessaires pour permettre aux nœuds de communiquer avec le serveur d’API, ou de télécharger et d’installer les principaux composants du cluster Kubernetes et les correctifs de sécurité des nœuds. Par exemple, le cluster doit extraire les images conteneurs système de base de Microsoft Container Registry (MCR).

Les dépendances sortantes AKS sont presque entièrement définies avec des noms FQDN, qui n’ont pas d’adresses statiques derrière eux. En raison de cette absence d’adresses statiques, il n’est pas possible d’utiliser des groupes de sécurité réseau pour verrouiller le trafic sortant d’un cluster AKS. 

Par défaut, les clusters AKS ont un accès illimité sortant à Internet. Ce niveau d’accès réseau permet aux nœuds et services que vous exécutez d’accéder aux ressources externes en fonction des besoins. Si vous souhaitez restreindre le trafic de sortie, un nombre limité de ports et adresses doit être accessible afin de gérer les tâches de maintenance d’intégrité du cluster. La solution la plus simple pour sécuriser les adresses sortantes consiste à utiliser un dispositif de pare-feu permettant de contrôler le trafic sortant en fonction des noms de domaine. Le Pare-feu Azure, par exemple, peut restreindre le trafic HTTP et HTTPS sortant en fonction du nom FQDN de la destination. Vous pouvez également configurer les règles de pare-feu et de sécurité de votre choix pour autoriser ces ports et adresses requis.

> [!IMPORTANT]
> Ce document explique uniquement comment verrouiller le trafic sortant du sous-réseau AKS. AKS ne présente par défaut aucune exigence d’entrée.  Le blocage du **trafic de sous-réseau interne** à l’aide de groupes de sécurité réseau (NSG) et de pare-feu n’est pas pris en charge. Pour contrôler et bloquer le trafic au sein du cluster, utilisez des [**_Stratégies réseau_* _][network-policy].

## <a name="required-outbound-network-rules-and-fqdns-for-aks-clusters"></a>Règles de réseau sortantes et noms FQDN requis pour les clusters AKS

Les règles de réseau et de nom FQDN/d’application suivantes sont requises pour un cluster AKS. Vous pouvez les utiliser si vous souhaitez configurer une solution autre que le Pare-feu Azure.

_ Les dépendances d’adresses IP sont destinées au trafic non HTTP/S (à la fois le trafic TCP et UDP)
* Les points de terminaison HTTP/HTTPS avec des noms FQDN peuvent être placés dans votre dispositif de pare-feu.
* Les points de terminaison HTTP/HTTPS avec caractères génériques constituent des dépendances qui peuvent varier avec votre cluster AKS selon le nombre de qualificateurs.
* AKS utilise un contrôleur d’admission pour injecter le nom FQDN comme variable d’environnement pour tous les déploiements sous kube-system et gatekeeper-system. Ainsi, toutes les communications système entre les nœuds et le serveur d’API ont recours au nom FQDN du serveur d’API et non à son adresse IP. 
* Si l’une de vos applications ou solutions doit communiquer avec le serveur d’API, vous devez ajouter une règle de réseau **supplémentaire** pour autoriser la *communication TCP sur le port 443 de l’adresse IP du serveur d’API*.
* Il peut arriver en de rares occasions que l’adresse IP de votre serveur d’API change en cas d’opération de maintenance. Les opérations de maintenance planifiée susceptibles de modifier l’adresse IP du serveur d’API sont toujours communiquées à l’avance.


### <a name="azure-global-required-network-rules"></a>Règles de réseau requises pour Azure Global

Les règles de réseau et les dépendances d’adresse IP requises sont les suivantes :

| Point de terminaison de destination                                                             | Protocol | Port    | Utilisation  |
|----------------------------------------------------------------------------------|----------|---------|------|
| **`*:1194`** <br/> *Ou* <br/> [Balise de service](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.<Region>:1194`** <br/> *Ou* <br/> [CIDR régionaux](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:1194`** <br/> *Ou* <br/> **`APIServerPublicIP:1194`** `(only known after cluster creation)`  | UDP           | 1194      | Pour les communications sécurisées par tunnel entre les nœuds et le plan de contrôle. Ce n’est pas obligatoire pour les [clusters privés](private-clusters.md).|
| **`*:9000`** <br/> *Ou* <br/> [Balise de service](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.<Region>:9000`** <br/> *Ou* <br/> [CIDR régionaux](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:9000`** <br/> *Ou* <br/> **`APIServerPublicIP:9000`** `(only known after cluster creation)`  | TCP           | 9000      | Pour les communications sécurisées par tunnel entre les nœuds et le plan de contrôle. Ce n’est pas obligatoire pour les [clusters privés](private-clusters.md). |
| **`*:123`** ou **`ntp.ubuntu.com:123`** (en cas d’utilisation de règles de réseau de Pare-feu Azure)  | UDP      | 123     | Obligatoire pour la synchronisation date/heure NTP (Network Time Protocol) sur les nœuds Linux.                 |
| **`CustomDNSIP:53`** `(if using custom DNS servers)`                             | UDP      | 53      | Si vous utilisez des serveurs DNS personnalisés, vous devez vérifier qu’ils sont accessibles par les nœuds de cluster. |
| **`APIServerPublicIP:443`** `(if running pods/deployments that access the API Server)` | TCP      | 443     | Obligatoire en cas d’exécution de pods/déploiements qui accèdent au serveur d’API. Ces pods/déploiements utiliseront l’adresse IP de l’API. Ce n’est pas obligatoire pour les [clusters privés](private-clusters.md).  |

### <a name="azure-global-required-fqdn--application-rules"></a>Règles de nom FQDN/d’application requises pour Azure Global 

Les noms de domaine complets/règles d’application suivantes sont requis :

| Nom FQDN de destination                 | Port            | Utilisation      |
|----------------------------------|-----------------|----------|
| **`*.hcp.<location>.azmk8s.io`** | **`HTTPS:443`** | Obligatoire pour la communication Nœud <-> Serveur d’API. Remplacez *\<location\>* par la région où votre cluster AKS est déployé. |
| **`mcr.microsoft.com`**          | **`HTTPS:443`** | Obligatoire pour l’accès aux images de Microsoft Container Registry (MCR). Ce registre contient des images/graphiques internes (par exemple, CoreDNS), qui sont nécessaires à la création et au bon fonctionnement du cluster, y compris les opérations de scaling et de mise à niveau.  |
| **`*.data.mcr.microsoft.com`**   | **`HTTPS:443`** | Obligatoire pour le stockage MCR assuré par Azure Content Delivery Network (CDN). |
| **`management.azure.com`**       | **`HTTPS:443`** | Obligatoire pour les opérations Kubernetes sur l’API Azure. |
| **`login.microsoftonline.com`**  | **`HTTPS:443`** | Obligatoire pour l’authentification Azure Active Directory. |
| **`packages.microsoft.com`**     | **`HTTPS:443`** | Cette adresse est le référentiel de packages Microsoft utilisé pour les opérations *apt-get* mises en cache.  Moby, PowerShell et Azure CLI sont des exemples de packages. |
| **`acs-mirror.azureedge.net`**   | **`HTTPS:443`** | Cette adresse correspond au référentiel nécessaire pour télécharger et installer les binaires requis, comme kubenet et Azure CNI. |

### <a name="azure-china-21vianet-required-network-rules"></a>Règles de réseau obligatoires pour Azure China 21Vianet

Les règles de réseau et les dépendances d’adresse IP requises sont les suivantes :

| Point de terminaison de destination                                                             | Protocol | Port    | Utilisation  |
|----------------------------------------------------------------------------------|----------|---------|------|
| **`*:1194`** <br/> *Ou* <br/> [Balise de service](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.Region:1194`** <br/> *Ou* <br/> [CIDR régionaux](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:1194`** <br/> *Ou* <br/> **`APIServerPublicIP:1194`** `(only known after cluster creation)`  | UDP           | 1194      | Pour les communications sécurisées par tunnel entre les nœuds et le plan de contrôle. |
| **`*:9000`** <br/> *Ou* <br/> [Balise de service](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.<Region>:9000`** <br/> *Ou* <br/> [CIDR régionaux](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:9000`** <br/> *Ou* <br/> **`APIServerPublicIP:9000`** `(only known after cluster creation)`  | TCP           | 9000      | Pour les communications sécurisées par tunnel entre les nœuds et le plan de contrôle. |
| **`*:22`** <br/> *Ou* <br/> [Balise de service](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.<Region>:22`** <br/> *Ou* <br/> [CIDR régionaux](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:22`** <br/> *Ou* <br/> **`APIServerPublicIP:22`** `(only known after cluster creation)`  | TCP           | 22      | Pour les communications sécurisées par tunnel entre les nœuds et le plan de contrôle. |
| **`*:123`** ou **`ntp.ubuntu.com:123`** (en cas d’utilisation de règles de réseau de Pare-feu Azure)  | UDP      | 123     | Obligatoire pour la synchronisation date/heure NTP (Network Time Protocol) sur les nœuds Linux.                 |
| **`CustomDNSIP:53`** `(if using custom DNS servers)`                             | UDP      | 53      | Si vous utilisez des serveurs DNS personnalisés, vous devez vérifier qu’ils sont accessibles par les nœuds de cluster. |
| **`APIServerPublicIP:443`** `(if running pods/deployments that access the API Server)` | TCP      | 443     | Obligatoire en cas d’exécution de pods/déploiements qui accèdent au serveur d’API. Ces pods/déploiements utiliseront l’adresse IP de l’API.  |

### <a name="azure-china-21vianet-required-fqdn--application-rules"></a>Règles de nom FQDN/d’application requises pour Azure China 21Vianet

Les noms de domaine complets/règles d’application suivantes sont requis :

| Nom FQDN de destination                               | Port            | Utilisation      |
|------------------------------------------------|-----------------|----------|
| **`*.hcp.<location>.cx.prod.service.azk8s.cn`**| **`HTTPS:443`** | Obligatoire pour la communication Nœud <-> Serveur d’API. Remplacez *\<location\>* par la région où votre cluster AKS est déployé. |
| **`*.tun.<location>.cx.prod.service.azk8s.cn`**| **`HTTPS:443`** | Obligatoire pour la communication Nœud <-> Serveur d’API. Remplacez *\<location\>* par la région où votre cluster AKS est déployé. |
| **`mcr.microsoft.com`**                        | **`HTTPS:443`** | Obligatoire pour l’accès aux images de Microsoft Container Registry (MCR). Ce registre contient des images/graphiques internes (par exemple, CoreDNS), qui sont nécessaires à la création et au bon fonctionnement du cluster, y compris les opérations de scaling et de mise à niveau. |
| **`.data.mcr.microsoft.com`**                  | **`HTTPS:443`** | Obligatoire pour le stockage MCR assuré par Azure Content Delivery Network (CDN). |
| **`management.chinacloudapi.cn`**              | **`HTTPS:443`** | Obligatoire pour les opérations Kubernetes sur l’API Azure. |
| **`login.chinacloudapi.cn`**                   | **`HTTPS:443`** | Obligatoire pour l’authentification Azure Active Directory. |
| **`packages.microsoft.com`**                   | **`HTTPS:443`** | Cette adresse est le référentiel de packages Microsoft utilisé pour les opérations *apt-get* mises en cache.  Moby, PowerShell et Azure CLI sont des exemples de packages. |
| **`*.azk8s.cn`**                               | **`HTTPS:443`** | Cette adresse correspond au référentiel nécessaire pour télécharger et installer les binaires requis, comme kubenet et Azure CNI. |

### <a name="azure-us-government-required-network-rules"></a>Règles de réseau requises pour Azure US Government

Les règles de réseau et les dépendances d’adresse IP requises sont les suivantes :

| Point de terminaison de destination                                                             | Protocol | Port    | Utilisation  |
|----------------------------------------------------------------------------------|----------|---------|------|
| **`*:1194`** <br/> *Ou* <br/> [Balise de service](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.<Region>:1194`** <br/> *Ou* <br/> [CIDR régionaux](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:1194`** <br/> *Ou* <br/> **`APIServerPublicIP:1194`** `(only known after cluster creation)`  | UDP           | 1194      | Pour les communications sécurisées par tunnel entre les nœuds et le plan de contrôle. |
| **`*:9000`** <br/> *Ou* <br/> [Balise de service](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.<Region>:9000`** <br/> *Ou* <br/> [CIDR régionaux](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:9000`** <br/> *Ou* <br/> **`APIServerPublicIP:9000`** `(only known after cluster creation)`  | TCP           | 9000      | Pour les communications sécurisées par tunnel entre les nœuds et le plan de contrôle. |
| **`*:123`** ou **`ntp.ubuntu.com:123`** (en cas d’utilisation de règles de réseau de Pare-feu Azure)  | UDP      | 123     | Obligatoire pour la synchronisation date/heure NTP (Network Time Protocol) sur les nœuds Linux.                 |
| **`CustomDNSIP:53`** `(if using custom DNS servers)`                             | UDP      | 53      | Si vous utilisez des serveurs DNS personnalisés, vous devez vérifier qu’ils sont accessibles par les nœuds de cluster. |
| **`APIServerPublicIP:443`** `(if running pods/deployments that access the API Server)` | TCP      | 443     | Obligatoire en cas d’exécution de pods/déploiements qui accèdent au serveur d’API. Ces pods/déploiements utiliseront l’adresse IP de l’API.  |

### <a name="azure-us-government-required-fqdn--application-rules"></a>Règles de nom FQDN/d’application requises pour Azure US Government 

Les noms de domaine complets/règles d’application suivantes sont requis :

| Nom FQDN de destination                                        | Port            | Utilisation      |
|---------------------------------------------------------|-----------------|----------|
| **`*.hcp.<location>.cx.aks.containerservice.azure.us`** | **`HTTPS:443`** | Obligatoire pour la communication Nœud <-> Serveur d’API. Remplacez *\<location\>* par la région où votre cluster AKS est déployé.|
| **`mcr.microsoft.com`**                                 | **`HTTPS:443`** | Obligatoire pour l’accès aux images de Microsoft Container Registry (MCR). Ce registre contient des images/graphiques internes (par exemple, CoreDNS), qui sont nécessaires à la création et au bon fonctionnement du cluster, y compris les opérations de scaling et de mise à niveau. |
| **`*.data.mcr.microsoft.com`**                          | **`HTTPS:443`** | Obligatoire pour le stockage MCR assuré par Azure Content Delivery Network (CDN). |
| **`management.usgovcloudapi.net`**                      | **`HTTPS:443`** | Obligatoire pour les opérations Kubernetes sur l’API Azure. |
| **`login.microsoftonline.us`**                          | **`HTTPS:443`** | Obligatoire pour l’authentification Azure Active Directory. |
| **`packages.microsoft.com`**                            | **`HTTPS:443`** | Cette adresse est le référentiel de packages Microsoft utilisé pour les opérations *apt-get* mises en cache.  Moby, PowerShell et Azure CLI sont des exemples de packages. |
| **`acs-mirror.azureedge.net`**                          | **`HTTPS:443`** | Cette adresse correspond au référentiel requis pour installer les fichiers binaires requis comme kubenet et Azure CNI. |

## <a name="optional-recommended-fqdn--application-rules-for-aks-clusters"></a>Règles de nom FQDN/d’application facultatives mais recommandées pour les clusters AKS

Les règles de nom FQDN/d’application suivantes sont facultatives mais recommandées pour les clusters AKS :

| Nom FQDN de destination                                                               | Port          | Utilisation      |
|--------------------------------------------------------------------------------|---------------|----------|
| **`security.ubuntu.com`, `azure.archive.ubuntu.com`, `changelogs.ubuntu.com`** | **`HTTP:80`** | Cette adresse permet aux nœuds de cluster Linux de télécharger les correctifs et mises à jour de sécurité requis. |

Si vous choisissez de bloquer/ne pas autoriser ces noms FQDN, les nœuds ne recevront les mises à jour du système d’exploitation que lors des [mises à niveau des images de nœuds](node-image-upgrade.md) et des [mises à niveau du cluster](upgrade-cluster.md).

## <a name="gpu-enabled-aks-clusters"></a>Clusters AKS compatibles GPU

### <a name="required-fqdn--application-rules"></a>Règles de nom FQDN/d’application requises

Les noms de domaine complets/règles d’application suivants sont requis pour les clusters AKS avec processeur graphique (GPU) activé :

| Nom FQDN de destination                        | Port      | Utilisation      |
|-----------------------------------------|-----------|----------|
| **`nvidia.github.io`**                  | **`HTTPS:443`** | Cette adresse est utilisée pour la bonne installation du pilote et un bon fonctionnement sur les nœuds basés sur le processeur graphique. |
| **`us.download.nvidia.com`**            | **`HTTPS:443`** | Cette adresse est utilisée pour la bonne installation du pilote et un bon fonctionnement sur les nœuds basés sur le processeur graphique. |
| **`apt.dockerproject.org`**             | **`HTTPS:443`** | Cette adresse est utilisée pour la bonne installation du pilote et un bon fonctionnement sur les nœuds basés sur le processeur graphique. |

## <a name="windows-server-based-node-pools"></a>Pools de nœuds basés sur Windows Server 

### <a name="required-fqdn--application-rules"></a>Règles de nom FQDN/d’application requises

Les noms de domaine complets et les règles d’application ci-dessous sont nécessaires à l’utilisation des pools de nœuds Windows Server :

| Nom FQDN de destination                                                           | Port      | Utilisation      |
|----------------------------------------------------------------------------|-----------|----------|
| **`onegetcdn.azureedge.net, go.microsoft.com`**                            | **`HTTPS:443`** | Pour installer les fichiers binaires liés à Windows |
| **`*.mp.microsoft.com, www.msftconnecttest.com, ctldl.windowsupdate.com`** | **`HTTP:80`**   | Pour installer les fichiers binaires liés à Windows |

## <a name="aks-addons-and-integrations"></a>Modules complémentaires et intégrations AKS

### <a name="azure-monitor-for-containers"></a>Azure Monitor pour des conteneurs

Il existe deux moyens de fournir l’accès à Azure Monitor pour les conteneurs : autoriser la [balise de service](../virtual-network/service-tags-overview.md#available-service-tags) Azure Monitor **ou** fournir l’accès aux règles de nom FQDN/d’application requises.

#### <a name="required-network-rules"></a>Règles de réseau requises

Les noms de domaine complets/règles d’application suivantes sont requis :

| Point de terminaison de destination                                                             | Protocol | Port    | Utilisation  |
|----------------------------------------------------------------------------------|----------|---------|------|
| [Balise de service](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureMonitor:443`**  | TCP           | 443      | Ce point de terminaison est utilisé pour envoyer les données de métriques et des journaux à Azure Monitor et à Log Analytics. |

#### <a name="required-fqdn--application-rules"></a>Règles de nom FQDN/d’application requises

Les noms de domaine complets/règles d’application suivants sont requis pour les clusters AKS avec Azure Monitor pour conteneurs activé :

| FQDN                                    | Port      | Utilisation      |
|-----------------------------------------|-----------|----------|
| dc.services.visualstudio.com | **`HTTPS:443`**    | Ce point de terminaison est utilisé pour les métriques et le monitoring des données de télémétrie à l’aide d’Azure Monitor. |
| *.ods.opinsights.azure.com    | **`HTTPS:443`**    | Ce point de terminaison est utilisé par Azure Monitor pour l’ingestion des données Log Analytics. |
| *.oms.opinsights.azure.com | **`HTTPS:443`** | Ce point de terminaison est utilisé par omsagent, qui sert à authentifier le service Log Analytics. |
| *.monitoring.azure.com | **`HTTPS:443`** | Ce point de terminaison est utilisé pour envoyer des données de métriques à Azure Monitor. |

### <a name="azure-dev-spaces"></a>Azure Dev Spaces

Mettez à jour la configuration de votre pare-feu ou votre configuration de la sécurité de façon à autoriser le trafic réseau à destination et en provenance de tous les noms FQDN ci-dessous et des [services d’infrastructure Azure Dev Spaces][dev-spaces-service-tags].

#### <a name="required-network-rules"></a>Règles de réseau requises

| Point de terminaison de destination                                                             | Protocol | Port    | Utilisation  |
|----------------------------------------------------------------------------------|----------|---------|------|
| [Balise de service](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureDevSpaces`**  | TCP           | 443      | Ce point de terminaison est utilisé pour envoyer les données de métriques et des journaux à Azure Monitor et à Log Analytics. |

#### <a name="required-fqdn--application-rules"></a>Règles de nom FQDN/d’application requises 

Les règles de nom de domaine complet/d’application suivantes sont requises pour les clusters AKS avec Azure Dev Spaces activé :

| FQDN                                    | Port      | Utilisation      |
|-----------------------------------------|-----------|----------|
| `cloudflare.docker.com` | **`HTTPS:443`** | Cette adresse est utilisée pour extraire des images Linux Alpine et d’autres Azure Dev Spaces Alpine |
| `gcr.io` | **`HTTPS:443`** | Cette adresse est utilisée pour extraire les images Helm/Tiller |
| `storage.googleapis.com` | **`HTTPS:443`** | Cette adresse est utilisée pour extraire les images Helm/Tiller |


### <a name="azure-policy"></a>Azure Policy

#### <a name="required-fqdn--application-rules"></a>Règles de nom FQDN/d’application requises 

Les noms de domaine complets/règles d’application suivants sont requis pour les clusters AKS avec Azure Policy activé.

| FQDN                                          | Port      | Utilisation      |
|-----------------------------------------------|-----------|----------|
| **`data.policy.core.windows.net`** | **`HTTPS:443`** | Cette adresse est utilisée pour extraire les stratégies Kubernetes et pour signaler l’état de conformité du cluster au service de stratégie. |
| **`store.policy.core.windows.net`** | **`HTTPS:443`** | Cette adresse est utilisée pour extraire les artefacts Gatekeeper de stratégies intégrées. |
| **`gov-prod-policy-data.trafficmanager.net`** | **`HTTPS:443`** | Cette adresse est utilisée pour le bon fonctionnement d’Azure Policy.  |
| **`raw.githubusercontent.com`**               | **`HTTPS:443`** | Cette adresse est utilisée pour extraire les stratégies intégrées de GitHub afin de garantir le bon fonctionnement d’Azure Policy. |
| **`dc.services.visualstudio.com`**            | **`HTTPS:443`** | Le module complémentaire Azure Policy envoie des données de télémétrie au point de terminaison Applications Insights. |

## <a name="restrict-egress-traffic-using-azure-firewall"></a>Limitation du trafic de sortie à l’aide du Pare-feu Azure

Le Pare-feu Azure fournit une balise FQDN Azure Kubernetes Service (`AzureKubernetesService`) pour simplifier cette configuration. 

> [!NOTE]
> La balise FQDN, qui contient tous les noms FQDN listés ci-dessus, est mise à jour automatiquement.
>
> Nous vous recommandons d’avoir un minimum de 20 adresses IP frontales sur le pare-feu Azure pour les scénarios de production afin d’éviter les problèmes d’épuisement des ports SNAT.

Voici un exemple d’architecture du déploiement :

![Topologie verrouillée](media/limit-egress-traffic/aks-azure-firewall-egress.png)

* L’entrée publique est forcée à passer à travers les filtres du pare-feu :
  * Les nœuds d’agent AKS sont isolés dans un sous-réseau dédié.
  * Le [Pare-feu Azure](../firewall/overview.md) est déployé dans son propre sous-réseau.
  * Une règle DNAT convertit l’adresse IP publique du Pare-feu en adresse IP front-end LB.
* Les demandes sortantes vont des nœuds d’agent vers l’adresse IP interne du Pare-feu Azure suivant un [itinéraire défini par l’utilisateur](egress-outboundtype.md).
  * Les demandes des nœuds d’agent AKS suivent une route définie par l’utilisateur qui a été mise en place sur le sous-réseau où le cluster AKS a été déployé.
  * Le pare-feu Azure sort du réseau virtuel depuis un front-end d’adresses IP publiques
  * L’accès à l’Internet public ou à d’autres services Azure circule vers et depuis l’adresse IP du front-end du pare-feu
  * L’accès au plan de contrôle AKS peut être protégé par les [Plages d’adresses IP autorisées du serveur d’API](./api-server-authorized-ip-ranges.md), qui comprennent l’adresse IP front-end publique du pare-feu.
* Trafic interne
  * Si vous le souhaitez, à la place ou en plus d’un [Équilibreur de charge public](load-balancer-standard.md), vous pouvez utiliser un [Équilibreur de charge interne](internal-lb.md) pour le trafic interne, que vous pouvez également isoler sur son propre sous-réseau.


Les étapes ci-dessous utilisent la balise FQDN `AzureKubernetesService` du Pare-feu Azure pour limiter le trafic sortant du cluster AKS et donnent un exemple de configuration du trafic entrant public qui passe par le pare-feu.


### <a name="set-configuration-via-environment-variables"></a>Définir la configuration via des variables d’environnement

Définissez un ensemble de variables d’environnement à utiliser dans les créations de ressources.

```bash
PREFIX="aks-egress"
RG="${PREFIX}-rg"
LOC="eastus"
PLUGIN=azure
AKSNAME="${PREFIX}"
VNET_NAME="${PREFIX}-vnet"
AKSSUBNET_NAME="aks-subnet"
# DO NOT CHANGE FWSUBNET_NAME - This is currently a requirement for Azure Firewall.
FWSUBNET_NAME="AzureFirewallSubnet"
FWNAME="${PREFIX}-fw"
FWPUBLICIP_NAME="${PREFIX}-fwpublicip"
FWIPCONFIG_NAME="${PREFIX}-fwconfig"
FWROUTE_TABLE_NAME="${PREFIX}-fwrt"
FWROUTE_NAME="${PREFIX}-fwrn"
FWROUTE_NAME_INTERNET="${PREFIX}-fwinternet"
```

### <a name="create-a-virtual-network-with-multiple-subnets"></a>Créer un réseau virtuel comprenant plusieurs sous-réseaux

Provisionnez un réseau virtuel avec deux sous-réseaux distincts, un pour le cluster et un pour le pare-feu. Si vous le souhaitez, vous pouvez également en créer un pour l’entrée du service interne.

![Topologie de réseau vide](media/limit-egress-traffic/empty-network.png)

Créez un groupe de ressources pour héberger toutes les ressources.

```azurecli
# Create Resource Group

az group create --name $RG --location $LOC
```

Créez un réseau virtuel comportant deux sous-réseaux pour héberger le cluster AKS et le Pare-feu Azure. Chacun aura son propre sous-réseau. Commençons par le réseau AKS.

```
# Dedicated virtual network with AKS subnet

az network vnet create \
    --resource-group $RG \
    --name $VNET_NAME \
    --location $LOC \
    --address-prefixes 10.42.0.0/16 \
    --subnet-name $AKSSUBNET_NAME \
    --subnet-prefix 10.42.1.0/24

# Dedicated subnet for Azure Firewall (Firewall name cannot be changed)

az network vnet subnet create \
    --resource-group $RG \
    --vnet-name $VNET_NAME \
    --name $FWSUBNET_NAME \
    --address-prefix 10.42.2.0/24
```

### <a name="create-and-set-up-an-azure-firewall-with-a-udr"></a>Créer et configurer un Pare-feu Azure avec une route définie par l’utilisateur

Les règles de trafic entrant et sortant du pare-feu Azure doivent être configurées. L’objectif principal du pare-feu est de permettre aux organisations de configurer des règles de trafic entrant et sortant précises à l’intérieur et à l’extérieur du cluster AKS.

![Pare-feu et route définie par l’utilisateur](media/limit-egress-traffic/firewall-udr.png)


> [!IMPORTANT]
> Si votre cluster ou votre application crée un grand nombre de connexions sortantes dirigées vers les mêmes destinations ou une petite partie d’entre elles, il peut vous falloir davantage d’adresses IP front-end de pare-feu pour éviter d’atteindre le maximum de ports par adresse IP front-end.
> Pour plus d’informations sur la création d’un Pare-feu Azure avec plusieurs adresses IP, cliquez [**ici**](../firewall/quick-create-multiple-ip-template.md).

Créez une ressource d’adresse IP publique de référence SKU standard, qui sera utilisée comme adresse front-end du Pare-feu Azure.

```azurecli
az network public-ip create -g $RG -n $FWPUBLICIP_NAME -l $LOC --sku "Standard"
```

Inscrivez l’extension CLI en préversion pour créer un pare-feu Azure.
```azurecli
# Install Azure Firewall preview CLI extension

az extension add --name azure-firewall

# Deploy Azure Firewall

az network firewall create -g $RG -n $FWNAME -l $LOC --enable-dns-proxy true
```
L’adresse IP créée précédemment peut maintenant être affectée au front-end du pare-feu.

> [!NOTE]
> La configuration de l’adresse IP publique sur le Pare-feu Azure peut prendre quelques minutes.
> Pour pouvoir utiliser le nom FQDN sur les règles de réseau, il faut que le proxy DNS soit activé, car le pare-feu écoute alors le port 53 et transfère les requêtes DNS au serveur DNS spécifié ci-dessus. Il peut ainsi traduire automatiquement ce nom FQDN.

```azurecli
# Configure Firewall IP Config

az network firewall ip-config create -g $RG -f $FWNAME -n $FWIPCONFIG_NAME --public-ip-address $FWPUBLICIP_NAME --vnet-name $VNET_NAME
```

Une fois que la commande précédente a réussi, enregistrez l’adresse IP du front-end du pare-feu pour la configurer ultérieurement.

```bash
# Capture Firewall IP Address for Later Use

FWPUBLIC_IP=$(az network public-ip show -g $RG -n $FWPUBLICIP_NAME --query "ipAddress" -o tsv)
FWPRIVATE_IP=$(az network firewall show -g $RG -n $FWNAME --query "ipConfigurations[0].privateIpAddress" -o tsv)
```

> [!NOTE]
> Si vous utilisez un accès sécurisé au serveur d’API AKS avec des [plages d’adresses IP autorisées](./api-server-authorized-ip-ranges.md), vous devez ajouter l’adresse IP publique du pare-feu dans la plage d’adresses IP autorisées.

### <a name="create-a-udr-with-a-hop-to-azure-firewall"></a>Créer une route définie par l’utilisateur avec un tronçon vers le pare-feu Azure

Azure achemine automatiquement le trafic entre les sous-réseaux, les réseaux virtuels et les réseaux locaux Azure. Si vous souhaitez modifier un routage par défaut d’Azure, vous pouvez le faire en créant une table de routage.

Créez une table de routage à associer à un sous-réseau donné. La table de routage définit le tronçon suivant comme étant le pare-feu Azure créé plus haut. Chaque sous-réseau peut avoir zéro ou une table de routage associée.

```azurecli
# Create UDR and add a route for Azure Firewall

az network route-table create -g $RG -l $LOC --name $FWROUTE_TABLE_NAME
az network route-table route create -g $RG --name $FWROUTE_NAME --route-table-name $FWROUTE_TABLE_NAME --address-prefix 0.0.0.0/0 --next-hop-type VirtualAppliance --next-hop-ip-address $FWPRIVATE_IP --subscription $SUBID
az network route-table route create -g $RG --name $FWROUTE_NAME_INTERNET --route-table-name $FWROUTE_TABLE_NAME --address-prefix $FWPUBLIC_IP/32 --next-hop-type Internet
```

Pour savoir comment vous pouvez remplacer les routes système par défaut d’Azure ou ajouter des routes supplémentaires à la table de routage d’un sous-réseau, consultez la [documentation des tables de routage de réseau virtuel](../virtual-network/virtual-networks-udr-overview.md#user-defined).

### <a name="adding-firewall-rules"></a>Ajout de règles de pare-feu

Voici trois règles de réseau que vous pouvez configurer sur votre pare-feu. Vous devrez peut-être les adapter en fonction de votre déploiement. La première règle autorise l’accès au port 9000 via TCP. La deuxième autorise l’accès aux ports 1194 et 123 via UDP (si vous effectuez un déploiement sur Azure China 21Vianet, il peut vous en falloir [d’autres](#azure-china-21vianet-required-network-rules)). Ces deux règles autorisent uniquement le trafic destiné au CIDR de la région Azure que nous utilisons, dans ce cas USA Est. Enfin, nous allons ajouter une troisième règle réseau ouvrant le port 123 au nom FQDN `ntp.ubuntu.com` via UDP (l’ajout d’un nom FQDN comme règle de réseau est l’une des fonctionnalités spécifiques du Pare-feu Azure ; vous devrez l’adapter quand vous utiliserez vos propres options).

Après avoir défini les règles de réseau, nous allons ajouter également une règle d’application à l’aide de `AzureKubernetesService` qui couvre tous les noms FQDN nécessaires accessibles via le port TCP 443 et le port 80.

```
# Add FW Network Rules

az network firewall network-rule create -g $RG -f $FWNAME --collection-name 'aksfwnr' -n 'apiudp' --protocols 'UDP' --source-addresses '*' --destination-addresses "AzureCloud.$LOC" --destination-ports 1194 --action allow --priority 100
az network firewall network-rule create -g $RG -f $FWNAME --collection-name 'aksfwnr' -n 'apitcp' --protocols 'TCP' --source-addresses '*' --destination-addresses "AzureCloud.$LOC" --destination-ports 9000
az network firewall network-rule create -g $RG -f $FWNAME --collection-name 'aksfwnr' -n 'time' --protocols 'UDP' --source-addresses '*' --destination-fqdns 'ntp.ubuntu.com' --destination-ports 123

# Add FW Application Rules

az network firewall application-rule create -g $RG -f $FWNAME --collection-name 'aksfwar' -n 'fqdn' --source-addresses '*' --protocols 'http=80' 'https=443' --fqdn-tags "AzureKubernetesService" --action allow --priority 100
```

Pour plus d’informations sur le service Pare-feu Azure, consultez la [documentation de Pare-feu Azure](../firewall/overview.md).

### <a name="associate-the-route-table-to-aks"></a>Associer la table de routage à AKS

Pour associer le cluster au pare-feu, le sous-réseau dédié pour le sous-réseau du cluster doit référencer la table de routage créée ci-dessus. L’association peut être effectuée en émettant une commande vers le réseau virtuel contenant à la fois le cluster et le pare-feu pour mettre à jour la table de routage du sous-réseau du cluster.

```azurecli
# Associate route table with next hop to Firewall to the AKS subnet

az network vnet subnet update -g $RG --vnet-name $VNET_NAME --name $AKSSUBNET_NAME --route-table $FWROUTE_TABLE_NAME
```

### <a name="deploy-aks-with-outbound-type-of-udr-to-the-existing-network"></a>Déployer AKS avec le type de sortie UDR sur le réseau existant

Un cluster AKS peut maintenant être déployé dans le réseau virtuel existant. Nous utiliserons également le [type sortant `userDefinedRouting`](egress-outboundtype.md). Cette fonctionnalité garantit que tout le trafic sortant sera forcé à transiter par le pare-feu et qu’il n’existera aucun autre chemin de sortie (par défaut, le type sortant Load Balancer pourrait être utilisé).

![aks-deploy](media/limit-egress-traffic/aks-udr-fw.png)

### <a name="create-a-service-principal-with-access-to-provision-inside-the-existing-virtual-network"></a>Créer un principal de service avec un accès pour provisionner à l’intérieur du réseau virtuel existant

Un principal de service est utilisé par AKS pour créer des ressources de cluster. Le principal de service passé au moment de la création est utilisé pour créer les ressources AKS sous-jacentes, telles que les ressources de stockage, les adresses IP et les équilibreurs de charge utilisés par AKS (il est sinon possible d’utiliser une [identité managée](use-managed-identity.md)). Si vous n’avez pas accordé les autorisations appropriées ci-dessous, vous ne pourrez pas provisionner le cluster AKS.

```azurecli
# Create SP and Assign Permission to Virtual Network

az ad sp create-for-rbac -n "${PREFIX}sp" --skip-assignment
```

À présent, remplacez `APPID` et `PASSWORD` ci-dessous par l’ID d’application et le mot de passe du principal de service générés automatiquement par le résultat de la commande précédente. Nous allons faire référence à l’ID de ressource du réseau virtuel pour accorder au principal de service les autorisations permettant à AKS d’y déployer des ressources.

```azurecli
APPID="<SERVICE_PRINCIPAL_APPID_GOES_HERE>"
PASSWORD="<SERVICEPRINCIPAL_PASSWORD_GOES_HERE>"
VNETID=$(az network vnet show -g $RG --name $VNET_NAME --query id -o tsv)

# Assign SP Permission to VNET

az role assignment create --assignee $APPID --scope $VNETID --role "Network Contributor"
```

Vous pouvez consulter [ici](kubernetes-service-principal.md#delegate-access-to-other-azure-resources) les autorisations détaillées requises.

> [!NOTE]
> Si vous utilisez le plug-in réseau kubenet, vous devez accorder au principal de service AKS ou à l’identité managée des autorisations vis-à-vis de la table de route précréée, car kubenet a besoin d’une table de route pour ajouter les règles d’acheminement nécessaires. 
> ```azurecli-interactive
> RTID=$(az network route-table show -g $RG -n $FWROUTE_TABLE_NAME --query id -o tsv)
> az role assignment create --assignee $APPID --scope $RTID --role "Network Contributor"
> ```

### <a name="deploy-aks"></a>Déployer AKS

Enfin, le cluster AKS peut être déployé dans le sous-réseau existant que nous avons dédié au cluster. Le sous-réseau cible où effectuer le déploiement est défini avec la variable d’environnement `$SUBNETID`. Nous n’avons pas défini la variable `$SUBNETID` dans les étapes précédentes. Pour définir la valeur de l’ID de sous-réseau, vous pouvez utiliser la commande suivante :

```azurecli
SUBNETID=$(az network vnet subnet show -g $RG --vnet-name $VNET_NAME --name $AKSSUBNET_NAME --query id -o tsv)
```

Vous allez définir le type sortant de façon à utiliser la route définie par l’utilisateur qui existe déjà sur le sous-réseau. Cette configuration permet à AKS d’ignorer l’installation et le provisionnement IP pour l’équilibreur de charge.

> [!IMPORTANT]
> Pour plus d’informations sur la route définie par l’utilisateur de type sortant, notamment les limitations, consultez [**Route définie par l’utilisateur de type sortant de sortie**](egress-outboundtype.md#limitations).


> [!TIP]
> Des fonctionnalités supplémentaires peuvent être ajoutées au déploiement du cluster, comme [**Cluster privé**](private-clusters.md). 
>
> La fonctionnalité AKS de [**plages d’adresses IP autorisées du serveur d’API**](api-server-authorized-ip-ranges.md) peut être ajoutée pour limiter l’accès du serveur d’API au point de terminaison public du pare-feu. Elle est indiquée comme facultative dans le diagramme. Quand vous activez la fonctionnalité de plage d’adresses IP autorisées pour limiter l’accès au serveur d’API, vos outils de développement doivent utiliser une jumpbox à partir du réseau virtuel du pare-feu, ou vous devez ajouter tous les points de terminaison de développeur à la plage d’adresses IP autorisées.

```azurecli
az aks create -g $RG -n $AKSNAME -l $LOC \
  --node-count 3 --generate-ssh-keys \
  --network-plugin $PLUGIN \
  --outbound-type userDefinedRouting \
  --service-cidr 10.41.0.0/16 \
  --dns-service-ip 10.41.0.10 \
  --docker-bridge-address 172.17.0.1/16 \
  --vnet-subnet-id $SUBNETID \
  --service-principal $APPID \
  --client-secret $PASSWORD \
  --api-server-authorized-ip-ranges $FWPUBLIC_IP
```

### <a name="enable-developer-access-to-the-api-server"></a>Activer l’accès des développeurs au serveur d’API

Si vous avez utilisé les plages d’adresses IP autorisées pour le cluster à l’étape précédente, vous devez ajouter les adresses IP de vos outils de développement à la liste de clusters AKS des plages d’adresses IP approuvées pour pouvoir accéder au serveur d’API à partir de cet emplacement. Une autre option est de configurer une jumpbox avec les outils nécessaires à l’intérieur d’un sous-réseau distinct dans le réseau virtuel du pare-feu.

Ajoutez une autre adresse IP aux plages approuvées avec la commande suivante

```bash
# Retrieve your IP address
CURRENT_IP=$(dig @resolver1.opendns.com ANY myip.opendns.com +short)

# Add to AKS approved list
az aks update -g $RG -n $AKSNAME --api-server-authorized-ip-ranges $CURRENT_IP/32

```

 Exécutez la commande [az aks get-credentials][az-aks-get-credentials] pour configurer `kubectl` de sorte qu’il se connecte à votre nouveau cluster Kubernetes. 

 ```azurecli
 az aks get-credentials -g $RG -n $AKSNAME
 ```

### <a name="deploy-a-public-service"></a>Déploiement d’un service public
Vous pouvez maintenant commencer à exposer des services et à déployer des applications sur ce cluster. Dans cet exemple, nous allons exposer un service public, mais vous pouvez également choisir d’exposer un service interne au moyen d’un [équilibreur de charge interne](internal-lb.md).

![Règle DNAT de service public](media/limit-egress-traffic/aks-create-svc.png)

Déployez l’application de vote Azure en copiant le code YAML ci-dessous dans un fichier nommé `example.yaml`.

```yaml
# voting-storage-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting-storage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: voting-storage
  template:
    metadata:
      labels:
        app: voting-storage
    spec:
      containers:
      - name: voting-storage
        image: mcr.microsoft.com/aks/samples/voting/storage:2.0
        args: ["--ignore-db-dir=lost+found"]
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_DATABASE
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
---
# voting-storage-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: voting-storage-secret
type: Opaque
data:
  MYSQL_USER: ZGJ1c2Vy
  MYSQL_PASSWORD: UGFzc3dvcmQxMg==
  MYSQL_DATABASE: YXp1cmV2b3Rl
  MYSQL_ROOT_PASSWORD: UGFzc3dvcmQxMg==
---
# voting-storage-pv-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
# voting-storage-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: voting-storage
  labels: 
    app: voting-storage
spec:
  ports:
  - port: 3306
    name: mysql
  selector:
    app: voting-storage
---
# voting-app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: voting-app
  template:
    metadata:
      labels:
        app: voting-app
    spec:
      containers:
      - name: voting-app
        image: mcr.microsoft.com/aks/samples/voting/app:2.0
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: http
        env:
        - name: MYSQL_HOST
          value: "voting-storage"
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_DATABASE
        - name: ANALYTICS_HOST
          value: "voting-analytics"
---
# voting-app-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: voting-app
  labels: 
    app: voting-app
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
    name: http
  selector:
    app: voting-app
---
# voting-analytics-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting-analytics
spec:
  replicas: 1
  selector:
    matchLabels:
      app: voting-analytics
      version: "2.0"
  template:
    metadata:
      labels:
        app: voting-analytics
        version: "2.0"
    spec:
      containers:
      - name: voting-analytics
        image: mcr.microsoft.com/aks/samples/voting/analytics:2.0
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: http
        env:
        - name: MYSQL_HOST
          value: "voting-storage"
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_DATABASE
---
# voting-analytics-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: voting-analytics
  labels: 
    app: voting-analytics
spec:
  ports:
  - port: 8080
    name: http
  selector:
    app: voting-analytics
```

Déployez le service en exécutant :

```bash
kubectl apply -f example.yaml
```

### <a name="add-a-dnat-rule-to-azure-firewall"></a>Ajouter une règle DNAT au pare-feu Azure

> [!IMPORTANT]
> Lorsque vous utilisez le Pare-feu Azure pour limiter le trafic de sortie et créer un itinéraire défini par l’utilisateur (UDR) afin de forcer tout le trafic de sortie, veillez à créer une règle DNAT appropriée dans le Pare-feu pour autoriser le trafic d’entrée. L’utilisation du Pare-feu Azure avec une UDR perturbe la configuration d’entrée en raison d’un routage asymétrique (ce problème se produit si le sous-réseau AKS a un itinéraire par défaut qui conduit à l’adresse IP privée du pare-feu, alors que vous utilisez un service d’équilibreur de charge public, d’entrée ou Kubernetes de type : LoadBalancer). Dans ce cas, le trafic d’équilibreur de charge entrant est reçu par le biais de son adresse IP publique, mais le chemin de retour passe par l’adresse IP privée du pare-feu. Le pare-feu étant avec état, il supprime le paquet de retour, car le pare-feu n’a pas connaissance d’une session établie. Pour découvrir comment intégrer un Pare-feu Azure avec votre équilibreur de charge d’entrée ou de service, voir [Intégrer un pare-feu Azure avec Azure Standard Load Balancer](../firewall/integrate-lb.md).


Pour configurer la connectivité entrante, une règle DNAT doit être écrite sur le pare-feu Azure. Pour tester la connectivité au cluster, une règle est définie de sorte que l’adresse IP publique front-end du pare-feu soit acheminée vers l’adresse IP interne exposée par le service interne.

L’adresse de destination peut être personnalisée, car il s’agit du port d’accès sur le pare-feu. L’adresse traduite doit être l’adresse IP de l’équilibreur de charge interne. Le port traduit doit être le port exposé pour votre service Kubernetes.

Vous devrez spécifier l’adresse IP interne affectée à l’équilibreur de charge créé par le service Kubernetes. Récupérez l’adresse en exécutant :

```bash
kubectl get services
```

L’adresse IP nécessaire est listée dans la colonne EXTERNAL-IP, comme ci-dessous.

```bash
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes         ClusterIP      10.41.0.1       <none>        443/TCP        10h
voting-analytics   ClusterIP      10.41.88.129    <none>        8080/TCP       9m
voting-app         LoadBalancer   10.41.185.82    20.39.18.6    80:32718/TCP   9m
voting-storage     ClusterIP      10.41.221.201   <none>        3306/TCP       9m
```

Récupérez l’adresse IP du service en exécutant :
```bash
SERVICE_IP=$(k get svc voting-app -o jsonpath='{.status.loadBalancer.ingress[*].ip}')
```

Ajoutez la règle NAT en exécutant :
```azurecli
az network firewall nat-rule create --collection-name exampleset --destination-addresses $FWPUBLIC_IP --destination-ports 80 --firewall-name $FWNAME --name inboundrule --protocols Any --resource-group $RG --source-addresses '*' --translated-port 80 --action Dnat --priority 100 --translated-address $SERVICE_IP
```

### <a name="validate-connectivity"></a>Valider la connectivité

Accédez à l’adresse IP du front-end du pare-feu Azure dans un navigateur pour valider la connectivité.

L’application de vote AKS s’affiche. Dans cet exemple, l’IP publique du pare-feu était `52.253.228.132`.


![Capture d’écran montrant l’application AKS Voting avec des boutons pour Cats, Dogs, Reset et les totaux.](media/limit-egress-traffic/aks-vote.png)


### <a name="clean-up-resources"></a>Nettoyer les ressources

Pour nettoyer les ressources Azure, supprimez le groupe de ressources AKS.

```azurecli
az group delete -g $RG
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert les ports et adresses à autoriser pour limiter le trafic de sortie du cluster. Vous avez également vu comment sécuriser votre trafic sortant à l’aide du Pare-feu Azure. 

Si nécessaire, vous pouvez généraliser la procédure ci-dessus pour transférer le trafic vers votre solution de sortie préférée, en suivant la [Documentation `userDefinedRoute` de type sortant](egress-outboundtype.md).

Si vous souhaitez limiter la communication entre les pods et le trafic Est-Ouest au sein du cluster, consultez [Sécurisation du trafic entre les pods à l’aide de stratégies réseau dans AKS][network-policy].

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[network-policy]: use-network-policies.md
[azure-firewall]: ../firewall/overview.md
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[aks-upgrade]: upgrade-cluster.md
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[dev-spaces-service-tags]: ../dev-spaces/configure-networking.md#virtual-network-or-subnet-configurations
