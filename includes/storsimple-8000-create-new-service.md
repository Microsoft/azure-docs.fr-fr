---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 746198f87e23cd7aca2a3177c23974917cb4b12a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96025770"
---
#### <a name="to-create-a-new-service"></a>Création d’un service

1. Utilisez les informations d’identification de votre compte Microsoft pour vous connecter au [portail Azure](https://portal.azure.com/).

2. Dans le portail Azure, cliquez sur **Créer une ressource**, puis, dans la Place de marché, cliquez sur **Afficher tout**.

    ![Créer StorSimple Device Manager](./media/storsimple-8000-create-new-service/createssdevman1.png)

    Recherchez _StorSimple physique_. Sélectionnez **Série d’appareils physiques StorSimple** et cliquez dessus, puis cliquez sur **Créer**. Autrement, dans le portail Azure, cliquez sur **+** , puis sous **Stockage**, cliquez sur **Série d’appareils physiques StorSimple**.

    ![Créer StorSimple Device Manager 2](./media/storsimple-8000-create-new-service/createssdevman11.png)

3. Dans le panneau **StorSimple Device Manager** qui s’ouvre, procédez comme suit :

   1. Fournissez un unique **Nom de la ressource** pour votre service. Il s’agit d’un nom convivial qui peut être utilisé pour identifier le service. Le nom peut comporter entre 2 et 50 caractères qui peuvent être des lettres, des chiffres et des traits d’union. Il doit commencer et se terminer par une lettre ou un chiffre.

   2. Choisissez un **abonnement** dans la liste déroulante. L’abonnement est lié à votre compte de facturation. Ce champ n’est pas présent si vous n’avez qu’un seul abonnement.

   3. Pour le **Groupe de ressources**, sélectionnez un groupe existant en cliquant sur **Utiliser existant** ou créez-en un avec l’option **Créer**. Pour plus d’informations, consultez la page [Groupes de ressources dans Azure](../articles/azure-resource-manager/management/manage-resource-groups-portal.md).

   4. Indiquez un **emplacement** pour votre service. De manière générale, choisissez l’emplacement le plus proche de la région géographique dans laquelle vous souhaitez déployer votre appareil. Vous pouvez également prendre en compte les considérations suivantes :

      * Si vous avez des charges de travail existantes dans Azure que vous souhaitez également déployer avec votre périphérique StorSimple, nous vous invitons à utiliser ce centre de données.
      * Votre service StorSimple Device Manager et le stockage Azure peuvent se trouver dans deux emplacements différents. Dans ce cas, vous devez créer séparément le gestionnaire d’appareil StorSimple et le compte de stockage Azure. Pour créer un compte de stockage Azure, accédez au service Stockage Azure dans le portail Azure et suivez les étapes indiquées dans [Créer un compte Stockage Azure](../articles/storage/common/storage-account-create.md). Une fois que vous avez créé ce compte, ajoutez-le au service StorSimple Device Manager en suivant les étapes indiquées dans [Configurer un nouveau compte de stockage pour le service](../articles/storsimple/storsimple-8000-deployment-walkthrough-u2.md#configure-a-new-storage-account-for-the-service).

   5. Sélectionnez **Créer un nouveau compte de stockage** pour créer automatiquement un compte de stockage avec le service. Spécifiez un nom pour ce compte de stockage. Si vous avez besoin de vos données dans un autre emplacement, désactivez cette case à cocher.

   6. Cochez la case **Épingler au tableau de bord** si vous souhaitez disposer d’un lien rapide vers ce service sur votre tableau de bord.

   7. Pour créer le service StorSimple Device Manager, cliquez sur **Créer**.

       ![Créer StorSimple Device Manager 3](./media/storsimple-8000-create-new-service/createssdevman2.png)

La création de service dure quelques minutes. Une fois le service correctement créé, une notification s’affiche et le nouveau panneau de service s’ouvre.

![Créer StorSimple Device Manager 4](./media/storsimple-8000-create-new-service/createssdevman5.png)