---
title: Architecture de la reprise d’activité après sinistre Azure vers Azure dans Azure Site Recovery
description: Vue d’ensemble de l’architecture utilisée lorsque vous configurez une récupération d’urgence entre des régions Azure pour des machines virtuelles Azure en utilisant le service Azure Site Recovery.
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 3/13/2020
ms.author: raynew
ms.openlocfilehash: 64d1084fd7025c74676977f065062e5e94dabf1d
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97652243"
---
# <a name="azure-to-azure-disaster-recovery-architecture"></a>Architecture pour la récupération d’urgence d’Azure vers Azure


Cet article décrit l’architecture, les composants et les processus qui sont utilisés pour la reprise d’activité de machines virtuelles Azure à l’aide du service [Azure Site Recovery](site-recovery-overview.md). Lorsque vous configurez la reprise d’activité, les machines virtuelles Azure sont répliquées continuellement vers d’autres régions cibles. Si une panne se produit, vous pouvez basculer les machines virtuelles vers la région secondaire et y accéder à partir de cette région. Si tout s’exécute à nouveau normalement, vous pouvez effectuer une restauration automatique et continuer à travailler dans l’emplacement principal.



## <a name="architectural-components"></a>Composants architecturaux

Les composants impliqués dans la reprise d’activité des machines virtuelles Azure sont répertoriés dans le tableau suivant.

