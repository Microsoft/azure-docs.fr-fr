---
title: Comment documenter les sources de données dans Azure Data Catalog
description: Article de procédure expliquant comment documenter les ressources de données dans Azure Data Catalog.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: how-to
ms.date: 08/01/2019
ms.openlocfilehash: 3674c316e34b1e54ed8282ac7d2c228a2b774c06
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019019"
---
# <a name="how-to-document-data-sources-in-azure-data-catalog"></a>Comment documenter les sources de données dans Azure Data Catalog

## <a name="introduction"></a>Introduction
**Microsoft Azure Data Catalog** est un service cloud entièrement géré qui sert de système d'inscription et de détection des sources de données d'entreprise. En d’autres termes, **Microsoft Azure Data Catalog** vise essentiellement à aider les utilisateurs à détecter, *comprendre* et utiliser des sources de données, et permet aux organisations de mieux exploiter leurs données.

Lorsqu’une source de données est inscrite dans **Azure Data Catalog**, ses métadonnées sont copiées et indexées par le service. Mais ce n’est pas tout. **Azure Data Catalog** permet également aux utilisateurs de fournir leur propre documentation complète pour décrire l’utilisation et les scénarios courants de la source de données.

L’article [Annotation de sources de données](data-catalog-how-to-annotate.md)explique comment des experts qui connaissent la source de données peuvent l’annoter avec des balises et une description. Le portail **Azure Data Catalog** inclut un éditeur de texte qui permet aux utilisateurs de documenter entièrement les ressources de données et les conteneurs. L’éditeur inclut une mise en forme des paragraphes, avec notamment des en-têtes, la mise en forme du texte, des listes à puces, des listes numérotées et des tableaux.

Les balises et descriptions sont idéales pour ajouter des annotations simples. Toutefois, pour aider les consommateurs de données à mieux comprendre l’utilisation d’une source de données et les scénarios commerciaux d’une source de données, un expert peut fournir une documentation complète et détaillée. Il est facile de documenter une source de données. Il suffit de sélectionner une ressource de données ou un conteneur et de choisir l’option **Documentation**.

![Onglet Documentation dans un catalogue de données](media/data-catalog-documentation/data-catalog-documentation.png)

## <a name="documenting-data-assets"></a>Documentation de ressources de données
La documentation **Azure Data Catalog** vous permet d’utiliser votre catalogue de données comme référentiel de contenu pour créer une narration complète de vos ressources de données. Vous pouvez explorer le contenu détaillé décrivant les conteneurs et tables. Si vous utilisez déjà un autre référentiel de contenu, tel que SharePoint ou un partage de fichiers, vous pouvez ajouter des liens de documentation à votre ressource pour référencer ce contenu existant. Vos documents existants seront ainsi mieux exposés.

> [!NOTE]
> La documentation n’est pas incluse dans l’index de recherche.
>

![Onglet Documentation et lien hypertexte vers lien web](media/data-catalog-documentation/data-catalog-documentation2.png)

Le niveau de documentation peut aller d’une simple description des caractéristiques et de la valeur d’un conteneur de ressources de données à une description détaillée du schéma de table dans un conteneur. Le niveau de documentation fourni doit être dicté par vos besoins métiers. De façon générale, voici cependant quelques avantages et inconvénients associés à la documentation de ressources de données :

* Documenter uniquement un conteneur : tout le contenu se trouve dans un même emplacement, mais ne comporte sans doute pas les informations nécessaires pour permettre aux utilisateurs de prendre une décision informée.
* Documenter uniquement les tables : le contenu se rapporte à l’objet en question, mais les documents sont disséminés à plusieurs emplacements.
* Documenter les conteneurs et les tables : l’approche la plus complète, qui peut toutefois impliquer un effort supplémentaire pour la gestion des documents.

## <a name="summary"></a>Résumé
La documentation des sources de données avec **Azure Data Catalog** peut créer une narration sur vos ressources de données avec le degré de détail dont vous avez besoin.  À l’aide de liens, vous pouvez référencer du contenu stocké dans un référentiel de contenu existant qui rassemble vos documents et vos ressources de données existants. Une fois que vos utilisateurs accèdent aux ressources de données appropriées, ils peuvent bénéficier d’un ensemble de documentation complet.
