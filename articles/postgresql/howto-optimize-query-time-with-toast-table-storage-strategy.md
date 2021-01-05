---
title: Optimiser le temps de requête en utilisant la stratégie de stockage de table TOAST dans Azure Database pour PostgreSQL - Serveur unique
description: Cet article explique comment optimiser le temps de requête à l'aide de la stratégie de stockage de table TOAST sur un serveur Azure Database pour PostgreSQL - Serveur unique.
author: dianaputnam
ms.author: dianas
ms.service: postgresql
ms.topic: how-to
ms.date: 5/6/2019
ms.openlocfilehash: 3f1fa0affb821b00d4f5529841533e854e634377
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86116180"
---
# <a name="optimize-query-time-with-the-toast-table-storage-strategy"></a>Optimiser le temps de requête à l'aide de la stratégie de stockage de table TOAST 
Cet article explique comment optimiser les temps de requête à l'aide de la stratégie de stockage de table TOAST (The Oversized-Attribute Storage Technique - technique de stockage des attributs trop grands).

## <a name="toast-table-storage-strategies"></a>Stratégies de stockage de table TOAST
Quatre stratégies différentes sont utilisées pour stocker sur disque les colonnes qui peuvent utiliser TOAST. Celles-ci représentent diverses combinaisons entre la compression et le stockage hors ligne. La stratégie peut être définie au niveau du type de données et au niveau des colonnes.
- **Normale** empêche la compression ou le stockage hors ligne. Cette stratégie désactive l'utilisation des en-têtes à un octet pour les types varlena. Normale est la seule stratégie possible pour les colonnes de types de données qui ne peuvent pas utiliser TOAST.
- **Étendue** permet la compression et le stockage hors-ligne. Étendue est la valeur par défaut pour la plupart des types de données qui peuvent utiliser TOAST. Une compression est d'abord tentée. Un stockage hors ligne est tenté si la ligne est encore trop grande.
- **Externe** permet le stockage hors ligne, mais pas la compression. Externe accélère les opérations de sous-chaîne sur les grandes colonnes de texte et d'octets. Cette vitesse s'accompagne d'une pénalité d'espace de stockage accru. Ces opérations sont optimisées pour n'extraire que les parties requises de la valeur hors ligne lorsqu'elle n'est pas compressée.
- **Principale** permet la compression, mais pas le stockage hors-ligne. Le stockage hors ligne peut encore être effectué pour ces colonnes, mais uniquement en dernier recours. Il intervient lorsqu'il n'existe aucun autre moyen de rendre la ligne suffisamment petite pour tenir sur une page.

## <a name="use-toast-table-storage-strategies"></a>Utiliser les stratégies de stockage de table TOAST
Si vos requêtes accèdent à des types de données qui peuvent utiliser TOAST, pensez à utiliser la stratégie Principale au lieu de l'option par défaut Étendue afin de réduire les temps de requête. La stratégie Principale n'exclut pas le stockage hors ligne. Si vos requêtes n'accèdent pas à des types de données qui peuvent utiliser TOAST, nous vous recommandons de conserver la stratégie Étendue. Une plus grande partie des lignes de la table principale tiendra dans le cache partagé des tampons, améliorant ainsi les performances.

Si l'une de vos charges de travail utilise un schéma contenant de larges tableaux et un grand nombre de caractères, nous vous recommandons d'utiliser des tables TOAST PostgreSQL. Prenons un exemple de table client contenant plus de 350 colonnes dont plusieurs couvrent 255 caractères. Après sa conversion à la stratégie Principale de table TOAST, son temps de requête de référence est passé de 4 203 secondes à 467 secondes. Ce qui représente une amélioration de 89 %.

## <a name="next-steps"></a>Étapes suivantes
Examinez les caractéristiques précédentes de votre charge de travail. 

Passez en revue la documentation PostgreSQL suivante : 
- [Chapitre 68, Stockage physique de base de données](https://www.postgresql.org/docs/current/storage-toast.html) 