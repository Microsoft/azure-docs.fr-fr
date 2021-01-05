---
title: Bonnes pratiques de déploiement
description: Cet article présente les meilleures pratiques de déploiement d’Azure Purview. Azure Purview permet à n’importe quel utilisateur d’inscrire, de découvrir, de comprendre et d’utiliser des sources de données.
author: shsandeep123
ms.author: sandeepshah
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 11/23/2020
ms.openlocfilehash: 48966e481f9cf8796c866b5c15a4e2a8616eade7
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97695858"
---
# <a name="azure-purview-deployment-best-practices"></a>Meilleures pratiques de déploiement d’Azure Purview

Cet article identifie des tâches courantes qui peuvent vous aider à déployer Purview en production. Vous pouvez accomplir ces tâches en plusieurs phases au cours d’un plusieurs mois. Même des organisations ayant déjà déployé Azure AD peuvent utiliser ce guide pour s’assurer de tirer le meilleur parti de leur investissement.

Un déploiement bien planifié d’une plateforme de gouvernance des données (telle qu’Azure Purview) peut offrir les avantages suivants :

- meilleure découverte des données ;
- collaboration analytique améliorée ;
- retour sur investissement maximisé.

## <a name="prerequisites"></a>Prérequis

* Accès à Microsoft Azure avec un abonnement de développement ou de production.
* Possibilité de créer des ressources Azure, dont Purview.
* Accès à des sources de données telles qu’Azure Data Lake Storage ou Azure SQL dans des environnements de test, de développement ou de production :
  * Pour Data Lake Storage, le rôle requis pour l’analyse est le rôle Lecteur.
  * Pour SQL, l’identité doit être en mesure d’interroger des tables pour échantillonner des classifications.
* Accès à Azure Security Center ou possibilité de collaborer avec l’administrateur de Security Center pour l’étiquetage des données.

## <a name="identify-objectives-and-goals"></a>Identifier les objectifs

De nombreuses organisations ont commencé leur parcours de gouvernance des données en développant des solutions individuelles répondant à des exigences spécifiques de groupes et de domaines de données isolés au sein de l’organisation. Bien que les expériences puissent varier en fonction du secteur, du produit et de la culture, la plupart des organisations trouvent difficile de maintenir des contrôles et stratégies cohérents pour ces types de solutions.

Voici quelques-uns des objectifs courants de gouvernance des données que vous pouvez identifier au cours des premières phases :

* maximisation de la valeur commerciale de vos données ;
* promotion d’une culture des données permettant aux consommateurs de facilement trouver, interpréter et approuver celles-ci ;
* renforcement de la collaboration entre les différentes unités commerciales pour offrir une expérience de données cohérente ;
* stimulation de l’innovation en accélérant l’analyse des données pour profiter des avantages du cloud ;
* réduction du temps nécessaire pour découvrir des données via des options en libre-service pour divers groupes de compétences ;
* raccourcissement du délai de mise sur le marché de solutions d’analyse qui améliorent le service pour les clients ;
* réduction des risques opérationnels dus à l’utilisation d’outils spécifiques d’un domaine et d’une technologie non prise en charge.

L’approche générale consiste à répartir ces objectifs généraux en divers objectifs et catégories spécifiques. Quelques exemples :

|Category|Objectif|
|---------|---------|
|Découverte|Les utilisateurs administratifs doivent être en mesure d’analyser des sources de données Azure et non-Azure (y compris des sources locales) pour collecter automatiquement des informations sur des ressources de données.|
|classification ;|La plateforme doit automatiquement classer les données en fonction d’un échantillonnage de celles-ci, et autoriser leur remplacement manuel à l’aide de classifications personnalisées.|
|Consommation|Les utilisateurs professionnels doivent être en mesure de trouver des informations sur chaque ressource pour les métadonnées métier et techniques.|
|Traçabilité|Chaque ressource doit présenter une vue graphique des jeux de données sous-jacents afin que les utilisateurs comprennent les sources d’origine et les modifications apportées.|
|Collaboration|La plateforme doit permettre aux utilisateurs de collaborer en fournissant des informations supplémentaires sur chaque ressource de données.|
|Rapports|Les utilisateurs doivent être en mesure d’afficher des rapports sur le patrimoine de données, notamment les données sensibles et les données nécessitant un enrichissement supplémentaire.|
|Gouvernance des données|La plateforme doit permettre à l’administrateur de définir des stratégies pour le contrôle d’accès et d’appliquer automatiquement l’accès aux données en fonction de chaque utilisateur.|
|Workflow|La plateforme doit être capable de créer et modifier le flux de travail afin de faciliter la montée en charge et l’automatisation de diverses tâches en son sein.|
|Intégration|D’autres technologies tierces, telles que la création de tickets ou l’orchestration, doivent pouvoir s’intégrer à la plateforme via des API REST ou de script.|

