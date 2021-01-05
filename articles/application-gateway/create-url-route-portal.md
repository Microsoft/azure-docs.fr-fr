---
title: 'Tutoriel : Règles de routage basées sur un chemin d’URL à l’aide du portail – Azure Application Gateway'
description: Dans ce tutoriel, découvrez comment créer des règles d’acheminement par chemin d’accès URL pour une passerelle d’application et un groupe de machines virtuelles identiques avec le portail Azure.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: tutorial
ms.date: 08/13/2020
ms.author: victorh
ms.openlocfilehash: 407bd5679c6afebf26c2e6b768e0f8513ac39123
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397584"
---
# <a name="tutorial-create-an-application-gateway-with-path-based-routing-rules-using-the-azure-portal"></a>Tutoriel : Créer une passerelle d’application avec des règles d’acheminement par chemin d’accès à l’aide du portail Azure

Vous pouvez utiliser le portail Azure pour configurer des [règles d’acheminement par chemin d’accès URL](./url-route-overview.md) lors de la création d’une [passerelle d’application](./overview.md). Ce didacticiel montre comment créer des pools principaux à l’aide de machines virtuelles. Il permet ensuite de définir des règles d’acheminement qui font en sorte que le trafic web arrive sur les serveurs voulus dans les pools.

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Créer une passerelle Application Gateway
> * Créer des machines virtuelles pour les serveurs principaux
> * Créer des pools principaux avec les serveurs principaux
> * Créer un écouteur principal
> * Créer une règle d’acheminement par chemin d’accès

![Exemple d’acheminement d’URL](./media/application-gateway-create-url-route-portal/scenario.png)

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="create-virtual-machines"></a>Créer des machines virtuelles

Dans cet exemple, vous créez trois machines virtuelles à utiliser en tant que serveurs principaux pour la passerelle d’application. Vous installez également IIS sur les machines virtuelles pour vérifier que la passerelle d’application fonctionne comme prévu.

1. Dans le portail Azure, sélectionnez **Créer une ressource**.
2. Sélectionnez **Windows Server 2016 Datacenter** dans la liste Populaire.
3. Entrez ces valeurs pour la machine virtuelle :

    - **Groupe de ressources** , sélectionnez **Créer nouveau** , puis tapez *myResourceGroupAG*.
    - **Nom de la machine virtuelle**  : *myVM1*
    - **Région**  : *(États-Unis) USA Est*
    - **Nom d’utilisateur**  : *azureuser*
    - **Mot de passe**  : *Azure123456!*


4. Sélectionnez **Suivant : Disques**.
5. Sélectionnez **Suivant : Réseaux**
6. Pour **Réseau virtuel** , sélectionnez **Créer nouveau** , puis tapez ces valeurs pour le réseau virtuel :

   - *myVNet* : pour le nom du réseau virtuel.
   - *10.0.0.0/16* : pour l’espace d’adressage du réseau virtuel.
   - *myBackendSubnet* pour le premier nom de sous-réseau
   - *10.0.1.0/24* : pour l’espace d’adressage du sous-réseau.
   - *myAGSubnet* pour le deuxième nom de sous-réseau.
   - *10.0.0.0/24* : pour l’espace d’adressage du sous-réseau.
7. Sélectionnez **OK**.

8. Vérifiez que, sous **Interface réseau** , **myBackendSubnet** est sélectionné pour le sous-réseau, puis sélectionnez **Suivant : Gestion**.
9. Sélectionnez **Désactivé** pour désactiver les diagnostics de démarrage.
10. Cliquez sur **Vérifier + créer** , vérifiez les paramètres dans la page de résumé, puis sélectionnez **Créer**.
11. Créez deux autres machines virtuelles, *myVM2* et *myVM3* , puis placez-les dans le réseau virtuel *MyVNet* et le sous-réseau *myBackendSubnet*.

### <a name="install-iis"></a>Installer IIS

1. Ouvrez l’interpréteur de commandes interactif et assurez-vous qu’il est défini sur **PowerShell**.

    ![Installer l’extension personnalisée](./media/application-gateway-create-url-route-portal/application-gateway-extension.png)

2. Exécutez la commande suivante pour installer IIS sur la machine virtuelle : 

    ```azurepowershell
         $publicSettings = @{ "fileUris" = (,"https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/appgatewayurl.ps1");  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File appgatewayurl.ps1" }

        Set-AzVMExtension `
         -ResourceGroupName myResourceGroupAG `
         -Location eastus `
         -ExtensionName IIS `
         -VMName myVM1 `
         -Publisher Microsoft.Compute `
         -ExtensionType CustomScriptExtension `
         -TypeHandlerVersion 1.4 `
         -Settings $publicSettings
    ```

