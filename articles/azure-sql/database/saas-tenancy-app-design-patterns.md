---
title: Modèles SaaS multilocataires
description: Découvrez les exigences et les modèles d’architecture de données les plus courants pour les applications de base de données SaaS (Software as a Service) multi-locataires qui s’exécutent dans l’environnement cloud Azure.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/25/2019
ms.custom: seoapril2019, sqldbrb=1
ms.openlocfilehash: 8a13c641d50a68d9661b4aa6caf8effb82d53dd7
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92793226"
---
# <a name="multi-tenant-saas-database-tenancy-patterns"></a>Modèles de location de base de données SaaS multi-locataire
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Cet article décrit les différents modèles de location disponibles pour une application SaaS multilocataire.

Quand vous concevez une application SaaS multi-locataire, vous devez choisir avec soin le modèle de location qui répond le mieux aux besoins de votre application.  Un modèle de location détermine la façon dont les données de chaque locataire sont mappées au stockage.  Le modèle de location que vous choisissez a un impact sur la conception et la gestion des applications.  Le fait de passer à un autre modèle par la suite peut parfois s’avérer coûteux.

## <a name="a-saas-concepts-and-terminology"></a>R. Concepts et terminologie SaaS

Dans le modèle SaaS (Software as a Service), votre entreprise ne vend pas de *licences* de vos logiciels. À la place, chaque client paie un loyer à votre entreprise, qui fait de chaque client un *locataire* de votre entreprise.

En échange du loyer, chaque client reçoit l’accès aux composants de votre application SaaS et voit ses données stockées dans le système SaaS.

Le terme *modèle de location* fait référence à la façon dont sont organisées les données stockées des locataires :

- *Monolocation :* &nbsp; Chaque base de données stocke les données d’un seul locataire.
- *Multilocation :* &nbsp; Chaque base de données stocke les données de plusieurs locataires distincts (avec des mécanismes de protection de la confidentialité des données).
- Des modèles de location hybrides sont également disponibles.

## <a name="b-how-to-choose-the-appropriate-tenancy-model"></a>B. Comment choisir le modèle de location approprié

En général, le modèle de location n’affecte pas le fonctionnement d’une application, mais il peut avoir un impact sur d’autres aspects de la solution dans son ensemble.  Les critères suivants sont utilisés pour évaluer chacun des modèles :

- **Scalabilité :**
    - Nombre de locataires.
    - Stockage par locataire.
    - Stockage dans l’agrégat.
    - Charge de travail.

- **Isolation des locataires :** &nbsp; Niveau de performance et isolation des données (impact de la charge de travail d’un locataire sur les autres).

- **Coût par locataire :** &nbsp; Coûts des bases de données.

- **Complexité du développement :**
    - Changements apportés au schéma.
    - Changements apportés aux requêtes (exigés par le modèle).

- **Complexité opérationnelle :**
    - Surveillance et gestion des performances.
    - Gestion des schémas.
    - Restauration d’un locataire.
    - Récupération d’urgence.

- **Possibilités de personnalisation :** &nbsp; Facilité de prise en charge des personnalisations de schéma propres au locataire ou à la classe du locataire.

La discussion sur la location est axée sur la couche *Données* .  Mais réfléchissons un instant à la couche *Application* .  La couche Application est traitée comme une entité monolithique.  Si vous divisez l’application en plusieurs composants, le modèle de location que vous avez choisi peut être amené à changer.  Vous pouvez traiter certains composants différemment des autres sur le plan de la location et de la plateforme/technologie de stockage utilisées.

## <a name="c-standalone-single-tenant-app-with-single-tenant-database"></a>C. Application à locataire unique autonome avec une base de données à locataire unique

#### <a name="application-level-isolation"></a>Isolation au niveau de l’application

Dans ce modèle, l’application est installée en intégralité à plusieurs reprises, une fois par locataire.  Chaque instance de l’application étant autonome, une instance n’interagit jamais avec une autre instance autonome.  Chaque instance de l’application n’a qu’un seul locataire et ne nécessite donc qu’une base de données.  Le locataire dispose de la base de données pour lui tout seul.

