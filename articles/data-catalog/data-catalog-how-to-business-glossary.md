---
title: Configurer le glossaire métier dans Azure Data Catalog
description: Article de procédure relatif au glossaire métier d’Azure Data Catalog, qui permet de définir et d’utiliser un vocabulaire métier commun en vue de baliser les ressources de données inscrites.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: how-to
ms.date: 08/01/2019
ms.openlocfilehash: f91728435b885f3b4d9415bc81e18fdaaea0148f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021943"
---
# <a name="set-up-the-business-glossary-for-governed-tagging"></a>Configurer le glossaire métier pour un balisage géré

## <a name="introduction"></a>Introduction

Azure Data Catalog autorise la détection de sources de données pour que vous puissiez découvrir et comprendre facilement les sources de données dont vous avez besoin pour effectuer des analyses et prendre des décisions. Ces fonctionnalités se révèlent tout particulièrement utiles lorsque vous pouvez trouver et comprendre le plus large éventail de sources de données disponibles.

L’une des fonctionnalités de Data Catalog qui favorise une meilleure compréhension des données de ressources est le balisage. En utilisant le balisage, vous pouvez associer des mots clés à une ressource ou une colonne, ce qui rendra la recherche et la navigation plus efficaces. Le balisage vous permet aussi de mieux comprendre le contexte et le rôle d’une ressource.

Cependant, le balisage peut parfois engendrer des problèmes qui lui sont propres. Voici quelques exemples de problèmes potentiels liés au balisage :

* Utilisation d’abréviations pour certaines ressources, et de texte développé pour d’autres. Cette incohérence est un obstacle à la découverte de ressources, même si l’intention était de baliser les ressources avec la même balise.
* Le sens peut varier, selon le contexte. Par exemple, l’application de la balise *Chiffre d’affaires* sur un jeu de données clients pourrait désigner le chiffre d’affaires par client, alors que l’application de la même balise à un jeu de données de ventes trimestrielles pourrait désigner le chiffre d’affaires trimestriel de l’entreprise.  

Pour faire face à ce type de difficulté, Data Catalog met à disposition un glossaire métier.

Le glossaire métier de Data Catalog permet aux organisations de rassembler les principaux termes métier et leurs définitions pour créer un vocabulaire métier commun. Cette gouvernance favorise une utilisation cohérente des données à l’échelle de l’organisation. Après avoir défini un terme dans le glossaire métier, il peut être affecté à une ressource de données du catalogue. Cette approche, *le balisage géré*, est similaire au balisage classique.

## <a name="glossary-availability-and-privileges"></a>Disponibilité du glossaire et privilèges

Le glossaire métier est disponible uniquement dans l’édition Standard d’Azure Data Catalog. L’édition gratuite de Data Catalog n’inclut pas de glossaire et ne fournit aucune fonctionnalité de balisage géré.

Le glossaire métier est accessible via l’option **Glossaire** du menu de navigation du portail Data Catalog.  

![Data Catalog - Accéder au glossaire métier](./media/data-catalog-how-to-business-glossary/01-portal-menu.png)

Les administrateurs de Data Catalog et les membres du rôle Administrateurs de glossaire peuvent créer, modifier et supprimer des termes dans le glossaire métier. Tous les utilisateurs de Data Catalog peuvent consulter les définitions des termes et baliser les ressources avec ces termes du glossaire.

![Data Catalog - Ajouter un nouveau terme de glossaire](./media/data-catalog-how-to-business-glossary/02-new-term.png)

## <a name="creating-glossary-terms"></a>Création de termes de glossaire

Les administrateurs de Data Catalog et les administrateurs de glossaire peuvent créer des termes de glossaire en cliquant sur **Nouveau terme**. Chaque terme de glossaire contient les champs suivants :

* définition métier du terme ;
* description qui rend compte de l’usage prévu ou des règles métier de la ressource ou de la colonne ;
* liste des parties prenantes qui connaissent le mieux le terme ;
* terme parent qui définit la hiérarchie dont dépend le terme.

## <a name="glossary-term-hierarchies"></a>Hiérarchies de termes de glossaire

Le glossaire métier de Data Catalog permet aux organisations de construire son vocabulaire métier comme une hiérarchie de termes et de créer une classification de termes qui représente mieux la taxonomie de son entreprise.

Un terme doit être unique à un niveau donné de la hiérarchie. Les noms en double ne sont pas autorisés. Il n’existe aucune limite quant au nombre de niveaux dont est constituée une hiérarchie, mais celle-ci sera plus facile à cerner si elle n’en comporte pas plus de trois.

L’utilisation de hiérarchies dans le glossaire métier est facultative. Le fait de laisser le champ Terme parent vide pour des termes du glossaire a pour effet de créer une liste de termes plate (non hiérarchisée) dans le glossaire.  

## <a name="tagging-assets-with-glossary-terms"></a>Balisage de ressources avec des termes de glossaire

Après avoir défini des termes de glossaire dans le catalogue, le balisage de ressources est optimisé pour les recherches dans le glossaire à mesure qu’un utilisateur tape une balise. Le portail Data Catalog affiche une liste de termes de glossaire correspondants à choisir. Si l’utilisateur sélectionne un terme de glossaire dans la liste, celui-ci est ajouté à la ressource sous forme de balise (également appelée balise de glossaire). L’utilisateur peut aussi choisir de créer une balise en tapant un terme qui ne se trouve pas dans le glossaire (dans ce cas, on parle de balise utilisateur).

![Ressource de données balisée avec une balise utilisateur et deux balises de glossaire](./media/data-catalog-how-to-business-glossary/03-tagged-asset.png)

> [!NOTE]
> La balise utilisateur est le seul type de balise pris en charge dans l’édition gratuite de Data Catalog.

### <a name="hover-behavior-on-tags"></a>Comportement du pointage sur les balises

Dans le portail Data Catalog, les deux types de balise se distinguent l’un de l’autre tant sur le plan visuel que sur le plan du comportement du pointage. En pointant sur une balise utilisateur, vous pouvez voir le texte de la balise et l’identité du ou des utilisateurs ayant ajouté la balise. En pointant sur une balise de glossaire, vous obtenez aussi la définition du terme de glossaire, ainsi qu’un lien permettant d’ouvrir le glossaire métier pour y consulter la définition complète du terme.

### <a name="search-filters-for-tags"></a>Filtres de recherche pour les balises

Les balises de glossaire comme les balises utilisateur peuvent non seulement faire l’objet de recherches, mais elles peuvent aussi servir de filtres dans une recherche.

## <a name="summary"></a>Résumé

Le glossaire métier d’Azure Data Catalog et le balisage géré qu’il autorise vous permettent d’identifier, de gérer et de découvrir les ressources de données de façon cohérente. Le glossaire métier permet de promouvoir l’apprentissage du vocabulaire métier par les employés. Le glossaire prend également en charge la capture de métadonnées explicites, ce qui simplifie la compréhension et la détection de ressource.

## <a name="next-steps"></a>Étapes suivantes

* [Documentation de l’API REST pour les opérations de glossaire métier](/rest/api/datacatalog/data-catalog-glossary)