3. Créez deux autres machines virtuelles et installez IIS à l’aide de la procédure que vous venez de terminer. Entrez les noms *myVM2* et *myVM3* pour les noms et les valeurs de VMName dans Set-AzVMExtension.

## <a name="create-an-application-gateway"></a>Créer une passerelle Application Gateway

1. Sélectionnez **Créer une ressource** dans le menu de gauche du portail Azure. La fenêtre **Nouvelle** apparaît.

2. Sélectionnez **Mise en réseau** , puis sélectionnez **Application Gateway** dans la liste **Sélection**.

### <a name="basics-tab"></a>Onglet Informations de base

1. Sous l’onglet **Informations de base** , entrez ces valeurs pour les paramètres de passerelle d’application suivants :

   - **Groupe de ressources**  : sélectionnez **myResourceGroupAG** comme nom de groupe de ressources.
   - **Nom de passerelle d’application**  : entrez *myAppGateway* comme nom de passerelle d’application.
   - **Région**  : sélectionnez **(États-Unis) USA Est**.

        ![Créer une passerelle d’application : Concepts de base](./media/application-gateway-create-gateway-portal/application-gateway-create-basics.png)

2.  Sous **Configurer un réseau virtuel** , sélectionnez **myVNet** pour le nom du réseau virtuel.
3. Sélectionnez **myAGSubnet** pour le sous-réseau.
3. Acceptez les valeurs par défaut des autres paramètres, puis sélectionnez **Suivant : Serveurs frontaux**.

### <a name="frontends-tab"></a>Onglet Front-ends

1. Sous l’onglet **Front-ends** , vérifiez que **Type d’adresse IP de front-end** est défini sur **Publique**.

   > [!NOTE]
   > Pour la référence SKU Application Gateway v2, vous pouvez uniquement choisir la configuration d’une adresse IP front-end **publique**. La configuration d’une adresse IP de front-end privée n’est pas activée pour la référence SKU v2.

2. Choisissez **Créer nouveau** pour **Adresse IP publique** , puis entrez *myAGPublicIPAddress* comme nom d’adresse IP publique, puis sélectionnez **OK**. 
3. Sélectionnez **Suivant : Back-ends**.

### <a name="backends-tab"></a>Onglet Back-ends

Le pool de back-ends est utilisé pour router les demandes vers les serveurs back-end qui les traitent. Les pools back-end peuvent être composés de cartes d’interface réseau, de groupes de machines virtuelles identiques, d’adresses IP publiques, d’adresses IP internes, de noms de domaine complets et de serveurs back-end multi-locataires comme Azure App Service.

1. Sous l’onglet **Backends** , sélectionnez **+Ajouter un pool de back-ends**.

2. Dans la fenêtre **Ajouter un pool de back-ends** qui s’ouvre, entrez les valeurs suivantes pour créer un pool de back-ends vide :

    - **Name**  : Entrez *myBackendPool* comme nom du pool de back-ends.
3. Sous **Cibles de back-end** , **Type de cible** , sélectionnez **Machine virtuelle** dans la liste déroulante.

5. Sous **Cible** , sélectionnez l’interface réseau pour **myVM1**.
6. Sélectionnez **Ajouter**.
7. Répétez l’opération pour ajouter un pool back-end *Images* avec *myVM2* comme cible et un pool back-end *Vidéo* avec *myVM3* comme cible.
8. Sélectionnez **Ajouter** pour enregistrer la configuration du pool back-end et revenir à l’onglet **Back-ends**.

4. Sous l’onglet **Back-ends** , sélectionnez **Suivant : Configuration**.

### <a name="configuration-tab"></a>Onglet Configuration

Sous l’onglet **Configuration** , vous allez connecter le front-end et le pool de back-ends que vous avez créés à l’aide d’une règle de routage.

1. Sélectionnez **Ajouter une règle** dans la colonne **Règles de routage**.

2. Dans la fenêtre **Ajouter une règle de routage** qui s’ouvre, entrez *myRoutingRule* pour **Nom de la règle**.

