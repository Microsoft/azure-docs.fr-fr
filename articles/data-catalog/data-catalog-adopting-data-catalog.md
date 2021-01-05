---
title: Approche et processus d’adoption d’Azure Data Catalog
description: Cet article présente une approche et un processus destiné aux organisations qui envisagent d’adopter Azure Data Catalog, y compris la définition de la vision du projet, l’identification des principaux cas d’utilisation métier au sein de l’organisation et le choix d’un projet pilote.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: conceptual
ms.date: 08/01/2019
ms.openlocfilehash: 63d1213a2dded41df3f462722b354b03fe9f6e33
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "72801880"
---
# <a name="approach-and-process-for-adopting-azure-data-catalog"></a>Approche et processus d’adoption d’Azure Data Catalog

Cet article va vous aider dans la prise en main de l’adoption **d’Azure Data Catalog** dans votre organisation. Pour mener à bien l’adoption d’**Azure Data Catalog**, vous pouvez vous concentrer sur trois éléments clés : définition de votre vision du projet, identification des principaux cas d’utilisation métier au sein de votre organisation et choix d’un projet pilote.

## <a name="introducing-the-azure-data-catalog"></a>Présentation d’Azure Data Catalog

Dans le monde du travail, les attentes sur la façon de trouver des informations provenant d’experts sur les ressources de données ont changé. Aujourd’hui, avec l’utilisation répandue des réseaux sociaux d’entreprise comme Yammer, les utilisateurs escomptent pouvoir obtenir rapidement assistance et conseils sur une large gamme de sujets. **Azure Data Catalog** permet aux entreprises et aux équipes de consolider des informations sur les ressources de données d’entreprise dans un référentiel central. Les utilisateurs de données peuvent découvrir ces ressources de données et bénéficier des connaissances d’experts du sujet.

Cet article présente une méthode de prise en main d’utilisation **d’Azure Data Catalog**. Il décrit un plan d’adoption standard de Data Catalog pour la société fictive Adventure Works.

**Qu’est-ce qu’Azure Data Catalog ?**