## <a name="top-questions-to-ask"></a>Principales questions à se poser

Une fois que votre organisation a adopté les objectifs de haut niveau, divers groupes soulèvent de nombreuses questions. Il est essentiel de compiler ces questions afin d’élaborer un plan répondant à tous les problèmes. Voici quelques exemples de questions que vous pouvez rencontrer pendant la phase initiale :

1. Quelles sont les principaux systèmes et sources de données de l’organisation ?
2. Pour les sources de données que Purview ne prend pas encore en charge, quelles sont mes options ?
3. De combien d’instances Purview avons-nous besoin ?
4. Qui sont les utilisateurs ?
5. Qui peut analyser les nouvelles sources de données ?
6. Qui peut modifier le contenu de Purview ?
7. Quel processus puis-je utiliser pour améliorer la qualité des données dans Purview ?
8. Comment démarrer la plateforme avec des ressources critiques, des termes de glossaire et des contacts existants ?
9. Comment effectuer l’intégration avec des systèmes existants ?
10. Comment rassembler des commentaires et créer un processus durable ?

Bien que vous n’ayez peut-être pas la réponse à la plupart de ces questions, celles-ci peuvent aider votre organisation à élaborer ce projet et à s’assurer que toutes les exigences impératives peuvent être respectées.

## <a name="include-the-right-stakeholders"></a>Inclure les parties prenantes appropriées

Afin de garantir la réussite de l’implémentation de Purview pour l’ensemble de l’entreprise, il est important d’impliquer les parties prenantes appropriées. Seules quelques personnes sont mises à contribution durant la phase initiale. Cependant, à mesure que croît l’étendue du projet, vous avez besoin de personnages supplémentaires pour y contribuer et formuler des commentaires.

Les parties prenantes clés que vous pouvez inclure sont les suivantes :

|Utilisateur|Rôles|
|---------|---------|
|**Directeur des données**|Supervise une série de fonctions qui peuvent inclure la gestion des données, la qualité des données, la gestion des données de référence, la science des données, l’aide à la décision et la création de stratégie de données. Peut être le commanditaire du projet d’implémentation de Purview.|
|**Propriétaire du domaine ou de l’entreprise**|Influence l’utilisation des outils et contrôle le budget.|
|**Analyste Data**|Est capable de cerner un problème et d’analyser les données pour aider les dirigeants à prendre des décisions.|
|**Architecte de données**|Conçoit des bases de données pour les applications métier stratégiques, ainsi que la conception et l’implémentation de la sécurité des données.|
|**Ingénieur Data**|Exploite et gère la pile de données, extrait des données de différentes sources, intègre et prépare des données, configure des pipelines de données.|
|**Data Scientist**|Crée des modèles analytiques et configure des produits de données auxquels accèdent les API.|
|**Administrateur de base de données**|Possède, suit et résout les incidents et demandes liés aux bases de données dans le cadre de contrats de niveau de service (SLA). Peut configurer des pipelines de données|
|**DevOps**|Développe et implémente des applications métier, ce qui peut inclure l’écriture de scripts et de fonctionnalités d’orchestration.|
|**Spécialiste en sécurité des données**|Évalue la sécurité globale du réseau et des données, qui implique des données en provenance et à destination de Purview.|

## <a name="identify-key-scenarios"></a>Identifier les principaux scénarios

Purview permet de gérer de manière centralisée la gouvernance des données dans le patrimoine de données d’une organisation couvrant des environnements cloud et locaux. Pour réussir l’implémentation, vous devez identifier les principaux scénarios critiques pour l’entreprise. Ces scénarios peuvent franchir les frontières d’unités commerciales ou avoir un impact sur plusieurs personnes, tant en amont qu’en aval.

