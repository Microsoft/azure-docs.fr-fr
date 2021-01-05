---
title: Redirection HTTP vers HTTPS dans le portail - Azure Application Gateway
description: Apprenez à créer une passerelle d’application avec redirection du trafic de HTTP vers HTTPS à l’aide du portail Azure.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/13/2019
ms.author: victorh
ms.openlocfilehash: 67153fa750fee765dcaa1072eec87a2f6169b918
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397278"
---
# <a name="create-an-application-gateway-with-http-to-https-redirection-using-the-azure-portal"></a>Créer une passerelle d’application avec redirection de HTTP vers HTTPS à l’aide du portail Azure

Vous pouvez utiliser le portail Azure pour créer une [passerelle d’application](overview.md) avec un certificat pour la terminaison TLS. Une règle de routage est utilisée pour rediriger le trafic HTTP vers le port HTTPS dans votre passerelle d’application. Dans cet exemple, vous créez également un [groupe de machines virtuelles identiques](../virtual-machine-scale-sets/overview.md) pour le pool backend de la passerelle d’application qui contient deux instances de machine virtuelle.

Dans cet article, vous apprendrez comment :

* Créer un certificat auto-signé
* Configurer un réseau
* Créer une passerelle d’application avec le certificat
* Ajouter un écouteur et une règle de redirection
* Créer un groupe de machines virtuelles identiques avec le pool backend par défaut

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour créer un certificat et installer IIS, ce tutoriel requiert le module Azure PowerShell version 1.0.0 ou ultérieure. Exécutez `Get-Module -ListAvailable Az` pour trouver la version. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Pour exécuter les commandes de ce didacticiel, vous devez également exécuter `Login-AzAccount` pour créer une connexion avec Azure.

## <a name="create-a-self-signed-certificate"></a>Créer un certificat auto-signé

Dans un environnement de production, vous devez importer un certificat valide signé par un fournisseur approuvé. Pour ce didacticiel, vous créez un certificat auto-signé à l’aide de [New-SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate). Vous pouvez utiliser [Export-PfxCertificate](/powershell/module/pkiclient/export-pfxcertificate) avec l’empreinte numérique qui a été retournée pour exporter un fichier pfx du certificat.

```powershell
New-SelfSignedCertificate `
  -certstorelocation cert:\localmachine\my `
  -dnsname www.contoso.com
```

Un résultat comme l’exemple suivant devrait s’afficher :

```
PSParentPath: Microsoft.PowerShell.Security\Certificate::LocalMachine\my

Thumbprint                                Subject
----------                                -------
E1E81C23B3AD33F9B4D1717B20AB65DBB91AC630  CN=www.contoso.com
```

Utilisez l’empreinte numérique pour créer le fichier pfx :

```powershell
$pwd = ConvertTo-SecureString -String "Azure123456!" -Force -AsPlainText
Export-PfxCertificate `
  -cert cert:\localMachine\my\E1E81C23B3AD33F9B4D1717B20AB65DBB91AC630 `
  -FilePath c:\appgwcert.pfx `
  -Password $pwd
