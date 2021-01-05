---
title: Installation du runtime d’Azure Functions
description: Installation de la préversion 2 du runtime Azure Functions
author: apwestgarth
ms.topic: conceptual
ms.date: 11/28/2017
ms.author: anwestg
ms.openlocfilehash: 73b9533fa00c783e1cfb85270198f5c00c66afd5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86540381"
---
# <a name="install-the-azure-functions-runtime-preview-2"></a>Installer la préversion 2 du runtime Azure Functions

[!INCLUDE [intro](../../includes/functions-runtime-preview-note.md)]

Si vous souhaitez installer la préversion 2 du runtime d’Azure Functions, suivez ces étapes :

1. Vérifiez que votre ordinateur respecte la configuration minimale requise.
1. Téléchargez le [programme d’installation de la version préliminaire du runtime d’Azure Functions](https://aka.ms/azafrv2).
1. Désinstallez la préversion 1 du runtime Azure Functions.
1. Installez la préversion 2 du runtime Azure Functions.
1. Terminez la configuration de la préversion 2 du runtime Azure Functions.
1. Créer votre première fonction dans la préversion du runtime Azure Functions

## <a name="prerequisites"></a>Prérequis

Avant d’installer la version préliminaire du runtime d’Azure Functions, vous devez disposer des ressources suivantes :

1. Un ordinateur exécutant Microsoft Windows Server 2016 ou Windows 10 Creators Update (édition Professionnelle ou Entreprise).
1. Une instance SQL Server en cours d’exécution au sein de votre réseau.  La version minimale requise est SQL Server Express.

## <a name="uninstall-previous-version"></a>Désinstaller la version précédente

Si vous avez déjà installé la préversion du runtime Azure Functions, vous devez la désinstaller avant d'installer la dernière version.  Désinstallez la préversion du runtime Azure Functions en supprimant le programme dans le menu Ajout/Suppression de programmes de Windows.

## <a name="install-the-azure-functions-runtime-preview"></a>Installer la version préliminaire du runtime d’Azure Functions

Le programme d’installation de la préversion du runtime Azure Functions vous guide lors de l’installation de ses rôles de gestion et de travail.  Il est possible d’installer le rôle de gestion et de travail sur le même ordinateur.  Toutefois, si vous ajoutez d'autres applications de fonction, vous devez déployer davantage de rôles de travail sur des ordinateurs supplémentaires pour faire évoluer vos fonctions sur plusieurs rôles de travail.

## <a name="install-the-management-and-worker-role-on-the-same-machine"></a>Installer le rôle de gestion et de travail sur le même ordinateur

1. Exécutez le programme d’installation de la version préliminaire du runtime d’Azure Functions.

    ![Programme d’installation de la préversion du runtime d’Azure Functions][1]

1. Cliquez sur **Suivant**.
1. Une fois que vous avez lu les conditions d’utilisation du **CLUF**, **cochez la case** pour accepter les conditions d’utilisation et cliquez sur **Suivant** pour passer à l’étape suivante.
1. Sélectionnez les rôles que vous souhaitez installer sur cet ordinateur **Rôle de gestion Functions** et/ou **Rôle de travail Functions** et cliquez sur **Suivant**.

    ![Programme d’installation de la préversion du runtime d’Azure Functions - sélection du rôle][3]

    > [!NOTE]
    > Vous pouvez installer le **rôle de travail Functions** sur de nombreuses autres machines. Pour cela, suivez ces instructions, puis sélectionnez uniquement **Rôle de travail Functions** dans le programme d’installation.

1. Cliquez sur **Suivant** pour que **l’Assistant Installation du runtime Azure Functions** démarre le processus d’installation sur votre ordinateur.
1. Une fois terminé, l’Assistant Installation lance l’outil de configuration du **runtime Azure Functions**.

    ![Programme d’installation de la préversion du runtime d’Azure Functions terminé][6]

    > [!NOTE]
    > Si vous effectuez l’installation sur **Windows 10** et que la fonctionnalité **Conteneur** n’avait pas été activée, le **programme d’installation du runtime Azure Functions** vous invite à redémarrer votre ordinateur pour terminer l’installation.

## <a name="configure-the-azure-functions-runtime"></a>Configurer le runtime d’Azure Functions

Pour terminer l’installation du runtime d’Azure Functions, vous devez terminer la configuration.

1. L’outil de configuration du **runtime Azure Functions** vous indique les rôles installés sur votre ordinateur.

    ![Outil de configuration de la préversion du runtime d’Azure Functions terminé][7]

1. Cliquez sur l’onglet **Base de données**, entrez les informations de connexion de votre instance SQL Server, en spécifiant notamment une [clé principale de base de données](/sql/relational-databases/security/encryption/sql-server-and-database-encryption-keys-database-engine), puis cliquez sur **Appliquer**.  Une connectivité à une instance SQL Server est nécessaire pour que le runtime Azure Functions crée une base de données afin de prendre en charge le runtime.

    ![Configuration de la base de données de la préversion du runtime d’Azure Functions][8]

1. Cliquez sur l’onglet **Informations d’identification** .  Ici, vous devez créer deux nouveaux jeux d’informations d’identification à utiliser avec un partage de fichiers pour l’hébergement de toutes vos applications de fonction.  Spécifiez les combinaisons de **nom d’utilisateur** et de **mot de passe** du **propriétaire du partage de fichiers** et de l’**utilisateur du partage de fichiers**, puis cliquez sur **Appliquer**.

    ![Informations d’identification de la préversion du runtime d’Azure Functions][9]

1. Cliquez sur l’onglet **Partage de fichiers**.  Vous devez ici spécifier les détails de l’emplacement du partage de fichiers.  Le partage de fichiers peut être créé pour vous ou vous pouvez utiliser un partage de fichier existant, puis cliquez sur **Appliquer**.  Si vous sélectionnez un nouvel emplacement de partage de fichiers, vous devez spécifier un répertoire que le runtime d’Azure Functions doit utiliser.

    ![Partage de fichiers de la préversion du runtime d’Azure Functions][10]

1. Cliquez sur l’onglet **IIS**.  Cet onglet affiche les détails des sites web dans IIS, que l’outil de configuration du runtime Azure Functions crée.  Vous pouvez spécifier ici un nom DNS personnalisé pour le portail de la préversion du runtime Azure Functions.  Cliquez sur **Appliquer** pour terminer.

    ![IIS de la préversion du runtime d’Azure Functions][11]

1. Cliquez sur l’onglet **Services**.  Cet onglet affiche l’état des services dans votre outil de configuration du runtime Azure Functions.  Si le **service d’activation hôte Azure Functions** ne fonctionne pas après la configuration initiale, cliquez sur **Démarrer le service**.

    ![Configuration de la préversion du runtime d’Azure Functions terminée][12]

1. Accédez au **portail du runtime Azure Functions** en tant que `https://<machinename>.<domain>/`.

    ![Portail du runtime d’Azure Functions en préversion][13]

## <a name="create-your-first-function-in-azure-functions-runtime-preview"></a>Créer votre première fonction dans la préversion du runtime Azure Functions

Pour créer votre première fonction dans la préversion du runtime Azure Functions

1. Accédez au **portail du runtime d’Azure Functions**, dont l’adresse est au format `https://<machinename>.<domain>` (par exemple, `https://mycomputer.mydomain.com`).

1. Vous êtes invité à vous **connecter**. Avec un déploiement dans un domaine, utilisez le nom d'utilisateur et le mot de passe de votre compte de domaine. Sinon, utilisez le nom d'utilisateur et le mot de passe de votre compte local pour vous connecter au portail.

    ![Connexion au portail de la préversion du runtime Azure Functions][14]

1. Pour créer des applications de fonction, vous devez créer un abonnement.  Dans l’angle supérieur gauche du portail, cliquez sur l’option **+** en regard des abonnements.

    ![Abonnements du portail de la préversion du runtime Azure Functions][15]

1. Choisissez **DefaultPlan**, entrez un nom pour votre abonnement, puis cliquez sur **Créer**.

    ![Plan et nom de l'abonnement du portail de la préversion du runtime Azure Functions][16]

1. Toutes vos applications de fonction sont répertoriées dans le volet gauche du portail.  Pour créer une application de fonction, sélectionnez l'en-tête **Applications de fonction**, puis cliquez sur l'option **+** .

1. Entrez un nom pour votre application de fonction, sélectionnez l'abonnement approprié, choisissez la version du runtime Azure Functions pour la programmation, puis cliquez sur **Créer**

    ![Nouvelle application de fonction du portail de la préversion du runtime Azure Functions][17]

1. Votre nouvelle application de fonction est répertoriée dans le volet gauche du portail.  Sélectionnez Fonctions, puis cliquez sur **Nouvelle fonction** en haut du volet central du portail.

    ![Modèles de préversion du runtime Azure Functions][18]

1. Sélectionnez la fonction Déclencheur de minuteur, nommez votre fonction dans le menu déroulant de droite, définissez le calendrier sur `*/5 * * * * *` (cette expression cron déclenche le minuteur toutes les cinq secondes), puis cliquez sur **Créer**

    ![Configuration d'une nouvelle fonction de minuteur pour la préversion du runtime Azure Functions][19]

1. Votre fonction est maintenant créée.  Vous pouvez afficher le journal d'exécution de votre application de fonction en développant le volet du **journal** au bas du portail.

    ![Exécution de la fonction de la préversion du runtime Azure Functions][20]

<!--Image references-->
[1]: ./media/functions-runtime-install/AzureFunctionsRuntime_Installer1.png
[2]: ./media/functions-runtime-install/AzureFunctionsRuntime_Installer2-EULA.png
[3]: ./media/functions-runtime-install/AzureFunctionsRuntime_Installer3-ChooseRoles.png
[4]: ./media/functions-runtime-install/AzureFunctionsRuntime_Installer4-Install.png
[5]: ./media/functions-runtime-install/AzureFunctionsRuntime_Installer5-Progress.png
[6]: ./media/functions-runtime-install/AzureFunctionsRuntime_Installer6-InstallComplete.png
[7]: ./media/functions-runtime-install/AzureFunctionsRuntime_Configuration1.png
[8]: ./media/functions-runtime-install/AzureFunctionsRuntime_Configuration2_SQL.png
[9]: ./media/functions-runtime-install/AzureFunctionsRuntime_Configuration3_Credentials.png
[10]: ./media/functions-runtime-install/AzureFunctionsRuntime_Configuration4_Fileshare.png
[11]: ./media/functions-runtime-install/AzureFunctionsRuntime_Configuration5_IIS.png
[12]: ./media/functions-runtime-install/AzureFunctionsRuntime_Configuration6_Services.png
[13]: ./media/functions-runtime-install/AzureFunctionsRuntime_Portal.png
[14]: ./media/functions-runtime-install/AzureFunctionsRuntime_Portal_Login.png
[15]: ./media/functions-runtime-install/AzureFunctionsRuntime_Portal_Subscriptions.png
[16]: ./media/functions-runtime-install/AzureFunctionsRuntime_Portal_Subscriptions1.png
[17]: ./media/functions-runtime-install/AzureFunctionsRuntime_Portal_NewFunctionApp.png
[18]: ./media/functions-runtime-install/AzureFunctionsRuntime_v1FunctionsTemplates.png
[19]: ./media/functions-runtime-install/AzureFunctionsRuntime_Portal_NewTimerFunction.png
[20]: ./media/functions-runtime-install/AzureFunctionsRuntime_Portal_RunningV2Function.png
