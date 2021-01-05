---
title: Exécuter un test de reprise d’activité après sinistre pour NHyper-V sur un site secondaire avec Azure Site Recovery
description: Découvrez comment effectuer un test de reprise d’activité de machines virtuelles Hyper-V de clouds VMM dans un centre de données secondaire local à l’aide d’Azure Site Recovery.
author: sideeksh
manager: gaggupta
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/27/2018
ms.author: sideeksh
ms.openlocfilehash: 569af28f5773d843f49dd9c8143b45e308ae142e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87420411"
---
# <a name="run-a-dr-drill-for-hyper-v-vms-to-a-secondary-site"></a>Exécuter un test de récupération d’urgence de machines virtuelles Hyper-V sur un site secondaire


Cet article décrit comment effectuer un test de récupération d’urgence pour les machines virtuelles Hyper-V qui sont managées dans des clouds System Center Virtual Machine Manager (VMM), sur un site local secondaire en utilisant [Azure Site Recovery](site-recovery-overview.md).

Vous exécutez un test de basculement pour valider votre stratégie de réplication, et vous effectuez un test de récupération d’urgence sans perte de données ni temps d’arrêt. L’exécution d’un test de basculement n’a aucun impact sur la réplication en cours ou sur votre environnement de production. 

## <a name="how-do-test-failovers-work"></a>Comment fonctionnent les tests de basculement ?

Vous exécutez un test de basculement depuis le site principal vers le site secondaire. Si vous souhaitez simplement vérifier qu’une machine virtuelle bascule, vous pouvez réaliser un test de basculement sans rien configurer sur le site secondaire. Si vous voulez vérifier le bon fonctionnement d’un basculement d’application, vous devez configurer la mise en réseau et l’infrastructure dans l’emplacement secondaire.
- Vous pouvez exécuter un test de basculement sur une seule machine virtuelle, ou sur un [plan de récupération](site-recovery-create-recovery-plans.md).
- Vous pouvez exécuter un test de basculement sans réseau, avec un réseau existant, ou par le biais d’un réseau créé automatiquement. Des informations plus détaillées sur ces options sont fournies dans le tableau ci-dessous.
    - Vous pouvez exécuter un test de basculement sans réseau. Cette option est pratique si vous souhaitez simplement vérifier la possibilité de basculement d’une machine virtuelle, par contre vous ne pouvez pas contrôler une configuration réseau.
    - Exécutez le basculement par le biais d’un réseau existant. Nous vous recommandons de ne pas utiliser de réseau de production.
    - Exécutez le basculement et laissez Site Recovery créer automatiquement le réseau de test. Dans ce cas, Site Recovery crée automatiquement le réseau, puis le nettoie lorsque le test de basculement est terminé.
- Vous devez sélectionner un point de récupération pour le test de basculement : 
    - **Dernier point traité** : cette option bascule une machine virtuelle vers le dernier point de récupération traité par Site Recovery. Cette option fournit un objectif de délai de récupération (RTO) faible, car aucun temps n’est consacré à traiter les données non traitées.
    - **Dernier point de cohérence des applications** : cette option bascule une machine virtuelle vers le dernier point de récupération de cohérence des applications traité par Site Recovery. 
    - **Dernier point dans le temps** : cette option permet de traiter d’abord toutes les données qui ont été envoyées au service Site Recovery afin de créer un point de récupération pour chaque machine virtuelle avant de basculer les machines virtuelles vers celui-ci. Elle fournit l’objectif de point de récupération (RPO) le plus faible, car la machine virtuelle créée après le basculement comporte toutes les données répliquées vers Site Recovery au moment où le basculement a été déclenché.
    - **Dernier point multimachine virtuelle traité** : disponible pour les plans de récupération qui comportent une ou plusieurs machines virtuelles dont la cohérence multimachine virtuelle est activée. Les machines virtuelles pour lesquelles ce paramètre est activé basculent vers le dernier point de récupération de cohérence multimachine virtuelle commun. Les autres machines virtuelles basculent vers le dernier point de récupération traité.
    - **Dernière cohérence des applications multimachines virtuelles** : cette option est disponible pour les plans de récupération qui comportent une ou plusieurs machines virtuelles pour laquelle ou lesquelles la cohérence multimachine virtuelle est activée. Les machines virtuelles qui font partie d’un groupe de réplication basculent vers le dernier point de récupération de cohérence des applications multimachine virtuelle commun. Les autres machines virtuelles basculent vers leur dernier point de récupération de cohérence des applications.
    - **Personnalisé** : utilisez cette option pour basculer une machine virtuelle spécifique vers un point de récupération particulier.