Ils peuvent être écrits de différentes façons, mais vous devez inclure au moins les cinq dimensions suivantes :

1. Personnage : qui sont les utilisateurs ?
2. Système source : quelles sont les sources de données telles qu’Azure Data Lake Storage Gen2 ou Azure SQL Database ?
3. Zone d’impact : quelle est la catégorie de ce scénario ?
4. Scénarios détaillés : comment les utilisateurs utilisent-ils Purview pour résoudre les problèmes ?
5. Résultat attendu : quels sont les critères de réussite ?

Les scénarios doivent être spécifiques, actionnables et exécutables avec des résultats mesurables. Voici quelques exemples de scénarios que vous pouvez utiliser :

|Scénario|Détail|Utilisateur|
|---------|---------|---------|
|Cataloguer les ressources vitales pour l’entreprise|J’ai besoin d’informations sur les différents jeux de données afin de bien comprendre leur contenu. Ce scénario inclut des métadonnées métier et techniques sur le jeu de données dans le catalogue. Les sources de données sont Azure Data Lake Storage Gen2, Azure Synapse DW et/ou Power BI. Ce scénario inclut également une ressource locale telle que SQL Server.|Analyste métier, scientifique Données, ingénieur Données|
|Découvrir les ressources vitales pour l’entreprise|J’ai besoin d’un moteur de recherche capable d’effectuer des recherches dans toutes les métadonnées du catalogue. Je dois pouvoir utiliser des termes techniques et des termes métier pour effectuer des recherches simples ou complexes à l’aide de caractères génériques.|Analyste métier, scientifique Données, ingénieur Données, administrateur des données|
|Suivre des données pour comprendre leur origine et résoudre les problèmes liés à celles-ci|J’ai besoin d’une traçabilité des données de rapports, de prévisions ou de modèles jusqu’à leur source d’origine, ainsi que de comprendre leurs modifications et emplacements tout au long de leur cycle de vie. Ce scénario doit prendre en charge les pipelines de données hiérarchisés Azure Data Factory et Databricks.|Ingénieur Données, scientifique Données|
|Enrichir les métadonnées de ressources de données critiques|J’ai besoin d’enrichir le jeu de données dans le catalogue avec des métadonnées techniques générées automatiquement, par exemple, à des fins de classification et d’étiquetage.|Ingénieur Données, propriétaire de domaine ou d’entreprise|
|Gérer les ressources de données avec une expérience utilisateur conviviale|J’ai besoin d’un glossaire métier pour les métadonnées spécifiques de l’activité. Les utilisateurs dans l’entreprise peuvent utiliser Purview pour des scénarios en libre-service afin d’annoter leurs données et de faciliter leur découverte via une recherche.|Propriétaire de domaine ou d’entreprise, analyste métier, scientifique Données, ingénieur Données|

## <a name="deployment-models"></a>Modèles de déploiement

Si vous n’avez qu’un petit groupe de personnes utilisant Purview avec des cas d’usage correspondant à une consommation de base, l’approche peut être aussi simple qu’une instance Purview pour servir l’ensemble du groupe. Toutefois, vous pouvez également vous demander si votre organisation a besoin de plusieurs instances Purview. Et si vous utilisez plusieurs instances Purview, comment les employés peuvent-ils promouvoir les ressources d’une étape à l’autre.

### <a name="determine-the-number-of-purview-instances"></a>Déterminer le nombre d’instances Purview

Dans la plupart des cas, il ne doit y avoir qu’un seul compte Purview pour l’ensemble de l’organisation. Cette approche tire le meilleur parti des « effets réseau » où la valeur de la plateforme augmente de façon exponentielle en fonction des données qui résident à l’intérieur de celle-ci.

Toutefois, il existe des exceptions à ce modèle :

