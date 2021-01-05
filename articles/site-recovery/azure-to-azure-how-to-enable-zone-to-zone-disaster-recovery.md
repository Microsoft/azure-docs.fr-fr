---
title: Activer la récupération d’urgence de zone à zone pour les machines virtuelles Azure
description: Cet article explique quand et comment utiliser la récupération d’urgence de zone à zone pour les machines virtuelles Azure.
author: sideeksh
manager: gaggupta
ms.service: site-recovery
ms.topic: article
ms.date: 04/28/2020
ms.author: sideeksh
ms.openlocfilehash: 6a7d29d8487c6be6915ae4171b3759f0980f5448
ms.sourcegitcommit: 642988f1ac17cfd7a72ad38ce38ed7a5c2926b6c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94874353"
---
# <a name="enable-azure-vm-disaster-recovery-between-availability-zones"></a>Activer la récupération d’urgence des machines virtuelles Azure entre les zones de disponibilité

Cet article explique comment répliquer, basculer et restaurer automatiquement des machines virtuelles Azure d’une zone de disponibilité à une autre au sein de la même région Azure.

>[!NOTE]
>
>- La prise en charge de la récupération d’urgence de zone à zone est actuellement limitée à deux régions : Asie Sud-Est et Royaume-Uni Sud.  
>- Site Recovery ne déplace pas et ne stocke pas les données client hors de la région dans laquelle il est déployé lorsque le client utilise la récupération d’urgence de zone à zone. S’ils le souhaitent, les clients peuvent sélectionner un coffre Recovery Services dans une autre région. Le coffre Recovery Services contient des métadonnées, mais pas de données client réelles.

Le service Site Recovery contribue à votre stratégie de continuité d’activité et reprise d’activité en veillant à ce que vos applications métier restent opérationnelles lors d’interruptions planifiées ou non. Il s’agit de l’option de récupération d’urgence recommandée pour maintenir vos applications opérationnelles en cas de pannes régionales.

Les Zones de disponibilité sont des emplacements physiques uniques au sein d’une région Azure. Chaque zone possède un ou plusieurs centres de données. 

Si vous souhaitez déplacer des machines virtuelles vers une zone de disponibilité située dans une autre région, [consultez cet article](../resource-mover/move-region-availability-zone.md).

## <a name="using-availability-zones-for-disaster-recovery"></a>Utilisation de zones de disponibilité pour la récupération d’urgence 

En règle générale, des zones de disponibilité sont utilisés pour déployer des machines virtuelles dans une configuration à haute disponibilité. Elles peuvent être trop proches les unes des autres pour servir de solution de récupération d’urgence en cas de catastrophe naturelle.

Toutefois, dans certains scénarios, il est possible de tirer parti des zones de disponibilité pour la récupération d’urgence :

- De nombreux clients qui avaient mis en place une stratégie de récupération d’urgence Metro quand ils hébergeaient des applications localement cherchent parfois à imiter cette stratégie après avoir migré des applications vers Azure. Ces clients sont conscients qu’une stratégie de récupération d’urgence Metro risque de ne pas fonctionner en cas de sinistre physique à grande échelle, et acceptent ce risque. Pour ces clients, une récupération d’urgence de zone à zone peut être une bonne option de récupération d’urgence.

- De nombreux autres clients ont une infrastructure réseau complexe qu’ils ne souhaitent pas recréer dans une région secondaire en raison des coûts et de la complexité que cela entraînerait. La récupération d’urgence de zone à zone réduit la complexité, car elle exploite des concepts de mise en réseau redondante dans des zones de disponibilité, qui simplifient considérablement la configuration. Ces clients préfèrent la simplicité et peuvent également utiliser des zones de disponibilité pour la récupération d’urgence.

- Dans certaines régions qui n’ont pas de région couplée au sein de la même juridiction (par exemple, Asie Sud-Est), la récupération d’urgence de zone à zone peut servir de fait de solution de récupération d’urgence, car elle permet de garantir la conformité légale du fait que les applications et données ne franchissent pas de frontières nationales. 

