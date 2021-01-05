---
title: Installer une passerelle de données locale pour Azure Analysis Services | Microsoft Docs
description: Découvrez comment installer et configurer une passerelle de données locale pour vous connecter à des sources de données locales à partir d’un serveur Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/29/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 64bd9e4a4cf78d2628e946af30c2d290ff002cf7
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93081142"
---
# <a name="install-and-configure-an-on-premises-data-gateway"></a>Installer et configurer une passerelle de données locale

Une passerelle de données locale est requise lorsqu’un ou plusieurs serveurs Azure Analysis Services de la même région se connectent aux sources de données locales.  Bien que la passerelle que vous installez soit identique à celle utilisée par d’autres services tels que Power BI, Power Apps et Logic Apps, lors de son installation pour Azure Analysis Services, vous devez effectuer certaines étapes supplémentaires. Cet article sur l’installation est spécifique d’ **Azure Analysis Services**. 

Pour en savoir plus sur le fonctionnement d’Azure Analysis Services avec la passerelle, voir [Connexion à des sources de données locales](analysis-services-gateway.md). Pour en savoir plus sur les scénarios d’installation avancée et la passerelle en général, voir la [Documentation sur les passerelles de données locales](/data-integration/gateway/service-gateway-onprem).

## <a name="prerequisites"></a>Prérequis

**Configuration minimale requise :**

* .NET Framework 4.5
* Version 64 bits de Windows 8/Windows Server 2012 R2 (ou version ultérieure)

**Recommandé :**

* Processeur 8 cœurs
* 8 Go de mémoire
* Version 64 bits de Windows 8/Windows Server 2012 R2 (ou version ultérieure)

**Points importants à prendre en compte :**

