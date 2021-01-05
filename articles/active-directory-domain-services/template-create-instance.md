---
title: Activer Azure DS Domain Services à l’aide d’un modèle | Microsoft Docs
description: Découvrir comment configurer et activer Azure Active Directory Domain Services à l’aide d’un modèle Resource Manager
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: sample
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: e18825da64d0d200f55ce72985ac843b93b1e612
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618788"
---
# <a name="create-an-azure-active-directory-domain-services-managed-domain-using-an-azure-resource-manager-template"></a>Créer un domaine managé Azure Active Directory Domain Services à l’aide d’un modèle Resource Manager

Azure Active Directory Domain Services (Azure AD DS) fournit des services de domaine managés, comme la jonction de domaine, la stratégie de groupe, le protocole LDAP, et l’authentification Kerberos/NTLM entièrement compatible avec Windows Server Active Directory. Vous consommez ces services de domaine sans déployer, gérer et mettre à jour avec des correctifs les contrôleurs de domaine vous-même. Azure AD DS s’intègre à votre locataire Azure AD existant. Cette intégration permet aux utilisateurs de se connecter en utilisant leurs informations d’identification d’entreprise, et vous pouvez utiliser des groupes et des comptes d’utilisateur existants pour sécuriser l’accès aux ressources.

Cet article vous montre comment créer un domaine managé à l’aide d’un modèle de Azure Resource Manager. Les ressources de prise en charge sont créées à l’aide d’Azure PowerShell.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce qui est décrit dans cet article, vous avez besoin des ressources suivantes :

* Installez et configurez Azure PowerShell.
    * Si nécessaire, suivez les instructions pour [installer le module Azure PowerShell et vous connecter à votre abonnement Azure](/powershell/azure/install-az-ps).
    * Veillez à vous connecter à votre abonnement Azure à l’aide de l’applet de commande [Connect-AzAccount][Connect-AzAccount].
* Installez et configurez Azure AD PowerShell.
    * Si nécessaire, suivez les instructions pour [installer le module Azure AD PowerShell et vous connecter à Azure AD](/powershell/azure/active-directory/install-adv2).
    * Veillez à vous connecter à votre locataire Azure AD à l’aide de l’applet de commande [Connect-AzureAD][Connect-AzureAD].
* Vous devez disposer des privilèges d’*Administrateur global* dans votre locataire Azure AD pour activer Azure AD DS.
* Vous avez besoin de privilèges de *Contributeur* dans votre abonnement Azure pour créer les ressources Azure AD DS nécessaires.

## <a name="dns-naming-requirements"></a>Exigences relatives aux noms DNS

Quand vous créez un domaine managé Azure AD DS, vous spécifiez un nom DNS. Voici quelques considérations liées au choix de ce nom DNS :

* **Nom de domaine intégré :** Par défaut, le nom de domaine intégré de l’annuaire est utilisé (un suffixe *.onmicrosoft.com*). Si vous voulez activer l’accès LDAP sécurisé au domaine managé via Internet, vous ne pouvez pas créer un certificat numérique pour sécuriser la connexion avec ce domaine par défaut. Microsoft détient le domaine *.onmicrosoft.com* : une autorité de certification n’émettra donc pas de certificat.
* **Noms de domaine personnalisés :** L’approche la plus courante consiste à spécifier un nom de domaine personnalisé, en général celui que vous possédez déjà et qui est routable. Quand vous utilisez un domaine personnalisé routable, le trafic peut s’écouler correctement en fonction des besoins pour prendre en charge vos applications.
* **Suffixes de domaine non routables :** D’une façon générale, nous vous recommandons d’éviter un suffixe de nom de domaine non routable, comme *contoso.local*. Le suffixe *.local* n’est pas routable et peut entraîner des problèmes de résolution DNS.

> [!TIP]
> Si vous créez un nom de domaine personnalisé, faites attention aux espaces de noms DNS existants. Il est recommandé d’utiliser un nom de domaine distinct de tout espace de noms DNS local ou Azure existant.
>
> Par exemple, si vous disposez de l’espace de noms DNS existant *contoso.com*, créez un domaine managé avec le nom de domaine personnalisé *aaddscontoso.com*. Si vous devez utiliser le protocole LDAP sécurisé, vous devez inscrire et avoir ce nom de domaine personnalisé pour générer les certificats requis.
>
> Vous devrez peut-être créer des enregistrements DNS supplémentaires pour d’autres services dans votre environnement, ou des redirecteurs DNS conditionnels entre les espaces de noms DNS existants dans votre environnement. Par exemple, si vous exécutez un serveur web qui héberge un site à l’aide du nom DNS racine, il peut y avoir des conflits de nommage qui nécessitent des entrées DNS supplémentaires.
>
> Dans cet exemple et dans les articles de guide pratique, le domaine personnalisé *aaddscontoso.com* est utilisé comme exemple rapide. Dans toutes les commandes, spécifiez votre propre nom de domaine.