- La récupération d’urgence de zone à zone implique la réplication de données sur des distances plus courtes en comparaison de la récupération d’urgence d’Azure vers Azure. Il se peut que vous constatiez une latence faible et, par conséquent, un RPO inférieur.

Bien que ces avantages soient significatifs, il est possible que la récupération d’urgence de zone à zone ne soit pas à la hauteur des exigences de résilience en cas de catastrophe naturelle à l’échelle d’une région.

## <a name="networking-for-zone-to-zone-disaster-recovery"></a>Mise en réseau pour la récupération d’urgence de zone à zone

Comme mentionné ci-dessus, la récupération d’urgence de zone à zone réduit la complexité, car elle exploite des concepts de mise en réseau redondante dans des zones de disponibilité, qui simplifient considérablement la configuration. Le comportement des composants de mise en réseau dans le scénario de récupération d’urgence de zone en zone est décrit ci-dessous : 

- Réseau virtuel : pour des basculements réels, vous pouvez utiliser le même réseau virtuel que le réseau source. Pour des basculements de test, utilisez un réseau virtuel différent du réseau virtuel source.

- Sous-réseau : un basculement vers le même sous-réseau est pris en charge.

- Adresse IP privée : lorsque vous utilisez des adresses IP statiques, vous pouvez utiliser les mêmes adresses IP dans la zone cible si vous choisissez de les configurer de cette manière.

- Mise en réseau accélérée : comme pour la récupération d’urgence d’Azure vers Azure, vous pouvez activer la mise en réseau accélérée si la référence (SKU) de machine virtuelle la prend en charge.

- Adresse IP publique : vous pouvez attacher une adresse IP publique standard créée précédemment dans la même région à la machine virtuelle cible. Les adresses IP publiques de base ne prennent pas en charge les scénarios liés à une zone de disponibilité.

- Équilibreur de charge : un équilibreur de charge standard étant une ressource régionale, la machine virtuelle cible peut être attachée au pool principal du même équilibreur de charge. Aucun nouvel équilibreur de charge n’est requis.

- Groupe de sécurité réseau : vous pouvez utiliser les mêmes groupes de sécurité réseau que ceux appliqués à la machine virtuelle source.

## <a name="pre-requisites"></a>Conditions préalables

Avant de déployer une récupération d’urgence de zone à zone pour des machines virtuelles, il est important de s’assurer que les autres fonctionnalités activées sur les machines virtuelles sont interopérables.

|Fonctionnalité  | Déclaration de support  |
|---------|---------|
|les machines virtuelles Classic,   |     Non pris en charge    |
|Machines virtuelles ARM    |    Prise en charge    |
|Azure Disk Encryption v1 (deux passes, avec Azure Active Directory [Azure AD])     |     Prise en charge   |
|Azure Disk Encryption v2 (passage unique, sans Azure AD)    |    Prise en charge    |
|Disques non managés    |    Non pris en charge    |
|Disques managés    |    Prise en charge    |
|Clés gérées par le client    |    Prise en charge    |
|Groupes de placements de proximité    |    Prise en charge    |
|Interopérabilité de la sauvegarde    |    La sauvegarde et la restauration au niveau fichier sont prises en charge. La sauvegarde et restauration aux niveaux disque et machine virtuelle ne sont pas prises en charge.    |
|Ajout/suppression à chaud    |    Il est possible d’ajouter des disques après activation de la réplication de zone à zone. La suppression de disques après l’activation de la réplication de zone à zone n’est pas prise en charge.    | 

## <a name="set-up-site-recovery-zone-to-zone-disaster-recovery"></a>Configurer la récupération d’urgence de zone à zone de Site Recovery

### <a name="log-in"></a>Se connecter

Connectez-vous au portail Azure.

### <a name="enable-replication-for-the-zonal-azure-virtual-machine"></a>Activer la réplication pour la machine virtuelle Azure zonale

