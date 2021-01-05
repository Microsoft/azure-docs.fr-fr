---
title: Informations sur les plans de récupération dans Azure Site Recovery
description: Apprenez-en plus sur les plans de récupération dans Azure Site Recovery.
ms.topic: conceptual
ms.date: 01/23/2020
ms.openlocfilehash: 8d191781cacc37242dd1be31d6cb87ef196e5e7a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84343912"
---
# <a name="about-recovery-plans"></a>À propos des plans de récupération

Cet article propose une vue d’ensemble des plans de récupération dans [Azure Site Recovery](site-recovery-overview.md).

Un plan de récupération rassemble les machines dans des groupes de récupération à des fins de basculement. Un plan de récupération vous permet de définir un processus de récupération systématique en créant de petites unités indépendantes que vous pouvez basculer. Une unité représente généralement une application dans votre environnement.

- Un plan de récupération définit comment les machines basculent et l’ordre dans lequel elles démarrent après un basculement.
- Vous pouvez utiliser des plans de récupération pour le basculement vers Azure et la restauration automatique à partir d’Azure.
- Vous pouvez ajouter jusqu’à 100 instances protégées à un même plan de récupération.
- Vous pouvez personnaliser un plan en y ajoutant un ordre, des instructions et des tâches.
- Lorsqu’un plan est défini, vous pouvez basculer vers celui-ci.
- Les machines peuvent être référencées dans plusieurs plans de récupération, dans lesquels les plans suivants ignoreront le déploiement/démarrage d’une machine si elle a été déployée au préalable à l’aide d’un autre plan de récupération.



### <a name="why-use-a-recovery-plan"></a>Pourquoi utiliser un plan de récupération ?

Utilisez des plans de récupération pour :

* Modéliser une application autour de ses dépendances.
* Automatisez les tâches de récupération pour réduire l’objectif de délai de récupération (RTO).
* Vérifiez que vous êtes prêt pour la migration ou la récupération d’urgence en vous assurant que vos applications font partie d’un plan de récupération.
* Exécutez des tests de basculement sur des plans de récupération, pour vous assurer que la récupération d’urgence ou la migration fonctionne comme prévu.


## <a name="model-apps"></a>Modéliser des applications 
Vous pouvez planifier et créer un groupe de récupération pour capturer les propriétés propres à l’application. Par exemple, prenons une application à trois niveaux classique avec un serveur SQL principal, un intergiciel et un serveur web frontal. En règle générale, vous personnalisez le plan de récupération afin que les machines de chaque niveau démarrent dans l’ordre approprié après un basculement.

- Le serveur SQL principal doit être le premier, l’intergiciel suivant et enfin le serveur web frontal.
- Cet ordre de démarrage garantit le bon fonctionnement de l’application au moment où la dernière machine démarre.
- Cet ordre permet de s’assurer que lorsque l’intergiciel démarre et tente de se connecter au niveau SQL Server, le niveau SQL Server est déjà en cours d’exécution. 
- Cet ordre permet également de s’assurer que les serveurs frontaux démarrent en dernier, de sorte que les utilisateurs finaux ne se connectent pas à l’URL de l’application avant que tous les composants ne soient opérationnels, et que l’application soit prête à accepter des requêtes.

Pour créer cet ordre, vous ajoutez des groupes au groupe de récupération et ajoutez des machines dans les groupes.
- Si un ordre est spécifié, un séquencement est utilisé. Les actions sont exécutées en parallèle le cas échéant afin d’améliorer le RTO de récupération de l’application.
- Les machines d’un même groupe basculent en parallèle.
- Les machines de groupes différents basculent dans l’ordre du groupe, de sorte que les machines du groupe 2 ne démarrent leur basculement que lorsque toutes les machines du groupe 1 ont basculé et démarré.

    ![Exemple de plan de récupération](./media/recovery-plan-overview/rp.png)

Avec cette personnalisation, voici ce qui se passe lorsque vous exécutez un basculement sur le plan de récupération : 

