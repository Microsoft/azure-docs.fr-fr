---
title: Configurer un point de terminaison d’équilibreur de charge interne (ILB)
titleSuffix: Azure Application Gateway
description: Cet article fournit des informations sur la configuration d’Application Gateway avec une adresse IP frontale privée.
services: application-gateway
author: abshamsft
ms.service: application-gateway
ms.topic: how-to
ms.date: 04/16/2020
ms.author: victorh
ms.openlocfilehash: 64dfe284772faf2a345b7959f1a1bd6f474cd1bf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90562483"
---
# <a name="configure-an-application-gateway-with-an-internal-load-balancer-ilb-endpoint"></a>Configurer une passerelle Application Gateway avec un point de terminaison d’équilibreur de charge interne (ILB)

Vous pouvez configurer Azure Application Gateway avec une adresse IP virtuelle accessible via Internet ou avec un point de terminaison interne non exposé à Internet. Un point de terminaison interne utilise une adresse IP privée pour le serveur frontal, également appelée *point de terminaison d’équilibreur de charge interne*.

Il est utile de configurer la passerelle à l’aide d’une adresse IP privée frontale pour des applications métier internes non exposées à Internet. Elle est également utile pour les services et niveaux au sein d'une application multiniveau qui se trouve dans une limite de sécurité non exposée à Internet, mais qui requiert tout de même une distribution de charge par tourniquet, une adhérence de session ou une terminaison TLS (Transport Layer Security), anciennement SSL (Secure Sockets Layer).

Cet article vous guide dans les étapes de configuration d’une passerelle Application Gateway avec une adresse IP privée frontale à l’aide du portail Azure.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur <https://portal.azure.com>

## <a name="create-an-application-gateway"></a>Créer une passerelle Application Gateway

Azure a besoin d’un réseau virtuel pour communiquer avec les différentes ressources que vous créez. Vous pouvez créer un réseau virtuel ou en utiliser un. Dans cet exemple, vous allez créer un réseau virtuel. Vous pouvez créer un réseau virtuel en même temps que la passerelle d’application. Les instances Application Gateway sont créées dans des sous-réseaux séparés. Vous créez deux sous-réseaux dans cet exemple : un pour la passerelle d’application et un autre pour les serveurs back-end.

1. Développez le menu du portail, puis sélectionnez **Créer une ressource**.
2. Sélectionnez **Mise en réseau**, puis sélectionnez **Application Gateway** dans la liste de suggestions.
3. Entrez *myAppGateway* pour le nom de la passerelle d’application et *myResourceGroupAG* pour le nouveau groupe de ressources.
4. Pour **Région**, sélectionnez **USA Centre**.
5. Pour **Niveau**, sélectionnez **Standard**.
6. Sous **Configurer un réseau virtuel**, sélectionnez **Créer**, puis entrez les valeurs suivantes pour le réseau virtuel :
   - *myVNet* : pour le nom du réseau virtuel.
   - *10.0.0.0/16* : pour l’espace d’adressage du réseau virtuel.
   - *myAGSubnet* : pour le nom du sous-réseau.
   - *10.0.0.0/24* : pour l’espace d’adressage du sous-réseau.
   - *myBackendSubnet* : pour le nom de sous-réseau principal.
   - *10.0.1.0/24* : pour l’espace d’adressage de sous-réseau principal.

    ![Création d’un réseau virtuel](./media/configure-application-gateway-with-private-frontend-ip/private-frontendip-1.png)

6. Sélectionnez **OK** pour créer le réseau virtuel et le sous-réseau.
7. Sélectionnez **Suivant : Serveurs frontaux**.
8. Pour **Type d’adresse IP de serveur frontal**, sélectionnez **Privée**.

   Par défaut, il s’agit d’une affectation d’adresse IP dynamique. La première adresse disponible du sous-réseau configuré est affectée en tant qu’adresse IP du serveur frontal.
   > [!NOTE]
   > Une fois alloué, le type d’adresse IP (statique ou dynamique) ne peut pas être modifié ultérieurement.
9. Sélectionnez **Suivant : Serveurs principaux**.
10. Sélectionnez **Ajouter un pool principal**.
11. Pour **Nom**, tapez *appGatewayBackendPool*.
12. Pour **Ajouter un pool principal sans cible**, sélectionnez **Oui**. Vous ajouterez les cibles ultérieurement.
13. Sélectionnez **Ajouter**.
14. Sélectionnez **Suivant : Configuration**.
15. Sous **Règle d’acheminement**, sélectionnez **Ajouter une règle**.
16. Pour **Nom de la règle**, tapez *Rule-01*.
17. Pour **Nom de l’écouteur**, tapez *Listener-01*.
18. Pour **Adresse IP du front-end**, sélectionnez **Privée**.
19. Acceptez les valeurs par défaut restantes, puis sélectionnez l’onglet **Cibles de back-end**.
20. Pour **Type de cible**, sélectionnez **Pool principal**, puis **appGatewayBackendPool**.
21. Pour **Paramètres HTTP**, sélectionnez **Créer**.
22. Pour **Nom du paramètre HTTP**, tapez *http-Setting-01*.
23. Pour **Protocole principal**, sélectionnez **HTTP**.
24. Pour **Port principal**, tapez *80*.
25. Acceptez les valeurs par défaut restantes, puis sélectionnez **Ajouter**.
26. Dans la page **Ajouter une règle de routage**, sélectionnez **Ajouter**.
27. Sélectionnez **Suivant : Balises**.
28. Sélectionnez **Suivant : Vérifier + créer**.
29. Passez en revue les paramètres sur la page de résumé, puis sélectionnez **Créer** pour créer les ressources réseau et la passerelle d’application. La création de la passerelle d’application peut prendre plusieurs minutes. Patientez jusqu’à ce que le déploiement soit terminé avant de passer à la section suivante.

