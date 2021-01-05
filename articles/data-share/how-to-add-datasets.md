---
title: Ajouter des jeux de données à Azure Data Share existant
description: Découvrez comment ajouter des jeux de données à un partage de données existant dans Azure Data Share, et comment les partager avec les mêmes destinataires.
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: how-to
ms.date: 07/30/2020
ms.openlocfilehash: 3bfa911921e9bacde2649ee8c4f0d4bc31b56f54
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92910540"
---
# <a name="how-to-add-datasets-to-an-existing-share-in-azure-data-share"></a>Comment ajouter des jeux de données à un partage existant dans Azure Data Share

Cet article explique comment ajouter des jeux de données à un partage de données préexistant à l’aide d’Azure Data Share. Cela vous permet de partager plus de données avec les mêmes destinataires sans avoir à créer un nouveau partage.

Pour plus d’informations sur l’ajout de jeux de données au fur et à mesure de la création d’un partage, consultez le tutoriel [Partager des données](share-your-data.md).

## <a name="navigate-to-a-sent-data-share"></a>Accéder à un partage de données envoyé

Dans Azure Data Share, accédez à votre partage envoyé et sélectionnez l’onglet **Jeux de données**. Cliquez sur le bouton **+Ajouter des jeux de données** pour ajouter d’autres jeux de données.

![Capture d’écran de l’option Ajouter des jeux de données sélectionnée](./media/how-to/how-to-add-datasets/add-datasets.png)

Dans le volet de droite, sélectionnez le type de jeu de données que vous souhaitez ajouter, puis cliquez sur **Suivant**. Sélectionnez l’abonnement et le groupe de ressources des données que vous souhaitez ajouter. À l’aide des flèches de liste déroulante, recherchez puis activez la case à cocher en regard des données à ajouter.

![Capture d’écran du volet Ajouter un Stockage Blob permettant de sélectionner des données](./media/how-to/how-to-add-datasets/add-datasets-side.png)

Une fois que vous avez cliqué sur **Ajouter des jeux de données** , les jeux de données sont ajoutés à votre partage. Remarque : Vos consommateurs doivent déclencher un instantané afin de voir les nouveaux jeux de données. Si des paramètres d’instantané sont configurés, les consommateurs voient les nouveaux jeux de données une fois que l’instantané planifié suivant a été pris. Si aucun paramètre d’instantané n’est configuré, le consommateur doit déclencher manuellement une copie complète ou incrémentielle des données pour recevoir les mises à jour. Pour plus d’informations sur les instantanés, consultez [Instantanés](terminology.md).

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [ajouter des destinataires à un partage de données existant](how-to-add-recipients.md).