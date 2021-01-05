---
title: 'Tutoriel : Héberger plusieurs sites web à l’aide du Portail Azure'
titleSuffix: Azure Application Gateway
description: Dans ce tutoriel, vous découvrez comment créer une passerelle d’application qui héberge plusieurs sites web à l’aide du portail Azure.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: tutorial
ms.date: 08/21/2020
ms.author: victorh
ms.openlocfilehash: 16f55dc88ed2d2d019a2fed355a14741263c20af
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397601"
---
# <a name="tutorial-create-and-configure-an-application-gateway-to-host-multiple-web-sites-using-the-azure-portal"></a>Tutoriel : Créer et configurer une passerelle d’application pour héberger plusieurs sites web avec le Portail Azure

Vous pouvez utiliser le Portail Azure pour [configurer l’hébergement de plusieurs sites web](multiple-site-overview.md) quand vous créez une [passerelle d’application](overview.md). Dans ce tutoriel, vous allez définir des pools d’adresses principaux à l’aide de machines virtuelles. Vous configurez ensuite des écouteurs et des règles en fonction des domaines qui vous appartiennent pour vérifier que le trafic web arrive sur les serveurs appropriés dans les pools. Ce didacticiel, qui part du principe que vous avez plusieurs domaines, utilise *www.contoso.com* et *www.fabrikam.com* en guise d’exemples.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une passerelle Application Gateway
> * Créer des machines virtuelles pour les serveurs principaux
> * Créer des pools principaux avec les serveurs principaux
> * Créer des écouteurs principaux
> * Créer des règles d’acheminement
> * Créer un enregistrement CNAME dans votre domaine

:::image type="content" source="./media/create-multiple-sites-portal/scenario.png" alt-text="Passerelle d’application multisite":::

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="create-an-application-gateway"></a>Créer une passerelle Application Gateway

1. Sélectionnez **Créer une ressource** dans le menu de gauche du portail Azure. La fenêtre **Nouvelle** apparaît.

2. Sélectionnez **Mise en réseau** , puis sélectionnez **Application Gateway** dans la liste **Sélection**.

### <a name="basics-tab"></a>Onglet Informations de base

1. Sous l’onglet **Informations de base** , entrez ces valeurs pour les paramètres de passerelle d’application suivants :

   - **Groupe de ressources**  : sélectionnez **myResourceGroupAG** comme nom de groupe de ressources. Si ce groupe n’existe pas encore, sélectionnez **Créer** pour le créer.
   - **Nom de passerelle d’application**  : entrez *myAppGateway* comme nom de passerelle d’application.

     :::image type="content" source="./media/application-gateway-create-gateway-portal/application-gateway-create-basics.png" alt-text="Créer une passerelle Application Gateway":::

2.  Azure a besoin d’un réseau virtuel pour communiquer avec les différentes ressources que vous créez. Vous pouvez créer un réseau virtuel ou en utiliser un. Dans cet exemple, vous allez créer un réseau virtuel en même temps que la passerelle d’application. Les instances Application Gateway sont créées dans des sous-réseaux séparés. Vous créez deux sous-réseaux dans cet exemple : un pour la passerelle d’application et un autre pour les serveurs back-end.

    Sous **Configurer le réseau virtuel** , sélectionnez **Créer nouveau** pour créer un réseau virtuel. Dans la fenêtre **Créer un réseau virtuel** qui s’ouvre, entrez les valeurs suivantes pour créer le réseau virtuel et deux sous-réseaux :

    - **Name**  : entrez *myVNet* comme nom de réseau virtuel.

    - **Nom de sous-réseau** (sous-réseau Application Gateway) : La grille **Sous-réseaux** affiche un sous-réseau nommé *Par défaut*. Remplacez le nom de ce sous-réseau par *myAGSubnet*.<br>Le sous-réseau de passerelle d’application peut contenir uniquement des passerelles d’application. Aucune autre ressource n’est autorisée.

    - **Nom de sous-réseau** (sous-réseau de serveur principal) : Dans la deuxième ligne de la grille **Sous-réseaux** , entrez *myBackendSubnet* dans la colonne **Nom de sous-réseau**.

    - **Plage d’adresses** (sous-réseau de serveur principal) : Dans la deuxième ligne de la grille **Sous-réseaux** , entrez une plage d’adresses qui ne chevauche pas la plage d’adresses de *myAGSubnet*. Par exemple, si la plage d’adresses de *myAGSubnet* est 10.0.0.0/24, entrez *10.0.1.0/24* pour la plage d’adresses de *myBackendSubnet*.

    Sélectionnez **OK** pour fermer la fenêtre **Créer un réseau virtuel** et enregistrez les paramètres du réseau virtuel.

     :::image type="content" source="./media/application-gateway-create-gateway-portal/application-gateway-create-vnet.png" alt-text="Créer un réseau virtuel":::
    