## <a name="add-backend-pool"></a>Ajouter le pool principal

Le pool de back-ends est utilisé pour router les demandes vers les serveurs back-end qui les traitent. Le pool principal peut se composer de cartes d’interface réseau, de groupes de machines virtuelles identiques, d’adresses IP publiques, d’adresses IP internes, de noms de domaine complets (FQSN) et de services principaux mutualisés comme Azure App Service. Dans cet exemple, vous allez utilisez des machines virtuelles comme back-end cible. Vous pouvez utiliser des machines virtuelles existantes ou en créer de nouvelles. Dans cet exemple, vous créez deux machines virtuelles qu’Azure va utiliser comme serveurs back-end pour la passerelle d’application.

Pour ce faire, procédez comme suit :

1. Créez deux machines virtuelles, *myVM* et *myVM2*, utilisées en tant que serveurs principaux.
2. Installer IIS sur les machines virtuelles pour vérifier que la passerelle d’application a bien été créée.
3. Ajoutez les serveurs principaux au pool principal.

### <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

1. Sélectionnez **Créer une ressource**.
2. Sélectionnez **Compute**, puis **Machine virtuelle**.
4. Entrez ces valeurs pour la machine virtuelle :
   - Sélectionnez *myResourceGroupAG* pour **Groupe de ressources**.
   - *myVM* - pour **Nom de la machine virtuelle**.
   - Sélectionnez **Windows Server 2019 Datacenter** pour **Image**.
   - un **nom d'utilisateur** valide.
   - un **mot de passe** valide.
5. Acceptez les valeurs par défaut restantes, puis sélectionnez **Suivant : Disques**.
6. Acceptez les valeurs par défaut, puis sélectionnez **Suivant : Mise en réseau**.
7. Assurez-vous que **myVNet** est sélectionné pour le réseau virtuel et que le sous-réseau est **myBackendSubnet**.
8. Acceptez les valeurs par défaut restantes, puis sélectionnez **Suivant : Gestion**.
9. Sélectionnez **Désactivé** pour désactiver les diagnostics de démarrage.
10. Acceptez les valeurs par défaut restantes, puis sélectionnez **Suivant : Avancé**.
11. Sélectionnez **Suivant : Balises**.
12. Sélectionnez **Suivant : Vérifier + créer**.
13. Vérifiez les paramètres sur la page de récapitulatif, puis sélectionnez **Créer**. La création de la machine virtuelle peut nécessiter plusieurs minutes. Patientez jusqu’à ce que le déploiement soit terminé avant de passer à la section suivante.

### <a name="install-iis"></a>Installer IIS

1. Ouvrez le service Cloud Shell et assurez-vous qu’il est défini sur **PowerShell**.
    ![Capture d’écran montrant une fenêtre de console ouverte Azure Cloud Shell qui utilise PowerShell.](./media/configure-application-gateway-with-private-frontend-ip/private-frontendip-3.png)
2. Exécutez la commande suivante pour installer IIS sur la machine virtuelle :

   ```azurepowershell
   Set-AzVMExtension `
   
     -ResourceGroupName myResourceGroupAG `
   
     -ExtensionName IIS `
   
     -VMName myVM `
   
     -Publisher Microsoft.Compute `
   
     -ExtensionType CustomScriptExtension `
   
     -TypeHandlerVersion 1.4 `

     -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `

     -Location CentralUS `

   ```



3. Créez une deuxième machine virtuelle et installez IIS à l’aide de la procédure que vous venez de terminer. Entrez myVM2 pour le nom et pour VMName dans Set-AzureRmVMExtension.

### <a name="add-backend-servers-to-backend-pool"></a>Ajouter des serveurs principaux pour le pool principal

1. Sélectionnez **Toutes les ressources**, puis **myAppGateway**.
2. Sélectionnez **Pools principaux**. Sélectionnez **appGatewayBackendPool**.
3. Sous **Type de cible**, sélectionnez **Machine virtuelle**, puis, sous **Cible**, sélectionnez la carte réseau virtuelle associée à myVM.
4. Répétez l’opération pour ajouter MyVM2.
   ![Capture d’écran montrant le volet Modifier le pool principal avec les Types de cible et les Cibles mis en surbrillance.](./media/configure-application-gateway-with-private-frontend-ip/private-frontendip-4.png)
5. sélectionnez **Enregistrer**.

## <a name="test-the-application-gateway"></a>Tester la passerelle d’application

1. Vérifiez votre adresse IP frontale affectée en cliquant sur la page **Configurations d’adresses IP frontales** dans le portail.
    ![Capture d’écran montrant le volet Configurations d’adresses IP frontales avec le type Privé mis en surbrillance.](./media/configure-application-gateway-with-private-frontend-ip/private-frontendip-5.png)
2. Copiez l’adresse IP privée, collez-la dans la barre d’adresse du navigateur d’une machine virtuelle se trouvant dans le même réseau virtuel ou locale, disposant d’une connectivité à ce réseau virtuel, puis essayez d’accéder à l’Application Gateway.

## <a name="next-steps"></a>Étapes suivantes

Si vous souhaitez surveiller l’intégrité de votre serveur principal, voir [Intégrité du serveur principal et journaux de diagnostic pour la passerelle Application Gateway](application-gateway-diagnostics.md).