![Conception d’une application autonome avec une seule base de données à locataire unique.][image-standalone-app-st-db-111a]

Chaque instance de l’application est installée dans un groupe de ressources Azure distinct.  Le groupe de ressources peut appartenir à un abonnement dont le propriétaire est l’éditeur du logiciel ou le locataire.  Dans les deux cas, l’éditeur peut gérer le logiciel pour le locataire.  Chaque instance de l’application est configurée pour se connecter à la base de données correspondante.

Chaque base de données de locataire est déployée en tant que base de données unique.  Ce modèle offre le niveau d’isolation de base de données le plus élevé.  Mais l’isolation nécessite que des ressources suffisantes soient allouées à chaque base de données pour gérer les pics de charge.  L’important ici, c’est que les pools élastiques ne peuvent pas être utilisés pour les bases de données déployées sur des groupes de ressources ou des abonnements différents.  En raison de cette limitation, le modèle d’application autonome à locataire unique constitue la solution la plus onéreuse du point de vue du coût global des bases de données.

#### <a name="vendor-management"></a>Gestion des fournisseurs

Le fournisseur peut accéder à toutes les bases de données dans toutes les instances d’application autonomes, même si les instances d’application sont installées dans des abonnements appartenant à des locataires différents.  L’accès s’effectue par le biais de connexions SQL.  Cet accès entre instances peut permettre au fournisseur de centraliser la gestion des schémas et les requêtes entre bases de données dans le but de créer des rapports ou à des fins analytiques.  Si ce type de gestion centralisée est souhaité, vous devez déployer un catalogue qui mappe les identificateurs de locataire aux URI de base de données.  Azure SQL Database propose une bibliothèque de partitionnement qui fonctionne pour fournir un catalogue.  La bibliothèque de partitionnement a pour nom officiel [Bibliothèque cliente de bases de données élastiques][docu-elastic-db-client-library-536r].

## <a name="d-multi-tenant-app-with-database-per-tenant"></a>D. Application multi-locataire avec une base de données par locataire

Le modèle suivant utilise une application multi-locataire avec plusieurs bases de données qui sont toutes à locataire unique.  Une nouvelle base de données est provisionnée pour chaque nouveau locataire.  Vous pouvez mettre à l’échelle la couche Application *verticalement* en ajoutant davantage de ressources par nœud.  Vous pouvez aussi mettre à l’échelle l’application *horizontalement* en ajoutant des nœuds supplémentaires.  La mise à l’échelle est basée sur la charge de travail et est indépendante du nombre ou de l’échelle des bases de données individuelles.

![Conception d’une application multi-locataire avec une base de données par locataire.][image-mt-app-db-per-tenant-132d]

#### <a name="customize-for-a-tenant"></a>Personnaliser en fonction d’un locataire

Comme le modèle d’application autonome, l’utilisation de bases de données à locataire unique offre un niveau élevé d’isolation des locataires.  Dans toute application dont le modèle spécifie uniquement des bases de données à locataire unique, vous pouvez personnaliser et optimiser le schéma d’une base de données quelconque en fonction de son locataire.  Cette personnalisation n’affecte pas les autres locataires dans l’application. Par exemple, un locataire peut nécessiter des données qui ne figurent pas dans les champs de données de base communs à tous les locataires.  Par ailleurs, le champ de données supplémentaire peut nécessiter un index.

Grâce au modèle de base de données par locataire, vous pouvez facilement personnaliser le schéma pour un ou plusieurs locataires individuels.  Le fournisseur de l’application doit concevoir des procédures pour gérer avec soin les personnalisations du schéma à l’échelle.

#### <a name="elastic-pools"></a>Pools élastiques

Quand des bases de données sont déployées dans le même groupe de ressources, elles peuvent être regroupées dans des pools élastiques.  Les pools offrent un moyen économique de partager des ressources entre plusieurs bases de données.  Le fait de recourir à des pools revient moins cher que d’exiger une base de données suffisamment volumineuse pour faire face aux pics d’utilisation.  Même si les bases de données mises en pool partagent l’accès aux ressources, vous pouvez toujours obtenir un niveau élevé d’isolation des performances.

