---
title: Questions courantes sur la récupération d’urgence de machine virtuelle Azure avec Azure Site Recovery
description: Cet article répond aux questions courantes sur la récupération d’urgence de machine virtuelle Azure avec Azure Site Recovery.
author: sideeksh
manager: rochakm
ms.date: 11/03/2019
ms.topic: conceptual
ms.openlocfilehash: d4fa7348df647cf699ebd55dd6415a79454ab5f1
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397948"
---
# <a name="common-questions-azure-to-azure-disaster-recovery"></a>Questions courantes : Récupération d'urgence d'Azure vers Azure

Cet article répond aux questions courantes concernant la récupération d’urgence de machines virtuelles Azure dans une autre région Azure avec le service [Azure Site Recovery](site-recovery-overview.md).

## <a name="general"></a>Général

### <a name="how-is-site-recovery-priced"></a>Comment les tarifs Azure Site Recovery sont-ils fixés ?

En savoir plus sur les [coûts](https://azure.microsoft.com/blog/know-exactly-how-much-it-will-cost-for-enabling-dr-to-your-azure-vm/) de la récupération d’urgence des machines virtuelles Azure.

### <a name="how-does-the-free-tier-work"></a>Comment fonctionne le niveau gratuit ?

Chaque instance protégée par Site Recovery est gratuite pendant les 31 premiers jours de protection. Au terme de cette période, la protection de chaque instance est facturée aux tarifs résumés dans [les détails de la tarification](https://azure.microsoft.com/pricing/details/site-recovery/). Vous pouvez estimer les coûts à l’aide de la [calculatrice de tarification Azure](https://azure.microsoft.com/pricing/calculator/?service=site-recovery).

### <a name="do-i-incur-other-azure-charges-in-the-first-31-days"></a>Des frais Azure sont-ils facturés au cours des 31 premiers jours ?

Oui. Oui, bien qu’Azure Site Recovery soit gratuit pendant les 31 premiers jours d’une instance protégée, des frais peuvent s’appliquer pour Stockage Azure, les transactions de stockage et les transferts de données. Des frais de calcul Azure peuvent également être facturés pour une machine virtuelle récupérée. G

### <a name="how-do-i-get-started-with-azure-vm-disaster-recovery"></a>Comment prendre en main la récupération d’urgence de machines virtuelles Azure ?

1. [Comprendre](azure-to-azure-architecture.md) l’architecture de récupération d’urgence des machines virtuelles Azure.
2. [Examiner](azure-to-azure-support-matrix.md) les conditions de prise en charge.
3. [Configurer](azure-to-azure-how-to-enable-replication.md) la récupération d’urgence pour les machines virtuelles Azure.
4. [Exécuter un test de récupération d’urgence](azure-to-azure-tutorial-dr-drill.md) avec un test de basculement.
5. [Exécuter un basculement complet](azure-to-azure-tutorial-failover-failback.md) vers une région Azure secondaire.
6. [Effectuer une restauration automatique](azure-to-azure-tutorial-failback.md) de la région secondaire vers la région primaire.

### <a name="how-do-we-ensure-capacity-in-the-target-region"></a>Comment garantir la capacité dans la région cible ?

L’équipe Site Recovery et l’équipe de gestion de la capacité Azure planifient une capacité d’infrastructure suffisante. Lorsque vous démarrez un basculement, les équipes vous aident également à vous assurer que les instances de machine virtuelle protégées par Site Recovery se déploient dans la région cible.

## <a name="replication"></a>Réplication

### <a name="can-i-replicate-vms-with-disk-encryption"></a>Puis-je répliquer des machines virtuelles avec le chiffrement de disque ?

Oui. Site Recovery prend en charge la récupération d’urgence des machines virtuelles pour lesquelles le chiffrement Azure Disk Encryption (ADE) est activé. Lorsque vous activez la réplication, Azure copie l’ensemble des clés de chiffrement de disque et des secrets requis de la région source vers la région cible dans le contexte utilisateur. Si vous ne disposez pas des autorisations requises, votre administrateur de la sécurité peut utiliser un script pour copier les clés et les secrets.

- Site Recovery prend en charge ADE pour les machines virtuelles Azure exécutant Windows.
- Site Recovery prend en charge :
    - ADE version 0.1 dont le schéma requiert Azure Active Directory (Azure AD).
    - ADE version 1.1, qui ne nécessite pas Azure AD. Pour la version  1.1, les machines virtuelles Windows Azure doivent avoir des disques managés.
    - [Plus d’informations](../virtual-machines/extensions/azure-disk-enc-windows.md#extension-schema) à propos des schémas d’extension.

[En savoir plus](azure-to-azure-how-to-enable-replication-ade-vms.md) sur l’activation de la réplication pour les machines virtuelles chiffrées.

### <a name="can-i-select-an-automation-account-from-a-different-resource-group"></a>Puis-je sélectionner le compte Automation d’un autre groupe de ressources ?

Lorsque vous autorisez Site Recovery à gérer les mises à jour de l’extension de service Mobility fonctionnant sur des machines virtuelles Azure répliquées, il déploie un runbook global (utilisé par les services Azure), via un compte Azure Automation. Vous pouvez utiliser le compte Automation que Site Recovery crée ou choisir d’utiliser un compte Automation existant. 

Actuellement, dans le portail, vous pouvez uniquement sélectionner un compte Automation dans le même groupe de ressources que le coffre. Vous pouvez sélectionner le compte Automation d’un autre groupe de ressources à l’aide de PowerShell. [Plus d’informations](azure-to-azure-autoupdate.md#enable-automatic-updates)

### <a name="if-i-use-a-customer-automation-account-thats-not-in-the-vault-resource-group-can-i-delete-the-default-runbook"></a>Si j’utilise un compte Automation client qui ne se trouve pas dans le groupe de ressources du coffre, puis-je supprimer le runbook par défaut ?

Oui, vous pouvez le supprimer si vous n’en avez pas besoin. 

### <a name="can-i-replicate-vms-to-another-subscription"></a>Puis-je répliquer des machines virtuelles vers un autre abonnement ?

Oui, vous pouvez répliquer des machines virtuelles Azure vers n’importe quel abonnement au sein du même locataire Azure AD. Lorsque vous activez la récupération d’urgence pour les machines virtuelles, l’abonnement cible indiqué par défaut est celui de la machine virtuelle source. Vous pouvez modifier l’abonnement cible, et les autres paramètres (tels que le groupe de ressources et le réseau virtuel) sont renseignés automatiquement à partir de l’abonnement sélectionné.

### <a name="can-i-replicate-vms-in-an-availability-zone-to-another-region"></a>Puis-je répliquer des machines virtuelles d’une zone de disponibilité dans une autre région ?

Oui, vous pouvez répliquer des machines virtuelles de zones de disponibilité vers une autre région Azure. La machine virtuelle cible peut être déployée en tant qu’instance unique, dans un groupe à haute disponibilité ou dans une zone de disponibilité si elle est prise en charge dans la région cible. 

### <a name="can-i-replicate-non-zone-vms-to-a-zone-within-the-same-region"></a>Puis-je répliquer des machines virtuelles non zones dans une zone de la même région ? 

Cela n’est pas pris en charge dans le portail. Pour ce faire, vous pouvez utiliser l’API REST ou PowerShell.

### <a name="can-i-replicate-zoned-vms-to-a-different-zone-in-the-same-region"></a>Puis-je répliquer des machines virtuelles zonées dans une autre zone de la même région ?

La prise en charge est limitée à quelques régions. [Plus d’informations](azure-to-azure-how-to-enable-zone-to-zone-disaster-recovery.md)

### <a name="can-i-exclude-disks-from-replication"></a>Puis-je exclure des disques de la réplication ?

Oui, vous pouvez exclure des disques lorsque vous configurez la réplication à l’aide de PowerShell. [Plus d’informations](azure-to-azure-exclude-disks.md)

### <a name="can-i-replicate-new-disks-added-to-replicated-vms"></a>Puis-je répliquer les nouveaux disques ajoutés aux machines virtuelles répliquées ?

Pour les machines virtuelles répliquées avec des disques managés, vous pouvez ajouter de nouveaux disques et activer la réplication. Lorsque vous ajoutez un nouveau disque, la machine virtuelle répliquée affiche un message d’avertissement indiquant qu’un ou plusieurs disques sur la machine virtuelle peuvent être protégés. 

- Si vous activez la réplication pour les disques ajoutés, l’avertissement disparaît après la réplication initiale.
- Si vous ne souhaitez pas activer la réplication pour le disque, vous pouvez ignorer l’avertissement.
- Si vous basculez une machine virtuelle avec des disques ajoutés, les points de réplication affichent les disques disponibles pour la récupération. Par exemple, si vous ajoutez un deuxième disque à une machine virtuelle avec un seul disque, un point de réplication créé avant l’ajout indique « 1 sur 2 disques ».

Site Recovery ne prend pas en charge le « retrait à chaud » des disques d’une machine virtuelle répliquée. Si vous retirez un disque d’une machine virtuelle, vous devez désactiver puis réactiver la réplication pour la machine virtuelle.

### <a name="how-often-can-i-replicate-to-azure"></a>À quelle fréquence puis-je répliquer vers Azure ?

La réplication de machines virtuelles Azure vers une autre région Azure est continue. [En savoir plus](./azure-to-azure-architecture.md#replication-process) sur le fonctionnement de la réplication.

### <a name="can-i-replicate-virtual-machines-within-a-region"></a>Puis-je répliquer des machines virtuelles au sein d’une même région ? 

Vous ne pouvez pas utiliser Site Recovery pour répliquer des disques dans une région.

### <a name="can-i-replicate-vm-instances-to-any-azure-region"></a>Puis-je répliquer des instances de machine virtuelle vers n’importe quelle Azure ?

Vous pouvez répliquer et restaurer des machines virtuelles entre deux régions appartenant au même cluster géographique. Les clusters géographiques sont définis en gardant à l’esprit la souveraineté et la latence des données. [En savoir plus](./azure-to-azure-support-matrix.md#region-support) sur la prise en charge des régions.

### <a name="does-site-recovery-need-internet-connectivity"></a>Site Recovery nécessite-t-il une connexion Internet ?

Non, mais les machines virtuelles ont besoin d’accéder aux URL et aux plages d’adresses IP de Site Recovery. [Plus d’informations](./azure-to-azure-about-networking.md#outbound-connectivity-for-urls)

### <a name="can-i-replicate-an-application-tiered-across-resource-groups"></a>Puis-je répliquer une application hiérarchisée sur différents groupes de ressources ?

Oui, vous pouvez répliquer l’application et conserver la configuration de récupération d’urgence dans un groupe de ressources distinct.

Par exemple, si les applications ont trois niveaux (application/base de données/web) dans différents groupes de ressources, vous devez activer la réplication trois fois pour protéger tous les niveaux. Site Recovery réplique les trois niveaux dans trois groupes de ressources différents.

### <a name="can-i-move-storage-accounts-across-resource-groups"></a>Puis-je déplacer les comptes de stockage entre les groupes de ressources ?

Non, ce n’est pas pris en charge. Si vous déplacez accidentellement des comptes de stockage vers un autre groupe de ressources et que vous supprimez le groupe de ressources d’origine, vous pouvez créer un nouveau groupe de ressources portant le même nom que l’ancien groupe de ressources, puis déplacer le compte de stockage vers celui-ci.

## <a name="replication-policy"></a>Stratégie de réplication

### <a name="what-is-a-replication-policy"></a>Qu’est-ce qu’une stratégie de réplication ?

Une stratégie de réplication définit l’historique de conservation des points de récupération et la fréquence des captures instantanées cohérentes au niveau application.  Site Recovery crée une stratégie de réplication par défaut comme suit :

- Conserver les points de récupération pendant 24 heures.
- Effectuer des captures instantanées cohérentes au niveau application toutes les quatre heures.

[En savoir plus](azure-to-azure-how-to-enable-replication.md#customize-target-resources) sur les paramètres de réplication.

### <a name="whats-a-crash-consistent-recovery-point"></a>Qu’est-ce qu’un point de récupération cohérent en cas d’incident ?

Un point de récupération cohérent en cas d’incident contient les données sur disque comme si vous aviez débranché le cordon d’alimentation du serveur lors de la capture instantanée. Il n’inclut rien de ce qui était en mémoire lors de la prise de l’instantané.

Aujourd’hui, la plupart des applications peuvent récupérer correctement à partir d’instantanés cohérents en cas d’incident. Un point de récupération cohérent en cas d’incident ne suffit généralement pas pour des systèmes d’exploitation de base de données et des applications telles que des serveurs de fichiers, des serveurs DHCP et des serveurs d’impression.

Site Recovery crée automatiquement un point de récupération cohérent en cas d’incident toutes les cinq minutes.

### <a name="whats-an-application-consistent-recovery-point"></a>Qu’est-ce qu’un point de récupération cohérent au niveau application ?

Les points de récupération de cohérence des applications sont créés à partir d’instantanés de cohérence des applications. Ils capturent les mêmes données que des instantanés cohérents en cas d’incident et saisissent en outre toutes les données en mémoire et toutes les transactions en cours.

En raison de leur contenu supplémentaire, les instantanés cohérents au niveau application sont davantage sollicités et prennent le plus de temps. Les points de récupération cohérent au niveau application sont recommandés pour des systèmes d’exploitation de base de données et des applications telles que SQL Server. Pour Windows, les instantanés cohérents au niveau application utilisent le service VSS :

### <a name="do-app-consistent-recovery-points-impact-performance"></a>Les points de récupération cohérents au niveau application ont-ils un impact sur les performances ?

 Étant donné que les points de récupération cohérents au niveau application capturent toutes les données en mémoire et les traitent, si celles-ci sont capturées fréquemment, cela peut affecter les performances lorsque la charge de travail est déjà occupée. Nous vous déconseillons de capturer trop souvent les charges de travail autres que celles de base de données. Même pour les charges de travail de base de données, une heure devrait suffire.

### <a name="whats-the-minimum-frequency-for-generating-app-consistent-recovery-points"></a>Quelle est la fréquence minimale pour générer des points de récupération cohérents au niveau application ?

Site Recovery peut créer des points de récupération cohérents au niveau application à une fréquence minimale d’une heure.

### <a name="can-i-enable-app-consistent-replication-for-linux-vms"></a>Puis-je activer la réplication cohérente au niveau application pour des machines virtuelles Linux ?

Oui. L’agent Mobility pour Linux prend en charge des scripts personnalisés pour la cohérence au niveau application. Un script personnalisé avec des options préalables et postérieures est utilisé par l’agent. [En savoir plus](site-recovery-faq.md#can-i-enable-replication-with-app-consistency-in-linux-servers)

### <a name="how-are-recovery-points-generated-and-saved"></a>Comment les points de récupération sont-ils générés et enregistrés ?

Pour comprendre comment Site Recovery génère des points de récupération, utilisons un exemple. 

- Une stratégie de réplication conserve les points de récupération pendant 24 heures et prend un instantané de fréquence cohérent au niveau application toutes les heures.
- Site Recovery crée un point de récupération cohérent en cas d’incident toutes les cinq minutes. Vous ne pouvez pas modifier cette fréquence.
- Site Recovery élague les points de récupération après une heure, en enregistrant un point par heure.

Ainsi, au cours de la dernière heure, vous pouvez choisir parmi 12 points cohérents en cas d’incident et un point cohérent au niveau application, comme illustré dans le graphique.

   ![Liste des points de récupération générés](./media/azure-to-azure-common-questions/recovery-points.png)

### <a name="how-far-back-can-i-recover"></a>Jusqu’à quand peut remonter la récupération ?

Le point de récupération le plus ancien que vous pouvez utiliser remonte à 72 heures.

### <a name="what-happens-if-site-recovery-cant-generate-recovery-points-for-more-than-24-hours"></a>Que se passe-t-il si Site Recovery ne parvient pas à générer des points de récupération pendant plus de 24 heures ? 

Si vous avez une stratégie de réplication de 24 heures et que Site Recovery ne peut pas générer de points de récupération pendant plus de 24 heures, vos anciens points de récupération sont conservés. Site Recovery remplace le point le plus ancien uniquement s’il génère de nouveaux points. Tant qu’il n’y a pas de nouveaux points de récupération, tous les anciens points subsistent une fois la fenêtre de rétention atteinte.

### <a name="can-i-change-the-replication-policy-after-replication-is-enabled"></a>Puis-je modifier la stratégie de réplication après l’activation de la réplication ?

Oui. Dans le coffre > **Infrastructure Site Recovery** > **Stratégies de réplication**, sélectionnez et modifiez la stratégie. Les modifications s’appliquent également aux stratégies existantes.

### <a name="are-all-recovery-points-a-complete-vm-copy"></a>Les points de récupération sont-ils tous une copie complète de la machine virtuelle ?

Le premier point de récupération qui est généré possède la copie complète. Les points de récupération successifs ont des modifications d’ordre différentiel.

### <a name="do-increases-in-recovery-point-retention-increase-storage-costs"></a>L’augmentation de la rétention des points de récupération augmente-t-elle les coûts de stockage ?

Oui. Par exemple, si vous augmentez la durée de rétention de 24 à 72 heures, Site Recovery enregistre des points de récupération pendant 48 heures supplémentaires. Le temps ajouté entraîne des modifications de stockage. À titre d’exemple uniquement, si un point de récupération unique a des modifications d’ordre différentiel de 10 Go, avec un coût par Go de 0,16 $ par mois, les frais supplémentaires seraient de 1,60 $ × 48 par mois.

## <a name="multi-vm-consistency"></a>Cohérence multimachine virtuelle

### <a name="what-is-multi-vm-consistency"></a>Définition de la cohérence multimachine virtuelle

La cohérence multimachine virtuelle veille à ce que les points de récupération soient cohérents entre les machines virtuelles répliquées.

- Lorsque vous activez la cohérence multimachine virtuelle, Site Recovery crée un groupe de réplication de toutes les machines avec l’option activée. 
- Lorsque vous basculez les machines dans le groupe de réplication, elles ont des points de récupération cohérents en cas d’incident et au niveau application.

[En savoir plus](azure-to-azure-tutorial-enable-replication.md#enable-replication) sur la façon d’activer la cohérence multimachine virtuelle.

### <a name="can-i-fail-over-a-single-vm-in-a-replication-group"></a>Puis-je basculer une seule machine virtuelle dans un groupe de réplication ?

Non. Lorsque vous activez la cohérence multimachine virtuelle, cela signifie qu’une application dépend de toutes les machines virtuelles du groupe de réplication, et que le basculement d’une machine virtuelle unique n’est pas autorisé.

### <a name="how-many-vm-can-i-replicate-together-in-a-group"></a>Combien de machines virtuelles puis-je répliquer ensemble dans un groupe ?

Vous pouvez répliquer 16 machines virtuelles ensemble dans un groupe de réplication.

### <a name="when-should-i-enable-multi-vm-consistency"></a>Quand dois-je activer la cohérence multimachine virtuelle ?

La cohérence multimachine virtuelle est gourmande en ressources processeur et son activation peut avoir un impact sur les performances de la charge de travail. Activez-la uniquement si les machines virtuelles exécutent la même charge de travail et si vous avez besoin de cohérence sur plusieurs machines. Par exemple, si vous avez deux instances SQL et deux serveurs web dans une application, activez la cohérence multimachine virtuelle uniquement pour les instances SQL.

### <a name="can-i-add-a-replicating-vm-to-a-replication-group"></a>Puis-je ajouter une machine virtuelle de réplication à un groupe de réplication ?

Lorsque vous activez la réplication pour une machine virtuelle, vous pouvez l’ajouter à un nouveau groupe de réplication ou à un groupe existant. Vous ne pouvez pas ajouter une machine virtuelle qui est déjà en cours de réplication à un groupe. 
 
## <a name="failover"></a>Basculement

### <a name="how-do-we-ensure-capacity-in-the-target-region"></a>Comment garantir la capacité dans la région cible ?

L’équipe Site Recovery et l’équipe de gestion de la capacité Azure planifient une capacité d’infrastructure suffisante. Lorsque vous démarrez un basculement, les équipes vous aident également à vous assurer que les instances de machine virtuelle protégées par Site Recovery peuvent être déployées dans la région cible.

### <a name="is-failover-automatic"></a>Le basculement est-il automatique ?

Le basculement n’est pas automatique. Vous pouvez démarrer un basculement d’un simple clic dans le portail ou en utilisant [PowerShell](azure-to-azure-powershell.md).

### <a name="can-i-keep-a-public-ip-address-after--failover"></a>Puis-je conserver une IP publique après un basculement ?

Vous pouvez conserver l’IP publique de l’application de production après un basculement.

Lorsque vous impliquez une charge de travail dans le processus de basculement, vous devez lui attribuer une ressource IP publique Azure. La ressource doit être disponible dans la région cible. Vous pouvez attribuer la ressource IP publique Azure manuellement, ou vous pouvez automatiser cette opération avec un plan de récupération. [En savoir plus ](concepts-public-ip-address-with-site-recovery.md#public-ip-address-assignment-using-recovery-plan) sur la configuration des IP publiques après le basculement.

### <a name="can-i-keep-a-private-ip-address-after-failover"></a>Puis-je conserver une adresse IP privée après un basculement ?

Oui. Par défaut, quand vous activez la récupération d’urgence pour les machines virtuelles Azure, Site Recovery crée des ressources cibles en fonction des paramètres de ressources sources. Pour les machines virtuelles Azure configurées avec des adresses IP statiques, Site Recovery tente de provisionner la même adresse IP pour la machine virtuelle cible, si elle n’est pas en cours d’utilisation.
[En savoir plus](site-recovery-retain-ip-azure-vm-failover.md) sur la conservation des adresses IP après le basculement.

### <a name="why-is-a-vm-assigned-a-new-ip-address-after-failover"></a>Pourquoi une machine virtuelle reçoit-elle une nouvelle adresse IP après le basculement ?

Site Recovery tente de fournir l’adresse IP au moment du basculement. Si une autre machine virtuelle utilise cette adresse, Site Recovery définit l’adresse IP suivante disponible comme cible.

[En savoir plus](azure-to-azure-network-mapping.md#set-up-ip-addressing-for-target-vms) sur la configuration du mappage réseau et de l’adressage IP pour les réseaux virtuels.

### <a name="whats-the-latest-recovery-point"></a>Quel est le point de récupération *le plus récent* ?

Le point de récupération *le plus récent (RPO le plus bas)* permet ce qui suit :

1. Il traite tout d’abord toutes les données qui ont été envoyées à Site Recovery.
2. Une fois que le service a traité les données, il crée un point de récupération pour chaque machine virtuelle avant de basculer vers celle-ci. Cette option fournit l’objectif de point de récupération (RPO) le plus bas.
3. La machine virtuelle créée après le basculement comprend toutes les données répliquées sur le service Site Recovery depuis le déclenchement du basculement.

### <a name="do-latest-recovery-points-impact-failover-rto"></a>Les points de récupération *les plus récents* ont-ils un impact sur le RTO de basculement ?

Oui. Site Recovery traitant toutes les données en attente avant le basculement, cette option a un objectif de délai de récupération (RTO) supérieur à celui d’autres options.

### <a name="whats-the-latest-processed-recovery-option"></a>Quelle est l’option de récupération *Dernier traitement* ?

L’option *Dernier traitement* effectue ce qui suit :

1. Elle bascule toutes les machines virtuelles vers le dernier point de récupération traité par Site Recovery. Cette option fournit un objectif de délai de récupération faible, car aucun temps n’est consacré à traiter les données non traitées.

### <a name="what-if-theres-an-unexpected-outage-in-the-primary-region"></a>Que se passe-t-il en cas de panne inattendue dans la région primaire ?

Vous pouvez démarrer le basculement. Site Recovery n’a pas besoin de connexion à partir de la région primaire pour opérer le basculement.

### <a name="what-is-the-rto-of-a-vm-failover"></a>Quel est le RTO d’un basculement de machine virtuelle ?

Site Recovery à un RTO de [deux heures](https://azure.microsoft.com/support/legal/sla/site-recovery/v1_2/) en vertu du contrat de niveau de service. La plupart du temps, Site Recovery bascule les machines virtuelles en quelques minutes. Pour calculer le RTO, examinez le travail de basculement, qui indique le temps nécessaire à l’affichage d’une machine virtuelle. 

## <a name="recovery-plans"></a>Plans de récupération

### <a name="whats-a-recovery-plan"></a>Qu’est-ce qu’un plan de récupération ?

Un [plan de récupération](site-recovery-create-recovery-plans.md) dans Site Recovery orchestre le basculement et la récupération des machines virtuelles. Il aide à rendre la récupération cohérente, reproductible et automatisée. Elle effectue les opérations suivantes :

- Définit un groupe de machines virtuelles qui basculent ensemble.
- Définit les dépendances entre les machines virtuelles afin que l’application opère avec précision.
- Automatise la récupération avec l’option d’actions manuelles personnalisées pour les tâches autres que le basculement de machine virtuelle. 


### <a name="how-does-sequencing-work"></a>Comment fonctionne le séquencement ?

Dans un plan de récupération, vous pouvez créer plusieurs groupes de machines virtuelles pour le séquencement. Les groupes basculent un par un, afin que les machines virtuelles qui font partie du même groupe basculent ensemble. [Plus d’informations](recovery-plan-overview.md#model-apps)

### <a name="how-can-i-find-the-rto-of-a-recovery-plan"></a>Comment trouver l’objectif de délai de récupération d’un plan de récupération ?

Pour vérifier le RTO d’un plan de récupération, testez le basculement du plan de récupération. Dans **Travaux Site Recovery**, vérifiez la durée du test de basculement. Dans l’exemple de capture d’écran, le travail de test de basculement **SAPTestRecoveryPlan** a duré 8 minutes et 59 secondes.

![Liste des travaux indiquant la durée du test de basculement pour le RTO](./media/azure-to-azure-common-questions/recovery-plan-rto.png)

### <a name="can-i-add-automation-runbooks-to-recovery-plans"></a>Puis-je ajouter des runbooks Automation à des plans de récupération ?

Oui. [Plus d’informations](site-recovery-runbook-automation.md)

## <a name="reprotection-and-failback"></a>Reprotection et restauration automatique

### <a name="after-failover-are-vms-in-the-secondary-region-protected-automatically"></a>Après le basculement, les machines virtuelles de la région secondaire sont-elles protégées automatiquement ? 

Non. Lorsque vous basculez des machines virtuelles d’une région vers une autre, les machines virtuelles démarrent dans la région de récupération d’urgence cible dans un état non protégé. Pour [reprotéger](./azure-to-azure-how-to-reprotect.md) les machines virtuelles dans la région secondaire, réactivez la réplication vers la région primaire.

### <a name="when-i-reprotect-is-all-data-replicated-from-the-secondary-region-to-primary"></a>Lorsque je les reprotège, toutes les données sont-elles répliquées de la région secondaire vers la région primaire ? 

Cela dépend. Si la machine virtuelle de la région source existe, seules les différences entre le disque source et le disque cible sont synchronisées. Site Recovery relève les différences en comparant les disques, puis transfère les données. Ce processus prend généralement plusieurs heures. [Plus d’informations](azure-to-azure-how-to-reprotect.md#what-happens-during-reprotection)

### <a name="how-long-does-it-take-fail-back"></a>Combien de temps faut-il pour effectuer une restauration automatique ?

Une fois la reprotection effectuée, la restauration automatique prend à peu près le même temps que le basculement de la région primaire vers une région secondaire.

## <a name="capacity"></a><a name="capacity"></a>Capacité

### <a name="how-do-we-ensure-capacity-in-the-target-region"></a>Comment garantir la capacité dans la région cible ?

L’équipe Site Recovery et l’équipe de gestion de la capacité Azure planifient une capacité d’infrastructure suffisante. Lorsque vous démarrez un basculement, les équipes vous aident également à vous assurer que les instances de machine virtuelle protégées par Site Recovery peuvent être déployées dans la région cible.

### <a name="does-site-recovery-work-with-reserved-instances"></a>Site Recovery fonctionne-t-il avec les instances réservées ?

Oui, vous pouvez acheter des [machines virtuelles Azure réservées](https://azure.microsoft.com/pricing/reserved-vm-instances/) dans la région de récupération d’urgence, que les opérations de basculement Site Recovery utiliseront. Aucune configuration supplémentaire n’est nécessaire.

## <a name="security"></a>Sécurité

### <a name="is-replication-data-sent-to-the-site-recovery-service"></a>Les données de réplication sont-elles envoyées vers le service Site Recovery ?

Non, le service Site Recovery n’intercepte pas de données répliquées et ne dispose d’aucune information sur ce qu’exécutent vos machines virtuelles. Seules les métadonnées nécessaires pour coordonner la réplication et le basculement sont envoyées au service Site Recovery.

Le service Site Recovery est certifié ISO 27001:2013, 27018, HIPAA et DPA. Il fait actuellement l’objet des évaluations SOC2 et JAB FedRAMP sont en cours.

### <a name="does-site-recovery-encrypt-replication"></a>Site Recovery chiffre-t-il la réplication ?

Oui, le chiffrement en transit et le [chiffrement au repos dans Azure](../storage/common/storage-service-encryption.md) sont tous deux pris en charge.

## <a name="next-steps"></a>Étapes suivantes

- [Consultez les conditions requises pour la prise en charge d’Azure vers Azure](azure-to-azure-support-matrix.md).
- [Configurer la réplication d’Azure vers Azure](azure-to-azure-tutorial-enable-replication.md).
- Si, après avoir lu cet article, vous avez des questions, posez-les sur la [page de questions Microsoft Q&A pour Azure Recovery Services](/answers/topics/azure-site-recovery.html).
