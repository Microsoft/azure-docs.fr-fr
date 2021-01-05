---
title: Déterminer le type d’application – Hyperscale (Citus) – Azure Database pour PostgreSQL
description: Identifier votre application pour une modélisation des données distribuée efficace
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 07/17/2020
ms.openlocfilehash: 92333857177d33307d6997bfcbdf79787d3ab127
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90895955"
---
# <a name="determining-application-type"></a>Détermination du type d’application

Pour exécuter des requêtes efficaces sur un groupe de serveurs Hyperscale (Citus), les tables doivent être correctement réparties entre les serveurs. La distribution recommandée varie selon le type d’application et ses modèles de requête.

Il existe grosso modo deux types d’applications qui fonctionnent bien sur Hyperscale (Citus). La première étape de la modélisation des données consiste à identifier celle qui ressemble le plus à votre application.

## <a name="at-a-glance"></a>Aperçu

| Applications multilocataires                                 | Applications en temps réel                                |
|-----------------------------------------------------------|-------------------------------------------------------|
| Parfois des dizaines ou des centaines de tables dans le schéma          | Petit nombre de tables                                |
| Requêtes relatives à un locataire (société/magasin) à la fois | Requêtes analytiques relativement simples avec agrégations |
| Charges de travail OLTP pour le traitement des clients Web                    | Volume d’ingestion élevé de données essentiellement immuables           |
| Charges de travail OLAP qui servent des requêtes analytiques par locataire   | Centrage fréquent sur les grandes tables d’événements            |

## <a name="examples-and-characteristics"></a>Exemples et caractéristiques

**Application multilocataire**

> Il s’agit généralement d’applications SaaS qui servent d’autres sociétés, comptes ou organisations. La plupart des applications SaaS sont relationnelles par nature. Elles ont une dimension naturelle sur laquelle les données peuvent être distribuées sur les nœuds : partitionnez simplement par tenant\_id.
>
> Hyperscale (Citus) vous permet de mettre à l’échelle votre base de données vers des millions de locataires sans avoir à recréer l’architecture de votre application. Vous pouvez conserver la sémantique relationnelle dont vous avez besoin, comme les jointures, les contraintes de clé étrangère, les transactions, ACID et la cohérence.
>
> -   **Exemples :** Sites web qui hébergent des vitrines pour d’autres entreprises, par exemple une solution de marketing numérique ou un outil d’automatisation des ventes.
> -   **Caractéristiques** : Requêtes relatives à un locataire unique plutôt que joignant des informations entre locataires. Cela comprend les charges de travail OLTP pour servir les clients web et les charges de travail OLAP qui servent des requêtes analytiques par locataire. La présence de dizaines ou de centaines de tables dans votre schéma de la base de données constitue également un indicateur pour le modèle de données mutualisées.
>
> La mise à l’échelle d’une application mutualisée avec Hyperscale (Citus) nécessite également des modifications minimes du code de l’application. Nous prenons en charge les infrastructures populaires telles que Ruby on Rails et Django.

**Analytique en temps réel**

> Applications nécessitant un parallélisme massif, coordonnant des centaines de cœurs pour obtenir des résultats rapides aux requêtes numériques, statistiques ou de comptage.  Grâce à partitionnement et à la parallélisation des requêtes SQL sur plusieurs nœuds, Hyperscale (Citus) permet d’effectuer des requêtes en temps réel sur des milliards d’enregistrements en moins d’une seconde.
>
> Les tables des modèles de données d’analyse en temps réel sont généralement distribuées par des colonnes telles que user\_id, host\_id ou device\_id.
>
> -   **Exemples :** Tableaux de bord d’analyse destinés aux clients nécessitant des temps de réponse inférieurs à une seconde.
> -   **Caractéristiques** : Peu de tables, souvent axées sur un grand tableau d’événements de périphérique, de site ou d’utilisateur, et nécessitant un volume élevé d’ingestion de données essentiellement immuables. Requêtes analytiques relativement simples (mais gourmandes en calculs) impliquant plusieurs agrégations et clauses GROUP BY.

Si votre situation ressemble à l’une ou l’autre des cas ci-dessus, l’étape suivante consiste à décider comment partitionner vos données dans le groupe de serveurs. Le choix de l\'administrateur de base de données en ce qui concerne les colonnes de distribution doit correspondre aux modèles d’accès des requêtes typiques pour garantir les performances.

## <a name="next-steps"></a>Étapes suivantes

* [Choisir une colonne de distribution](concepts-hyperscale-choose-distribution-column.md) pour les tables de votre application afin de distribuer efficacement des données
