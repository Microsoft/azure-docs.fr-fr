---
title: Interface utilisateur de Microsoft Azure StorSimple Data Manager
description: Apprenez à utiliser l’interface utilisateur StorSimple Data Manager pour transformer les données hébergées sur les appareils StorSimple série 8000.
author: alkohli
ms.service: storsimple
ms.topic: conceptual
ms.date: 01/16/2018
ms.author: alkohli
ms.openlocfilehash: 802f732e8d62f5df861be525316b3c31ab4d0655
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94957926"
---
# <a name="manage-the-storsimple-data-manager-service-in-azure-portal"></a>Gérer le service StorSimple Data Manager dans le portail Azure

Cet article vous explique comment utiliser l’interface utilisateur StorSimple Data Manager pour transformer les données hébergées sur les appareils StorSimple série 8000. Les données transformées peuvent ensuite être consommées par d’autres services Azure, comme Azure Media Services, Azure HDInsight, Azure Machine Learning et Recherche cognitive Azure.


## <a name="use-storsimple-data-transformation"></a>Utiliser la transformation des données StorSimple

StorSimple Data Manager est la ressource dans laquelle la transformation des données est instanciée. Le service Transformation de données vous permet de transformer des données du format StorSimple au format natif dans des objets blob ou Azure Files. Pour transformer les données au format natif StorSimple, vous devez spécifier les détails sur l’appareil StorSimple série 8000 et les données intéressantes que vous souhaitez transformer.

### <a name="create-a-storsimple-data-manager-service"></a>Créer un service StorSimple Data Manager

Pour créer un service StorSimple Data Manager, procédez comme suit.

