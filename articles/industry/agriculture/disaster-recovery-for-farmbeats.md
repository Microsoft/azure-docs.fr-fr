---
title: Récupération d'urgence pour FarmBeats
description: Cet article explique comment la récupération des données vous protège contre la perte de vos données.
author: uhabiba04
ms.topic: article
ms.date: 04/13/2020
ms.author: v-umha
ms.openlocfilehash: bdd59d078da1df021b7e23ec858ba7305f72cfa1
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92164208"
---
# <a name="disaster-recovery-for-farmbeats"></a>Récupération d'urgence pour FarmBeats

La récupération des données vous protège contre la perte des données lors d'un événement tel que l'effondrement d'une région Azure. Vous pouvez entamer un basculement et récupérer les données stockées dans votre déploiement FarmBeats.

La fonctionnalité de récupération des données n'est pas disponible par défaut dans Azure FarmBeats. Vous pouvez définir cette fonctionnalité manuellement en configurant les ressources Azure à utiliser par FarmBeats pour stocker des données dans une région jumelée Azure. Utilisez une approche active/passive pour activer la récupération.

Les sections suivantes fournissent des informations sur la configuration de la récupération des données dans Azure FarmBeats :

- [Activer la redondance des données](#enable-data-redundancy)
- [Restaurer le service à partir d'une sauvegarde en ligne](#restore-service-from-online-backup)


## <a name="enable-data-redundancy"></a>Activer la redondance des données

FarmBeats stocke les données dans les trois services internes suivants d'Azure : **Stockage Azure** , **Cosmos DB** et **Time Series Insights** . Procédez comme suit pour activer la redondance des données de ces services dans une région jumelée Azure :

1.  **Stockage Azure**  : suivez ces instructions pour activer la redondance des données sur chacun des comptes de stockage de votre déploiement FarmBeats.
2.  **Azure Cosmos DB**  : suivez ces instructions pour activer la redondance des données sur le compte Cosmos DB de votre déploiement FarmBeats.
3.  **Azure Time Series Insights (TSI)**  : la redondance des données n'est actuellement pas disponible pour TSI. Pour récupérer les données Time Series Insights, accédez à votre capteur/partenaire météo et renvoyez (push) les données vers le déploiement FarmBeats.

## <a name="restore-service-from-online-backup"></a>Restaurer le service à partir d'une sauvegarde en ligne

Vous pouvez entamer le basculement et récupérer les données stockées dans chacun des magasins de données de votre déploiement FarmBeats, comme mentionné ci-dessus. Une fois les données récupérées pour Stockage Azure et Cosmos DB, créez un autre déploiement FarmBeats dans la région jumelée Azure, puis configurez le nouveau déploiement de manière à utiliser les données des magasins de données restaurés (à savoir Stockage Azure et Cosmos DB) en procédant comme suit :

1. [Configurer Cosmos DB](#configure-cosmos-db)
2. [Configurer un compte de stockage](#configure-storage-account)


### <a name="configure-cosmos-db"></a>Configurer Cosmos DB

Copiez la clé d'accès de l'instance restaurée de Cosmos DB et mettez à jour le nouveau coffre de clés FarmBeats Datahub.


  ![Capture d’écran mettant en évidence où récupérer la copie de la clé d’accès.](./media/disaster-recovery-for-farmbeats/key-vault-secrets.png)

> [!NOTE]
> Copiez l'URL de l'instance restaurée de Cosmos DB et mettez-la à jour dans la nouvelle configuration de FarmBeats Datahub App Service. Vous pouvez maintenant supprimer le compte Cosmos DB dans le nouveau déploiement FarmBeats.

  ![Capture d’écran montrant où copier l’URL de l’instance restaurée de Cosmos DB.](./media/disaster-recovery-for-farmbeats/configuration.png)

### <a name="configure-storage-account"></a>Configurer un compte de stockage

Copiez la clé d'accès du compte de stockage restauré et mettez-la à jour dans le nouveau coffre de clés FarmBeats Datahub.

![Capture d’écran montrant où copier la clé d’accès du compte de stockage restauré.](./media/disaster-recovery-for-farmbeats/key-vault-7-secrets.png)

>[!NOTE]
> Veillez à mettre à jour le nom du compte de stockage dans le nouveau fichier de configuration de la machine virtuelle FarmBeats Batch.

![Récupération d’urgence](./media/disaster-recovery-for-farmbeats/batch-prep-files.png)

De même, si vous avez activé la récupération des données pour votre compte de stockage Accélérateur, suivez l'étape 2 pour mettre à jour la clé d'accès et le nom du compte de stockage Accélérateur dans la nouvelle instance de FarmBeats.
