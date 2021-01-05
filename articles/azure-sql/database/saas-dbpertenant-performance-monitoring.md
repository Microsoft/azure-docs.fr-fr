---
title: 'Application SaaS : Monitorage des performances de nombreuses bases de données'
description: Superviser et gérer les performances d’Azure SQL Database dans une application SaaS multilocataire
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/25/2019
ms.openlocfilehash: 21c0a7a3fe6d5be9d99ea53dbfa74cf72e163272
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92780663"
---
# <a name="monitor-and-manage-performance-of-azure-sql-database-in-a-multi-tenant-saas-app"></a>Superviser et gérer les performances d’Azure SQL Database dans une application SaaS multilocataire
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Ce didacticiel aborde plusieurs scénarios de gestion de performance clés utilisés dans les applications SaaS. Les fonctionnalités intégrées de surveillance et d’alerte de base de données de SQL Database, ainsi que les pools élastiques sont illustrés à l’aide d’un générateur de charge destiné à simuler l’activité de toutes les bases de données client.

L’application de base de données Wingtip Tickets SaaS par client utilise un modèle de données par locataire, où chaque lieu (locataire) possède sa propre base de données. Comme de nombreuses applications SaaS, le modèle de charge de travail de locataire anticipé est imprévisible et sporadique. En d’autres termes, les ventes de tickets peuvent se produire à tout moment. Pour tirer parti de ce modèle d’utilisation de base de données typique, les bases de données de locataire sont déployées dans des pools élastiques. Les pools élastiques optimisent le coût d’une solution en partageant des ressources entre de nombreuses bases de données. Avec ce type de modèle, il est important de surveiller l’utilisation des ressources des bases de données et des pools pour veiller à ce que les charges soient raisonnablement équilibrées entre les pools. Vous devez également vous assurer que les bases de données ont des ressources appropriées et que les pools n’atteignent pas les limites d’[eDTU](purchasing-models.md#dtu-based-purchasing-model). Ce didacticiel explore plusieurs moyens de surveiller et de gérer des bases de données et des pools et montre comment prendre des mesures correctives en réponse aux variations de la charge de travail.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> 
> * Simuler l’utilisation sur les bases de données de locataire en exécutant un générateur de charge fourni
> * Surveiller les bases de données de locataire à mesure qu’elles répondent à l’augmentation de la charge
> * Augmenter la taille du pool élastique en réponse à la charge accrue sur les bases de données
> * Approvisionner un deuxième pool élastique pour équilibrer la charge de l’activité des bases de données


Pour suivre ce didacticiel, vérifiez que les prérequis suivants sont remplis :

* L’application de base de données Wingtip Tickets SaaS par client est déployée. Pour procéder à un déploiement en moins de cinq minutes, consultez la page [Déployer et explorer l’application de base de données Wingtip Tickets SaaS par client](./saas-dbpertenant-get-started-deploy.md)
* Azure PowerShell est installé. Pour plus d’informations, voir [Bien démarrer avec Azure PowerShell](/powershell/azure/get-started-azureps).

## <a name="introduction-to-saas-performance-management-patterns"></a>Présentation des modèles de gestion de la performance SaaS

La gestion des performances des bases de données se compose des opérations suivantes : compilation et analyse des données de performances, puis réaction à ces données en ajustant les paramètres afin de conserver un temps de réponse acceptable pour votre application. Lorsque vous hébergez plusieurs locataires, les pools élastiques sont un moyen économique pour fournir et gérer des ressources d’un groupe de bases de données avec des charges de travail imprévisibles. Avec certains modèles de charge de travail, il peut être avantageux de gérer ne serait-ce que 2 bases de données S3 dans un pool.

![Diagramme de l’application](./media/saas-dbpertenant-performance-monitoring/app-diagram.png)

Les pools et les bases de données qu’ils incluent doivent être surveillés pour vérifier qu’ils restent dans les limites de performances acceptables. Paramétrez la configuration du pool pour répondre aux besoins de la charge de travail agrégée de toutes les bases de données, en faisant en sorte que les eDTU du pool soient adaptés à la charge de travail globale. Ajustez les valeurs d’eDTU minimales et maximales par base de données en fonction des besoins spécifiques de votre application.

### <a name="performance-management-strategies"></a>Stratégies de gestion des performances

* Pour éviter de devoir analyser manuellement les performances, il est plus efficace de **définir des alertes qui se déclenchent si les bases de données ou les pools s’éloignent des limites normales** .
* Pour répondre aux fluctuations à court terme de la taille de calcul agrégée d’un pool, le **niveau eDTU du pool peut être augmenté ou diminué** . Si cette fluctuation est régulière ou prévisible, la **mise à l’échelle du pool peut être planifiée automatiquement** . Par exemple, diminuez la taille du pool lorsque vous savez que votre charge de travail est faible, disons la nuit ou le week-end.
* Pour répondre aux fluctuations à plus long terme ou à la modification du nombre de bases de données, **des bases de données spécifiques peuvent être déplacées dans d’autres pools** .
* Pour répondre aux augmentations à court terme de la charge de bases de données *spécifiques* , **vous pouvez sortir celles-ci du pool et leur attribuer une taille de calcul spécifique** . Une fois que la charge est réduite, la base de données peut être remise dans le pool. Si cela est connu à l’avance, la base de données peut être déplacée préalablement pour vérifier qu’elle a toujours les ressources nécessaires et pour éviter l’impact sur les autres bases de données du pool. Si ce besoin est prévisible, par exemple un lieu accueillant un événement populaire avec une vente de tickets à grande échelle, ce comportement de gestion peut être intégré à l’application.

Le [portail Azure](https://portal.azure.com) offre des fonctionnalités intégrées de surveillance et d’alerte sur la plupart des ressources. La supervision et les alertes sont disponibles pour les bases de données et les pools. Ces fonctionnalités de surveillance et d’alertes intégrées sont propres à la ressource. Par conséquent, il est pratique de les utiliser pour un petit nombre de ressources, mais pas pour de nombreuses ressources.

Pour les scénarios à volume important où vous travaillez avec de nombreuses ressources, vous pouvez utiliser les [journaux Azure Monitor](./saas-dbpertenant-log-analytics.md). Il s’agit d’un service Azure distinct offrant l’analytique des journaux émis rassemblés dans un espace de travail Log Analytics. Les journaux Azure Monitor peuvent collecter des données de télémétrie à partir de nombreux services, et être utilisés pour interroger et définir des alertes.

## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>Obtenir les scripts de l'application Wingtip Tickets SaaS Database Per Tenant

Les scripts et le code de l’application de base de données multi-locataire SaaS Wingtip Tickets sont disponibles dans le dépôt GitHub [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant). Consultez les [conseils généraux](saas-tenancy-wingtip-app-guidance-tips.md) avant de télécharger et de débloquer les scripts Wingtip Tickets SaaS.

## <a name="provision-additional-tenants"></a>Approvisionner des locataires supplémentaires

Bien que les pools puissent être économiques avec seulement deux bases de données S3, plus le nombre de bases de données du pool est élevé, plus les économies sont notables. Pour une bonne compréhension du fonctionnement de la gestion et de la surveillance des performances à grande échelle, ce didacticiel requiert le déploiement d’au moins 20 bases de données.

Si vous avez déjà configuré un lot de clients dans le cadre d’un didacticiel précédent, passez à la section [Simuler l’utilisation sur toutes les bases de données client](#simulate-usage-on-all-tenant-databases).

1. Ouvrez... **Modules d’apprentissage** Analyse et gestion des performances\\\\Demo-PerformanceMonitoringAndManagement.ps1\\ dans *PowerShell ISE* . Gardez ce script ouvert, car vous exécuterez plusieurs scénarios au cours de ce didacticiel.
1. Définissez **$DemoScenario** = **1** , **Approvisionner un lot de locataires**
1. Appuyez sur **F5** pour exécuter le script.

Le script déploie 17 locataires en moins de cinq minutes.

Le script *New-TenantBatch* utilise un ensemble imbriqué ou lié de modèles [Resource Manager](../../azure-resource-manager/index.yml) qui créent un lot de clients, ce qui a pour effet de copier par défaut la base de données **basetenantdb** sur le serveur de catalogue pour créer de nouvelles bases de données client, puis d’inscrire celles-ci dans le catalogue, avant de les initialiser avec le type de lieu et le nom du client. Ce comportement est cohérent avec la manière dont l’application approvisionne un nouveau locataire. Les modifications apportées à *basetenantdb* sont appliquées aux nouveaux clients approvisionnés par la suite. Pour découvrir comment apporter des modifications de schéma à des bases de données client *existantes* , y compris à la base de données *basetenantdb* , voir le [didacticiel relatif à la gestion de schéma](saas-tenancy-schema-management.md).

## <a name="simulate-usage-on-all-tenant-databases"></a>Simuler l’utilisation sur toutes les bases de données de locataire

Le script *Demo-PerformanceMonitoringAndManagement.ps1* simulant une charge de travail s’exécutant sur toutes les bases de données clients est fourni. La charge est générée à l’aide de l’un des scénarios de charge disponibles :

| Démonstration | Scénario |
|:--|:--|
| 2 | Générer une charge d’intensité normale (environ 40 DTU) |
| 3 | Générer une charge avec des pics plus longs et plus fréquents par base de données|
| 4 | Générer une charge avec des pics de DTU supérieurs par base de données (environ 80 DTU)|
| 5 | Générer une charge normale et une charge élevée sur un seul locataire (environ 95 DTU)|
| 6 | Générer une charge déséquilibrée entre plusieurs pools|

Le générateur de charge applique une charge CPU *synthétique* à chaque base de données de locataire. Le générateur démarre un travail pour chaque base de données de locataire, qui appelle périodiquement une procédure stockée qui génère la charge. Les niveaux de charge (exprimés en eDTU), la durée et les intervalles varient selon les bases de données afin de simuler l’activité d’un locataire imprévisible.

1. Ouvrez... **Modules d’apprentissage** Analyse et gestion des performances\\\\Demo-PerformanceMonitoringAndManagement.ps1\\ dans *PowerShell ISE* . Gardez ce script ouvert, car vous exécuterez plusieurs scénarios au cours de ce didacticiel.
1. Définissez **$DemoScenario** = **2** , *Générer une charge d’intensité normale* .
1. Appuyez sur **F5** pour appliquer une charge à toutes vos bases de données de locataire.

L’application de base de données Wingtip Tickets SaaS par client est une application SaaS, et dans le monde réel, la charge se trouvant sur une application SaaS est généralement sporadique et imprévisible. Pour simuler cette situation, le générateur de charge produit une charge aléatoire répartie sur tous les locataires. Plusieurs minutes étant nécessaires pour que le modèle de charge émerge, laissez au générateur de charge environ 3 à 5 minutes avant d’essayer de surveiller la charge dans les sections suivantes.

> [!IMPORTANT]
> Le générateur de charge exécute une série de travaux dans votre session PowerShell locale. Laissez le fichier *Demo-PerformanceMonitoringAndManagement.ps1* ouvert. Si vous fermez l’onglet ou que vous interrompez votre ordinateur, le générateur de charge s’arrête. Le générateur de charge reste à l’état *job-invoking* où il génère la charge sur les nouveaux clients approvisionnés après le démarrage du générateur. Utilisez *Ctrl-C* pour arrêter l’appel de nouvelles tâches et de quitter le script. Le générateur de charge continue de s’exécuter, mais uniquement sur les clients existants.

## <a name="monitor-resource-usage-using-the-azure-portal"></a>Surveiller l’utilisation des ressources via le portail Azure

Pour surveiller l’utilisation de ressources qui résulte de la charge appliquée, dans le portail, ouvrez le pool contenant les bases de données client :

1. Ouvrez le [portail Azure](https://portal.azure.com), puis accédez au serveur *tenants1-dpt-&lt;UTILISATEUR&gt;* .
1. Faites défiler vers le bas, recherchez les pools élastiques, puis cliquez sur **Pool1** . Ce pool contient toutes les bases de données de locataire créées jusqu’à présent.

Observez les graphiques **Surveillance du pool élastique** et **Surveillance de la base de données élastique** .

L’utilisation des ressources du pool est l’utilisation agrégée de toutes les bases de données du pool. Le graphique de base de données affiche les cinq bases de données les plus sollicitées :

![Graphique de base de données](./media/saas-dbpertenant-performance-monitoring/pool1.png)

Comme il existe d’autres bases de données dans le pool en plus des cinq principales, l’utilisation du pool affiche une activité qui n’est pas reflétée dans le graphique des cinq bases de données principales. Pour plus de détails, cliquez sur **Utilisation des ressources de base de données**  :

![Utilisation des ressources de base de données](./media/saas-dbpertenant-performance-monitoring/database-utilization.png)


## <a name="set-performance-alerts-on-the-pool"></a>Définir les alertes de performance sur le pool

Définissez une alerte sur le pool, qui se déclenche quand \>l’utilisation atteint 75 % comme suit :

1. Ouvrez *Pool1* (sur le serveur *tenants1-dpt-\<user\>* ) dans le [portail Azure](https://portal.azure.com).
1. Cliquez sur **Règles d’alerte** , puis sur **+ Ajouter une alerte** .

   ![ajouter une alerte](./media/saas-dbpertenant-performance-monitoring/add-alert.png)

1. Entrez un nom, par exemple **Nombre élevé de DTU** ,
1. Définissez les valeurs suivantes :
   * **Métrique = pourcentage eDTU**
   * **Condition = supérieur à**
   * **Seuil = 75**
   * **Période = Au cours des 30 dernières minutes**
1. Ajoutez une adresse e-mail à la zone *Adresse(s) e-mail administrateur supplémentaire(s)* , puis cliquez sur **OK** .

   ![définir une alerte](./media/saas-dbpertenant-performance-monitoring/alert-rule.png)


## <a name="scale-up-a-busy-pool"></a>Augmenter la taille d’un pool occupé

Si le niveau de charge global d’un pool augmente jusqu’à atteindre la limite maximale du pool et l’utilisation de 100 % des eDTU, les performances de chaque base de données sont affectées, ce qui peut ralentir les temps de réponse pour l’ensemble des bases de données du pool.

**À court terme** , envisagez d’augmenter la taille du pool pour fournir des ressources supplémentaires ou de retirer des bases de données du pool (en les déplaçant vers d’autres pools ou hors du pool vers un niveau de service autonome).

**À long terme** , envisagez d’optimiser les requêtes ou l’utilisation de l’index pour améliorer les performances des bases de données. En fonction de la sensibilité de l’application aux problèmes de performances, il est recommandé d’augmenter la taille d’un pool avant que celui-ci atteigne l’utilisation d’eDTU maximale. Utilisez une alerte pour vous avertir à l’avance.

Vous pouvez simuler un pool occupé en augmentant la charge produite par le générateur. Cela a pour effet de demander aux bases de données de fonctionner au maximum plus fréquemment et plus longtemps et d’augmenter la charge globale sur le pool sans modifier les besoins des bases de données individuelles. Vous pouvez facilement augmenter la taille du pool dans le portail ou à partir de PowerShell. Cet exercice utilise le portail.

1. Définissez *$DemoScenario* = **3** , _Générer une charge avec des pics plus longs et plus fréquents par base de données_ pour augmenter l’intensité de la charge globale sur le pool sans modifier le pic de charge requis par chaque base de données.
1. Appuyez sur **F5** pour appliquer une charge à toutes vos bases de données de locataire.

1. Accédez à **Pool1** dans le portail Azure.

Surveillez l’augmentation de l’utilisation d’eDTU du pool sur le graphique du haut. L’affichage de la nouvelle charge supérieure peut prendre quelques minutes, mais vous devez rapidement voir que le pool commence à atteindre une utilisation maximale, et qu’à mesure que la charge se stabilise dans le nouveau modèle, le pool est rapidement surchargé.

1. Pour augmenter la taille du pool, cliquez sur **Configurer le pool** en haut de la page **Pool1** .
1. Réglez le paramètre **eDTU du pool** sur **100** . La modification du nombre d’eDTU du pool ne modifie pas les paramètres de chaque base de données (qui est toujours de 50 eDTU par base de données). Vous pouvez voir les paramètres de chaque base de données sur le côté droit de la page **Configurer le pool** .
1. Cliquez sur **Enregistrer** pour envoyer la demande de mise à l’échelle du pool.

Revenez à **Pool1** > **Vue d’ensemble** pour afficher les graphiques de surveillance. Surveillez l’effet produit par la fourniture de davantage de ressources au pool (bien qu’avec quelques bases de données et une charge aléatoire, il n’est pas toujours facile de voir l’effet de façon concluante avant un certain temps d’exécution). Lorsque vous examinez les graphiques, gardez à l’esprit le fait que 100 % sur le graphique du haut représente maintenant 100 eDTU, alors que sur le graphique du bas, 100 % correspond toujours à 50 eDTU, car la valeur maximale par base de données est toujours de 50 eDTU.

La base de données reste en ligne et entièrement disponible tout au long du processus. Au dernier moment, lorsque chaque base de données est prête à être activée avec le nouveau nombre d’eDTU du pool, toutes les connexions actives sont interrompues. Le code d’application doit toujours être écrit de façon à retenter les connexions ignorées et à ce que la reconnexion à la base de données ait lieu dans le pool mis à l’échelle.

## <a name="load-balance-between-pools"></a>Équilibrage de charge entre pools

Comme alternative à la mise à l’échelle du pool, créez un deuxième pool et déplacez-y des bases de données pour équilibrer la charge entre les deux pools. Pour ce faire, le nouveau pool doit être créé sur le même serveur que le premier.

1. Dans le [portail Azure](https://portal.azure.com), ouvrez le serveur **tenants1-dpt-&lt;UTILISATEUR&gt;** .
1. Cliquez sur **+ Nouveau pool** pour créer un pool sur le serveur actuel.
1. Sur le modèle de **pool élastique** :

   1. Définissez **Nom** sur *Pool2* .
   1. Laissez le niveau tarifaire **Pool Standard** .
   1. Cliquez sur **Configurer le pool** .
   1. Définissez **eDTU du Pool** sur *50 eDTU* .
   1. Cliquez sur **Ajouter des bases de données** pour afficher la liste des bases de données sur le serveur qui peuvent être ajoutées au *Pool2* .
   1. Choisissez 10 bases de données à déplacer vers le nouveau pool, puis cliquez sur **Sélectionner** . Si vous avez exécuté le générateur de charge, le service sait déjà que votre profil de performance requiert un pool d’une taille supérieure à la taille par défaut de 50 eDTU et vous conseille de commencer avec un paramètre 100 eDTU.

      ![Recommandation](./media/saas-dbpertenant-performance-monitoring/configure-pool.png)

   1. Pour ce didacticiel, conservez la valeur par défaut de 50 eDTU, puis cliquez de nouveau sur **Sélectionner** .
   1. Sélectionnez **OK** pour créer le nouveau pool et y déplacer les bases de données sélectionnées.

La création du pool et le déplacement des bases de données prend quelques minutes. Durant le déplacement, les bases de données restent en ligne et totalement accessibles jusqu’au dernier moment, après quoi toutes les connexions ouvertes sont fermées. Pour autant que vous disposiez d’une logique de nouvelle tentative, les clients se connectent ensuite à la base de données dans le nouveau pool.

Accédez à **Pool2** (sur le serveur *tenants1-dpt-\<user\>* ) pour ouvrir le pool et superviser ses performances. Si vous ne voyez pas le pool, attendez que l’approvisionnement du nouveau pool soit terminé.

Vous voyez à présent que l’utilisation des ressources sur le *Pool1* a chuté et que le *Pool2* est chargé dans la même mesure.

## <a name="manage-performance-of-an-individual-database"></a>Gérer les performances d’une base de données individuelle

Si une base de données individuelle d’un pool connaît une charge élevée soutenue, selon la configuration du pool, elle peut dominer les ressources du pool et avoir un impact sur d’autres bases de données. Si l’activité est susceptible de continuer pendant un certain temps, la base de données peut être déplacée temporairement hors du pool. Cela permet à la base de données de disposer des ressources supplémentaires dont elle a besoin, et l’isole les autres bases de données.

Cet exercice simule l’effet de la salle de concert Contoso qui subit une charge élevée quand les tickets sont mis en vente pour un concert populaire.

1. Ouvrez le script **Demo-PerformanceMonitoringAndManagement.ps1** dans \\*PowerShell ISE* .
1. Définissez **$DemoScenario = 5, Générer une charge normale et une charge élevée sur un locataire unique (environ 95 DTU).**
1. Définissez **$SingleTenantDatabaseName = contosoconcerthall**
1. Exécutez le script en appuyant sur **F5** .


1. Dans le [portail Azure](https://portal.azure.com), accédez à la liste des bases de données sur le serveur *tenants1-dpt-\<user\>* . 
1. Cliquez sur la base de données **contosoconcerthall** .
1. Cliquez sur le pool dans lequel se trouve **contosoconcerthall** . Recherchez le pool dans la section **Pool élastique** .

1. Inspectez le graphique **Surveillance du pool élastique** , et recherchez l’utilisation d’eDTU du pool accrue. Après une ou deux minutes, la charge plus élevée doit commencer à apparaître et vous devez rapidement voir que le pool a atteint 100 % d’utilisation.
2. Examinez également l’affichage **Surveillance de la base de données élastique** qui illustre les bases de données les plus actives au cours de la dernière heure. La base de données *contosoconcerthall* doit figurer rapidement parmi les cinq bases de données les plus sollicitées.
3. **Cliquez sur le **graphique** Supervision de la base de données élastique** pour ouvrir la page **Utilisation des ressources de base de données** dans laquelle vous pouvez superviser toutes les bases de données. Cela vous permet d’isoler l’affichage de la base de données *contosoconcerthall* .
4. Dans la liste des bases de données, cliquez sur **contosoconcerthall** .
5. Cliquez sur **Niveau tarifaire (mise à l’échelle de DTU)** pour ouvrir la page **Configurer les performances** dans laquelle vous pouvez définir une taille de calcul autonome pour la base de données.
6. Cliquez sur l’onglet **Standard** pour ouvrir les options de mise à l’échelle du niveau Standard.
7. Faites glisser le **curseur DTU** vers la droite pour sélectionner **100**  DTU. Notez que cela correspond à l’objectif de service **S3** .
8. Cliquez sur **Appliquer** pour déplacer la base de données hors du pool et en faire une base de données *S3 Standard* .
9. Une fois la mise à l’échelle terminée, surveillez l’effet exercé sur la base de données contosoconcerthall et sur Pool1 dans les panneaux du pool élastique et de la base de données.

Lorsque la charge élevée sur la base de données contosoconcerthall diminue, vous devez rapidement ramener celle-ci dans le pool pour réduire les coûts associés. S’il est difficile de savoir quand cela se produit, vous pouvez définir une alerte sur la base de données qui se déclenche lorsque son utilisation en DTU descend en dessous du maximum par base de données sur le pool. Le déplacement d’une base de données dans un pool est décrite dans l’exercice 5.

## <a name="other-performance-management-patterns"></a>Autres modèles de gestion des performances

**Mise à l’échelle préemptive**  : dans l’exercice ci-dessus où vous avez exploré la mise à l’échelle d’une base de données isolée, vous saviez quelle base de données rechercher. Si la direction du Contoso Concert Hall avait informé Wingtip d’une vente imminente de tickets, la base de données aurait pu être préalablement déplacée hors du pool. Dans le cas contraire, il aurait probablement été nécessaire de configurer une alerte sur le pool ou la base de données pour détecter ce qui s’est produit. Vous ne voulez pas prendre connaissance de ce type de problème par d’autres locataires du pool qui se plaignent d’une dégradation des performances. Et si le locataire peut prédire la durée pendant laquelle il va avoir besoin de ressources supplémentaires, vous pouvez configurer un runbook Azure Automation pour déplacer la base de données hors du pool puis la réimporter selon une planification définie.

**Mise à l’échelle libre-service par le locataire**  : comme la mise à l’échelle est une tâche facilement appelée via l’API de gestion, vous pouvez aisément configurer la possibilité de mettre à l’échelle des bases de données de locataire dans votre application côté locataire et l’offrir en tant que fonctionnalité de votre service SaaS. Par exemple, laissez les locataires administrer la mise à l’échelle en liant ces opérations à leur facturation.

**Mise à l’échelle d’un pool selon une planification pour une correspondance avec les modèles d’utilisation**

Lorsque l’utilisation globale du locataire suit des modèles d’utilisation prévisibles, vous pouvez utiliser Azure Automation pour mettre à l’échelle un pool selon une planification. Par exemple, diminuez la taille d’un pool après 18 h et avant 6 h en semaine lorsque vous savez que les besoins en ressources baissent.



## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Simuler l’utilisation sur les bases de données de locataire en exécutant un générateur de charge fourni
> * Surveiller les bases de données de locataire à mesure qu’elles répondent à l’augmentation de la charge
> * Augmenter la taille du pool élastique en réponse à la charge accrue sur les bases de données
> * Approvisionner un deuxième pool élastique pour équilibrer la charge de l’activité de la base de données

[Didacticiel Restaurer une base de données à locataire unique](saas-dbpertenant-restore-single-tenant.md)


## <a name="additional-resources"></a>Ressources supplémentaires

* Autres [didacticiels reposant sur le déploiement de l’application de base de données Wingtip Tickets SaaS par client](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [Pools élastiques SQL](elastic-pool-overview.md)
* [Azure Automation](../../automation/automation-intro.md)
* [Journaux Azure Monitor](./saas-dbpertenant-log-analytics.md) : tutoriel sur la configuration et l’utilisation des journaux Azure Monitor