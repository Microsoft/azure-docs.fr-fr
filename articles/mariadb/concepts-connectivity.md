---
title: Erreurs de connectivité temporaires - Azure Database for MariaDB
description: Découvrez comment gérer les erreurs de connectivité temporaires pour la base de données Azure Database pour MariaDB.
keywords: connexion mysql,chaîne de connexion,problèmes de connectivité,erreur temporaire,erreur de connexion
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: 2a651d87411654f1a52c4a097f115ba848ce569c
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541672"
---
# <a name="handling-of-transient-connectivity-errors-for-azure-database-for-mariadb"></a>Gestion des erreurs de connectivité temporaires pour la base de données Azure Database pour MariaDB

Cet article explique comment gérer les erreurs de connectivité temporaires pour la base de données Azure Database for MariaDB.

## <a name="transient-errors"></a>Erreurs temporaires

Une erreur temporaire, aussi connue sous le nom de défaut temporaire, est une erreur qui se résout d’elle-même. Le plus souvent, ces erreurs se manifestent sous forme de connexion au serveur de base de données en cours de suppression. De même, les nouvelles connexions à un serveur ne peuvent pas être ouvertes. Des erreurs temporaires peuvent se produire par exemple en cas de défaillance matérielle ou réseau. Elles peuvent également être dues au déploiement d'une nouvelle version de service PaaS. La plupart de ces événements sont automatiquement corrigés par le système en moins de 60 secondes. En matière de conception et de développement d'applications dans le cloud, il convient de s'attendre à des erreurs temporaires. Partez du principe qu’elles peuvent se produire dans n’importe quel composant à n’importe quel moment et qu'il existe une logique appropriée pour faire face à ces situations.

## <a name="handling-transient-errors"></a>Gestion des erreurs temporaires

Les erreurs temporaires doivent être gérées à l’aide de la logique de nouvelle tentative. Situations qui doivent être prises en compte :

* Une erreur se produit lorsque vous essayez d’ouvrir une connexion
* Une connexion inactive est supprimée côté serveur. Lorsque vous essayez d’émettre une commande, elle ne peut pas s’exécuter
* Une connexion active qui exécute actuellement une commande est supprimée.

Le premier et le deuxième cas sont assez simples à gérer. Essayez d’ouvrir à nouveau la connexion. Lorsque vous réussissez, l’erreur temporaire a été atténuée par le système. Vous pouvez utiliser de nouveau votre base de données Azure Database for MariaDB. Nous vous recommandons d’attendre avant de réessayer la connexion. Abandonnez si les tentatives initiales échouent. Ainsi, le système peut utiliser toutes les ressources disponibles pour résoudre la situation d’erreur. Nous vous recommandons de procéder comme suit :

* Patientez 5 secondes avant votre première nouvelle tentative.
* Pour chaque nouvelle tentative suivante, augmentez l’attente de façon exponentielle, jusqu’à 60 secondes.
* Définissez un nombre maximal de tentatives au-delà duquel votre application considère que l’opération a échoué.

Lorsqu’une connexion échoue avec une transaction active, il est plus difficile de gérer correctement la récupération. Il existe deux cas : Si la transaction était en lecture seule par nature, vous pouvez rouvrir la connexion et relancer la transaction. Toutefois, si la transaction a également été écrite dans la base de données, vous devez déterminer si la transaction a été annulée ou si elle a réussi avant que l’erreur temporaire se produise. Dans ce cas, vous n’avez peut-être pas reçu l’accusé de réception de validation du serveur de base de données.

Une façon d’y remédier est de générer un identifiant unique sur le client qui est utilisé pour toutes les tentatives. Vous transmettez cet ID unique dans le cadre de la transaction au serveur et vous le stockez dans une colonne avec une contrainte unique. Ainsi, vous pouvez retenter la transaction en toute sécurité. Elle aboutit si la transaction précédente a été restaurée et que l’ID unique généré par le client n’existe pas encore dans le système. Elle échoue et indique une violation de clé en double si l’ID unique a été précédemment stocké, car la transaction précédente s’est correctement terminée.

Si votre programme communique avec Azure Database pour MariaDB par le biais d’un intergiciel (middleware) tiers, demandez au fournisseur s’il contient une logique de nouvelle tentative pour les erreurs temporaires.

Veillez à tester votre logique de nouvelle tentative. Par exemple, essayez d’exécuter votre code tout en augmentant ou en diminuant les ressources de calcul de votre serveur Azure Database for MariaDB. Votre application doit être en mesure de gérer la courte interruption qui survient au cours de cette opération sans le moindre problème.

## <a name="next-steps"></a>Étapes suivantes

* [Résoudre les problèmes de connexion à la base de données Azure Database for MariaDB](howto-troubleshoot-common-connection-issues.md)