Les restrictions de nom DNS suivantes s’appliquent également :

* **Restrictions de préfixe de domaine :** Vous ne pouvez pas créer de domaine managé avec un préfixe de plus de 15 caractères. Le préfixe du nom de domaine spécifié (par exemple, *aaddscontoso* dans le nom de domaine *aaddscontoso.com*) doit contenir au maximum 15 caractères.
* **Conflits de noms de réseau :** Le nom de domaine DNS de votre domaine managé ne doit pas déjà exister dans le réseau virtuel. En particulier, recherchez les scénarios suivants, qui aboutiraient à un conflit de noms :
    * Si vous avec un domaine Active Directory avec le même nom de domaine DNS sur le réseau virtuel Azure.
    * Si le réseau virtuel dans lequel vous envisagez d’activer le domaine managé a une connexion VPN avec votre réseau local. Dans ce scénario, veillez à ne pas avoir de domaine portant le même nom de domaine DNS sur votre réseau local.
    * Si vous avez un service cloud Azure avec ce nom sur le réseau virtuel Azure.

## <a name="create-required-azure-ad-resources"></a>Créer les ressources Azure AD nécessaires

Azure AD DS nécessite un principal du service et un groupe Azure AD. Ces ressources permettent au domaine managé de synchroniser les données et de définir les utilisateurs qui disposent d’autorisations administratives dans le domaine managé.

Dans un premier temps, inscrivez le fournisseur de ressources Azure AD Domain Services à l’aide de l’applet de commande [Register-AzResourceProvider][Register-AzResourceProvider] :

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.AAD
```

Créez un principal de service Azure AD à l’aide de l’applet de commande [New-AzureADServicePrincipal][New-AzureADServicePrincipal] pour permettre à Azure AD DS de communiquer et de s’authentifier. Un ID d’application spécifique est utilisé. Il se nomme *Domain Controller Services* et son ID est *6ba9a5d4-8456-4118-b521-9c5ca10cdf84*. Ne modifiez pas cet ID d’application.

```powershell
New-AzureADServicePrincipal -AppId "6ba9a5d4-8456-4118-b521-9c5ca10cdf84"
```

Créez maintenant un groupe Azure AD nommé *AAD DC Administrators* à l’aide de l’applet de commande [New-AzureADGroup][New-AzureADGroup]. Les utilisateurs ajoutés à ce groupe se voient ensuite accorder des autorisations pour effectuer des tâches d’administration dans le domaine managé.

```powershell
New-AzureADGroup -DisplayName "AAD DC Administrators" `
  -Description "Delegated group to administer Azure AD Domain Services" `
  -SecurityEnabled $true -MailEnabled $false `
  -MailNickName "AADDCAdministrators"
```

Une fois le groupe *AAD DC Administrators* créé, ajoutez un utilisateur au groupe à l’aide de l’applet de commande [Add-AzureADGroupMember][Add-AzureADGroupMember]. Vous obtenez d’abord l’ID d’objet du groupe *AAD DC Administrators* via l’applet de commande [Get-AzureADGroup][Get-AzureADGroup] et ensuite l’ID d’objet de l’utilisateur souhaité via l’applet de commande [Get-AzureADUser][Get-AzureADUser].

Dans l’exemple suivant, l’ID d’objet utilisateur du compte a le nom d’utilisateur principal (UPN) `admin@contoso.onmicrosoft.com`. Remplacez ce compte d’utilisateur par l’UPN de l’utilisateur que vous souhaitez ajouter au groupe *AAD DC Administrators* :

```powershell
# First, retrieve the object ID of the newly created 'AAD DC Administrators' group.
$GroupObjectId = Get-AzureADGroup `
  -Filter "DisplayName eq 'AAD DC Administrators'" | `
  Select-Object ObjectId

# Now, retrieve the object ID of the user you'd like to add to the group.
$UserObjectId = Get-AzureADUser `
  -Filter "UserPrincipalName eq 'admin@contoso.onmicrosoft.com'" | `
  Select-Object ObjectId

# Add the user to the 'AAD DC Administrators' group.
Add-AzureADGroupMember -ObjectId $GroupObjectId.ObjectId -RefObjectId $UserObjectId.ObjectId
```

