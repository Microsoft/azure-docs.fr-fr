---
title: Connectivité de point de terminaison public pour les machines virtuelles Azure avec Standard ILB dans les scénarios de haute disponibilité SAP
description: Connectivité de point de terminaison public pour les machines virtuelles avec Azure Standard Load Balancer dans les scénarios de haute disponibilité SAP
services: virtual-machines-windows,virtual-network,storage,
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 12/01/2020
ms.author: radeltch
ms.openlocfilehash: 9c9979699b5bcb3636adc0f9b58331568ea9cad1
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96486300"
---
# <a name="public-endpoint-connectivity-for-virtual-machines-using-azure-standard-load-balancer-in-sap-high-availability-scenarios"></a>Connectivité de point de terminaison public pour les machines virtuelles avec Azure Standard Load Balancer dans les scénarios de haute disponibilité SAP

L’objectif de cet article est de décrire les configurations qui permettront une connectivité sortante vers les points de terminaison publics. Les configurations sont principalement dans le contexte de la haute disponibilité avec Pacemaker pour SUSE/RHEL.  

Si vous utilisez Pacemaker avec l’agent d’isolation Azure dans votre solution de haute disponibilité, les machines virtuelles doivent disposer d’une connectivité sortante à l’API de gestion Azure. L’article présente plusieurs options pour vous permettre de sélectionner l’option qui convient le mieux à votre scénario.  

## <a name="overview"></a>Vue d’ensemble

Lors de l’implémentation de la haute disponibilité pour les solutions SAP via le clustering, l’un des composants nécessaires est [Azure Load Balancer](../../../load-balancer/load-balancer-overview.md). Azure propose deux références SKU d’équilibrage de charge : Standard et De base.

L’équilibrage de charge Azure Standard offre des avantages par rapport à celui De base. Par exemple, il fonctionne sur les zones de disponibilité Azure, il offre de meilleures fonctions de surveillance et de journalisation pour faciliter la résolution des problèmes, et une latence réduite. La fonctionnalité « Ports haute disponibilité » couvre tous les ports, autrement dit, il n’est plus nécessaire de répertorier tous les ports individuels.  

Il existe quelques différences importantes entre la référence De base et la référence Standard d’Azure Load Balancer. L’une d’entre elles est la gestion du trafic sortant vers le point de terminaison public. Pour en savoir plus sur la comparaison entre les équilibrages de charge Standard et De base, consultez [Comparaison des références SKU d’équilibrage de charge](../../../load-balancer/load-balancer-overview.md).  
 
Lorsque des machines virtuelles sans adresse IP publique sont placées dans le pool principal d’Azure Standard Load Balancer interne (aucune adresse IP publique), il n’y a pas de connectivité sortante aux points de terminaison publics, sauf si une configuration supplémentaire est effectuée.  

Si une machine virtuelle est affectée à une adresse IP publique ou que celle-ci se trouve dans le pool principal d’un équilibreur de charge avec une adresse IP publique, elle aura une connectivité sortante aux points de terminaison publics.  

Les systèmes SAP contiennent souvent des données d’entreprise sensibles. Il est rarement acceptable que des machines virtuelles hébergeant des systèmes SAP soient accessibles via des adresses IP publiques. Dans le même temps, il existe des scénarios qui nécessitent une connectivité sortante entre la machine virtuelle et les points de terminaison publics.  