## <a name="prepare-networking"></a>Préparer la mise en réseau

Lorsque vous exécutez un test de basculement, vous êtes invité à sélectionner les paramètres réseau des machines de réplication utilisées pour le test, comme résumé dans le tableau.

| **Option** | **Détails** | |
| --- | --- | --- |
| **Aucun** | La machine virtuelle de test est créée sur l’hôte où se trouve la machine virtuelle de réplication. Elle n’est pas ajoutée au cloud et n’est connectée à aucun réseau.<br/><br/> Vous pouvez connecter la machine à un réseau de machines virtuelles après sa création.| |
| **Utiliser l’existant** | La machine virtuelle de test est créée sur l’hôte où se trouve la machine virtuelle de réplication. Elle n’est pas ajoutée au cloud.<br/><br/>Créez un réseau de machines virtuelles isolé de votre réseau de production.<br/><br/>Si vous utilisez un réseau basé sur un réseau VLAN, nous vous recommandons de créer un réseau logique distinct (non utilisé en production) dans VMM, à cet effet. Ce réseau logique est utilisé pour créer des réseaux de machines virtuelles pour le test de basculement.<br/><br/>Le réseau logique doit être associé à au moins une des cartes réseau de tous les serveurs Hyper-V hébergeant des machines virtuelles.<br/><br/>Pour les réseaux logiques VLAN, les sites de réseau que vous ajoutez au réseau logique doivent être isolés.<br/><br/>Si vous utilisez un réseau logique basé sur la fonction de virtualisation réseau Windows, Azure Site Recovery crée automatiquement des réseaux de machines virtuelles isolés. | |
| **Créer un réseau** | Un réseau de test temporaire est créé automatiquement, en fonction du paramètre que vous spécifiez dans le champ **Réseau logique** et sur les sites réseau associés.<br/><br/> Le basculement vérifie que les machines virtuelles sont créées.<br/><br/> Vous devez utiliser cette option si un plan de récupération fait appel à plusieurs réseaux de machines virtuelles.<br/><br/> Si vous exploitez des réseaux de virtualisation de réseau Windows, cette option peut être utilisée pour créer automatiquement des réseaux de machines virtuelles à partir des mêmes paramètres (sous-réseaux et pools d’adresses IP) que ceux du réseau de l’ordinateur virtuel de réplication. Ces réseaux de machines virtuelles sont automatiquement nettoyés une fois le test de basculement terminé.<br/><br/> La machine virtuelle de test est créée sur l’hôte où se trouve la machine virtuelle de réplication. Elle n’est pas ajoutée au cloud.|

### <a name="best-practices"></a>Meilleures pratiques

- Tester un réseau de production entraîne une interruption des charges de travail de production. Demandez aux utilisateurs de ne pas utiliser d’applications associées lorsque la simulation de récupération d’urgence est en cours.

- Il n’est pas nécessaire que le réseau de test corresponde au type de réseau logique VMM utilisé pour le test de basculement. Toutefois, certaines combinaisons ne fonctionnent pas :

     - Si le réplica utilise l’isolement basé sur VLAN ou DHCP, le réseau de machines virtuelles associé au réplica n’a pas besoin d’un pool d’adresses IP statiques. Ainsi, l’utilisation de la fonction de virtualisation de réseau Windows pour un test de basculement ne peut pas aboutir, car aucun pool d’adresses n’est disponible. 
        
     - Le test de basculement ne fonctionne pas si le réseau de réplication n’est associé à aucune isolation, et si le réseau de test utilise la fonction de virtualisation de réseau Windows. En effet, un réseau ne présentant aucune isolation n’inclut aucun sous-réseau requis pour créer un réseau utilisant la fonction de virtualisation de réseau Windows.
        
- Nous vous recommandons de ne pas utiliser pour test de basculement le réseau que vous avez sélectionné pour le mappage réseau.