1. **Test de nouvelles configurations** : les organisations peuvent créer plusieurs instances pour tester des configurations d’analyse ou des classifications dans des environnements isolés. Bien qu’une fonctionnalité « contrôle de version » soit disponible dans certaines zones de la plateforme, tel le glossaire, il serait plus facile d’avoir une instance « à usage unique » pour tester librement.
2. **Séparation des tests, de la préproduction et de la production** : des organisations souhaitent créer des plateformes distinctes pour divers types de données stockées dans différents environnements. Cette solution n’est pas recommandée, car les données de ce genre sont de types de contenus différents. Vous pouvez utiliser un terme de glossaire au niveau supérieur de la hiérarchie, ou une catégorie pour séparer les types de contenus.
3. **Conglomérats et modèle fédéré** : les conglomérats ont souvent de nombreuses unités commerciales qui fonctionnent séparément et, dans certains cas, ne partagent même pas la facturation. Dans ce cas, l’organisation finit par créer une instance Purview pour chaque unité commerciale. Ce modèle n’est pas idéal, mais peut être nécessaire, en particulier parce que, souvent, les unités commerciales ne souhaitent pas partager la facturation.
4. **Conformité** : il existe des régimes de conformité stricts qui traitent même les métadonnées comme sensibles, et exigent qu’elles se trouvent dans une zone géographique spécifique. Si une société possède plusieurs zones géographiques, la seule solution consiste à avoir plusieurs instances Purview, à raison d’une par zone géographique.

### <a name="create-a-process-to-move-to-production"></a>Créer un processus pour passer en production

Certaines organisations peuvent décider de simplifier les choses en utilisant une seule version de production de Purview. Elles n’ont probablement pas besoin d’aller au-delà des scénarios de découverte, de recherche et de navigation. Si certaines ressources comportent des termes de glossaire incorrects, il est tout à fait possible de permettre aux utilisateurs d’effectuer des corrections automatiques. Toutefois, la plupart des organisations qui souhaitent déployer Purview dans différentes unités commerciales doivent avoir une forme quelconque de processus et de contrôle.

Un autre aspect important à inclure dans votre processus de production est la façon dont les classifications et les étiquettes peuvent être migrées. Purview offre plus de 90 classifieurs système. Vous pouvez appliquer des classifications système ou personnalisées à des ressources de type fichier, table ou colonne. Les classifications sont similaires aux étiquettes d’objet et permettent de marquer et d’identifier le contenu d’un type spécifique qui se trouve dans votre patrimoine de données en cours d’analyse. Elles servent à identifier des catégories de types de classifications dans les données de l’organisation, puis à regrouper les stratégies à appliquer à chaque catégorie. Purview utilise les mêmes types d’informations sensibles que Microsoft 365, ce qui vous permet d’étendre votre protection et vos stratégies de sécurité existantes à l’ensemble de votre patrimoine de contenus et de données. Le service peut analyser et classer automatiquement des documents. Par exemple, si vous avez un fichier nommé multiple.docx contenant un numéro d’identification national, Azure Purview ajoute une classification telle que Numéro d’identification national européen à la page des détails de la ressource.

Purview inclut plusieurs zones dans lesquelles les administrateurs du catalogue doivent garantir la cohérence et les meilleures pratiques de maintenance tout au long de son cycle de vie :

* **Ressources de données** : les sources de données doivent être réanalysées dans l’ensemble des environnements. Il n’est pas recommandé de les analyser uniquement dans le cadre du développement, puis de les régénérer à l’aide d’API en production. La raison principale est que les scanneurs Purview effectuent beaucoup plus de « câblage » en coulisses sur les ressources de données, ce qui peut complexifier leur déplacement vers une autre instance Purview. Il est beaucoup plus facile d’ajouter simplement la même source de données en production et de réanalyser les sources. La meilleure pratique générale consiste à avoir une documentation sur l’ensemble des analyses, connexions et mécanismes d’authentification utilisés.
* **Ensembles de règles d’analyse** : il s’agit de votre collection de règles affectées à une analyse spécifique, telle que le type de fichier et les classifications à détecter. Si vous ne disposez pas de nombreuses règles d’analyse, vous pouvez les recréer manuellement en production. Cela nécessite un processus interne et une bonne documentation. Toutefois, si vos ensembles de règles changent à un rythme quotidien ou hebdomadaire, vous pouvez résoudre ce cas en explorant l’itinéraire de l’API REST.
* **Classifications personnalisées** : il se peut que vos classifications ne changent pas régulièrement. Pendant la phase initiale de déploiement, il peut être nécessaire de prendre un certain temps pour comprendre les différentes exigences relatives à l’apparition de classifications personnalisées. Toutefois, une fois cela réglé, peu de changements seront requis. C’est pourquoi nous vous recommandons de migrer manuellement les classifications personnalisées ou d’utiliser l’API REST.
* **Glossaire** : il est possible d’exporter et d’importer des termes de glossaire par le biais de l’expérience utilisateur. Pour les scénarios d’automatisation, vous pouvez également utiliser l’API REST.
* **Stratégies de modèle de jeu de ressources** : l’application de cette fonctionnalité est très avancée pour des organisations classiques. Dans certains cas, votre Azure Data Lake Storage possède des conventions d’affectation de noms de dossiers et une structure spécifique qui peuvent entraîner des problèmes lorsque Purview génère l’ensemble de ressources. Votre unité commerciale peut également souhaiter modifier la construction de l’ensemble de ressources avec des personnalisations supplémentaires pour répondre aux besoins métier. Pour ce scénario, il est préférable d’effectuer le suivi de toutes les modifications via l’API REST, et de documenter les modifications via la plateforme de contrôle de version externe.
* **Attribution de rôle** : c’est là que vous contrôlez qui a accès à Purview et les autorisations dont il dispose. Purview offre également l’API REST pour prendre en charge l’exportation et l’importation d’utilisateurs et de rôles, mais cela n’est pas compatible avec l’API Atlas. La recommandation consiste à attribuer un groupe de sécurité Azure et à gérer l’appartenance au groupe à la place.