Pour finir, créez un groupe de ressources avec l’applet de commande [New-AzResourceGroup][New-AzResourceGroup]. Dans l’exemple suivant, le groupe de ressources est nommé *myResourceGroup* et est créé dans la région *westus*. Utilisez votre propre nom et la région souhaitée :

```powershell
New-AzResourceGroup `
  -Name "myResourceGroup" `
  -Location "WestUS"
```

Si vous choisissez une région qui prend en charge les Zones de disponibilité, les ressources Azure AD DS sont réparties entre les zones pour assurer une redondance supplémentaire. Les Zones de disponibilité sont des emplacements physiques uniques au sein d’une région Azure. Chaque zone de disponibilité est composée d’un ou de plusieurs centres de données équipés d’une alimentation, d’un système de refroidissement et d’un réseau indépendants. Pour garantir la résilience, un minimum de trois zones distinctes sont activées dans toutes les régions.

Vous ne devez rien configurer pour la répartition d’Azure AD DS entre les zones. La plateforme Azure gère automatiquement la répartition de zone des ressources. Pour plus d’informations et pour connaître la disponibilité régionale, voir [Zones de disponibilité dans Azure][availability-zones].

## <a name="resource-definition-for-azure-ad-ds"></a>Définition de ressource pour Azure AD DS

Dans le cadre de la définition de ressources Resource Manager, les paramètres de configuration suivants sont requis :

| Paramètre               | Valeur |
|-------------------------|---------|
| domainName              | Le nom de domaine DNS de votre domaine managé, en prenant en considération les points précédents sur les préfixes d’attribution de noms et les conflits. |
| filteredSync            | Azure AD DS vous permet de synchroniser *tous* les utilisateurs et les groupes disponibles dans Azure AD, ou d’effectuer une synchronisation *limitée* seulement à des groupes spécifiques.<br /><br /> Pour en savoir plus sur la synchronisation limitée, consultez [Synchronisation limitée d’Azure AD Domain Services][scoped-sync].|
| notificationSettings    | Si des alertes sont générées dans le domaine managé, des notifications par e-mail peuvent être envoyées. <br /><br />Les *administrateurs généraux* du locataire Azure et des membres du groupe *AAD DC Administrators* peuvent être *activés* pour ces notifications.<br /><br /> Si vous le souhaitez, vous pouvez ajouter des destinataires supplémentaires auxquels doivent être envoyées les notifications des alertes qui nécessitent une attention particulière.|
| domainConfigurationType | Par défaut, un domaine managé est créé en tant que forêt d’*utilisateurs*. Ce type de forêt synchronise tous les objets d’Azure AD, notamment les comptes d’utilisateur créés dans un environnement AD DS local. Vous n’avez pas besoin de spécifier une valeur *domainConfiguration* pour créer une forêt d’utilisateurs.<br /><br /> Une forêt de *ressources* synchronise uniquement les utilisateurs et les groupes créés directement dans Azure AD. Définissez la valeur sur *ResourceTrusting* pour créer une forêt de ressources.<br /><br />Pour plus d’informations sur les forêts de *ressources*, notamment sur la raison pour laquelle vous pouvez en utiliser une et comment créer des approbations de forêts avec des domaines AD DS locaux, consultez [Vue d’ensemble des forêts de ressources Azure AD DS][resource-forests].|

La définition des paramètres condensés suivants montre comment ces valeurs sont déclarées. Une forêt d’utilisateurs nommée *aaddscontoso.com* est créée avec tous les utilisateurs d’Azure AD DS synchronisés avec le domaine managé :

```json
"parameters": {
    "domainName": {
        "value": "aaddscontoso.com"
    },
    "filteredSync": {
        "value": "Disabled"
    },
    "notificationSettings": {
        "value": {
            "notifyGlobalAdmins": "Enabled",
            "notifyDcAdmins": "Enabled",
            "additionalRecipients": []
        }
    },
    [...]
}
```

Le type de ressource de modèle Resource Manager condensé suivant est ensuite utilisé pour définir et créer le domaine managé. Un réseau virtuel et un sous-réseau Azure doivent déjà exister ou être créés dans le cadre du modèle Resource Manager. Le domaine managé est connecté à ce sous-réseau.

