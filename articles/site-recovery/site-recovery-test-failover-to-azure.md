---
title: Exécuter un test de basculement (exercice de récupération d'urgence) sur Azure dans Azure Site Recovery
description: Découvrez l’exécution d’un test de basculement à partir d’un site local vers Azure, à l’aide du service Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/14/2019
ms.author: raynew
ms.openlocfilehash: 815a35c167bbcd3ac03dfdaaf6d699e58a791f33
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92369384"
---
# <a name="run-a-test-failover-disaster-recovery-drill-to-azure"></a>Exécuter un test de basculement (exercice de récupération d'urgence) sur Azure 


Cet article explique comment exécuter une simulation de récupération d’urgence vers Azure, à l’aide d’un test de basculement Site Recovery.  

Vous exécutez un test de basculement pour valider votre stratégie de réplication et de récupération d’urgence sans perte de données ou temps d’arrêt. Un test de basculement n’a aucun effet sur la réplication en cours ni sur votre environnement de production. Vous pouvez exécuter un test de basculement sur une machine virtuelle spécifique ou sur un [plan de récupération](site-recovery-create-recovery-plans.md) contenant plusieurs machines virtuelles.


## <a name="run-a-test-failover"></a>Exécuter un test de basculement
Cette procédure explique comment exécuter un test de basculement pour un plan de récupération. Si vous souhaitez exécuter un test de basculement pour une seule machine virtuelle, suivez les étapes décrites [ici](tutorial-dr-drill-azure.md#run-a-test-failover-for-a-single-vm).

![Capture d’écran de la page Test de basculement dans le portail Azure.](./media/site-recovery-test-failover-to-azure/TestFailover.png)


1. Dans Site Recovery dans le portail Azure, cliquez sur **Plans de récupération** > *nom_plan_récupération* > **Test de basculement** .
2. Sélectionnez un **point de récupération** vers lequel basculer. Vous pouvez utiliser l’une des options suivantes :
    - **Dernier point traité**  : cette option bascule toutes les machines virtuelles du plan vers le dernier point de récupération traité par Site Recovery. Pour voir le dernier point de récupération d’une machine virtuelle spécifique, cochez **Derniers points de récupération** dans les paramètres de la machine virtuelle. Cette option fournit un objectif de délai de récupération (RTO) faible, car aucun temps n’est consacré à traiter les données non traitées.
    - **Dernier point de cohérence des applications**  : cette option bascule toutes les machines virtuelles du plan vers le dernier point de récupération de cohérence des applications traité par Site Recovery. Pour voir le dernier point de récupération d’une machine virtuelle spécifique, cochez **Derniers points de récupération** dans les paramètres de la machine virtuelle.
    - **Les dernières**  : cette option permet de traiter d’abord toutes les données qui ont été envoyées au service Site Recovery afin de créer un point de récupération pour chaque machine virtuelle avant de basculer vers celui-ci. Elle fournit l’objectif de point de récupération (RPO) le plus faible, car la machine virtuelle créée après le basculement comporte toutes les données répliquées vers Site Recovery au moment où le basculement a été déclenché.
    - **Dernier point multimachine virtuelle traité**  : cette option est disponible pour les plans de récupération qui comportent une ou plusieurs machines virtuelles dont la cohérence multimachine virtuelle est activée. Les machines virtuelles pour lesquelles ce paramètre est activé basculent vers le dernier point de récupération de cohérence multimachine virtuelle commun. Les autres machines virtuelles basculent vers le dernier point de récupération traité.  
    - **Dernière cohérence des applications multimachines virtuelles**  : cette option est disponible pour les plans de récupération qui comportent une ou plusieurs machines virtuelles dont la cohérence multimachine virtuelle est activée. Les machines virtuelles qui font partie d’un groupe de réplication basculent vers le dernier point de récupération de cohérence des applications multimachine virtuelle commun. Les autres machines virtuelles basculent vers leur dernier point de récupération de cohérence des applications.
    - **Personnalisé**  : utilisez cette option pour basculer une machine virtuelle spécifique vers un point de récupération particulier.
3. Sélectionnez un réseau virtuel Azure dans lequel des machines virtuelles de test seront créées.

    - Site Recovery essaie de créer des machines virtuelles de test dans un sous-réseau avec le même nom et la même adresse IP que ceux fournis dans les paramètres **Calcul et réseau** de la machine virtuelle.
    - Si aucun sous-réseau portant le même nom n’est disponible dans le réseau virtuel Azure utilisé pour le test de basculement, alors la machine virtuelle de test est créée dans le premier sous-réseau dans l’ordre alphabétique.
    - Si la même adresse IP n’est pas disponible dans le sous-réseau, alors la machine virtuelle reçoit une autre adresse IP disponible dans le sous-réseau. [Plus d’informations](#create-a-network-for-test-failover)
4. Si vous effectuez le basculement vers Azure alors que le chiffrement des données est activé, accédez à la zone **Clé de chiffrement** et sélectionnez le certificat émis lorsque vous avez activé le chiffrement pendant l’installation du fournisseur. Vous pouvez ignorer cette étape si le chiffrement n’est pas activé.
5. Effectuez un suivi de l’opération sur l’onglet **Tâches** . Vous devriez voir apparaître l’ordinateur virtuel de réplication de test sur le portail Microsoft Azure.
6. Pour lancer une connexion RDP à la machine virtuelle Azure, vous devez [ajouter une adresse IP publique](/archive/blogs/srinathv/how-to-add-a-public-ip-address-to-azure-vm-for-vm-failed-over-using-asr) sur l’interface réseau de la machine virtuelle basculée.
7. Lorsque tout fonctionne comme prévu, cliquez sur **Nettoyer le test de basculement** . Cette opération supprime les machines virtuelles qui ont été créées au cours du test de basculement.
8. Cliquez sur **Notes** pour consigner et enregistrer d’éventuelles observations sur le test de basculement.


![Capture d’écran de l’onglet Tâches du test de basculement.](./media/site-recovery-test-failover-to-azure/TestFailoverJob.png)

Quand vous déclenchez un test de basculement, voici ce qui se produit :

1. **Prérequis** : Une vérification des prérequis est effectuée pour garantir que toutes les conditions nécessaires pour le basculement sont respectées.
2. **Basculement** : Le basculement se produit et prépare les données, afin qu’une machine virtuelle Azure puisse être créée.
3. **Les dernières**  : Si vous avez choisi le dernier point de récupération, un point de récupération est créé à partir des données envoyées au service.
4. **Démarrer** : cette étape crée une machine virtuelle Azure à l’aide des données traitées à l’étape précédente.

### <a name="failover-timing"></a>Durée du basculement

Dans les scénarios suivants, le basculement nécessite une étape intermédiaire supplémentaire qui prend généralement entre 8 et 10 minutes pour s’accomplir :

* Machines virtuelles VMware exécutant une version du service Mobility antérieure à 9.8
* Serveurs physiques
* Machines virtuelles VMware Linux
* Machines virtuelles Hyper-V protégées en tant que serveurs physiques
* Machines virtuelles VMware où les pilotes suivants ne sont pas des pilotes de démarrage :
    * storvsc
    * vmbus
    * storflt
    * intelide
    * atapi
* Machines virtuelles VMware pour lesquelles DHCP n’est pas activé, qu’elles utilisent ou non des adresses DHCP ou IP statiques.

Dans tous les autres cas, aucune étape intermédiaire n’est exigée et le basculement prend beaucoup moins de temps.


## <a name="create-a-network-for-test-failover"></a>Créer un réseau pour le test de basculement

Pour le test de basculement, nous vous recommandons de choisir un réseau isolé du réseau du site de récupération de production indiqué dans les paramètres **Calcul et réseau** de chaque machine virtuelle. Lorsque vous créez un réseau virtuel Azure, celui-ci est par défaut isolé des autres réseaux. Le réseau de test doit reproduire votre réseau de production :

- Le réseau de test doit avoir le même nombre de sous-réseaux que votre réseau de production. Les sous-réseaux doivent porter les mêmes noms.
- Le réseau de test doit utiliser la même plage d’adresses IP.
- Mettez à jour le DNS du réseau de test avec l’adresse IP spécifiée pour la machine virtuelle DNS dans les paramètres **Calcul et réseau** . Pour plus d’informations, consultez [Considérations relatives au test de basculement pour Active Directory](site-recovery-active-directory.md#test-failover-considerations).


## <a name="test-failover-to-a-production-network-in-the-recovery-site"></a>Test de basculement vers un réseau de production sur le site de récupération

Bien que nous vous recommandions d’utiliser un réseau de test distinct de votre réseau de production, si vous ne souhaitez pas exécuter une simulation de récupération d’urgence dans votre réseau de production, notez les points suivants :

- Vérifiez que la machine virtuelle principale est arrêtée quand vous exécutez le test de basculement. Si vous ne le faites pas, deux machines virtuelles avec la même identité s’exécuteront simultanément sur le même réseau. Les conséquences risquent alors d’être inattendues.
- Toute modification apportée aux machines virtuelles créées pour le test de basculement est perdue quand vous nettoyez le basculement. Ces modifications ne sont pas répliquées vers la machine virtuelle principale.
- Les tests effectués dans votre environnement de production entraînent un temps d’arrêt de votre application de production. Les utilisateurs ne doivent pas utiliser des applications qui s’exécutent sur des machines virtuelles quand le test de basculement est en cours d’exécution.  



## <a name="prepare-active-directory-and-dns"></a>Préparer Active Directory et DNS

Pour exécuter un test de basculement afin de tester des applications, vous devez créer une copie de votre environnement Active Directory de production dans votre environnement de test. Pour plus d’informations, consultez [Considérations relatives au test de basculement pour Active Directory](site-recovery-active-directory.md#test-failover-considerations).

## <a name="prepare-to-connect-to-azure-vms-after-failover"></a>Préparer la connexion aux machines virtuelles Azure après le basculement

Si vous souhaitez vous connecter à des machines virtuelles Azure à l’aide de RDP/SSH après le basculement, respectez les exigences récapitulées dans le tableau.

**Type de basculement** | **Lieu** | **Actions**
--- | --- | ---
**Machine virtuelle Azure exécutant Windows** | Machine locale avant le basculement | Pour accéder à la machine virtuelle Azure par Internet, activez la fonction RDP, vérifiez que les règles TCP et UDP sont ajoutées pour **Public** et que RDP est autorisé pour tous les profils dans **Pare-feu Windows** > **Applications autorisées** .<br/><br/> Pour accéder à la machine virtuelle Azure via une connexion de site à site, activez RDP sur la machine, en vérifiant que ce dernier est autorisé dans **Pare-feu Windows** -> **Applications et fonctionnalités autorisées** , pour les réseaux de types **Domaine et Privé** .<br/><br/>  Vérifiez que la stratégie SAN du système d’exploitation est définie sur la valeur **OnlineAll** . [Plus d’informations](https://support.microsoft.com/kb/3031135)<br/><br/> Vérifiez qu’aucune mise à jour de Windows n’est en attente sur la machine virtuelle quand vous déclenchez un basculement. Une mise à jour de Windows peut démarrer quand vous procédez au basculement, et vous ne pouvez pas ouvrir de session sur la machine virtuelle tant que la mise à jour n’est pas terminée.
**Machine virtuelle Azure exécutant Windows** | Machine virtuelle Azure après le basculement |  [Ajoutez une adresse IP publique](/archive/blogs/srinathv/how-to-add-a-public-ip-address-to-azure-vm-for-vm-failed-over-using-asr) pour la machine virtuelle.<br/><br/> Les règles des groupes de sécurité réseau figurant sur la machine virtuelle basculée (et le sous-réseau Azure auquel elle est connectée) doivent autoriser les connexions entrantes avec le port RDP.<br/><br/> Cochez **Diagnostics de démarrage** pour examiner une capture d’écran de la machine virtuelle.<br/><br/> Si vous ne parvenez pas à vous connecter, vérifiez que la machine virtuelle est en cours d’exécution et consultez ces [conseils de résolution des problèmes](https://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx).
**Machine virtuelle Azure exécutant Linux** | Machine locale avant le basculement | Vérifiez que le service Secure Shell, sur la machine virtuelle, est défini pour démarrer automatiquement au démarrage du système.<br/><br/> Vérifiez que les règles de pare-feu autorisent une connexion SSH à ce dernier.
**Machine virtuelle Azure exécutant Linux** | Machine virtuelle Azure après le basculement | Les règles des groupes de sécurité réseau figurant sur la machine virtuelle basculée (et le sous-réseau Azure auquel elle est connectée) doivent autoriser les connexions entrantes avec le port SSH.<br/><br/> [Ajoutez une adresse IP publique](/archive/blogs/srinathv/how-to-add-a-public-ip-address-to-azure-vm-for-vm-failed-over-using-asr) pour la machine virtuelle.<br/><br/> Cochez **Diagnostics de démarrage** pour obtenir une capture d’écran de la machine virtuelle.<br/><br/>

Suivez les étapes décrites [ici](site-recovery-failover-to-azure-troubleshoot.md) pour résoudre les problèmes de connectivité après le basculement.

## <a name="next-steps"></a>Étapes suivantes
Une fois que vous avez exécuté une simulation de récupération d’urgence, découvrez les autres types de [basculement](site-recovery-failover.md).