### <a name="plan-and-implement-different-integration-points-with-purview"></a>Planifier et implémenter différents points d’intégration avec Purview

Une organisation mature dispose probablement déjà d’un catalogue de données existant. La question clé est de savoir s’il faut continuer à utiliser la technologie existante et à synchroniser avec Purview. Purview permet de publier des informations via les API Atlas mais celles-ci ne sont pas destinées à prendre en charge ce type de scénario. Certaines organisations peuvent décider initialement de démarrer l’utilisation de Purview en migrant les ressources de données existantes à partir d’autres solutions de catalogue de données. Cela peut être effectué via les API Atlas dans le cadre d’une approche unidirectionnelle. La synchronisation entre les différentes technologies de catalogue ne doit pas être prise en compte dans la conception à long terme. Généralement, chaque unité commerciale peut continuer à utiliser les solutions existantes pour les ressources de données plus anciennes, tandis que Purview est utilisé pour analyser les sources de données plus récentes.

Pour d’autres scénarios d’intégration, tels que la création de tickets, l’interface utilisateur personnalisée et l’orchestration, vous pouvez utiliser des API Atlas et des points de terminaison Kafka. En général, il existe quatre points d’intégration avec Purview :

* **Ressource de données** : permet à Purview d’analyser les ressources d’un magasin afin d’énumérer ce que sont ces ressources et de collecter toutes les métadonnées immédiatement disponibles à leur sujet. Pour SQL, il peut s’agir d’une liste de bases de données, de tables, de procédures stockées, d’affichages et de données de configuration qui se trouvent dans des emplacements tels que `sys.tables`. Par exemple, pour Azure Data Factory (ADF), il peut s’agir de l’énumération de tous les pipelines et de l’obtention de données relatives au moment de leur création, à leur dernière exécution et à leur état actuel.
* **Traçabilité** : permet à Purview de collecter des informations à partir d’un système de mutation d’analyse ou de données sur la façon dont les données sont déplacées. Par exemple, pour Spark, il peut s’agir de la collecte d’informations relatives à l’exécution d’un bloc-notes pour voir les données que celui-ci a ingérées, la manière dont il les a transformées et l’emplacement de leur sortie. Pour SQL, il peut s’agir d’analyser les journaux des requêtes pour rétroconcevoir les opérations de mutation qui ont été exécutées et ce qu’elles ont fait. Nous prenons en charge la traçabilité d’envoi et d’extraction en fonction des besoins.
* **Classification**  : permet à Purview d’extraire des échantillons physiques de sources de données et de les soumettre à notre système de classification. Le système de classification identifie la sémantique d’un élément de données. Par exemple, nous pouvons déterminer qu’un fichier est un fichier Parquet comportant trois colonnes dont la troisième est une chaîne. Toutefois, les classifieurs que nous exécutons sur les échantillons nous disent que la chaîne est un nom, une adresse ou un numéro de téléphone. L’éclairage de ce point d’intégration signifie que nous avons défini la manière dont Purview peut ouvrir des objets tels que des blocs-notes, des pipelines, des fichiers Parquet, des tables et des conteneurs.
* **Expérience incorporée** : des produits qui offrent une expérience de type « studio » (par exemple, ADF, Synapse, SQL Studio, PBI et Dynamics) visent généralement à permettre aux utilisateurs de découvrir les données avec lesquelles ils souhaitent interagir, ainsi que les emplacements de sortie des données. Le catalogue de Purview peut vous aider à accélérer ces expériences en offrant une expérience d’incorporation. Celle-ci peut se manifester au niveau de l’API ou de l’expérience utilisateur, au choix du partenaire. En incorporant un appel à Purview, l’organisation peut tirer parti de la carte du patrimoine de données de Purview pour rechercher des ressources de données, voir la traçabilité, vérifier les schémas, ainsi qu’examiner les évaluations, les contacts, etc.

