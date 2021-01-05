---
title: Présentation d’Azure Purview (préversion)
description: Cet article présente une vue d’ensemble d’Azure Purview, y compris ses fonctionnalités et les problèmes qu’il résout. Azure Purview permet à n’importe quel utilisateur de s’inscrire, de découvrir, de comprendre et d’utiliser des sources de données.
author: hophanms
ms.author: hophan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: overview
ms.date: 11/30/2020
ms.openlocfilehash: 77da3406897c31e48d808541f1f9d579eb0a9cb7
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97401540"
---
# <a name="what-is-azure-purview"></a>Qu’est-ce qu’Azure Purview ?

> [!IMPORTANT]
> Azure Purview est actuellement disponible en PRÉVERSION. L’[Avenant aux conditions d’utilisation pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) contient des conditions légales supplémentaires qui s’appliquent aux fonctionnalités Azure en version bêta, en préversion ou pas encore en disponibilité générale.

Azure Purview est un service unifié de gouvernance des données qui vous aide à gérer et à régir vos données locales, multiclouds et SaaS (software-as-a-service). Créez facilement une carte holistique et à jour du paysage de vos données, avec la découverte automatisée des données, la classification des données sensibles et la traçabilité des données de bout en bout. Donnez les moyens aux consommateurs de données de trouver des données utiles et fiables.

Le mappage des données Azure Purview fournit la base de la découverte des données et une gouvernance efficace des données. Le mappage des données Purview est un service PaaS natif cloud, qui capture les métadonnées relatives aux données d’entreprise présentes dans les systèmes d’exploitation et d’analytique, en local et dans le cloud. Le mappage des données Purview est automatiquement mis à jour avec le système intégré de classification et d’analyse automatisé. Les utilisateurs professionnels peuvent configurer et utiliser le mappage des données Purview à l’aide d’une interface utilisateur intuitive, et les développeurs interagir programmatiquement avec le mappage des données au moyen des API Apache Atlas 2.0 open source.

Le mappage des données Azure Purview alimente le catalogue de données Purview et les insights de données Purview, sous la forme d’expériences unifiées dans Purview Studio.
 
Avec le catalogue de données Purview, les utilisateurs commerciaux comme les utilisateurs techniques peuvent rapidement et facilement trouver des données pertinentes en utilisant un procédé de recherche doté de filtres à différents calibres, tels que les termes de glossaire, les classifications, les étiquettes de confidentialité, etc. Pour les experts techniques ainsi que les gestionnaires et responsables de données, le catalogue de données Purview offre des fonctionnalités de collecte de données, telles que la gestion de glossaire métier et la possibilité d’automatiser l’étiquetage des ressources de données avec des termes de glossaire. Les producteurs et consommateurs de données peuvent également afficher la traçabilité des ressources de données, depuis des systèmes opérationnels locaux jusqu’à la consommation dans un système d’analytique tel que Power BI, en passant par le mouvement, la transformation et l’enrichissement avec divers systèmes de stockage et de traitement de données dans le cloud.

Avec les insights de données Purview, les responsables des données et les chargés de la sécurité peuvent profiter d’une vue d’ensemble et comprendre en un clin d’œil quelles données sont en cours d’analyse, à quel endroit se trouvent les données sensibles et comment elles se déplacent.

## <a name="discovery-challenges-for-data-consumers"></a>Défis des consommateurs de données en matière de découverte

En règle générale, la détection des sources de données d’entreprise est un processus organique basé sur un savoir communautaire. Cette approche soulève de nombreuses difficultés pour les entreprises qui souhaitent tirer le meilleur parti de leurs informations :

* Étant donné qu’aucun emplacement centralisé ne leur permet d’inscrire des sources de données, les utilisateurs peuvent ignorer l’existence d’une source de données, sauf s’ils entrent en contact avec elle dans le cadre d’un autre processus.
* À moins de connaître l’emplacement d’une source de données, les utilisateurs ne peuvent pas se connecter aux données à l’aide d’une application cliente. Les expériences d’utilisation de données requièrent de la part des utilisateurs de connaître la chaîne de connexion ou le chemin d’accès.
* L’utilisation voulue des données est invisible aux yeux des utilisateurs, sauf s’ils connaissent l’emplacement de la documentation d’une source de données. La documentation et les sources de données peuvent se trouver à plusieurs endroits, et être consommées via différents types d’utilisations.
* Si les utilisateurs ont des questions sur une ressource d’informations, ils doivent localiser l’expert ou l’équipe responsable des données et les faire intervenir en mode hors connexion. Il n’existe aucun lien explicite entre les données et les experts qui ont comme projet de les utiliser.
* Sauf si un utilisateur comprend le processus de demande d’accès à la source de données, la détection de la source de données et sa documentation ne lui permettra pas d’accéder aux données.

## <a name="discovery-challenges-for-data-producers"></a>Défis des producteurs de données en matière de découverte