**Composant** | **Configuration requise**
--- | ---
**Machines virtuelles dans la région source** | Une ou plusieurs machines virtuelles Azure dans une [région source prise en charge](azure-to-azure-support-matrix.md#region-support).<br/><br/> Les machines virtuelles peuvent exécuter tout type de [système d’exploitation pris en charge](azure-to-azure-support-matrix.md#replicated-machine-operating-systems).
**Stockage de machines virtuelles sources** | Les machines virtuelles Azure peuvent avoir des disques managés ou non managés répartis sur plusieurs comptes de stockage.<br/><br/>[En savoir plus](azure-to-azure-support-matrix.md#replicated-machines---storage) sur le stockage Azure pris en charge
**Réseaux machines virtuelles sources** | Les machines virtuelles peuvent se trouver dans un ou plusieurs sous-réseaux d’un réseau virtuel de la région source. [En savoir plus](azure-to-azure-support-matrix.md#replicated-machines---networking) sur les exigences réseau
**Compte de stockage de cache** | Vous avez besoin d’un compte de stockage de cache dans le réseau source. Lors de la réplication, les modifications apportées aux machines virtuelles sont stockées dans le cache avant d’être envoyées vers le stockage cible.  Les comptes de stockage de cache doivent être Standard.<br/><br/> L’utilisation d’un cache garantit un impact minimal sur les applications de production qui sont exécutées sur une machine virtuelle.<br/><br/> [En savoir plus](azure-to-azure-support-matrix.md#cache-storage) sur les exigences de stockage de cache 
**Ressources cibles** | Les ressources cibles sont utilisées pendant la réplication et lors d’un basculement. Site Recovery peut configurer une ressource cible par défaut. Vous pouvez également en créer ou en personnaliser une.<br/><br/> Dans la région cible, vérifiez que vous pouvez créer des machines virtuelles, et que votre abonnement dispose de suffisamment de ressources pour prendre en charge les machines virtuelles qui seront nécessaires dans la région cible. 

![Schéma illustrant la réplication source et cible.](./media/concepts-azure-to-azure-architecture/enable-replication-step-1-v2.png)

## <a name="target-resources"></a>Ressources cibles

Lorsque vous activez la réplication pour une machine virtuelle, Site Recovery vous donne la possibilité de créer automatiquement des ressources cibles. 

**Ressource cible** | **Paramètre par défaut**
--- | ---
**Abonnement cible** | Identique à celui de l’abonnement source.
**Groupe de ressources cible** | Groupe de ressources auquel appartiennent les machines virtuelles après le basculement.<br/><br/> Il peut se trouver dans n’importe quelle région Azure, à l’exception de la région source.<br/><br/> Site Recovery crée un groupe de ressources dans la région cible avec un suffixe « asr ».<br/><br/>
**Réseau virtuel cible** | Réseau virtuel dans lequel les machines virtuelles répliquées sont situées après le basculement. Un mappage réseau est créé entre les réseaux virtuels source et cible, et inversement.<br/><br/> Site Recovery crée un réseau virtuel et un sous-réseau avec le suffixe « asr ».
**Compte de stockage cible** |  Si la machine virtuelle n’utilise pas de disque managé, il s’agit du compte de stockage dans lequel les données sont répliquées.<br/><br/> Site Recovery crée un compte de stockage dans la région cible qui reflète le compte de stockage source.
**Disques managés de réplica** | Si la machine virtuelle utilise un disque managé, il s’agit du disque managé vers lequel les données sont répliquées.<br/><br/> Site Recovery crée des disques managés de réplica dans la région de stockage pour mettre en miroir la source.
**Groupes à haute disponibilité cibles** |  Groupes à haute disponibilité dans lesquels se trouvent les machines virtuelles répliquées après le basculement.<br/><br/> Site Recovery crée un groupe à haute disponibilité dans la région cible avec le suffixe « asr » pour les machines virtuelles qui se trouvent dans un groupe à haute disponibilité de l’emplacement source. Si un groupe à haute disponibilité existe déjà, celui-ci est utilisé et aucun nouveau groupe n’est créé.
**Zones de disponibilité cibles** | Si la région cible prend en charge les zones de disponibilité, Site Recovery affecte le même nombre de zones que celui utilisé dans la région source.

### <a name="managing-target-resources"></a>Gestion des ressources cibles

Vous pouvez gérer les ressources cibles de la façon suivante :

- Vous pouvez modifier les paramètres de la cible lorsque vous activez la réplication.
- Vous pouvez modifier les paramètres de la cible lorsque la réplication est active. Notez que la référence SKU par défaut pour la machine virtuelle de la région cible est la même que la référence SKU de la machine virtuelle source (ou la meilleure référence SKU suivante par rapport à la référence SKU de la machine virtuelle source). Comme pour d’autres ressources comme le groupe de ressources cible, le nom cible, etc., la référence de machine virtuelle de la région cible peut également être mise à jour une fois la réplication en cours. Le type de disponibilité (instance unique, groupe ou zone) ne peut cependant pas être mis à jour. Pour modifier ce paramètre, vous devez désactiver la réplication, modifier le paramètre, puis la réactiver. 


## <a name="replication-policy"></a>Stratégie de réplication 

Par défaut, lorsque vous activez la réplication de machines virtuelles Azure, Site Recovery crée une stratégie de réplication avec les paramètres par défaut qui sont répertoriés dans le tableau.

**Paramètre de stratégie** | **Détails** | **Par défaut**
--- | --- | ---
**Conservation des points de récupération** | Spécifie la durée pendant laquelle Site Recovery conserve les points de récupération. | 24 heures
**Fréquence des instantanés de cohérence des applications** | Fréquence à laquelle Site Recovery prend des instantanés de cohérence des applications. | Toutes les quatre heures

### <a name="managing-replication-policies"></a>Gestion des stratégies de réplication

Vous pouvez gérer et modifier les paramètres des stratégies de réplication par défaut de la façon suivante :
- Vous pouvez modifier les paramètres lorsque vous activez la réplication.
- Vous pouvez créer une stratégie de réplication à tout moment, puis l’appliquer lorsque vous activez la réplication.

### <a name="multi-vm-consistency"></a>Cohérence multimachine virtuelle

Si vous souhaitez que plusieurs machines virtuelles soient répliquées en même temps et que celles-ci partagent les mêmes points de récupération de cohérence des applications et de cohérence en cas d’incident au moment du basculement, vous pouvez les rassembler dans un groupe de réplication. La cohérence multimachine virtuelle impacte les performances des charges de travail, et doit uniquement être utilisée pour les machines virtuelles qui exécutent des charges de travail nécessitant la cohérence de toutes les machines. 



## <a name="snapshots-and-recovery-points"></a>Captures instantanées et points de récupération

Les points de récupération sont créés à partir de captures instantanées des disques des machines virtuelles qui sont prises à un moment précis dans le temps. Lorsque vous basculez une machine virtuelle, vous utilisez un point de récupération pour restaurer la machine virtuelle à l’emplacement cible.

Lorsque nous effectuons un basculement, nous souhaitons garantir que la machine virtuelle démarre sans perte ni altération des données, et que les données soient cohérentes à la fois sur le système d’exploitation et dans les applications qui s’exécutent sur la machine virtuelle. Cela dépend du type des captures instantanées qui sont prises.

Site Recovery prend des captures instantanées de la façon suivante :

1. Par défaut, Site Recovery prend des instantanés de cohérence en cas d’incident à partir des données ainsi que des instantanés de cohérence des applications si vous spécifiez une fréquence pour ces instantanés.
2. Les points de récupération sont créés à partir de captures instantanées et sont stockés conformément aux paramètres de conservation de la stratégie de réplication.

### <a name="consistency"></a>Cohérence

Le tableau suivant explique les différents types de cohérence.

### <a name="crash-consistent"></a>Cohérence en cas d’incident

**Description** | **Détails** | **Recommandation**
--- | --- | ---
Un instantané de cohérence en cas d’incident capture les données qui se trouvaient sur le disque lorsque l’instantané a été pris. Il n’ajoute aucune donnée en mémoire.<br/><br/> Il contient l’équivalent des données qui étaient présentes sur le disque lorsque la machine virtuelle a planté ou lorsque le cordon d’alimentation a été retiré du serveur au moment où l’instantané a été pris.<br/><br/> La cohérence en cas d’incident ne garantit pas la cohérence des données sur le système d’exploitation ou dans les applications présentes sur la machine virtuelle. | Par défaut, Site Recovery crée des points de récupération de cohérence en cas d’incident toutes les cinq minutes. Ce paramètre ne peut pas être modifié.<br/><br/>  | Aujourd’hui, la plupart des applications peuvent récupérer correctement à partir de points de cohérence en cas d’incident.<br/><br/> Les points de récupération de cohérence en cas d’incident sont généralement suffisants pour la réplication des systèmes d’exploitation et des applications telles que les serveurs DHCP et les serveurs d’impression.

### <a name="app-consistent"></a>Cohérence des applications

**Description** | **Détails** | **Recommandation**
--- | --- | ---
Les points de récupération de cohérence des applications sont créés à partir d’instantanés de cohérence des applications.<br/><br/> Un instantané de cohérence des applications contient toutes les informations d’un instantané de cohérence en cas d’incident ainsi que toutes les données en mémoire et les transactions en cours. | Les instantanés de cohérence des applications utilisent le service de cliché instantané de volume (VSS) :<br/><br/>   1) Azure Site Recovery utilise la méthode de sauvegarde de copie uniquement (VSS_BT_COPY) qui ne change pas le numéro de séquence et l’heure de la sauvegarde de fichier journal de Microsoft SQL </br></br> 2) Lorsqu’un instantané est lancé, le service VSS effectue une opération de copie pour écriture sur le volume.<br/><br/>   3) Avant d’effectuer l’opération de copie pour écriture, le service VSS informe chaque application de l’ordinateur qu’il a besoin de vider ses données résidant en mémoire sur le disque.<br/><br/>   4) VSS permet ensuite à l’application de sauvegarde ou de récupération d’urgence (ici, Site Recovery) de lire les données d’instantanés et de poursuivre. | Les instantanés de cohérence des applications sont réalisés selon la fréquence que vous avez spécifiée. Cette fréquence doit toujours être inférieure à celle que vous définissez pour conserver les points de récupération. Par exemple, si vous conservez les points de récupération à l’aide du paramètre par défaut (24 heures), vous devez définir une fréquence inférieure à 24 heures.<br/><br/>Ces instantanés sont plus complexes et plus longs à réaliser que les instantanés de cohérence en cas d’incident.<br/><br/> Ils affectent les performances des applications qui s’exécutent sur les machines virtuelles où est activée la réplication. 

