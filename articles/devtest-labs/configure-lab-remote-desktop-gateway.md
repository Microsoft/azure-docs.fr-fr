---
title: Configurer un labo pour utiliser une passerelle des services Bureau à distance dans Azure DevTest Labs
description: Découvrez comment configurer un labo dans Azure DevTest Labs avec une passerelle des services Bureau à distance afin de garantir un accès sécurisé aux machines virtuelles de labo sans avoir à exposer le port RDP.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: dcf5191dea64c3d7bf28b9ce1c616d3d2defb73e
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97695693"
---
# <a name="configure-your-lab-in-azure-devtest-labs-to-use-a-remote-desktop-gateway"></a>Configurer votre labo dans Azure DevTest Labs pour utiliser une passerelle des services Bureau à distance
Dans Azure DevTest Labs, vous pouvez configurer une passerelle des services Bureau à distance pour votre labo afin de garantir un accès sécurisé aux machines virtuelles de labo sans avoir à exposer le port RDP. Le labo fournit un emplacement centralisé permettant aux utilisateurs de votre labo de voir toutes les machines virtuelles auxquelles ils ont accès et de s’y connecter. Le bouton **Se connecter** dans la page **Machine virtuelle** crée un fichier RDP spécifique à la machine que vous pouvez ouvrir pour vous connecter à cette dernière. Vous pouvez personnaliser et sécuriser davantage la connexion RDP en connectant votre labo à une passerelle des services Bureau à distance. 

Cette approche est plus sécurisée car l’utilisateur du labo s’authentifie directement auprès de la machine de passerelle ou peut utiliser les informations d’identification de l’entreprise sur une machine de passerelle jointe à un domaine pour se connecter à ses machines. Le labo prend également en charge l’utilisation de l’authentification du jeton sur la machine de passerelle qui permet aux utilisateurs de se connecter à leurs machines virtuelles de labo sans que le port RDP soit exposé à Internet. Cet article présente un exemple illustrant la configuration d’un labo qui utilise l’authentification du jeton pour se connecter à des machines de labo.

## <a name="architecture-of-the-solution"></a>Architecture de la solution

![Architecture de la solution](./media/configure-lab-remote-desktop-gateway/architecture.png)

1. L’action [Obtenir le contenu du fichier RDP](/rest/api/dtl/virtualmachines/getrdpfilecontents) est appelée quand vous sélectionnez le bouton **Connect** (Se connecter) (1). 
1. L’action Obtenir le contenu du fichier RDP appelle `https://{gateway-hostname}/api/host/{lab-machine-name}/port/{port-number}` pour demander un jeton d’authentification.
    1. `{gateway-hostname}` est le nom d’hôte de passerelle spécifié dans la page **Paramètres lab** de votre labo dans le portail Azure. 
    1. `{lab-machine-name}` est le nom de l’ordinateur auquel vous tentez de vous connecter.
    1. `{port-number}` est le port sur lequel la connexion doit être établie. Il s’agit généralement du port 3389. Si la machine virtuelle de labo utilise la fonctionnalité d’[adresse IP partagée](devtest-lab-shared-ip.md) dans DevTest Labs, le port sera différent.
1. La passerelle des services Bureau à distance confie à une fonction Azure l’appel à partir de `https://{gateway-hostname}/api/host/{lab-machine-name}/port/{port-number}` pour générer le jeton d’authentification. Le service DevTest Labs inclut automatiquement la clé de la fonction dans l’en-tête de demande. La clé de la fonction doit être enregistrée dans le coffre de clés du labo. Le nom de ce secret s’affiche comme **Secret du jeton de passerelle** dans la page **Paramètres lab** du labo.
1. La fonction Azure est censée retourner un jeton pour l’authentification du jeton basée sur un certificat auprès de la machine de passerelle.  
1. L’action Obtenir le contenu du fichier RDP retourne alors le fichier RDP complet, dont les informations d’authentification.
1. Vous ouvrez le fichier RDP à l’aide de votre programme de connexion RDP préféré. N’oubliez pas que tous les programmes de connexion RDP ne prennent pas en charge l’authentification du jeton. Le jeton d’authentification a une date d’expiration, définie par l’application de fonction. Établissez la connexion à la machine virtuelle de labo avant l’expiration du jeton.
1. Une fois que la machine de passerelle des services Bureau à distance authentifie le jeton dans le fichier RDP, la connexion est transférée vers votre machine de labo.