Pendant que les consommateurs de données relèvent les défis mentionnés précédemment, les utilisateurs responsables de la production et de la gestion des ressources d’informations relèvent, quant à eux, des défis qui leur sont propres :

* L’annotation des sources de données avec des métadonnées descriptives est souvent peine perdue. En général, les applications clientes ignorent les descriptions stockées dans la source de données.
* La création d’une documentation pour des sources de données peut s’avérer difficile, et c’est une responsabilité permanente que celle de maintenir la synchronisation de la documentation avec les sources de données. Les utilisateurs peuvent ne plus avoir confiance dans la documentation si celle-ci paraît obsolète.
* Créer et gérer une documentation pour des sources de données est une tâche plutôt longue et complexe. Rendre disponible cette documentation pour tous les utilisateurs de la source de données peut l’être plus encore.
* Restreindre l’accès à la source de données et veiller à ce que les utilisateurs de données sachent comment demander l’accès est un défi récurrent.

Lorsqu’ils sont combinés, ces défis représentent un obstacle majeur pour les entreprises qui souhaitent encourager et promouvoir l’utilisation et la compréhension des données d’entreprise.

## <a name="discovery-challenges-for-security-administrators"></a>Défis des administrateurs de sécurité en matière de découverte

Les utilisateurs qui sont chargés de garantir la sécurité des données de leur organisation peuvent se retrouver face aux défis répertoriés plus haut en tant que consommateurs et producteurs de données, ainsi qu’aux défis supplémentaires suivants :

* Les données d’une organisation sont en constante évolution, sans cesse stockées et partagées dans de nouvelles directions. La tâche de découverte, de protection et d’administration de vos données sensibles est une tâche sans fin. Vous souhaitez vous assurer que le contenu de votre organisation est partagé avec les personnes et les applications qui conviennent, et avec les autorisations appropriées.
* La compréhension des niveaux de risque dans les données de votre organisation nécessite d’explorer en profondeur votre contenu, de rechercher les mots clés, les modèles RegEx et/ou les types de données sensibles. Les types de données sensibles peuvent inclure des numéros de carte de crédit, des numéros de Sécurité sociale ou des numéros de comptes bancaires, pour n’en citer que quelques-uns. Vous devez surveiller en permanence toutes les sources de données à la recherche du contenu sensible, car même la plus infime perte de données peut être capitale pour votre organisation.
* S’assurer que votre organisation continue de se conformer aux stratégies de sécurité de l’entreprise constitue une mission difficile, car votre contenu évolue et change alors même que ces exigences et stratégies sont mises à jour face à des réalités numériques en mutation. Les administrateurs de la sécurité sont souvent chargés de garantir la sécurité des données dans les plus brefs délais.

## <a name="azure-purview-advantages"></a>Avantages d’Azure Purview

Azure Purview est conçu pour résoudre les problèmes mentionnés dans les sections précédentes, et permettre aux entreprises de tirer le meilleur parti de leurs ressources d’informations existantes. Le catalogue rend les sources de données facilement détectables et compréhensibles par les utilisateurs qui gèrent les données.

Azure Purview fournit un service cloud dans lequel vous pouvez inscrire des sources de données. Lors de l’inscription, les données demeurent à leur emplacement existant, mais une copie de leurs métadonnées, ainsi qu’une référence à l’emplacement de la source de données, sont ajoutées à Azure Purview. Les métadonnées sont également indexées de manière à ce que chaque source de données soit facilement détectable via la recherche, et compréhensible pour les utilisateurs qui la découvrent.

Après avoir inscrit une source de données, vous pouvez enrichir ses métadonnées. C’est soit l’utilisateur ayant inscrit la source de données, soit un autre utilisateur de l’entreprise qui ajoute les métadonnées. Tous les utilisateurs peuvent annoter une source de données en fournissant des descriptions, des étiquettes ou d’autres métadonnées en vue de la demande d’accès à la source de données. Ces métadonnées descriptives complètent les métadonnées structurelles, comme les noms de colonnes et les types de données, qui sont inscrites à partir de la source de données.

La détection, la compréhension et l’utilisation des sources de données sont le principal objectif de l’inscription des sources. Les utilisateurs professionnels peuvent avoir besoin de données pour le décisionnel, le développement d’applications, la science des données ou toute autre tâche nécessitant des données adéquates. Ils utilisent l’expérience de découverte du catalogue de données pour rechercher rapidement des données correspondant à leurs besoins, pour comprendre les données afin d’évaluer leur pertinence par rapport à l’utilisation et pour utiliser les données en ouvrant la source de données dans l’outil de leur choix.

Dans le même temps, les utilisateurs peuvent contribuer au catalogue en balisant, en documentant et en annotant les sources de données qui ont déjà été enregistrées. Ils peuvent également enregistrer de nouvelles sources de données, qui sont ensuite détectées, comprises et utilisées par la communauté d’utilisateurs du catalogue.

## <a name="next-steps"></a>Étapes suivantes

Pour bien démarrer avec Azure Purview, consultez [Créer un compte Azure Purview](create-catalog-portal.md).
