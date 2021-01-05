---
title: Utiliser le Pare-feu Azure pour protéger des déploiements d’Azure Kubernetes Service (AKS)
description: Découvrez comment utiliser le Pare-feu Azure pour protéger des déploiements d’Azure Kubernetes Service (AKS)
author: vhorne
ms.service: firewall
services: firewall
ms.topic: how-to
ms.date: 09/03/2020
ms.author: victorh
ms.openlocfilehash: 43755b312a64c429b38a07c8c4fad8c85b08342a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89437851"
---
# <a name="use-azure-firewall-to-protect-azure-kubernetes-service-aks-deployments"></a>Utiliser le Pare-feu Azure pour protéger des déploiements d’Azure Kubernetes Service (AKS)

Azure Kubernetes service (AKS) offre un cluster Kubernetes managé sur Azure. Il permet de réduire la complexité et la surcharge opérationnelle liées à la gestion d’un cluster Kubernetes en déléguant une grande partie de cette responsabilité à Azure. AKS gère des tâches critiques, telles que l’analyse du fonctionnement et la maintenance à votre place, et fournit un cluster sécurisé de niveau entreprise avec une gouvernance facilitée.

Kubernetes orchestre les clusters de machines virtuelles et planifie les conteneurs à exécuter sur ces machines virtuelles en fonction de leurs ressources de calcul disponibles et des besoins en ressources de chaque conteneur. Les conteneurs sont regroupés en pods, unités opérationnelles de base dans Kubernetes, qui adaptent leur échelle à l’état de votre choix.

Pour la gestion et à des fins opérationnelles, les nœuds d’un cluster AKS doivent accéder à certains ports et noms de domaine complet (FQDN). Ces actions peuvent consister à communiquer avec le serveur d’API, ou à télécharger, puis installer les principaux composants de cluster Kubernetes et des mises à jour de sécurité des nœuds. Le Pare-feu Azure peut vous aider à verrouiller votre environnement et à filtrer le trafic sortant.

Suivez les instructions de cet article pour renforcer la protection de votre cluster Azure Kubernetes à l’aide du Pare-feu Azure.

## <a name="prerequisites"></a>Prérequis

- Un cluster Azure Kubernetes déployé avec une application en cours d’exécution.

   Pour plus d’informations, consultez le [Guide pratique pour déployer un cluster Azure Kubernetes Service (AKS)](../aks/tutorial-kubernetes-deploy-cluster.md) et le [Guide pratique pour exécuter des applications dans Azure Kubernetes Service (AKS)](../aks/tutorial-kubernetes-deploy-application.md).


## <a name="securing-aks"></a>Sécurisation d’AKS

Le Pare-feu Azure fournit une balise de nom de domaine complet AKS pour simplifier la configuration. Effectuez les étapes suivantes pour autoriser le trafic sortant de la plateforme AKS :

- Lorsque vous utilisez le Pare-feu Azure pour limiter le trafic sortant et créer un itinéraire défini par l’utilisateur afin d’acheminer tout le trafic sortant, veillez à créer une règle DNAT appropriée dans le Pare-feu pour autoriser le trafic entrant. 

   L’utilisation du Pare-feu Azure avec un itinéraire défini par l’utilisateur perturbe la configuration d’entrée en raison d’un routage asymétrique Ce problème se produit si le sous-réseau AKS a un itinéraire par défaut qui conduit à l’adresse IP privée du pare-feu, alors que vous utilisez un service d’équilibreur de charge public. Par exemple, un service d’entrée ou Kubernetes de type *LoadBalancer*.

   Dans ce cas, le trafic d’équilibreur de charge entrant est reçu par le biais de son adresse IP publique, mais le chemin de retour passe par l’adresse IP privée du pare-feu. Le pare-feu étant avec état, il supprime le paquet de retour, car le pare-feu n’a pas connaissance d’une session établie. Pour découvrir comment intégrer un Pare-feu Azure avec votre équilibreur de charge d’entrée ou de service, voir [Intégrer un pare-feu Azure avec Azure Standard Load Balancer](integrate-lb.md).
- Créez une collection de règles d’application et ajoutez une règle pour activer l’étiquette FQDN *AzureKubernetesService*. La plage d’adresses IP source est le réseau virtuel du pool d’hôtes, le protocole est https et la destination est AzureKubernetesService.
- Les ports de sortie/règles de réseau suivants sont requis pour un cluster AKS :

   - Port TCP 443
   - Le port TCP [*IPAddrOfYourAPIServer*]:443 est requis si vous avez une application qui doit communiquer avec le serveur d’API. Cette modification peut être définie après la création du cluster.
   - Port TCP 9000 et port UDP 1194 pour que le pod frontal du tunnel communique avec la fin du tunnel sur le serveur d’API.

      Pour plus de précisions, consultez * *.hcp.<location>.azmk8s.io* et les adresses dans le tableau suivant :

   | Point de terminaison de destination                                                             | Protocol | Port    | Utilisation  |
   |----------------------------------------------------------------------------------|----------|---------|------|
   | **`*:1194`** <br/> *Ou* <br/> [Balise de service](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.<Region>:1194`** <br/> *Ou* <br/> [CIDR régionaux](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:1194`** <br/> *Ou* <br/> **`APIServerIP:1194`** `(only known after cluster creation)`  | UDP           | 1194      | Pour les communications sécurisées par tunnel entre les nœuds et le plan de contrôle. |
   | **`*:9000`** <br/> *Ou* <br/> [Balise de service](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.<Region>:9000`** <br/> *Ou* <br/> [CIDR régionaux](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:9000`** <br/> *Ou* <br/> **`APIServerIP:9000`** `(only known after cluster creation)`  | TCP           | 9000      | Pour les communications sécurisées par tunnel entre les nœuds et le plan de contrôle. |

   - Port UDP 123 pour la synchronisation temporelle NTP (Network Time Protocol) (nœuds Linux).
   - Le port UDP 53 pour DNS est également requis si vous avez des pods qui accèdent directement au serveur d’API.

   Pour plus d’informations, consultez [Contrôler le trafic de sortie pour les nœuds de cluster dans Azure Kubernetes Service (AKS)](../aks/limit-egress-traffic.md).
- Configurez les étiquettes de service AzureMonitor et Storage. Azure Monitor reçoit les données Log Analytics.

   Vous pouvez également autoriser l’URL de votre espace de travail individuellement : `<worksapceguid>.ods.opinsights.azure.com`et `<worksapceguid>.oms.opinsights.azure.com`. Vous pouvez résoudre cette situation de l’une des manières suivantes :

    - Autorisez l’accès https à partir de votre sous-réseau de pool d’hôtes à `*. ods.opinsights.azure.com` et `*.oms. opinsights.azure.com`. Ces noms de domaine complets génériques octroient l’accès requis, mais sont moins restrictifs.
    - Utilisez la requête Log Analytics suivante pour lister les noms de domaine complets requis exacts, puis autorisez-les explicitement dans les règles d’application de votre pare-feu :
   ```
   AzureDiagnostics 
   | where Category == "AzureFirewallApplicationRule" 
   | search "Allow" 
   | search "*. ods.opinsights.azure.com" or "*.oms. opinsights.azure.com"
   | parse msg_s with Protocol " request from " SourceIP ":" SourcePort:int " to " FQDN ":" * 
   | project TimeGenerated,Protocol,FQDN 
   ```


## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur Azure Container Service, consultez [Concepts de base de Kubernetes pour AKS (Azure Kubernetes Service)](../aks/concepts-clusters-workloads.md).