Voici quelques exemples de scénarios nécessitant l’accès au point de terminaison public Azure :  
- Azure Fence Agent requiert l’accès à **management.azure.com** et à **login.microsoftonline.com**  
- [Azure Backup](https://docs.microsoft.com/azure/backup/tutorial-backup-sap-hana-db#set-up-network-connectivity)
- [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-about-networking#outbound-connectivity-for-urls)  
- Utilisation d’un référentiel public pour la mise à jour corrective du système d’exploitation
- Le trafic de données de l’application SAP peut nécessiter une connectivité sortante vers le point de terminaison public

Si votre déploiement SAP ne nécessite pas de connectivité sortante vers les points de terminaison publics, vous n’avez pas besoin d’implémenter la configuration supplémentaire. Il suffit de créer une référence SKU Azure Load Balancer Standard interne pour votre scénario de haute disponibilité, en supposant qu’il n’y a pas non plus besoin de connectivité entrante à partir de points de terminaison publics.  

> [!Note]
> Lorsque des machines virtuelles sans adresse IP publique sont placées dans le pool principal d’Azure Standard Load Balancer interne (aucune adresse IP publique), il n’y a pas de connectivité Internet sortante, sauf si une configuration supplémentaire est effectuée pour autoriser le routage vers des points de terminaison publics.  
>Si les machines virtuelles ont des adresses IP publiques ou se trouvent déjà dans le pool principal d’Azure Load Balancer avec une adresse IP publique, la machine virtuelle dispose déjà d’une connectivité sortante aux points de terminaison publics.


Lisez tout d’abord les documents suivants :

* Azure Standard Load Balancer
  * [Présentation d’Azure Standard Load Balancer](../../../load-balancer/load-balancer-overview.md) : vue d’ensemble complète de l’équilibrage de charge Standard Azure, principes, concepts et didacticiels importants 
  * [Connexions sortantes dans Azure](../../../load-balancer/load-balancer-outbound-connections.md#scenarios) : scénarios sur la façon d’obtenir une connectivité sortante dans Azure
  * [Règles de trafic sortant dans Load Balancer](../../../load-balancer/load-balancer-outbound-connections.md#outboundrules) : explique les concepts des règles de trafic sortant de Load Balancer et comment créer des règles de trafic sortant
* Pare-feu Azure
  * [Présentation du pare-feu Azure](../../../firewall/overview.md) : une présentation du pare-feu Azure
  * [Tutoriel : Déployer et configurer le pare-feu Azure](../../../firewall/tutorial-firewall-deploy-portal.md) : des instructions sur la configuration du pare-feu Azure via le portail Azure
* [Réseaux virtuels - Règles définies par l’utilisateur](../../../virtual-network/virtual-networks-udr-overview.md#user-defined) : concepts et règles de routage Azure  
* [Balises de service des groupes de sécurité](../../../virtual-network/network-security-groups-overview.md#service-tags) : comment simplifier vos groupes de sécurité réseau et la configuration du pare-feu avec des balises de service

## <a name="option-1-additional-external-azure-standard-load-balancer-for-outbound-connections-to-internet"></a>Option 1 : Un Azure Standard Load Balancer supplémentaire pour les connexions sortantes vers Internet

Une option pour obtenir une connectivité sortante aux points de terminaison publics, et ce sans autoriser la connectivité entrante à la machine virtuelle à partir du point de terminaison public, consiste à créer un second équilibreur de charge avec une adresse IP publique, à ajouter les machines virtuelles au pool principal du deuxième équilibreur de charge et à définir uniquement les [règles sortantes](../../../load-balancer/load-balancer-outbound-connections.md#outboundrules).  
Utilisez les [groupes de sécurité réseau](../../../virtual-network/network-security-groups-overview.md) pour contrôler les points de terminaison publics, qui sont accessibles pour les appels sortants à partir de la machine virtuelle.  
Pour plus d’informations, consultez le scénario 2 dans le document [Connexions sortantes](../../../load-balancer/load-balancer-outbound-connections.md#scenarios).  
La configuration ressemblerait à ceci :  

![Contrôler la connectivité aux points de terminaison publics avec les groupes de sécurité réseau](./media/high-availability-guide-standard-load-balancer/high-availability-guide-standard-load-balancer-public.png)

### <a name="important-considerations"></a>Points importants à prendre en compte

- Vous pouvez utiliser un Load Balancer public supplémentaire pour plusieurs machines virtuelles dans le même sous-réseau pour obtenir une connectivité sortante vers le point de terminaison public et optimiser les coûts  
- Utilisez les [groupes de sécurité réseau](../../../virtual-network/network-security-groups-overview.md) pour contrôler les points de terminaison publics accessibles pour les machines virtuelles. Vous pouvez affecter le groupe de sécurité réseau au sous-réseau ou à chaque machine virtuelle. Dans la mesure du possible, utilisez des [balises de service](../../../virtual-network/network-security-groups-overview.md#service-tags) pour réduire la complexité des règles de sécurité.  
- L’équilibrage de charge Standard Azure avec une adresse IP publique et des règles de trafic sortant permet l’accès direct au point de terminaison public. Si vous avez des exigences de sécurité d’entreprise pour que tout le trafic sortant passe par une solution d’entreprise centralisée pour audit et journalisation, vous ne pourrez peut-être pas répondre aux exigences avec ce scénario.  

>[!TIP]
>Dans la mesure du possible, utilisez des [balises de service](../../../virtual-network/network-security-groups-overview.md#service-tags) pour réduire la complexité du groupe de sécurité réseau. 

### <a name="deployment-steps"></a>Étapes du déploiement

1. Créer un équilibreur de charge  
   1. Dans [le portail Azure](https://portal.azure.com), cliquez sur Toutes les ressources, Ajouter, puis sur **Équilibreur de charge**  
   1. Cliquez sur **Créer** 
   1. Nom de l’équilibreur de charge **MyPublicILB**  
   1. Sélectionnez **Public** en tant que type, **Standard** comme référence SKU  
   1. Sélectionnez **Créer une adresse IP publique** et spécifiez en tant que nom **MyPublicILBFrondEndIP**  
   1. Sélectionnez **Redondant interzone** pour Zone de disponibilité  
   1. Cliquez sur Vérifier et créer, puis sur Créer  
2. Créez un pool principal **MyBackendPoolOfPublicILB** et ajoutez les machines virtuelles.  
   1. Sélectionnez le réseau virtuel  
   1. Sélectionnez les machines virtuelles et leurs adresses IP et ajoutez-les au pool principal  
3. [Créez des règles sortantes](../../../load-balancer/quickstart-load-balancer-standard-public-cli.md?tabs=option-1-create-load-balancer-standard%3ftabs%3doption-1-create-load-balancer-standard#create-outbound-rule-configuration). Actuellement, il n’est pas possible de créer des règles sortantes à partir du portail Azure. Vous pouvez créer des règles de trafic sortant avec [Azure CLI](../../../cloud-shell/overview.md?view=azure-cli-latest).  

   ```azurecli
    az network lb outbound-rule create --address-pool MyBackendPoolOfPublicILB --frontend-ip-configs MyPublicILBFrondEndIP --idle-timeout 30 --lb-name MyPublicILB --name MyOutBoundRules  --outbound-ports 10000 --enable-tcp-reset true --protocol All --resource-group MyResourceGroup
   ```

4. Créez des règles de groupe de sécurité réseau pour restreindre l’accès à des points de terminaison publics spécifiques. S’il existe un groupe de sécurité réseau, vous pouvez l’ajuster. L’exemple ci-dessous montre comment activer l’accès à l’API de gestion Azure : 
   1. Accéder au groupe de sécurité réseau
   1. Cliquez sur Règles de sécurité de trafic entrant
   1. Ajoutez une règle pour **Refuser** tout accès sortant à **Internet**.
   1. Ajoutez une règle à **Autoriser** l’accès à **AzureCloud**, avec une priorité inférieure à la priorité de la règle pour refuser tout accès à Internet.


   Les règles de sécurité de trafic sortant ressembleraient alors à ceci : 

   ![Connexion sortante avec le deuxième équilibreur de charge avec une adresse IP publique](./media/high-availability-guide-standard-load-balancer/high-availability-guide-standard-load-balancer-network-security-groups.png)

   Pour plus d’informations sur les groupes de sécurité réseau Azure, consultez [Groupes de sécurité](../../../virtual-network/network-security-groups-overview.md). 

## <a name="option-2-azure-firewall-for-outbound-connections-to-internet"></a>Option n°2 : Pare-feu Azure pour les connexions sortantes vers Internet

Une autre option pour obtenir une connectivité sortante aux points de terminaison publics, sans autoriser la connectivité entrante à la machine virtuelle à partir de points de terminaison publics, est le pare-feu Azure. Le pare-feu Azure est un service géré avec une haute disponibilité intégrée qui peut s’étendre sur plusieurs zones de disponibilité.  
Vous devez également déployer un [Itinéraire défini par l’utilisateur](../../../virtual-network/virtual-networks-udr-overview.md#custom-routes), associé au sous-réseau dans lequel les machines virtuelles et l’équilibreur de charge Azure sont déployés, pointant vers le pare-feu Azure, pour acheminer le trafic via le pare-feu Azure.  
Pour plus d’informations sur le déploiement du pare-feu Azure, consultez [Déployer et configurer un pare-feu Azure](../../../firewall/tutorial-firewall-deploy-portal.md).  

L’architecture ressemblerait à :

![Connexion sortante avec le pare-feu Azure](./media/high-availability-guide-standard-load-balancer/high-availability-guide-standard-load-balancer-firewall.png)

### <a name="important-considerations"></a>Points importants à prendre en compte

- Le pare-feu Azure est un service cloud natif, avec une haute disponibilité intégrée et prenant en charge le déploiement zonal.
- Nécessite un sous-réseau supplémentaire qui doit être nommé AzureFirewallSubnet. 
- Si vous transférez des jeux de données volumineux sortants du réseau virtuel, sur lequel se trouvent les machines virtuelles SAP, vers une machine virtuelle dans un autre réseau virtuel ou vers un point de terminaison public, il pourrait ne pas s’agir d’une solution économique. Par exemple en cas de copie de sauvegardes volumineuses sur des réseaux virtuels. Pour plus de détails, consultez la tarification du pare-feu Azure.  
- Si la solution de pare-feu d’entreprise n’est pas un pare-feu Azure et que vous avez des exigences de sécurité pour que tout le trafic sortant passe par une solution d’entreprise centralisée, cette solution peut ne pas être pratique.  

>[!TIP]
>Dans la mesure du possible, utilisez des [balises de service](../../../virtual-network/network-security-groups-overview.md#service-tags) pour réduire la complexité des règles du pare-feu Azure.  

### <a name="deployment-steps"></a>Étapes du déploiement

1. Les étapes de déploiement supposent que vous avez déjà défini un réseau virtuel et un sous-réseau pour vos machines virtuelles.  
2. Créez un sous-réseau **AzureFirewallSubnet** dans le même réseau virtuel, où les machines virtuelles et les Standard Load Balancer sont déployés.  
   1. Dans le portail Azure, accédez au réseau virtuel : Cliquez sur Toutes les ressources, recherchez le réseau virtuel, cliquez sur le réseau virtuel, puis sur Sous-réseaux.  
   1. Cliquez sur Ajouter un sous-réseau. Entrez **AzureFirewallSubnet** comme nom. Entrez la plage d’adresses appropriée. Enregistrez.  
3. Créez un pare-feu Azure.  
   1. Dans le portail Azure, sélectionnez Toutes les ressources, cliquez sur Ajouter, Pare-feu, Créer. Sélectionnez Groupe de ressources (sélectionnez le même groupe de ressources que celui sur lequel se trouve le réseau virtuel).  
   1. Entrez le nom de la ressource de pare-feu Azure. Par exemple, **MyAzureFirewall**.  
   1. Sélectionnez Région et sélectionnez au moins deux zones de disponibilité, alignées avec les zones de disponibilité dans lesquelles vos machines virtuelles sont déployées.  
   1. Sélectionnez votre réseau virtuel, dans lequel les machines virtuelles SAP et l’équilibreur de charge standard Azure sont déployés.  
   1. Adresse IP publique : Cliquez sur Créer et entrez un nom. Par exemple **MyFirewallPublicIP**.  
4. Créez une règle de pare-feu Azure pour autoriser la connectivité sortante vers les points de terminaison publics spécifiés. L’exemple montre comment autoriser l’accès au point de terminaison public de l’API de gestion Azure.  
   1. Sélectionnez Règles, Collection de règles de réseau, puis cliquez sur Ajouter une collection de règles de réseau.  
   1. Nom : **MyOutboundRule**, entrez la Priorité, et sélectionnez l’action **Autoriser**.  
   1. Service : Nom **ToAzureAPI**.  Protocole : Sélectionnez **N’importe laquelle**. Adresse source : entrez la plage de votre sous-réseau, où les machines virtuelles et les Standard Load Balancer sont déployés pour l’instance : **11.97.0.0/24**. Ports de destination : saisissez <b>*</b>.  
   1. Enregistrer
   1. Comme vous êtes toujours positionné sur le pare-feu Azure, sélectionnez Vue d’ensemble. Notez l’adresse IP privée du pare-feu Azure.  
5. Créer un itinéraire vers le pare-feu Azure  
   1. Dans le portail Azure sélectionnez Toutes les ressources, puis cliquez sur Ajouter, Table de routage, Créer.  
   1. Entrez le nom MyRouteTable, sélectionnez l’abonnement, le groupe de ressources et l’emplacement (correspondant à l’emplacement de votre réseau virtuel et de votre pare-feu).  
   1. Enregistrer  

   La règle de pare-feu devrait ressembler à ceci : ![Diagramme montrant à quoi ressemble le pare-feu.](./media/high-availability-guide-standard-load-balancer/high-availability-guide-standard-load-balancer-firewall-rule.png)

6. Créez un itinéraire défini par l’utilisateur à partir du sous-réseau de vos machines virtuelles vers l’adresse IP privée de **MyAzureFirewall**.
   1. Comme vous êtes positionné sur la table de routage, cliquez sur Itinéraires. Sélectionnez Ajouter. 
   1. Nom de l’itinéraire : ToMyAzureFirewall, préfixe d’adresse : **0.0.0.0/0**. Type de tronçon suivant : Sélectionnez Appliance virtuelle. Adresse du tronçon suivant : entrez l’adresse IP privée du pare-feu que vous avez configuré : **11.97.1.4**.  
   1. Enregistrer

## <a name="option-3-using-proxy-for-pacemaker-calls-to-azure-management-api"></a>Option 3 : Utilisation du proxy pour les appels de Pacemaker à l’API de gestion Azure

Vous pouvez utiliser le proxy pour autoriser les appels de Pacemaker au point de terminaison public de l’API de gestion Azure.  

### <a name="important-considerations"></a>Points importants à prendre en compte

  - Si un proxy d’entreprise est déjà en place, vous pouvez acheminer les appels sortants vers des points de terminaison publics. Les appels sortants vers des points de terminaison publics passent par le point de contrôle de l’entreprise.  
  - Assurez-vous que la configuration du proxy autorise la connectivité sortante vers l’API de gestion Azure : `https://management.azure.com` et `https://login.microsoftonline.com`  
  - Vérifiez qu’il existe un itinéraire entre les machines virtuelles et le proxy  
  - Le proxy gère uniquement les appels HTTP/HTTPS. S’il est nécessaire d’effectuer des appels sortants vers le point de terminaison public sur différents protocoles (par exemple, RFC), une solution alternative sera nécessaire  
  - La solution de proxy doit être hautement disponible, afin d’éviter l’instabilité dans le cluster Pacemaker  
  - En fonction de l’emplacement du proxy, cela peut ajouter de la latence supplémentaire dans les appels de l’agent d’isolation Azure à l’API de gestion Azure. Si votre proxy d’entreprise est toujours sur site alors que votre cluster Pacemaker est dans Azure, mesurez la latence et déterminez si cette solution est adaptée à vos besoins  
  - Si le proxy d’entreprise n’est pas déjà en place, nous ne recommandons pas cette option, car le client aurait des coûts et une complexité supplémentaires. Néanmoins, si vous décidez de déployer une solution proxy supplémentaire, afin d’autoriser la connectivité sortante de Pacemaker à l’API publique de gestion Azure, vérifiez que le proxy est hautement disponible et que la latence entre les machines virtuelles et le proxy est faible.  

### <a name="pacemaker-configuration-with-proxy"></a>Configuration Pacemaker avec proxy 

Il existe de nombreuses options de proxy disponibles dans le secteur. Les instructions pas à pas pour le déploiement du proxy n’entrent pas dans le cadre de ce document. Dans l’exemple ci-dessous, nous supposons que votre proxy répond à **MyProxyService** et qu’il écoute le port **MyProxyPort**.  
Pour autoriser Pacemaker à communiquer avec l’API de gestion Azure, procédez comme suit sur tous les nœuds de cluster :  

1. Modifiez le fichier de configuration de Pacemaker /etc/sysconfig/pacemaker et ajoutez les lignes suivantes (tous les nœuds de cluster) :

   ```console
   sudo vi /etc/sysconfig/pacemaker
   # Add the following lines
   http_proxy=http://MyProxyService:MyProxyPort
   https_proxy=http://MyProxyService:MyProxyPort
   ```

2. Redémarrez le service Pacemaker sur **tous** les nœuds de cluster.  
  - SUSE
 
     ```console
     # Place the cluster in maintenance mode
     sudo crm configure property maintenance-mode=true
     #Restart on all nodes
     sudo systemctl restart pacemaker
     # Take the cluster out of maintenance mode
     sudo crm configure property maintenance-mode=true
     ```

  - Red Hat  

     ```console
     # Place the cluster in maintenance mode
     sudo pcs property set maintenance-mode=true
     #Restart on all nodes
     sudo systemctl restart pacemaker
     # Take the cluster out of maintenance mode
     sudo pcs property set maintenance-mode=false
     ```

## <a name="other-options"></a>Autres options

Si le trafic sortant est routé via un proxy de pare-feu tiers basé sur une URL :

- Si vous utilisez l’agent de clôture Azure, assurez-vous que la configuration du pare-feu autorise la connectivité sortante vers l’API de gestion Azure : `https://management.azure.com` et `https://login.microsoftonline.com`.   
- Si vous utilisez l’infrastructure de mise à jour du cloud public Azure de SUSE pour appliquer les mises à jour et les correctifs, consultez [Infrastructure de mise à jour de cloud public Azure 101](https://suse.com/c/azure-public-cloud-update-infrastructure-101/)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment configurer Pacemaker sur SUSE dans Azure](./high-availability-guide-suse-pacemaker.md)
* [Découvrez comment configurer Pacemaker sur Red Hat dans Azure](./high-availability-guide-rhel-pacemaker.md)