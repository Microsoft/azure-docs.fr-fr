---
title: Comment tester votre application dans Azure | Microsoft Docs
description: Découvrez, dans le cadre d’un labo, comment créer un partage de fichiers et le monter sur votre ordinateur local et une machine virtuelle, puis déployer des applications de bureau/web sur le partage de fichiers et les tester.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: b2dbbf349da4e352fe20a22db03cc9063d801990
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87282243"
---
# <a name="test-your-app-in-azure"></a>Tester votre application dans Azure 
Cet article décrit la procédure de test de votre application dans Azure à l’aide de DevTest Labs. Tout d’abord, dans le cadre d’un labo, vous configurez un partage de fichiers et le montez en tant que lecteur sur votre ordinateur de développement local et une machine virtuelle. Ensuite, vous utilisez Visual Studio 2019 pour déployer votre application sur le partage de fichiers afin de pouvoir exécuter l’application sur la machine virtuelle.  

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis 
1. [Créez un abonnement Azure](https://azure.microsoft.com/free/) si vous n’en avez pas, puis connectez-vous au [portail Azure](https://portal.azure.com).
2. Suivez les instructions de [cet article](devtest-lab-create-lab.md) pour créer un labo à l’aide d’Azure DevTest Labs. Épinglez le labo à votre tableau de bord afin de pouvoir le retrouver facilement lors de la prochaine connexion. Azure DevTest Labs vous permet de créer rapidement des ressources dans Azure en minimisant le gaspillage et en contrôlant les coûts. Pour en savoir plus sur DevTest Labs, voir [Vue d’ensemble](devtest-lab-overview.md). 
3. Créez un compte Stockage Azure dans le groupe de ressources du labo en suivant les instructions de l’article [Créer un compte de stockage](../storage/common/storage-account-create.md). Dans la page **Créer un compte de stockage**, sélectionnez **Utiliser l’existant** pour **Groupe de ressources**, puis sélectionnez le **groupe de ressources du labo**. 
4. Créez un partage de fichiers dans votre stockage Azure en suivant les instructions de l’article [Créer un partage de fichiers dans Azure Files](../storage/files/storage-how-to-create-file-share.md). 

## <a name="mount-the-file-share-on-your-local-machine"></a>Monter le partage de fichiers sur votre ordinateur local
1. Sur votre ordinateur local, utilisez le script de la section [Monter le partage de fichiers Azure](../storage/files/storage-how-to-use-files-windows.md#mount-the-azure-file-share) de l’article [Utiliser un partage de fichiers Azure avec Windows](../storage/files/storage-how-to-use-files-windows.md). 
2. Ensuite, utilisez la commande `net use` pour monter le partage de fichiers sur votre ordinateur. Voici l’exemple de commande : Spécifiez le nom de votre stockage Azure et le nom du partage de fichiers avant d’exécuter la commande. 

    `net use Z: \\<YOUR AZURE STORAGE NAME>.file.core.windows.net\<YOUR FILE SHARE NAME> /persistent:yes`

## <a name="create-a-vm-in-the-lab"></a>Créer une machine virtuelle dans le laboratoire
1. Dans la page **Partage de fichiers**, sélectionnez le **groupe de ressources** dans le menu de navigation en haut. Vous voyez la page **Groupe de ressources**. 
    
    ![Sélectionner le groupe de ressources dans le menu de navigation](media/test-app-in-azure/select-resource-group-bread-crump.png)
2. Dans la page **Groupe de ressources**, sélectionnez le **labo** que vous avez créé dans DevTest Labs.

    ![Sélectionner le laboratoire](media/test-app-in-azure/select-devtest-lab-in-resource-group.png)
3. Dans la page **DevTest Lab** pour votre labo, sélectionnez **+ Ajouter** dans la barre d’outils. 

    ![Bouton Ajouter pour le labo](media/test-app-in-azure/add-button-in-lab.png)
4. Dans la page **Choisir une base**, recherchez **smalldisk**, puis sélectionnez **[smalldisk] Centre de données Windows Server 2016**. 

    ![Choisir un petit disque Windows Server](media/test-app-in-azure/choose-small-disk-windows-server.png)
5. Dans la page **Machine virtuelle**, spécifiez un **nom de machine virtuelle**, un **nom d’utilisateur**, un **mot de passe**, puis sélectionnez **Créer** .    
    
    ![Page Créer une machine virtuelle](media/test-app-in-azure/create-virtual-machine-page.png)    

## <a name="mount-the-file-share-on-your-vm"></a>Monter le partage de fichiers sur votre machine virtuelle
1. Une fois la machine virtuelle créée, sélectionnez la **machine virtuelle** dans la liste.    

    ![Sélectionner la machine virtuelle de labo](media/test-app-in-azure/select-lab-vm.png)
2. Sélectionnez **Connexion** dans la barre d’outils pour vous connecter à la machine virtuelle. 
3. [Installez Azure PowerShell](/powershell/azure/install-az-ps).
4. Suivez les instructions de la section Monter le partage de fichiers. 

## <a name="publish-your-app-from-visual-studio"></a>Publier votre application à partir de Visual Studio
Dans cette section, vous publiez votre application à partir de Visual Studio sur un ordinateur virtuel de test dans le cloud.

1. Créez une application web/de bureau à l’aide de Visual Studio 2019.
2. Générez votre application.
3. Pour publier votre application, dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur votre projet, puis sélectionnez **Publier**. 
4. Dans l’**Assistant Publication**, entrez le **lecteur** mappé à votre partage de fichiers.

    **Application de bureau :**

    ![Application de bureau](media/test-app-in-azure/desktop-app.png)

    **Application web :**

    ![Application web](media/test-app-in-azure/web-app.png)

1. Sélectionnez **Suivant** pour achever le workflow de publication, puis choisissez **Terminer**. Lorsque vous avez terminé les étapes de l’Assistant, Visual Studio génère votre application et la publie sur votre partage de fichiers. 


## <a name="test-the-app-on-your-test-vm-in-the-lab"></a>Tester l’application sur votre ordinateur virtuel de test dans le labo

1. Accédez à la page de votre machine virtuelle dans le labo. 
2. Sélectionnez **Démarrer** dans la barre d’outils pour démarrer la machine virtuelle si elle est en état arrêté. Vous pouvez configurer des stratégies de démarrage et d’arrêt automatiques pour votre machine virtuelle, afin d’éviter de devoir la démarrer et l’arrêter à chaque fois. 
3. Sélectionnez **Connecter**.

    ![Page de machine virtuelle](media/test-app-in-azure/virtual-machine-page.png)
4. Dans la machine virtuelle, lancez l’**Explorateur de fichiers**, puis sélectionnez **Ce PC** pour trouver votre partage de fichiers.

    ![Trouver le partage sur la machine virtuelle](media/test-app-in-azure/find-share-on-vm.png)

    > [!NOTE]
    > Si, pour une raison quelconque, vous ne parvenez pas à trouver votre partage de fichiers sur votre machine virtuelle ou sur votre ordinateur local, vous pouvez le remonter en exécutant la commande `net use`. Vous pouvez trouver la commande `net use` dans l’Assistant **Connexion** de votre **Partage de fichiers** sur le portail Azure.
1. Ouvrez le partage de fichiers et vérifiez que l’application que vous avez déployée à partir de Visual Studio y figure. 

    ![Ouvrir le partage sur la machine virtuelle](media/test-app-in-azure/open-file-share.png)

    Vous pouvez maintenant accéder à votre application et la tester sur l’ordinateur virtuel de test que vous avez créé dans Azure.

## <a name="next-steps"></a>Étapes suivantes
Pour apprendre à utiliser des machines virtuelles dans un labo, voir les articles suivants. 

- [Ajouter une machine virtuelle à un laboratoire](devtest-lab-add-vm.md)
- [Redémarrer une machine virtuelle de laboratoire](devtest-lab-restart-vm.md)
- [Redimensionner une machine virtuelle de laboratoire](devtest-lab-resize-vm.md)