* Pendant la configuration, lors de l’inscription de votre passerelle auprès d’Azure, la région par défaut de votre abonnement est sélectionnée. Vous pouvez choisir un autre abonnement et une autre région. Si vous avez des serveurs dans plusieurs régions, vous devez installer une passerelle pour chaque région. 
* La passerelle ne peut pas être installée sur un contrôleur de domaine.
* Une seule passerelle peut être installée sur un ordinateur.
* Installez la passerelle sur un ordinateur qui reste activé et qui ne se met pas en veille.
* N’installez pas la passerelle sur un ordinateur ne disposant que d’une connexion sans fil à votre réseau. Les performances peuvent être réduites.
* Lorsque vous installez la passerelle, le compte d’utilisateur à l’aide duquel vous êtes connecté à votre ordinateur doit avoir des privilèges de connexion en tant que service. Une fois l’installation terminée, le service de passerelle de données locale utilise le compte SERVICE\PBIEgwService NT pour se connecter en tant que service. Un autre compte peut être spécifié pendant l’installation ou dans Services une fois l’installation terminée. Assurez-vous que les paramètres de stratégie de groupe autorisent le compte avec lequel vous vous êtes connecté lors de l’installation et le compte de service que vous choisissez disposent de privilèges de connexion en tant que service.
* Connectez-vous à Azure avec le compte Azure AD du même [locataire](/previous-versions/azure/azure-services/jj573650(v=azure.100)#what-is-an-azure-ad-tenant) que l’abonnement dans lequel vous inscrivez la passerelle. Les comptes Azure B2B (invité) ne sont pas pris en charge lors de l’installation et de l’inscription d’une passerelle.
* Si les sources de données se trouvent sur un réseau virtuel (VNet) Azure, vous devez configurer la propriété de serveur [AlwaysUseGateway](analysis-services-vnet-gateway.md).

## <a name="download"></a>Téléchargement

 [Télécharger la passerelle](https://go.microsoft.com/fwlink/?LinkId=820925&clcid=0x409)

## <a name="install"></a>Installer

1. Exécutez le programme d’installation.

2. Sélectionnez **Passerelle de données locale**.

   ![Sélectionnez](media/analysis-services-gateway-install/aas-gateway-installer-select.png)

2. Sélectionnez un emplacement, acceptez les termes du contrat, puis cliquez sur **Installer**.

   ![Emplacement d’installation et termes du contrat de licence](media/analysis-services-gateway-install/aas-gateway-installer-accept.png)

3. Connectez-vous à Azure. Le compte doit se trouver dans l’Azure Active Directory de votre locataire. Ce compte est utilisé pour l’administrateur de passerelle. Les comptes Azure B2B (invité) ne sont pas pris en charge lors de l’installation et de l’inscription de la passerelle.

   ![Connexion à Azure](media/analysis-services-gateway-install/aas-gateway-installer-account.png)

   > [!NOTE]
   > Si vous vous connectez avec un compte de domaine, ce compte sera mappé à votre compte professionnel dans Azure AD. Votre compte professionnel sert de compte d’administrateur de passerelle.

## <a name="register"></a>Inscrire

Pour créer une ressource de passerelle dans Azure, vous devez inscrire l’instance locale que vous avez installée auprès du service cloud de passerelle. 

1.  Sélectionnez **Inscrivez une nouvelle passerelle sur cet ordinateur**.

    ![Capture d’écran mettant en évidence l’option Inscrivez une nouvelle passerelle sur cet ordinateur.](media/analysis-services-gateway-install/aas-gateway-register-new.png)

2. Saisissez un nom et une clé de récupération pour votre passerelle. Par défaut, la passerelle utilise la région par défaut de votre abonnement. Si vous souhaitez choisir une autre région, sélectionnez **Changer la région**.

    > [!IMPORTANT]
    > Enregistrez votre clé de récupération dans un endroit sûr. La clé de récupération est requise en cas de prise en charge, migration ou restauration d’une passerelle. 

   ![Inscrire](media/analysis-services-gateway-install/aas-gateway-register-name.png)


## <a name="create-an-azure-gateway-resource"></a>Créer une ressource de passerelle Azure

Une fois que vous avez installé et inscrit votre passerelle, vous devez créer une ressource de passerelle dans Azure. Connectez-vous à Azure avec le même compte que celui utilisé lors de l’inscription de la passerelle.

1. Dans portail Azure, cliquez sur **Créer une ressource** , recherchez **Passerelle de données locale** , puis cliquez sur **Créer**.

   ![Créer une ressource de passerelle](media/analysis-services-gateway-install/aas-gateway-new-azure-resource.png)

2. Dans **Créer une passerelle connexion** , entrez les paramètres suivants:

   * **Name**  : entrez un nom pour votre ressource de passerelle. 

   * **Abonnement** : sélectionnez l’abonnement Azure à associer à votre ressource de passerelle. 
   
     L’abonnement par défaut est basé sur le compte Azure que vous avez utilisé pour vous connecter.

   * **Groupe de ressources**  : Sélectionnez un groupe de ressources existant ou créez-en un.

   * **Emplacement** : sélectionnez la région dans laquelle vous avez inscrit votre passerelle.

   * **Nom de l’installation**  : si votre installation de passerelle n’est pas encore sélectionnée, sélectionnez la passerelle que vous installée sur votre ordinateur et inscrite. 

     Une fois ces opérations effectuées, cliquez sur **Créer**.

## <a name="connect-gateway-resource-to-server"></a>Connecter la ressource de passerelle au serveur

> [!NOTE]
> La connexion à une ressource de passerelle dans un abonnement différent de celui de votre serveur n’est pas prise en charge dans le portail. Cependant, elle est prise en charge par PowerShell.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans la vue d’ensemble du serveur Azure Analysis Services, cliquez sur **Passerelle de données locale**.

   ![Connecter le serveur à la passerelle](media/analysis-services-gateway-install/aas-gateway-connect-server.png)

2. Dans la zone **Pick an On-Premises Data Gateway to connect** (Choisir une passerelle de données locale à connecter), sélectionnez votre ressource de passerelle, puis cliquez sur **Connect selected gateway** (Connecter la passerelle sélectionnée).

   ![Connecter le serveur à la ressource de passerelle](media/analysis-services-gateway-install/aas-gateway-connect-resource.png)

    > [!NOTE]
    > Si votre passerelle n’apparaît pas dans la liste, votre serveur se trouve probablement dans une autre région que celle spécifiée lors de l’inscription de la passerelle.

    Une fois la connexion entre le serveur et la ressource de passerelle établie, l’état affiché est **Connecté**.


    ![Réussite de la connexion du serveur à la ressource de passerelle](media/analysis-services-gateway-install/aas-gateway-connect-success.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Utilisez [Get-AzResource](/powershell/module/az.resources/get-azresource) pour récupérer la valeur ResourceID de la passerelle. Ensuite, connectez la ressource de passerelle à un serveur existant ou nouveau en spécifiant **-GatewayResourceID** dans [Set-AzAnalysisServicesServer](/powershell/module/az.analysisservices/set-azanalysisservicesserver) ou [New-AzAnalysisServicesServer](/powershell/module/az.analysisservices/new-azanalysisservicesserver).

Pour récupérer l’ID de ressource de la passerelle :

```azurepowershell-interactive
Connect-AzAccount -Tenant $TenantId -Subscription $subscriptionIdforGateway -Environment "AzureCloud"
$GatewayResourceId = $(Get-AzResource -ResourceType "Microsoft.Web/connectionGateways" -Name $gatewayName).ResourceId  

```

Pour configurer un serveur existant :

```azurepowershell-interactive
Connect-AzAccount -Tenant $TenantId -Subscription $subscriptionIdforAzureAS -Environment "AzureCloud"
Set-AzAnalysisServicesServer -ResourceGroupName $RGName -Name $servername -GatewayResourceId $GatewayResourceId

```
---

Vous avez terminé. Si vous devez ouvrir des ports ou effectuer des opérations de dépannage, veillez à consulter la page [Passerelle de données locale](analysis-services-gateway.md).

## <a name="next-steps"></a>Étapes suivantes

* [Gérer Analysis Services](analysis-services-manage.md)   
* [Obtenir les données d’Azure Analysis Services](analysis-services-connect.md)   
* [Utiliser la passerelle pour les sources de données sur un réseau virtuel Azure](analysis-services-vnet-gateway.md)