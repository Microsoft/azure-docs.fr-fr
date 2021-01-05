---
title: Concevoir des services disponibles dans le monde entier
description: Découvrez comment concevoir des applications pour des services hautement disponibles à l’aide d’Azure SQL Database.
keywords: récupération d’urgence cloud, solutions de récupération d’urgence, sauvegarde de données d’application, géo-réplication, planification de la continuité des activités
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: sstein
ms.date: 07/28/2020
ms.openlocfilehash: be632ba06edc858e7eadcd6e57a4f7769f69f2cb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91321677"
---
# <a name="designing-globally-available-services-using-azure-sql-database"></a>Conception de services disponibles à l’échelle mondiale à l’aide d’Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Pour créer et déployer des services cloud avec Azure SQL Database, utilisez une [géoréplication active](active-geo-replication-overview.md) ou des [groupes de basculement automatique](auto-failover-group-overview.md) pour fournir une tolérance aux pannes régionales et aux défaillances graves. La même fonctionnalité vous permet de créer des applications distribuées mondialement et optimisées pour l’accès local aux données. Cet article aborde les modèles d’application courants et présente les avantages et inconvénients de chacun d’eux.

> [!NOTE]
> Si vous utilisez des bases de données et des pools élastiques Premium ou Critique pour l’entreprise, vous pouvez les rendre résistants aux pannes régionales en les transformant en configuration de déploiement redondante dans une zone. Consultez [Bases de données redondantes interzone](high-availability-sla.md).  

## <a name="scenario-1-using-two-azure-regions-for-business-continuity-with-minimal-downtime"></a>Scénario 1 : Utilisation de deux régions Azure pour la continuité d’activité avec temps d’arrêt minimal

Dans ce scénario, les applications ont les caractéristiques suivantes :

* Chaque application est active dans une région Azure
* Toutes les sessions de base de données nécessitent un accès aux données en lecture et écriture
* La couche Web et la couche Données doivent être colocalisées pour réduire la latence et les coûts liés au trafic
* Les temps d’arrêt représentent un plus grand risque pour ces applications que la perte de données

Dans ce cas, la topologie de déploiement d’applications est optimisée pour la gestion des sinistres régionaux lorsque tous les composants d’application doivent être basculés en même temps. Le diagramme ci-dessous illustre cette topologie. Pour assurer la géoredondance, les ressources de l’application sont déployées dans la région A et la région B. Toutefois, les ressources de la région B sont utilisées seulement en cas de défaillance de la région A. Un groupe de basculement est configuré entre les deux régions pour gérer le basculement, la réplication et la connectivité à la base de données. Le service web des deux régions est configuré pour accéder à la base de données via l’écouteur en lecture-écriture **&lt;nom-groupe-basculement&gt;.database.windows.net** (1). Azure Traffic Manager est configuré pour utiliser la [méthode de routage prioritaire](../../traffic-manager/traffic-manager-configure-priority-routing-method.md) (2).  

> [!NOTE]
> [Azure Traffic Manager](../../traffic-manager/traffic-manager-overview.md) est utilisé dans cet article aux fins d’illustration uniquement. Vous pouvez utiliser toute solution d’équilibrage de charge qui prend en charge la méthode de routage prioritaire.

Le diagramme suivant illustre cette configuration avant une panne :

![Scénario 1 Configuration avant la panne.](./media/designing-cloud-solutions-for-disaster-recovery/scenario1-a.png)

Après une panne dans la région primaire, SQL Database détecte que la base de données primaire n’est pas accessible et déclenche un basculement vers la région secondaire conformément aux paramètres de la stratégie de basculement automatique (1). Selon le contrat de niveau de service (SLA) de votre application, vous pouvez choisir de configurer une période de grâce entre la détection de la panne et le basculement proprement dit. Il est possible qu’Azure Traffic Manager démarre le basculement de point de terminaison avant que le groupe de basculement ne déclenche le basculement de la base de données. Dans ce cas, l’application web ne peut pas se reconnecter immédiatement à la base de données. Toutefois, une reconnexion automatique est effectuée dès que le basculement de la base de données est terminé. Lorsque la région défaillante est restaurée et de nouveau en ligne, l’ancienne région primaire se reconnecte automatiquement en tant que nouvelle région secondaire. Le diagramme ci-dessous illustre la configuration après le basculement.

