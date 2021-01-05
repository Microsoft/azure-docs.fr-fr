---
title: Créer une forêt de ressources Azure AD Domain Services à l’aide d’Azure PowerShell | Microsoft Docs
description: Cet article vous apprend à créer et à configurer une forêt de ressources Azure Active Directory Domain Services et une forêt sortante dans un environnement Active Directory Domain Services local à l’aide d’Azure PowerShell.
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 07/27/2020
ms.author: justinha
ms.openlocfilehash: ebfc2476b7955b926f86094de03973155386eb8f
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619965"
---
# <a name="create-an-azure-active-directory-domain-services-resource-forest-and-outbound-forest-trust-to-an-on-premises-domain-using-azure-powershell"></a>Créer une forêt de ressources Azure Active Directory Domain Services et une approbation de forêt sortante vers un domaine local à l’aide d’Azure PowerShell

Dans les environnements ne permettant pas la synchronisation des hachages de mot de passe, ou en présence d’utilisateurs se connectant exclusivement à l’aide de cartes à puce et ne connaissant pas leur mot de passe, vous pouvez utiliser une forêt de ressources dans Azure Active Directory Domain Services (Azure AD DS). Une forêt de ressources utilise une approbation unidirectionnelle sortante entre Azure AD DS et un ou plusieurs environnements AD DS locaux. Cette relation d’approbation permet aux utilisateurs, applications et ordinateurs de s’authentifier auprès d’un domaine local à partir du domaine managé Azure AD DS. Dans une forêt de ressources, les codes de hachage de mot de passe ne sont jamais synchronisés.

![Diagramme d'approbation de forêt entre Azure AD DS et les instances AD DS locales](./media/concepts-resource-forest/resource-forest-trust-relationship.png)

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Créer une forêt de ressources Azure AD DS à l’aide d’Azure PowerShell
> * Créer une approbation de forêt sortante unidirectionnelle dans le domaine managé à l’aide d’Azure PowerShell
> * Configurer DNS dans un environnement AD DS local pour prendre en charge la connectivité du domaine managé
> * Créer une approbation de forêt unidirectionnelle entrante dans un environnement AD DS local
> * Tester et valider la relation d’approbation à des fins d'authentification et d’accès aux ressources

Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

> [!IMPORTANT]
> Les forêts de ressources de domaine managé ne prennent actuellement pas en charge Azure HDInsight et Azure Files. Les forêts d’utilisateurs de domaine managé par défaut prennent en charge ces deux services supplémentaires.

## <a name="prerequisites"></a>Prérequis

Pour faire ce qui est décrit dans cet article, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].

* Installez et configurez Azure PowerShell.
    * Si nécessaire, suivez les instructions pour [installer le module Azure PowerShell et vous connecter à votre abonnement Azure](/powershell/azure/install-az-ps).
    * Veillez à vous connecter à votre abonnement Azure à l’aide de l’applet de commande [Connect-AzAccount][Connect-AzAccount].
* Installez et configurez Azure AD PowerShell.
    * Si nécessaire, suivez les instructions pour [installer le module Azure AD PowerShell et vous connecter à Azure AD](/powershell/azure/active-directory/install-adv2).
    * Veillez à vous connecter à votre locataire Azure AD à l’aide de l’applet de commande [Connect-AzureAD][Connect-AzureAD].
* Vous devez disposer des privilèges d’*Administrateur global* dans votre locataire Azure AD pour activer Azure AD DS.
* Vous avez besoin de privilèges de *Contributeur* dans votre abonnement Azure pour créer les ressources Azure AD DS nécessaires.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Dans cet article, vous créez et vous configurez une approbation de forêt sortante à partir d'un domaine managé avec le portail Azure. Pour commencer, connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="deployment-process"></a>Processus de déploiement

Il s’agit d’un processus en plusieurs parties qui permet de créer une forêt de ressources de domaine managé et la relation d’approbation avec un AD DS local. La procédure générale suivante crée votre environnement hybride approuvé :

1. Créez un principal de service de domaine managé.
1. Créez une forêt de ressources de domaine managé.
1. Créez une connectivité réseau hybride à l’aide d’un VPN site à site ou d’Express Route.
1. Créez le côté domaine managé de la relation d’approbation.
1. Créez le côté AD DS local de la relation d’approbation.