3. Une règle de routage requiert un écouteur. Sous l’onglet **Écouteur** dans la fenêtre **Ajouter une règle de routage** , entrez les valeurs suivantes pour l’écouteur :

    - **Nom de l’écouteur**  : Entrez *myListener* comme nom pour l’écouteur.
    - **Adresse IP du front-end**  : Sélectionnez **Publique** pour choisir l’adresse IP publique que vous avez créée pour le front-end.
    - **Port** : Tapez *8080*
  
        Acceptez les valeurs par défaut pour les autres paramètres sous l’onglet **Écouteur** , puis sélectionnez l’onglet **Cibles de back-end** pour configurer le reste de la règle de routage.

4. Sous l’onglet **Cibles de back-end** , sélectionnez **myBackendPool** pour la **Cible de back-end**.

5. Pour **Paramètre HTTP** , sélectionnez **Créer nouveau** pour créer un nouveau paramètre HTTP. Le paramètre HTTP détermine le comportement de la règle de routage. 

6. Dans la fenêtre **Ajouter un paramètre HTTP** qui s’ouvre, entrez *myHTTPSetting* pour le **Nom du paramètre HTTP**. Acceptez les valeurs par défaut pour les autres paramètres de la fenêtre **Ajouter un paramètre HTTP** , puis sélectionnez **Ajouter** pour revenir à la fenêtre **Ajouter une règle de routage**.
7. Sous **Routage basé sur le chemin** , sélectionnez **Ajouter plusieurs cibles pour créer une règle basée sur le chemin**.
8. Pour **Chemin** , tapez */images/* \*.
9. Pour **Nom de la règle de chemin** , tapez *Images*.
10. Pour **Paramètre HTTP** , sélectionnez **myHTTPSetting**
11. Pour **Cible de back-end** , sélectionnez **Images**.
12. Sélectionnez **Ajouter** pour enregistrer la règle de chemin et revenir à l’onglet **Ajouter une règle de routage**.
13. Répétez cette opération pour ajouter une autre règle pour Vidéo.
14. Sélectionnez **Ajouter** pour ajouter la règle de routage et revenir à l’onglet **Configuration**.
15. Sélectionnez **Suivant : Balises** , puis sur **Suivant : Vérifier + créer**.

> [!NOTE]
> Vous n’avez pas besoin d’ajouter une règle de chemin */* * personnalisée pour gérer les cas par défaut. Cela est géré automatiquement par le pool back-end par défaut.

### <a name="review--create-tab"></a>Onglet Vérifier + créer

Examinez les paramètres sous l’onglet **Vérifier + créer** , puis sélectionnez **Créer** pour créer le réseau virtuel, l’adresse IP publique et la passerelle d’application. La création de la passerelle d’application par Azure peut prendre plusieurs minutes. Patientez jusqu’à ce que le déploiement soit terminé avant de passer à la section suivante.


## <a name="test-the-application-gateway"></a>Tester la passerelle d’application

1. Sélectionnez **Toutes les ressources** , puis **myAppGateway**.

    ![Enregistrer l’adresse IP publique de la passerelle d’application](./media/application-gateway-create-url-route-portal/application-gateway-record-ag-address.png)

2. Copiez l’adresse IP publique, puis collez-la dans la barre d’adresses de votre navigateur. Par exemple, http:\//52.188.72.175:8080.

    ![Tester l’URL de base dans la passerelle d’application](./media/application-gateway-create-url-route-portal/application-gateway-iistest.png)

   L’écouteur sur le port 8080 route cette demande vers le pool back-end par défaut.

3. Modifiez l’URL en *http://&lt;ip-address&gt;:8080/images/test.htm* , en remplaçant &lt;ip-address&gt; par votre adresse IP. Vous devriez voir quelque chose ressemblant à ceci :

    ![Tester l’URL images dans la passerelle d’application](./media/application-gateway-create-url-route-portal/application-gateway-iistest-images.png)

   L’écouteur sur le port 8080 route cette demande vers le pool back-end *Images*.

4. Modifiez l’URL en *http://&lt;ip-address&gt;:8080/video/test.htm* , en remplaçant &lt;ip-address&gt; par votre adresse IP. Vous devriez voir quelque chose ressemblant à ceci :

    ![Tester l’URL vidéo dans la passerelle d’application](./media/application-gateway-create-url-route-portal/application-gateway-iistest-video.png)

   L’écouteur sur le port 8080 route cette demande vers le pool back-end *Vidéo*.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, supprimez le groupe de ressources et toutes les ressources associées. Pour ce faire, sélectionnez le groupe de ressources, puis **Supprimer le groupe de ressources**.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Activer le protocole TLS de bout en bout sur Azure Application Gateway](./ssl-overview.md)