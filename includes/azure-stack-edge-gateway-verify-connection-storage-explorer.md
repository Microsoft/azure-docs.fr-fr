---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 12/02/2019
ms.author: alkohli
ms.openlocfilehash: 203c977fe9109cd8b2b6de561e975e20aacf700e
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185537"
---
Si vous utilisez l’Explorateur Stockage pour la première fois, vous devez effectuer les étapes suivantes.

1. Dans la barre de commandes du haut, accédez à **Modifier > Cibler les API Azure Stack**.

    ![Configurer l’Explorateur Stockage](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-1.png)

2. Redémarrez l’Explorateur Stockage pour la prise en compte des modifications.


Suivez ces étapes pour vous connecter au compte de stockage et vérifier la connexion.

1. Dans l’Explorateur Stockage, sélectionnez Comptes de stockage. Cliquez avec le bouton droit et sélectionnez l’option **Se connecter au Stockage Azure**. 

    ![Configurer l’Explorateur Stockage 2](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-2.png)

2. Dans la boîte de dialogue **Connexion au stockage Azure**, sélectionnez **Utiliser un nom et une clé de compte de stockage**.

    ![Configurer l’Explorateur Stockage 3](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-3.png)

2. Dans la boîte de dialogue **Se connecter avec le nom et la clé**, procédez comme suit :

    1. Entrez un nom d’affichage pour votre compte de stockage Edge. 
    2. Indiquez le nom de compte de stockage Edge.
    3. Collez la clé d’accès que vous avez obtenue à partir des API locales de l’appareil via Azure Resource Manager.
    4. Sélectionnez Domaine de stockage en tant que **Autre (Entrez ci-dessous)** , puis précisez le suffixe ou point de terminaison du service Blob au format : `<appliance name>.<DNSdomain>`. 
    5. Cochez l’option **Utiliser HTTP** lorsque le transfert se fait sur *http*. 
    6. Sélectionnez **Suivant**.

    ![Configurer l’Explorateur Stockage 4](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-4.png)    

3. Dans la boîte de dialogue **Résumé de la connexion**, vérifiez les informations fournies. Sélectionnez **Connecter**.

    ![Configurer l’Explorateur Stockage 5](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-5.png)

4. Le compte que vous avez correctement ajouté s’affiche dans le volet gauche de l’Explorateur Stockage avec (Externe, Autre) ajouté à son nom. Sélectionnez **Conteneurs d’objets Blob** pour afficher le conteneur.

    ![Afficher des conteneurs d’objets Blob](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-6.png)

L’étape suivante consiste à vérifier que le transfert de données fonctionne correctement sur cette connexion.

Procédez comme suit pour charger des données dans votre compte de stockage Edge sur l’appareil, et elles devraient automatiquement être hiérarchisées sur le compte de stockage Azure mappé.

1. Sélectionnez le conteneur dans lequel vous voulez charger les données dans votre compte de stockage Edge. Sélectionnez **Charger**, puis **Charger des fichiers**.

    ![Vérifier le transfert de données](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-1.png)

2. Dans la boîte de dialogue **Télécharger des fichiers**, accédez à et sélectionnez les fichiers que vous souhaitez télécharger. Sélectionnez **Suivant**.

    ![Vérifier le transfert de données 2](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-2.png)

3. Vérifiez que les fichiers ont été téléchargés. Les fichiers téléchargés apparaissent dans le conteneur.

    ![Vérifier le transfert de données 3](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-3.png)

4. Ensuite, vous allez vous connecter au compte de stockage Azure qui a été mappé à ce compte de stockage Edge. Toutes les données qui sont téléchargées vers le compte de stockage Edge doivent automatiquement être hiérarchisées sur le compte de stockage Azure. 
    
    Pour obtenir la chaîne de connexion pour le compte de stockage Azure, accédez au **Compte de stockage Azure > Clés d’accès** et copiez la chaîne de connexion.

    ![Vérifier le transfert de données 4](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-5.png)

    Utilisez la chaîne de connexion pour joindre un compte Stockage Azure hébergé.  

    ![Vérifier le transfert de données 5](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-4.png)


5. Dans la boîte de dialogue **Résumé de la connexion**, vérifiez les informations fournies. Sélectionnez **Connecter**.

    ![Vérifier le transfert de données 6](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-6.png)

6. Vous verrez que les fichiers que vous avez téléchargés dans le compte de stockage Edge ont été transférés vers le compte de stockage Azure.

    ![Vérifier le transfert de données 7](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-7.png)
