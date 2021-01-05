---
title: Points de terminaison privés
description: Apprenez à créer des points de terminaison privés pour le service Sauvegarde Azure et découvrez les scénarios où l’utilisation des points de terminaison privés contribue à maintenir la sécurité de vos ressources.
ms.topic: conceptual
ms.date: 05/07/2020
ms.openlocfilehash: 0ca4e7a83e18ac72e25131d320737ce9578b1cf3
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184743"
---
# <a name="private-endpoints-for-azure-backup"></a>Points de terminaison privés pour le service Sauvegarde Azure

Le service Sauvegarde Azure vous permet de sauvegarder vos données dans vos coffres Recovery Services pour les restaurer ultérieurement en toute sécurité en utilisant des [points de terminaison privés](../private-link/private-endpoint-overview.md). Les points de terminaison privés utilisent une adresse IP privée de votre réseau virtuel (ou « VNet »), plaçant de fait le service dans votre réseau virtuel.

Cet article vous aidera à comprendre le processus de création de points de terminaison privés pour le service Sauvegarde Azure et les scénarios dans lesquels l’utilisation de points de terminaison privés contribue à maintenir la sécurité de vos ressources.

## <a name="before-you-start"></a>Avant de commencer

- Les points de terminaison privés ne peuvent être créés que pour les nouveaux coffres Recovery Services (qui ne contiennent pas d’éléments). Les points de terminaison privés doivent donc être créés avant que vous ne tentiez de protéger vos éléments en les plaçant dans un coffre.
- Un réseau virtuel peut contenir des points de terminaison privés pour plusieurs coffres Recovery Services. De son côté, un coffre Recovery Services peut être associé à plusieurs points de terminaison privés dans plusieurs réseaux virtuels. Toutefois, vous ne pouvez pas associer 1 coffre à plus de 12 points de terminaison privés.
- Lorsque vous associez un point de terminaison privé à un coffre, ce dernier est verrouillé. Cela veut dire qu’un coffre sera uniquement accessible (pour les sauvegardes et restaurations) sur les réseaux qui contiennent un point de terminaison privé associé à ce coffre. Si tous les points de terminaison privés associés au coffre sont supprimés, le coffre redevient accessible sur tous les réseaux.
- Une connexion de point de terminaison privée pour la sauvegarde utilise un total de 11 adresses IP privées dans votre sous-réseau, y compris celles utilisées par Sauvegarde Azure pour le stockage. Ce nombre peut être plus élevé (jusqu’à 25) pour certaines régions Azure. Nous vous suggérons donc d’avoir suffisamment d’adresses IP privées disponibles lorsque vous tentez de créer des points de terminaison privés pour la sauvegarde.
- Les coffres Recovery Services sont compatibles avec les services Sauvegarde Azure et Azure Site Recovery. Cependant, cet article traite uniquement de l’utilisation des points de terminaison privés pour le service Sauvegarde Azure.
- Azure Active Directory ne prend pas en charge les points de terminaison privés pour le moment. Par conséquent, les adresses IP et les noms de domaine complets requis pour le bon fonctionnement du service Azure Active Directory dans une région doivent bénéficier d’une autorisation d’accès sortant sur le réseau sécurisé lors de la sauvegarde de bases de données dans des machines virtuelles Azure et de la sauvegarde à l’aide de l’agent MARS. Vous pouvez aussi utiliser des balises de groupe de sécurité réseau (NSG) et des balises du service Pare-feu Azure pour autoriser l’accès à Azure AD, le cas échéant.
- Les réseaux virtuels avec des stratégies réseau ne sont pas compatibles avec les points de terminaison privés. Vous devez donc désactiver les stratégies réseau avant de continuer.
- Vous devrez réinscrire le fournisseur de ressources Recovery Services auprès de l’abonnement si vous l’avez enregistré avant le 1er mai 2020. Pour réinscrire le fournisseur, accédez à votre abonnement dans le portail Azure, accédez à **Fournisseur de ressources** dans la barre de navigation de gauche, sélectionnez **Microsoft.RecoveryServices**, puis sélectionnez **Réinscrire**.

## <a name="recommended-and-supported-scenarios"></a>Scénarios recommandés et pris en charge