## <a name="phase-1-pilot"></a>Phase 1 : Pilote

Au cours de cette phase, Purview doit être créé et configuré pour un très petit ensemble d’utilisateurs. En règle générale, il s’agit simplement d’un groupe de 2 à 3 personnes qui collaborent pour exécuter des scénarios de bout en bout. Celles-ci sont considérées comme les partisans de Purview dans leur organisation. L’objectif principal de cette phase est de s’assurer que les fonctionnalités clés peuvent être obtenues et que les parties prenantes appropriées sont informées du projet.

### <a name="tasks-to-complete"></a>Tâches à effectuer

|Tâche|Detail|Duration|
|---------|---------|---------|
|Regrouper et accepter les exigences|Discutez avec toutes les parties prenantes pour recueillir un ensemble complet d’exigences. Différents personnages doivent participer à l’accord sur un sous-ensemble d’exigences à respecter à chaque phase du projet.|1 semaine|
|Configurer le starter kit|Consultez le [Démarrage rapide de Purview](create-catalog-portal.md) et configurez le [Starter kit de Purview](tutorial-scan-data.md) pour présenter la démonstration de Purview à toutes les parties prenantes.|1 jour|
|Naviguer dans Purview|Comprenez comment utiliser Purview à partir de la page d’accueil.|1 jour|
|Configurer ADF pour la traçabilité|Identifiez les principaux pipelines et ressources de données. Recueillez toutes les informations requises pour se connecter à un compte ADF interne.|1 jour|
|Analyser une source de données telle qu’Azure Data Lake Storage|Ajoutez la source de données et configurer une analyse. Assurez-vous que l’analyse détecte toutes les ressources.|2 jours|
|Rechercher et parcourir|Autorisez les utilisateurs finaux à accéder à Purview et à effectuer des scénarios de recherche et de parcours de bout en bout.|1 jour|

### <a name="acceptance-criteria"></a>Critères d'acceptation

* Le compte Purview est créé avec succès dans l’abonnement de l’organisation sous le locataire de l’organisation.
* Un petit groupe d’utilisateurs avec plusieurs rôles peut accéder à Purview.
* Purview est configuré pour analyser au moins une source de données.
* Les utilisateurs doivent être en mesure d’extraire des valeurs clés de Purview, par exemple :
  * Rechercher et parcourir
  * Traçabilité
* Les utilisateurs doivent être en mesure d’attribuer la propriété de la ressource dans la page des ressources.
* Présentation et démonstration pour sensibiliser les principales parties prenantes.
* Assentiment de direction à l’approbation de ressources supplémentaires pour la phase Produit minimum viable .

## <a name="phase-2-minimum-viable-product"></a>Phase 2 : Produit minimum viable

Une fois que vous disposez des exigences convenues et des unités commerciales participantes à l’intégration de Purview, l’étape suivante consiste à travailler sur une mise en production de Produit minimum viable. Au cours de cette phase, vous allez étendre l’utilisation de Purview à d’autres utilisateurs qui auront des besoins supplémentaires horizontalement et verticalement. Il y aura des scénarios clés à respecter horizontalement pour tous les utilisateurs, tels que les termes du glossaire, la recherche et la navigation. Il y aura également des exigences approfondies verticalement pour chaque unité commerciale ou groupe afin de couvrir des scénarios de bout en bout spécifiques tels que la traçabilité d’Azure Data Lake Storage à Azure Synapse DW, puis à Power BI.

### <a name="tasks-to-complete"></a>Tâches à effectuer