```json
"resources": [
    {
        "apiVersion": "2017-06-01",
        "type": "Microsoft.AAD/DomainServices",
        "name": "[parameters('domainName')]",
        "location": "[parameters('location')]",
        "dependsOn": [
            "[concat('Microsoft.Network/virtualNetworks/', parameters('vnetName'))]"
        ],
        "properties": {
            "domainName": "[parameters('domainName')]",
            "subnetId": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/virtualNetworks/', parameters('vnetName'), '/subnets/', parameters('subnetName'))]",
            "filteredSync": "[parameters('filteredSync')]",
            "notificationSettings": "[parameters('notificationSettings')]"
        }
    },
    [...]
]
```

Ces paramètres et le type de ressource peuvent être utilisés dans le cadre d’un modèle Resource Manager plus étendu pour déployer un domaine managé, comme indiqué dans la section suivante.

## <a name="create-a-managed-domain-using-sample-template"></a>Créer un domaine managé à l’aide d’un exemple de modèle

L’exemple de modèle complet Resource Manager suivant crée un domaine managé et les règles de réseau virtuel, de sous-réseau et de groupe de sécurité réseau associés. Les règles de groupe de sécurité réseau sont requises pour sécuriser le domaine géré et s’assurer que le trafic puisse circuler correctement. Une forêt d’utilisateurs avec le nom DNS *aaddscontoso.com* est créée, avec tous les utilisateurs synchronisés à partir d’Azure AD :

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "apiVersion": {
            "value": "2017-06-01"
        },
        "domainConfigurationType": {
            "value": "FullySynced"
        },
        "domainName": {
            "value": "aaddscontoso.com"
        },
        "filteredSync": {
            "value": "Disabled"
        },
        "location": {
            "value": "westus"
        },
        "notificationSettings": {
            "value": {
                "notifyGlobalAdmins": "Enabled",
                "notifyDcAdmins": "Enabled",
                "additionalRecipients": []
            }
        },
        "subnetName": {
            "value": "aadds-subnet"
        },
        "vnetName": {
            "value": "aadds-vnet"
        },
        "vnetAddressPrefixes": {
            "value": [
                "10.1.0.0/24"
            ]
        },
        "subnetAddressPrefix": {
            "value": "10.1.0.0/24"
        },
        "nsgName": {
            "value": "aadds-nsg"
        }
    },
    "resources": [
        {
            "apiVersion": "2017-06-01",
            "type": "Microsoft.AAD/DomainServices",
            "name": "[parameters('domainName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Network/virtualNetworks/', parameters('vnetName'))]"
            ],
            "properties": {
                "domainName": "[parameters('domainName')]",
                "subnetId": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/virtualNetworks/', parameters('vnetName'), '/subnets/', parameters('subnetName'))]",
                "filteredSync": "[parameters('filteredSync')]",
                "domainConfigurationType": "[parameters('domainConfigurationType')]",
                "notificationSettings": "[parameters('notificationSettings')]"
            }
        },
        {
            "type": "Microsoft.Network/NetworkSecurityGroups",
            "name": "[parameters('nsgName')]",
            "location": "[parameters('location')]",
            "properties": {
                "securityRules": [
                    {
                        "name": "AllowSyncWithAzureAD",
                        "properties": {
                            "access": "Allow",
                            "priority": 101,
                            "direction": "Inbound",
                            "protocol": "Tcp",
                            "sourceAddressPrefix": "AzureActiveDirectoryDomainServices",
                            "sourcePortRange": "*",
                            "destinationAddressPrefix": "*",
                            "destinationPortRange": "443"
                        }
                    },
                    {
                        "name": "AllowPSRemoting",
                        "properties": {
                            "access": "Allow",
                            "priority": 301,
                            "direction": "Inbound",
                            "protocol": "Tcp",
                            "sourceAddressPrefix": "AzureActiveDirectoryDomainServices",
                            "sourcePortRange": "*",
                            "destinationAddressPrefix": "*",
                            "destinationPortRange": "5986"
                        }
                    },
                    {
                        "name": "AllowRD",
                        "properties": {
                            "access": "Allow",
                            "priority": 201,
                            "direction": "Inbound",
                            "protocol": "Tcp",
                            "sourceAddressPrefix": "CorpNetSaw",
                            "sourcePortRange": "*",
                            "destinationAddressPrefix": "*",
                            "destinationPortRange": "3389"
                        }
                    }
                ]
            },
            "apiVersion": "2018-04-01"
        },
        {
            "type": "Microsoft.Network/virtualNetworks",
            "name": "[parameters('vnetName')]",
            "location": "[parameters('location')]",
            "apiVersion": "2018-04-01",
            "dependsOn": [
                "[concat('Microsoft.Network/NetworkSecurityGroups/', parameters('nsgName'))]"
            ],
            "properties": {
                "addressSpace": {
                    "addressPrefixes": "[parameters('vnetAddressPrefixes')]"
                },
                "subnets": [
                    {
                        "name": "[parameters('subnetName')]",
                        "properties": {
                            "addressPrefix": "[parameters('subnetAddressPrefix')]",
                            "networkSecurityGroup": {
                                "id": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/NetworkSecurityGroups/', parameters('nsgName'))]"
                            }
                        }
                    }
                ]
            }
        }
    ],
    "outputs": {}
}
```

Ce modèle peut être déployé à l’aide de votre méthode de déploiement préférée, comme le [Portail Azure][portal-deploy], [Azure PowerShell][powershell-deploy] ou un pipeline CI/CD. L’exemple suivant utilise l’applet de commande [New-AzResourceGroupDeployment][New-AzResourceGroupDeployment]. Spécifiez vos propres nom de groupe de ressources et nom de fichier de modèle :

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "myResourceGroup" -TemplateFile <path-to-template>
```