1. Dans le menu du portail Azure, sélectionnez Machines virtuelles, ou recherchez et sélectionnez Machines virtuelles sur n’importe quelle page. Sélectionnez la machine virtuelle à répliquer. Pour la récupération d’urgence de zone à zone, cette machine virtuelle doit déjà se trouver dans une zone de disponibilité.

2. Dans Opérations, sélectionnez Récupération d’urgence.

3. Comme indiqué ci-dessous, sous l’onglet Fonctions de base, sélectionnez « Oui » pour « Récupération d’urgence entre Zones de disponibilité ? ».

    ![Page Paramètres de base](./media/azure-to-azure-how-to-enable-zone-to-zone-disaster-recovery/zonal-disaster-recovery-basic-settings-blade.png)

4. Si vous acceptez toutes les valeurs par défaut, cliquez sur « Examiner + démarrer la réplication », puis sur « Démarrer la réplication ».

5. Si vous souhaitez apporter des modifications aux paramètres de réplication, cliquez sur « Suivant : Paramètres avancés ».

6. Modifiez les paramètres par défaut partout où c’est approprié. Cette page peut paraître familière aux utilisateurs de la récupération d’urgence d’Azure vers Azure. Vous trouverez plus d’informations sur les options présentées dans ce panneau [ici](./azure-to-azure-tutorial-enable-replication.md)

    ![Page Paramètres avancés](./media/azure-to-azure-how-to-enable-zone-to-zone-disaster-recovery/zonal-disaster-recovery-advanced-settings-blade.png)

7. Cliquez sur « Suivant : Examiner + démarrer la réplication », puis sur « Démarrer la réplication ».

## <a name="faqs"></a>FAQ

**1. Comment fonctionne la tarification de la récupération d’urgence de zone à zone ?**
La tarification de la récupération d’urgence de zone en zone est identique à la tarification de la récupération d’urgence d’Azure vers Azure. Vous trouverez plus d’informations sur la page de tarification [ici](https://azure.microsoft.com/pricing/details/site-recovery/) et [ici](https://azure.microsoft.com/blog/know-exactly-how-much-it-will-cost-for-enabling-dr-to-your-azure-vm/). Notez que les frais de sortie qui pourraient être associés à une récupération d’urgence de zone à zone seraient inférieurs à ceux associés à une récupération d’urgence de région à région.

**2. Quel est le Contrat de niveau de service (SLA) pour le RTO et le RPO ?**
Le SLA pour le RTO est le même que pour Site Recovery dans son ensemble. Nous mettons en place un RTO pouvant atteindre 2 heures. Il n’existe aucun SLA défini pour le RPO.

**3. La capacité est-elle garantie dans la zone secondaire ?**
L’équipe Site Recovery et l’équipe de gestion de la capacité Azure planifient une capacité d’infrastructure suffisante. Lorsque vous démarrez un basculement, les équipes vous aident également à vous assurer que les instances de machine virtuelle protégées par Site Recovery se déploient dans la zone cible.

**4. Quels systèmes d’exploitation sont pris en charge ?**
La récupération d’urgence de zone à zone prend en charge les mêmes systèmes d’exploitation que la récupération d’urgence d’Azure vers Azure. Reportez-vous au tableau de prise en charge [ici](./azure-to-azure-support-matrix.md).

**5. Les groupes de ressources source et cible peuvent-ils être identiques ?**
Non, vous devez basculer vers un autre groupe de ressources.

## <a name="next-steps"></a>Étapes suivantes

La procédure à suivre pour exécuter une simulation de récupération d’urgence, un basculement, une reprotection et une restauration automatique est la même que dans le scénario de récupération d’urgence d’Azure vers Azure.

Pour effectuer une simulation de récupération d’urgence, procédez de la manière décrite [ici](./azure-to-azure-tutorial-dr-drill.md).

Pour opérer un basculement et reprotéger les machines virtuelles dans la zone secondaire, procédez de la manière décrite [ici](./azure-to-azure-tutorial-failover-failback.md).

Pour effectuer une restauration automatique vers la zone principale, procédez de la manière décrite [ici](./azure-to-azure-tutorial-failback.md).