Même si des points de terminaison privés sont activés dans un coffre, ils sont uniquement utilisés pour la sauvegarde et la restauration des charges de travail SQL et SAP HANA en cas de sauvegarde de machines virtuelles Azure et de sauvegarde à l’aide de l’agent MARS. Vous pouvez également utiliser le coffre pour la sauvegarde d’autres charges de travail (cependant, cela ne nécessiterait pas de point de terminaison privé). En plus de la sauvegarde des charges de travail SQL et SAP HANA et de la sauvegarde à l’aide de l’agent MARS, les points de terminaison privés servent également à effectuer des récupérations de fichiers pour la sauvegarde de machines virtuelles Azure. Pour plus d’informations, voir le tableau suivant :

| Sauvegarde de charges de travail dans une machine virtuelle Azure (SQL, SAP HANA) à l’aide de l’agent MARS | Nous vous recommandons d’utiliser des points de terminaison privés pour permettre la sauvegarde et la restauration sans avoir à créer de liste verte des adresses IP/FQDN pour les services Sauvegarde Azure ou le Stockage Azure depuis vos réseaux virtuels. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Sauvegarde des machines virtuelles Azure**                                         | La sauvegarde de machine virtuelle ne vous oblige pas à autoriser l’accès à des adresses IP ou des noms de domaine complets. Ainsi, les points de terminaison privés ne sont pas requis pour la sauvegarde et la restauration des disques.  <br><br>   Toutefois, la récupération de fichiers à partir d’un coffre contenant des points de terminaison privés est limitée aux réseaux virtuels qui contiennent un point de terminaison privé associé au coffre. <br><br>    Lorsque vous utilisez des disques non managés ACL, assurez-vous que le compte de stockage contenant les disques autorise l’accès à des **services Microsoft approuvés** s’il est ACL. |
| **Sauvegarde Azure Files**                                      | Les sauvegardes du service Azure Files sont stockées dans le compte de stockage local. Ainsi, vous n’avez pas besoin de points de terminaison privés pour la sauvegarde et la restauration. |

## <a name="creating-and-using-private-endpoints-for-backup"></a>Création et utilisation de points de terminaison privés pour le service Sauvegarde Azure

Cette section décrit les étapes de création et d’utilisation des points de terminaison privés pour le service Sauvegarde Azure à l’intérieur de vos réseaux virtuels.

>[!IMPORTANT]
> Nous vous recommandons vivement de suivre l’ordre des étapes établi dans ce document. Si vous ne le faites pas, le coffre rendu risque d’être incompatible avec l’utilisation des points de terminaison privés. Cela vous obligera à recommencer la procédure avec un nouveau coffre.

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