3. Sous l’onglet **Informations de base** , acceptez les valeurs par défaut des autres paramètres, puis sélectionnez **Suivant : Serveurs frontaux**.

### <a name="frontends-tab"></a>Onglet Front-ends

1. Sous l’onglet **Front-ends** , vérifiez que **Type d’adresse IP de front-end** est défini sur **Publique**. <br>Vous pouvez l’adresse IP frontale pour qu’elle soit publique ou privée conformément à votre cas d’utilisation. Dans cet exemple, vous allez choisir une adresse IP front-end publique.
   > [!NOTE]
   > Pour la référence SKU Application Gateway v2, vous pouvez uniquement choisir la configuration d’une adresse IP front-end **publique**. La configuration d’une adresse IP de front-end privée n’est pas activée pour la référence SKU v2.

2. Choisissez **Créer nouveau** pour **Adresse IP publique** , puis entrez *myAGPublicIPAddress* comme nom d’adresse IP publique, puis sélectionnez **OK**. 

     :::image type="content" source="./media/application-gateway-create-gateway-portal/application-gateway-create-frontends.png" alt-text="Créer un autre réseau virtuel":::

3. Sélectionnez **Suivant : Back-ends**.

### <a name="backends-tab"></a>Onglet Back-ends

Le pool de back-ends est utilisé pour router les demandes vers les serveurs back-end qui les traitent. Les pools de back-ends peuvent être composés de cartes d’interface réseau, de groupes de machines virtuelles identiques, d’adresses IP publiques, d’adresses IP internes, de noms de domaine complets et de back-ends multilocataires comme Azure App Service. Dans cet exemple, vous allez créer un pool de back-ends vide avec votre passerelle d’application, puis ajouter des cibles de back-end au pool.

1. Sous l’onglet **Backends** , sélectionnez **+Ajouter un pool de back-ends**.

2. Dans la fenêtre **Ajouter un pool de back-ends** qui s’ouvre, entrez les valeurs suivantes pour créer un pool de back-ends vide :

    - **Name**  : Entrez *contosoPool* pour le nom du pool de back-ends.
    - **Ajouter un pool backend sans cible**  : Sélectionnez **Oui** pour créer un pool de back-ends sans cible. Vous ajouterez des cibles de back-end après avoir créé la passerelle d’application.

3. Dans la fenêtre **Ajouter un pool de back-ends** , sélectionnez  **Ajouter** pour enregistrer la configuration du pool de back-ends et revenir à l’onglet **Back-ends**.
4. Ajoutez maintenant un autre pool de back-ends appelé *fabrikamPool*.

    :::image type="content" source="./media/create-multiple-sites-portal/backend-pools.png" alt-text="Créer des backends":::

4. Sous l’onglet **Back-ends** , sélectionnez **Suivant : Configuration**.

### <a name="configuration-tab"></a>Onglet Configuration

Sous l’onglet **Configuration** , vous allez connecter les pools de front-ends et de back-ends que vous avez créés à l’aide d’une règle de routage.

1. Sélectionnez **Ajouter une règle** dans la colonne **Règles de routage**.

2. Dans la fenêtre **Ajouter une règle de routage** qui s’ouvre, entrez *contosoRule* pour le **Nom de la règle**.

3. Une règle de routage requiert un écouteur. Sous l’onglet **Écouteur** dans la fenêtre **Ajouter une règle de routage** , entrez les valeurs suivantes pour l’écouteur :

    - **Nom de l’écouteur**  : Entrez *contosoListener* pour le nom de l’écouteur.
    - **Adresse IP du front-end**  : Sélectionnez **Publique** pour choisir l’adresse IP publique que vous avez créée pour le front-end.

   Sous **Paramètres supplémentaires**  :
   - **Type d’écouteur**  : Plusieurs sites
   - **Nom de l’hôte**  : **www.contoso.com**

   Acceptez les valeurs par défaut pour les autres paramètres sous l’onglet **Écouteur** , puis sélectionnez l’onglet **Cibles de back-end** pour configurer le reste de la règle de routage.

   :::image type="content" source="./media/create-multiple-sites-portal/routing-rule.png" alt-text="Créer une règle d’acheminement":::

