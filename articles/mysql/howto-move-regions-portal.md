---
title: Déplacer des régions Azure - Portail Azure - Azure Database pour MySQL
description: Déplacez un serveur azure Database pour MySQL d’une région Azure vers une autre à l’aide d’un réplica en lecture et du Portail Azure.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 06/26/2020
ms.openlocfilehash: 279526d31adf25d1f4cd5a6d9b15519679bd88b9
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94540227"
---
# <a name="move-an-azure-database-for-mysql-server-to-another-region-by-using-the-azure-portal"></a>Déplacer un serveur Azure Database pour MySQL vers une autre région à l’aide du Portail Azure

Il existe différents scénarios pour déplacer un serveur Azure Database pour MySQL existant d’une région à une autre. Par exemple, vous pourriez vouloir déplacer un serveur de production vers une autre région dans le cadre de votre planification de la récupération d'urgence.

Vous pouvez utiliser un [réplica en lecture entre région](concepts-read-replicas.md#cross-region-replication) Azure Database pour MySQL pour terminer le déplacement vers une autre région. Pour ce faire, créez tout d’abord un réplica en lecture dans la région cible. Ensuite, arrêtez la réplication vers le serveur de réplica en lecture pour en faire un serveur autonome qui accepte à la fois le trafic en lecture et en écriture. 

> [!NOTE]
> Cet article se concentre sur le déplacement de votre serveur vers une région différente. Si vous souhaitez déplacer votre serveur vers un groupe de ressources ou un abonnement différent, référez vous à l’article de [déplacement](../azure-resource-manager/management/move-resource-group-and-subscription.md). 

## <a name="prerequisites"></a>Prérequis

- La fonctionnalité de réplica en lecture est disponible uniquement pour les serveurs Azure Database pour MySQL dans les niveaux tarifaires Usage général ou Mémoire optimisée. Vérifiez que le serveur source se trouve dans l’un de ces niveaux tarifaires.

- Assurez-vous que votre serveur source Azure Database pour MySQL se trouve dans la région Azure à partir de laquelle vous souhaitez vous déplacer.

## <a name="prepare-to-move"></a>Préparer le déplacement

Pour créer un serveur de réplica en lecture interrégional dans la région cible à l’aide du portail Azure, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Sélectionnez le serveur Azure Database pour MySQL que vous souhaitez utiliser comme serveur source. Cette action ouvre la page **Vue d’ensemble**.
1. Sélectionnez **Réplication** dans le menu, sous **PARAMÈTRES**.
1. Sélectionnez **Ajouter un réplica**.
1. Entrez un nom pour le serveur de réplica.
1. Sélectionnez l’emplacement du serveur de réplica. L'emplacement par défaut est le même que celui du serveur source. Vérifiez que vous avez sélectionné l’emplacement cible souhaité où le réplica sera déployé.
1. Sélectionnez **OK** pour confirmer la création du réplica. Pendant la création de réplica, les données sont copiées depuis le serveur source vers le réplica. Le temps de création peut durer plusieurs minutes ou plus, proportionnellement à la taille du serveur source.

>[!NOTE]
> Lorsque vous créez un réplica, il n'hérite pas des points de terminaison de service VNet du serveur source. Ces règles doivent être configurés indépendamment pour le réplica.

## <a name="move"></a>Déplacer

> [!IMPORTANT]
> Le serveur autonome ne peut pas être retransformé en réplica.
> Avant d’arrêter la réplication sur un réplica en lecture, vérifiez que celui-ci contient toutes les données nécessaires.

En arrêtant la réplication sur le serveur de réplication, il devient un serveur autonome. Afin d’arrêter la réplication vers le réplica depuis le portail Azure, procédez comme suit :

1. Une fois le réplica créé, recherchez et sélectionnez votre serveur source Azure Database pour MySQL. 
1. Sélectionnez **Réplication** dans le menu, sous **PARAMÈTRES**.
1. Sélectionnez le serveur réplica.
1. Sélectionnez **Arrêter la réplication**.
1. Validez que vous voulez arrêter la réplication en cliquant sur **OK**.

## <a name="clean-up-source-server"></a>Nettoyer le serveur source

Vous pourriez vouloir supprimer le serveur source Azure Database pour MySQL. Pour ce faire, procédez comme suit :

1. Une fois le réplica créé, recherchez et sélectionnez votre serveur source Azure Database pour MySQL.
1. Dans la fenêtre **Vue d’ensemble**, sélectionnez **Supprimer**.
1. Tapez le nom du serveur source pour confirmer que vous souhaitez le supprimer.
1. Sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé un serveur Azure Database pour MySQL d’une région à une autre en utilisant le portail Azure et vous avez ensuite nettoyé les ressources sources non nécessaires. 

- Découvrir plus en détail les [réplicas en lecture](concepts-read-replicas.md)
- Découvrir plus en détail la [gestion des réplica en lecture dans le portail Azure](howto-read-replicas-portal.md)
- En savoir plus sur les options de [continuité d’activité](concepts-business-continuity.md).