## <a name="replication-process"></a>Processus de réplication

Lorsque vous activez la réplication pour une machine virtuelle Azure, il se produit les événements suivants :

1. L’extension du service Mobilité de Site Recovery est automatiquement installée sur la machine virtuelle.
2. L’extension inscrit la machine virtuelle auprès de Site Recovery.
3. Une réplication continue commence pour la machine virtuelle.  Les écritures sur disque sont transférées immédiatement vers le compte de stockage de cache à l’emplacement source.
4. Site Recovery traite les données dans le cache, puis les envoie au compte de stockage cible ou aux disques managés de réplica.
5. Une fois les données traitées, des points de récupération de cohérence en cas d’incident sont générés toutes les cinq minutes. Les points de récupération de cohérence des applications sont générés en fonction du paramètre spécifié dans la stratégie de réplication.

![Schéma illustrant le processus de réplication, étape 2.](./media/concepts-azure-to-azure-architecture/enable-replication-step-2-v2.png)

**Processus de réplication**

## <a name="connectivity-requirements"></a>Connectivité requise

 Les machines virtuelles Azure que vous répliquez ont besoin d’une connectivité sortante. Site Recovery n’a jamais besoin de connectivité entrante à la machine virtuelle. 

### <a name="outbound-connectivity-urls"></a>Connectivité sortante (URL)