Créer la ressource et retourner le contrôle à l’invite PowerShell prend quelques minutes. Le provisionnement du domaine managé se poursuit en arrière-plan et le déploiement peut prendre jusqu’à une heure. Dans le portail Azure, la page **Vue d’ensemble** de votre domaine managé indique l’état actuel tout au long de cette phase de déploiement.

Une fois que le portail Azure a indiqué que le provisionnement du domaine managé était terminé, voici les tâches qu’il convient d’effectuer :

* Mettez à jour les paramètres DNS pour le réseau virtuel afin que les machines virtuelles puissent trouver le domaine géré pour l’authentification ou la jonction de domaine.
    * Pour configure le système DNS, sélectionnez votre domaine managé dans le portail. Dans la fenêtre **Vue d’ensemble**, vous êtes invité à configurer automatiquement ces paramètres DNS.
* [Activez la synchronisation de mot de passe avec Azure AD DS](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds) afin que les utilisateurs finaux puissent se connecter au domaine managé avec leurs informations d’identification d’entreprise.

## <a name="next-steps"></a>Étapes suivantes

Pour voir le domaine managé en action, vous pouvez [joindre une machine virtuelle Windows à un domaine][windows-join], [configurer le protocole LDAP sécurisé][tutorial-ldaps] et [configurer la synchronisation du hachage de mot de passe][tutorial-phs].

<!-- INTERNAL LINKS -->
[windows-join]: join-windows-vm.md
[tutorial-ldaps]: tutorial-configure-ldaps.md
[tutorial-phs]: tutorial-configure-password-hash-sync.md
[availability-zones]: ../availability-zones/az-overview.md
[portal-deploy]: ../azure-resource-manager/templates/deploy-portal.md
[powershell-deploy]: ../azure-resource-manager/templates/deploy-powershell.md
[scoped-sync]: scoped-synchronization.md
[resource-forests]: concepts-resource-forest.md

<!-- EXTERNAL LINKS -->
[Connect-AzAccount]: /powershell/module/Az.Accounts/Connect-AzAccount
[Connect-AzureAD]: /powershell/module/AzureAD/Connect-AzureAD
[New-AzureADServicePrincipal]: /powershell/module/AzureAD/New-AzureADServicePrincipal
[New-AzureADGroup]: /powershell/module/AzureAD/New-AzureADGroup
[Add-AzureADGroupMember]: /powershell/module/AzureAD/Add-AzureADGroupMember
[Get-AzureADGroup]: /powershell/module/AzureAD/Get-AzureADGroup
[Get-AzureADUser]: /powershell/module/AzureAD/Get-AzureADUser
[Register-AzResourceProvider]: /powershell/module/Az.Resources/Register-AzResourceProvider
[New-AzResourceGroup]: /powershell/module/Az.Resources/New-AzResourceGroup
[Get-AzSubscription]: /powershell/module/Az.Accounts/Get-AzSubscription
[cloud-shell]: ../cloud-shell/cloud-shell-windows-users.md
[naming-prefix]: /windows-server/identity/ad-ds/plan/selecting-the-forest-root-domain
[New-AzResourceGroupDeployment]: /powershell/module/Az.Resources/New-AzResourceGroupDeployment