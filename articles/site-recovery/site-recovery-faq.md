---
title: Questions générales sur le service Azure Site Recovery
description: Cet article traite des questions générales fréquemment posées sur Azure Site Recovery.
ms.topic: conceptual
ms.date: 7/14/2020
ms.author: raynew
ms.openlocfilehash: 3da86eead5b927a2a71d7b1a28bc5966bf5f8840
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92369435"
---
# <a name="general-questions-about-azure-site-recovery"></a>Questions générales sur Azure Site Recovery

Cet article résume les questions fréquentes sur Azure Site Recovery. Pour des scénarios spécifiques, veuillez consulter les articles suivants

- [Questions sur la récupération d’urgence de machine virtuelle Azure vers Azure](azure-to-azure-common-questions.md)
- [Questions sur la récupération d’urgence de machine virtuelle VMware vers Azure](vmware-azure-common-questions.md)
- [Questions sur la récupération d’urgence de machine virtuelle Hyper-V vers Azure](hyper-v-azure-common-questions.md)
 
## <a name="general"></a>Général

### <a name="what-does-site-recovery-do"></a>À quoi sert Site Recovery ?

Site Recovery contribue à mettre en œuvre la stratégie de continuité d’activité et de récupération d’urgence (BCDR) de votre entreprise en coordonnant et en automatisant la réplication de machines virtuelles Azure entre des régions, de machines virtuelles et serveurs physiques locaux sur Azure et de machines locales sur un centre de données secondaire. [Plus d’informations](site-recovery-overview.md)

### <a name="can-i-protect-a-virtual-machine-that-has-a-docker-disk"></a>Puis-je protéger une machine virtuelle dotée d’un disque Docker ?

Non, ce scénario n’est pas pris en charge.

### <a name="what-does-site-recovery-do-to-ensure-data-integrity"></a>Que fait Site Recovery pour garantir l'intégrité des données ?

Site Recovery prend différentes mesures pour garantir l'intégrité des données. Une connexion sécurisée est établie entre tous les services à l'aide du protocole HTTPS. Vous êtes ainsi assuré qu'aucun programme malveillant ou aucune entité extérieure n'altérera les données. Une autre mesure consiste à utiliser des sommes de contrôle. Le transfert de données entre la source et la cible est exécuté en calculant les sommes de contrôle des données entre elles. Cela permet de garantir la cohérence des données transférées.

## <a name="service-providers"></a>Fournisseurs de services

### <a name="im-a-service-provider-does-site-recovery-work-for-dedicated-and-shared-infrastructure-models"></a>Je suis un fournisseur de services. Site Recovery fonctionne-t-il pour les modèles d’infrastructure dédiée ou partagée ?
Oui, Site Recovery prend en charge les modèles d’infrastructure dédiée et partagée.

### <a name="for-a-service-provider-is-the-identity-of-my-tenant-shared-with-the-site-recovery-service"></a>Pour un fournisseur de services, l’identité de mon locataire est-elle communiquée au service Site Recovery ?
Non. L’identité du locataire reste anonyme. Vos locataires n’ont pas besoin d’accéder au portail Site Recovery. Seul l’administrateur du fournisseur de services interagit avec le portail.

### <a name="will-tenant-application-data-ever-go-to-azure"></a>Les données d’application de mes locataires sont-elles diffusées sur Azure ?
Lors de la réplication entre des sites appartenant à un fournisseur de services, les données d’application n’accèdent jamais à Azure. Les données sont chiffrées en transit et répliquées directement entre les sites du fournisseur de services.

Si vous répliquez vers Azure, les données d’application sont envoyées vers le stockage Azure, mais pas vers le service Site Recovery. Les données sont chiffrées en transit et restent chiffrées dans Azure.

### <a name="will-my-tenants-receive-a-bill-for-any-azure-services"></a>Mes clients recevront-ils une facture pour les services Azure ?
Non. La relation de facturation d’Azure concerne directement le fournisseur de services. Les fournisseurs de services sont responsables de la création de factures spécifiques pour leurs locataires.

### <a name="if-im-replicating-to-azure-do-we-need-to-run-virtual-machines-in-azure-at-all-times"></a>Si je réplique vers Azure, faut-il exécuter à tout moment des machines virtuelles dans ce dernier ?
Non, les données sont répliquées vers le stockage Azure de votre abonnement. Lorsque vous effectuez un basculement de test (test de récupération d’urgence) ou un basculement réel, Site Recovery crée automatiquement les machines virtuelles dans votre abonnement.

### <a name="do-you-ensure-tenant-level-isolation-when-i-replicate-to-azure"></a>Assurez-vous l’isolation au niveau des clients lors de la réplication vers Azure ?
Oui.

