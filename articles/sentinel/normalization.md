---
title: Normalisation des données dans Azure Sentinel | Microsoft Docs
description: Cet article explique comment Azure Sentinel normalise les données provenant de sources différentes et détaille le schéma de normalisation.
services: sentinel
cloud: na
documentationcenter: na
author: yelevin
manager: rkarlin
ms.assetid: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: yelevin
ms.openlocfilehash: a9d2cd48e3b686614f7361d2007f6f8183c2361e
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94657021"
---
# <a name="normalization-in-azure-sentinel"></a>Normalisation dans Azure Sentinel

Cet article explique la normalisation des schémas de données dans Azure Sentinel, et en particulier le schéma des connexions et sessions réseau, vers lequel un lien est inclus.

## <a name="what-is-normalization"></a>Qu’est-ce que la normalisation

L’utilisation conjointe de différents types de données et de tables présente des défis. Vous devez vous familiariser avec un grand nombre de types de données et de schémas, en devant écrire et utiliser un ensemble unique de règles d’analyse, de classeurs et de requêtes de chasse pour chacun d’entre eux, même pour ceux qui partagent des points communs (par exemple, concernant les périphériques de pare-feu). La corrélation entre les différents types de données, nécessaire à l’investigation et la chasse, est également difficile. Azure Sentinel offre une expérience transparente pour le traitement des données de diverses sources dans des affichages uniformes et normalisés.

Le modèle Azure Sentinel **Common Information Model** se compose de trois aspects :

- **Les schémas normalisés** couvrent les ensembles communs de types d’événements prévisibles (tables), faciles à utiliser et pour créer des fonctionnalités unifiées (par exemple, une table de mise en réseau). Le schéma comprend également une convention sur les colonnes normalisées, ainsi que des définitions pour la normalisation des valeurs et des formats (représentation cohérente standard des données telles que les adresses IP).

- **Les analyseurs** mappent les données existantes de différents types au schéma normalisé. D’après le modèle, les données peuvent être analysées dans le schéma normalisé au moment de la requête (à l’aide de fonctions) ou au moment de l’ingestion. Pour l’instant, seule l’analyse au moment de la requête est prise en charge.

- **Le contenu de chaque de schéma normalisé** comprend des règles d’analyse, des classeurs interactifs, des requêtes de chasse et du contenu supplémentaire.

### <a name="current-release"></a>Version actuelle

Avec cette version, le [schéma de sessions et de connexions réseau normalisées](./normalization-schema.md) (v 1.0.0) est disponible en préversion publique. Ce schéma décrit les connexions réseau ou les sessions telles que celles journalisées par les pare-feu, les données de transmission, le groupe de sécurité réseau, le réseau de distribution de contenu, les systèmes proxy et les passerelles de sécurité web.  Les analyseurs de requêtes et les règles d’analyse sélectionnés sont disponibles avec le schéma et en font usage.

Le schéma est actuellement disponible uniquement via les analyseurs au moment de la requête (voir la section parseurs).

