---
title: Ajouter un dépôt Git à un laboratoire dans Azure DevTest Labs | Microsoft Docs
description: Découvrez comment ajouter un dépôt GitHub ou un Git Azure DevOps Services pour votre source d’artefacts personnalisés dans Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 225aae5e0a017b711f29a47829b06f8bb7a6a8ee
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85483140"
---
# <a name="add-a-git-repository-to-store-custom-artifacts-and-resource-manager-templates"></a>Ajouter un dépôt Git pour stocker des artefacts personnalisés et des modèles Resource Manager

Vous pouvez [créer des artefacts personnalisés](devtest-lab-artifact-author.md) pour les machines virtuelles dans votre laboratoire ou [utiliser des modèles Azure Resource Manager pour créer un environnement de test personnalisé](devtest-lab-create-environment-from-arm.md). Vous devez ajouter un dépôt Git privé pour les artefacts ou les modèles Resource Manager que votre équipe crée. Le référentiel peut être hébergé sur [GitHub](https://github.com) ou sur [Azure DevOps Services](https://visualstudio.com).

Nous mettons à votre disposition un [dépôt GitHub d’artefacts](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts) que vous pouvez déployer tel quel, ou vous pouvez les personnaliser pour vos laboratoires. Quand vous personnalisez ou créez un artefact, vous ne pouvez pas stocker l’artefact dans le dépôt public. Vous devez créer votre propre dépôt privé pour les artefacts personnalisés et pour les artefacts que vous créez. 

Quand vous créez une machine virtuelle, vous pouvez enregistrer le modèle Resource Manager, le personnaliser éventuellement, puis l’utiliser pour créer des machines virtuelles supplémentaires. Vous devez créer votre propre dépôt privé afin de stocker vos modèles Resource Manager personnalisés.  

* Pour découvrir comment créer un dépôt GitHub, consultez [GitHub Bootcamp](https://help.github.com/categories/bootcamp/).
* Pour savoir comment créer un projet Azure DevOps Services qui a un dépôt Git, consultez [Se connecter à Azure DevOps Services](https://www.visualstudio.com/get-started/setup/connect-to-visual-studio-online).

La figure suivante illustre un dépôt contenant des artefacts dans GitHub :  

![Exemple de dépôt d’artefacts GitHub](./media/devtest-lab-add-repo/devtestlab-github-artifact-repo-home.png)

## <a name="get-the-repository-information-and-credentials"></a>Obtention des informations de référentiel et des informations d’identification
Pour ajouter un dépôt à votre laboratoire, obtenez d’abord des informations essentielles de votre dépôt. Les sections suivantes expliquent comment obtenir les informations requises pour les dépôts qui sont hébergés sur GitHub ou Azure DevOps Services.

### <a name="get-the-github-repository-clone-url-and-personal-access-token"></a>Obtenir l’URL de clonage du dépôt GitHub et le jeton d’accès personnel

1. Accédez à la page d’accueil du dépôt GitHub contenant l’artefact ou les définitions de modèle Resource Manager.
2. Sélectionnez **Cloner ou télécharger**.
3. Pour copier l’URL dans le Presse-papiers, sélectionnez le bouton **URL de clonage HTTPS**. Enregistrez l’URL pour une utilisation ultérieure.
4. Dans le coin supérieur droit de GitHub, Sélectionnez l’image de profil, puis sélectionnez **Paramètres**.
5. Dans le menu **Paramètres personnels** situé à gauche, sélectionnez **Jetons d’accès personnels**.
6. Sélectionnez **Générer un nouveau jeton**.
7. Dans la page **Nouveau jeton d’accès personnel**, sous **Description du jeton**, entrez une description. Accepter les éléments par défaut sous **Sélectionner des étendues**, puis sélectionnez **Générer un jeton**.
8. Enregistrez le jeton généré. Vous l’utiliserez ultérieurement.
9. Fermez GitHub.   
10. Passez à la section [Connecter votre laboratoire au référentiel](#connect-your-lab-to-the-repository).

### <a name="get-the-azure-repos-clone-url-and-personal-access-token"></a>Obtenir l’URL du clone Azure Repos et le jeton d’accès personnel

1. Accédez à la page d’accueil de votre collection d’équipe (par exemple, `https://contoso-web-team.visualstudio.com`), puis sélectionnez votre projet.
2. Sur la page d’accueil du projet, sélectionnez **Code**.
3. Pour afficher l’URL de clonage, sélectionnez dans la page **Code** du projet l’option **Cloner**.
4. Enregistrez l’URL. Vous l’utiliserez ultérieurement.
5. Pour créer un jeton d’accès personnel, dans le menu déroulant du compte d’utilisateur, sélectionnez **Mon profil**.
6. Sur la page d’informations du profil, sélectionnez **Sécurité**.
7. Sous l’onglet **Sécurité**, sélectionnez **Ajouter**.
8. Dans la page **Créer un jeton d’accès personnel** :
   1. Entrez une **Description** pour le jeton.
   2. Dans la liste **Expire dans**, sélectionnez **180 jours**.
   3. Dans la liste **Comptes**, sélectionnez **Tous les comptes accessibles**.
   4. Sélectionnez l’option **Lecture seule**.
   5. Sélectionnez **Créer le jeton**.
9. Le nouveau jeton apparaît dans la liste **Jetons d’accès personnels**. Sélectionnez **Copier le jeton**puis enregistrez la valeur du jeton pour utilisation ultérieure.
10. Passez à la section [Connecter votre laboratoire au référentiel](#connect-your-lab-to-the-repository).

## <a name="connect-your-lab-to-the-repository"></a>Connecter votre laboratoire au référentiel
1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
2. Sélectionnez **Autres services**, puis **DevTest Labs** dans la liste de services.
3. Dans la liste de laboratoires, sélectionnez votre laboratoire. 
4. Sélectionnez **Configuration et stratégies** > **Dépôts** >  **+ Ajouter**.

    ![Bouton pour ajouter un dépôt](./media/devtest-lab-add-repo/devtestlab-add-repo.png)
5. Sur la seconde page **Référentiels**, spécifiez les informations suivantes :
   1. **Nom**. Entrez un nom pour le dépôt.
   2. **URL Git Clone**. Entrez l’URL de clonage Git HTTPS que vous avez précédemment copiée à partir de GitHub ou d’Azure DevOps Services.
   3. **Branche**. Pour obtenir vos définitions, entrez la branche.
   4. **Jeton d’accès personnel**. Entrez le jeton d’accès personnel que vous avez obtenu à partir de GitHub ou Azure DevOps Services.
   5. **Chemins de dossiers**. Entrez au moins un chemin de dossier relatif à l’URL du clone contenant votre artefact ou vos définitions de modèle Resource Manager. Quand vous spécifiez un sous-répertoire, veillez à inclure la barre oblique dans le chemin du dossier.

      ![Zone Référentiels](./media/devtest-lab-add-repo/devtestlab-repo-blade.png)
6. Sélectionnez **Enregistrer**.

### <a name="related-blog-posts"></a>Billets de blog connexes
* [Résoudre les échecs des artefacts dans DevTest Labs](devtest-lab-troubleshoot-artifact-failure.md)
* [Joindre une machine virtuelle à un domaine Active Directory existant à l’aide d’un modèle Resource Manager dans DevTest Labs](https://www.visualstudiogeeks.com/blog/DevOps/Join-a-VM-to-existing-AD-domain-using-ARM-template-AzureDevTestLabs)

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>Étapes suivantes
Après avoir créé votre référentiel Git privé, vous pouvez effectuer au moins l’une des opérations suivantes, selon vos besoins :
* Stocker vos [artefacts personnalisés](devtest-lab-artifact-author.md). Vous pouvez les utiliser ultérieurement pour créer des machines virtuelles.
* [Créer des environnements de plusieurs machines virtuelles et des ressources PaaS à l’aide des modèles Resource Manager](devtest-lab-create-environment-from-arm.md). Ensuite, vous pouvez stocker les modèles dans votre dépôt privé.

Lorsque vous créez une machine virtuelle, vous pouvez vérifier que les artefacts ou les modèles sont ajoutés à votre référentiel Git. Ils sont immédiatement disponibles dans la liste des modèles ou des artefacts. Le nom de votre dépôt privé est affiché dans la colonne qui spécifie la source. 