- Le mode de connexion des ordinateurs virtuels de réplication aux réseaux de machines virtuelles mappés après le basculement dépend de la configuration choisie pour le réseau de machines virtuelles dans la console VMM.


### <a name="vm-network-configured-with-no-isolation-or-vlan-isolation"></a>Réseau de machines virtuelles configuré sans isolation ou avec isolation du VLAN

Si un réseau de machines virtuelles est configuré dans VMM sans isolation, ou avec l’isolation VLAN, notez les points suivants :

- Si le protocole DHCP est défini pour le réseau de machines virtuelles, l’ordinateur virtuel de réplication est connecté à l’ID VLAN au moyen des paramètres spécifiés pour le site réseau dans le réseau logique associé. La machine virtuelle reçoit son adresse IP du serveur DHCP disponible.
- Vous n’avez pas besoin de définir un pool d’adresses IP statiques pour le réseau de machines virtuelles cible. Si un pool d’adresses IP statiques est utilisé pour le réseau de machines virtuelles, l’ordinateur virtuel de réplication est connecté à l’ID VLAN au moyen des paramètres spécifiés pour le site réseau dans le réseau logique associé.
- La machine virtuelle reçoit son adresse IP du pool défini pour le réseau de machines virtuelles. Si aucun pool d’adresses IP statiques n’est défini sur le réseau de machines virtuelles cible, le processus d’allocation d’une adresse IP échoue. Créez le pool d’adresses IP sur les serveurs VMM source et cible que vous allez utiliser à des fins de protection et de récupération.

### <a name="vm-network-with-windows-network-virtualization"></a>Réseau de machines virtuelles avec virtualisation de réseau Windows

Si un réseau de machines virtuelles est configuré dans VMM avec la virtualisation de réseau Windows, tenez compte des informations données ici.

- Vous devez définir un pool statique pour le réseau de machines virtuelles cible, que le réseau de machines virtuelles source soit configuré ou non pour utiliser le protocole DHCP ou un pool d’adresses IP statiques. 
- Si vous définissez le protocole DHCP, le serveur VMM cible joue le rôle de serveur DHCP et fournit une adresse IP provenant du pool défini pour le réseau de machines virtuelles cible.
- Si l’utilisation d’un pool d’adresses IP statiques est définie pour le serveur source, le serveur VMM cible alloue une adresse IP à partir du pool. Dans les deux cas, le processus d’allocation d’une adresse IP échoue si aucun pool d’adresses IP statiques n’est défini.



## <a name="prepare-the-infrastructure"></a>Préparer l’infrastructure

Si vous souhaitez simplement vérifier la possibilité de basculement d’une machine virtuelle, vous pouvez exécuter un test de basculement sans infrastructure. Si vous souhaitez effectuer un test de récupération d’urgence complet pour tester le basculement d’application, vous devez préparer l’infrastructure sur le site secondaire.

- Si vous exécutez un test de basculement à l’aide d’un réseau existant, vous devez préparer les services Active Directory, DHCP et DNS de ce réseau.
- Si vous exécutez un test de basculement avec l’idée de créer un réseau de machines virtuelles automatiquement, vous devez ajouter des ressources d’infrastructure au réseau créé automatiquement, avant d’exécuter le test de basculement. Dans un plan de récupération, cette opération est facilitée en ajoutant une étape manuelle avant le groupe 1, dans le plan de récupération que vous allez utiliser pour le test de basculement. Ajoutez ensuite les ressources de l’infrastructure au réseau créé automatiquement avant d’exécuter le test de basculement.


### <a name="prepare-dhcp"></a>Préparer le service DHCP
Si les machines virtuelles impliquées dans le test de basculement utilisent le protocole DHCP, créez un serveur DHCP de test dans le réseau isolé pour les besoins du test de basculement.