Consultez [cette section](#create-a-recovery-services-vault-using-the-azure-resource-manager-client) pour savoir comment créer un coffre à l’aide du client Azure Resource Manager. Cela crée un coffre dont l’identité managée est déjà activée. [En savoir plus](./backup-azure-recovery-services-vault-overview.md) sur les coffres Recovery Services.

## <a name="enable-managed-identity-for-your-vault"></a>Activer une identité managée sur votre machine virtuelle

Les identités managées permettent au coffre de créer et d’utiliser des points de terminaison privés. Cette section traite de l’activation de l’identité managée pour votre coffre.

1. Accédez à votre coffre Recovery Services, puis au paramètre **Identity** (Identité).

    ![Capture d’écran montrant l’option d’activation de l’état de l’identité](./media/private-endpoints/identity-status-on.png)

1. Cliquez sur le bouton bascule sous **État** pour le définir sur la valeur **Activé**, puis sélectionnez **Enregistrer**.

1. Un **ID d’objet** est généré. Il correspond à l’identité managée du coffre.

    >[!NOTE]
    >Une fois activée, l’identité managée ne doit **pas** être désactivée (même temporairement). La désactivation de l’identité managée peut entraîner un comportement incohérent.

## <a name="grant-permissions-to-the-vault-to-create-required-private-endpoints"></a>Accorder des autorisations au coffre pour créer des points de terminaison privés requis

Pour créer les points de terminaison privés requis pour le service Sauvegarde Azure, le coffre (ou plus exactement, l’identité managée du coffre) doit disposer d’autorisations sur les groupes de ressources suivants :

- Le groupe de ressources qui contient le réseau virtuel (VNet) cible.
- Le groupe de ressources dans lequel les points de terminaison privés doivent être créés.
- Le groupe de ressources qui contient Private DNS Zones, comme discuté en détail [ici](#creating-private-endpoints-for-backup)

Nous vous recommandons d’accorder le rôle **Contributor** (Contributeur) à ces trois groupes de ressources pour le coffre (identité managée). Les étapes suivantes expliquent comment effectuer cette opération pour un groupe de ressources particulier (cette opération doit être effectuée pour chacun des trois groupes de ressources) :

1. Accédez au groupe de ressources puis, dans la barre de gauche, accédez à **Access Control (IAM)** (Contrôle d'accès (Gestion des identités et des accès)).
1. Une fois dans **Access Control** (Contrôle d'accès), accédez à **Add a role assignment** (Ajouter une attribution de rôle).

    ![Ajouter une attribution de rôle](./media/private-endpoints/add-role-assignment.png)

1. Dans le volet **Add role assignment** (Ajouter une attribution de rôle), sélectionnez **Contributor** (Contributeur) en tant que **Role** (Rôle), puis utilisez le **Name** (Nom) du coffre en tant que **Principal**. Lorsque vous avez terminé, sélectionnez votre coffre, puis **Enregistrer**.

    ![Choisir un rôle et un principal](./media/private-endpoints/choose-role-and-principal.png)

Pour gérer les autorisations plus précisément, consultez la section [Créer manuellement des rôles et des autorisations](#create-roles-and-permissions-manually).

## <a name="creating-and-approving-private-endpoints-for-azure-backup"></a>Création et approbation de points de terminaison privés pour le service Sauvegarde Azure

### <a name="creating-private-endpoints-for-backup"></a>Création de points de terminaison privés pour le service Sauvegarde Azure

Cette section décrit le processus de création d’un point de terminaison privé pour votre coffre.

1. Dans la barre de recherche, recherchez et sélectionnez **Private Link** (Liaison privée). Vous accédez alors au **Private Link Center** (Centre de liaison privée).

    ![Rechercher un cloud privé](./media/private-endpoints/search-for-private-link.png)

1. Dans la barre de navigation de gauche, sélectionnez **points de terminaison privés**. Une fois dans le volet **Points de terminaison privés**, sélectionnez **+Ajouter** pour démarrer le processus de création d’un point de terminaison privé pour votre coffre.

    ![Ajouter un point de terminaison privé dans le Centre de liaison privée](./media/private-endpoints/add-private-endpoint.png)

1. Une fois dans le processus **Create Private Endpoint** (Créer un point de terminaison privé), vous devez entrer des informations requises pour créer la connexion de votre point de terminaison privé.

    1. **Paramètres de base**: Entrez les informations de base de vos points de terminaison privés. La région doit être la même que celle de votre coffre et votre ressource.

        ![Capture d’écran montrant les champs d’informations de base à remplir](./media/private-endpoints/basic-details.png)

    1. **Ressource** : Dans cet onglet, vous devez mentionner la ressource PaaS pour laquelle vous souhaitez créer votre connexion. Sélectionnez **Microsoft.RecoveryServices/vaults** à partir du type de ressource pour l’abonnement souhaité. Lorsque vous avez terminé, choisissez le nom de votre coffre Recovery Services dans la section **Resource** (Ressource) et **AzureBackup** dans la section **Target sub-resource** (Sous-ressource cible).

        ![Capture d’écran montrant les champs à remplir de l’onglet Resource (Ressource)](./media/private-endpoints/resource-tab.png)

    1. **Configuration** : Dans Configuration, spécifiez le réseau virtuel et le sous-réseau où vous souhaitez que le point de terminaison privé soit créé. Il s’agit du réseau virtuel sur lequel se trouve la machine virtuelle. Vous pouvez **intégrer votre point de terminaison privé** à une zone DNS privée. Vous pouvez également utiliser votre serveur DNS personnalisé ou créer une zone DNS privée.

        ![Capture d’écran montrant l’onglet Configuration](./media/private-endpoints/configuration-tab.png)

        Reportez-vous à [cette section](#dns-changes-for-custom-dns-servers) si vous souhaitez utiliser vos serveurs DNS personnalisés au lieu de les intégrer à Azure Private DNS Zones.  

    1. Si vous le souhaitez, vous pouvez ajouter des **balises** à votre point de terminaison privé.

    1. Passez à **Vérifier + créer** lorsque vous avez terminé la saisie des informations requises. Une fois la validation terminée, sélectionnez **Créer** pour créer le point de terminaison privé.

## <a name="approving-private-endpoints"></a>Approbation des points de terminaison privés

Si l’utilisateur qui crée le point de terminaison privé est également le propriétaire du coffre Recovery Services, le point de terminaison privé créé précédemment est approuvé automatiquement. Dans le cas contraire, le propriétaire du coffre doit approuver le point de terminaison privé avant de pouvoir l’utiliser. Cette section présente l’approbation manuelle des points de terminaison privés via le portail Azure.

Si vous souhaitez utiliser le client Azure Resource Manager pour approuver vos points de terminaison privés, consultez la section [Approbation manuelle des points de terminaison privés à l’aide du client Azure Resource Manager](#manual-approval-of-private-endpoints-using-the-azure-resource-manager-client) client Azure Resource Manager pour utiliser le client Azure Resource Manager pour l’approbation des points de terminaison privés.

1. Accédez à votre coffre Recovery Services. Dans la barre de gauche, accédez à **Private endpoint connections** (Connexions de point de terminaison privé).
1. Sélectionnez la connexion de point de terminaison privé à approuver
1. Dans la barre supérieure, sélectionnez **Approve** (Approuver). Vous pouvez également sélectionner **Reject** (Rejeter) ou **Remove** (Supprimer) si vous souhaitez rejeter ou supprimer la connexion au point de terminaison.

    ![Capture d’écran montrant la procédure d’approbation d’un point de terminaison privé](./media/private-endpoints/approve-private-endpoints.png)

## <a name="using-private-endpoints-for-backup"></a>Points de terminaison privés pour le service Sauvegarde Azure

Après l’approbation des points de terminaison privés créés pour le coffre de votre réseau virtuel, vous pouvez commencer à les utiliser pour effectuer vos sauvegardes et restaurations.

>[!IMPORTANT]
>Avant de continuer, vérifiez que vous avez suivi toutes les étapes précédentes de ce document. Voici une liste des de ces étapes :
>
>1. Créer un (nouveau) coffre Recovery Services
>1. Activer le coffre pour utiliser l’identité managée affectée par le système
>1. Attribuer les autorisations appropriées à l’identité managée du coffre
>1. Créer un point de terminaison privé pour votre coffre
>1. Approuver le point de terminaison privé (s’il n’a pas été approuvé automatiquement)

### <a name="backup-and-restore-of-workloads-in-azure-vm-sql-sap-hana"></a>Sauvegarde et restauration des charges de travail dans une machine virtuelle Azure (SQL, SAP HANA)

Lorsque le point de terminaison privé est créé et approuvé, aucune modification supplémentaire n’est requise côté client pour utiliser le point de terminaison privé. Toutes les communications et tous les transferts de données de votre réseau sécurisé vers le coffre sont effectués via le point de terminaison privé.
Toutefois, si vous supprimez des points de terminaison privés associés au coffre après l’enregistrement d’un serveur (SQL/SAP HANA), vous devez réinscrire le conteneur auprès du coffre. Vous n’avez pas besoin d’arrêter leur protection.

### <a name="backup-and-restore-through-mars-agent"></a>Sauvegarde et restauration par le biais de l’agent MARS

Lorsque vous utilisez l’agent MARS pour sauvegarder vos ressources locales, assurez-vous que votre réseau local (contenant vos ressources à sauvegarder) est homologué avec le réseau virtuel Azure qui contient un point de terminaison privé pour le coffre, afin de pouvoir l’utiliser. Vous pouvez ensuite continuer à installer l’agent MARS et configurer la sauvegarde comme indiqué ici. Toutefois, vous devez vous assurer que toutes les communications pour la sauvegarde s’effectuent uniquement par le biais du réseau homologué.

Si vous supprimez des points de terminaison privés pour le coffre après l’enregistrement d’un agent MARS, vous devez réinscrire le conteneur auprès du coffre. Vous n’avez pas besoin d’arrêter leur protection.

## <a name="additional-topics"></a>Rubriques supplémentaires

### <a name="create-a-recovery-services-vault-using-the-azure-resource-manager-client"></a>Utiliser le client Azure Resource Manager pour créer un coffre Recovery Services

Vous pouvez créer un coffre Recovery Services et activer son identité managée (l’activation de l’identité managée est requise, comme nous le verrons plus tard) à l’aide du client Azure Resource Manager. Un exemple de cette procédure est partagé ci-dessous :

```rest
armclient PUT /subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/Vaults/<vaultname>?api-version=2017-07-01-preview @C:\<filepath>\MSIVault.json
```

Le fichier JSON ci-dessus doit avoir le contenu suivant :

Requête JSON :

```json
{
  "location": "eastus2",
  "name": "<vaultname>",
  "etag": "W/\"datetime'2019-05-24T12%3A54%3A42.1757237Z'\"",
  "tags": {
    "PutKey": "PutValue"
  },
  "properties": {},
  "id": "/subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/Vaults/<vaultname>",
  "type": "Microsoft.RecoveryServices/Vaults",
  "sku": {
    "name": "RS0",
    "tier": "Standard"
  },
  "identity": {
    "type": "systemassigned"
  }
}
```

Réponse JSON :

```json
{
   "location": "eastus2",
   "name": "<vaultname>",
   "etag": "W/\"datetime'2020-02-25T05%3A26%3A58.5181122Z'\"",
   "tags": {
     "PutKey": "PutValue"
   },
   "identity": {
     "tenantId": "<tenantid>",
     "principalId": "<principalid>",
     "type": "SystemAssigned"
   },
   "properties": {
     "provisioningState": "Succeeded",
     "privateEndpointStateForBackup": "None",
     "privateEndpointStateForSiteRecovery": "None"
   },
   "id": "/subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/Vaults/<vaultname>",
   "type": "Microsoft.RecoveryServices/Vaults",
   "sku": {
     "name": "RS0",
     "tier": "Standard"
   }
 }
```

>[!NOTE]
>Le coffre créé dans cet exemple via le client Azure Resource Manager est déjà créé avec une identité managée affectée par le système.

### <a name="managing-permissions-on-resource-groups"></a>Gestion des autorisations dans les groupes de ressources

L’identité managée pour le coffre doit disposer des autorisations suivantes dans le groupe de ressources et le réseau virtuel où les points de terminaison privés seront créés :

- `Microsoft.Network/privateEndpoints/*` : cette autorisation est nécessaire pour exécuter CRUD sur des points de terminaison privés dans le groupe de ressources. Elle doit être affectée au niveau du groupe de ressources.
- `Microsoft.Network/virtualNetworks/subnets/join/action` : cette autorisation est nécessaire sur le réseau virtuel où l’adresse IP privée est attachée au point de terminaison privé.
- `Microsoft.Network/networkInterfaces/read` : cette autorisation est nécessaire sur le groupe de ressources pour obtenir l’interface réseau créée pour le point de terminaison privé.
- Private DNS Zone Contributor Role (Rôle Contributeur de zone DNS privée) : ce rôle existe déjà et peut être utilisé pour fournir les autorisations `Microsoft.Network/privateDnsZones/A/*` et `Microsoft.Network/privateDnsZones/virtualNetworkLinks/read`.

Vous pouvez utiliser l’une des méthodes suivantes pour créer des rôles avec les autorisations requises :

#### <a name="create-roles-and-permissions-manually"></a>Créer manuellement des rôles et des autorisations

Créez les fichiers JSON suivants et utilisez la commande PowerShell à la fin de la section pour créer des rôles :

//PrivateEndpointContributorRoleDef.json

```json
{
  "Name": "PrivateEndpointContributor",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows management of Private Endpoint",
  "Actions": [
    "Microsoft.Network/privateEndpoints/*",
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000"
  ]
}
```

//NetworkInterfaceReaderRoleDef.json

```json
{
  "Name": "NetworkInterfaceReader",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows read on networkInterfaces",
  "Actions": [
    "Microsoft.Network/networkInterfaces/read"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000"
  ]
}
```

//PrivateEndpointSubnetContributorRoleDef.json

```json
{
  "Name": "PrivateEndpointSubnetContributor",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows adding of Private Endpoint connection to Virtual Networks",
  "Actions": [
    "Microsoft.Network/virtualNetworks/subnets/join/action"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000"
  ]
}
```

```azurepowershell
 New-AzRoleDefinition -InputFile "PrivateEndpointContributorRoleDef.json"
 New-AzRoleDefinition -InputFile "NetworkInterfaceReaderRoleDef.json"
 New-AzRoleDefinition -InputFile "PrivateEndpointSubnetContributorRoleDef.json"
```

#### <a name="use-a-script"></a>Utiliser un script

1. Dans le portail Azure, démarrez le **Cloud Shell**, puis sélectionnez **Upload** (Charger) dans la fenêtre PowerShell.

    ![Capture d’écran montrant l’option Upload (Charger) de la fenêtre PowerShell](./media/private-endpoints/upload-file-in-powershell.png)

1. Chargez le script suivant : [VaultMsiPrereqScript](https://download.microsoft.com/download/1/2/6/126a410b-0e06-45ed-b2df-84f353034fa1/VaultMsiPrereqScript.ps1)

1. Accédez à votre dossier de démarrage (par exemple : `cd /home/user`)

1. Exécutez le script suivant :

    ```azurepowershell
    ./VaultMsiPrereqScript.ps1 -subscription <subscription-Id> -vaultPEResourceGroup <vaultPERG> -vaultPESubnetResourceGroup <subnetRG> -vaultMsiName <msiName>
    ```

    Les paramètres sont les suivants :

    - **subscription** : **SubscriptionId contenant le groupe de ressources dans lequel le point de terminaison privé du coffre doit être créé et le sous-réseau auquel le point de terminaison privé de l’archivage sera attaché

    - **vaultPEResourceGroup** : groupe de ressources dans lequel le point de terminaison privé de l’archivage sera créé

    - **vaultPESubnetResourceGroup** : groupe de ressources du sous-réseau auquel le point de terminaison privé sera joint

    - **vaultMsiName** : Nom du fichier MSI du coffre, qui est le même que **VaultName**

1. Terminez l’authentification et le script prendra le contexte de l’abonnement donné, fourni ci-dessus. Il créera les rôles appropriés si le locataire ne les contient pas, puis affectera des rôles au MSI du coffre.

### <a name="creating-private-endpoints-using-azure-powershell"></a>Création d’un point de terminaison privé à l’aide d’Azure PowerShell

#### <a name="auto-approved-private-endpoints"></a>Points de terminaison privés approuvés automatiquement

```azurepowershell
$vault = Get-AzRecoveryServicesVault `
        -ResourceGroupName $vaultResourceGroupName `
        -Name $vaultName
  
$privateEndpointConnection = New-AzPrivateLinkServiceConnection `
        -Name $privateEndpointConnectionName `
        -PrivateLinkServiceId $vault.ID `
        -GroupId "AzureBackup"  
  
$privateEndpoint = New-AzPrivateEndpoint `
        -ResourceGroupName $vmResourceGroupName `
        -Name $privateEndpointName `
        -Location $location `
        -Subnet $subnet `
        -PrivateLinkServiceConnection $privateEndpointConnection `
        -Force
```

### <a name="manual-approval-of-private-endpoints-using-the-azure-resource-manager-client"></a>Approbation manuelle des points de terminaison privés à l’aide du client Azure Resource Manager

1. Utilisez **GetVault** pour obtenir l’ID de connexion de point de terminaison privé de votre point de terminaison privé.

    ```rest
    armclient GET /subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/vaults/<vaultname>?api-version=2017-07-01-preview
    ```

    Cela retourne l’ID de connexion du point de terminaison privé. Vous pouvez récupérer le nom de la connexion à l’aide de la première partie de l’ID de connexion, comme suit :

    `privateendpointconnectionid = {peName}.{vaultId}.backup.{guid}`

1. Dans la réponse, récupérez le paramètre **Private Endpoint Connection ID** (ID de connexion de point de terminaison) (et le paramètre **Private Endpoint Name** (Nom du point de terminaison privé), lorsque cela est nécessaire), remplacez-les par le code JSON et l’URI de Azure Resource Manager suivants, puis essayez de définir le paramètre Status (État) sur « Approved/Rejected/Disconnected » (« Approuvé/Rejeté/Déconnecté », comme illustré dans l’exemple ci-dessous :

    ```rest
    armclient PUT /subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/Vaults/<vaultname>/privateEndpointConnections/<privateendpointconnectionid>?api-version=2020-02-02-preview @C:\<filepath>\BackupAdminApproval.json
    ```

    JSON :

    ```json
    {
    "id": "/subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/Vaults/<vaultname>/privateEndpointConnections/<privateendpointconnectionid>",
    "properties": {
        "privateEndpoint": {
        "id": "/subscriptions/<subscriptionid>/resourceGroups/<pergname>/providers/Microsoft.Network/privateEndpoints/pename"
        },
        "privateLinkServiceConnectionState": {
        "status": "Disconnected",  //choose state from Approved/Rejected/Disconnected
        "description": "Disconnected by <userid>"
        }
    }
    }
    ```

### <a name="dns-changes-for-custom-dns-servers"></a>Modifications de DNS pour créer des serveurs DNS personnalisés

#### <a name="create-dns-zones-for-custom-dns-servers"></a>Créer des zones DNS pour des serveurs DNS personnalisés

Vous devez créer trois zones DNS privées et les lier à votre réseau virtuel.

| **Zone**                                                     | **Service** |
| ------------------------------------------------------------ | ----------- |
| `privatelink.<geo>.backup.windowsazure.com`      | Sauvegarde      |
| `privatelink.blob.core.windows.net`                            | Objet blob        |
| `privatelink.queue.core.windows.net`                           | File d'attente       |

>[!NOTE]
>Dans le texte ci-dessus, *geo* fait référence au code de région. Par exemple, *wcus* et *ne* représentent les régions USA Centre-Ouest et Europe Nord.

[Cette liste](https://download.microsoft.com/download/1/2/6/126a410b-0e06-45ed-b2df-84f353034fa1/AzureRegionCodesList.docx) répertorie tous les codes de région. Consultez les liens suivants pour les conventions d'affectation de noms d’URL dans les régions nationales :

- [Chine](/azure/china/resources-developer-guide#check-endpoints-in-azure)
- [Allemagne](../germany/germany-developer-guide.md#endpoint-mapping)
- [Gouvernement des États-Unis](../azure-government/documentation-government-developer-guide.md)

#### <a name="adding-dns-records-for-custom-dns-servers"></a>Ajout d’enregistrements DNS pour les serveurs DNS personnalisés

Pour cela, vous devez créer des entrées pour chaque FQDN (nom de domaine complet) de votre point de terminaison privé dans votre zone de DNS privée.

Notez que nous allons utiliser les points de terminaison privés créés pour les services Sauvegarde, Blob et File d’attente.

- Le point de terminaison privé du coffre utilise le nom spécifié lors de la création du point de terminaison privé
- Les points de terminaison privés pour les services d’objets BLOB et de files d’attente ont pour préfixe le nom du coffre.

Par exemple, l’illustration suivante montre les trois points de terminaison privés créés pour une connexion de point de terminaison privée nommée *pee2epe* :

![Capture d’écran montrant trois points de terminaison privés d’une connexion de point de terminaison privé](./media/private-endpoints/three-private-endpoints.png)

Zone DNS pour le service Sauvegarde Azure (`privatelink.<geo>.backup.windowsazure.com`) :

1. Accédez à votre point de terminaison privé pour le service Sauvegarde dans le **Private Link Center** (Centre de liaison privé). La page Overview (Vue d’ensemble) répertorie le nom de domaine complet (FQDN) et les adresses IP privées de votre point de terminaison privé.

1. Ajoutez une entrée pour chaque nom de domaine complet (FQDN) et adresse IP privée en tant qu’enregistrement de type.

    ![Capture d’écran montrant l’ajout d’une entrée pour chaque nom de domaine complet (FQDN) et adresse IP privée](./media/private-endpoints/add-entry-for-each-fqdn-and-ip.png)

Zone DNS du service BLOB (`privatelink.blob.core.windows.net`) :

1. Accédez à votre point de terminaison privé pour le service Blob dans le **Private Link Center** (Centre de liaison privé). La page Overview (Vue d’ensemble) répertorie le nom de domaine complet (FQDN) et les adresses IP privées de votre point de terminaison privé.

1. Ajoutez une entrée pour le nom de domaine complet (FQDN) et l’adresse IP privée en tant qu’enregistrement A type (Type A).

    ![Capture d’écran montrant une entrée pour le nom de domaine complet (FQDN) et l’adresse IP privée en tant qu’enregistrement A type (Type A) pour le service Blob](./media/private-endpoints/add-type-a-record-for-blob.png)

Zone DNS du service de File d’attente Azure (`privatelink.queue.core.windows.net`) :

1. Accédez à votre point de terminaison privé pour le service File d’attente dans le **Private Link Center** (Centre de liaison privé). La page Overview (Vue d’ensemble) répertorie le nom de domaine complet (FQDN) et les adresses IP privées de votre point de terminaison privé.

1. Ajoutez une entrée pour le nom de domaine complet (FQDN) et l’adresse IP privée en tant qu’enregistrement A type (Type A).

    ![Capture d’écran montrant une entrée pour le nom de domaine complet (FQDN) et l’adresse IP privée en tant qu’enregistrement A type (Type A) pour le service de File d’attente](./media/private-endpoints/add-type-a-record-for-queue.png)

## <a name="frequently-asked-questions"></a>Forum Aux Questions (FAQ)

Q. Puis-je créer un point de terminaison privé pour un coffre de sauvegarde existant ?<br>
R. Non, les points de terminaison privés ne peuvent être créés que pour les nouveaux coffres de sauvegarde. Le coffre ne doit donc pas contenir d’éléments protégés. En fait, aucune tentative de protection des éléments dans le coffre ne peut être effectuée avant de créer des points de terminaison privés.

Q. J’ai essayé de protéger un élément dans mon coffre, mais l’opération a échoué et le coffre ne contient toujours aucun élément protégé. Puis-je créer des points de terminaison privés pour ce coffre ?<br>
R. Non, le coffre ne doit pas avoir subi de tentatives de protection.

Q. J’ai un coffre qui utilise des points de terminaison privés pour la sauvegarde et la restauration. Puis-je ajouter ou supprimer ultérieurement des points de terminaison privés pour ce coffre, même si des éléments de sauvegarde y sont protégés ?<br>
R. Oui. Si vous avez déjà créé des points de terminaison privés pour un coffre et des éléments de sauvegarde protégés, vous pouvez ajouter ou supprimer ultérieurement des points de terminaison privés en fonction des besoins.

Q. Le point de terminaison privé du service Sauvegarde Azure peut-il également être utilisé pour le service Azure Site Recovery ?<br>
R. Non, le point de terminaison privé du service Sauvegarde ne peut être utilisé que pour le service Sauvegarde Azure. Vous devez créer un nouveau point de terminaison privé pour le service Azure Site Recovery, si cela est pris en charge par le service.

Q. Je n’ai pas suivi l’une des étapes de cet article, mais j’ai pu protéger ma source de données. Puis-je continuer à utiliser des points de terminaison privés ?<br>
R. Si vous ne suivez pas chaque étape de cet article et que vous continuez à protéger des éléments, le coffre risque de ne pas pouvoir utiliser des points de terminaison privés. C’est pourquoi nous vous recommandons de vous référer à cette liste de vérification avant de continuer à protéger les éléments.

Q. Puis-je utiliser mon propre serveur DNS au lieu d’utiliser la zone DNS privée Azure ou une zone DNS privée intégrée ?<br>
R. Oui, vous pouvez utiliser vos propres serveurs DNS. Toutefois, assurez-vous que tous les enregistrements DNS requis sont ajoutés comme indiqué dans cette section.

Q. Dois-je suivre des étapes supplémentaires sur mon serveur après avoir suivi toutes les procédures de cet article ?<br>
R. Après avoir suivi toutes les procédures de cet article, vous avez besoin de rien d’autre pour utiliser des points de terminaison privés pour la sauvegarde et la restauration.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur toutes les [fonctionnalités de sécurité de la Sauvegarde Azure](security-overview.md)