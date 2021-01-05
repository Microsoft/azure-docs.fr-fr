---
title: Notifications de maintenance
description: Vue d’ensemble des notifications maintenance pour les machines virtuelles s’exécutant dans Azure.
author: shants123
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 8/12/2020
ms.author: shants
ms.openlocfilehash: 53cde1178a4faae0fbd11222e4219f70be29145d
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94560806"
---
# <a name="handling-planned-maintenance-notifications"></a>Gestion des notifications de maintenance planifiée

Azure exécute régulièrement des mises à jour afin d’améliorer la fiabilité, les performances et la sécurité de l’infrastructure hôte des machines virtuelles. Les mises à jour sont des modifications telles que la mise à jour corrective de l’environnement d’hébergement ou la mise à niveau et la désactivation de matériel. Une majorité de ces mises à jour sont effectuées sans incidence sur les machines virtuelles hébergées. Cependant, il existe des cas où les mises à jour ont un impact :

- Si la maintenance ne nécessite pas de redémarrage, Azure met en pause la machine virtuelle pour quelques secondes pendant la mise à jour de l’hôte. Ces types d’opérations de maintenance sont appliquées domaine d’erreur par domaine d’erreur. Elles sont arrêtées si des signaux d’avertissement sont reçus.

- Si la maintenance nécessite un redémarrage, une notification vous dira pour quand est prévue la maintenance. Vous disposez d’une fenêtre de temps données d’environ 35 jours pour commencer la maintenance vous-même, au moment qui vous convient.


La maintenance planifiée nécessitant un redémarrage s’effectue par vagues. Chaque vague a une portée différente (régions).

- Une vague commence par une notification aux clients. Par défaut, la notification est envoyée à l’administrateur et aux coadministrateurs de l’abonnement. Vous pouvez ajouter plusieurs destinataires et options de messagerie, telles que les e-mails, des SMS et des webhooks, en utilisant des [Alertes de journal d’activité](../service-health/alerts-activity-log-service-notifications-portal.md).  
- Quand une notification est envoyée, une *fenêtre en libre-service* est rendue disponible. Durant cette fenêtre, vous pouvez interroger les machines virtuelles affectées et démarrer une maintenance en fonction de vos besoins en matière de planification. La fenêtre en libre-service est généralement d’environ 35 jours.
- Après la fenêtre de libre-service, une *fenêtre de maintenance planifiée* apparaît. Au cours de cette fenêtre, Azure planifie et applique la maintenance requise à votre machine virtuelle. 

L’objectif de ces deux fenêtres est de vous donner suffisamment de temps pour commencer la maintenance et redémarrer votre machine virtuelle tout en sachant à quel moment Azure démarrera automatiquement la maintenance.


Vous pouvez utiliser le portail Azure, PowerShell, les API REST et l’interface de ligne de commande pour demander les fenêtres de maintenance pour vos machines virtuelles et démarrer la maintenance en libre-service.

 
## <a name="should-you-start-maintenance-using-during-the-self-service-window"></a>Devez-vous démarrer la maintenance pendant la fenêtre de libre-service ?  

Les instructions suivantes doivent vous aider à décider si vous devez utiliser cette fonctionnalité et démarrer la maintenance à votre rythme. 

> [!NOTE] 
> La maintenance de libre-service n’est peut-être pas disponible pour toutes vos machines virtuelles. Pour déterminer si le redéploiement proactif est disponible pour votre machine virtuelle, recherchez **Démarrer maintenant** dans l’état de maintenance. La maintenance de libre-service n’est pas disponible pour les Services cloud (rôle de travail/web) et Service Fabric.


Une maintenance en libre-service n’est pas recommandée pour des déploiements utilisant des **groupes à haute disponibilité**. Les groupes à haute disponibilité ne sont déjà mis à jour qu’un domaine de mise à jour à la fois. 

- Laissez Azure déclencher la maintenance. Une maintenance nécessitant un redémarrage est effectuée domaine de mise à jour par domaine de mise à jour. Les domaines de mise à jour ne reçoivent pas nécessairement la maintenance de manière séquentielle, et il y a une pause de 30 minutes entre les domaines de mise à jour. 
- Si une perte temporaire de capacité (1 domaine de mise à jour) pose problème, vous pouvez ajouter des instances durant la période de maintenance. 
- Dans le cas d’une maintenance qui ne nécessite pas un redémarrage, les mises à jour sont appliquées au niveau du domaine d’erreur. 