```

## <a name="create-an-application-gateway"></a>Créer une passerelle Application Gateway

Un réseau virtuel est nécessaire pour la communication entre les ressources que vous créez. Deux sous-réseaux sont créés dans cet exemple : une pour la passerelle d’application et l’autre pour les serveurs principaux. Vous pouvez créer un réseau virtuel en même temps que la passerelle d’application.

1. Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).
2. Cliquez sur  **Créer une ressource** dans le coin supérieur gauche du portail Azure.
3. Sélectionnez **Mise en réseau** , puis sélectionnez **Application Gateway** dans la liste de suggestions.
4. Entrez ces valeurs pour la passerelle d’application :

   - *myAppGateway* : pour le nom de la passerelle d’application.
   - *myResourceGroupAG* : pour le nouveau groupe de ressources.

     ![Créer une nouvelle passerelle d’application](./media/create-url-route-portal/application-gateway-create.png)

5. Acceptez les valeurs par défaut pour les autres paramètres, puis cliquez sur **OK**.
6. Cliquez sur **Choisir un réseau virtuel** , cliquez sur **Créer nouveau** , puis entrez ces valeurs pour le réseau virtuel :

   - *myVNet* : pour le nom du réseau virtuel.
   - *10.0.0.0/16* : pour l’espace d’adressage du réseau virtuel.
   - *myAGSubnet* : pour le nom du sous-réseau.
   - *10.0.0.0/24* : pour l’espace d’adressage du sous-réseau.

     ![Création d’un réseau virtuel](./media/create-url-route-portal/application-gateway-vnet.png)

7. Cliquez sur **OK** pour créer le réseau virtuel et le sous-réseau.
8. Sous **Configuration d’adresse IP frontale** , vérifiez que **Type d’adresse IP** est défini sur **public** et que l’option **Créer** est sélectionnée. Entrez *myAGPublicIPAddress* pour le nom. Acceptez les valeurs par défaut pour les autres paramètres, puis cliquez sur **OK**.
9. Dans **Configuration de l’écouteur** , sélectionnez **HTTPS**. Ensuite, choisissez **Sélectionner un fichier** et accédez au fichier *c:\appgwcert.pfx*. Sélectionnez **Ouvrir**.
10. Saisissez *appgwcert* comme nom de certificat et *Azure123456!* pour le mot de passe.
11. Laissez le pare-feu d’applications Web désactivé, puis sélectionnez **OK**.
12. Passez en revue les paramètres sur la page de résumé, puis cliquez sur **OK** pour créer les ressources réseau et la passerelle d’application. La création de la passerelle d’application peut prendre plusieurs minutes. Patientez jusqu’à ce que le déploiement soit terminé avant de passer à la section suivante.

### <a name="add-a-subnet"></a>Ajouter un sous-réseau

1. Sélectionnez **Toutes les ressources** dans le menu de gauche, puis cliquez sur **myVNet** dans la liste des ressources.
2. Cliquez sur **Sous-réseaux** , puis sur **Sous-réseau**.

    ![Créer un sous-réseau](./media/create-url-route-portal/application-gateway-subnet.png)

3. Saisissez *myBackendSubnet* comme nom du sous-réseau.
4. Indiquez *10.0.2.0/24* comme plage d’adresses, puis sélectionnez **OK**.

## <a name="add-a-listener-and-redirection-rule"></a>Ajouter un écouteur et une règle de redirection

### <a name="add-the-listener"></a>Ajouter l’écouteur

Tout d’abord, ajoutez l’écouteur appelé *myListener* pour le port 80.

1. Ouvrez le groupe de ressources **myResourceGroupAG** , puis sélectionnez **myAppGateway**.
2. Sélectionnez **Écouteurs** , puis **+ De base**.
3. Saisissez *MyListener* pour le nom.
4. Saisissez *httpPort* pour le nouveau nom de port frontal, et *80* pour le port.
5. Assurez-vous que le protocole est défini sur **HTTP** , puis sélectionnez **OK**.

### <a name="add-a-routing-rule-with-a-redirection-configuration"></a>Ajouter une règle d’acheminement avec une configuration de redirection

1. Dans **myAppGateway** , sélectionnez **Règles** , puis **+Règle de routage des demandes**.
2. Pour le **Nom de la règle** , tapez *Rule2*.
3. Vérifiez que **MyListener** est sélectionné pour l’écouteur.
4. Cliquez sur l’onglet **Cibles principales** et sélectionnez **Type de cible** en tant que *Redirection*.
5. Pour **Type de redirection** , sélectionnez **Permanent**.
6. Pour **Cible de redirection** , sélectionnez **Écouteur**.
7. Vérifiez que l’ **écouteur cible** est défini sur **appGatewayHttpListener**.
8. Pour **Inclure la chaîne de requête** et **Inclure le chemin d’accès** , sélectionnez *Oui*.
9. Sélectionnez **Ajouter**.

## <a name="create-a-virtual-machine-scale-set"></a>Créer un groupe de machines virtuelles identiques

Dans cet exemple, vous créez un groupe de machines virtuelles identiques pour fournir des serveurs pour le pool principal dans la passerelle d’application.

1. Sélectionnez **+ Créer une ressource** en haut à gauche du portail.
2. Sélectionnez **Calcul**.
3. Dans la zone de recherche, tapez *groupe identique* et appuyez sur « Entrée ».
4. Sélectionnez **Groupe de machines virtuelles identiques** , puis **Créer**.
5. Dans **Nom du groupe de machines virtuelles identiques** , saisissez *myvmss*.
6. Pour l’image de disque du système d’exploitation, ** vérifiez que **Windows Server 2016 Datacenter** est sélectionné.
7. Pour le **Groupe de ressources** , sélectionnez **myResourceGroupAG**.
8. Pour le **Nom d’utilisateur** , saisissez *azureuser*.
9. Pour le **mot de passe** , saisissez *Azure123456!* , puis confirmez le mot de passe.
10. Pour le **nombre d’instances** , vérifiez la valeur est définie sur **2**.
11. Pour la **taille de l’instance** , sélectionnez **D2s_v3**.
12. Dans la section **Mise en réseau** , vérifiez que le paramètre **Choisir les options d’équilibrage de charge** est défini sur **Passerelle d’application**.
13. Vérifiez que la **Passerelle d’application** est définie sur **myAppGateway**.
14. Vérifiez que le **sous-réseau** est défini sur **myBackendSubnet**.
15. Sélectionnez **Create** (Créer).

### <a name="associate-the-scale-set-with-the-proper-backend-pool"></a>Associer le groupe identique au pool principal approprié

L’IU du portail du groupe de machines virtuelles identiques crée un nouveau pool principal pour le groupe identique. Toutefois, vous voulez associer ce groupe à votre pool existant appGatewayBackendPool.

1. Ouvrez le groupe de ressources **myResourceGroupAG**.
2. Sélectionnez **myAppGateway**.
3. Sélectionnez **Pools principaux**.
4. Sélectionnez **myAppGatewaymyvmss**.
5. Sélectionnez **Remove all targets from backend pool (Supprimer toutes les cibles du pool principal)** .
6. Sélectionnez **Enregistrer**.
7. Une fois ce processus terminé, sélectionnez le pool principal **myAppGatewaymyvmss** , puis **Supprimer**. Ensuite, cliquez sur **OK** pour confirmer.
8. Sélectionnez **appGatewayBackendPool**.
9. Dans **Cibles** , sélectionnez **VMSS**.
10. Dans **VMSS** , sélectionnez **myvmss**.
11. Dans **Configurations d’interface réseau** , sélectionnez **myvmssNic**.
12. Sélectionnez **Enregistrer**.

### <a name="upgrade-the-scale-set"></a>Mettre à niveau le groupe identique

Pour finir, vous devez mettre à niveau le groupe identique en intégrant ces modifications.

1. Sélectionnez le groupe identique **myvmss**.
2. Dans **Paramètres** , sélectionnez **Instances**.
3. Sélectionnez les deux instances, puis **Mettre à niveau**.
4. Sélectionnez **Oui** pour confirmer.
5. Une fois cette opération terminée, revenez à **myAppGateway** et sélectionnez **Pools principaux**. Vous devriez normalement constater que **appGatewayBackendPool** a désormais deux cibles, et que **myAppGatewaymyvmss** n’en a aucune.
6. Sélectionnez **myAppGatewaymyvmss** , puis **Supprimer**.
7. Sélectionnez **OK** pour confirmer.

### <a name="install-iis"></a>Installer IIS

Pour installer facilement IIS sur le groupe identique, vous pouvez utiliser PowerShell. Dans le portail, cliquez sur l’icône Cloud Shell et vérifiez que **PowerShell** est sélectionné.

Copiez le code suivant dans la fenêtre PowerShell, puis appuyez sur Entrée.

```azurepowershell
$publicSettings = @{ "fileUris" = (,"https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/appgatewayurl.ps1"); 
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File appgatewayurl.ps1" }
$vmss = Get-AzVmss -ResourceGroupName myResourceGroupAG -VMScaleSetName myvmss
Add-AzVmssExtension -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.8 `
  -Setting $publicSettings
Update-AzVmss `
  -ResourceGroupName myResourceGroupAG `
  -Name myvmss `
  -VirtualMachineScaleSet $vmss
```

### <a name="upgrade-the-scale-set"></a>Mettre à niveau le groupe identique

Après avoir modifié les instances avec IIS, vous devez encore une fois mettre à niveau le groupe identique en intégrant cette modification.

1. Sélectionnez le groupe identique **myvmss**.
2. Dans **Paramètres** , sélectionnez **Instances**.
3. Sélectionnez les deux instances, puis **Mettre à niveau**.
4. Sélectionnez **Oui** pour confirmer.

## <a name="test-the-application-gateway"></a>Tester la passerelle d’application

Vous pouvez obtenir l’adresse IP publique d’application à partir de la page de présentation de la passerelle d’application.

1. Sélectionnez **myAppGateway**.
2. Sur la page de **présentation** , prenez note de l’adresse IP indiquée dans **Adresses IP publiques frontales**.

3. Copiez l’adresse IP publique, puis collez-la dans la barre d’adresses de votre navigateur. Par exemple : http://52.170.203.149

   ![Avertissement de sécurité](./media/redirect-http-to-https-powershell/application-gateway-secure.png)

4. Pour accepter l’avertissement de sécurité si vous avez utilisé un certificat auto-signé, sélectionnez **Détails** , puis **Atteindre la page web**. Votre site IIS sécurisé apparaît maintenant comme dans l’exemple suivant :

   ![Tester l’URL de base dans la passerelle d’application](./media/redirect-http-to-https-powershell/application-gateway-iistest.png)

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [créer une passerelle d’application avec redirection interne](redirect-internal-site-powershell.md).