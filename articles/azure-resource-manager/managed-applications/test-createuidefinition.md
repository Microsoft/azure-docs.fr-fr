---
title: Tester le fichier de définition de l’interface utilisateur
description: Décrit comment tester l’expérience utilisateur de création de votre application gérée Azure via le portail.
author: tfitzmac
ms.topic: conceptual
ms.date: 08/06/2019
ms.author: tomfitz
ms.openlocfilehash: f76d3b81c2d5425f7bb91c5c86a79faa097794e3
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96434998"
---
# <a name="test-your-portal-interface-for-azure-managed-applications"></a>Tester votre interface de portail pour Applications managées Azure

Après la [création du fichier createUiDefinition.json](create-uidefinition-overview.md) pour votre application managée, vous devez tester l’expérience utilisateur. Pour simplifier le test, utilisez un environnement de bac à sable qui charge votre fichier dans le portail. Vous n’avez pas besoin de déployer votre application managée. Le bac à sable présente votre interface utilisateur dans l’expérience portail actuelle et en plein écran. Ou, vous pouvez utiliser un script pour tester l’interface. Les deux approches sont présentées dans cet article. Le bac à sable constitue la méthode recommandée pour afficher un aperçu de l’interface.

## <a name="prerequisites"></a>Conditions préalables requises

* Un fichier **createUiDefinition.json**. Si vous n’avez pas ce fichier, copiez l’[exemple de fichier](https://github.com/Azure/azure-quickstart-templates/blob/master/100-marketplace-sample/createUiDefinition.json).

* Un abonnement Azure. Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="use-sandbox"></a>Utiliser le bac à sable

1. Ouvrez [Créer un bac à sable (sandbox) de définition d’interface utilisateur](https://portal.azure.com/?feature.customPortal=false&#blade/Microsoft_Azure_CreateUIDef/SandboxBlade).

   ![Afficher le bac à sable](./media/test-createuidefinition/show-sandbox.png)

1. Remplacez la définition vide par le contenu de votre fichier createUiDefinition.json. Sélectionnez **Aperçu**.

   ![Sélectionner l’aperçu](./media/test-createuidefinition/select-preview.png)

1. Le formulaire que vous avez créé s’affiche. Vous pouvez parcourir l’expérience utilisateur et renseigner les valeurs.

   ![Afficher le formulaire](./media/test-createuidefinition/show-ui-form.png)

### <a name="troubleshooting"></a>Dépannage

Si votre formulaire ne s’affiche après la sélection de l’option **Aperçu**, cela signifie peut-être que vous avez fait une erreur de syntaxe. Recherchez l’indicateur rouge dans la barre de défilement sur la droite et accédez à ce dernier.

![Afficher une erreur de syntaxe](./media/test-createuidefinition/show-syntax-error.png)

Si votre formulaire ne s’affiche pas, et si à la place vous voyez une icône représentant un nuage avec une larme, cela signifie que votre formulaire comporte une erreur, comme une propriété manquante. Ouvrez Web Developer Tools dans votre navigateur. La **Console** affiche des messages importants sur votre interface.

![Afficher l’erreur](./media/test-createuidefinition/show-error.png)

## <a name="use-test-script"></a>Utiliser le script de test

Pour tester votre interface dans le portail, copiez un des scripts suivants sur votre ordinateur local :

* [Script PowerShell side-load - Module Az](https://github.com/Azure/azure-quickstart-templates/blob/master/SideLoad-AzCreateUIDefinition.ps1)
* [Script PowerShell side-load - Module Azure](https://github.com/Azure/azure-quickstart-templates/blob/master/SideLoad-CreateUIDefinition.ps1)
* [Script Azure CLI side-load](https://github.com/Azure/azure-quickstart-templates/blob/master/sideload-createuidef.sh)

Pour afficher votre fichier d’interface dans le portail, exécutez le script téléchargé. Le script crée un compte de stockage dans votre abonnement Azure et charge votre fichier createUiDefinition.json dans le compte de stockage. Le compte de stockage est créé quand vous exécutez le script pour la première fois, ou si le compte de stockage a été supprimé. Si le compte de stockage existe déjà dans votre abonnement Azure, le script le réutilise. Le script ouvre le portail et charge votre fichier à partir du compte de stockage.

Indiquez un emplacement pour le compte de stockage et spécifiez le dossier contenant votre fichier createUiDefinition.json.

Pour PowerShell, utilisez la commande suivante :

```powershell
.\SideLoad-AzCreateUIDefinition.ps1 `
  -StorageResourceGroupLocation southcentralus `
  -ArtifactsStagingDirectory .\100-Marketplace-Sample
```

Pour l’interface de ligne de commande Azure, consultez :

```bash
./sideload-createuidef.sh \
  -l southcentralus \
  -a .\100-Marketplace-Sample
```

Si votre fichier createUiDefinition.json se trouve dans le même dossier que le script et que vous avez déjà créé le compte de stockage, vous n’avez pas besoin de fournir ces paramètres.

Pour PowerShell, utilisez la commande suivante :

```powershell
.\SideLoad-AzCreateUIDefinition.ps1
```

Pour l’interface de ligne de commande Azure, consultez :

```bash
./sideload-createuidef.sh
```

Le script ouvre un nouvel onglet dans votre navigateur. Il affiche le portail avec votre interface de création de l’application managée.

Donnez des valeurs aux champs. Quand vous avez terminé, vous voyez les valeurs qui sont passées au modèle se trouvant dans la console des outils de développement de votre navigateur.

![Afficher les valeurs](./media/test-createuidefinition/show-json.png)

Vous pouvez utiliser ces valeurs en tant que fichier de paramètres pour tester votre modèle de déploiement.

Si le portail se bloque sur l’écran récapitulatif, cela signifie qu’il y a peut-être un bogue dans la section de sortie. Par exemple, vous avez référencé un contrôle inexistant. Si un paramètre de la sortie est vide, ce paramètre peut faire référence à une propriété inexistante. Par exemple, la référence au contrôle est valide, mais pas la référence à la propriété.

## <a name="test-your-solution-files"></a>Tester vos fichiers de solution

Maintenant que vous avez vérifié que votre interface de portail fonctionne comme prévu, il est temps de vérifier que votre fichier createUiDefinition est correctement intégré à votre fichier mainTemplate.json. Vous pouvez exécuter un script de validation test pour tester le contenu de vos fichiers de solution, y compris le fichier createUiDefinition. Le script valide la syntaxe JSON, vérifie les expressions d’expression régulière sur les champs de texte et permet de s’assurer que les valeurs de sortie de l’interface du portail correspondent aux paramètres de votre modèle. Pour plus d’informations sur l’exécution de ce script, consultez [Run static validation checks for templates](https://github.com/Azure/azure-quickstart-templates/tree/master/test) (Effectuer les vérifications de validation statiques des modèles).

## <a name="next-steps"></a>Étapes suivantes

Après avoir validé votre interface du portail, découvrez comment [rendre disponible votre application gérée Azure dans la Place de marché](../../marketplace/create-new-azure-apps-offer.md).