![Conception d’une application multi-locataire avec une base de données par locataire, à l’aide d’un pool élastique.][image-mt-app-db-per-tenant-pool-153p]

Azure SQL Database fournit les outils nécessaires pour configurer, surveiller et gérer le partage.  Les métriques de performances au niveau du pool et de la base de données sont disponibles dans le portail Azure et par le biais des journaux Azure Monitor.  Les métriques peuvent donner d’excellents insights sur les performances agrégées et celles spécifiques au locataire.  Vous pouvez déplacer les bases de données individuelles entre pools pour fournir des ressources réservées à un locataire spécifique.  Ces outils vous permettent de garantir un bon niveau de performance de façon rentable.

#### <a name="operations-scale-for-database-per-tenant"></a>Mise à l’échelle des opérations pour le modèle de base de données par locataire

Azure SQL Database propose de nombreuses fonctionnalités de gestion conçues pour gérer plus de 100 000 bases de données à grande échelle.  Ces fonctionnalités font du modèle de base de données par locataire un modèle plausible.

Prenons l’exemple d’un système constitué d’une seule base de données à 1 000 locataires.  La base de données peut avoir 20 index.  Si le système passe à 1000 bases de données à locataire unique, le nombre d’index s’élève à 20 000.  Dans le cadre du [paramétrage automatique][docu-sql-db-automatic-tuning-771a] Azure SQL Database, les fonctionnalités d’indexation automatiques sont activées par défaut.  L’indexation automatique gère pour vous les 20 000 index et l’optimisation des opérations continues de création et de suppression.  Effectuées dans une base de données individuelle, ces actions automatisées ne sont ni coordonnées ni restreintes par des actions similaires dans d’autres bases de données.  L’indexation automatique traite différemment les index selon qu’ils se trouvent dans une base de données occupée ou non.  Cette tâche colossale de personnalisation de la gestion des index serait difficile à réaliser manuellement à l’échelle du modèle de base de données par locataire.

Parmi les autres fonctionnalités de gestion pouvant facilement être mises à l’échelle, citons les suivantes :

- Sauvegardes intégrées.
- Haute disponibilité :
- Chiffrement sur le disque.
- Télémétrie des performances.

#### <a name="automation"></a>Automatisation

Les opérations de gestion peuvent être scriptées et proposées par le biais d’un modèle [devops][http-visual-studio-devops-485m].  Vous pouvez même automatiser et exposer les opérations dans l’application.

Par exemple, vous pouvez automatiser la récupération d’un locataire unique à un point antérieur dans le temps.  La récupération doit seulement restaurer la base de données à locataire unique qui stocke le locataire.  Cette restauration n’a aucun impact sur les autres locataires, ce qui confirme que les opérations de gestion sont effectuées très précisément au niveau de chaque locataire.

## <a name="e-multi-tenant-app-with-multi-tenant-databases"></a>E. Application multi-locataire avec des bases de données multi-locataires

Un autre modèle disponible consiste à stocker plusieurs locataires dans une base de données multi-locataire.  L’instance d’application peut avoir autant de bases de données multi-locataires que vous le souhaitez.  Le schéma d’une base de données doit avoir une ou plusieurs colonnes d’identification des locataires pour permettre la récupération sélective des données de n’importe quel locataire.  En outre, le schéma peut nécessiter quelques tables ou colonnes qui sont uniquement utilisées par un sous-ensemble de locataires.  Toutefois, le code statique et les données de référence ne sont stockés qu’une seule fois et sont partagés par tous les locataires.

#### <a name="tenant-isolation-is-sacrificed"></a>L’isolation des locataires est pénalisée