|Tâche|Detail|Duration|
|---------|---------|---------|
|[Analyser Azure Synapse Analytics](register-scan-azure-synapse-analytics.md)|Commencez à intégrer vos sources de base de données et à les analyser pour remplir les ressources clés.|2 jours|
|[Créer des classifications et des règles personnalisées](create-a-custom-classification-and-classification-rule.md)|Une fois vos ressources analysées, vos utilisateurs peuvent se rendre compte qu’il existe d’autres cas d’usage pour une classification plus avancée à côté des classifications par défaut de Purview.|2 à 4 semaines|
|[Analyser Power BI](register-scan-power-bi-tenant.md)|Si votre organisation utilise Power BI, vous pouvez analyser Power BI afin de collecter toutes les ressources de données qu’utilisent les scientifiques Données ou analystes Données qui ont pour exigence d’inclure la traçabilité à partir de la couche de stockage.|1 à 2 semaines|
|[Importer les termes de glossaire](how-to-create-import-export-glossary.md)|Dans la plupart des cas, votre organisation peut déjà développer une collection de termes de glossaire et une affectation de termes à des ressources. Cela nécessite un processus d’importation dans Purview via un fichier. csv.|1 semaine|
|Ajouter des contacts aux ressources|Pour les ressources principales, vous pouvez établir un processus pour permettre à d’autres personnages d’attribuer des contacts ou d’importer via des API REST.|1 semaine|
|Ajouter des étiquettes sensibles et une analyse|Cela peut être facultatif pour certaines organisations, en fonction de l’utilisation de l’étiquetage à partir de M365.|1 à 2 semaines|
|Obtenir une classification et des insights sensibles|Pour créer des rapports et insights dans Purview, vous pouvez accéder à cette fonctionnalité afin d’obtenir divers rapports et de fournir une présentation à la direction.|1 jour|
|Intégrer des utilisateurs supplémentaires avec des utilisateurs gérés par Purview|Cette étape nécessite que l’administrateur Purview collabore avec l’administrateur Azure Active Directory pour établir de nouveaux groupes de sécurité afin d’accorder l’accès à Purview.|1 semaine|

### <a name="acceptance-criteria"></a>Critères d'acceptation

* Intégrer avec succès un groupe plus grand d’utilisateurs à Purview (> 50)
* Analyser les sources de données critiques métier
* Importer et attribuer tous les termes de glossaire critiques
* Tester avec succès l’étiquetage important sur les ressources clés
* Suivre des scénarios minimums pour les utilisateurs des unités commerciales participantes

## <a name="phase-3-pre-production"></a>Phase 3 : Préproduction

Une fois la phase Produit minimum viable passée, il est temps de planifier une étape majeure de préproduction. Votre organisation peut décider d’avoir une instance distincte de Purview pour la pré-production et la production, ou de conserver la même instance, mais limiter l’accès. Toujours au cours de cette phase, vous pouvez inclure l’analyse de sources de données locales telles que SQL Server. S’il existe un écart dans les sources de données non prises en charge par Purview, il est temps d’explorer l’API Atlas pour comprendre les options supplémentaires.

### <a name="tasks-to-complete"></a>Tâches à effectuer

|Tâche|Detail|Duration|
|---------|---------|---------|
|Affiner votre analyse à l’aide d’un ensemble de règles d’analyse|Votre organisation disposera de nombreuses sources de données pour la pré-production. Il est important de prédéfinir des critères clés pour l’analyse afin que les classifications et l’extension de fichier puissent être appliquées systématiquement.|1 à 2 jours|
|Évaluer la disponibilité des régions pour l’analyse|Selon la région des sources de données et les exigences organisationnelles en matière de conformité et de sécurité, vous souhaiterez peut-être prendre en compte les régions qui doivent être disponibles pour l’analyse.|1 jour|
|Comprendre le concept de pare-feu lors de l’analyse|Cette étape nécessite une certaine exploration de la façon dont l’organisation configure son pare-feu et dont Purview peut s’authentifier pour accéder aux sources de données à des fins d’analyse.|1 jour|
|Comprendre le concept de liaison privée lors de l’analyse|Si votre organisation utilise le service Private Link, vous devez établir le fondement de la sécurité réseau pour inclure une liaison privée dans les exigences.|1 jour|
|[Analyser le SQL Server local](register-scan-on-premises-sql-server.md)|Cette option est facultative si vous disposez d’un SQL Server local. L’analyse nécessite la configuration d’un [runtime d’intégration auto-hébergé](manage-integration-runtimes.md) et l’ajout de SQL Server en tant que source de données.|1 à 2 semaines|
|Utiliser l’API REST Purview pour les scénarios d’intégration|Si vous avez des exigences pour intégrer Purview avec d’autres technologies tierces telles qu’un système d’orchestration ou de création de tickets, vous pouvez explorer la zone de l’API REST.|1 à 4 semaines|
|Comprendre la tarification de Purview|Cette étape fournit à l’organisation des informations financières importantes pour la prise de décision.|1 à 5 jours|