1. Une étape d’arrêt tente de mettre hors tension les machines locales. Une exception est que si vous exécutez un test de basculement, le site principal continue de s’exécuter. 
2. L’arrêt déclenche un basculement en parallèle de toutes les machines du plan de récupération.
3. Le basculement prépare les disques des machines virtuelles au moyen des données répliquées.
4. Les groupes de démarrage s’exécutent dans l’ordre et démarrent les machines dans chaque groupe. Le groupe 1 s’exécute en premier, puis le groupe 2 et enfin le groupe 3. S’il existe plusieurs machines dans un groupe, toutes les machines démarrent en parallèle.


## <a name="automate-tasks-in-recovery-plans"></a>Automatiser des tâches dans des plans de récupération

La récupération d’applications de grande taille peut être une tâche complexe. Les étapes manuelles du processus sont source d’erreurs et la personne qui exécute le basculement n’a peut-être pas connaissance de toutes les subtilités de l’application. Vous pouvez utiliser un plan de récupération pour imposer un ordre et automatiser les actions nécessaires à chaque étape, à l’aide de runbooks Azure Automation pour le basculement vers Azure, ou de scripts. Pour les tâches qui ne peuvent pas être automatisées, vous pouvez insérer des pauses pour des actions manuelles dans les plans de récupération. Vous pouvez configurer deux types de tâches :

* **Tâches sur la machine virtuelle Azure après le basculement** : quand vous basculez vers Azure, vous devez généralement effectuer des actions afin de pouvoir vous connecter à la machine virtuelle après le basculement. Par exemple : 
    * Créer une adresse IP publique sur la machine virtuelle Azure.
    * Associer un groupe de sécurité réseau à la carte réseau de la machine virtuelle Azure.
    * Ajouter un équilibreur de charge à un groupe à haute disponibilité
* **Tâches dans la machine virtuelle après le basculement** : ces tâches reconfigurent généralement l’application exécutée sur la machine afin qu’elle continue à fonctionner correctement dans le nouvel environnement. Par exemple :
    * Modifier la chaîne de connexion de base de données dans la machine.
    * Modifier les règles ou la configuration du serveur web


### <a name="run-a-test-failover-on-recovery-plans"></a>Exécuter un test de basculement sur des plans de récupération

Vous pouvez utiliser un plan de récupération pour déclencher un test de basculement. Utilisez les meilleures pratiques suivantes :

- Effectuez toujours un test de basculement sur une application avant de procéder à un basculement complet. Les tests de basculement vous permettent de vérifier le démarrage de l’application sur le site de récupération.
- Si vous pensez avoir manqué quelque chose, procédez à un nettoyage, puis réexécutez le test de basculement. 
- Exécutez plusieurs fois un test de basculement jusqu’à ce que vous soyez sûr que l’application est correctement récupérée.
- Chaque application étant unique, vous devez créer des plans de récupération personnalisés pour chacune et exécuter un test de basculement sur chacune.
- Les applications et leurs dépendances changent fréquemment. Pour vous assurer que les plans de récupération sont à jour, exécutez un test de basculement pour chaque application chaque trimestre.

    ![Capture d’écran d’un exemple de test de plan de récupération dans Site Recovery](./media/recovery-plan-overview/rptest.png)

## <a name="watch-a-recovery-plan-video"></a>Regarder une vidéo sur un plan de récupération

Regardez une courte vidéo d’exemple Montrant un basculement en un clic pour un plan de récupération, pour une application WordPress à deux niveaux.
    
> [!VIDEO https://channel9.msdn.com/Series/Azure-Site-Recovery/One-click-failover-of-a-2-tier-WordPress-application-using-Azure-Site-Recovery/player]



## <a name="next-steps"></a>Étapes suivantes

- [Créer](site-recovery-create-recovery-plans.md) un plan de récupération.
- [Exécuter](site-recovery-failover.md) des basculements. 