Si un accès sortant aux machines virtuelles est contrôlé à l’aide d’URL, vous devez autoriser ces URL.

| **Nom**                  | **Commercial**                               | **Secteur public**                                 | **Description** |
| ------------------------- | -------------------------------------------- | ---------------------------------------------- | ----------- |
| Stockage                   | `*.blob.core.windows.net`                  | `*.blob.core.usgovcloudapi.net` | Permet d’écrire les données dans le compte de stockage de cache dans la région source à partir de la machine virtuelle. |
| Azure Active Directory    | `login.microsoftonline.com`                | `login.microsoftonline.us`                   | Fournit l’autorisation et l’authentification aux URL du service Site Recovery. |
| Réplication               | `*.hypervrecoverymanager.windowsazure.com` | `*.hypervrecoverymanager.windowsazure.com`     | Permet à la machine virtuelle de communiquer avec le service Site Recovery. |
| Service Bus               | `*.servicebus.windows.net`                 | `*.servicebus.usgovcloudapi.net`             | Permet à la machine virtuelle d’écrire des données de surveillance et de diagnostic Site Recovery. |
| Key Vault                 | `*.vault.azure.net`                        | `*.vault.usgovcloudapi.net`                  | Autorise l’accès à la réplication pour les machines virtuelles compatibles avec ADE via le portail |
| Azure Automation          | `*.automation.ext.azure.com`               | `*.azure-automation.us`                      | Autorise l’activation de la mise à niveau automatique de l’agent de mobilité pour un élément répliqué via le portail |

### <a name="outbound-connectivity-for-ip-address-ranges"></a>Connectivité sortante pour les plages d’adresses IP