### <a name="prepare-active-directory"></a>Préparation du système Active Directory
Pour exécuter un test de basculement afin de tester des applications, vous devez créer une copie de l’environnement Active Directory de production dans votre environnement de test. Pour plus d’informations, consultez la rubrique [Considérations en matière de test de basculement pour Active Directory](site-recovery-active-directory.md#test-failover-considerations).

### <a name="prepare-dns"></a>Préparer le service DNS
Préparer un serveur DNS pour le test de basculement en procédant comme suit :

* **DHCP** : si les machines virtuelles utilisent DHCP, l’adresse IP du serveur DNS de test doit être mise à jour sur le serveur DHCP de test. Si vous utilisez un type de réseau associé à la virtualisation de réseau Windows, le serveur VMM joue le rôle de serveur DHCP. Par conséquent, l’adresse IP du serveur DNS doit être mise à jour dans le réseau de test de basculement. Dans ce cas, les machines virtuelles s’enregistrent auprès du serveur DNS pertinent.
* **Adresse statique** : si les machines virtuelles utilisent une adresse IP statique, l’adresse IP du serveur DNS de test doit être mise à jour dans le réseau de test de basculement. Vous devrez peut-être mettre à jour le service DNS en indiquant l’adresse IP des machines virtuelles de test. À cette fin, vous pouvez utiliser l’exemple de script suivant :

  ```powershell
  Param(
  [string]$Zone,
  [string]$name,
  [string]$IP
  )
  $Record = Get-DnsServerResourceRecord -ZoneName $zone -Name $name
  $newrecord = $record.clone()
  $newrecord.RecordData[0].IPv4Address  =  $IP
  Set-DnsServerResourceRecord -zonename $zone -OldInputObject $record -NewInputObject $Newrecord
  ```

## <a name="run-a-test-failover"></a>Exécuter un test de basculement

Cette procédure explique comment exécuter un test de basculement pour un plan de récupération. Vous pouvez également exécuter le basculement d’une machine virtuelle unique, via l’onglet **Machines virtuelles** .

1. Sélectionnez **Plans de récupération** > *nom_planrécupération*. Cliquez sur **Type de basculement** > **Test Type de basculement**.
2. Dans le panneau **Test de basculement**, indiquez le mode à utiliser pour la connexion des machines virtuelles de réplication aux réseaux après le test de basculement.
3. Effectuez un suivi de l’opération sur l’onglet **Tâches** .
4. Une fois le basculement terminé, vérifiez que les machines virtuelles démarrent correctement.
5. Une fois que vous avez terminé, cliquez sur **Nettoyer le test de basculement** sur le plan de récupération. Cliquez sur **Notes** pour consigner et enregistrer d’éventuelles observations sur le test de basculement. Cette étape supprime toutes les machines virtuelles et les réseaux qui ont été créés par Site Recovery pendant le test de basculement. 

![Test de basculement](./media/hyper-v-vmm-test-failover/TestFailover.png)
 


> [!TIP]
> L’adresse IP donnée à une machine virtuelle durant un test de basculement est identique à l’adresse IP qu’elle recevrait durant un basculement planifié ou non planifié (à condition que l’adresse IP soit disponible dans le réseau de test de basculement). Si la même adresse IP n’est pas disponible dans le réseau de test de basculement, la machine virtuelle reçoit une autre adresse IP disponible dans le réseau de test de basculement.



### <a name="run-a-test-failover-to-a-production-network"></a>Exécuter un test de basculement vers un réseau de production

Nous vous recommandons de ne pas exécuter de test de basculement vers le réseau de site de récupération de production que vous avez spécifié lors du mappage réseau. Cependant, si vous voulez vraiment valider la connectivité réseau de bout en bout dans une machine virtuelle basculée, tenez compte des informations précisées ici.

* Vérifiez que la machine virtuelle principale est arrêtée quand vous procédez au test de basculement. Si vous ne le faites pas, deux machines virtuelles avec la même identité s’exécuteront sur le même réseau en même temps. Cette situation peut entraîner des conséquences indésirables.
* Les modifications que vous apportez aux machines virtuelles du test de basculement sont perdues au moment du nettoyage de ces machines. Ces modifications ne sont pas répliquées vers les machines virtuelles principales.
* Cette manière d’effectuer le test entraîne un temps d’arrêt de votre application de production. Demandez aux utilisateurs de l’application de ne pas utiliser l’application au cours du test de récupération d’urgence.  


## <a name="next-steps"></a>Étapes suivantes
Après l’exécution d’un test de récupération d’urgence concluant, vous pouvez [exécuter un basculement complet](site-recovery-failover.md).



