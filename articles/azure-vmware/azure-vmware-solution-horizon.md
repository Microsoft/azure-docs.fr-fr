---
title: Déployer Horizon sur Azure VMware Solution
description: Découvrez comment déployer VMware Horizon sur Azure VMware Solution.
ms.topic: how-to
ms.date: 09/29/2020
ms.openlocfilehash: 6d5d8e12e358e2289128af9840660be18f5f217a
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95537439"
---
# <a name="deploy-horizon-on-azure-vmware-solution"></a>Déployer Horizon sur Azure VMware Solution 

>[!NOTE]
>Ce document se concentre sur le produit VMware Horizon, anciennement connu sous le nom d'Horizon 7. Horizon est une solution différente d’Horizon Cloud sur Azure, bien qu’il existe des composants partagés. Les principaux avantages d'Azure VMware Solution sont les suivants : méthode de dimensionnement plus simple et intégration de la gestion VMware Cloud Foundation sur le portail Azure.

[VMware Horizon](https://www.vmware.com/products/horizon.html)® est une plateforme de bureaux virtuels et d'applications qui s'exécute dans le centre de données et offre une gestion simple et centralisée. Elle fournit des bureaux virtuels et des applications sur tous types d'appareils, où qu'ils se trouvent. Horizon vous permet de créer et de répartir des connexions à des bureaux virtuels Windows et Linux, à des applications hébergées par les services Bureau à distance (RDS), à des appareils de bureau et à des ordinateurs physiques.

Ici, nous nous concentrons spécifiquement sur le déploiement d’Horizon sur Azure VMware Solution. Pour obtenir des informations générales sur VMware Horizon, reportez-vous à la documentation de production d’Horizon :

* [Qu’est-ce que VMware Horizon ?](https://www.vmware.com/products/horizon.html)

* [En savoir plus sur VMware Horizon](https://docs.vmware.com/en/VMware-Horizon/index.html)

* [Architecture de référence Horizon](https://techzone.vmware.com/resource/workspace-one-and-horizon-reference-architecture)

Avec l’introduction d’Horizon sur Azure VMware Solution, il existe désormais deux solutions d’infrastructure de bureau virtuel sur la plateforme Azure. Le diagramme suivant résume les principales différences à un haut niveau.

:::image type="content" source="media/horizon/difference-horizon-azure-vmware-solution-horizon-cloud-azure.png" alt-text="Horizon sur Azure VMware Solution et Horizon Cloud sur Azure" border="false":::

Horizon 2006 et versions ultérieures sur la série de versions Horizon 8 prend en charge le déploiement local et le déploiement de solutions VMware Azure. Il existe quelques fonctionnalités d’Horizon qui sont prises en charge localement, mais pas sur Azure VMware Solution. D’autres produits de l’écosystème Horizon sont également pris en charge. Pour plus d’informations, consultez [Parité des fonctionnalités et interopérabilité](https://kb.vmware.com/s/article/80850).

## <a name="deploy-horizon-in-a-hybrid-cloud"></a>Déployer Horizon dans un cloud hybride

Vous pouvez déployer Horizon dans un environnement cloud hybride lorsque vous utilisez l'architecture CPA (Cloud Pod Architecture) d'Horizon pour interconnecter des centres de données locaux et des centres de données Azure. L'architecture CPA effectue un scale-up de votre déploiement, crée un cloud hybride et assure la redondance pour la continuité d'activité et la reprise d'activité.  Pour plus d'informations, consultez [Extension des environnements Horizon 7 existants](https://techzone.vmware.com/resource/business-continuity-vmware-horizon#_Toc41650874).

>[!IMPORTANT]
>La CPA n’est pas un déploiement étendu ; chaque pod est distinct, et tous les serveurs de connexion appartenant à chacun des pods individuels doivent être situés dans un emplacement unique et s’exécuter sur le même domaine de diffusion du point de vue du réseau.

À l’instar d’un centre de données local ou privé, Horizon peut être déployé dans un cloud privé Azure VMware Solution. Dans les sections suivantes, nous examinerons les principales différences entre le déploiement d'Horizon en local et sur Azure VMware Solution.

Le cloud privé Azure est identique dans le concept au SDDC (centre de données défini par logiciel) de VMware, un terme généralement utilisé dans la documentation d’Horizon. Le reste de ce document utilise les termes cloud privé Azure et SDDC VMware de façon interchangeable.

Le connecteur Horizon Cloud est requis pour Horizon sur Azure VMware Solution pour gérer les licences d’abonnement. Le connecteur cloud peut être déployé dans un réseau virtuel Azure avec des serveurs de connexion Horizon.

>[!IMPORTANT]
>La prise en charge du plan de contrôle d’Horizon sur Azure VMware Solution n’est pas encore disponible. Veillez à télécharger la version VHD du connecteur Horizon Cloud.

## <a name="vcenter-cloud-admin-role"></a>Rôle d’administrateur du cloud vCenter

Étant donné qu’Azure VMware Solution est un service SDDC et qu’Azure gère le cycle de vie de la solution SDDC sur Azure VMware, le modèle d’autorisation vCenter sur Azure VMware Solution est limité par nature.

Les clients doivent utiliser le rôle d’administrateur cloud, qui dispose d’un ensemble limité d’autorisations vCenter. Le produit Horizon a été modifié pour fonctionner avec le rôle d’administrateur cloud sur Azure VMware Solution, plus précisément :

* La configuration de clonage instantané a été modifiée pour s’exécuter sur Azure VMware Solution.

* Une stratégie vSAN spécifique (VMware_Horizon) a été créée sur Azure VMware Solution pour fonctionner avec Horizon ; elle doit être disponible et utilisée dans les SDDC déployés pour Horizon.

* Le cache de lecture basé sur le contenu (CBRC) vSphere, également connu sous le nom de View Storage Accelerator, est désactivé lors de l’exécution sur Azure VMware Solution.

>[!IMPORTANT]
>Le CBRC ne doit pas être réactivé.

>[!NOTE]
>Azure VMware Solution configure automatiquement les paramètres spécifiques à Horizon tant que vous déployez Horizon 2006 (aussi appelé Horizon 8) ou une version ultérieure sur la branche Horizon 8 et que vous sélectionnez l’option **Azure** dans le programme d’installation du serveur de connexion Horizon.

## <a name="horizon-on-azure-vmware-solution-deployment-architecture"></a>Horizon sur l’architecture de déploiement Azure VMware Solution

Une conception d’architecture d’Horizon classique utilise une stratégie de pods et de blocs. Un bloc est un vCenter unique, tandis que plusieurs blocs combinés forment un pod. Un pod Horizon est une unité d’organisation déterminée par les limites de scalabilité d’Horizon. Chaque pod Horizon est doté d'un portail de gestion distinct. Par conséquent, une pratique de conception standard consiste à réduire le nombre de pods.

Chaque cloud possède son propre schéma de connectivité réseau. Associée à la mise en réseau du SDDC VMware/NSX Edge, la connectivité réseau d’Azure VMware Solution présente des exigences uniques pour le déploiement d’Horizon, différentes de l’environnement local.

Chaque cloud privé Azure/SDDC peut gérer 4 000 sessions de bureau ou d'application, à condition que :

* Le trafic de la charge de travail soit aligné sur le profil du collaborateur associé à la tâche LoginVSI.

* Seul le trafic de protocole est pris en compte, pas les données utilisateur.

* NSX Edge est configuré pour être grand.

>[!NOTE]
>Le profil et les besoins de votre charge de travail peuvent être différents. Par conséquent, les résultats peuvent varier en fonction de votre cas d’utilisation. Les volumes de données utilisateur peuvent réduire les limites d’échelle dans le contexte de votre charge de travail. Dimensionnez et planifiez votre déploiement en conséquence. Pour plus d’informations, consultez les instructions de dimensionnement dans la section [Dimensionnement des hôtes Azure VMware Solution pour les déploiements Horizon](#size-azure-vmware-solution-hosts-for-horizon-deployments).

Compte tenu de la limite maximale du cloud privé Azure et du SDDC, nous vous recommandons d'utiliser une architecture de déploiement dans laquelle les instances d'Horizon Connection Server et de VMware Unified Access Gateway (UAG) sont exécutées au sein du Réseau virtuel Azure. Chaque cloud privé Azure et chaque SDDC seront ainsi transformés en bloc, ce qui optimisera la scalabilité d'Horizon lors de son exécution sur Azure VMware Solution.

La connexion entre le réseau virtuel Azure et les clouds privés Azure/SDDC doit être configurée avec ExpressRoute FastPath. Le diagramme suivant illustre un déploiement de pod Horizon de base.

:::image type="content" source="media/horizon/horizon-pod-deployment-expresspath-fast-path.png" alt-text="Déploiement standard d’un pod à l’aide d’ExpressPath FastPath" border="false":::

## <a name="network-connectivity-to-scale-horizon-on-azure-vmware-solution"></a>Connectivité réseau pour mettre à l’échelle Horizon sur Azure VMware Solution

Cette section présente l'architecture réseau à un niveau élevé, avec des exemples de déploiement courants, pour faciliter la mise à l'échelle d'Horizon sur Azure VMware Solution. L'accent est mis sur les éléments critiques de la mise en réseau. 

### <a name="single-horizon-pod-on-azure-vmware-solution"></a>Pod Horizon unique sur Azure VMware Solution

:::image type="content" source="media/horizon/single-horizon-pod-azure-vmware-solution.png" alt-text="Pod Horizon unique sur Azure VMware Solution" border="false":::

Le scénario de déploiement d'un pod Horizon unique est le plus simple, car vous ne déployez qu'un seul pod Horizon dans la région USA Est.  Comme chaque cloud privé/SDDC peut gérer 4 000 sessions de bureau, vous déployez la taille maximale du pod Horizon.  Vous pouvez au maximum planifier le déploiement de trois clouds privés/SDDC.

Avec un déploiement des machines virtuelles de l'infrastructure d'Horizon sur le Réseau virtuel Azure, vous pouvez atteindre 12 000 sessions par pod Horizon. La connexion entre chaque cloud privé/SDDC et le Réseau virtuel Azure s'effectue via ExpressRoute FastPath.  Aucun trafic est-ouest n'est nécessaire entre les clouds privés. 

Voici quelques hypothèses clés pour cet exemple de déploiement de base :

* Vous n’avez pas de module Horizon local que vous souhaitez connecter à ce nouveau pod à l’aide de l’architecture du pod cloud (CPA).

* Les utilisateurs finaux se connectent à leurs bureaux virtuels via Internet (plutôt que par le biais d'un centre de données local).

Sur le Réseau virtuel Azure, vous connectez votre contrôleur de domaine AD à votre instance d'AD locale via un réseau VPN ou un circuit ExpressRoute.

Une variante de l'exemple de base pourrait consister à prendre en charge la connectivité pour les ressources locales. Par exemple, il pourrait s'agir d'utilisateurs qui accèdent aux appareils de bureau et génèrent du trafic d'application de bureau virtuel ou se connectent à un pod local à l'aide de l'architecture CPA.

Le diagramme montre comment prendre en charge la connectivité pour les ressources locales. Pour connecter votre réseau d'entreprise au Réseau virtuel Azure, vous aurez besoin d'un circuit ExpressRoute.  Vous devrez également connecter votre réseau d'entreprise à chacun des cloud privés et des SDDC à l'aide d'ExpressRoute Global Reach.  Celui-ci permet la connectivité entre le SDDC et le circuit ExpressRoute et les ressources locales. 

:::image type="content" source="media/horizon/connect-corporate-network-azure-virtual-network.png" alt-text="Connectez votre réseau d’entreprise à un réseau virtuel Azure" border="false":::

### <a name="multiple-horizon-pods-on-azure-vmware-solution-across-multiple-regions"></a>Plusieurs modules Horizon sur Azure VMware Solution dans plusieurs régions

Un autre scénario consiste à procéder à la mise à l'échelle d'Horizon sur plusieurs pods.  Dans ce scénario, vous déployez deux pods Horizon dans deux régions différentes et les fédérez à l'aide de l'architecture CPA. La configuration réseau est semblable à celle de l'exemple précédent, mais avec quelques liaisons supplémentaires entre les régions. 

Vous connecterez le Réseau virtuel Azure de chaque région aux clouds privés/SDDC de l'autre région. Cela permettra aux serveurs de connexion Horizon qui font partie de la fédération CPA de se connecter à tous les appareils de bureau sous gestion. L'ajout de clouds privés/SDDC supplémentaires à cette configuration vous permettrait de passer à 24 000 sessions au total. 

Les mêmes principes s'appliquent si vous déployez deux pods Horizon dans la même région.  Veillez à déployer le deuxième pod Horizon sur un *Réseau virtuel Azure distinct*. Comme dans l'exemple à pod unique, vous pouvez connecter votre réseau d'entreprise et votre pod local à cet exemple de pods multiples/région à l'aide d'ExpressRoute et de Global Reach. 

:::image type="content" source="media/horizon/multiple-horizon-pod-azure-vmware-solution.png" alt-text="Plusieurs modules Horizon sur Azure VMware Solution dans plusieurs régions" border="false":::

## <a name="size-azure-vmware-solution-hosts-for-horizon-deployments"></a>Dimensionnement des hôtes Azure VMware Solution pour les déploiements Horizon 

La méthodologie de dimensionnement d'Horizon sur un hôte exécuté dans Azure VMware Solution est plus simple qu'en local.  Cela est dû au fait que l'hôte d'Azure VMware Solution est standardisé.  Le dimensionnement exact de l'hôte permet de déterminer le nombre d'hôtes nécessaires pour répondre à vos besoins en matière d'infrastructure VDI.  Il est essentiel pour déterminer le coût par appareil de bureau.

### <a name="sizing-tables"></a>Dimensionnement des tables

Les tables affichent les charges de travail courantes de travailleur du savoir et travailleur avancé Login VSI.

#### <a name="knowledge-worker-workloads"></a>Charges de travail de travailleur du savoir

:::image type="content" source="media/horizon/common-vdi-profiles-vsi-workloads-knowledge.png" alt-text="Table des profils VDI courants pour VMware Horizon pour les charges de travail de travailleur du savoir VSI" lightbox="media/horizon/common-vdi-profiles-vsi-workloads-knowledge.png" border="false":::

#### <a name="power-worker-workloads"></a>Charges de travail de travailleur avancé

:::image type="content" source="media/horizon/common-vdi-profiles-vsi-workloads-power.png" alt-text="Table des profils VDI courants pour VMware Horizon pour les charges de travail de travailleur avancé VSI" lightbox="media/horizon/common-vdi-profiles-vsi-workloads-power.png" border="false":::

### <a name="horizon-sizing-inputs"></a>Entrées de dimensionnement d’Horizon

Voici ce que vous devrez collecter pour votre charge de travail planifiée :

* Nombre de bureaux simultanés

* Processeurs virtuels requis par bureau

* VRAM requise par bureau

* Stockage requis par bureau

En général, les déploiements VDI sont soumis à des contraintes de processeur ou de RAM, ce qui détermine la taille de l'hôte. Prenons l’exemple suivant pour un type de charge de travail Travailleur du savoir LoginVSI, validé avec les tests de performances :

* Déploiement simultané de 2 000 postes de travail

* 2 processeurs virtuels par bureau.

* 4 Go de vRAM par appareil de bureau.

* 50 Go de stockage par bureau

Pour cet exemple, le nombre total d’hôtes a pour valeur 18, ce qui donne une densité de machines virtuelles par hôte de 111.

>[!IMPORTANT]
>Les charges de travail des clients seront différentes de cet exemple de travailleur du savoir LoginVSI. Dans le cadre de la planification de votre déploiement, collaborez avec vos ingénieurs VMware EUC pour répondre à vos besoins spécifiques en matière de dimensionnement et de performances. Veillez à exécuter vos propres tests de performances sur la base de la charge de travail réelle et planifiée avant de finaliser le dimensionnement de l’hôte et de l’ajuster en conséquence.

## <a name="horizon-on-azure-vmware-solution-licensing"></a>Frais de licence Horizon sur Azure VMware Solution 

Il y a quatre composants pour les coûts globaux d’exécution d’Horizon sur Azure VMware Solution. 

### <a name="azure-vmware-solution-capacity-cost"></a>Coût de la capacité Azure VMware Solution

Pour plus d’informations sur la tarification, consultez la page [Tarification d’Azure VMware Solution](https://azure.microsoft.com/pricing/details/azure-vmware/)

### <a name="horizon-licensing-cost"></a>Coût des licences Horizon

Deux licences sont disponibles pour une utilisation avec Azure VMware Solution, avec un utilisateur simultané (CCU) ou un utilisateur nommé (NU) :

* Licence d’abonnement Horizon

* Licence d’abonnement universelle Horizon

Si vous ne prévoyez de déployer qu’Horizon sur Azure VMware Solution dans un avenir proche, utilisez la licence d’abonnement Horizon, car elle est moins coûteuse.

S'il est déployé sur Azure VMware Solution et localement, comme dans un cas d'usage de récupération d'urgence, choisissez la licence d'abonnement universelle Horizon. Celle-ci comprend une licence vSphere pour le déploiement local, ce qui se traduit par un coût plus élevé.

Collaborez avec votre équipe commerciale VMware EUC pour déterminer le coût des licences Horizon en fonction de vos besoins.

### <a name="cost-of-the-horizon-infrastructure-vms-on-azure-virtual-network"></a>Coût des machines virtuelles de l’infrastructure Horizon sur le réseau virtuel Azure

Basées sur l'architecture de déploiement standard, les machines virtuelles de l'infrastructure Horizon sont constituées d'instances de Connection Server et d'UAG, ainsi que de gestionnaires de volume d'application. Elles sont déployées sur le Réseau virtuel Azure du client. Des instances Azure natives supplémentaires sont requises pour prendre en charge les services de haute disponibilité (HA), Microsoft SQL ou Microsoft Active Directory (AD) sur Azure. Le tableau répertorie les instances Azure sur la base d'un exemple de déploiement de 2 000 appareils de bureau. 

>[!NOTE]
>Pour être en mesure de gérer la défaillance, déployez un serveur de plus qu’il n’est requis pour le nombre de connexions (n+1). Le nombre minimal recommandé d’instances du serveur de connexion, d’UAG et du gestionnaire de volume d’application est 2, et le nombre requis augmente en fonction du nombre d’utilisateurs que l’environnement prendra en charge.  Un serveur de connexion unique prend en charge un maximum de 4 000 sessions, même si 2 000 sont recommandées comme bonne pratique. Jusqu’à sept serveurs de connexion sont pris en charge par pod, avec une recommandation de 12 000 sessions actives au total par pod. Pour obtenir les nombres les plus récents, consultez l’article de base de connaissances VMware sur les [limites et recommandations de dimensionnement d’Horizon 7](https://kb.vmware.com/s/article/2150348).

| Composant d'infrastructure Horizon | Instance Azure | Nombre d’instances nécessaires (pour 2 000 bureaux)    | Comment  |
|----------------------------------|----------------|----------------------------------------------------|----------|
| Serveur de connexion                | D4sv3          | 2       | *Voir la remarque ci-dessus*                         |    
| UAG                              | F2sv2          | 2       | *Voir la remarque ci-dessus*                         |
| Gestionnaire des volumes d’application              | D4sv3          | 2       | *Voir la remarque ci-dessus*                         |
| Connecteur cloud                  | D4sv3          | 1       |                                          |
| Contrôleur AD                    | D4sv3          | 2       | *Option pour utiliser le service AD MSFT sur Azure* |
| Base de données MS-SQL                  | D4sv3          | 2       | *Option permettant d’utiliser le service SQL sur Azure*     |
| Partage de fichiers Windows               | D4sv3          |         | *Facultatif*                               |

Le coût de la machine virtuelle d’infrastructure est de 0,36 \$ par utilisateur et par mois pour le déploiement de 2 000 bureaux dans l’exemple ci-dessus. Cet exemple utilise la tarification de juin 2020 des instances Azure pour la région USA Est. Votre tarification peut varier en fonction de la région, des options sélectionnées et du moment applicable.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus concernant VMware Horizon sur Azure VMware Solution, consultez le [FAQ VMware Horizon](https://www.vmware.com/content/dam/digitalmarketing/vmware/en/pdf/products/horizon/vmw-horizon-on-microsoft-azure-vmware-solution-faq.pdf).