**Ne pas** utiliser la maintenance de libre-service dans les scénarios suivants : 
- Si vous arrêtez vos machines virtuelles fréquemment, soit manuellement, à l’aide de DevTest Labs, en utilisant l’arrêt automatique ou en suivant une planification, cela peut rétablir l’état de maintenance et donc entraîner un temps d’arrêt supplémentaire.
- Sur les machines virtuelles à durée de vie limitée dont vous savez qu’elles seront supprimées avant la fin de la vague de maintenance. 
- Pour les charges de travail avec un état volumineux stockées dans le disque local (éphémère) qui doivent être maintenues lors de la mise à jour. 
- Dans les cas où vous redimensionnez souvent votre machine virtuelle, comme cela peut rétablir l’état de maintenance. 
- Si vous avez adopté des événements planifiés qui permettent un basculement proactif ou l’arrêt approprié de votre charge de travail, 15 minutes avant le début de l’arrêt de la maintenance

**Utilisez** la maintenance de libre-service, si vous envisagez d’exécuter votre machine virtuelle sans interruption pendant la phase de maintenance planifiée et qu’aucune des contre-indications mentionnées ci-dessus n’est applicable. 

Il est conseillé d’utiliser la maintenance de libre-service dans les cas suivants :
- Vous devez communiquer une fenêtre de maintenance exacte à votre administration ou à votre client final. 
- Vous devez réaliser la maintenance pour une date donnée. 
- Vous devez contrôler la séquence de maintenance, par exemple, les applications multiniveaux pour garantir la récupération sans échec.
- Plus de 30 minutes de temps de récupération de machine virtuelle sont nécessaires entre deux domaines de mise à jour. Pour contrôler le délai entre les domaines de mise à jour, vous devez déclencher la maintenance sur vos machines virtuelles, un domaine de mise à jour (UD) à la fois.


## <a name="faq"></a>Questions fréquentes (FAQ)


**Q : Pourquoi devez-vous redémarrer mes machines virtuelles maintenant ?**

**R :** Bien que la majorité des mises à jour et des mises à niveau vers la plateforme Azure n’impactent pas la disponibilité des machines virtuelles, parfois, nous ne pouvons pas éviter le redémarrage des machines virtuelles hébergées dans Azure. Nous avons accumulé plusieurs modifications qui nous obligent à redémarrer nos serveurs, ce qui entraînera le redémarrage des machines virtuelles.

**Q : Si je suis vos recommandations concernant la haute disponibilité en utilisant un groupe à haute disponibilité, mon environnement est-il sécurisé ?**

**R :** Les machines virtuelles déployées dans un groupe à haute disponibilité ou dans des groupes de machines virtuelles identiques incluent la notion de domaines de mise à jour (UD). Lors de la réalisation de la maintenance, Azure respecte la contrainte de domaine de mise à jour et ne redémarre pas les machines virtuelles à partir de différents domaines de mise à jour (dans le même groupe à haute disponibilité).  Azure attend également au moins 30 minutes avant de passer au groupe de machines virtuelles suivant. 

Pour plus d’informations sur la haute disponibilité, consultez [Disponibilité des machines virtuelles dans Azure](availability.md).

**Q : Comment être averti d’une maintenance planifiée ?**

**R :** Une vague d’opérations de maintenance planifiée commence par une planification sur une ou plusieurs régions Azure. Peu après, une notification par e-mail est envoyée aux administrateurs, aux coadministrateurs, aux propriétaires et aux contributeurs d’abonnement (un e-mail par abonnement). Il est possible de configurer des canaux et des destinataires supplémentaires pour cette notification à l’aide de la fonctionnalité Alertes de journal d’activité. Dans le cas où vous déployez une machine virtuelle dans une région où la planification de la maintenance est déjà effectuée, vous ne recevez pas la notification. Au lieu de cela, vous devez vérifier l’état de maintenance de la machine virtuelle.

**Q : Je ne vois aucune indication de maintenance planifiée dans le portail, dans PowerShell ou dans l’interface CLI. D’où vient le problème ?**

**R :** Les informations relatives à la maintenance planifiée sont disponibles pendant une vague d’opérations de maintenance planifiées uniquement pour les machines virtuelles qu’elle impacte. En d’autres termes, si vous ne voyez pas de données, il est possible que la vague d’opérations de maintenance soit déjà terminée (ou qu’elle n’ait pas démarré), ou bien que votre machine virtuelle soit déjà hébergée sur un serveur mis à jour.

