---
title: Inscrire des sources de données dans Azure Data Catalog
description: Cet article traite de l’inscription des sources de données dans Azure Data Catalog, y compris des champs de métadonnées extraits lors de l’inscription.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: how-to
ms.date: 08/01/2019
ms.openlocfilehash: b8c6c6e33577fe3d49d8f0c7a94a941827f5cf68
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86523432"
---
# <a name="register-data-sources-in-azure-data-catalog"></a>Inscrire des sources de données dans Azure Data Catalog
## <a name="introduction"></a>Introduction
Azure Data Catalog est un service cloud entièrement géré qui permet d’inscrire et de détecter les sources de données d’entreprise. En d’autres termes, Data Catalog aide les utilisateurs à détecter, comprendre et utiliser les sources de données, et permet aux organisations de mieux exploiter leurs données existantes. La première étape pour qu’une source de données puisse être détectée via Data Catalog est de l’inscrire.

## <a name="register-data-sources"></a>Inscription des sources de données
Lorsque vous inscrivez une source de données, ses métadonnées sont extraites puis copiées dans le service Data Catalog. Les données restent à leur emplacement d’origine, toujours sous le contrôle des administrateurs et des stratégies du système actuel.

Pour inscrire une source de données, effectuez les étapes suivantes :
1. Dans le portail Azure Data Catalog, lancez l’outil d’inscription des sources de données Data Catalog. 
2. Connectez-vous à votre compte professionnel ou scolaire à l’aide des mêmes informations d’identification Azure Active Directory que vous utilisez pour vous connecter au portail.
3. Sélectionnez la source de données que vous souhaitez inscrire.

Pour obtenir des informations détaillées, consultez le didacticiel [Bien démarrer avec Azure Data Catalog](data-catalog-get-started.md).

Une fois que vous avez enregistré la source de données, le catalogue suit son emplacement et indexe ses métadonnées. Les utilisateurs peuvent rechercher, parcourir et détecter la source de données, ainsi qu’utiliser son emplacement pour s’y connecter à l’aide de l’application ou de l’outil de leur choix.

## <a name="supported-data-sources"></a>Sources de données prises en charge
Pour obtenir la liste des sources de données prises en charge, consultez [Sources de données prises en charge par Azure Data Catalog](data-catalog-dsr.md).

## <a name="structural-metadata"></a>Métadonnée structurelle
Lorsque vous inscrivez une source de données, l’outil d’inscription extrait des informations concernant la structure des objets sélectionnés. Ces informations sont appelées métadonnées structurelles.

Ces métadonnées structurelles incluent l’emplacement des objets. Les utilisateurs qui détectent les données peuvent donc utiliser ces informations pour se connecter à l’objet à partir des outils clients de leur choix. Parmi les autres métadonnées structurelles, on peut citer le nom et le type d’objet, le nom d’attribut/colonne et le type de données.

## <a name="descriptive-metadata"></a>Métadonnée descriptive
En plus des métadonnées structurelles de base extraites de la source de données, l’outil d’inscription extrait des métadonnées descriptives. Pour SQL Server Analysis Services et SQL Server Reporting Services, ces métadonnées sont extraites des propriétés de description exposées par ces services. Pour SQL Server, ce sont les valeurs fournies à l’aide de la propriété étendue ms\_description qui sont extraites. Pour la base de données Oracle, l’outil d’inscription des sources de données extrait les informations de la colonne Comments de la vue All\_Tab\_Comments.

En plus des métadonnées descriptives extraites de la source de données, les utilisateurs peuvent saisir des métadonnées descriptives à l’aide de l’outil d’inscription des sources de données. Les utilisateurs peuvent ajouter des balises et peuvent identifier des experts pour les objets en cours d’inscription. Toutes ces métadonnées descriptives sont copiées dans le service Data Catalog en même temps que les métadonnées structurelles.

## <a name="include-previews"></a>Inclure des aperçus
Par défaut, seules les métadonnées sont extraites des sources de données et copiées dans le service Data Catalog. Toutefois, pour mieux comprendre une source de données, il est préférable de voir un exemple des données qu’elle contient.

L’outil d’inscription des sources de données Data Catalog permet d’inclure un aperçu des données de chaque table et de chaque vue inscrites. Si vous choisissez d’inclure des aperçus lors de l’inscription, l’outil d’inscription va inclure jusqu’à 20 enregistrements de chaque table et vue. Cet aperçu est ensuite copié dans le catalogue avec les métadonnées structurelles et descriptives.

> [!NOTE]
> Il se peut que l’aperçu des tableaux larges (c’est-à-dire comportant un grand nombre de colonnes) contienne moins de 20 enregistrements.
>
>

## <a name="include-data-profiles"></a>Inclure des profils de données
Tout comme l’inclusion d’aperçus peut fournir un contexte précieux aux utilisateurs recherchant des sources de données dans Data Catalog, l’inclusion d’un profil de données peut aussi faciliter la compréhension des sources de données détectées.

L’outil d’inscription des sources de données Data Catalog permet d’inclure un profil de données pour chaque table et chaque vue inscrites. Si vous choisissez d’inclure un profil de données pendant l’inscription, l’outil d’inscription va inclure des statistiques agrégées sur les données de chaque table et vue, à savoir :

* Le nombre de lignes et la taille des données dans l’objet
* La date de la dernière mise à jour des données et du schéma de l’objet
* Le nombre d’enregistrements Null et de valeurs distinctes pour les colonnes
* Les valeurs minimale, maximale, moyenne et d’écart type pour les colonnes

Ces statistiques sont ensuite copiées dans le catalogue avec les métadonnées structurelles et descriptives.

> [!NOTE]
> Les colonnes de texte et de date n’incluent pas de statistiques de valeurs moyennes ou d’écart type dans leur profil de données.
>
>

## <a name="update-registrations"></a>Mettre à jour les inscriptions
L’inscription d’une source de données rend celle-ci détectable dans Data Catalog grâce aux métadonnées et à l’aperçu facultatif extraits lors de l’inscription. Si la source de données doit être mise à jour dans le catalogue (par exemple, si le schéma d’un objet a changé, ou si les tables initialement exclues doivent être incluses, ou si vous souhaitez mettre à jour les données incluses dans les aperçus), l’outil d’inscription des sources de données peut être réexécuté.

La réinscription d’une source de données déjà inscrite aboutit à une opération de fusion de type « upsert » : les objets existants sont actualisés, tandis que les nouveaux objets sont créés. Toutes les métadonnées fournies par les utilisateurs via le portail Data Catalog sont conservées.

## <a name="summary"></a>Résumé
L’inscription des sources de données dans Data Catalog facilite leur détection et leur compréhension, car elle consiste à copier les métadonnées descriptives et structurelles des sources de données dans le service Catalog. Une fois que vous avez enregistré la source de données, vous pouvez l’annoter, la gérer et la détecter à l’aide du portail Data Catalog.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur l’inscription des sources de données, consultez le didacticiel [Bien démarrer avec Azure Data Catalog](data-catalog-get-started.md).
