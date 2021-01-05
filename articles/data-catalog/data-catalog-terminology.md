---
title: Terminologie d’Azure Data Catalog
description: Cet article présente les concepts et les termes utilisés dans la documentation d’Azure Data Catalog.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: conceptual
ms.date: 08/01/2019
ms.openlocfilehash: 6e636a039302d2ea61c7135bff1fceb4763f25df
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92151455"
---
# <a name="azure-data-catalog-terminology"></a>Terminologie d’Azure Data Catalog

Cet article présente les concepts et les termes utilisés dans la documentation d’Azure Data Catalog.

## <a name="catalog"></a>Catalogue

Azure Data Catalog est un référentiel de métadonnées de cloud dans lequel des données sources et des ressources de données peuvent être inscrites. Le catalogue sert d’emplacement de stockage central pour les métadonnées structurelles, extraites de sources de données, et les métadonnées descriptives ajoutées par les utilisateurs.

## <a name="data-source"></a>Source de données

Une source de données est un système ou un conteneur qui gère des ressources de données. Exemples : les bases de données SQL Server, les bases de données Oracle, les bases de données SQL Server Analysis Services (tabulaires ou multidimensionnelles) et les serveurs SQL Server Reporting Services.

## <a name="data-asset"></a>Ressource de données

Les ressources de données sont des objets contenus dans les sources de données qui peuvent être inscrits dans le catalogue. Exemples : les tables et les vues SQL Server, les tables et les vues Oracle, les mesures SQL Server Analysis Services, les dimensions et les indicateurs clés de performance, et les rapports SQL Server Reporting Services.

## <a name="data-asset-location"></a>Emplacement des ressources de données

Le catalogue stocke l'emplacement d'une source de données ou d'une ressource de données, qui peut être utilisé pour se connecter à la source à l'aide d'une application cliente. Le format et les détails de l'emplacement varient en fonction du type de source de données. Une table SQL Server peut être, par exemple, identifiée par quatre éléments : le nom du serveur, le nom de la base de données, le nom du schéma et le nom de l'objet, alors qu’un rapport SQL Server Reporting Services peut être identifié par son URL.

## <a name="structural-metadata"></a>Métadonnée structurelle

Les métadonnées structurelles sont des métadonnées extraites d'une source de données qui décrit la structure d'une ressource de données. Cela inclut l'emplacement des ressources, le nom et le type de l'objet, et les caractéristiques supplémentaires spécifiques au type. Les métadonnées structurelles des tables et des vues incluent, par exemple, les noms et les types de données des colonnes de l'objet.

## <a name="descriptive-metadata"></a>Métadonnée descriptive

Les métadonnées descriptives sont des métadonnées qui décrivent l'objectif d'une ressource de données. En règle générale, les métadonnées descriptives sont ajoutées par les utilisateurs du catalogue à l'aide du portail Azure Data Catalog, mais elles peuvent être également extraites de la source de données pendant l'inscription. Par exemple, l’outil de référencement pour Azure Data Catalog extrait les descriptions de la propriété Description dans SQL Server Analysis Services et SQL Server Reporting Services, et de la [propriété étendue ms_description](/previous-versions/sql/sql-server-2008-r2/ms190243(v=sql.105)) dans les bases de données SQL Server, si ces propriétés ont été remplies avec des valeurs.

## <a name="request-access"></a>Demander l'accès

Des métadonnées descriptives d'une ressource de données peuvent inclure des informations sur la façon de demander l'accès à la ressource de données ou à la source de données. Ces informations sont présentées à l'emplacement des ressources de données et peuvent inclure une ou plusieurs des options suivantes :

* L'adresse électronique de l'utilisateur ou de l'équipe responsable de l'octroi de l’accès à la source de données.
* L'URL du processus documenté que les utilisateurs doivent suivre pour accéder à la source de données.
* L'URL d'un outil de gestion des identités et des accès (comme Microsoft Identity Manager) qui peut être utilisé pour accéder à la source de données.
* Une entrée de texte libre qui décrit la façon dont les utilisateurs peuvent accéder à la source de données.