*Données :* &nbsp; Une base de données multi-locataire nuit nécessairement à l’isolation des locataires.  Les données de plusieurs locataires sont stockées ensemble dans une base de données.  Pendant le développement, vérifiez que les requêtes n’exposent jamais les données de plusieurs locataires.  SQL Database prend en charge la [sécurité au niveau des lignes][docu-sql-svr-db-row-level-security-947w], ce qui permet de limiter l’étendue des données retournées par une requête à un seul locataire.

*Traitement :* &nbsp; Une base de données multi-locataire partage les ressources de calcul et de stockage entre tous ses locataires.  Vous pouvez surveiller la base de données dans son ensemble pour vérifier que ses performances sont acceptables.  Toutefois, le système Azure n’intègre aucun outil permettant de surveiller ou de gérer l’utilisation de ces ressources par un locataire individuel.  La base de données multi-locataire accroît donc le risque de rencontre de voisins bruyants, où la charge de travail d’un locataire hyperactif a un impact sur les performances des autres locataires dans la même base de données.  La mise en place d’une surveillance supplémentaire au niveau de l’application peut permettre de surveiller les performances au niveau du locataire.

#### <a name="lower-cost"></a>Coûts réduits

En général, les bases de données multi-locataires offrent le coût le plus bas par locataire.  Les coûts des ressources pour une base de données unique sont inférieurs à ceux d’un pool élastique de taille équivalente.  De plus, pour les scénarios où les locataires ont seulement besoin d’un stockage limité, vous pouvez potentiellement stocker des millions de locataires dans une même base de données.  Un pool élastique ne peut pas contenir des millions de bases de données.  Toutefois, si vous avez une solution contenant 1 000 bases de données par pool et 1 000 pools, vous risquez de vous retrouver avec une solution lourde à gérer composée de millions d’éléments.

Deux variantes du modèle de base de données multi-locataire sont abordées ci-après, le modèle multi-locataire partitionné étant le plus souple et le plus scalable.

## <a name="f-multi-tenant-app-with-a-single-multi-tenant-database"></a>F. Application multi-locataire avec une base de données multi-locataire unique

Le modèle de base de données multilocataire le plus simple emploie une base de données unique pour héberger des données pour tous les locataires.  Au fur et à mesure que des locataires sont ajoutés, la base de données est mise à l’échelle verticalement avec davantage de ressources de stockage et de calcul.  Cette mise à l’échelle verticale peut suffire, bien qu’elle soit toujours soumise à une limite.  Cependant, la base de données devient lourde à gérer bien avant que cette limite ne soit atteinte.

Les opérations de gestion qui portent sur des locataires individuels sont plus complexes à implémenter dans une base de données multi-locataire.  Et à grande échelle, ces opérations peuvent devenir beaucoup trop lentes.  C’est le cas par exemple d’une restauration dans le temps des données pour un seul locataire.

## <a name="g-multi-tenant-app-with-sharded-multi-tenant-databases"></a>G. Application multi-locataire avec des bases de données multi-locataires partitionnées

La plupart des applications SaaS accèdent aux données d’un seul locataire à la fois.  Ce modèle d’accès permet aux données du locataire d’être distribuées entre plusieurs bases de données ou partitions, où toutes les données d’un locataire sont contenues dans une seule partition.  Associé à un modèle de base de données multi-locataire, un modèle partitionné autorise une mise à l’échelle pratiquement illimitée.

![Conception d’une application multi-locataire avec des bases de données multi-locataires partitionnées.][image-mt-app-sharded-mt-db-174s]

#### <a name="manage-shards"></a>Gérer les partitions

Le partitionnement rend la conception et la gestion des opérations plus complexes.  Vous devez disposer d’un catalogue dans lequel le mappage entre les locataires et les bases de données est tenu à jour.  Des procédures de gestion sont également nécessaires pour gérer les partitions et la population des locataires.  Par exemple, vous devez concevoir des procédures pour ajouter et supprimer des partitions et pour déplacer les données des locataires entre partitions.  L’un des moyens d’effectuer la mise à l’échelle consiste à ajouter une nouvelle partition et à la peupler de nouveaux locataires.  En d’autres occasions, vous pouvez fractionner une partition très peuplée en deux partitions moins peuplées.  Si plusieurs locataires sont déplacés ou mis hors service, vous pouvez fusionner des partitions peu peuplées ensemble.  La fusion aboutit à une utilisation plus rentable des ressources.  Vous pouvez également déplacer les locataires entre partitions pour équilibrer les charges de travail.

