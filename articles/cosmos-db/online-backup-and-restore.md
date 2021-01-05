---
title: Sauvegarde en ligne et restauration de données à la demande dans Azure Cosmos DB
description: Cet article décrit le fonctionnement de la sauvegarde automatique et de la restauration de données à la demande, ainsi que la configuration de l’intervalle de sauvegarde et de la rétention. Il explique également comment contacter le support pour demander une restauration des données dans Azure Cosmos DB.
author: kanshiG
ms.service: cosmos-db
ms.topic: how-to
ms.date: 10/13/2020
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: 43625a80df76ff35b8bb1804df5f5fd1524326c5
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097530"
---
# <a name="online-backup-and-on-demand-data-restore-in-azure-cosmos-db"></a>Sauvegarde en ligne et restauration de données à la demande dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB sauvegarde automatiquement vos données à intervalles réguliers. Les sauvegardes automatiques sont effectuées sans affecter les performances ou la disponibilité des opérations de base de données. Toutes les sauvegardes sont stockées séparément dans un service de stockage, et ces sauvegardes sont répliquées globalement pour garantir la résilience contre les sinistres régionaux. Les sauvegardes automatiques sont utiles dans les scénarios où vous supprimez ou mettez à jour accidentellement votre compte, base de données ou conteneur Azure Cosmos et où vous avez besoin ultérieurement de récupérer les données.

## <a name="automatic-and-online-backups"></a>Sauvegardes automatiques et en ligne

Avec Azure Cosmos DB, vos données et leurs sauvegardes sont rendues hautement redondantes et résilientes aux sinistres régionaux. Les étapes suivantes montrent comment Azure Cosmos DB sauvegarde les données :

* Azure Cosmos DB effectue automatiquement une sauvegarde complète de votre base de données toutes les 4 heures, quel que soit le moment. Par défaut, seules les deux dernières sauvegardes sont stockées. Si les intervalles par défaut sont insuffisants pour vos charges de travail, vous pouvez modifier l’intervalle de sauvegarde et la période de rétention à partir du portail Azure. Vous pouvez modifier la configuration de la sauvegarde pendant ou après la création du compte Azure Cosmos. En cas de suppression du conteneur ou de la base de données, Azure Cosmos DB conserve leurs captures instantanées pendant 30 jours.

* Azure Cosmos DB stocke ces sauvegardes dans Stockage Blob Azure, tandis que les données réelles résident localement dans Azure Cosmos DB.

* Pour garantir une latence faible, la capture instantanée de votre sauvegarde est stockée dans le Stockage Blob Azure, dans la région d’écriture actuelle (ou dans l’ **une** des régions d’écriture si vous avez une configuration d’écriture multirégion). Pour assurer la résilience contre les sinistres régionaux, chaque capture instantanée de vos données de sauvegarde dans le stockage blob Azure est à nouveau répliqué vers une autre région par le biais du stockage géoredondant (GRS). La région vers laquelle la sauvegarde est répliquée dépend de votre région source et de la paire régionale associée à la région source. Pour plus d’informations, consultez la [liste des paires de régions Azure géoredondantes](../best-practices-availability-paired-regions.md). Vous ne pouvez pas accéder directement à cette sauvegarde. L’équipe Azure Cosmos DB restaurera votre sauvegarde lorsque vous le demanderez par le biais d’une demande de support.

   L’image suivante montre comment un conteneur Azure Cosmos avec les trois partitions physiques principales dans la région USA Ouest est sauvegardé dans un compte Stockage Blob Azure distant dans la région USA Ouest, puis répliqué dans la région USA Est :

  :::image type="content" source="./media/online-backup-and-restore/automatic-backup.png" alt-text="Sauvegardes complètes périodiques de toutes les entités Cosmos DB dans Stockage Azure GRS" border="false":::

* Les sauvegardes sont effectuées sans affecter les performances ou la disponibilité de votre application. Azure Cosmos DB effectue la sauvegarde des données en arrière-plan sans consommer de débit (RU) provisionné supplémentaire et sans affecter les performances ou la disponibilité de votre base de données.

## <a name="modify-the-backup-interval-and-retention-period"></a><a id="configure-backup-interval-retention"></a>Modifier l’intervalle de sauvegarde et la période de rétention