### <a name="what-platforms-do-you-currently-support"></a>Quelles plates-formes prenez-vous en charge, actuellement ?
Nous prenons en charge Azure Pack et le système Cloud Platform, ainsi que les déploiements basés sur System Center (2012 et versions supérieures). [En savoir plus](/previous-versions/azure/windows-server-azure-pack/dn850370(v=technet.10)) sur l’intégration d’Azure Pack et de Site Recovery.

### <a name="do-you-support-single-azure-pack-and-single-vmm-server-deployments"></a>Prenez-vous en charge les déploiements uniques de serveurs VMM et Azure Pack ?
Oui, vous pouvez répliquer des machines virtuelles Hyper-V vers Azure, ou entre des sites du fournisseur de service.  Notez que si vous répliquez entre des sites du fournisseur de services, l’intégration de runbooks Azure n’est pas disponible.

## <a name="pricing"></a>Tarifs

### <a name="where-can-i-find-pricing-information"></a>Où puis-je obtenir des informations sur la tarification ?
Consultez les [détails de la tarification Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/).


### <a name="how-can-i-calculate-approximate-charges-during-the-use-of-site-recovery"></a>Comment puis-je calculer les frais approximatifs d'utilisation de Site Recovery ?

Vous pouvez utiliser la [calculatrice de prix](https://aka.ms/asr_pricing_calculator) pour estimer les coûts d’utilisation de Site Recovery.

Pour obtenir une estimation détaillée des coûts, exécutez l’outil Planificateur de déploiement pour [VMware](./site-recovery-deployment-planner.md) ou [Hyper-V](https://aka.ms/asr-deployment-planner) et utilisez le [rapport d’estimation des coûts](./site-recovery-vmware-deployment-planner-cost-estimation.md).


### <a name="managed-disks-are-now-used-to-replicate-vmware-vms-and-physical-servers-do-i-incur-additional-charges-for-the-cache-storage-account-with-managed-disks"></a>Les disques managés sont à présent utilisés pour répliquer des machines virtuelles VMware et des serveurs physiques. Des frais supplémentaires sont-ils facturés pour le compte de stockage de cache avec des disques managés ?

Non, le cache n’entraîne pas la facturation de frais. Lorsque vous effectuez une réplication vers le compte de stockage standard, ce stockage de cache fait partie du même compte de stockage cible.

### <a name="i-have-been-an-azure-site-recovery-user-for-over-a-month-do-i-still-get-the-first-31-days-free-for-every-protected-instance"></a>J’utilise Azure Site Recovery depuis plus d’un mois. Puis-je tout de même bénéficier des 31 premiers jours gratuits pour chaque instance protégée ?

Oui. Chaque instance protégée n’engendre aucun frais Azure Site Recovery pendant les 31 premiers jours. Par exemple, si vous protégez 10 instances depuis 6 mois et que vous connectez une 11e instance à Azure Site Recovery, aucuns frais ne sont facturés pour cette 11e instance pendant les 31 premiers jours. Des frais Azure Site Recovery continuent d’être facturés pour les 10 premières instances, car celles-ci sont protégées depuis plus de 31 jours.

### <a name="during-the-first-31-days-will-i-incur-any-other-azure-charges"></a>Pendant les 31 premiers jours, d'autres frais Azure sont-ils facturés ?

Oui, bien qu’Azure Site Recovery soit gratuit pendant les 31 premiers jours d’une instance protégée, des frais peuvent s’appliquer pour Stockage Azure, les transactions de stockage et le transfert de données. Des frais de calcul Azure peuvent également être facturés pour une machine virtuelle récupérée.


### <a name="is-there-a-cost-associated-to-perform-disaster-recovery-drillstest-failover"></a>L’exécution de simulations de reprise d’activité après sinistre/tests de basculement génère-t-elle des frais ?

Les simulations de reprise d'activité n'entraînent pas de coût distinct. Des frais de calcul interviennent une fois la machine virtuelle créée, après le test de basculement.



## <a name="security"></a>Sécurité

### <a name="is-replication-data-sent-to-the-site-recovery-service"></a>Les données de réplication sont-elles envoyées vers le service Site Recovery ?
Non, Site Recovery n’intercepte pas les données répliquées et n’a pas d’informations sur les éléments exécutés sur vos machines virtuelles ou serveurs physiques.
Les données de réplication sont échangées entre des hôtes Hyper-V, des hyperviseurs VMware ou des serveurs physiques locaux et le stockage Azure ou votre site secondaire. Site Recovery n’a aucun moyen d’intercepter ces données. Seules les métadonnées nécessaires pour coordonner la réplication et le basculement sont envoyées au service Site Recovery.  

Le logiciel Site Recovery est certifié conforme aux normes ISO 27001:2013, 27018, HIPAA et DPA. Il fait actuellement l’objet d’une évaluation de conformité aux exigences SOC2 et JAB FedRAMP.

### <a name="for-compliance-reasons-even-our-on-premises-metadata-must-remain-within-the-same-geographic-region-can-site-recovery-help-us"></a>Pour des raisons de conformité, même nos métadonnées locales doivent rester dans la même région géographique. Site Recovery peut-il nous aider ?
Oui. Quand vous créez un coffre Site Recovery dans une région, nous vérifions que toutes les métadonnées dont nous avons besoin pour activer et coordonner la réplication et le basculement restent au sein de cette région.

### <a name="does-site-recovery-encrypt-replication"></a>Site Recovery chiffre-t-il la réplication ?
Pour la réplication de machines virtuelles et de serveurs physiques entre des sites locaux, le chiffrement en transit est pris en charge. Pour la réplication de machines virtuelles et de serveurs physiques vers Azure, le chiffrement en transit et le [chiffrement au repos (dans Azure)](../storage/common/storage-service-encryption.md) sont tous deux pris en charge.

### <a name="does-azure-to-azure-site-recovery-use-tls-12-for-all-communications-across-microservices-of-azure"></a>Le scénario Azure vers Azure Site Recovery utilise-t-il TLS 1.2 pour toutes les communications entre les microservices d’Azure ?
Oui, le protocole TLS 1.2 est appliqué par défaut pour le scénario Azure vers Azure Site Recovery. 

### <a name="how-can-i-enforce-tls-12-on-vmware-to-azure-and-physical-server-to-azure-site-recovery-scenarios"></a>Comment appliquer TLS 1.2 dans les scénarios VMware vers Azure et serveur physique vers Azure Site Recovery ?
Les agents de mobilité installés sur les éléments répliqués communiquent avec le serveur de traitement uniquement sur TLS 1.2. Toutefois, la communication entre le serveur de configuration et Azure, et entre le serveur de processus et Azure, peut se faire sur TLS 1.1 ou 1.0. Suivez les [instructions](https://support.microsoft.com/en-us/help/3140245/update-to-enable-tls-1-1-and-tls-1-2-as-default-secure-protocols-in-wi) pour appliquer le protocole TLS 1.2 sur tous les serveurs de configuration et serveurs de processus configurés par vous-même.

### <a name="how-can-i-enforce-tls-12-on-hyperv-to-azure-site-recovery-scenarios"></a>Comment appliquer le protocole TLS 1.2 dans les scénarios HyperV vers Azure Site Recovery ?
Toutes les communications entre les microservices d’Azure Site Recovery se produisent sur le protocole TLS 1.2. Site Recovery utilise le dernier protocole TLS disponible ainsi que des fournisseurs de sécurité configurés dans le système d’exploitation. Vous devez activer explicitement le protocole TLS 1.2 dans le registre, afin que Site Recovery puisse commencer à utiliser TLS 1.2 pour la communication avec les services. 

### <a name="how-can-i-enforce-restricted-access-on-my-storage-accounts-which-are-accessed-by-site-recovery-service-for-readingwriting-replication-data"></a>Comment appliquer un accès restreint sur mes comptes de stockage, auxquels Site Recovery Service accède pour lire/écrire des données de réplication ?
Vous pouvez activer l’identité managée du coffre Recovery Services en accédant au paramètre *Identité* . Une fois le coffre inscrit auprès d’Azure Active Directory, accédez à vos comptes de stockage et attribuez les rôles suivantes au coffre :

- Comptes de stockage basés sur Resource Manager (type Standard) :
  - [Contributeur](../role-based-access-control/built-in-roles.md#contributor)
  - [Contributeur aux données Blob du stockage](../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)
- Comptes de stockage basés sur Resource Manager (type Premium) :
  - [Contributeur](../role-based-access-control/built-in-roles.md#contributor)
  - [Propriétaire des données Blob du stockage](../role-based-access-control/built-in-roles.md#storage-blob-data-owner)
- Comptes de stockage Classic :
  - [Contributeur de compte de stockage classique](../role-based-access-control/built-in-roles.md#classic-storage-account-contributor)
  - [Rôle de service d’opérateur de clé de compte de stockage classique](../role-based-access-control/built-in-roles.md#classic-storage-account-key-operator-service-role)

## <a name="disaster-recovery"></a>Récupération d'urgence

### <a name="what-can-site-recovery-protect"></a>Que peut protéger Site Recovery ?
* **Machines virtuelles Azure**  : Site Recovery peut répliquer n’importe quelle charge de travail exécutée sur une machine virtuelle Azure prise en charge.
* **Machines virtuelles Hyper-V**  : Site Recovery peut protéger toute charge de travail en cours d’exécution sur une machine virtuelle Hyper-V.
* **Serveurs physiques**  : Site Recovery peut protéger les serveurs physiques exécutant Windows ou Linux.
* **Machines virtuelles VMware**  : Site Recovery peut protéger toute charge de travail en cours d’exécution dans une machine virtuelle VMware.

### <a name="what-workloads-can-i-protect-with-site-recovery"></a>Quelles charges de travail puis-je protéger avec Site Recovery ?
Vous pouvez utiliser Site Recovery pour protéger la plupart des charges de travail en cours d’exécution sur une machine virtuelle ou un serveur physique pris(e) en charge. Site Recovery assure la prise en charge de la réplication compatible avec les applications afin qu’elles puissent être récupérées dans un état intelligent. Site Recovery s’intègre aux applications Microsoft, notamment à SharePoint, Exchange, Dynamics, SQL Server et Active Directory, et fonctionne en étroite collaboration avec les principaux fournisseurs, notamment Oracle, SAP, IBM et Red Hat. [En savoir plus](site-recovery-workload.md) sur la protection des charges de travail.

### <a name="can-i-manage-disaster-recovery-for-my-branch-offices-with-site-recovery"></a>Puis-je gérer la récupération d’urgence pour mes succursales avec Site Recovery ?
Oui. Lorsque vous utilisez Site Recovery pour coordonner la réplication et le basculement dans vos succursales, vous obtenez une orchestration unifiée et l’affichage de toutes les charges de travail de vos succursales dans un emplacement central. Vous pouvez facilement exécuter les basculements et gérer la récupération d’urgence de toutes les succursales à partir de votre siège social, sans vous rendre dans ces succursales.


### <a name="is-disaster-recovery-supported-for-azure-vms"></a>La reprise d’activité après sinistre est-elle prise en charge pour les machines virtuelles Azure ?

Oui, Site Recovery prend en charge la reprise d’activité après sinistre pour les machines virtuelles Azure entre des régions Azure. [Lisez les questions fréquentes](azure-to-azure-common-questions.md) sur la reprise d’activité après sinistre des machines virtuelles Azure. Si vous souhaitez effectuer une réplication entre deux régions Azure sur le même continent, utilisez notre offre DR Azure vers Azure. Vous n’avez pas besoin de configurer le serveur de configuration/serveur de processus ni les connexions ExpressRoute.

### <a name="is-disaster-recovery-supported-for-vmware-vms"></a>La reprise d’activité après sinistre est-elle prise en charge pour les machines virtuelles VMware ?

Oui, Site Recovery prend en charge la reprise d’activité après sinistre des machines virtuelles VMware locales. [Lisez les questions fréquentes](vmware-azure-common-questions.md) sur la reprise d’activité après sinistre des machines virtuelles VMware.

### <a name="is-disaster-recovery-supported-for-hyper-v-vms"></a>La reprise d’activité après sinistre est-elle prise en charge pour les machines virtuelles Hyper-V ?
Oui, Site Recovery prend en charge la reprise d’activité après sinistre des machines virtuelles Hyper-V locales. [Lisez les questions fréquentes](hyper-v-azure-common-questions.md) sur la reprise d’activité après sinistre des machines virtuelles Hyper-V.

### <a name="is-disaster-recovery-supported-for-physical-servers"></a>La reprise d’activité après sinistre est-elle prise en charge pour les serveurs physiques ?
Oui, Site Recovery prend en charge la reprise d’activité après sinistre des serveurs physiques locaux exécutant Windows et Linux dans Azure ou dans un site secondaire. En savoir plus sur la configuration requise pour la reprise d’activité après sinistre vers [Azure](vmware-physical-azure-support-matrix.md#replicated-machines) et [vers un site secondaire](vmware-physical-secondary-support-matrix.md#replicated-vm-support).
Notez que les serveurs physiques sont exécutés en tant que machines virtuelles dans Azure après le basculement. La restauration automatique à partir d’Azure sur un serveur physique local n’est actuellement pas prise en charge. Vous pouvez uniquement effectuer la restauration automatique d’une machine virtuelle VMware.





## <a name="replication"></a>Réplication

### <a name="can-i-replicate-over-a-site-to-site-vpn-to-azure"></a>Puis-je répliquer un VPN de site à site vers Azure ?
Azure Site Recovery réplique des données vers un compte de stockage Azure ou des disques managés, via un point de terminaison public. La réplication ne s’effectue pas via un réseau VPN de site à site. 

### <a name="why-cant-i-replicate-over-vpn"></a>Pourquoi ne puis-je pas répliquer via VPN ?

Lorsque vous répliquez vers Azure, le trafic de réplication atteint les points de terminaison publics d’un stockage Azure. Par conséquent, vous pouvez uniquement répliquer sur l’Internet public ou via ExpressRoute (Peering Microsoft ou Peering public existant).

### <a name="can-i-use-riverbed-steelheads-for-replication"></a>Puis-je utiliser Riverbed SteelHeads pour la réplication ?

Notre partenaire, Riverbed, fournit des instructions détaillées sur l’utilisation d’Azure Site Recovery. Lisez le [guide de leur solution](https://community.riverbed.com/s/article/DOC-4627).

### <a name="can-i-use-expressroute-to-replicate-virtual-machines-to-azure"></a>Puis-je utiliser ExpressRoute pour répliquer des machines virtuelles vers Azure ?
Oui, [vous pouvez utiliser ExpressRoute](concepts-expressroute-with-site-recovery.md) pour répliquer des machines virtuelles locales vers Azure.

- Azure Site Recovery réplique des données vers un stockage Azure via un point de terminaison public. Vous devez configurer le [peering Microsoft](../expressroute/expressroute-circuit-peerings.md#microsoftpeering) ou utiliser un [peering public existant](../expressroute/about-public-peering.md) (déconseillé pour de nouveaux circuits) afin d’utiliser ExpressRoute pour la réplication Site Recovery.
- Le peering Microsoft est le domaine de routage recommandé pour la réplication.
- La réplication n’est pas pris en charge via un peering privé.
- Si vous protégez des machines VMware ou des machines physiques, vérifiez que les [exigences de mise en réseau](vmware-azure-configuration-server-requirements.md#network-requirements) pour le serveur de configuration sont également remplies. La connectivité à des URL spécifiques est requise par le serveur de configuration pour l’orchestration de la réplication Site Recovery. ExpressRoute ne peut pas être utilisé pour cette connectivité.
- Une fois que les machines virtuelles ont été basculées vers un réseau virtuel Azure, vous pouvez y accéder à l’aide de la configuration du [peering privé](../expressroute/expressroute-circuit-peerings.md#privatepeering) avec le réseau virtuel Azure.


### <a name="if-i-replicate-to-azure-what-kind-of-storage-account-or-managed-disk-do-i-need"></a>Si j’effectue une réplication vers Azure, de quel type de compte de stockage ou disque managé ai-je besoin ?

Vous devez disposer d’un stockage LRS ou GRS. Nous vous recommandons d’utiliser un compte GRS, afin que les données soient résilientes si une panne se produit au niveau régional, ou si la région principale ne peut pas être récupérée. Ce compte doit se trouver dans la même région que le coffre Recovery Services. Le Stockage Premium est pris en charge pour les machines virtuelles VMware, les machines virtuelles Hyper-V et la réplication de serveurs physiques lorsque vous déployez Site Recovery dans le portail Azure. Les disques managés prennent uniquement en charge LRS.

### <a name="how-often-can-i-replicate-data"></a>À quelle fréquence puis-je répliquer les données ?
* **Hyper-V :** Les machines virtuelles Hyper-V peuvent être répliquées toutes les 30 secondes (sauf pour le stockage premium), toutes les 5 minutes ou toutes les 15 minutes.
* **Machines virtuelles Azure, machines virtuelles VMware et serveurs physiques :** une fréquence de réplication n’est pas pertinente ici. La réplication est continue.

### <a name="can-i-extend-replication-from-existing-recovery-site-to-another-tertiary-site"></a>Puis-je étendre la réplication depuis un site de récupération existant à un site tiers ?
La réplication étendue ou chaînée n’est pas prise en charge. Demandez cette fonctionnalité dans le [forum de commentaires](https://feedback.azure.com/forums/256299-site-recovery/suggestions/6097959).

### <a name="can-i-do-an-offline-replication-the-first-time-i-replicate-to-azure"></a>Puis-je effectuer une réplication hors connexion la première fois que je réplique vers Azure ?
Ceci n’est pas pris en charge. Demandez cette fonctionnalité dans le [forum de commentaires](https://feedback.azure.com/forums/256299-site-recovery/suggestions/6227386-support-for-offline-replication-data-transfer-from).

### <a name="can-i-exclude-specific-disks-from-replication"></a>Puis-je exclure des disques spécifiques de la réplication ?
Cette fonctionnalité est prise en charge dans le cadre d’une réplication de machines virtuelles VMware et de machines virtuelles Hyper-V vers Azure à l’aide du portail Azure.

### <a name="can-i-replicate-virtual-machines-with-dynamic-disks"></a>Puis-je répliquer des machines virtuelles avec des disques dynamiques ?
Les disques dynamiques sont pris en charge lors de la réplication des machines virtuelles Hyper-V et lors de la réplication des machines virtuelles VMware et physiques vers Azure. Le disque du système d’exploitation doit être un disque de base.


### <a name="can-i-throttle-bandwidth-allotted-for-replication-traffic"></a>Puis-je limiter la bande passante allouée pour le trafic de réplication ?
Oui. Pour plus d’informations sur la limitation de bande passante, consultez les articles suivants :

* [Planification de la capacité pour la réplication de machines virtuelles VMware et de serveurs physiques](site-recovery-plan-capacity-vmware.md)
* [Planification de la capacité pour la réplication de machines virtuelles Hyper-V dans Azure](./hyper-v-deployment-planner-overview.md)

### <a name="can-i-enable-replication-with-app-consistency-in-linux-servers"></a>Puis-je activer la réplication avec la cohérence des applications dans les serveurs Linux ? 
Oui. Azure Site Recovery pour le système d’exploitation Linux prend en charge les scripts personnalisés des applications à des fins de cohérence. Le script personnalisé avec options pré et post-script sera utilisé par l’agent Mobilité Azure Site Recovery durant la cohérence des applications. Voici les étapes pour activer cela.

1. Connectez-vous en tant qu’utilisateur racine à l’ordinateur.
2. Basculez le répertoire sur l’emplacement d’installation de l’agent Mobilité Azure Site Recovery. La valeur par défaut est « /usr/local/ASR »<br>
    `# cd /usr/local/ASR`
3. Remplacez le répertoire par « VX/scripts » sous l’emplacement d’installation<br>
    `# cd VX/scripts`
4. Créez un script d’interpréteur de commandes bash nommé « customscript.sh » avec des autorisations d’exécution pour l’utilisateur racine.<br>
    a. Le script doit prendre en charge les options de ligne de commande « --pre » et « --post » (notez les doubles tirets)<br>
    b. Lorsque le script est appelé avec l’option pre, il doit geler l’entrée/la sortie de l’application et, lorsqu’il est appelé avec l’option post, il doit libérer l’entrée/la sortie de l’application.<br>
    c. Exemple de modèle -<br>

    `# cat customscript.sh`<br>

```
    #!/bin/bash

    if [ $# -ne 1 ]; then
        echo "Usage: $0 [--pre | --post]"
        exit 1
    elif [ "$1" == "--pre" ]; then
        echo "Freezing app IO"
        exit 0
    elif [ "$1" == "--post" ]; then
        echo "Thawed app IO"
        exit 0
    fi
```

5. Ajoutez les commandes permettant de geler et de libérer l’entrée/la sortie dans les étapes pre et post pour les applications nécessitant une cohérence des applications. Vous pouvez choisir d’ajouter un autre script spécifiant ceux-ci et l’appeler à partir de « customscript.sh » avec les options pre et post.

>[!Note]
>La version de l’agent Site Recovery doit être 9.24 ou une version ultérieure pour prendre en charge les scripts personnalisés.

## <a name="replication-policy"></a>Stratégie de réplication

### <a name="what-is-a-replication-policy"></a>Qu’est-ce qu’une stratégie de réplication ?

Une stratégie de réplication définit les paramètres de l’historique de rétention des points de récupération. La stratégie définit également la fréquence des captures instantanées de cohérence des applications. Par défaut, Azure Site Recovery crée une nouvelle stratégie de réplication avec les paramètres par défaut suivants :

- 24 heures pour l’historique de rétention des points de récupération.
- 4 heures pour la fréquence des captures instantanées cohérentes au niveau application.

### <a name="what-is-a-crash-consistent-recovery-point"></a>Qu’est-ce qu’un point de récupération cohérent en cas d’incident ?

Un point de récupération de cohérence en cas d’incident contient les données sur disque comme si vous aviez débranché le cordon d’alimentation du serveur lors de la capture instantanée. Le point de récupération de cohérence en cas d’incident n’inclut rien de ce qui était en mémoire lors de la capture instantanée.

Aujourd’hui, la plupart des applications peuvent récupérer correctement à partir de captures instantanées cohérentes en cas d’incident. Un point de récupération cohérent en cas d’incident ne suffit généralement pas pour des systèmes d’exploitation de base de données et des applications telles que des serveurs de fichiers, des serveurs DHCP et des serveurs d’impression.

### <a name="what-is-the-frequency-of-crash-consistent-recovery-point-generation"></a>Quelle est la fréquence de génération de points de récupération cohérents en cas d’incident ?

Site Recovery crée un point de récupération cohérent en cas d’incident toutes les 5 minutes.

### <a name="what-is-an-application-consistent-recovery-point"></a>Qu’est-ce qu’un point de récupération cohérent au niveau application ?

Les points de récupération cohérents au niveau application sont créés à partir de captures instantanées cohérentes au niveau application. Des points de récupération de cohérence des applications capturent les mêmes données que des captures instantanées de cohérence en cas d’incident, ainsi que de toutes les données en mémoire et toutes les transactions en cours.

En raison de leur contenu supplémentaire, les captures instantanées de cohérence des applications sont davantage sollicitées et prennent le plus de temps. Les points de récupération cohérent au niveau application sont recommandés pour des systèmes d’exploitation de base de données et des applications telles que SQL Server.

### <a name="what-is-the-impact-of-application-consistent-recovery-points-on-application-performance"></a>Quel est l’impact des points de récupération cohérents au niveau de l'application sur les performances de cette dernière ?

Les points de récupération de cohérence des applications capturent toutes les données en mémoire et en cours. Étant donné que les points de récupération capturent ces données, ils nécessitent une infrastructure telle que VSS sur Windows pour suspendre l’application. Si le processus de capture est fréquent, cela peut affecter les performances lorsque la charge de travail est déjà occupée. Nous déconseillons d’utiliser une fréquence faible pour les points de récupération de cohérence des applications en lien avec des charges de travail autres que de bases de données. Même pour une charge de travail de base de données, une heure suffit.

### <a name="what-is-the-minimum-frequency-of-application-consistent-recovery-point-generation"></a>Quelle est la fréquence minimale de génération de points de récupération cohérents en cas d’incident sur les applications ?

Site Recovery peut créer un point de récupération de cohérence des applications généré à une fréquence minimale d’une heure.

### <a name="how-are-recovery-points-generated-and-saved"></a>Comment les points de récupération sont-ils générés et enregistrés ?

Pour comprendre comment Site Recovery génère les points de récupération, voyons un exemple de stratégie de réplication. Cette stratégie de réplication utilise un point de récupération avec une fenêtre de rétention de 24 heures et une capture instantanée de cohérence des applications à la fréquence d’une heure.

Site Recovery crée un point de récupération cohérent en cas d’incident toutes les 5 minutes. Vous ne pouvez pas modifier cette fréquence. Pour la dernière heure, vous pouvez choisir parmi 12 points de cohérence en cas d’incident et 1 point de cohérence des applications. Au fil du temps, au-delà de la dernière heure, Site Recovery élague les points de récupération pour n’en enregistrer qu’un seul par heure.

La capture d’écran suivante illustre cet exemple. Dans la capture d’écran :

- Au cours de la dernière heure, la fréquence des points de récupération est de 5 minutes.
- Au-delà de la dernière heure, Site Recovery ne conserve qu’un seul point de récupération.

   ![Liste des points de récupération générés](./media/azure-to-azure-troubleshoot-errors/recoverypoints.png)

### <a name="how-far-back-can-i-recover"></a>Jusqu’à quand peut remonter la récupération ?

Le point de récupération le plus ancien que vous pouvez utiliser remonte à 72 heures.

### <a name="i-have-a-replication-policy-of-24-hours-what-will-happen-if-a-problem-prevents-site-recovery-from-generating-recovery-points-for-more-than-24-hours-will-my-previous-recovery-points-be-lost"></a>J’ai une stratégie de réplication de 24 heures. Que se passe-t-il si un problème empêche Site Recovery de générer des points de récupération pendant plus de 24 heures ? Mes points de récupération antérieurs sont-ils perdus ?

Non, Site Recovery conserve tous vos points de récupération antérieurs. En fonction de la fenêtre de rétention des points de récupération, Site Recovery ne remplace le point le plus ancien que s’il génère de nouveaux points. En raison de ce problème, Site Recovery ne peut pas générer de nouveaux points de récupération. Tant qu’il n’y a pas de nouveaux points de récupération, tous les anciens points subsistent une fois la fenêtre de rétention atteinte.

### <a name="after-replication-is-enabled-on-a-vm-how-do-i-change-the-replication-policy"></a>Une fois la réplication activée sur une machine virtuelle, comment modifier la stratégie de réplication ?

Accédez à **Coffre Site Recovery** > **Infrastructure Site Recovery** > **Stratégies de réplication** . Sélectionnez la stratégie à modifier, modifiez-la, puis enregistrez les modifications. Toute modification s’applique également à toutes les réplications existantes.

### <a name="are-all-the-recovery-points-a-complete-copy-of-the-vm-or-a-differential"></a>Tous les points de récupération sont-ils une copie complète ou différentielle de la machine virtuelle ?

Le premier point de récupération qui est généré possède la copie complète. Les points de récupération successifs ont des modifications d’ordre différentiel.

### <a name="does-increasing-the-retention-period-of-recovery-points-increase-the-storage-cost"></a>L’accroissement de la période de rétention des points de récupération augmente-t-elle le coût de stockage ?

Si vous allongez la période de rétention de 24 à 72 heures, Site Recovery enregistre les points de récupération pendant 48 heures supplémentaires. Cette durée supplémentaire occasionne des frais de stockage. Par exemple, un point de récupération unique peut avoir des modifications différentielles de 10 Go avec un coût par Go de 0,16 USD par mois. Les frais supplémentaires sont alors de 1,60 × 48 USD par mois.


## <a name="failover"></a>Basculement
### <a name="if-im-failing-over-to-azure-how-do-i-access-the-azure-vms-after-failover"></a>Si j’effectue un basculement vers Azure, comment accéder aux machines virtuelles Azure après le basculement ?

Vous pouvez accéder aux machines virtuelles Azure via une connexion Internet sécurisée, via un réseau privé virtuel de site à site ou via Azure ExpressRoute. Vous devez préparer un certain nombre d’éléments afin de vous connecter. [Plus d’informations](site-recovery-test-failover-to-azure.md#prepare-to-connect-to-azure-vms-after-failover)


### <a name="if-i-fail-over-to-azure-how-does-azure-make-sure-my-data-is-resilient"></a>Si j’effectue le basculement vers Azure, comment Azure s’assure-t-il de la résilience de mes données ?
Azure est conçu pour la résilience. Site Recovery est déjà prévu pour assurer le basculement vers un centre de données Azure secondaire, dans le respect du contrat SLA Azure. Dans ce cas, nous nous assurons que vos métadonnées et vos coffres restent dans la même région géographique que vous avez choisie pour votre coffre.  

### <a name="if-im-replicating-between-two-datacenters-what-happens-if-my-primary-datacenter-experiences-an-unexpected-outage"></a>Si j’effectue une réplication entre deux centres de données, que se passe-t-il si mon centre de données principal connaît une panne inattendue ?
Vous pouvez déclencher un basculement non planifié à partir du site secondaire. Site Recovery n’a pas besoin d’être connecté au site principal pour effectuer le basculement.

### <a name="is-failover-automatic"></a>Le basculement est-il automatique ?
Le basculement n’est pas automatique. Vous lancez les basculements d’un seul clic dans le portail, ou vous pouvez utiliser [Site Recovery PowerShell](/powershell/module/az.recoveryservices) pour déclencher un basculement. La restauration automatique est une action simple dans le portail Site Recovery.

Pour automatiser les processus, vous pouvez utiliser Orchestrator ou Operations Manager localement pour détecter une défaillance de machine virtuelle, puis déclencher le basculement à l’aide du kit SDK.

* [Découvrez plus d’informations](site-recovery-create-recovery-plans.md) sur les plans de récupération.
* [En savoir plus](site-recovery-failover.md) sur le basculement.
* [En savoir plus](./vmware-azure-failback.md) sur la restauration automatique de serveurs physiques et de machines virtuelles VMware

### <a name="if-my-on-premises-host-is-not-responding-or-crashed-can-i-fail-back-to-a-different-host"></a>Si mon hôte local ne répond pas ou est bloqué, puis-je effectuer une restauration automatique vers un hôte différent ?
Oui, vous pouvez utiliser la récupération à un autre emplacement pour la restauration automatique vers un hôte différent depuis Azure.

* [Pour les machines virtuelles VMware](concepts-types-of-failback.md#alternate-location-recovery-alr)
* [Pour les machines virtuelles Hyper-V](hyper-v-azure-failback.md#fail-back-to-an-alternate-location)

## <a name="automation"></a>Automatisation

### <a name="can-i-automate-site-recovery-scenarios-with-an-sdk"></a>Puis-je automatiser des scénarios Site Recovery avec un kit SDK ?
Oui. Vous pouvez automatiser les flux de travail Site Recovery à l’aide de l’API Rest, de PowerShell ou du kit SDK Azure. Scénarios actuellement pris en charge pour le déploiement de Site Recovery à l’aide de PowerShell :

* [Réplication vers Azure de machines virtuelles Hyper-V hébergées dans des clouds VMM à l’aide de PowerShell et d’Azure Resource Manager](hyper-v-vmm-powershell-resource-manager.md)
* [Réplication vers Azure de machines virtuelles Hyper-V (sans VMM) à l’aide de PowerShell et d’Azure Resource Manager](hyper-v-azure-powershell-resource-manager.md)
* [Réplication de VMware vers Azure avec le gestionnaire des ressources de PowerShell](vmware-azure-disaster-recovery-powershell.md)

## <a name="componentprovider-upgrade"></a>Mise à niveau du composant/fournisseur

### <a name="where-can-i-find-the-release-notesupdate-rollups-of-site-recovery-upgrades"></a>Où trouver les notes de publication/correctifs cumulatifs des mises à niveau Azure Site Recovery ?

[Apprenez-en davantage](site-recovery-whats-new.md) sur les nouvelles mises à jour et [obtenez des informations sur les correctifs cumulatifs](service-updates-how-to.md).

## <a name="next-steps"></a>Étapes suivantes
* Lisez la [Vue d’ensemble de Microsoft Azure Site Recovery](site-recovery-overview.md)