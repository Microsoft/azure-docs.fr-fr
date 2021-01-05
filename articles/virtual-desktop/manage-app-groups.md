---
title: Gérer des groupes d’applications Windows Virtual Desktop - Portail Azure
description: Explique comment gérer des groupes d’applications Windows Virtual Desktop avec le portail Azure.
author: Heidilohr
ms.topic: tutorial
ms.date: 10/09/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: ec26f021ffa581b0713973904c97349df83a08ef
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91930269"
---
# <a name="tutorial-manage-app-groups-with-the-azure-portal"></a>Tutoriel : Gérer des groupes d’applications avec le portail Azure

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop avec des objets Windows Virtual Desktop Azure Resource Manager. Si vous utilisez la version Windows Virtual Desktop (classique) sans objets Azure Resource Manager, consultez [cet article](./virtual-desktop-fall-2019/manage-app-groups-2019.md).

Le groupe d’applications par défaut créé pour un nouveau pool d’hôtes Windows Virtual Desktop publie également l’intégralité du bureau. De plus, vous pouvez créer un ou plusieurs groupes d’applications RemoteApp pour le pool d’hôtes. Suivez ce tutoriel pour créer un groupe d’applications RemoteApp et publier des applications individuelles du menu Démarrer.

Dans ce tutoriel, vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer un groupe RemoteApp.
> * Accorder l’accès aux programmes RemoteApp.

## <a name="create-a-remoteapp-group"></a>Créer un groupe RemoteApp

Si vous avez déjà créé un pool d’hôtes et des machines virtuelles hôtes de session à l’aide du portail Azure ou de PowerShell, vous pouvez ajouter des groupes d’applications à partir du portail Azure en appliquant la procédure suivante :