## <a name="preview"></a>PRÉVERSION

Une version préliminaire d’Azure Data Catalog est un instantané de maximum 20 enregistrements qui peuvent être extraits de la source de données pendant l'inscription et stockés dans le catalogue avec les métadonnées de ressources de données. La version préliminaire peut permettre aux utilisateurs, qui découvrent une ressource de données, de mieux comprendre son fonctionnement et son rôle. En d'autres termes, les exemples de données peuvent être plus utiles que les noms de colonnes et les types de données.
Les versions préliminaires sont uniquement prises en charge par les tables et les vues et doivent être explicitement sélectionnées par l'utilisateur pendant l'inscription.

## <a name="data-profile"></a>Profil de données

Un profil de données d’Azure Data Catalog est un instantané de métadonnées de niveau colonne et table sur des ressources de données inscrites, qui peuvent être extraites de la source de données pendant l’inscription et stockées dans le catalogue avec les métadonnées de ressources de données. Le profil de données peut permettre aux utilisateurs qui découvrent une ressource de données de mieux comprendre son fonctionnement et son rôle. Comme pour les versions préliminaires, les profils de données doivent être explicitement sélectionnés par l’utilisateur pendant l’inscription.

> [!NOTE]
> L’extraction d’un profil de données peut être une opération coûteuse pour les grandes tables et vues, et peut augmenter considérablement le temps nécessaire pour inscrire une source de données.


## <a name="user-perspective"></a>Point de vue de l’utilisateur

Dans Azure Data Catalog, tout utilisateur peut fournir des métadonnées descriptives pour une ressource de données inscrite. Chaque utilisateur dispose d'un point de vue différent sur les données et leur utilisation. L'administrateur responsable d'un serveur peut, par exemple, fournir les détails de son contrat de niveau de service (SLA) ou de sa sauvegarde Windows ; un administrateur de données peut fournir des liens vers la documentation des processus d’entreprise que les données prennent en charge ; et un analyste peut fournir aux autres analystes une description des informations les plus pertinentes, qui peuvent être utiles aux utilisateurs qui ont besoin de découvrir et de comprendre les données.

Chacun de ces points de vue possède une valeur intrinsèque. Grâce à Azure Data Catalog, chaque utilisateur peut fournir les informations qui les plus pertinentes pour lui, tandis que tous les utilisateurs peuvent utiliser ces informations pour comprendre les données et leur objectif.

## <a name="expert"></a>Expert

Un expert est un utilisateur qui a été identifié comme une personne pouvant apporter son point de vue d’« expert » pour une ressource de données. Tout utilisateur peut s’ajouter lui-même ou ajouter un autre utilisateur en tant qu'expert pour une ressource. Le statut d’expert ne permet pas aux utilisateurs de disposer de privilèges supplémentaires dans Azure Data Catalog ; mais il leur permet de localiser facilement ces points de vue qui peuvent être utiles lorsqu’ils examinent les métadonnées descriptives d'une ressource.

## <a name="owner"></a>Propriétaire

Un propriétaire est un utilisateur qui dispose de privilèges supplémentaires pour assurer la gestion d'une ressource de données d’Azure Data Catalog. Les utilisateurs peuvent s’approprier des ressources de données inscrites et les propriétaires peuvent ajouter d'autres utilisateurs en tant que copropriétaires. Pour plus d'informations, consultez [Gestion des ressources de données](data-catalog-how-to-manage.md)  

> [!NOTE]
> La propriété et la gestion sont uniquement disponibles dans l'édition Standard d’Azure Data Catalog.

## <a name="registration"></a>Inscription

L'inscription représente le fait d’extraire des métadonnées de ressources de données à partir d'une source de données et de les copier dans le service Azure Data Catalog. Les ressources de données qui ont été inscrites peuvent ensuite être annotées et découvertes.

## <a name="next-steps"></a>Étapes suivantes

[Démarrage rapide : Créer un catalogue Azure Data Catalog](data-catalog-get-started.md)