Azure Cosmos DB effectue automatiquement une sauvegarde complète de vos données toutes les 4 heures, quel que soit le moment. Les deux dernières sauvegardes sont stockées. Il s’agit de la configuration par défaut offerte sans coût supplémentaire. Vous pouvez modifier l’intervalle de sauvegarde et la période de rétention par défaut pendant ou après la création du compte Azure Cosmos. La configuration de la sauvegarde est définie au niveau du compte Azure Cosmos, et vous devez la configurer sur chaque compte. Une fois que vous avez configuré les options de sauvegarde pour un compte, la configuration est appliquée à tous les conteneurs de ce compte. Actuellement, vous pouvez modifier les options de sauvegarde uniquement à partir du portail Azure.

Si vous avez accidentellement supprimé ou endommagé vos données, **avant de créer une demande de support pour restaurer les données, veillez à allonger la période de rétention des sauvegardes de votre compte à au moins sept jours. Il est préférable d’allonger la période de rétention dans les 8 heures qui suivent l’événement.** Ainsi, l’équipe Azure Cosmos DB dispose de suffisamment de temps pour restaurer votre compte.

Procédez comme suit pour modifier les options de sauvegarde par défaut pour un compte Azure Cosmos existant :

1. Connectez-vous au [portail Azure.](https://portal.azure.com/)
1. Accédez à votre compte Azure Cosmos et ouvrez le volet **Sauvegarder et restaurer**. Mettez à jour l’intervalle de sauvegarde et la période de rétention de sauvegarde selon les besoins.

   * **Intervalle de sauvegarde**  : intervalle auquel Azure Cosmos DB tente d’effectuer une sauvegarde de vos données. La sauvegarde prend un certain temps et, dans certains cas, elle peut échouer en raison de dépendances en aval. Azure Cosmos DB tente d’effectuer une sauvegarde à l’intervalle configuré. Il n’est cependant nullement garanti que la sauvegarde s’achève dans cet intervalle de temps. Vous pouvez configurer cette valeur en heures ou minutes. L’intervalle de sauvegarde ne peut pas être inférieur à 1 heure et supérieur à 24 heures. Lorsque vous modifiez cet intervalle, le nouvel intervalle prend effet à partir de l’heure à laquelle la dernière sauvegarde a été effectuée.

   * **Rétention des sauvegardes**  : période pendant laquelle chaque sauvegarde est conservée. Vous pouvez configurer cette valeur en heures ou en jours. La période de rétention minimale ne peut pas être inférieure à deux fois l’intervalle de sauvegarde (en heures), et ne peut pas être supérieure à 720 heures.

   * **Copies des données conservées**  : par défaut, deux copies de sauvegarde de vos données sont offertes gratuitement. Des frais supplémentaires sont facturés si vous avez besoin de plus de deux copies. Pour connaître le prix exact des copies supplémentaires, consultez la section Stockage consommé dans la [page relative à la tarification](https://azure.microsoft.com/pricing/details/cosmos-db/).

   :::image type="content" source="./media/online-backup-and-restore/configure-backup-interval-retention.png" alt-text="Configurer l’intervalle de sauvegarde et la rétention pour un compte Azure Cosmos existant" border="true":::

Si vous configurez les options de sauvegarde lors de la création du compte, vous pouvez configurer une **Stratégie de sauvegarde** **Périodique** ou **Continue**. La stratégie périodique vous permet de configurer l’intervalle de sauvegarde et la rétention des sauvegardes. La stratégie continue est actuellement disponible uniquement par inscription. L’équipe Azure Cosmos DB évalue votre charge de travail et approuve votre demande.

:::image type="content" source="./media/online-backup-and-restore/configure-periodic-continuous-backup-policy.png" alt-text="Configurer une stratégie de sauvegarde périodique ou continue pour de nouveaux comptes Azure Cosmos" border="true":::

## <a name="request-data-restore-from-a-backup"></a>Demander une restauration des données à partir d’une sauvegarde

En cas de suppression accidentelle de votre base de données ou conteneur, vous pouvez [émettre un ticket de support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) ou [appeler le support Azure](https://azure.microsoft.com/support/options/) pour restaurer les données à partir de sauvegardes en ligne automatiques. Le support Azure est disponible uniquement pour certains plans, tels que **Standard** , **Développeur** et des plans plus élevés. Le support technique Azure n’est pas disponible avec un plan **De base**. Pour plus d’informations sur les différents plans de support technique, consultez la page [Plans de support Azure](https://azure.microsoft.com/support/plans/).

Pour restaurer une capture instantanée spécifique de la sauvegarde, Azure Cosmos DB exige que les données soient accessibles pendant la durée du cycle de sauvegarde de cette capture instantanée.
Vous devez avoir les informations suivantes avant de demander une restauration :

* Votre ID d’abonnement.

* En fonction de la manière dont vos données ont été accidentellement supprimées ou modifiées, vous devez vous préparer à disposer d’informations supplémentaires. Nous vous recommandons d’avoir ces informations à portée de main à l’avance, afin de minimiser les aller-retours qui peuvent être préjudiciables dans certains cas d’urgence.

* Si l’intégralité du compte Azure Cosmos DB est supprimé, vous devez fournir le nom du compte supprimé. Si vous créez un autre compte avec le même nom que le compte supprimé, partagez-le avec l’équipe de support, car cela aide à identifier le compte correct. Nous vous recommandons d’émettre différents tickets de support pour chaque compte supprimé, car cela permet de réduire la confusion de l’état de la restauration.

* Si une ou plusieurs bases de données sont supprimées, vous devez fournir le compte Azure Cosmos ainsi que les noms des bases de données Azure Cosmos, et spécifier s’il existe une base de données portant le même nom.

* Si un ou plusieurs conteneurs sont supprimés, vous devez fournir le nom du compte Azure Cosmos, les noms des bases de données et les noms des conteneurs. Spécifiez aussi s’il existe un conteneur portant le même nom.

* Si vous avez accidentellement supprimé ou endommagé vos données, vous devez contacter le [support Azure](https://azure.microsoft.com/support/options/) dans les 8 heures afin que l’équipe Azure Cosmos DB puisse vous aider à les restaurer à partir des sauvegardes. **Avant de créer une demande de support pour restaurer les données, veillez à [allonger la période de rétention des sauvegardes](#configure-backup-interval-retention) de votre compte à au moins sept jours. Il est préférable d’allonger la période de rétention dans les 8 heures qui suivent l’événement.** Ainsi, l’équipe de support Azure Cosmos DB disposera de suffisamment de temps pour restaurer votre compte.

En plus du nom du compte Azure Cosmos, des noms de bases de données et des noms de conteneurs, vous devez spécifier le moment dans le temps auquel les données peuvent être restaurées. Il est important d’être aussi précis que possible, afin de nous aider à déterminer les sauvegardes disponibles à ce moment-là. **Il est également important de spécifier l’heure au format UTC.**

La capture d’écran suivante illustre comment créer une demande de support pour un conteneur (collection/graphe/table) afin de restaurer des données à l’aide du portail Azure. Fournissez des détails supplémentaires tels que le type de données, l’objectif de la restauration et le moment auquel les données ont été supprimées, afin de nous aider à affecter une priorité à la demande.

:::image type="content" source="./media/online-backup-and-restore/backup-support-request-portal.png" alt-text="Créer une demande de support de sauvegarde à l’aide du portail Azure":::

## <a name="considerations-for-restoring-the-data-from-a-backup"></a>Considérations relatives à la restauration des données à partir d’une sauvegarde

Vous risquez d’effacer ou de modifier accidentellement vos données dans l’un des cas suivants :  

* Vous supprimez le compte Azure Cosmos entier.

* Vous supprimez une ou plusieurs bases de données Azure Cosmos.

* Vous supprimez un ou plusieurs conteneurs Azure Cosmos.

* Vous supprimez ou modifiez des éléments Azure Cosmos (par exemple, des documents) dans un conteneur. Dans ce cas, on parle généralement d’« altération des données ».

* Une base de données d’offre partagée ou des conteneurs au sein d’une base de données d’offre partagée sont supprimés ou endommagés.

Azure Cosmos DB peut restaurer les données dans tous les scénarios ci-dessus. Lors de la restauration, un nouveau compte Azure Cosmos est créé pour contenir les données restaurées. Le nom du nouveau compte, s’il n’est pas spécifié, a le format `<Azure_Cosmos_account_original_name>-restored1`. Le dernier chiffre est incrémenté quand plusieurs tentatives de restauration sont effectuées. Vous ne pouvez pas restaurer les données sur un compte Azure Cosmos créé au préalable.

Quand vous supprimez accidentellement un compte Azure Cosmos, nous pouvons restaurer les données dans un compte du même nom à condition que le nom du compte ne soit pas utilisé. Nous vous recommandons donc de ne pas recréer le compte après l’avoir supprimé. En effet, cela empêche les données restaurées d’utiliser le même nom, et complique la découverte du compte approprié à restaurer.

Quand vous supprimez accidentellement une base de données Azure Cosmos, nous pouvons restaurer la base de données entière ou un sous-ensemble des conteneurs à l’intérieur de celle-ci. Vous pouvez sélectionner des conteneurs spécifiques dans des bases de données et les restaurer sur un nouveau compte Azure Cosmos.

Quand vous supprimez ou modifiez accidentellement un ou plusieurs éléments d’un conteneur (altération des données), vous devez spécifier l’heure à laquelle effectuer la restauration. L’heure est importante en cas d’altération des données. Le conteneur étant en ligne, la sauvegarde est toujours en cours d’exécution. Par conséquent, si vous attendez au-delà de la période de rétention (de huit heures par défaut), les sauvegardes seront remplacées. Pour éviter le remplacement de la sauvegarde, allongez la période de rétention de la sauvegarde de votre compte à au moins sept jours. Il est préférable d’allonger la période de rétention dans les 8 heures qui suivent l’altération des données.

Si vous avez accidentellement supprimé ou endommagé vos données, vous devez contacter le [support Azure](https://azure.microsoft.com/support/options/) dans les 8 heures afin que l’équipe Azure Cosmos DB puisse vous aider à les restaurer à partir des sauvegardes. Ainsi, l’équipe de support Azure Cosmos DB disposera de suffisamment de temps pour restaurer votre compte.

> [!NOTE]
> Une fois les données restaurées, les fonctionnalités ou les paramètres source ne sont pas tous reportés sur le compte restauré. Les paramètres suivants ne sont pas reportés sur le nouveau compte :
> * Listes de contrôle d'accès VNET
> * Procédures stockées, déclencheurs et fonctions définies par l’utilisateur
> * Paramètres multirégion  

Si vous approvisionnez le débit au niveau de la base de données, le processus de sauvegarde et de restauration se produit au niveau de la base de données entière, non au niveau de conteneurs individuels. Dans ce cas, vous ne pouvez pas sélectionner un sous-ensemble de conteneurs à restaurer.

## <a name="options-to-manage-your-own-backups"></a>Options pour gérer vos propres sauvegardes

Avec les comptes d’API SQL Azure Cosmos DB, vous pouvez également tenir à jour vos propres sauvegardes en adoptant l’une des approches suivantes :

* Utiliser [Azure Data Factory](../data-factory/connector-azure-cosmos-db.md) pour déplacer régulièrement les données vers un stockage de votre choix

* Utilisez le [flux de modification](change-feed.md) d’Azure Cosmos DB pour lire périodiquement les données pour des sauvegardes complètes ou des modifications incrémentielles, et les stocker dans votre propre stockage.

## <a name="post-restore-actions"></a>Actions de post-restauration

L’objectif principal de la restauration des données est de récupérer les données que vous avez supprimées ou modifiées accidentellement. Nous recommandons donc d’inspecter d’abord le contenu des données restaurées, afin de vous assurer qu’il correspond à vos attentes. Si tout semble correct, vous pouvez de nouveau migrer les données vers le compte principal. Bien qu’il soit possible d’utiliser le compte restauré comme nouveau compte actif, cette option n’est pas recommandée si vous avez des charges de travail de production. 

Après avoir restauré les données, vous recevez une notification concernant le nom du nouveau compte (il est généralement au format `<original-name>-restored1`) et le moment auquel le compte a été restauré. Le compte restauré aura le même débit provisionné et les mêmes stratégies d’indexation, et il se trouvera dans la même région que le compte d’origine. Un utilisateur administrateur ou coadministrateur de l’abonnement peut voir le compte restauré.

### <a name="migrate-data-to-the-original-account"></a>Migration des données vers le compte d’origine

Voici différentes méthodes pour migrer les données vers le compte d’origine :

* Vous utilisez l’[outil de migration de données Azure Cosmos DB](import-data.md).
* Vous utilisez Le service [Azure Data Factory](../data-factory/connector-azure-cosmos-db.md).
* Vous utilisez le [flux de modification](change-feed.md) dans Azure Cosmos DB.
* Vous pouvez écrire votre propre code personnalisé.

Nous vous conseillons de supprimer le conteneur ou la base de données immédiatement après la migration des données. Si vous ne supprimez pas les bases de données ou les conteneurs restaurés, ils génèreront des coûts de stockage, de sortie et d’unités de requête.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez maintenant découvrir comment restaurer des données à partir d’un compte Azure Cosmos, ou comment migrer des données vers un compte Azure Cosmos.

* Pour effectuer une demande de restauration, [émettez un ticket à partir du portail Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
* [Utiliser le flux de modification Cosmos DB](change-feed.md) pour déplacer des données vers Azure Cosmos DB.
* [Utiliser Azure Data Factory](../data-factory/connector-azure-cosmos-db.md) pour déplacer des données vers Azure Cosmos DB.

