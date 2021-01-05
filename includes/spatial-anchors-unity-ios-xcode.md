---
author: msftradford
ms.service: azure-spatial-anchors
ms.topic: include
ms.date: 11/20/2020
ms.author: parkerra
ms.openlocfilehash: d969b1639dbb39b920c985964abe1239f068d740
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185265"
---
Sélectionnez **Build**. Dans le volet qui s’ouvre, sélectionnez un dossier vers lequel exporter le projet Xcode.

   À l’issue de l’exportation, un dossier contenant le projet Xcode exporté s’affiche.

   > [!NOTE]
   > Si une fenêtre affiche un message vous demandant de choisir entre remplacer ou ajouter, nous vous recommandons de sélectionner **Ajouter**, car c’est plus rapide. Vous devez sélectionner **Remplacer** seulement si vous modifiez des ressources dans votre scène. Par exemple, vous pouvez ajouter, supprimer ou modifier des relations parent/enfant ou des propriétés. Si vous changez seulement le code source, **Append** doit suffire.

## <a name="open-the-xcode-project"></a>Ouvrir le projet Xcode

Vous pouvez maintenant ouvrir votre projet `Unity-iPhone.xcodeproj` dans Xcode. 

Vous pouvez soit lancer Xcode et ouvrir le projet `Unity-iPhone.xcodeproj` exporté, soit lancer le projet dans Xcode en exécutant la commande suivante à partir de l’emplacement vers lequel vous avez exporté le projet :

 ```bash
open ./Unity-iPhone.xcodeproj
```

Sélectionnez le nœud racine **Unity-iPhone** pour voir les paramètres du projet, puis sélectionnez l’onglet **General**.

Sous **Signing** (Signature), vérifiez que l’option **Automatically manage signing** (Gérer automatiquement la signature) est activée. Si ce n’est pas le cas, activez-la, puis réinitialisez les paramètres de génération en sélectionnant **Enable Automatic** (Activer l’option automatique) dans le volet qui s’affiche.

Sous **Deployment Info** (Informations sur le déploiement), assurez-vous que l’option **Deployment Target** (Cible de déploiement) est définie sur **11.0**.

## <a name="deploy-the-app-to-your-ios-device"></a>Déployer l’application sur votre appareil iOS

Connectez l’appareil iOS au Mac et définissez le **schéma actif** sur votre appareil iOS.

   ![Capture d’écran du bouton Mon iPhone pour la sélection de l’appareil.](./media/spatial-anchors-unity/select-device.png)

Sélectionnez **Build and then run the current scheme**.

   ![Capture d’écran du bouton fléché « Déployer et exécuter ».](./media/spatial-anchors-unity/deploy-run.png)