4. Sous l’onglet **Cibles de back-end** , sélectionnez **contosoPool** pour la **Cible de back-end**.

5. Pour **Paramètre HTTP** , sélectionnez **Créer nouveau** pour créer un nouveau paramètre HTTP. Le paramètre HTTP détermine le comportement de la règle de routage. Dans la fenêtre **Ajouter un paramètre HTTP** qui s’ouvre, entrez *contosoHTTPSetting* pour le **Nom du paramètre HTTP**. Acceptez les valeurs par défaut pour les autres paramètres de la fenêtre **Ajouter un paramètre HTTP** , puis sélectionnez **Ajouter** pour revenir à la fenêtre **Ajouter une règle de routage**. 

6. Dans la fenêtre **Ajouter une règle de routage** , sélectionnez **Ajouter** pour enregistrer la règle de routage et revenir à l’onglet **Configuration**.
7. Sélectionnez **Ajouter une règle** et ajoutez une règle, un écouteur, une cible de back-end et un paramètre HTTP similaires pour Fabrikam.

     :::image type="content" source="./media/create-multiple-sites-portal/fabrikam-rule.png" alt-text="Règle Fabrikam":::

7. Sélectionnez **Suivant : Balises** , puis sur **Suivant : Vérifier + créer**.

### <a name="review--create-tab"></a>Onglet Vérifier + créer

Examinez les paramètres sous l’onglet **Vérifier + créer** , puis sélectionnez **Créer** pour créer le réseau virtuel, l’adresse IP publique et la passerelle d’application. La création de la passerelle d’application par Azure peut prendre plusieurs minutes.

Patientez jusqu’à ce que le déploiement soit terminé avant de passer à la section suivante.

## <a name="add-backend-targets"></a>Ajouter des cibles de back-end

Dans cet exemple, vous allez utiliser des machines virtuelles comme back-end cible. Vous pouvez utiliser des machines virtuelles existantes ou en créer de nouvelles. Vous allez créer deux machines virtuelles qu’Azure va utiliser comme serveurs back-end pour la passerelle d’application.

Pour ajouter des cibles de back-end, vous allez :

1. Créez deux machines virtuelles, *contosoVM* et *fabrikamVM* , à utiliser comme serveurs back-end.
2. Installer IIS sur les machines virtuelles pour vérifier que la passerelle d’application a bien été créée.
3. Ajoutez les serveurs back-end au pool de back-ends.

### <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

1. Dans le portail Azure, sélectionnez **Créer une ressource**. La fenêtre **Nouvelle** apparaît.
2. Sélectionnez **Compute** , puis **Windows Server 2016 Datacenter** dans la liste **Populaire**. La page **Créer une machine virtuelle** s’affiche.<br>Application Gateway peut acheminer le trafic vers n’importe quel type de machine virtuelle utilisée dans son pool principal. Dans cet exemple, vous utilisez un serveur Windows Server 2016 Datacenter.
3. Sous l’onglet **De base** , entrez ces valeurs pour les paramètres de machine virtuelle suivants :

    - **Groupe de ressources**  : sélectionnez **myResourceGroupAG** comme nom de groupe de ressources.
    - **Nom de la machine virtuelle** : Entrez *contosoVM* pour le nom de la machine virtuelle.
    - **Nom d’utilisateur** : Entrez un nom d’utilisateur administrateur.
    - **Mot de passe**  : Entrez un mot de passe pour l’administrateur.
1. Acceptez les autres valeurs par défaut, puis sélectionnez **Suivant : Disques**.  
2. Acceptez les valeurs par défaut sous l’onglet **Disques** , puis sélectionnez **Suivant : Mise en réseau**.
3. Sous l’onglet **Mise en réseau** , vérifiez que **myVNet** est sélectionné comme **Réseau virtuel** et que **Sous-réseau** est défini sur  **myBackendSubnet**. Acceptez les autres valeurs par défaut, puis sélectionnez **Suivant : Gestion**.<br>Application Gateway peut communiquer avec des instances en dehors du réseau virtuel dans lequel il réside, mais vous devez vérifier qu’il existe une connectivité IP.
4. Sous l’onglet **Gestion** , définissez **Diagnostics de démarrage** sur **Désactivé**. Acceptez les autres valeurs par défaut, puis sélectionnez **Vérifier + créer**.
5. Sous l’onglet **Vérifier + créer** , passez en revue les paramètres, corrigez les éventuelles erreurs de validation et sélectionnez **Créer**.
6. Attendez la fin de la création de la machine virtuelle avant de continuer.