### <a name="solution-requirements"></a>Exigences de la solution
Pour utiliser la fonctionnalité d’authentification du jeton DevTest Labs, il existe quelques exigences de configuration pour les machines, les services DNS (Domain Name Service) et les fonctions de passerelle.

### <a name="requirements-for-remote-desktop-gateway-machines"></a>Exigences relatives aux machines de passerelle des services Bureau à distance
- Un certificat TLS/SSL doit être installé sur la machine de passerelle pour gérer le trafic HTTPS. Le certificat doit correspondre le nom de domaine complet (FQDN) de l’équilibreur de charge de la batterie de serveurs de passerelle ou au nom de domaine complet de la machine elle-même s’il n’en existe qu’une. Les certificats TLS/SSL utilisant des caractères génériques ne fonctionnent pas.  
- Un certificat de signature installé sur la ou les machines de passerelle. Créez un certificat de signature à l’aide du script [Create-SigningCertificate.ps1](https://github.com/Azure/azure-devtestlab/blob/master/samples/DevTestLabs/GatewaySample/tools/Create-SigningCertificate.ps1).
- Installez le module d’[authentification enfichable](https://code.msdn.microsoft.com/windowsdesktop/Remote-Desktop-Gateway-517d6273) qui prend en charge l’authentification du jeton pour la passerelle des services Bureau à distance. `RDGatewayFedAuth.msi`, qui est fourni avec des [images System Center Virtual Machine Manager (VMM)](/system-center/vmm/install-console?view=sc-vmm-1807), est un exemple de ce type de module. Pour plus d’informations sur System Center, consultez la [documentation de System Center](/system-center/) et les [détails des tarifs](https://www.microsoft.com/cloud-platform/system-center-pricing).  
- Le serveur de passerelle peut gérer les requêtes adressées à `https://{gateway-hostname}/api/host/{lab-machine-name}/port/{port-number}`.

    gateway-hostname est le nom de domaine complet (FQDN) de l’équilibreur de charge de la batterie de serveurs de passerelle ou le nom de domaine complet de la machine elle-même s’il n’en existe qu’une. `{lab-machine-name}` est le nom de la machine de labo que vous essayez de connecter, et `{port-number}` est le port sur lequel la connexion sera établie.  Par défaut, ce port est le port 3389.  Toutefois, si la machine virtuelle utilise la fonctionnalité d’[adresse IP partagée](devtest-lab-shared-ip.md) dans DevTest Labs, le port sera différent.
- Le module [Application Routing Request](/iis/extensions/planning-for-arr/using-the-application-request-routing-module) pour Internet Information Server (IIS) peut être utilisé pour rediriger les requêtes `https://{gateway-hostname}/api/host/{lab-machine-name}/port/{port-number}` vers à la fonction Azure, qui gère la requête afin d’obtenir un jeton pour l’authentification.


## <a name="requirements-for-azure-function"></a>Exigences relatives à la fonction Azure
La fonction Azure gère les requêtes au format `https://{function-app-uri}/app/host/{lab-machine-name}/port/{port-number}` et retourne le jeton d’authentification basé sur le même certificat de signature que celui installé sur les machines de passerelle. `{function-app-uri}` est l’URI utilisé pour accéder à la fonction. La clé de la fonction est automatiquement passée dans l’en-tête de la requête. Pour obtenir un exemple de fonction, consultez [https://github.com/Azure/azure-devtestlab/blob/master/samples/DevTestLabs/GatewaySample/src/RDGatewayAPI/Functions/CreateToken.cs](https://github.com/Azure/azure-devtestlab/blob/master/samples/DevTestLabs/GatewaySample/src/RDGatewayAPI/Functions/CreateToken.cs). 


## <a name="requirements-for-network"></a>Exigences relatives au réseau

- DNS pour le nom de domaine complet (FQDN) associé au certificat TLS/SSL installé sur les machines de passerelle doit diriger le trafic vers la machine de passerelle ou de l’équilibreur de charge de la batterie de serveurs de machines de passerelle.
- Si la machine de labo utilise des adresses IP privées, il doit y avoir un chemin réseau de la machine de passerelle à la machine de labo, soit en partageant le même réseau virtuel, soit en utilisant des réseaux virtuels appairés.

## <a name="configure-the-lab-to-use-token-authentication"></a>Configurer le labo pour utiliser une authentification du jeton 
Cette section montre comment configurer un labo pour utiliser une machine de passerelle des services Bureau à distance qui prend en charge l’authentification du jeton. Cette section n’explique pas comment configurer une batterie de serveurs de passerelle des services Bureau à distance proprement dite. Pour plus d’informations, consultez la section [Exemple pour créer une passerelle des services Bureau à distance](#sample-to-create-a-remote-desktop-gateway) à la fin de cet article. 

Avant de mettre à jour les paramètres du labo, stockez la clé nécessaire pour réussir l’exécution de la fonction afin de retourner un jeton d’authentification dans le coffre de clés du labo. Vous pouvez obtenir la valeur de clé de la fonction dans la page **Gérer** de la fonction dans le portail Azure. Pour plus d’informations sur la façon d’enregistrer un secret dans un coffre de clés, consultez [Ajouter un secret au coffre de clés](../key-vault/secrets/quick-create-portal.md#add-a-secret-to-key-vault). Enregistrez le nom du secret pour une utilisation ultérieure.

Pour rechercher l’ID du coffre de clés du labo, exécutez la commande Azure CLI suivante : 

```azurecli
az resource show --name {lab-name} --resource-type 'Microsoft.DevTestLab/labs' --resource-group {lab-resource-group-name} --query properties.vaultName
```

Configurer le labo pour utiliser l’authentification du jeton à l’aide des étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.
1. Dans la liste de labos, sélectionnez votre **labo**.
1. Dans la page du labo, sélectionnez **Configuration et stratégies**.
1. Dans le menu de gauche, dans la section **Paramètres**, sélectionnez **Paramètres lab**.
1. Dans la section **Bureau à distance**, entrez le nom de domaine complet (FQDN) ou l’adresse IP de la machine ou la batterie de serveurs de passerelle des services Bureau à distance pour le champ **Nom d’hôte de passerelle**. Cette valeur doit correspondre au nom de domaine complet (FQDN) du certificat TLS/SSL utilisé sur les machines de passerelle.

    ![Options de Bureau à distance dans les paramètres du labo](./media/configure-lab-remote-desktop-gateway/remote-desktop-options-in-lab-settings.png)
1. Dans la section **Bureau à distance**, pour le **Secret du jeton de passerelle**, entrez le nom du secret créé précédemment. Cette valeur n’est pas la clé de la fonction proprement dite, mais le nom du secret dans le coffre de clés du labo qui la contient.

    ![Secret du jeton de passerelle dans les paramètres du labo](./media/configure-lab-remote-desktop-gateway/gateway-token-secret.png)
1. **Enregistrez** les changements.

    > [!NOTE] 
    > En cliquant sur **Enregistrer**, vous acceptez les [termes du contrat de licence de la passerelle des services Bureau à distance](https://www.microsoft.com/licensing/product-licensing/products). Pour plus d’informations sur la passerelle distante, consultez [Bienvenue dans les services Bureau à distance](/windows-server/remote/remote-desktop-services/Welcome-to-rds) et [Déployer votre environnement Bureau à distance](/windows-server/remote/remote-desktop-services/rds-deploy-infrastructure).


S’il est préférable de configurer le labo par le biais de l’automatisation, consultez [Set-DevTestLabGateway.ps1](https://github.com/Azure/azure-devtestlab/blob/master/samples/DevTestLabs/GatewaySample/tools/Set-DevTestLabGateway.ps1) pour obtenir un exemple de script PowerShell permettant de définir les paramètres de **nom d’hôte de passerelle** et de **secret du jeton de passerelle**. Le [dépôt GitHub Azure DevTest Labs](https://github.com/Azure/azure-devtestlab) fournit également un modèle Azure Resource Manager qui crée ou met à jour un labo avec les paramètres de **nom d’hôte de passerelle** et de **secret du jeton de passerelle**.

## <a name="configure-network-security-group"></a>Configurer un groupe de sécurité réseau
Pour renforcer la sécurité du labo, un groupe de sécurité réseau (NSG) peut être ajouté au réseau virtuel utilisé par les machines virtuelles du labo. Pour obtenir des instructions sur la configuration d’un groupe de sécurité réseau, consultez [Créer, changer ou supprimer un groupe de sécurité réseau](../virtual-network/manage-network-security-group.md).

Voici un exemple de groupe de sécurité réseau qui autorise uniquement le trafic qui passe d’abord par la passerelle pour atteindre les machines de labo. La source de cette règle est l’adresse IP de l’unique machine de passerelle ou l’adresse IP de l’équilibreur de charge devant les machines de passerelle.

![Groupe de sécurité réseau - Règles](./media/configure-lab-remote-desktop-gateway/network-security-group-rules.png)

## <a name="sample-to-create-a-remote-desktop-gateway"></a>Exemple pour créer une passerelle des services Bureau à distance

> [!NOTE] 
> En utilisant les exemples de modèles, vous acceptez les [termes du contrat de licence de la passerelle des services Bureau à distance](https://www.microsoft.com/licensing/product-licensing/products). Pour plus d’informations sur la passerelle distante, consultez [Bienvenue dans les services Bureau à distance](/windows-server/remote/remote-desktop-services/Welcome-to-rds) et [Déployer votre environnement Bureau à distance](/windows-server/remote/remote-desktop-services/rds-deploy-infrastructure).

Le [dépôt GitHub Azure DevTest Labs](https://github.com/Azure/azure-devtestlab) fournit quelques exemples destinés à vous aider à configurer les ressources nécessaires pour utiliser l’authentification du jeton et la passerelle des services Bureau à distance avec DevTest Labs. Ces exemples incluent des modèles Azure Resource Manager pour les machines de passerelle, les paramètres de labo et l’application de fonction.

Suivez ces étapes afin de configurer un exemple de solution pour la batterie de serveurs de passerelle des services Bureau à distance.

1. Créez un certificat de signature.  Exécutez [Create-SigningCertificate.ps1](https://github.com/Azure/azure-devtestlab/blob/master/samples/DevTestLabs/GatewaySample/tools/Create-SigningCertificate.ps1). Enregistrez l’empreinte, le mot de passe et l’encodage en Base64 du certificat créé.
2. Obtenez un certificat TLS/SSL. Le nom de domaine complet (FQDN) associé au certificat TLS/SSL doit être pour le domaine que vous contrôlez. Enregistrez l’empreinte, le mot de passe et l’encodage en Base64 de ce certificat. Pour obtenir l’empreinte à l’aide de PowerShell, utilisez les commandes suivantes.

    ```powershell
    $cer = New-Object System.Security.Cryptography.X509Certificates.X509Certificate;
    $cer.Import(‘path-to-certificate’);
    $hash = $cer.GetCertHashString()
    ```

    Pour obtenir l’encodage en Base64 à l’aide de PowerShell, utilisez la commande suivante.

    ```powershell
    [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes(‘path-to-certificate’))
    ```
3. Télécharger les fichiers à partir de [https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/GatewaySample/arm/gateway](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/GatewaySample/arm/gateway).

    Le modèle nécessite un accès à quelques autres modèles Resource Manager et à des ressources associées au niveau du même URI de base. Copiez tous les fichiers à partir de [https://github.com/Azure/azure-devtestlab/blob/master/samples/DevTestLabs/GatewaySample/arm/gateway](https://github.com/Azure/azure-devtestlab/blob/master/samples/DevTestLabs/GatewaySample/arm/gateway) et RDGatewayFedAuth.msi vers un conteneur d’objets blob dans un compte de stockage.  
4. Déployez **azuredeploy.json** à partir de [https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/GatewaySample/arm/gateway](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/GatewaySample/arm/gateway). Le modèle prend les paramètres suivants :
    - adminUsername : obligatoire.  Nom d’utilisateur de l’administrateur des machines de passerelle.
    - adminPassword : obligatoire. Mot de passe du compte administrateur pour les machines de passerelle.
    - instanceCount : nombre de machines de passerelle à créer.  
    - alwaysOn : indique s’il faut maintenir l’application Azure Functions créée dans un état actif ou non. Maintenir l’application Azure Functions active permet d’éviter des retards quand les utilisateurs tentent de se connecter pour la première fois à leur machine virtuelle de labo, mais cela a des implications en termes de coût.  
    - tokenLifetime : durée pendant laquelle le jeton créé sera valide. Le format est HH :MM:SS.
    - sslCertificate : encodage en Base64 du certificat TLS/SSL pour la machine de passerelle.
    - sslCertificatePassword : mot de passe du certificat TLS/SSL pour la machine de passerelle.
    - sslCertificateThumbprint : empreinte du certificat pour l’identification dans le magasin de certificats local du certificat TLS/SSL.
    - signCertificate : encodage en Base64 du certificat de signature pour la machine de passerelle.
    - signCertificatePassword : mot de passe du certificat de signature pour la machine de passerelle.
    - signCertificateThumbprint - empreinte du certificat pour l’identification dans le magasin de certificats local du certificat de signature.
    - _artifactsLocation : emplacement URI où se trouvent toutes les ressources de support. Cette valeur doit être un URI complet, et non un chemin relatif.
    - _artifactsLocationSasToken : jeton de signature d’accès partagé (SAP) utilisé pour accéder aux ressources de support, si l’emplacement est un compte de stockage Azure.

    Le modèle peut être déployé avec l’interface Azure CLI à l’aide de la commande suivante :

    ```azurecli
    az deployment group create --resource-group {resource-group} --template-file azuredeploy.json --parameters @azuredeploy.parameters.json -–parameters _artifactsLocation="{storage-account-endpoint}/{container-name}" -–parameters _artifactsLocationSasToken = "?{sas-token}"
    ```

    Voici les descriptions des paramètres :

    - {storage-account-endpoint} peut être obtenu en exécutant `az storage account show --name {storage-acct-name} --query primaryEndpoints.blob`.  {storage-acct-name} est le nom du compte de stockage contenant les fichiers que vous avez chargés.  
    - {container-name} est le nom du conteneur se trouvant dans {storage-acct-name} qui contient les fichiers que vous avez chargés.  
    - {sas-token} peut être obtenu en exécutant `az storage container generate-sas --name {container-name} --account-name {storage-acct-name} --https-only –permissions drlw –expiry {utc-expiration-date}`. 
        - {storage-acct-name} est le nom du compte de stockage contenant les fichiers que vous avez chargés.  
        - {container-name} est le nom du conteneur se trouvant dans {storage-acct-name} qui contient les fichiers que vous avez chargés.  
        - {utc-expiration-date} est la date, au format UTC, à laquelle le jeton SAP expirera et à laquelle le jeton SAS ne peut plus être utilisé pour accéder au compte de stockage.

    Enregistrez les valeurs pour gatewayFQDN et gatewayIP à partir de la sortie de déploiement de modèle. Vous devez également enregistrer la valeur de la clé de fonction pour la fonction nouvellement créée, qui se trouve sous l’onglet [Paramètres Function App](../azure-functions/functions-how-to-use-azure-function-app-settings.md) .
5. Configurez DNS de sorte que le nom de domaine complet (FQDN) du certificat TLS/SSL mène à l’adresse IP de gatewayIP de l’étape précédente.

    Une fois la batterie de serveurs de passerelle des services Bureau à distance créée et les mises à jour DNS appropriées effectuées, il est prêt à être utilisé par un labo dans DevTest Labs. Les paramètres de **nom d’hôte de passerelle** et de **secret du jeton de passerelle** doivent être configurés pour utiliser la ou les machines de passerelle que vous avez déployées. 

    > [!NOTE]
    > Si la machine de labo utilise des adresses IP privées, il doit y avoir un chemin réseau de la machine de passerelle à la machine de labo, soit en partageant le même réseau virtuel, soit en utilisant un réseau virtuel appairé.

    Une fois la passerelle et le labo tous les deux configurés, le fichier de connexion créé quand l’utilisateur du labo clique sur le **Connect** (Se connecter) inclut automatiquement les informations nécessaires pour se connecter à l’aide de l’authentification du jeton.     

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur les services Bureau à distance, consultez l’article suivant : [Documentation des services Bureau à distance](/windows-server/remote/remote-desktop-services/Welcome-to-rds)