SQL Database propose un outil de fractionnement/fusion qui fonctionne conjointement avec la bibliothèque de partitionnement et la base de données de catalogues.  L’application fournie peut non seulement fractionner et fusionner les partitions, mais elle peut aussi déplacer les données des locataires entre partitions.  L’application gère également le catalogue pendant ces opérations ; pour cela, elle marque les locataires affectés comme étant hors connexion avant de les déplacer.  Au terme du déplacement, l’application remet à jour le catalogue avec le nouveau mappage et marque le locataire comme étant à nouveau en ligne.

#### <a name="smaller-databases-more-easily-managed"></a>Gestion plus facile des bases de données plus petites

La solution multi-locataire partitionnée répartit les locataires sur plusieurs bases de données, ce qui conduit à des bases de données plus petites et plus faciles à gérer.  Par exemple, pour restaurer un locataire spécifique à un point antérieur dans le temps, il suffit désormais de restaurer une seule base de données de petite taille à partir d’une sauvegarde (et non une base de données volumineuse contenant tous les locataires). Vous pouvez choisir la taille de la base de données et le nombre de locataires par base de données pour équilibrer la charge de travail et les efforts de gestion.

#### <a name="tenant-identifier-in-the-schema"></a>Identificateur de locataire dans le schéma

Selon l’approche de partitionnement utilisée, des contraintes supplémentaires peuvent être imposées au schéma de base de données.  L’application de fractionnement/fusion SQL Database exige que le schéma inclue la clé de partitionnement, qui est généralement l’identificateur de locataire.  L’identificateur de locataire est l’élément situé au début de la clé primaire de toutes les tables partitionnées.  L’identificateur de locataire permet à l’application de fractionnement/fusion de localiser et de déplacer rapidement les données associées à un locataire spécifique.

#### <a name="elastic-pool-for-shards"></a>Pool élastique pour les partitions

Vous pouvez placer les bases de données multi-locataires partitionnées dans des pools élastiques.  En général, le fait d’avoir plusieurs bases de données à locataire unique dans un pool est aussi rentable que d’avoir plusieurs locataires dans quelques bases de données multi-locataires.  Les bases de données multi-locataires sont avantageuses quand vous avez un grand nombre de locataires relativement inactifs.

## <a name="h-hybrid-sharded-multi-tenant-database-model"></a>H. Modèle de base de données multi-locataire partitionnée hybride

Dans le modèle hybride, l’identificateur de locataire est inclus dans le schéma de toutes les bases de données.  Les bases de données sont toutes capables de stocker plusieurs locataires, et elles peuvent être partitionnées.  Du point de vue du schéma, ces bases de données sont donc toutes multi-locataires.  Pourtant, dans la pratique, certaines de ces bases de données contiennent un seul locataire.  Quoi qu’il en soit, la quantité de locataires stockés dans une base de données n’a aucun effet sur le schéma de la base de données.

#### <a name="move-tenants-around"></a>Déplacer les locataires

Vous pouvez à tout moment déplacer un locataire spécifique dans sa propre base de données multi-locataire.  Vous pouvez aussi à tout moment replacer le locataire dans une base de données contenant plusieurs locataires.  Vous pouvez également affecter un locataire à une nouvelle base de données à locataire unique quand vous provisionnez la nouvelle base de données.

Le modèle hybride est particulièrement adapté quand des groupes identifiables de locataires ont des besoins en ressources très différents.  Par exemple, supposons que les locataires qui participent à un essai gratuit ne bénéficient pas systématiquement du même niveau de performance que les locataires abonnés.  La stratégie peut stipuler de stocker les locataires dans la phase d’essai gratuit dans une base de données multi-locataire qui est partagée entre tous les locataires de l’essai gratuit.  Quand un locataire dans la phase d’essai gratuit s’abonne au niveau de service de base, le locataire peut être déplacé dans une autre base de données multilocataire qui peut contenir moins de locataires.  Un abonné avec un niveau de service Premium peut être déplacé dans sa nouvelle base de données monolocataire.