Avant de commencer, assurez-vous de comprendre les [considérations relatives au réseau, l’attribution d’un nom à la forêt et la configuration DNS requise](tutorial-create-forest-trust.md#networking-considerations). Une fois que la forêt du domaine managé est déployée, vous ne pouvez pas modifier son nom.

## <a name="create-the-azure-ad-service-principal"></a>Créez le principal du service Azure AD

Azure AD DS nécessite un principal de service pour synchroniser les données depuis Azure AD. Ce principal doit être créé dans votre tenant Azure AD avant la création de la forêt de ressources de domaine managée.

Créez un principal du service Azure AD pour permettre à Azure AD DS de communiquer et de s’authentifier. Un ID d’application spécifique est utilisé. Il se nomme *Domain Controller Services* et son ID est *6ba9a5d4-8456-4118-b521-9c5ca10cdf84*. Ne modifiez pas cet ID d’application.

Créez un principal du service Azure AD à l’aide de l’applet de commande [New-AzureADServicePrincipal][New-AzureADServicePrincipal] :

```powershell
New-AzureADServicePrincipal -AppId "6ba9a5d4-8456-4118-b521-9c5ca10cdf84"
```

## <a name="create-a-managed-domain-resource-forest"></a>Créer une forêt de ressources de domaine managé

Pour créer une forêt de ressources de domaine managé, vous devez utiliser le script `New-AzureAaddsForest`. Ce script fait partie d’un ensemble de commandes plus vaste qui prend en charge la création et la gestion des forêts de ressources de domaine managé, notamment la création d’une forêt de liaison unidirectionnelle dans une prochaine section. Ces scripts sont disponibles à partir de [PowerShell Gallery](https://www.powershellgallery.com/) et sont signés numériquement par l’équipe d’ingénieurs d’Azure AD.

1. Créez tout d’abord un groupe de ressources avec la cmdlet [New-AzResourceGroup][New-AzResourceGroup]. Dans l’exemple suivant, le groupe de ressources est nommé *myResourceGroup* et est créé dans la région *westus*. Utilisez votre propre nom et la région souhaitée :

    ```azurepowershell
    New-AzResourceGroup `
      -Name "myResourceGroup" `
      -Location "WestUS"
    ```

1. Installez le script `New-AaddsResourceForestTrust` à partir de [PowerShell Gallery][powershell-gallery] à l’aide de la cmdlet [Install-script][Install-Script] :

    ```powershell
    Install-Script -Name New-AaddsResourceForestTrust
    ```

1. Passez en revue les paramètres suivants dans le script `New-AzureAaddsForest`. Assurez-vous que vous disposez également des modules **Azure PowerShell** et **Azure AD PowerShell** prérequis. Assurez-vous d’avoir planifié les exigences du réseau virtuel pour fournir l’application et la connectivité locale.

    | Nom                         | Paramètre de script          | Description |
    |:-----------------------------|---------------------------|:------------|
    | Abonnement                 | *-azureSubscriptionId*    | ID d’abonnement utilisé pour la facturation Azure AD DS. Vous pouvez obtenir la liste des abonnements en utilisant la cmdlet [Get-AzureRMSubscription][Get-AzureRMSubscription]. |
    | Groupe de ressources               | *-aaddsResourceGroupName* | Nom du groupe de ressources pour le domaine managé et les ressources associées. |
    | Emplacement                     | *-aaddsLocation*          | Région Azure devant héberger votre domaine managé. Pour savoir quelles régions sont disponibles, consultez [Régions prises en charge pour Azure AD DS.](https://azure.microsoft.com/global-infrastructure/services/?products=active-directory-ds&regions=all) |
    | Administrateur Azure AD DS    | *-aaddsAdminUser*         | Nom de l’utilisateur principal du premier administrateur de domaine managé. Ce compte doit être un compte d'utilisateur cloud déjà existant dans votre Azure Active Directory. L’utilisateur et l’utilisateur qui exécute le script sont ajoutés au groupe *AAD DC Administrators*. |
    | Nom de domaine Azure AD DS      | *-aaddsDomainName*        | FQDN du domaine managé basé sur les instructions précédentes relatives au choix d’un nom de forêt. |

    Le script `New-AzureAaddsForest` peut créer le réseau virtuel Azure et le sous-réseau Azure AD DS s’ils n’existent pas encore. Le script peut éventuellement créer les sous-réseaux de la charge de travail, le cas échéant :

    | Nom                              | Paramètre de script                  | Description |
    |:----------------------------------|:----------------------------------|:------------|
    | Nom du réseau virtuel              | *-aaddsVnetName*                  | Nom du réseau virtuel pour le domaine managé.|
    | Espace d’adressage                     | *-aaddsVnetCIDRAddressSpace*      | Plage d’adresses du réseau virtuel en notation CIDR (si création du réseau virtuel).|
    | Nom de sous-réseau Azure AD DS           | *-aaddsSubnetName*                | Nom du sous-réseau du réseau virtuel *aaddsVnetName* hébergeant le domaine managé. Ne déployez pas vos propres machines virtuelles et charges de travail dans ce sous-réseau. |
    | Plage d’adresses Azure AD DS         | *-aaddsSubnetCIDRAddressRange*    | Plage d’adresses de sous-réseau en notation CIDR pour l’instance AAD DS, par exemple *192.168.1.0/24*. Le plage d’adresses doit être contenue dans la plage d’adresses du réseau virtuel et être différente des autres sous-réseaux. |
    | Nom du sous-réseau de la charge de travail (facultatif)   | *-workloadSubnetName*             | Nom facultatif d’un sous-réseau dans le réseau virtuel *aaddsVnetName* à créer pour vos propres charges de travail d’application. La machines virtuelles et les applications peuvent également être connectées à un réseau virtuel Azure appairé à la place. |
    | Plage d’adresses de la charge de travail (facultatif) | *-workloadSubnetCIDRAddressRange* | Plage d’adresses de sous-réseau facultative en notation CIDR pour la charge de travail d’application, par exemple *192.168.2.0/24*. Le plage d’adresses doit être contenue dans la plage d’adresses du réseau virtuel et être différente des autres sous-réseaux.|

1. À présent, créez une forêt de ressources de domaine managé à l’aide du script `New-AzureAaaddsForest`. L’exemple suivant crée une forêt nommée *addscontoso.com* et un sous-réseau de charge de travail. Indiquez vos propres noms de paramètre et plages d’adresses IP ou réseaux virtuels existants.

    ```azurepowershell
    New-AzureAaddsForest `
        -azureSubscriptionId <subscriptionId> `
        -aaddsResourceGroupName "myResourceGroup" `
        -aaddsLocation "WestUS" `
        -aaddsAdminUser "contosoadmin@contoso.com" `
        -aaddsDomainName "aaddscontoso.com" `
        -aaddsVnetName "myVnet" `
        -aaddsVnetCIDRAddressSpace "192.168.0.0/16" `
        -aaddsSubnetName "AzureADDS" `
        -aaddsSubnetCIDRAddressRange "192.168.1.0/24" `
        -workloadSubnetName "myWorkloads" `
        -workloadSubnetCIDRAddressRange "192.168.2.0/24"
    ```

    La création de la forêt de ressources de domaine managé et la prise en charge des ressources prennent un certain temps. Autorisez l’exécution du script. Passez à la section suivante pour configurer votre connectivité réseau locale pendant que la forêt de ressources Azure AD est en cours de configuration en arrière-plan.

## <a name="configure-and-validate-network-settings"></a>Configurer et valider les paramètres réseau

Pendant que le déploiement du domaine managé suit son cours, configurez et validez la connectivité réseau hybride avec le centre de données local. Vous avez également besoin d’une machine virtuelle de gestion à utiliser avec le domaine managé pour une maintenance régulière. Une partie de la connectivité hybride existe peut-être déjà dans votre environnement ou vous devez peut-être travailler avec d’autres personnes de votre équipe pour configurer les connexions.

Avant de commencer, assurez-vous de comprendre les [considérations et recommandations relatives au réseau](tutorial-create-forest-trust.md#networking-considerations).

1. Créez la connectivité hybride de votre réseau local à Azure à l’aide d’une connexion VPN Azure ou Azure ExpressRoute. La configuration du réseau hybride n’entre pas dans le cadre de cette documentation et elle existe peut-être déjà dans votre environnement. Pour plus de détails sur des scénarios spécifiques, consultez les articles suivants :

    * [VPN de site à site Azure](../vpn-gateway/vpn-gateway-about-vpngateways.md).
    * [Vue d’ensemble d’Azure ExpressRoute](../expressroute/expressroute-introduction.md).

    > [!IMPORTANT]
    > Si vous créez une connexion directement vers votre réseau virtuel de domaine managé, utilisez un sous-réseau de passerelle distinct. Ne créez pas la passerelle dans le sous-réseau du domaine managé.

1. Pour administrer un domaine managé, vous devez créer une machine virtuelle de gestion, la joindre au domaine managé, puis installer les outils de gestion AD DS requis.

    Lors du déploiement de la forêt de ressources de domaine managé, [créez une machine virtuelle Windows Server](./join-windows-vm.md), puis [installez les outils de gestion AD DS de base](./tutorial-create-management-vm.md) pour installer les outils de gestion nécessaires. Attendez une des prochaines étapes après la réussite du déploiement du domaine pour joindre la machine virtuelle de gestion au domaine managé.

1. Validez la connectivité réseau entre votre réseau local et le réseau virtuel Azure.

    * Confirmez que le contrôleur de domaine local peut se connecter à la machine virtuelle managée à l’aide de `ping` ou du bureau à distance, par exemple.
    * Vérifiez que votre machine virtuelle de gestion peut se connecter à vos contrôleurs de domaine locaux, de nouveau à l’aide d’un utilitaire, tel que `ping`.

1. Sur le portail Azure, recherchez et sélectionnez **Azure AD Domain Services**. Choisissez votre domaine managé, tel que *aaddscontoso.com* et attendez que le statut soit signalé comme **En cours d’exécution**.

    Lors de l’exécution, [mettez à jour les paramètres DNS du réseau virtuel Azure](tutorial-create-instance.md#update-dns-settings-for-the-azure-virtual-network), puis [activez les comptes d’utilisateur pour Azure AD DS](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds) pour finaliser les configurations de votre forêt de ressources de domaine managé.

1. Prenez note des adresses DNS affichées sur la page de présentation. Vous avez besoin de ces adresses lors de la configuration du côté Active Directory local de la relation d’approbation dans une section suivante.
1. Redémarrez la machine virtuelle de gestion afin qu’elle reçoive les nouveaux paramètres DNS et qu’elle [joigne la machine virtuelle au domaine managé](join-windows-vm.md#join-the-vm-to-the-managed-domain).
1. Une fois la machine virtuelle de gestion jointe au domaine managé, reconnectez-vous à l’aide du bureau à distance.

    À partir d’une invite de commandes, utilisez `nslookup` et le nom de la forêt de ressources de domaine managé pour valider la résolution de noms pour la forêt de ressources.

    ```console
    nslookup aaddscontoso.com
    ```

    La commande doit retourner deux adresses IP pour la forêt de ressources.

## <a name="create-the-forest-trust"></a>Créer l’approbation de forêt

L’approbation de forêt se fait en deux parties : l’approbation de forêt sortante unidirectionnelle dans la forêt de ressources de domaine managé et l’approbation de la forêt entrante unidirectionnelle dans la forêt AD DS locale. Vous créez manuellement les deux côtés de cette relation d’approbation. Lorsque les deux côtés sont créés, les utilisateurs et les ressources peuvent s’authentifier avec succès à l’aide de l’approbation de forêt. Une forêt de ressources de domaine managé prend en charge jusqu’à cinq approbations de forêt sortante unidirectionnelle vers des forêts locales.

### <a name="create-the-managed-domain-side-of-the-trust-relationship"></a>Créer le côté domaine managé de la relation d’approbation

Utilisez le script `Add-AaddsResourceForestTrust` pour créer le côté domaine managé de la relation d’approbation. Tout d’abord, installez le script `Add-AaddsResourceForestTrust` à partir de [PowerShell Gallery][powershell-gallery] à l’aide de la cmdlet [Install-script][Install-Script] :

```powershell
Install-Script -Name Add-AaddsResourceForestTrust
```

Maintenant, apportez les informations suivantes au script :

| Nom                               | Paramètre de script     | Description |
|:-----------------------------------|:---------------------|:------------|
| Nom de domaine Azure AD DS            | *-ManagedDomainFqdn* | FQDN du domaine managé, par exemple *aaddscontoso.com* |
| Nom de domaine AD DS local      | *-TrustFqdn*         | FQDN de la forêt approuvée, par exemple *onprem.contoso.com* |
| Nom convivial de l’approbation                | *-TrustFriendlyName* | Nom convivial de la relation d’approbation. |
| Adresses IP DNS AD DS locales | *-TrustDnsIPs*       | Une liste, séparée par des virgules, des adresses de serveur DNS IPv4 pour le domaine approuvé listé. |
| Mot de passe d'approbation                     | *-TrustPassword*     | Mot de passe complexe pour la relation d’approbation. Ce mot de passe est également entré lors de la création de l’approbation entrante unidirectionnelle dans l’AD DS local. |
| Informations d'identification                        | *-Credentials*       | Informations d'identification permettant de s’authentifier dans Azure. L’utilisateur doit être dans le *groupe AAD DC Administrators*. S’il n’est pas fourni, le script vous invite à l’authentification. |

L’exemple suivant crée une relation d’approbation nommée *myAzureADDSTrust* vers *onprem.contoso.com*. Utilisez vos propres noms de paramètre et vos propres mots de passe.

```azurepowershell
Add-AaddsResourceForestTrust `
    -ManagedDomainFqdn "aaddscontoso.com" `
    -TrustFqdn "onprem.contoso.com" `
    -TrustFriendlyName "myAzureADDSTrust" `
    -TrustDnsIPs "10.0.1.10,10.0.1.11" `
    -TrustPassword <complexPassword>
```

> [!IMPORTANT]
> N’oubliez pas votre mot de passe d’approbation. Vous devez utiliser le même mot de passe lors de la création du côté local de l’approbation.

## <a name="configure-dns-in-the-on-premises-domain"></a>Configurer DNS dans le domaine local

Pour résoudre correctement le domaine managé à partir de l’environnement local, vous serez peut-être amené à ajouter des redirecteurs aux serveurs DNS existants. Si vous n’avez pas configuré l’environnement local de manière à ce qu’il communique avec le domaine managé, effectuez les étapes suivantes à partir d’une station de travail de gestion pour le domaine AD DS local :

1. Sélectionnez **Démarrer | Outils d'administration | DNS**.
1. Cliquez avec le bouton droit sur le serveur DNS, comme *myAD01*, sélectionnez **Propriétés**.
1. Sélectionnez **Redirecteurs**, puis **Modifier** pour ajouter des redirecteurs supplémentaires.
1. Ajoutez les adresses IP du domaine managé, telles que *10.0.1.4* et *10.0.1.5*.
1. À partir d’une invite de commandes locale, validez la résolution de noms à l’aide de **nslookup** du nom de domaine managé de la forêt de ressources de domaine managé. Par exemple, `Nslookup aaddscontoso.com` doit retourner les deux adresse IP pour la forêt de ressources de domaine managé.

## <a name="create-inbound-forest-trust-in-the-on-premises-domain"></a>Créer une approbation de forêt entrante dans le domaine local

Le domaine AD DS local nécessite une approbation de forêt entrante pour le domaine managé. Cette approbation doit être créée manuellement dans le domaine AD DS local ; sa création n'est pas possible à partir du portail Azure.

Pour configurer l’approbation entrante sur le domaine AD DS local, procédez comme suit à partir d’une station de travail de gestion pour le domaine AD DS local :

1. Sélectionnez **Démarrer | Outils d'administration | Domaines et approbations Active Directory**
1. Cliquez avec le bouton droit sur un domaine, par exemple *onprem.contoso.com*, sélectionnez **Propriétés**.
1. Sélectionnez l'onglet **Approbations**, puis **Nouvelle approbation**.
1. Entrez le nom du domaine managé, tel que *aaddscontoso.com*, puis sélectionnez **Suivant**
1. Sélectionnez l’option permettant de créer une **approbation de forêt**, puis une approbation **unidirectionnelle : entrante**.
1. Choisissez de créer l’approbation pour **ce domaine uniquement**. À l’étape suivante, vous allez créer l’approbation dans le portail Azure pour le domaine managé.
1. Choisissez d’utiliser l'**authentification à l'échelle de la forêt**, puis entrez et confirmez un mot de passe d’approbation. Ce même mot de passe est également entré dans le portail Azure à la section suivante.
1. Parcourez les fenêtres suivantes avec les options par défaut, puis sélectionnez l’option pour **Non, ne pas confirmer l'approbation sortante**. Vous ne pouvez pas valider la relation d’approbation, car votre compte administrateur délégué à la forêt de ressources de domaine managé ne dispose pas des autorisations requises. Ce comportement est normal.
1. Sélectionnez **Terminer**.

## <a name="validate-resource-authentication"></a>Valider l’authentification des ressources

Les scénarios courants suivants vous permettent de vérifier que l’approbation de forêt authentifie correctement les utilisateurs et l’accès aux ressources :

* [Authentification des utilisateurs locaux à partir de la forêt de ressources Azure AD DS](#on-premises-user-authentication-from-the-azure-ad-ds-resource-forest)
* [Accéder aux ressources de la forêt de ressources Azure AD DS à l’aide de l’utilisateur local](#access-resources-in-the-azure-ad-ds-resource-forest-using-on-premises-user)
    * [Activer le partage de fichiers et d'imprimantes](#enable-file-and-printer-sharing)
    * [Créer un groupe de sécurité et ajouter des membres](#create-a-security-group-and-add-members)
    * [Créer un partage de fichiers pour l’accès inter-forêts](#create-a-file-share-for-cross-forest-access)
    * [Valider l'authentification inter-forêts vers une ressource](#validate-cross-forest-authentication-to-a-resource)

### <a name="on-premises-user-authentication-from-the-azure-ad-ds-resource-forest"></a>Authentification des utilisateurs locaux à partir de la forêt de ressources Azure AD DS

Vous devez avoir une machine virtuelle Windows Server jointe à la forêt de ressources du domaine managé. Utilisez cette machine virtuelle pour vérifier que votre utilisateur local peut s’authentifier sur une machine virtuelle.

1. Connectez-vous à la machine virtuelle Windows Server jointe à la forêt de ressources du domaine managé à l’aide du bureau à distance et de vos informations d’identification administrateur de domaine managé. Si une erreur d'authentification au niveau du réseau s'affiche, vérifiez que le compte d’utilisateur que vous avez utilisé ne correspond pas un compte d’utilisateur de domaine.

    > [!TIP]
    > Pour vous connecter en toute sécurité à vos machines virtuelles jointes à Azure AD Domain Services, vous pouvez utiliser le [service hôte Azure Bastion](../bastion/bastion-overview.md) dans les régions Azure prises en charge.

1. Ouvrez une invite de commandes et utilisez la commande `whoami` pour afficher le nom unique de l’utilisateur actuellement authentifié :

    ```console
    whoami /fqdn
    ```

1. Utilisez la commande `runas` pour vous authentifier en tant qu’utilisateur à partir du domaine local. Dans la commande suivante, remplacez `userUpn@trusteddomain.com` par l’UPN d’un utilisateur du domaine local approuvé. La commande vous invite à entrer le mot de passe de l’utilisateur :

    ```console
    Runas /u:userUpn@trusteddomain.com cmd.exe
    ```

1. Si l’authentification aboutit, une nouvelle invite de commandes s’ouvre. Le titre de la nouvelle invite de commandes comprend `running as userUpn@trusteddomain.com`.
1. Utilisez `whoami /fqdn` dans la nouvelle invite de commandes pour afficher le nom unique de l’utilisateur authentifié à partir de l'instance Active Directory locale.

### <a name="access-resources-in-the-azure-ad-ds-resource-forest-using-on-premises-user"></a>Accéder aux ressources de la forêt de ressources Azure AD DS à l’aide de l’utilisateur local

À l’aide de la machine virtuelle Windows Server jointe à la forêt de ressources de domaine managé, vous pouvez tester le scénario dans lequel les utilisateurs peuvent accéder aux ressources hébergées dans la forêt de ressources lorsqu’ils s’authentifient depuis des ordinateurs dans le domaine local avec des utilisateurs du domaine local. Les exemples suivants vous montrent comment créer et tester différents scénarios courants.

#### <a name="enable-file-and-printer-sharing"></a>Activer le partage de fichiers et d’imprimantes

1. Connectez-vous à la machine virtuelle Windows Server jointe à la forêt de ressources du domaine managé à l’aide du bureau à distance et de vos informations d’identification administrateur de domaine managé. Si une erreur d'authentification au niveau du réseau s'affiche, vérifiez que le compte d’utilisateur que vous avez utilisé ne correspond pas un compte d’utilisateur de domaine.

    > [!TIP]
    > Pour vous connecter en toute sécurité à vos machines virtuelles jointes à Azure AD Domain Services, vous pouvez utiliser le [service hôte Azure Bastion](../bastion/bastion-overview.md) dans les régions Azure prises en charge.

1. Ouvrez **Paramètres Windows**, puis recherchez et sélectionnez **Centre Réseau et partage**.
1. Sélectionnez l’option permettant de **modifier les paramètres de partage avancés**.
1. Sous **Profil du domaine**, sélectionnez **Activer le partage de fichiers et d'imprimantes**, puis **Enregistrer les modifications**.
1. Fermez **Centre Réseau et partage**.

#### <a name="create-a-security-group-and-add-members"></a>Créer un groupe de sécurité et ajouter des membres

1. Ouvrez **Utilisateurs et ordinateurs Active Directory**.
1. Cliquez avec le bouton droit sur le nom de domaine, sélectionnez **Nouveau**, puis **Unité d'organisation**.
1. Dans la zone de nom, entrez *LocalObjects*, puis sélectionnez **OK**.
1. Sélectionnez et cliquez avec le bouton droit sur **LocalObjects** dans le volet de navigation. Sélectionnez **Nouveau**, puis **Groupe**.
1. Entrez *FileServerAccess* dans la zone **Nom du groupe**. Pour **Étendue du groupe**, sélectionnez **Domaine local**, puis **OK**.
1. Dans le volet de contenu, double-cliquez sur **FileServerAccess**. Sélectionnez **Membres**, **Ajouter**, puis **Emplacements**.
1. Sélectionnez votre instance Active Directory locale dans l'affichage **Emplacement**, puis **OK**.
1. Entrez *Utilisateurs du domaine* dans la zone **Entrer les noms des objets à sélectionner**. Sélectionnez **Vérifier les noms**, fournissez les informations d’identification de l'instance Active Directory locale, puis sélectionnez **OK**.

    > [!NOTE]
    > La relation d'approbation étant unidirectionnelle, vous devez fournir les informations d’identification. Cela veut dire que les utilisateurs du domaine managé ne peuvent pas accéder aux ressources ni rechercher des utilisateurs ou des groupes dans le domaine (local) approuvé.

1. Le groupe **Utilisateurs du domaine** de votre instance Active Directory locale doit être membre du groupe **FileServerAccess**. Sélectionnez **OK** pour enregistrer le groupe et fermer la fenêtre.

#### <a name="create-a-file-share-for-cross-forest-access"></a>Créer un partage de fichiers pour l’accès inter-forêts

1. Sur la machine virtuelle Windows Server jointe à la forêt de ressources de domaine managé, créez un dossier et donnez-lui un nom, tel que *CrossForestShare*.
1. Cliquez avec le bouton droit sur le dossier, puis sélectionnez **Propriétés**.
1. Sélectionnez l'onglet **Sécurité**, puis **Modifier**.
1. Dans la boîte de dialogue *Autorisations pour CrossForestShare*, sélectionnez **Ajouter**.
1. Entrez *FileServerAccess* dans **Entrer les noms des objets à sélectionner**, puis sélectionnez **OK**.
1. Sélectionnez *FileServerAccess* dans la liste **Noms des groupes ou des utilisateurs**. Dans la liste **Autorisations pour FileServerAccess**, sélectionnez *Autoriser* pour les autorisations **Modifier** et **Écrire**, puis **OK**.
1. Sélectionnez l’onglet **Partage**, puis **Partage avancé…**
1. Sélectionnez **Partager ce dossier**, puis entrez un nom de partage de fichiers facile à retenir dans **Nom du partage**, par exemple *CrossForestShare*.
1. Sélectionnez **Autorisations**. Dans la liste **Autorisations pour tous**, sélectionnez **Autoriser** pour l'autorisation **Modifier**.
1. Sélectionnez **OK** deux fois, puis **Fermer**.

#### <a name="validate-cross-forest-authentication-to-a-resource"></a>Valider l’authentification inter-forêts sur une ressource

1. Connectez-vous à un ordinateur Windows joint à votre instance Active Directory locale à l’aide d’un compte d’utilisateur de votre instance Active Directory locale.
1. À l’aide de l'**Explorateur Windows**, connectez-vous au partage que vous avez créé à l’aide du nom d’hôte complet et le partage, par exemple `\\fs1.aaddscontoso.com\CrossforestShare`.
1. Pour valider l’autorisation d’écriture, cliquez avec le bouton droit dans le dossier, sélectionnez **Nouveau**, puis **Document texte**. Utilisez le nom par défaut **Nouveau document texte**.

    Si les autorisations d’écriture sont correctement définies, un nouveau document texte est créé. Les étapes suivantes permettent d'ouvrir, de modifier et de supprimer le fichier, selon vos besoins.
1. Pour valider l’autorisation de lecture, ouvrez **Nouveau document texte**.
1. Pour valider l’autorisation de modification, ajoutez du texte au fichier et fermez le **Bloc-notes**. Lorsque vous êtes invité à enregistrer les modifications, sélectionnez **Enregistrer**.
1. Pour valider l’autorisation de suppression, cliquez avec le bouton droit sur **Nouveau document texte**, puis sélectionnez **Supprimer**. Sélectionnez **Oui** pour confirmer la suppression du fichier.

## <a name="update-or-remove-outbound-forest-trust"></a>Mettre à jour ou supprimer l’approbation de la forêt sortante

Si vous devez mettre à jour une forêt sortante unidirectionnelle existante depuis le domaine managé, vous pouvez utiliser les scripts `Get-AaddsResourceForestTrusts` et `Set-AaddsResourceForestTrust`. Ces scripts vous aident dans les scénarios où vous souhaitez mettre à jour le nom convivial de la forêt d’approbation ou le mot de passe d’approbation. Pour supprimer une approbation sortante unidirectionnelle depuis le domaine managé, vous pouvez utiliser le script `Remove-AaddsResourceForestTrust`. Vous devez supprimer manuellement l’approbation de forêt sortante unidirectionnelle dans la forêt AD DS locale associée.

### <a name="update-a-forest-trust"></a>Mettre à jour une approbation de forêt

En cas de fonctionnement normal, la forêt du domaine managé et la forêt locale négocient entre elles un processus de mise à jour de mot de passe régulier. Cela fait partie du processus normal de sécurité de la relation d’approbation AD DS. Vous n’avez pas besoin d’opérer une rotation du mot de passe d’approbation manuellement, sauf si la relation d’approbation a rencontré un problème et que vous souhaitez réinitialiser manuellement un mot de passe connu. Pour en savoir plus, consultez [Changements de mot de passe de l’objet de domaine approuvé](concepts-forest-trust.md#tdo-password-changes).

L’exemple suivant illustre la procédure de mise à jour d’une relation d’approbation existante si vous devez réinitialiser manuellement le mot de passe d’approbation sortant :

1. Installez les scripts `Get-AaddsResourceForestTrusts` et `Set-AaddsResourceForestTrust` à partir de [PowerShell Gallery][powershell-gallery] à l’aide de la cmdlet [Install-script][Install-Script] :

    ```powershell
    Install-Script -Name Get-AaddsResourceForestTrusts,Set-AaddsResourceForestTrust
    ```

1. Avant de pouvoir mettre à jour l’approbation existante, obtenez la ressource d’approbation à l’aide du script `Get-AaddsResourceForestTrusts`. Dans l’exemple suivant, l’approbation existante est attribuée à un objet nommé *existingTrust*. Spécifiez votre propre nom de forêt de domaine managé et le nom de forêt locale à mettre à jour :

    ```powershell
    $existingTrust = Get-AaddsResourceForestTrust `
        -ManagedDomainFqdn "aaddscontoso.com" `
        -TrustFqdn "onprem.contoso.com" `
        -TrustFriendlyName "myAzureADDSTrust"
    ```

1. Pour mettre à jour le mot de passe d’approbation existant, utilisez le script `Set-AaddsResourceForestTrust`. Spécifiez l’objet d’approbation existant d’une étape précédente, puis un nouveau mot de passe de relation d’approbation. Étant donné que PowerShell ne garantit pas de complexité de mot de passe, assurez-vous de générer et utiliser un mot de passe sécurisé pour votre environnement.

    ```powershell
    Set-AaddsResourceForestTrust `
        -Trust $existingTrust `
        -TrustPassword <newComplexPassword>
    ```

### <a name="delete-a-forest-trust"></a>Supprimer une approbation de forêt

Si vous n’avez plus besoin d’une approbation de forêt sortante unidirectionnelle du domaine managé vers une forêt AD DS locale, vous pouvez la supprimer. Assurez-vous qu’aucune application et aucun service n’a besoin de s’authentifier sur la forêt AD DS locale avant de supprimer l’approbation. Vous devez supprimer manuellement l’approbation entrante unidirectionnelle dans la forêt AD DS locale également.

1. Installez le script `Remove-AaddsResourceForestTrust` à partir de [PowerShell Gallery][powershell-gallery] à l’aide de la cmdlet [Install-script][Install-Script] :

    ```powershell
    Install-Script -Name Remove-AaddsResourceForestTrust
    ```

1. À présent, supprimez l’approbation de forêt à l’aide du script `Remove-AaddsResourceForestTrust`. Dans l’exemple suivant, le nom de l’approbation *myAzureADDSTrust* entre la forêt du domaine managé nommé *aaddscontoso.com* et la forêt locale *onprem.contoso.com* est supprimé. Spécifiez votre propre nom de forêt de domaine managé et le nom de la forêt locale à supprimer :

    ```powershell
    Remove-AaddsResourceForestTrust `
        -ManagedDomainFqdn "aaddscontoso.com" `
        -TrustFqdn "onprem.contoso.com" `
        -TrustFriendlyName "myAzureADDSTrust"
    ```

Pour supprimer l’approbation entrante unidirectionnelle de la forêt AD DS locale, connectez-vous à un ordinateur de gestion avec accès à la forêt AD DS locale et procédez comme suit :

1. Sélectionnez **Démarrer | Outils d'administration | Domaines et approbations Active Directory**
1. Cliquez avec le bouton droit sur un domaine, par exemple *onprem.contoso.com*, sélectionnez **Propriétés**.
1. Choisissez l’onglet **Approbation**, puis sélectionnez l’approbation entrante existante de votre forêt de domaine managé.
1. Sélectionnez **Supprimer**, puis confirmez que vous souhaitez supprimer l’approbation entrante.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer une forêt de ressources de domaine managé à l’aide d’Azure PowerShell
> * Créer une approbation de forêt sortante unidirectionnelle dans le domaine managé à l’aide d’Azure PowerShell
> * Configurer un DNS dans un environnement AD DS local pour prendre en charge la connectivité du domaine managé
> * Créer une approbation de forêt unidirectionnelle entrante dans un environnement AD DS local
> * Tester et valider la relation d’approbation à des fins d'authentification et d’accès aux ressources

Pour plus d’informations conceptuelles sur les types de forêts dans Azure AD DS, consultez [Qu’est-ce que des forêts de ressources ?][concepts-forest] et [Comment fonctionnent les approbations de forêts dans Azure AD DS ?][concepts-trust]

<!-- INTERNAL LINKS -->
[concepts-forest]: concepts-resource-forest.md
[concepts-trust]: concepts-forest-trust.md
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance-advanced]: tutorial-create-instance-advanced.md
[Connect-AzAccount]: /powershell/module/Az.Accounts/Connect-AzAccount
[Connect-AzureAD]: /powershell/module/AzureAD/Connect-AzureAD
[New-AzResourceGroup]: /powershell/module/Az.Resources/New-AzResourceGroup
[network-peering]: ../virtual-network/virtual-network-peering-overview.md
[New-AzureADServicePrincipal]: /powershell/module/AzureAD/New-AzureADServicePrincipal
[Get-AzureRMSubscription]: /powershell/module/AzureRM.Profile/Get-AzureRmSubscription
[Install-Script]: /powershell/module/powershellget/install-script

<!-- EXTERNAL LINKS -->
[powershell-gallery]: https://www.powershellgallery.com/