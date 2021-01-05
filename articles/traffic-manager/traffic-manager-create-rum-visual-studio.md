---
title: Mesures utilisateur réelles avec Visual Studio Mobile Center – Azure Traffic Manager
description: Configurez votre application mobile développée à l’aide de Visual Studio Mobile Center pour envoyer des mesures d’utilisateurs réels à Traffic Manager.
services: traffic-manager
documentationcenter: traffic-manager
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/16/2018
ms.author: duau
ms.custom: devx-track-js
ms.openlocfilehash: c49672ad38ac8cf80214d03870b5876d741e76ec
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994874"
---
# <a name="how-to-send-real-user-measurements-to-traffic-manager-with-visual-studio-mobile-center"></a>Comment envoyer les mesures des utilisateurs réels à Traffic Manager avec Visual Studio Mobile Center

Vous pouvez configurer votre application mobile développée à l’aide de Visual Studio Mobile Center pour envoyer des mesures d’utilisateurs réels à Traffic Manager en suivant les étapes ci-dessous :

>[!NOTE]
> Pour le moment, l’envoi des mesures d’utilisateurs réels à Traffic Manager est uniquement pris en charge sur les appareils Android.

Pour configurer les mesures d’utilisateurs réels, vous devez obtenir une clé et instrumenter votre application avec le package de mesures des utilisateurs réels.

## <a name="step-1-obtain-a-key"></a>Étape 1 : Obtenir une clé
    
Les mesures que vous prenez et envoyez à Traffic Manager à partir de votre application cliente sont identifiées par le service à l’aide d’une chaîne unique, que l’on nomme « clé de mesures des utilisateurs réels ». Vous pouvez obtenir une clé de mesures des utilisateurs réels via le portail Azure, une API REST, PowerShell ou Azure CLI.

Pour obtenir la clé de mesures des utilisateurs réels via le portail Azure, utilisez la procédure suivante :
1. Dans un navigateur, connectez-vous au portail Azure. Si vous n’avez pas encore de compte, vous pouvez vous inscrire pour bénéficier d’un essai gratuit d’un mois.
2. Dans la barre de recherche du portail, recherchez le nom du profil Traffic Manager que vous souhaitez modifier, puis cliquez sur le profil Traffic Manager dans les résultats affichés.
3. Dans la page du profil Traffic Manager, cliquez sur **Mesures des utilisateurs réels** sous **Paramètres**.
4. Cliquez sur **Générer une clé** pour créer une clé de mesures des utilisateurs réels.
        
   ![Générer une clé de mesures des utilisateurs réels](./media/traffic-manager-create-rum-visual-studio/generate-rum-key.png)

   **Figure 1 : Génération de la clé de mesures des utilisateurs réels**

5. La page affiche maintenant la clé de mesures des utilisateurs réels générée et un extrait de code JavaScript qui doit être incorporé à votre page HTML.
 
   ![Code JavaScript pour la clé de mesures des utilisateurs réels](./media/traffic-manager-create-rum-visual-studio/rum-key.png)

   **Figure 2 : Clé de mesures des utilisateurs réels et code JavaScript de mesure**
 
6. Cliquez sur le bouton **Copier** pour copier la clé de mesures des utilisateurs réels. 

## <a name="step-2-instrument-your-app-with-the-rum-package-of-mobile-center-sdk"></a>Étape 2 : Instrumenter votre application avec le package de mesures des utilisateurs réels du SDK Mobile Center

Si vous utilisez les services Visual Studio Mobile Center, consultez leur [site web](https://mobile.azure.com). Pour obtenir des instructions détaillées sur l’intégration du SDK, consultez [Bien démarrer avec Android SDK](https://docs.microsoft.com/mobile-center/sdk/getting-started/Android).

Pour utiliser des mesures d’utilisateurs réels, effectuez la procédure suivante :

1.  Ajoutez le SDK au projet.

    Dans la préversion du SDK Mesures des utilisateurs réels Azure Traffic Manager, vous devez explicitement référencer le référentiel du package.

    Dans votre fichier **app/build.gradle**, ajoutez les lignes suivantes :

    ```groovy
    repositories {
        maven {
            url "https://dl.bintray.com/mobile-center/mobile-center-snapshot"
        }
    }
    ```
    Dans votre fichier **app/build.gradle**, ajoutez les lignes suivantes :

    ```groovy
    dependencies {
     
        def mobileCenterSdkVersion = '0.12.1-16+3fe5b08'
        compile "com.microsoft.azure.mobile:mobile-center-rum:${mobileCenterSdkVersion}"
    }
    ```

2. Commencez le SDK.

    Ouvrez la classe de l’activité principale de votre application, puis ajoutez les instructions d’importation suivantes :

    ```java
    import com.microsoft.azure.mobile.MobileCenter;
    import com.microsoft.azure.mobile.rum.RealUserMeasurements;
    ```

    Recherchez le rappel `onCreate` dans le même fichier, puis ajoutez le code suivant :

    ```java
    RealUserMeasurements.setRumKey("<Your RUM Key>");
    MobileCenter.start(getApplication(), "<Your Mobile Center AppSecret>", RealUserMeasurements.class);
    ```

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [mesures des utilisateurs réels](traffic-manager-rum-overview.md).
- En savoir plus sur le [fonctionnement de Traffic Manager](traffic-manager-overview.md)
- En savoir plus sur [Mobile Center](https://docs.microsoft.com/mobile-center/)
- [Inscrivez-vous](https://mobile.azure.com) à Mobile Center
- En savoir plus sur les [méthodes de routage du trafic](traffic-manager-routing-methods.md) prises en charge par Traffic Manager
- En savoir plus sur la [création d’un profil Traffic Manager](traffic-manager-create-profile.md)