**Azure Data Catalog** est un service entièrement géré dans Azure. Il s’agit d’un catalogue d’informations (métadonnées) d’entreprise, qui permet la découverte des sources de données en libre-service. Avec Data Catalog, vous pouvez inscrire, découvrir et annoter des ressources de données, et également vous y connecter. Data Catalog est conçu pour gérer diverses ressources d’informations afin de leur faciliter la recherche de ressources de données, de les comprendre et de s’y connecter. Il accélère l’obtention d’informations à partir des données disponibles et accroît la valeur pour les organisations. Pour en savoir plus, consultez le site web [Microsoft Azure Data Catalog](https://azure.microsoft.com/services/data-catalog/).

## <a name="azure-data-catalog-adoption-plan"></a>Plan d’adoption d’Azure Data Catalog

Un plan d’adoption d’**Azure Data Catalog** explique comment les avantages liés à l’utilisation du service sont communiqués aux parties prenantes et aux utilisateurs, ainsi que le type de formation que vous allez fournir aux utilisateurs. L’un des principaux facteurs de réussite pour adopter Data Catalog tient à la façon dont vous communiquez la valeur du service aux utilisateurs et parties prenantes. Les audiences principales d’un plan d’adoption initial rassemblent les utilisateurs du service. Quel que soit l’engagement des parties prenantes, si les utilisateurs, ou clients, de votre offre Data Catalog ne l’intègrent pas dans leurs processus, l’adoption sera un échec. Cet article part donc du principe que vous disposez d’un engagement des parties prenantes. Il est consacré à la création d’un plan d’adoption de Data Catalog.
Un plan d’adoption efficace implique les personnes dans les possibilités offertes avec Data Catalog, et leur donne les informations et les conseils pour atteindre ces objectifs. Les utilisateurs doivent comprendre la valeur que Data Catalog leur apporte pour les aider à mener à bien leurs tâches. Lorsque les utilisateurs voient de quelle manière Data Catalog peut les aider à tirer un meilleur parti de leurs données, la valeur apportée par l’adoption de Data Catalog devient claire. Comme apporter des modifications est difficile, un plan efficace doit prendre en compte les difficultés des modifications.

Un plan d’adoption vous permet de communiquer les éléments essentiels pour que les utilisateurs réussissent et atteignent leurs objectifs. Un plan standard explique en quoi Data Catalog va faciliter la vie des utilisateurs et comprend les parties suivantes :

* **Énoncé de vision** : vous aide à présenter en quelques lignes le plan d’adoption aux utilisateurs et aux parties prenantes. Il s’agit là de votre argumentaire éclair.
* **Équipe pilote et influenceurs** : tirer des enseignements d’une équipe pilote et d’influenceurs vous aide à affiner la façon dont vous allez présenter Data Catalog aux équipes et aux utilisateurs. Les influenceurs peuvent accompagner les autres utilisateurs. Cela vous permet également de déterminer les freins à l’adoption ainsi que ses facteurs de réussite.
* **Plan de communications et tendance** : aide les utilisateurs à comprendre dans quelle mesure Data Catalog peut les aider et peut faciliter son adoption naturelle au sein des équipes, puis dans toute l’organisation.
* **Plan de formation** : une formation complète amène généralement à la réussite de l’adoption et à des résultats performants.

Voici quelques conseils pour définir un plan d’adoption d’ **Azure Data Catalog** .

## <a name="define-your-data-catalog-project-vision"></a>Définir votre vision du projet Data Catalog

La première étape de définition d’un plan d’adoption d’ **Azure Data Catalog** consiste à écrire une description formelle de ce que vous souhaitez accomplir. Il est préférable que l’énoncé de vision soit suffisamment large, et cependant concis pour définir des objectifs particuliers à court et à long terme.

Voici quelques conseils pour vous aider à définir votre vision du projet :

* **Identifiez le pilote de déploiement principal** : réfléchissez aux besoins de gestion de source de données spécifiques de l’entreprise qui peuvent être remplis par Data Catalog. Cela vous permet également d’indiquer les avantages principaux liés à l’utilisation de Data Catalog. Il se peut, par exemple que tous les nouveaux utilisateurs doivent prendre connaissance de sources de données communes et apprendre à les utiliser, ou que seule une poignée de responsables doive comprendre complètement les sources de données importantes et complexes. **Azure Data Catalog** peut aider à faciliter la découverte et la compréhension de ces sources de données afin que ces points faibles bien connus puissent être traités directement et au début de l’adoption du service.
* **Soyez clair et précis** : une présentation claire de la vision du projet permet de mettre tout le monde d’accord sur la valeur apportée par Data Catalog à l’organisation et sur la façon dont la vision du projet facilite les objectifs de l’organisation.
* **Motivez l’utilisation de Data Catalog** : votre vision du projet et votre plan de communication doivent inciter les utilisateurs à reconnaître que Data Catalog peut leur permettre de trouver des sources de données et de s’y connecter pour exploiter davantage les données.
* **Spécifiez des objectifs spécifiques et une chronologie** : cela permet de garantir que votre plan d’adoption dispose de livrables spécifiques et réalisables. Une chronologie permet de maintenir l’attention de tous et autorise des points de contrôle pour mesurer la réussite.

Voici un exemple d’énoncé de vision pour un plan d’adoption de Data Catalog pour la société fictive Adventure Works :

**Azure Data Catalog** autorise l’équipe Finances d’Adventure Works à collaborer sur les principales sources de données afin que chaque membre de l’équipe puisse facilement trouver et utiliser les données dont elle a besoin et partager ses connaissances avec le reste de l’équipe.

Une fois que vous avez indiqué un énoncé de vision clair, vous devez identifier un projet pilote approprié pour Data Catalog. En règle générale, il existe plusieurs scénarios pour Data Catalog. La section suivante présente donc quelques conseils afin d’identifier les cas d’utilisation appropriés.

## <a name="identify-data-catalog-business-use-cases"></a>Identifier les cas d’utilisation métier de Data Catalog

Pour identifier les cas d’utilisation pertinents pour Data Catalog, nouez le dialogue avec des experts des différentes divisions pour identifier les cas d’utilisation appropriés et les problèmes commerciaux à résoudre. Passez en revue les difficultés actuelles identifiées et la description des ressources de données. Par exemple, les équipes apprennent-elles sur les ressources de données uniquement après avoir interrogé plusieurs personnes de l’organisation qui disposent des sources de données pertinentes ?

Il est préférable de choisir des cas d’utilisation correspondant aux tâches les plus simples : cas qui même s’ils sont importants présentent une probabilité élevée d’être résolus avec Data Catalog.

Voici quelques conseils pour identifier les cas d’utilisation :

* **Définissez les objectifs de l’équipe** : comment l’équipe atteint-elle ses objectifs ? Pour le moment, ne vous concentrez pas sur Data Catalog puisque vous souhaitez être objectif à ce stade. N’oubliez pas qu’il est question des résultats de l’entreprise, pas de la technologie.
* **Définissez le problème de l’entreprise** : quels sont les problèmes auxquels l’équipe est confrontée concernant la recherche et l’apprentissage des ressources de données ? Par exemple, les informations sur les sources de données importantes peuvent se trouver dans les classeurs Excel d’un dossier réseau, et l’équipe peut passer beaucoup de temps à rechercher les classeurs.
* **Comprenez la culture d’équipe associée au changement** : de nombreuses difficultés de l’adoption sont liées à la résistance aux modifications, et non à l’implémentation d’un nouvel outil. La façon dont une équipe réagit à une modification est importante lors de l’identification des cas d’utilisation, car le processus existant peut être en place « parce que c’est toujours ainsi que nous avons procédé » ou « parce que pourquoi le réparer s’il n’est pas cassé ? ». L’adoption d’un nouvel outil ou d’un nouveau processus est toujours plus aisée lorsque les personnes concernées comprennent la valeur à tirer parti de la modification, et apprécient l’importance des problèmes à résoudre.
* **Restez concentré sur les données** : lorsque vous abordez les problèmes d’entreprise auxquels une équipe est confrontée, vous devez éliminer les éléments superflus et vous concentrer sur ce qui est pertinent pour exploiter les ressources de données d’entreprise avec une meilleure efficacité.

Voici quelques exemples de cas d’utilisation associés à Data Catalog :

### <a name="example-use-cases"></a>Exemples de cas d’utilisation

* **Inscrivez les sources de données centrales à valeur élevée** : le service informatique gère les sources de données utilisées dans l’organisation. Il peut prendre en main Data Catalog en inscrivant et en annotant des sources de données d’entreprise communes.
* **Inscrivez des sources de données en équipe** : différentes équipes possèdent des sources de données métier utiles. Démarrez **Azure Data Catalog** en identifiant et en inscrivant les principales sources de données utilisées par de nombreuses équipes différentes. Capturez les connaissances communautaires de l’équipe dans des annotations **Azure Data Catalog**.
* **Décisionnel libre-service** : les équipes passent beaucoup de temps à combiner les données provenant de plusieurs sources. Inscrivez et annotez des sources de données dans un emplacement central pour éliminer un processus de découverte manuelle des sources de données.

Pour en savoir plus sur les scénarios de Data Catalog, consultez l’article [Scénarios courants d’utilisation d’Azure Data Catalog](data-catalog-common-scenarios.md).

Une fois que vous avez identifié certains cas d’utilisation de Data Catalog, des scénarios courants doivent émerger. La section suivante explique comment identifier votre premier projet pilote en fonction d’un cas d’utilisation.

## <a name="choose-a-data-catalog-pilot-project"></a>Choisir un projet pilote Data Catalog

L’un des principaux facteurs de réussite consiste à simplifier et à commencer petit. Un pilote bien défini avec une étendue limitée permet de continuer à faire avancer le projet sans s’embourber dans un projet trop complexe, ou impliquant un trop grand nombre de participants. Néanmoins, il est également important d’inclure une combinaison d’utilisateurs, allant des utilisateurs précoces aux sceptiques. Les utilisateurs qui adoptent la solution vous aident à affiner votre communication future et votre plan d’activité. Les sceptiques vous aident à identifier et à résoudre les problèmes de blocage. Lorsque les sceptiques deviennent des champions, vous pouvez utiliser leurs commentaires pour identifier les facteurs de succès.

Votre plan pilote doit mettre en œuvre les objectifs d’entreprise que vous souhaitez atteindre à l’aide de Data Catalog. À mesure que vous tirez les enseignements du pilote initial, vous pouvez développer votre base d’utilisateurs. Un pilote fermé initial est approprié pour établir un succès mesurable, mais l’objectif ultime est une croissance naturelle ou virale. Avec une croissance naturelle de Data Catalog, les utilisateurs maîtrisent leur propre utilisation des données et peuvent influencer et inciter les autres utilisateurs à adopter ce service et à participer à sa réalisation.

### <a name="target-the-right-team"></a>Cibler l’équipe appropriée

Lorsque vous choisissez votre projet pilote, sélectionnez une équipe avec les scénarios les plus intéressants, qui résout un problème d’entreprise existant. Par exemple, un analyste d’entreprise crée des rapports à partir d’une base de données SQL Server. Le problème tient au fait qu’il ne connaît la source de données qu’après avoir discuté avec plusieurs collègues. Pour finir, après avoir perdu du temps à essayer de trouver les sources de données à utiliser, il découvre un classeur Excel qui contient une description de chacune d’elles. Même si le classeur Excel décrit correctement les tables dont il a besoin, il aurait trouvé ces sources de données rapidement si elles avaient été inscrites et annotées dans **Azure Data Catalog**.

### <a name="identify-data-heroes"></a>Identifier les personnes clés en termes de données

Votre premier projet pilote doit être associé à quelques personnes, qui produisent et consomment des données de telle sorte que l’équipe possède une représentation équilibrée.

**producteurs de données** sont des personnes disposant de connaissances sur les sources de données. Par exemple, David, membre d’une autre équipe a beaucoup travaillé sur les principales sources de données Adventure Works. Avant l’adoption **d’Azure Data Catalog**, David a créé un classeur Excel pour capturer des informations sur les sources de données Adventure Works.

**consommateurs de données** sont des personnes disposant de connaissances sur l’utilisation de données pour résoudre les problèmes d’entreprise. Par exemple, Nancy est une analyste d’entreprise, qui utilise des sources de données SQL Server Adventure Works pour analyser les données.

L’un des problèmes d’entreprise traités par **Azure Data Catalog** est de connecter les **producteurs de données** aux **consommateurs de données**. Pour ce faire, il fait office de référentiel central pour les informations sur les sources de données d’entreprise. À l’aide de Data Catalog, David inscrit les sources de données SQL Server Adventure Works. À l’aide du crowdsourcing, tout utilisateur qui découvre cette source de données peut partager son avis sur les données, en plus d’utiliser les données qu’il a découvertes. Par exemple, Nancy découvre les sources de données en effectuant une recherche dans le catalogue, puis partage ses connaissances approfondies relatives aux données.  À présent, les autres utilisateurs de l’organisation tirent parti des connaissances partagées par une recherche dans le catalogue de données.

* Pour en savoir plus sur l’inscription des sources de données, consultez la section [Inscrire des sources de données](data-catalog-get-started.md).
* Pour en savoir plus sur la découverte des sources de données, consultez la section [Rechercher des sources de données](data-catalog-get-started.md).

### <a name="start-small-and-focused"></a>Démarrer petit et ciblé

Pour la plupart des projets pilotes d’entreprise, vous devez alimenter le catalogue avec des sources de données de valeur élevée afin que les utilisateurs professionnels puissent rapidement constater la valeur de Data Catalog. Le service informatique est un bon emplacement pour commencer à identifier les sources de données qui seraient intéressantes pour votre équipe pilote. Pour les sources de données prises en charge, telles que SQL Server, nous recommandons d’utiliser l’outil d’inscription des sources de données **d’Azure Data Catalog**. Avec l’outil d’inscription des sources de données, vous pouvez inscrire un large éventail de sources de données, y compris des bases de données SQL Server et Oracle, et des rapports SQL Server Reporting Services. Pour obtenir la liste complète des sources de données actuelles, consultez l’article [Sources de données prises en charge par Azure Data Catalog](data-catalog-dsr.md).

Une fois que vous avez identifié et inscrit les principales sources de données, il est possible d’importer également les descriptions de sources de données stockées dans d’autres emplacements. L’API Data Catalog permet aux développeurs de charger des descriptions et des annotations à partir d’un autre emplacement, par exemple le classeur Excel créé et géré par David.

La section suivante décrit un exemple de projet de la société Adventure Works.

### <a name="an-example-project"></a>Exemple de projet

Pour cet exemple, Nancy, l’analyste d’entreprise, crée des rapports pour son équipe, en se servant des données d’une base de données SQL Server. Le problème tient au fait qu’il ne connaît la source de données qu’après avoir discuté avec plusieurs collègues. Ils auraient rapidement trouvé ces sources de données si elles avaient été inscrites et annotées dans un emplacement central, par exemple **Azure Data Catalog**.

Pour illustrer la façon dont Nancy et son équipe peuvent trouver des sources de données de valeur élevée, vous utilisez l’outil d’inscription des sources de données pour remplir Data Catalog d’informations (métadonnées) sur les sources de données. De cette façon, les informations sur la base de données sont à la disposition de l’équipe et de l’entreprise, et pas seulement d’une poignée de personnes. Une fois que les sources de données sont inscrites dans Data Catalog, Nancy et son équipe peuvent facilement les utiliser. Le résultat est un catalogue de données plus complet et mieux approprié pour son équipe et l’entreprise. Comme de plus en plus d’équipes adoptent Data Catalog, les sources de données métier deviennent plus faciles à trouver et à utiliser, ce qui permet donc une culture plus centrée sur les données pour en tirer davantage parti.

Pour en savoir plus sur l’outil d’inscription des sources de données, consultez l’article [Prise en main d’Azure Data Catalog](data-catalog-get-started.md).

Dans le cadre du projet pilote, l’équipe de Nancy utilise également des sources de données, qui sont décrites dans un classeur Excel tenu à jour par David et ses collègues. Puisque d’autres équipes de l’entreprise utilisent également des classeurs Excel pour décrire les sources de données, l’équipe informatique décide de créer un outil pour migrer le classeur Excel vers Data Catalog. En utilisant l’API REST Data Catalog pour importer les annotations existantes, l’équipe du projet pilote peut disposer d’un catalogue de données complet comprenant les métadonnées extraites des sources de données à l’aide de l’outil d’inscription des sources de données, et le compléter avec les informations précédemment documentées par les producteurs et les consommateurs de données sans nécessité de nouvelle saisie manuelle. Comme le catalogue de données d’entreprise s’étoffe, l’organisation peut utiliser l’outil d’inscription des sources de données pour les sources de données courantes et l’API Data Catalog pour les sources personnalisées et les scénarios rares.

> [!NOTE]
> Nous avons écrit un exemple d’outil qui utilise l’API **Azure Data Catalog** pour migrer un classeur Excel vers Data Catalog. Pour découvrir l’API Data Catalog et l’exemple d’outil, vous pouvez [télécharger l’exemple de code de classeur Ad Hoc](https://github.com/Azure-Samples/data-catalog-dotnet-excel-register-data-assets) et découvrir la documentation sur [l’API REST Azure Data Catalog](/rest/api/datacatalog/).

Une fois le projet pilote en place, il est temps d’exécuter votre plan d’adoption Data Catalog.

### <a name="execute"></a>Execute

À ce stade vous avez identifié les cas d’utilisation de Data Catalog ainsi que votre premier projet. Par ailleurs, vous avez inscrit les principales sources de données Adventure Works et ajouté des informations du classeur Excel existant à l’aide de l’outil créé par le service informatique. Il est maintenant temps de collaborer avec l’équipe pilote pour démarrer le processus d’adoption de Data Catalog.

Voici quelques conseils pour bien commencer :

* **Suscitez de l’enthousiasme** : les utilisateurs professionnels seront enthousiasmés s’ils pensent qu’**Azure Data Catalog** va leur faciliter la vie. Essayez de faire tourner la conversation autour de la solution et des avantages qu’elle fournit, et non autour de la technologie.
* **Facilitez le changement** : commencez petit et communiquez le plan aux utilisateurs professionnels. Pour réussir, il est essentiel d’impliquer les utilisateurs dès le début afin qu’ils influencent les résultats et soit partie prenante à la solution.
* **Préparez les utilisateurs précoces** : les utilisateurs précoces sont des utilisateurs professionnels passionnés par ce qu’ils font, et heureux de parler des avantages d’**Azure Data Catalog** à leurs homologues.
* **Ciblez la formation** : comme les utilisateurs professionnels n’ont pas besoin de tout savoir sur Data Catalog, il convient de cibler la formation pour prendre en main les objectifs d’équipe spécifiques. Concentrez-vous sur ce que les utilisateurs font et sur la façon dont certaines de leurs tâches peuvent être modifiées afin d’intégrer **Azure Data Catalog** à leur routine quotidienne.
* **Soyez disposé à échouer** : si le pilote n’atteint pas les résultats escomptés, réévaluez et identifiez des zones pour modifier/résoudre les problèmes dans le pilote avant de passer à une plus grande étendue.

Avant que votre équipe pilote ne s’implique dans l’utilisation de Data Catalog, planifiez une réunion de lancement pour discuter des attentes du projet pilote, et dispensez une formation initiale.

### <a name="set-expectations"></a>Définir des attentes

La définition d’objectifs et d’exceptions permet aux utilisateurs professionnels de se concentrer sur des livrables spécifiques. Pour suivre l’évolution du projet, attribuez régulièrement des travaux (par exemple : sur une base quotidienne ou hebdomadaire en fonction de l’étendue et de la durée du pilote). L’une des fonctionnalités les plus intéressantes de Data Catalog est le crowdsourcing des ressources de données afin que les utilisateurs professionnels puissent bénéficier des connaissances tirées des données d’entreprise. Il est conseillé d’attribuer ce travail très intéressant : demandez à chaque membre d’équipe pilote d’inscrire ou d’annoter au moins une source de données qu’il a utilisée. Consultez les sections [Inscrire une source de données](data-catalog-get-started.md) et [Annotation des sources de données](data-catalog-get-started.md).

Rencontrez régulièrement l’équipe pour passer en revue quelques-unes des annotations. Des annotations correctes sur les sources de données sont au cœur d’une adoption de Data Catalog réussie, car elles apportent des informations intéressantes sur les sources de données dans un emplacement central. Sans annotations de bonne qualité, les connaissances sur les sources de données restent dispersées dans toute l’entreprise. Consultez les sections [Annotation des sources de données](data-catalog-get-started.md).

Et, pour finir, il convient de tester le projet pour vérifier que les utilisateurs peuvent découvrir et comprendre les sources de données qu’ils doivent utiliser. Les utilisateurs du pilote doivent régulièrement tester le catalogue pour s’assurer que les sources de données qu’ils utilisent pour leurs tâches quotidiennes sont pertinentes. Lorsqu’une source de données requise est manquante ou si elle n’est pas correctement annotée, elle doit servir de rappel pour inscrire des sources de données supplémentaires ou pour fournir des annotations supplémentaires. Cette pratique n’ajoute pas seulement de la valeur au pilote : elle génère également des habitudes efficaces, qui sont reportées aux autres équipes à l’issue de l’application du pilote.

### <a name="provide-training"></a>Dispenser une formation

La formation doit être suffisante pour faire commencer les utilisateurs, et elle doit être adaptée aux objectifs et niveau d’expérience spécifiques des membres de l’équipe pilote. Pour commencer la formation, vous pouvez suivre les étapes décrites dans l’article [Prise en main d’Azure Data Catalog](data-catalog-get-started.md) . En outre, vous pouvez télécharger la [Présentation de la formation au projet pilote Azure Data Catalog](https://github.com/Azure-Samples/data-catalog-dotnet-get-started/blob/master/Azure%20Data%20Catalog%20Training.pptx?raw=true). Cette présentation PowerPoint doit vous aider à présenter Data Catalog aux membres de votre équipe pilote.

## <a name="conclusion"></a>Conclusion

Une fois que votre équipe pilote fonctionne relativement correctement et que vous avez atteint vos objectifs initiaux, vous devez développer l’adoption de Data Catalog à d’autres équipes. Appliquez et affinez ce que vous avez appris de votre projet pilote pour développer Data Catalog au sein de votre organisation.

Les utilisateurs précoces qui ont participé au pilote peuvent être utiles pour vanter les avantages de Data Catalog. Ils peuvent indiquer aux autres équipes comment Data Catalog a permis à leur équipe de résoudre des problèmes professionnels, de découvrir des sources de données plus facilement et de partager des informations sur les sources de données utilisées. Par exemple, les utilisateurs précoces de l’équipe pilote Adventure Works peuvent montrer aux autres utilisateurs combien il est facile de trouver des informations sur les ressources de données Adventure Works qui étaient auparavant difficiles à trouver et à comprendre.

Cet article concernait la prise en main **d’Azure Data Catalog** dans votre organisation. Nous espérons que vous avez réussi à démarrer un projet pilote Data Catalog et à développer Data Catalog au sein de votre organisation.

## <a name="next-steps"></a>Étapes suivantes

[Créer un catalogue Azure Data Catalog](data-catalog-get-started.md)