Pour contrôler la connectivité sortante des machines virtuelles à l’aide d’adresses IP, vous devez autoriser ces adresses.
Veuillez noter que les détails des exigences de connectivité réseau sont disponibles dans le [livre blanc de la mise en réseau](azure-to-azure-about-networking.md#outbound-connectivity-using-service-tags) 

#### <a name="source-region-rules"></a>Règles de la région source

**Règle** |  **Détails** | **Balise du service**
--- | --- | --- 
Autoriser le trafic HTTPS sortant : port 443 | Autorise toutes les plages qui correspondent aux comptes de stockage de la région source | Storage.\<region-name>
Autoriser le trafic HTTPS sortant : port 443 | Autorise les plages qui correspondent à Azure Active Directory (Azure AD)  | AzureActiveDirectory
Autoriser le trafic HTTPS sortant : port 443 | Autorise les plages qui correspondent à Event Hub dans la région cible. | EventsHub.\<region-name>
Autoriser le trafic HTTPS sortant : port 443 | Autorise les plages qui correspondent à Azure Site Recovery.  | AzureSiteRecovery
Autoriser le trafic HTTPS sortant : port 443 | Autoriser les plages correspondant à Azure Key Vault (cela est nécessaire uniquement pour l’activation de la réplication des machines virtuelles compatibles avec ADE via le portail) | AzureKeyVault
Autoriser le trafic HTTPS sortant : port 443 | Autoriser les plages correspondant à Azure Automation Controller (cela est nécessaire uniquement pour l’activation de la mise à niveau automatique de l’agent de mobilité pour un élément répliqué via le portail) | GuestAndHybridManagement

#### <a name="target-region-rules"></a>Règles de la région cible

**Règle** |  **Détails** | **Balise du service**
--- | --- | --- 
Autoriser le trafic HTTPS sortant : port 443 | Autorise toutes les plages qui correspondent aux comptes de stockage de la région cible | Storage.\<region-name>
Autoriser le trafic HTTPS sortant : port 443 | Autorise les plages qui correspondent à Azure AD  | AzureActiveDirectory
Autoriser le trafic HTTPS sortant : port 443 | Autorise les plages qui correspondent à Event Hub dans la région source. | EventsHub.\<region-name>
Autoriser le trafic HTTPS sortant : port 443 | Autorise les plages qui correspondent à Azure Site Recovery.  | AzureSiteRecovery
Autoriser le trafic HTTPS sortant : port 443 | Autoriser les plages correspondant à Azure Key Vault (cela est nécessaire uniquement pour l’activation de la réplication des machines virtuelles compatibles avec ADE via le portail) | AzureKeyVault
Autoriser le trafic HTTPS sortant : port 443 | Autoriser les plages correspondant à Azure Automation Controller (cela est nécessaire uniquement pour l’activation de la mise à niveau automatique de l’agent de mobilité pour un élément répliqué via le portail) | GuestAndHybridManagement


#### <a name="control-access-with-nsg-rules"></a>Contrôler l’accès avec des règles de groupe de sécurité réseau

Si vous contrôlez la connectivité des machines virtuelles en filtrant le trafic entrant et sortant des réseaux/sous-réseaux Azure à l’aide de [règles NSG](../virtual-network/network-security-groups-overview.md), notez les exigences suivantes :

- Les règles NSG de la région Azure source doivent autoriser l’accès sortant pour le trafic de réplication.
- Nous vous recommandons de créer des règles dans un environnement de test avant de les utiliser en production.
- Utilisez des [étiquettes de service](../virtual-network/network-security-groups-overview.md#service-tags) au lieu d’autoriser les adresses IP individuelles.
    - Les étiquettes de service correspondent à un groupe de préfixes d’adresses IP permettant de simplifier la création de règles de sécurité.
    - Microsoft met automatiquement à jour les étiquettes de service. 
 
En savoir plus sur la [connectivité sortante](azure-to-azure-about-networking.md#outbound-connectivity-using-service-tags) Site Recovery et sur le [contrôle de la connectivité à l’aide de groupes de sécurité réseau](concepts-network-security-group-with-site-recovery.md)


### <a name="connectivity-for-multi-vm-consistency"></a>Connectivité pour la cohérence multimachine virtuelle

Si vous activez la cohérence multimachine virtuelle, les machines du groupe de réplication communiquent entre elles sur le port 20004.
- Vérifiez qu’aucun dispositif de pare-feu ne bloque la communication interne entre les machines virtuelles sur le port 20004.
- Si vous voulez que les machines virtuelles Linux fassent partie d’un groupe de réplication, vérifiez que le trafic sortant sur le port 20004 est ouvert manuellement conformément aux instructions de la version Linux spécifique.




## <a name="failover-process"></a>Processus de basculement

Quand vous démarrez un basculement, les machines virtuelles sont créées dans le groupe de ressources cible, le réseau virtuel cible, le sous-réseau cible et dans le groupe à haute disponibilité cible. Lors d’un basculement, vous pouvez utiliser n’importe quel point de récupération.

![Schéma illustrant le processus de basculement avec les environnements source et cible.](./media/concepts-azure-to-azure-architecture/failover-v2.png)

## <a name="next-steps"></a>Étapes suivantes

[Répliquer rapidement](azure-to-azure-quickstart.md) une machine virtuelle Azure dans une région secondaire.