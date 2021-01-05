---
title: Déplacer des régions Azure - Portail Azure - Azure Database pour PostgreSQL - Serveur unique
description: Déplacer un serveur Azure Database pour PostgreSQL d’une région Azure vers une autre à l’aide d’un réplica en lecture et du Portail Azure.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 06/29/2020
ms.openlocfilehash: d237d5709f8d2bb47de3e89b0b7103b195376e11
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92489742"
---
# <a name="move-an-azure-database-for-azure-database-for-postgresql---single-server-to-another-region-by-using-the-azure-portal"></a>Déplacer une base de données Azure pour Azure Database pour PostgreSQL - Serveur unique vers une autre région à l’aide du portail Azure Database pour PostgreSQL

Il existe plusieurs scénarios pour déplacer un serveur Azure Database pour PostgreSQL d’une région vers une autre. Par exemple, vous pourriez vouloir déplacer un serveur de production vers une autre région dans le cadre de votre planification de la récupération d'urgence.

Vous pouvez utiliser un [réplica en lecture interrégional](concepts-read-replicas.md#cross-region-replication)Azure Database pour PostgreSQL pour effectuer le déplacement vers une autre région. Pour ce faire, créez tout d’abord un réplica en lecture dans la région cible. Ensuite, arrêtez la réplication vers le serveur de réplica en lecture pour en faire un serveur autonome qui accepte à la fois le trafic en lecture et en écriture. 

> [!NOTE]
> Cet article se concentre sur le déplacement de votre serveur vers une région différente. Si vous souhaitez déplacer votre serveur vers un groupe de ressources ou un abonnement différent, référez vous à l’article de [déplacement](../azure-resource-manager/management/move-resource-group-and-subscription.md). 

## <a name="prerequisites"></a>Prérequis

- La fonctionnalité de réplica en lecture entre région est disponible uniquement pour les serveurs Azure Database pour PostgreSQL - Serveur Unique dans les niveaux tarifaires Usage général ou Mémoire optimisée. Vérifiez que le serveur source se trouve dans l’un de ces niveaux tarifaires.

- Assurez-vous que votre serveur source Azure Database pour PostgreSQL est dans la région Azure de laquelle vous souhaitez vous déplacer.

## <a name="prepare-to-move"></a>Préparer le déplacement

Pour préparer le serveur source pour la réplication à l’aide du portail Azure, procédez comme suit : 

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Sélectionnez le serveur Azure Database pour PostgreSQL existant que vous souhaitez utiliser comme serveur source. Cette action ouvre la page **Vue d’ensemble** .
1. Sélectionnez **Réplication** depuis le menu du serveur. Si un support de réplication Azure est réglé au moins sur **Réplica** , vous pouvez créer des réplicas en lecture. 
1. Si le support de réplication Azure n’est pas réglé sur au moins **Réplica** , réglez-le. Sélectionnez **Enregistrer** .
1. Redémarrez le serveur pour appliquer le changement en sélectionnant **Oui** .
1. Vous recevrez deux notifications du portail Azure une fois que l’opération sera terminée. Une notification concerne la mise à jour du paramètre du serveur. La seconde concerne le redémarrage du serveur qui suit immédiatement.
1. Actualisez la page du portail Azure pour mettre à jour la barre d’outils Réplication. Vous pouvez désormais créer des réplicas en lecture pour ce serveur.

Pour créer un serveur de réplica en lecture interrégional dans la région cible à l’aide du portail Azure, procédez comme suit :

1. Sélectionnez le serveur Azure Database pour PostgreSQL existant que vous souhaitez utiliser comme serveur source.
1. Sélectionnez **Réplication** dans le menu, sous **PARAMÈTRES** .
1. Sélectionnez **Ajouter un réplica** .
1. Entrez un nom pour le serveur de réplica.
1. Sélectionnez l’emplacement du serveur de réplica. L’emplacement par défaut est le même que celui du serveur principal. Vérifiez que vous avez sélectionné l’emplacement cible souhaité où le réplica sera déployé.
1. Sélectionnez **OK** pour confirmer la création du réplica. Pendant la création de réplica, les données sont copiées depuis le serveur source vers le réplica. Le temps de création peut durer plusieurs minutes ou plus, proportionnellement à la taille du serveur source.

>[!NOTE]
> Lorsque vous créez un réplica, il n’hérite pas des règles de pare-feu ni des points de terminaison de service Vnet du serveur principal. Ces règles doivent être configurés indépendamment pour le réplica.

## <a name="move"></a>Déplacer

> [!IMPORTANT]
> Le serveur autonome ne peut pas être retransformé en réplica.
> Avant d’arrêter la réplication sur un réplica en lecture, vérifiez que celui-ci contient toutes les données nécessaires.

Afin d’arrêter la réplication vers le réplica depuis le portail Azure, procédez comme suit :

1. Une fois que le réplica a été créé, cherchez et sélectionnez votre serveur source Azure Database pour PostgreSQL. 
1. Sélectionnez **Réplication** dans le menu, sous **PARAMÈTRES** .
1. Sélectionnez le serveur réplica.
1. Sélectionnez **Arrêter la réplication** .
1. Validez que vous voulez arrêter la réplication en cliquant sur **OK** .

## <a name="clean-up-source-server"></a>Nettoyer le serveur source

Vous pourriez vouloir supprimer le serveur Azure Database pour PostgreSQL source. Pour ce faire, procédez comme suit :

1. Une fois que le réplica a été créé, cherchez et sélectionnez votre serveur source Azure Database pour PostgreSQL.
1. Dans la fenêtre **Vue d’ensemble** , sélectionnez **Supprimer** .
1. Tapez le nom du serveur source pour confirmer que vous souhaitez le supprimer.
1. Sélectionnez **Supprimer** .

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé un serveur Azure Database pour PostgreSQL d’une région à une autre en utilisant le portail Azure et vous avez ensuite nettoyé les ressources sources non nécessaires. 

- Découvrir plus en détail les [réplicas en lecture](concepts-read-replicas.md)
- Découvrir plus en détail la [gestion des réplica en lecture dans le portail Azure](howto-read-replicas-portal.md)
- En savoir plus sur les options de [continuité d’activité](concepts-business-continuity.md).