> [!NOTE]
> Toutes les transactions validées après le basculement sont perdues lors de la reconnexion. Une fois le basculement terminé, l’application de la région B est en mesure de se reconnecter et de redémarrer le traitement des demandes utilisateur. L’application web et la base de données primaire se trouvent maintenant dans la région B et restent colocalisées.

![Scénario 1 Configuration après le basculement](./media/designing-cloud-solutions-for-disaster-recovery/scenario1-b.png)

Si une panne se produit dans la région B, le processus de réplication entre la base de données primaire et la base de données secondaire est suspendu, mais le lien entre les deux reste intact (1). Le trafic géré détecte que la connectivité à la Région B est interrompue et marque l’application web de point de terminaison 2 comme étant Détériorée (2). Les performances de l’application ne sont pas impactées dans ce cas, mais la base de données devient exposée, et par conséquent, est soumise à un risque plus élevé de perte de données en cas de défaillance de la région A.

> [!NOTE]
> Pour une récupération d’urgence, nous recommandons la configuration dans laquelle le déploiement de l’application est limité à deux régions. En effet, la plupart des zones géographiques Azure comptent seulement deux régions. Cette configuration ne protège pas votre application d’une défaillance grave simultanée des deux régions. Dans le cas peu probable d’une telle défaillance, vous pouvez restaurer vos bases de données dans une région tierce à l’aide de [l’opération de géo-restauration](disaster-recovery-guidance.md#recover-using-geo-restore).
>

 Une fois la panne atténuée, la base de données secondaire est automatiquement resynchronisée avec la base de données primaire. Pendant la synchronisation, les performances de la base de données principale peuvent être impactées. L’impact dépend de la quantité de données que la nouvelle base de données primaire a acquises depuis le basculement. 

> [!NOTE]
> Une fois la panne atténuée, Traffic Manager commencera à acheminer les connexions vers l’application dans la région A en tant que point de terminaison de priorité plus élevée. Si vous envisagez de conserver la base de donnée principale dans la région B pendant un certain temps, vous devez modifier la table de priorité dans le profil Traffic Manager en conséquence. 
>
 
 Le diagramme suivant illustre une panne dans la région secondaire :

![Scénario 1 Configuration après une panne dans la région secondaire.](./media/designing-cloud-solutions-for-disaster-recovery/scenario1-c.png)

Les **avantages** clés de ce modèle de conception sont les suivants :

* La même application web est déployée dans les deux régions sans configuration régionale, ni logique supplémentaire pour gérer le basculement.
* Les performances de l’application ne sont pas impactées par le basculement, car l’application web et la base de données sont toujours colocalisées.

Le principal **inconvénient** à cela est que les ressources de l’application dans la région B sont, la plupart du temps, sous-utilisées.

## <a name="scenario-2-azure-regions-for-business-continuity-with-maximum-data-preservation"></a>Scénario 2 : Régions Azure pour la continuité d’activité avec conservation maximale des données

Cette option est idéale pour les applications dotées des caractéristiques suivantes :

* Toute perte de données constitue un risque élevé pour l’entreprise. Le basculement de la base de données ne peut être utilisé qu’en dernier recours, si la panne est due à une défaillance grave.
* L’application prend en charge les opérations en modes lecture seule et lecture-écriture et peut fonctionner en mode lecture seule pendant une période donnée.

Dans ce modèle, l’application bascule vers le mode lecture seule lorsque les connexions de lecture-écriture commencent à recevoir des erreurs d’expiration du délai d’attente. L’application web est déployée dans les deux régions et inclut une connexion au point de terminaison de l’écouteur en lecture-écriture et une autre connexion au point de terminaison de l’écouteur en lecture seule (1). Le profil Traffic Manager doit utiliser le [routage prioritaire](../../traffic-manager/traffic-manager-configure-priority-routing-method.md). La [surveillance des points de terminaison](../../traffic-manager/traffic-manager-monitoring.md) doit être activée pour le point de terminaison de l’application dans chaque région (2).

Le diagramme suivant illustre cette configuration avant une panne :

![Scénario 2 Configuration avant la panne.](./media/designing-cloud-solutions-for-disaster-recovery/scenario2-a.png)

Si Traffic Manager détecte une défaillance de connectivité pour la région A, il bascule automatiquement le trafic utilisateur vers l’instance de l’application dans la région B. Avec ce modèle, il est important de définir la période de grâce en cas de risque de perte de données sur une valeur suffisamment élevée, par exemple 24 heures. Vous empêcherez ainsi toute perte de données si la panne est résolue durant ce laps de temps. Lorsque l’application web de la région B est activée, les opérations en lecture-écriture se mettent à échouer. À ce stade, elle doit basculer vers le mode lecture seule (1). Avec ce mode, les requêtes sont automatiquement acheminées vers la base de données secondaire. Si la panne est due à une défaillance grave, elle ne pourra très probablement pas être résolue pendant la période de grâce. Lorsque cette période expire, le groupe de basculement déclenche le basculement. Ensuite, l’écouteur en lecture-écriture redevient disponible et les connexions à l’écouteur cessent d’échouer (2). Le diagramme suivant illustre les deux étapes du processus de récupération.

> [!NOTE]
> Si la panne qui affecte la région primaire est résolue avant la fin de la période de grâce, Traffic Manager détecte la restauration de la connectivité dans la région primaire et rebascule le trafic utilisateur vers l’instance de l’application dans la région A. Cette instance de l’application reprend et fonctionne en mode lecture-écriture en s’appuyant sur la base de données primaire de la région A, comme illustré dans le diagramme précédent.

![Scénario 2 Étapes de la récupération d’urgence.](./media/designing-cloud-solutions-for-disaster-recovery/scenario2-b.png)

Si une panne se produit dans la région B, Traffic Manager détecte la défaillance de l’application du point de terminaison 2 dans la région B et la marque comme étant détériorée (1). Entre-temps, le groupe de basculement bascule l’écouteur en lecture seule vers la région A (2). Cette panne n’a pas d’impact sur l’expérience des utilisateurs finaux. Toutefois, la base de données primaire est exposée lors de la panne. Le diagramme suivant illustre une panne dans la région secondaire :

![Scénario 2 Panne dans la région secondaire.](./media/designing-cloud-solutions-for-disaster-recovery/scenario2-c.png)

Une fois la panne résolue, la base de données secondaire est immédiatement synchronisée avec la base de données primaire l’écouteur de lecture seule est rebasculé vers la base de données secondaire de la région B. Lors de la synchronisation, les performances de la base de données primaire peuvent être légèrement affectées selon la quantité de données à synchroniser.

Ce modèle de conception offre plusieurs **avantages**:

* Il évite la perte de données pendant les pannes temporaires.
* Le temps d’arrêt dépend uniquement de la vitesse selon laquelle Traffic Manager détecte le problème de connectivité, paramètre configurable.

**L’inconvénient** est que l’application doit être en mesure de fonctionner en lecture seule.

## <a name="scenario-3-application-relocation-to-a-different-geography-without-data-loss-and-near-zero-downtime"></a>Scénario 3 : Déplacement de l’application vers une autre zone géographique sans perte de données et quasiment sans temps d’arrêt

Dans ce scénario, l’application a les caractéristiques suivantes :

* Les utilisateurs finaux accèdent à l’application à partir de différentes zones géographiques
* L’application inclut des charges de travail en lecture seule qui ne dépendent pas d’une synchronisation complète avec les dernières mises à jour
* L’accès en écriture aux données doit être pris en charge dans la même zone géographique pour la majorité des utilisateurs
* La latence de lecture est essentielle pour l’expérience des utilisateurs finaux

Afin de remplir ces conditions, vous devez faire en sorte que l’appareil de l’utilisateur se connecte **toujours** à l’application déployée dans la même zone géographique pour les opérations en lecture seule, telles que l’exploration de données, l’analytique etc. Quant aux opérations OLTP, elles sont traitées dans la même zone géographique **la plupart du temps**. Par exemple, durant la journée, les opérations OLTP sont traitées dans la même zone géographique, mais pendant les heures creuses, elles peuvent être traitées dans une zone géographique différente. Si l’activité de l’utilisateur final a lieu principalement pendant les heures de travail, vous pouvez garantir des performances optimales pour la majorité des utilisateurs la plupart du temps. Le diagramme qui suit montre cette topologie :

Les ressources de l’application doivent être déployées dans chaque zone géographique pour laquelle vous avez une demande importante. Par exemple, si votre application est activement utilisée aux États-Unis, en Europe et en Asie Sud-Est, l’application doit être déployée sur l’ensemble de ces zones géographiques. La base de données primaire doit être basculée dynamiquement d’une zone géographique à la suivante à la fin des heures de travail. Cette méthode est appelée « follow the sun » (suivre le soleil). La charge de travail OLTP se connecte toujours à la base de données via l’écouteur en lecture-écriture **&lt;nom-groupe-basculement&gt;.database.windows.net** (1). La charge de travail en lecture seule se connecte directement à la base de données locale à l’aide du point de terminaison de serveur de bases de données **&lt;nom-serveur&gt;.database.windows.net** (2). Traffic Manager est configuré avec la [méthode de routage de performances](../../traffic-manager/traffic-manager-configure-performance-routing-method.md). Ainsi, l’appareil de l’utilisateur final est connecté au service web de la région la plus proche. L’analyse des points de terminaison doit être activée dans Traffic Manager pour chaque point de terminaison de service web (3).

> [!NOTE]
> La configuration du groupe de basculement définit la région qui est utilisée pour le basculement. Étant donné que la nouvelle base de données primaire se trouve dans une zone géographique différente, le basculement entraîne une latence plus longue pour les charges de travail OLTP et les charges en lecture seule, jusqu’à ce que la région impactée soit de nouveau en ligne.

![Scénario 3 Configuration de la base de données primaire pour la région USA Est.](./media/designing-cloud-solutions-for-disaster-recovery/scenario3-a.png)

À la fin de la journée (par exemple à 23 h, heure locale), les bases de données actives doivent être basculées vers la région suivante (Europe Nord). Cette tâche peut être entièrement automatisée à l’aide d’[Azure Logic Apps](../../logic-apps/logic-apps-overview.md). Cette tâche implique les étapes suivantes :

* Basculer le serveur principal du groupe de basculement vers la région Europe Nord à l’aide d’un basculement convivial (1)
* Supprimer le groupe de basculement situé entre la région USA Est et la région Europe Nord
* Créer un nouveau groupe de basculement portant le même nom, mais entre la région Europe Nord et la région Asie Est (2)
* Ajouter la base de données primaire de la région Europe Nord et la base de données secondaire de la région Asie Est à ce groupe de basculement (3)

Le diagramme suivant illustre la nouvelle configuration après le basculement planifié :

![Scénario 3 Transition de la base de données primaire vers la région Europe Nord.](./media/designing-cloud-solutions-for-disaster-recovery/scenario3-b.png)

Si une panne se produit en Europe Nord, par exemple, le basculement automatique de la base de données est démarré par le groupe de basculement, ce qui entraîne le déplacement de l’application vers la région suivante en avance (1).  Dans ce cas, la région USA Est est la seule région secondaire restante jusqu’à ce que la région Europe Nord soit de nouveau en ligne. Les deux zones restantes servent les clients des trois régions géographiques en échangeant leur rôle. Azure Logic Apps doit être réglé en conséquence. Étant donné que les régions restantes reçoivent du trafic utilisateur supplémentaire en provenance d’Europe, les performances de l’application sont perturbées non seulement par une latence supplémentaire, mais également par un plus grand nombre de connexions d’utilisateurs finaux. Une fois la panne résolue dans la région Europe Nord, la base de données secondaire est immédiatement synchronisée avec l’actuelle base de données primaire. Le diagramme suivant illustre une panne dans la région Europe Nord :

![Scénario 3 Panne dans la région Europe Nord.](./media/designing-cloud-solutions-for-disaster-recovery/scenario3-c.png)

> [!NOTE]
> Vous pouvez réduire le temps pendant lequel l’expérience des utilisateurs de la région en Europe est détériorée par la longue latence. Pour ce faire, vous devez déployer proactivement une copie de l’application et créer la ou les bases de données secondaires dans une autre région locale (Europe Ouest) en remplacement de l’instance d’application hors connexion de la région Europe Nord. Lorsque celle-ci est de nouveau en ligne, vous pouvez décider si vous souhaitez continuer à utiliser la région Europe Ouest, ou supprimer la copie de l’application qui s’y trouve et recommencer à utiliser la région Europe Nord.

Cette conception présente plusieurs **avantages** :

* La charge de travail de l’application en lecture seule accède aux données de la région la plus proche à tout moment.
* La charge de travail de l’application en lecture-écriture accède aux données de la région la plus proche pendant la période d’activité la plus élevée dans chaque zone géographique.
* Étant donné que l’application est déployée dans plusieurs régions, elle peut survivre à une perte de l’une des régions sans temps d’arrêt significatif.

Cela présente toutefois quelques **inconvénients** :

* En cas de panne régionale, la zone géographique est impactée par une plus longue latence. Les charges de travail en lecture-écriture et en lecture seule sont prises en charge par l’application dans une zone géographique différente.
* Les charges de travail en lecture seule doivent se connecter à un autre point de terminaison dans chaque région.

## <a name="business-continuity-planning-choose-an-application-design-for-cloud-disaster-recovery"></a>Planification de la continuité d’activité : choisir une conception d’application pour la récupération d’urgence cloud

Votre stratégie de récupération d’urgence cloud spécifique peut combiner ou étendre ces modèles de conception afin de mieux répondre aux besoins de votre application.  Comme mentionné précédemment, la stratégie que vous choisissez est basée sur le contrat de niveau de service que vous souhaitez proposer à vos clients et sur la topologie de déploiement d’applications. Pour vous aider à prendre votre décision, le tableau suivant compare les choix en fonction de l’objectif de point de récupération (RPO) et du temps de récupération estimé (ERT).

| Modèle | RPO | ERT |
|:--- |:--- |:--- |
| Déploiement actif / passif pour la récupération d’urgence avec accès à la base de données colocalisée |Accès en lecture-écriture < 5 s |Temps de détection des défaillances + TTL du DNS |
| Déploiement actif / actif pour l’équilibrage de charge d’applications |Accès en lecture-écriture < 5 s |Temps de détection des défaillances + TTL du DNS |
| Déploiement actif / passif pour la conservation des données |Accès en lecture seule < 5 s | Accès en lecture seule = 0 |
||Accès en lecture-écriture = 0 | Accès en lecture-écriture = temps de détection de défaillances + période de grâce en cas de risque de perte de données |
|||

## <a name="next-steps"></a>Étapes suivantes

* Pour une vue d’ensemble de la continuité des activités et des scénarios, consultez [Vue d’ensemble de la continuité des activités](business-continuity-high-availability-disaster-recover-hadr-overview.md)
* Pour plus d’informations sur la géoréplication active, voir la section [Géoréplication active](active-geo-replication-overview.md).
* Pour plus d’informations sur les groupes de basculement automatique, voir [Groupes de basculement automatique](auto-failover-group-overview.md).
* Pour plus d’informations sur la géoréplication active avec des pools élastiques, consultez [Stratégies de récupération d’urgence de pool élastique](disaster-recovery-strategies-for-applications-with-elastic-pool.md).
