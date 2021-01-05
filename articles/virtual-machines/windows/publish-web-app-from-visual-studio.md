---
title: Publier une application web sur une machine virtuelle Azure à partir de Visual Studio
description: Publier une application web ASP.NET sur une machine virtuelle Azure à partir de Visual Studio
author: ghogen
manager: jillfra
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: how-to
ms.date: 11/03/2017
ms.author: ghogen
ms.openlocfilehash: 7cf2474c9ca005d85cea28f2b8fa1e23836b191f
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96487556"
---
# <a name="publish-an-aspnet-web-app-to-an-azure-vm-from-visual-studio"></a>Publier une application web ASP.NET sur une machine virtuelle Azure à partir de Visual Studio

Ce document décrit comment publier une application web ASP.NET sur une machine virtuelle Azure avec la fonctionnalité de publication de **Machines virtuelles Microsoft Azure** dans Visual Studio 2019.  

## <a name="prerequisites"></a>Prérequis
Pour pouvoir publier un projet ASP.NET sur une machine virtuelle Azure à l’aide de Visual Studio, vous devez configurer la machine virtuelle correctement.

- Elle doit être configurée pour exécuter une application web ASP.NET et WebDeploy doit être installé. Pour plus d’informations, consultez [Créer une machine virtuelle ASP.NET avec WebDeploy](https://github.com/aspnet/Tooling/blob/AspNetVMs/docs/create-asp-net-vm-with-webdeploy.md).

- La machine virtuelle doit avoir un nom DNS configuré. Pour en savoir plus, consultez [Créer un nom de domaine complet dans le portail Azure pour une machine virtuelle Windows](../create-fqdn.md).

## <a name="publish-your-aspnet-web-app-to-the-azure-vm-using-visual-studio"></a>Publier votre application web ASP.NET sur la machine virtuelle Azure à l’aide de Visual Studio
La section suivante décrit comment publier une application web ASP.NET existante sur une machine virtuelle Azure.

1. Ouvrez votre solution d’application web dans Visual Studio 2019.
2. Cliquez avec le bouton droit sur le projet dans l’Explorateur de solutions, puis choisissez **Publier**.
3. Utilisez la flèche à droite de la page pour faire défiler les options de publication jusqu’à **Machines virtuelles Microsoft Azure**.  

   ![Page Publier - Flèche à droite]

4. Sélectionnez l’icône **Machines virtuelles Microsoft Azure**, puis sélectionnez **Publier**.

   ![Page Publier - Icône Machine virtuelle Microsoft Azure]

5. Choisissez le compte approprié (avec un abonnement Azure connecté à votre machine virtuelle).  
   - Si vous êtes connecté à Visual Studio, la liste des comptes est remplie avec tous vos comptes authentifiés.  
   - Si vous n’êtes pas connecté, ou si le compte dont vous avez besoin n’est pas répertorié, choisissez « Ajouter un compte » et suivez les invites pour vous connecter.  
   ![Sélecteur de compte Azure]  

6. Sélectionnez la machine virtuelle appropriée dans la liste des machines virtuelles existantes.

   > [!Note]
   > L’énumération de cette liste peut prendre un certain temps.

   ![Sélecteur de machine virtuelle Azure]

7. Cliquez sur OK pour commencer la publication.

8. Quand vous êtes invité à fournir les informations d’identification, spécifiez le nom d’utilisateur et le mot de passe d’un compte d’utilisateur sur la machine virtuelle cible, configuré avec des droits de publication. Il s’agit en général du nom d’utilisateur et du mot de passe d’administrateur utilisés lors de la création de la machine virtuelle.  

   ![Connexion à WebDeploy]

9. Acceptez le certificat de sécurité.

   ![Erreur de certificat]

10. Observez la fenêtre Sortie pour vérifier la progression de l’opération de publication.

    ![Fenêtre Sortie]

11. Si la publication réussit, un navigateur démarre et ouvre l’URL du site qui vient d’être publié.

**Vous avez réussi !**

Vous venez de publier votre application web sur une machine virtuelle Azure.

## <a name="publish-page-options"></a>Page Options de publication

Une fois l’Assistant Publication terminé, la page Publier est ouverte avec le nouveau profil de publication sélectionné.

### <a name="re-publish"></a>Republier

Pour publier des mises à jour de votre application web, sélectionnez le bouton **Publier** dans la page Publier.  
- Si vous y êtes invité, entrez un nom d’utilisateur et un mot de passe.  
- La publication commence immédiatement.

![Page Publier - Bouton Publier]

### <a name="modify-publish-profile-settings"></a>Modifier les paramètres de profil de publication

Pour afficher et modifier les paramètres de profil de publication, sélectionnez **Paramètres**.  

![Page Publier - Bouton Paramètres]

Vos paramètres doivent ressembler à ce qui suit :  

![Paramètres de publication - Page Connexion]

#### <a name="save-user-name-and-password"></a>Enregistrer le nom d’utilisateur et le mot de passe
- Évitez de fournir des informations d’authentification chaque fois que vous publiez. Pour cela, renseignez les champs **Nom d’utilisateur** et **Mot de passe**, puis sélectionnez la zone **Enregistrer le mot de passe**.
- Utilisez le bouton **Valider la connexion** pour confirmer que vous avez entré les informations correctes.

#### <a name="deploy-to-clean-web-server"></a>Déployer sur un nouveau serveur web

- Si vous voulez être sûr que le serveur web a une nouvelle copie de l’application web après chaque chargement et qu’il ne reste aucun autre fichier d’un déploiement précédent, vous pouvez cocher la case **Supprimer les fichiers supplémentaires de la destination** sous l’onglet **Paramètres**.

- Avertissement : La publication avec ce paramètre supprime tous les fichiers qui existent sur le serveur web (répertoire wwwroot). Assurez-vous de connaître l’état de la machine avant de publier avec cette option activée. 

![Paramètres de publication - Page Paramètres]

## <a name="next-steps"></a>Étapes suivantes

### <a name="set-up-cicd-for-automated-deployment-to-azure-vm"></a>Configurer l’intégration continue/la livraison continue pour le déploiement automatisé sur Machines virtuelles Azure

Pour configurer un pipeline de livraison continue avec Azure Pipelines, voir [Déployer sur une machine virtuelle Windows](/vsts/build-release/apps/cd/deploy-webdeploy-iis-deploygroups).

[VM Overview - DNS Name]: ../../../includes/media/publish-web-app-from-visual-studio/VMOverviewDNSName.png
[IP Address Config - DNS Name]: ../../../includes/media/publish-web-app-from-visual-studio/IPAddressConfigDNSName.png
[VM Overview - DNS Configured]: ../../../includes/media/publish-web-app-from-visual-studio/VMOverviewDNSConfigured.png
[Page Publier - Flèche à droite]: ../../../includes/media/publish-web-app-from-visual-studio/PublishPageRightArrow.png
[Page Publier - Icône Machine virtuelle Microsoft Azure]: ../../../includes/media/publish-web-app-from-visual-studio/PublishPageMicrosoftAzureVirtualMachineIcon.png
[Sélecteur de compte Azure]: ../../../includes/media/publish-web-app-from-visual-studio/ChooseVM-SelectAccount.png
[Sélecteur de machine virtuelle Azure]: ../../../includes/media/publish-web-app-from-visual-studio/ChooseVM-SelectVM.png
[Connexion à WebDeploy]: ../../../includes/media/publish-web-app-from-visual-studio/WebDeployLogin.png
[Erreur de certificat]: ../../../includes/media/publish-web-app-from-visual-studio/CertificateError.png
[Fenêtre Sortie]: ../../../includes/media/publish-web-app-from-visual-studio/OutputWindow.png
[Page Publier - Bouton Publier]: ../../../includes/media/publish-web-app-from-visual-studio/PublishPagePublishButton.png
[Page Publier - Bouton Paramètres]: ../../../includes/media/publish-web-app-from-visual-studio/PublishPageSettingsButton.png
[Paramètres de publication - Page Connexion]: ../../../includes/media/publish-web-app-from-visual-studio/PublishSettingsConnectionPage.png
[Paramètres de publication - Page Paramètres]: ../../../includes/media/publish-web-app-from-visual-studio/PublishSettingsSettingsPage.png