#### <a name="pools"></a>Pools

Dans ce modèle hybride, les bases de données à locataire unique pour les locataires abonnés peuvent être placées dans des pools de ressources pour réduire les coûts de la base de données par locataire.  Il en va de même dans le modèle de base de données par locataire.

## <a name="i-tenancy-models-compared"></a>I. Comparaison des modèles de location

Le tableau suivant récapitule les différences entre les principaux modèles de location.

| Mesure | Application autonome | Base de données par locataire | Multi-locataire partitionné |
| :---------- | :------------- | :------------------ | :------------------- |
| Scale | Moyenne<br />1-Plusieurs centaines | Très élevée<br />1-Plusieurs centaines de milliers | Illimité<br />1-Plusieurs millions |
| Isolation des locataires | Très élevée | Élevé | Faible ; à l’exception d’un locataire unique (seul dans une base de données multilocataire). |
| Coût de base de données par locataire | Élevé ; dimensionné pour les pics. | Bas ; pools utilisés. | Le plus bas ; pour les petits locataires dans les bases de données multi-locataires. |
| Surveillance et gestion des performances | Par locataire uniquement | Agrégat + par locataire | Agrégat ; mais par locataire uniquement pour les bases de données uniques. |
| Complexité du développement | Faible | Faible | Moyen ; en raison du partitionnement. |
| Complexité opérationnelle | Faible-Élevée. Simple au niveau individuel, complexe à grande échelle. | Faible-Moyenne. Les modèles permettent de réduire la complexité à grande échelle. | Faible-Élevée. La gestion des locataires individuels est complexe. |
| &nbsp; ||||

## <a name="next-steps"></a>Étapes suivantes

- [Déployer et explorer une application Wingtip multi-locataire qui utilise le modèle SaaS de base de données par locataire - Azure SQL Database][docu-sql-db-saas-tutorial-deploy-wingtip-db-per-tenant-496y]

- [Bienvenue dans l’exemple d’application de location Azure SQL Database Wingtip Tickets SaaS][docu-saas-tenancy-welcome-wingtip-tickets-app-384w]


<!--  Article link references.  -->

[http-visual-studio-devops-485m]: https://www.visualstudio.com/devops/

[docu-sql-svr-db-row-level-security-947w]: /sql/relational-databases/security/row-level-security

[docu-elastic-db-client-library-536r]:elastic-database-client-library.md
[docu-sql-db-saas-tutorial-deploy-wingtip-db-per-tenant-496y]: saas-dbpertenant-get-started-deploy.md
[docu-sql-db-automatic-tuning-771a]:automatic-tuning-overview.md
[docu-saas-tenancy-welcome-wingtip-tickets-app-384w]: saas-tenancy-welcome-wingtip-tickets-app.md


<!--  Image references.  -->

[image-standalone-app-st-db-111a]: media/saas-tenancy-app-design-patterns/saas-standalone-app-single-tenant-database-11.png "Conception d’une application autonome avec une seule base de données à locataire unique."

[image-mt-app-db-per-tenant-132d]: media/saas-tenancy-app-design-patterns/saas-multi-tenant-app-database-per-tenant-13.png "Conception d’une application multi-locataire avec une base de données par locataire."

[image-mt-app-db-per-tenant-pool-153p]: media/saas-tenancy-app-design-patterns/saas-multi-tenant-app-database-per-tenant-pool-15.png "Conception d’une application multi-locataire avec une base de données par locataire, à l’aide d’un pool élastique."

[image-mt-app-sharded-mt-db-174s]: media/saas-tenancy-app-design-patterns/saas-multi-tenant-app-sharded-multi-tenant-databases-17.png "Conception d’une application multi-locataire avec des bases de données multi-locataires partitionnées."