1. Utilisez les informations d’identification de votre compte Microsoft pour vous connecter au [portail Azure](https://portal.azure.com/).

2. Cliquez sur **+ Créer une ressource** et recherchez StorSimple Data Manager.

    ![Créer un service StorSimple Data Manager 1](./media/storsimple-data-manager-ui/create-service-1.png)

3. Cliquez sur StorSimple Data Manager, puis sur **Créer**.
    
    ![Créer un service StorSimple Data Manager 2](./media/storsimple-data-manager-ui/create-service-3.png)

3. Pour le nouveau service, spécifiez les informations suivantes :

   1. Fournissez un **nom de service** unique pour votre instance StorSimple Data Manager. Il s’agit d’un nom convivial qui peut être utilisé pour identifier le service. Le nom peut comporter entre 3 et 24 caractères qui peuvent être des lettres, des chiffres et des traits d’union. Il doit commencer et se terminer par une lettre ou un chiffre.

   2. Choisissez un **abonnement** dans la liste déroulante. L’abonnement est lié à votre compte de facturation. Ce champ est automatiquement rempli (et non sélectionnable) si vous n'avez qu’un seul abonnement.

   3. Sélectionnez un groupe de ressources existant ou créez-en un. Pour plus d’informations, consultez la page [Groupes de ressources dans Azure](../azure-resource-manager/management/manage-resource-groups-portal.md).

   4. Spécifiez l’**emplacement** qui héberge vos comptes de stockage et votre service StorSimple Data Manager. Votre service StorSimple Device Manager, votre service Data Manager et le compte de stockage associé doivent tous être situés dans les régions prises en charge.
    
   5. Pour disposer d’un lien vers ce service sur votre tableau de bord, sélectionnez **Épingler au tableau de bord**.
    
   6. Cliquez sur **Créer**.

      ![Créer un service StorSimple Data Manager 3](./media/storsimple-data-manager-ui/create-service-4.png)

La création de service dure quelques minutes. Une fois le service correctement créé, une notification apparaît et le nouveau service s’affiche.

### <a name="create-a-data-transformation-job-definition"></a>Créer une définition de travail de transformation de données

Au sein d’un service StorSimple Data Manager, vous devez créer une définition de travail de transformation de données. Une définition de travail spécifie des détails sur les données StorSimple que vous souhaitez transférer dans un compte de stockage au format natif. Après avoir créé une définition de travail, vous pouvez réexécuter ce travail avec d’autres paramètres d’exécution.

Procédez comme suit pour créer une définition de travail.

1. Accédez au service créé. Accédez à **Gestion > Définitions des travaux**.

2. Cliquez sur **+ Définition du travail**.

    ![Cliquez sur +Définition du travail](./media/storsimple-data-manager-ui/create-job-definition-1.png)

3. Entrez un nom pour votre définition de travail. Le nom peut contenir entre 3 et 63 caractères. Le nom peut contenir des majuscules, des minuscules ainsi que des lettres, des chiffres et des traits d’union.

4. Spécifiez un emplacement sur lequel s’exécute votre travail. Cet emplacement peut être différent de celui sur lequel le service est déployé.

5. Cliquez sur **Source** pour spécifier le référentiel de données source.

    ![Configurer le référentiel de données source](./media/storsimple-data-manager-ui/create-job-definition-2.png)

6. Dans la mesure où il s’agit d’un nouveau service Data Manager, aucun référentiel de données n’est configuré. Dans **Configurer la source de données**, spécifiez les détails de votre appareil StorSimple série 8000 et les données intéressantes.

   Pour ajouter votre instance StorSimple Device Manager en tant que référentiel de données, cliquez sur **Ajouter nouveau** dans la liste déroulante des référentiels de données, puis cliquez sur **Add Data Repository** (Ajouter un référentiel de données).

    ![Ajouter un nouveau référentiel de données](./media/storsimple-data-manager-ui/create-job-definition-3.png)
  
   1. Choisissez **StorSimple 8000 series Manager** comme type de référentiel de données.
    
   2. Entrez un nom convivial pour votre référentiel de données source.
    
   3. Dans la liste déroulante, choisissez un abonnement associé à votre service StorSimple Device Manager.
    
   4. Indiquez le nom du service StorSimple Device Manager comme **Ressource**.

   5. Entrez la clé **de chiffrement des données de service** pour le service StorSimple Device Manager. 

      ![Configurer le référentiel de données source 1](./media/storsimple-data-manager-ui/create-job-definition-4.png)

      Une fois l’opération terminée, cliquez sur **OK**. Votre référentiel de données est enregistré. Réutilisez ce StorSimple Device Manager dans d’autres définitions de travaux sans avoir à ressaisir ces paramètres. Une fois que vous avez cliqué sur **OK**, attendez quelques secondes pour voir apparaître le nouveau référentiel de données source dans la liste déroulante.

7. Dans la liste déroulante **Référentiel de données**, sélectionnez le référentiel de données que vous avez créé. 

   1. Entrez le nom de l’appareil StorSimple série 8000 qui contient les données intéressantes.

   2. Spécifiez le nom du volume résidant sur l’appareil StorSimple contenant vos données intéressantes.

   3. Dans la sous-section **Filtrer**, saisissez le répertoire racine comportant vos données intéressantes, au format _\MyRootDirectory\Data_. Les lettres de lecteurs de type _\C:\Data_ ne sont pas prises en charge. Les filtres de fichiers s’ajoutent également ici.

   4. Le service de transformation de données traite uniquement le dernier instantané de données qui est envoyé (push) à Azure.

   5. Cliquez sur **OK**.

      ![Configurer le référentiel de données source 2](./media/storsimple-data-manager-ui/create-job-definition-8.png)

8. Le référentiel de données cible doit maintenant être configuré. Choisissez les comptes de stockage, afin de placer des fichiers dans des objets blob de ce compte. Dans la liste déroulante, sélectionnez **Ajouter nouveau**, puis **Configuration des paramètres**.

9. Sélectionnez le type de référentiel cible que vous souhaitez ajouter et les autres paramètres qui lui sont associés.

    Si vous sélectionnez une cible de type de compte de stockage, vous pouvez spécifier un nom convivial, un abonnement (le même que celui du service ou un autre) et un compte de stockage.
        ![Configurer le référentiel de données cible 1](./media/storsimple-data-manager-ui/create-job-definition-10.png)

    Une file d’attente de stockage est créée durant l’exécution du travail. Cette file d’attente est renseignée avec des messages sur les objets blob transformés, au fur et à mesure de leur mise à disposition. Le nom de cette file d’attente est identique à celui de la définition du travail.
    
10. Après avoir ajouté le référentiel de données, attendez quelques minutes.
    
    1. Sélectionnez le référentiel que vous avez créé comme cible dans la liste déroulante **Nom de compte cible**.

    2. Choisissez des objets blob ou des fichiers comme type de stockage. Spécifiez le nom du conteneur de stockage où résident les données transformées. Cliquez sur **OK**.

        ![Configurer le compte de stockage du référentiel de données cible](./media/storsimple-data-manager-ui/create-job-definition-16.png)

11. Vous pouvez également sélectionner l’option pour afficher une estimation de la durée du travail avant de l’exécuter. Cliquez sur **OK** pour créer la définition du travail. Votre définition de travail est désormais terminée. Vous pouvez utiliser cette définition de travail plusieurs fois via l’interface utilisateur avec différents paramètres de runtime.

    ![Compléter la définition du travail](./media/storsimple-data-manager-ui/create-job-definition-13.png)

    La nouvelle définition de travail est ajoutée à la liste des définitions de travaux pour ce service.

### <a name="run-the-job-definition"></a>Exécuter la définition de travail

Chaque fois qu’il vous est nécessaire de transférer des données de StorSimple vers le compte de stockage spécifié dans la définition de travail, vous devez l’exécuter. Lors de l’exécution, certains paramètres peuvent être spécifiés différemment. La procédure comporte trois étapes :

1. Sélectionnez votre service StorSimple Data Manager, puis accédez à **Gestion > Définitions des travaux**. Cliquez sur la définition de travail que vous souhaitez exécuter.
     
     ![Lancer l’exécution du travail 1](./media/storsimple-data-manager-ui/start-job-run1.png)

2. Cliquez sur **Exécuter maintenant**.
     
     ![Lancer l’exécution du travail 2](./media/storsimple-data-manager-ui/start-job-run2.png)

3. Cliquez sur **Paramètres d'exécution** afin de modifier les paramètres à changer pour cette exécution de travail. Cliquez sur **OK**, puis sur **Exécuter** pour lancer votre travail.

    ![Lancer l’exécution du travail 3](./media/storsimple-data-manager-ui/start-job-run3.png)

4. Pour analyser ce travail, accédez à la section **Travaux** de votre instance StorSimple Data Manager. En plus du panneau **Travaux**, vous pouvez également examiner la file d’attente de stockage, dans laquelle un message est ajouté chaque fois qu’un fichier est déplacé de StorSimple vers le compte de stockage.

    ![Lancer l’exécution du travail 4](./media/storsimple-data-manager-ui/start-job-run4.png)

### <a name="view-logs-after-job-completion"></a>Afficher les journaux une fois le travail terminé

Une fois le travail terminé, vous pouvez afficher son état. L’état du travail peut être **Succès**, **Succès partiel** ou **Échec**. Vous pouvez afficher la liste des fichiers qui ont été copiés et de ceux qui n’ont pas pu être copiés. Ces listes sont disponibles dans un conteneur appelé **storsimple-data-manager-joblogs** de votre compte de stockage cible. Dans ce conteneur, vous pouvez rechercher un dossier portant le même nom que la définition de votre travail. Dans ce cas, un dossier est créé pour chaque exécution de travail qui contient vos listes. Le nom de ce dossier correspondra au GUID du travail, qui se trouve dans la page des détails du travail. Sinon, dans la plupart des cas, vous verrez un lien pour les journaux de copie dans la page des travaux.
Dans ce dossier, vous verrez deux ensembles de fichiers CSV. Tous les fichiers qui commencent par **copiedfilelist...** contiennent la liste des fichiers copiés. Tous les fichiers qui commencent par **failedfilelist...** contiennent la liste des fichiers qui n’ont pas pu être copiés, ainsi qu’un message d’erreur.


## <a name="next-steps"></a>Étapes suivantes

[Utilisez .NET SDK pour lancer les travaux StorSimple Data Manager](storsimple-data-manager-dotnet-jobs.md).