### <a name="install-iis-for-testing"></a>Installer IIS pour les tests

Dans cet exemple, vous allez installer IIS sur les machines virtuelles uniquement pour vérifier si Azure a bien créé la passerelle d’application.

1. Ouvrez [Azure PowerShell](../cloud-shell/quickstart-powershell.md). Pour ce faire, sélectionnez **Cloud Shell** dans la barre de navigation supérieure du portail Azure, puis sélectionnez **PowerShell** dans la liste déroulante. 

    ![Installer l’extension personnalisée](./media/application-gateway-create-gateway-portal/application-gateway-extension.png)

2. Exécutez la commande suivante pour installer IIS sur la machine virtuelle, en remplaçant la région de votre groupe de ressources par <location\> : 

    ```azurepowershell-interactive
    Set-AzVMExtension `
      -ResourceGroupName myResourceGroupAG `
      -ExtensionName IIS `
      -VMName contosoVM `
      -Publisher Microsoft.Compute `
      -ExtensionType CustomScriptExtension `
      -TypeHandlerVersion 1.4 `
      -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
      -Location <location>
    ```

3. Créez une deuxième machine virtuelle et installez IIS en suivant les mêmes étapes que vous avez effectuées précédemment. Utilisez *fabrikamVM* pour le nom de la machine virtuelle et pour le paramètre **VMName** de l’applet de commande **Set-AzVMExtension**.

### <a name="add-backend-servers-to-backend-pools"></a>Ajouter des serveurs back-end au pool de back-ends

1. Sélectionnez **Toutes les ressources** , puis **myAppGateway**.

2. Sélectionnez **Pools principaux** dans le menu de gauche.

3. Sélectionnez **contosoPool**.

4. Sous **Cibles** , sélectionnez **Machine virtuelle** dans la liste déroulante.

5. Sous **MACHINE VIRTUELLE** et **INTERFACES RÉSEAU** , sélectionnez la machine virtuelle **contosoVM** et son interface réseau associée dans les listes déroulantes.

    ![Ajouter des serveurs principaux](./media/create-multiple-sites-portal/edit-backend-pool.png)

6. Sélectionnez **Enregistrer**.
7. Répétez l’opération pour ajouter la machine virtuelle *fabrikamVM* et son interface à *fabrikamPool*.

Attendez que le déploiement se termine avant de passer à l’étape suivante.

## <a name="create-a-www-a-record-in-your-domains"></a>Créer un enregistrement www A dans vos domaines

Une fois la passerelle d’application créée avec son adresse IP publique, vous pouvez obtenir l’adresse IP et l’utiliser pour créer un enregistrement A dans vos domaines. 

1. Cliquez sur **Toutes les ressources** , puis sur **myAGPublicIPAddress**.

    ![Enregistrer l’adresse DNS de la passerelle d’application](./media/create-multiple-sites-portal/public-ip.png)

2. Copiez l’adresse IP et utilisez-la comme valeur pour un nouvel enregistrement *www* dans vos domaines.

## <a name="test-the-application-gateway"></a>Tester la passerelle d’application

1. Entrez votre nom de domaine dans la barre d’adresse de votre navigateur. Par exemple, `http://www.contoso.com`.

    ![Tester le site contoso dans la passerelle d’application](./media/create-multiple-sites-portal/application-gateway-iistest.png)

2. Remplacez l’adresse par celle de votre autre domaine. Voici ce qui doit apparaître :

    ![Tester le site fabrikam dans la passerelle d’application](./media/create-multiple-sites-portal/application-gateway-iistest2.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des ressources que vous avez créées avec la passerelle d’application, supprimez le groupe de ressources. Quand vous supprimez le groupe de ressources, vous supprimez aussi la passerelle d’application et toutes ses ressources associées.

Pour supprimer le groupe de ressources :

1. Dans le menu de gauche du portail Azure, sélectionnez **Groupes de ressources**.
2. Dans la page **Groupes de ressources** , recherchez **myResourceGroupAG** dans la liste, puis sélectionnez ce groupe de ressources.
3. Dans la page **Groupe de ressources** , sélectionnez **Supprimer le groupe de ressources**.
4. Entrez *myResourceGroupAG* dans **TYPE THE RESOURCE GROUP NAME** , puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur ce que vous pouvez faire avec Azure Application Gateway](./overview.md)