1.  Connectez-vous au [portail Azure](https://portal.azure.com/).
   
    >[!NOTE]
    > Si vous vous connectez au portail US Gov, accédez à [https://portal.azure.us/](https://portal.azure.us/) à la place.

2.  Recherchez et sélectionnez **Windows Virtual Desktop**.

3. Vous pouvez ajouter un groupe d’applications directement ou vous pouvez l’ajouter à partir d’un pool d’hôtes existant. Choisissez l’une des options suivantes :

    - Sélectionnez **Groupes d’applications** dans le menu situé à gauche dans la page, puis sélectionnez **+ Ajouter**.

    - Sélectionnez **Pools d’hôtes** dans le menu situé à gauche sur l’écran et sélectionnez successivement le nom du pool d’hôtes, **Groupes d’applications** dans le menu situé à gauche, puis **+ Ajouter**. Dans ce cas, le pool d’hôtes est déjà sélectionné sous l’onglet De base.

4. Sous l’onglet **De base**, sélectionnez l’**abonnement** et le **groupe de ressources** pour lesquels vous souhaitez créer le groupe d’applications. Vous pouvez également choisir de créer un nouveau groupe de ressources au lieu de sélectionner un groupe existant.

5. Sélectionnez le **pool d’hôtes** qui sera associé au groupe d’applications dans le menu déroulant.

    >[!NOTE]
    >Vous devez sélectionner le pool d’hôtes associé au groupe d’applications. Les groupes d’applications ont des applications ou des bureaux qui sont servis par un hôte de session, et les hôtes de session font partie des pools d’hôtes. Le groupe d’applications doit être associé à un pool d’hôtes lors de sa création.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de l’onglet De base dans le portail Azure.](media/basics-tab.png)

6. Sélectionnez **RemoteApp** sous **Type de groupe d’applications**, puis entrez un nom pour votre RemoteApp.

      > [!div class="mx-imgBorder"]
      > ![Capture d’écran des champs de type de groupe d’applications. « RemoteApp » est mis en surbrillance.](media/remoteapp-button.png)

7.  Sélectionnez **Suivant : Attributions >** (onglet).

8.  Pour affecter des utilisateurs individuels ou des groupes d’utilisateurs au groupe d’applications, sélectionnez **+ Ajouter des utilisateurs ou des groupes d’utilisateurs Azure AD**.

9.  Sélectionnez les utilisateurs qui doivent avoir accès aux applications. Vous pouvez sélectionner un ou plusieurs utilisateurs et groupes d’utilisateurs.

     > [!div class="mx-imgBorder"]
     > ![Capture d’écran du menu de sélection d’utilisateur.](media/select-users.png)

10.  Sélectionnez **Sélectionner**.

11.  Sélectionnez **Suivant : Applications >** , puis sélectionnez **+Ajouter des applications**.

12.  Pour ajouter une application à partir du menu Démarrer :

      - Sous **Source de l’application**, sélectionnez **Menu Démarrer** dans le menu déroulant. Ensuite, sous **Application**, choisissez l’application dans le menu déroulant.

     > [!div class="mx-imgBorder"]
     > ![Capture d’écran de l’écran Ajouter une application avec le menu Démarrer sélectionné.](media/add-app-start.png)

      - Dans **Nom complet**, entrez le nom de l’application qui sera présentée à l’utilisateur sur son client.

      - Laissez les autres options telles quelles et sélectionnez **Enregistrer**.

13.  Pour ajouter une application à partir d’un chemin de fichier spécifique :

      - Sous **Source de l’application**, sélectionnez **Chemin de fichier** dans le menu déroulant.

      - Dans **Chemin de l’application**, entrez le chemin de l’application sur l’hôte de session inscrit auprès du pool d’hôtes associé.

      - Entrez les détails de l’application dans les champs **Nom de l’application**, **Nom complet**, **Chemin d’accès à l’icône** et **Index de l’icône**.

      - Sélectionnez **Enregistrer**.

     > [!div class="mx-imgBorder"]
     > ![Capture d’écran de la page Ajouter une application avec le chemin de fichier sélectionné.](media/add-app-file.png)

14.  Répétez ce processus pour chaque application que vous souhaitez ajouter au groupe d’applications.

15.  Ensuite, sélectionnez **Suivant : Espace de travail >** .

16.  Si vous souhaitez inscrire le groupe d’applications auprès d’un espace de travail, sélectionnez **Oui** pour **Inscrire le groupe d’applications**. Si vous préférez inscrire le groupe d’applications ultérieurement, sélectionnez **Non**.

17.  Si vous sélectionnez **Oui**, vous pouvez sélectionner un espace de travail existant auprès duquel inscrire votre groupe d’applications.

       >[!NOTE]
       >Vous ne pouvez inscrire le groupe d’applications qu’auprès d’un espace de travail créé au même emplacement que le pool d’hôtes. De plus, si vous avez déjà inscrit un autre groupe d’applications du même pool d’hôtes en tant que nouveau groupe d’applications auprès d’un espace de travail, il sera sélectionné et vous ne pourrez pas le modifier. Tous les groupes d’applications d’un pool d’hôtes doivent être inscrits auprès du même espace de travail.

     > [!div class="mx-imgBorder"]
     > ![Capture d’écran de la page Inscrire le groupe d’applications pour un espace de travail existant. Le pool d’hôtes est présélectionné.](media/register-existing.png)

18.  Le cas échéant, si vous souhaitez créer des étiquettes pour faciliter l’organisation de votre espace de travail, sélectionnez l’onglet **Suivant : Étiquettes >** et entrez les noms de vos étiquettes.

19.  Quand vous avez terminé, sélectionnez **Vérifier + créer**.

20.  Attendez que le processus de validation soit terminé. Ensuite, sélectionnez **Créer** pour déployer votre groupe d’applications.

Le processus de déploiement effectue les tâches suivantes pour vous :

- Création du groupe d’applications RemoteApp
- Ajout des applications sélectionnées au groupe d’applications
- Publication du groupe d’applications publié sur les utilisateurs et les groupes d’utilisateurs que vous avez sélectionnés
- Inscription du groupe d’applications, si vous avez choisi de le faire
- Création d’un lien vers un modèle Azure Resource Manager en fonction de votre configuration, que vous pouvez télécharger et enregistrer pour plus tard

>[!IMPORTANT]
>Vous pouvez seulement créer 200 groupes d’applications pour chaque locataire Azure Active Directory. Nous avons ajouté cette limite en raison des limitations du service de récupération des flux de nos utilisateurs. Cette limite ne s’applique pas aux groupes d’applications créés dans Windows Virtual Desktop (classique).

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à créer des groupes d’applications, à les remplir avec des programmes RemoteApp et à y affecter des utilisateurs. Pour savoir comment créer un pool d’hôtes de validation, consultez le didacticiel suivant. Vous pouvez utiliser un pool d’hôtes de validation pour surveiller les mises à jour de service avant de les déployer dans votre environnement de production.

> [!div class="nextstepaction"]
> [Créer un pool d’hôtes pour valider les mises à jour de service](./create-validation-host-pool.md)