### <a name="acceptance-criteria"></a>Critères d'acceptation

* Intégrer avec succès au moins une unité commerciale avec tous les utilisateurs
* Analyser une source de données locale telle que SQL Server
* Effectuer la preuve de concept d’au moins un scénario d’intégration à l’aide de l’API REST
* Élaborer un plan de passage en production, qui doit inclure des zones clés sur l’infrastructure et la sécurité

## <a name="phase-4-production"></a>Phase 4 : Production

Les phases ci-dessus doivent être suivies pour créer une gouvernance des informations efficace, qui constitue le fondement de meilleurs programmes de gouvernance. La gouvernance des données permet à votre organisation de se préparer à des tendances croissantes, telles que l’intelligence artificielle, Hadoop, l’IoT et la blockchain. Ce n’est qu’un début pour de nombreuses données et analytiques, et il y a beaucoup plus d’aspects qui peuvent être abordés. Cette solution produirait les résultats suivants :

* **Orientation entreprise** : une solution adaptée aux besoins de l’entreprise et aux scénarios en lien avec des exigences techniques.
* **Préparation à l’avenir** : une solution maximisera les fonctionnalités par défaut de la plateforme et utilisera des pratiques sectorielles standardisées pour les activités de configuration ou de script, afin de prendre en charge les progrès ou l’évolution de la plateforme.

### <a name="tasks-to-complete"></a>Tâches à effectuer

|Tâche|Detail|Duration|
|---------|---------|---------|
|Analyser les sources de données de production avec le pare-feu activé|Si cette option est disponible lorsque le pare-feu est en place, il est important d’explorer les options de renforcement de votre infrastructure.|1 à 5 jours|
|Activer Azure Private Link|Si cette option est disponible lors de l’utilisation du service Private Link. Sinon, vous pouvez l’ignorer, car il s’agit d’un critère obligatoire lorsque le service Private Link est activé.|1 à 5 jours|
|Créer un flux de travail automatisé|Un flux de travail est important pour automatiser des processus tels que l’approbation, la réaffectation, la révision et la gestion des problèmes.|2 à 3 semaines|
|Documentation sur l’opération|La gouvernance des données n’est pas un projet ponctuel. Il s’agit d’un programme continu qui permet d’alimenter la prise de décision pilotée par les données et de créer des opportunités pour l’entreprise. Il est essentiel de documenter la procédure clé et les normes métier.|1 semaine|

### <a name="acceptance-criteria"></a>Critères d'acceptation

* Intégrer avec succès l’ensemble des unités commerciales et de leurs utilisateurs
* Répondre avec succès aux exigences en matière d’infrastructure et de sécurité pour la production
* Couvrir correctement tous les cas d’usage requis par les utilisateurs

## <a name="platform-hardening"></a>Renforcement de la plateforme

Des étapes de renforcement supplémentaires peuvent être entreprises :

* Augmenter la posture de sécurité en activant l’analyse des ressources de pare-feu ou l’utilisation du service Private Link
* Affiner l’analyse d’étendue pour améliorer les performances d’analyse
* Utiliser des API REST pour exporter des métadonnées et propriétés critiques pour la sauvegarde et la récupération
* Utiliser un flux de travail pour automatiser la création de tickets et les événements afin d’éviter les erreurs humaines

## <a name="next-steps"></a>Étapes suivantes

- [Tutoriel : Exécuter le starter kit et analyser les données](tutorial-scan-data.md)
- [Tutoriel : Parcourir la page d’accueil et rechercher une ressource](tutorial-asset-search.md)