Vous pouvez analyser les données dans d’autres représentations et utiliser le [modèle d’affectation de noms des entités OSSEM](https://ossemproject.com/cdm/entities/intro.html#) pour créer des colonnes qui seront cohérentes avec les tables normalisées, existantes ou futures.

## <a name="normalized-schema-and-parsing"></a>Schéma normalisé et analyse

### <a name="how-our-normalized-schemas-are-defined"></a>Comment nos schémas normalisés sont définis

Azure Sentinel est aligné sur le modèle CIM de [métadonnées des événements de sécurité Open source (OSSEM)](https://ossemproject.com/intro.html), ce qui permet d’établir une corrélation entre les entités prévisibles et les tables normalisées. OSSEM est un projet de la communauté qui se concentre principalement sur la documentation et la normalisation des journaux d’événements de sécurité provenant de diverses sources de données et de divers systèmes d’exploitation. En outre, ce projet fournit un modèle CIM (Common Information Model) qui peut être utilisé par les ingénieurs de données lors des procédures de normalisation des données pour permettre aux analystes de sécurité d’interroger et d’analyser les données dans diverses sources de données.

Le modèle [CIM OSSEM](https://ossemproject.com/cdm/intro.html) définit un ensemble d’entités (par exemple : fichier, hôte, IP, processus) et définit un ensemble d’attributs pour chaque entité de ce type. En outre, le CIM définit un ensemble de tables (par exemple, la table de [session réseau](https://ossemproject.com/cdm/tables/network_session.html)) qui utilisent les attributs pertinents de ces entités, ce qui permet une corrélation transparente et prévisible. Notez que les entités peuvent être imbriquées (par exemple, l’entité source peut contenir une entité de fichier qui aura un attribut de nom).

Pour en savoir plus sur la structure d’entité OSSEM, consultez la [référence officielle OSSEM](https://ossemproject.com/cdm/guidelines/entity_structure.html).

### <a name="how-the-normalized-schemas-are-implemented-in-azure-sentinel"></a>Comment les schémas normalisés sont implémentés dans Azure Sentinel

Dans l’implémentation du CIM OSSEM dans Azure Sentinel, nous allons projeter la représentation OSSEM sur une représentation tabulaire Log Analytics, que cette représentation soit une table intégrée ou qu’elle ait été créée à l’aide d’analyseurs de requêtes ou de fonctions qui mappent les données existantes sur cette représentation. Pour la représentation tabulaire, nous concaténons les noms d’entité OSSEM et les noms d’attributs, et nous les mappons collectivement sur un nom de colonne unique. Par exemple, une entité source contenant une entité de fichier contenant une entité de hachage contenant un attribut md5 est implémentée comme colonne Log Analytics suivante : SrcFileHashMd5. (OSSEM utilise *snake_case* par défaut, tandis qu’Azure Sentinel et Log Analytics utilisent *PascalCase*. Dans OSSEM, cette colonne serait src_file_hash_md5.)

Des champs personnalisés supplémentaires peuvent exister dans l’implémentation d’Azure Sentinel, en raison des exigences de la plateforme Log Analytics et des cas d’utilisation spécifiques aux clients Azure Sentinel.

### <a name="schema-reference"></a>Informations de référence sur les schémas

Pour en savoir plus, consultez le [document de référence sur le schéma](./normalization-schema.md), ainsi que la [documentation du projet OSSEM](https://ossemproject.com/cdm/intro.html) officielle.

La référence de schéma comprend également la normalisation de la valeur et du format. Les champs sources, originaux ou analysés, peuvent ne pas être dans un format standard ou utiliser la liste de valeurs standard du schéma, et doivent donc être convertis en représentation standard du schéma afin d’être entièrement normalisés.

## <a name="parsers"></a>Analyseurs

### <a name="what-is-parsing"></a>Qu’est-ce que l’analyse ?

Avec un ensemble de base de tables normalisées définies, vous devrez transformer (analyser/mapper) vos données dans ces tables. Autrement dit, vous allez extraire des données et les faire passer de leur forme brute vers des colonnes connues dans le schéma normalisé. L’analyse dans Azure Sentinel se produit au **moment de la requête**. Les parseurs sont créés en tant que fonction utilisateurs Log Analytics (à l’aide du langage de requête Kusto-KQL) qui transforment les données dans les tables existantes (telles que CommonSecurityLog, les tables de journaux personnalisées, syslog) dans le schéma des tables normalisées.

L’autre type d’analyse, qui n’est pas encore prise en charge dans Azure Sentinel, est au **moment de l’ingestion**, ce qui permet de collecter des données directement dans la ou les tables normalisées lorsqu’elles sont ingérées à partir de ses sources de données. L’analyse au moment de l’ingestion améliore les performances, car le modèle de données est interrogé directement, sans qu’il soit nécessaire d’utiliser des fonctions.

### <a name="using-query-time-parsers"></a>Utilisation des analyseurs au moment de la requête

#### <a name="installing-a-parser"></a>Installation d’un analyseur

Les analyseurs disponibles au moment de la requête sont disponibles dans le [référentiel GitHub officiel](https://github.com/Azure/Azure-Sentinel/tree/master/Parsers/Normalized%20Schema%20-%20Networking%20(v1.0.0)) d’Azure Sentinel. Chaque analyseur est associé à une version pour permettre aux clients d’utiliser et de surveiller facilement les futures mises à jour. Pour installer un analyseur :

1. Copiez le contenu de l’analyseur approprié à partir du fichier KQL approprié dans le lien GitHub ci-dessus

1. Dans le portail Azure Sentinel, ouvrez la page des journaux, collez le contenu du fichier KQL dans l’écran des journaux, puis cliquez sur **Enregistrer**.

    :::image type="content" source="./media/normalization/install-new-parser.png" alt-text="Installer un nouvel analyseur":::

1. Dans la boîte de dialogue Enregistrer, renseignez les champs comme suit :
    1. **Name** : Il est recommandé d’utiliser la même valeur que celle utilisée dans le champ **Function Alias** (dans l’exemple ci-dessus, *CheckPoint_Network_NormalizedParser*)
    
    1. **Enregistrer sous** : sélectionner **Function**

    1. **Alias de fonction** : doit être nommé dans la Convention d’affectation de noms suivante : *PRODUCT_Network_NormalizedParser* (dans l’exemple ci-dessus, *CheckPoint_Network_NormalizedParser*).

    1. **Catégorie** : vous pouvez sélectionner une catégorie existante ou créer une catégorie (par exemple, *NormalizedNetworkSessionsParsers*).
    
        :::image type="content" source="./media/normalization/save-new-parser.png" alt-text="Enregistrer l’analyseur":::

Pour utiliser correctement les analyseurs, vous devez également installer l’analyseur de schéma réseau vide (qui crée une vue tabulaire vide de tous les champs du schéma de sessions réseau) et l’analyseur de métadonnées de réseau (qui associe tous les analyseurs activés pour créer une vue unique des données à partir de différentes sources dans le schéma de mise en réseau). L’installation de ces deux analyseurs s’effectue de la même façon que pour les étapes mentionnées ci-dessus.

Lors de l’enregistrement d’une fonction de requête, il peut être nécessaire de fermer l’explorateur de requêtes et de le rouvrir pour que la nouvelle fonction soit prise en compte.

#### <a name="using-the-parsers"></a>Utilisation des analyseurs

Une fois activé, vous pouvez utiliser l’analyseur de métadonnées pour interroger une vue unifiée sur tous les analyseurs actuellement activés. Pour ce faire, accédez à la page des journaux Sentinel et interrogez l’analyseur de métadonnées :

:::image type="content" source="./media/normalization/query-parser.png" alt-text="Interroger l’analyseur":::
 
Vous pouvez également accéder à l’analyseur de métadonnées ou à des analyseurs individuels à l’aide de l’explorateur de requêtes de la page des journaux Sentinel, en cliquant sur Explorateur de requêtes :

:::image type="content" source="./media/normalization/query-explorer.png" alt-text="Explorateur de requêtes":::

Dans le volet de droite, développez la section « Requêtes enregistrées » et recherchez le dossier « NormalizedNetworkParsers » (ou le nom de la catégorie que vous avez choisi lors de la création des analyseurs) :

:::image type="content" source="./media/normalization/find-parser.png" alt-text="Rechercher votre analyseur":::

Vous pouvez cliquer sur chaque analyseur et voir la fonction sous-jacente qu’il utilise, puis l’exécuter (ou y accéder directement par son alias, comme décrit ci-dessus). Notez que certains analyseurs peuvent conserver les champs d’origine en parallèle des champs normalisés pour des raisons pratiques. Ceci peut être facilement modifié dans la requête de l’analyseur.

#### <a name="customizing-parsers"></a>Personnalisation des analyseurs

Vous pouvez répéter les étapes ci-dessus (recherche de l’analyseur dans l’explorateur de requêtes), cliquer sur l’analyseur approprié et voir son implémentation de la fonction.
Par exemple, vous pouvez décider de modifier l’analyseur de métadonnées pour ajouter ou supprimer des analyseurs.

:::image type="content" source="./media/normalization/customize-parser.png" alt-text="Personnalisation de votre analyseur":::
 
Une fois la fonction modifiée, cliquez à nouveau sur Enregistrer et utilisez le même nom, le même alias et la même catégorie. Une boîte de dialogue de remplacement s’ouvre. Appuyez sur « OK » :

:::image type="content" source="./media/normalization/are-you-sure.png" alt-text="Confirmez-vous cette action ?":::

#### <a name="additional-information"></a>Informations supplémentaires

En savoir plus sur les [requêtes enregistrées](../azure-monitor/log-query/example-queries.md) (implémentation de l’analyseur de temps de requête) dans Log Analytics.


## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez découvert le schéma de normalisation d’Azure Sentinel. Pour le schéma de référence lui-même, consultez [informations de référence sur le schéma de normalisation des données Azure Sentinel](./normalization-schema.md).

* [Blog Azure Sentinel](https://aka.ms/azuresentinelblog) : accédez à des billets de blog sur la sécurité et la conformité Azure.