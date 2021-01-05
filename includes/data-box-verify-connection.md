---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 12/07/2018
ms.author: alkohli
ms.openlocfilehash: 10feccb53b28181d260e7738ab99bdc2e3c9340c
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94553147"
---
Suivez ces étapes pour vous connecter au compte de stockage et vérifier la connexion.

1. Dans l’Explorateur Stockage, ouvrez la boîte de dialogue **Connexion au stockage Azure**. Dans la boîte de dialogue **Connexion au stockage Azure**, sélectionnez **Utiliser un nom et une clé de compte de stockage**.

    ![La capture d’écran montre la boîte de dialogue Connexion au stockage Azure avec l’option Utiliser un nom et une clé de compte de stockage sélectionnée.](media/data-box-verify-connection/data-box-connect-via-rest-9.png)

2. Collez votre **Nom du compte** et **Clé du compte** (valeur de la clé 1 à partir de la page **Connexion et copie** dans l’interface utilisateur web locale). Sélectionnez Domaine des points de terminaison de stockage en tant que **Autre (Entrez ci-dessous)** , puis précisez le point de terminaison du service BLOB, comme indiqué ci-dessous. Cochez l’option **Utiliser HTTP** uniquement si le transfert s’effectue via *http*. Si vous utilisez *https*, laissez l’option désactivée. Sélectionnez **Suivant**.

    ![La capture d’écran montre la boîte de dialogue Se connecter avec le nom et la clé ainsi que les valeurs entrées.](media/data-box-verify-connection/data-box-connect-via-rest-11.png)    

3. Dans la boîte de dialogue **Résumé de la connexion**, vérifiez les informations fournies. Sélectionnez **Connecter**.

    ![La capture d’écran montre la boîte de dialogue Résumé de la connexion avec l’option Se connecter sélectionnée.](media/data-box-verify-connection/data-box-connect-via-rest-12.png)

4. Le compte que vous avez correctement ajouté s’affiche dans le volet gauche de l’Explorateur Stockage avec (Externe, Autre) ajouté à son nom. Cliquez sur **Conteneurs d’objets Blob** pour afficher le conteneur.

    ![La capture d’écran montre le menu de l’Explorateur avec l’option Conteneurs d’objets blob sélectionnée.](media/data-box-verify-connection/data-box-connect-via-rest-17.png)