**Q : Est-il possible de savoir exactement quand ma machine virtuelle sera impactée ?**

**R :** Pendant la définition de la planification, nous définissons une fenêtre de temps de plusieurs jours. Toutefois, le séquencement exact des serveurs (et des machines virtuelles) au sein de cette fenêtre est inconnu. Les clients qui souhaitent connaître l’heure exacte pour leurs machines virtuelles peuvent utiliser des [événements planifiés](./linux/scheduled-events.md), faire une requête à partir de la machine virtuelle et recevoir une notification 15 minutes avant le redémarrage d’une machine virtuelle.

**Q : Combien de temps vous faut-il pour redémarrer ma machine virtuelle ?**

**R :**  En fonction de la taille de votre machine virtuelle, le redémarrage peut prendre plusieurs minutes pendant la fenêtre de maintenance libre-service. Tout redémarrage lancé par Azure pendant la fenêtre de maintenance planifiée dure habituellement 25 minutes environ. Notez que dans le cas où vous utilisez des services cloud (rôle de travail/web), Virtual Machine Scale Sets ou des groupes à haute disponibilité, vous disposez de 30 minutes entre chaque groupe de machines virtuelles (UD) pendant la fenêtre de maintenance planifiée.

**Q : Comment ça se passe avec Virtual Machine Scale Sets ?**

**R :** La maintenance planifiée est désormais disponible pour Virtual Machine Scale Sets. Pour obtenir des instructions sur la façon de lancer une maintenance en libre-service, consultez le document [Maintenance planifiée des groupes de machines virtuelles identiques](../virtual-machine-scale-sets/virtual-machine-scale-sets-maintenance-notifications.md).

**Q : Comment ça se passe avec les services cloud (rôle de travail/web) et Service Fabric ?**

**R :** Alors que ces plateformes sont affectées par une maintenance planifiée, les clients qui utilisent ces plateformes sont considérés comme sécurisés étant donné que seules les machines virtuelles d’un domaine de mise à niveau (UD) seront affectées à un moment donné. La maintenance de libre-service n’est pas disponible pour les Services cloud (rôle de travail/web) et Service Fabric.

**Q : Je ne vois aucune information de maintenance sur mes machines virtuelles. Quelle est la cause du problème ?**

**R :** Plusieurs raisons peuvent expliquer pourquoi vous ne voyez aucune information de maintenance sur vos machines virtuelles :
1.  Vous utilisez un abonnement marqué comme interne à Microsoft.
2.  Vos machines virtuelles ne sont pas planifiées pour la maintenance. Il est possible que la vague d’opérations de maintenance soit terminée, annulée ou modifiée, de sorte que celle-ci n’impacte plus vos machines virtuelles.
3. Vous avez désalloué la machine virtuelle, puis vous l’avez démarrée. Cela peut entraîner le déplacement de la machine virtuelle vers un emplacement pour lequel aucune vague de maintenance n’est planifiée. La machine virtuelle n’affichera plus les informations de maintenance. 
4.  La colonne **Maintenance** n’a pas été ajoutée à l’affichage de liste de votre machine virtuelle. Alors que nous avons ajouté cette colonne à la vue par défaut, les clients qui ont effectué une configuration pour afficher des colonnes non définies par défaut doivent ajouter manuellement la colonne **Maintenance** à la vue liste de leurs machines virtuelles.

**Q : La maintenance de ma machine virtuelle est planifiée pour la deuxième fois. Pourquoi ?**

**R :** Il existe plusieurs cas d’utilisation où vous verrez votre machine virtuelle planifiée pour la maintenance après avoir déjà effectué votre maintenance-redéploiement :
1.  Nous avons annulé la vague d’opérations de maintenance et l’avons redémarrée avec une charge utile différente. Il est possible que nous ayons détecté une charge utile ayant généré une erreur et nous voulons simplement déployer une charge utile supplémentaire.
2.  Votre machine virtuelle a été *réparée par service* sur un autre nœud en raison d’une panne matérielle.
3.  Vous avez choisi d’arrêter (de libérer) et de redémarrer la machine virtuelle.
4.  Vous avez activé **Arrêt automatique** pour la machine virtuelle.



## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également gérer la maintenance planifiée à l’aide d’[Azure CLI](maintenance-notifications-cli.md), d’[Azure PowerShell](maintenance-notifications-powershell.md) ou du [portail](maintenance-notifications-portal.md).
