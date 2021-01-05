---
title: Mettre à niveau des nœuds de cluster pour utiliser des disques managés Azure
description: Voici comment mettre à niveau un cluster Service Fabric existant pour utiliser des disques managés Azure avec peu ou pas de temps d’arrêt de votre cluster.
ms.topic: how-to
ms.date: 4/07/2020
ms.openlocfilehash: 36896a6cf471ff0c9312ab454465419471bb164d
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92316156"
---
# <a name="upgrade-cluster-nodes-to-use-azure-managed-disks"></a>Mettre à niveau des nœuds de cluster pour utiliser des disques managés Azure

Les [disques managés Azure](../virtual-machines/managed-disks-overview.md) sont les offres de stockage sur disque recommandées à utiliser avec les machines virtuelles Azure pour le stockage persistant de données. Vous pouvez améliorer la résilience de vos charges de travail Service Fabric en mettant à niveau les groupes de machines virtuelles identiques qui sous-tendent vos types de nœuds pour utiliser des disques managés. Voici comment mettre à niveau un cluster Service Fabric existant pour utiliser des disques managés Azure avec peu ou pas de temps d’arrêt de votre cluster.

La stratégie générale de mise à niveau d’un nœud de cluster Service Fabric pour utiliser des disques managés consiste à :

1. Déployer un groupe de machines virtuelles identiques de ce type de nœud qui serait autrement dupliqué, mais avec l’objet [managedDisk](/azure/templates/microsoft.compute/2019-07-01/virtualmachinescalesets/virtualmachines#ManagedDiskParameters) ajouté à la section `osDisk` du modèle de déploiement du groupe de machines virtuelles identiques. Le nouveau groupe identique doit être lié à l’équilibreur de charge/à l’adresse IP d’origine, de sorte que vos clients ne subissent aucune interruption de service pendant la migration.

2. Une fois que les groupes identiques d’origine et mis à niveau sont exécutés côte à côte, désactivez les instances de nœud d’origine une par une afin que les services système (ou réplicas de services avec état) migrent vers le nouveau groupe identique.

3. Vérifiez que le cluster et les nouveaux nœuds sont intègres, puis supprimez le groupe identique d’origine et l’état du nœud pour les nœuds supprimés.

Cet article vous guide tout au long des étapes de mise à niveau du type de nœud principal d’un exemple de cluster pour utiliser des disques managés tout en évitant les temps d’arrêt du cluster (voir remarque ci-dessous). L’état initial de l’exemple de cluster de test se compose d’un type de nœud de [durabilité Silver](service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster), soutenu par un groupe identique à cinq nœuds.

> [!NOTE]
> Les limitations d’un équilibreur de charge de référence (SKU) De base empêchent l’ajout d’un groupe identique supplémentaire. Nous vous recommandons d’utiliser à la place l’équilibreur de charge de référence (SKU) Standard. Pour plus d’informations, consultez la [comparaison des deux références (SKU)](../load-balancer/skus.md).

> [!CAUTION]
> Vous rencontrerez une panne avec cette procédure uniquement si vous avez des dépendances sur le DNS du cluster (par exemple, lors de l’accès à [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md)). La [meilleure pratique architecturale pour les services frontaux](/azure/architecture/microservices/design/gateway) consiste à avoir un certain type d’[équilibreur de charge](/azure/architecture/guide/technology-choices/load-balancing-overview) devant vos types de nœud pour rendre possible l’échange de nœuds sans provoquer de panne.

Voici les [modèles et cmdlets](https://github.com/microsoft/service-fabric-scripts-and-templates/tree/master/templates/nodetype-upgrade-no-outage) pour Azure Resource Manager que nous allons utiliser pour terminer le scénario de mise à niveau. Les modifications du modèle seront expliquées dans la partie [Déployer un groupe identique mis à niveau pour le type de nœud principal](#deploy-an-upgraded-scale-set-for-the-primary-node-type) ci-dessous.

## <a name="set-up-the-test-cluster"></a>Configurer le cluster de test

Configurons le cluster de test Service Fabric initial. Tout d’abord, [téléchargez](https://github.com/microsoft/service-fabric-scripts-and-templates/tree/master/templates/nodetype-upgrade-no-outage) les exemples de modèles Azure Resource Manager que nous allons utiliser pour terminer ce scénario.

Ensuite, connectez-vous à votre compte Azure.

```powershell
# Sign in to your Azure account
Login-AzAccount -SubscriptionId "<subscription ID>"
```

Les commandes suivantes vous guident dans la génération d’un nouveau certificat autosigné et le déploiement du cluster de test. Si vous avez déjà un certificat que vous souhaitez utiliser, passez à [Utiliser un certificat existant pour déployer le cluster](#use-an-existing-certificate-to-deploy-the-cluster).

### <a name="generate-a-self-signed-certificate-and-deploy-the-cluster"></a>Générer un certificat autosigné et déployer le cluster

Tout d’abord, attribuez les variables dont vous aurez besoin pour le déploiement du cluster Service Fabric. Ajustez les valeurs de `resourceGroupName`, `certSubjectName`, `parameterFilePath` et `templateFilePath` pour votre compte et votre environnement spécifiques :

```powershell
# Assign deployment variables
$resourceGroupName = "sftestupgradegroup"
$certOutputFolder = "c:\certificates"
$certPassword = "Password!1" | ConvertTo-SecureString -AsPlainText -Force
$certSubjectName = "sftestupgrade.southcentralus.cloudapp.azure.com"
$templateFilePath = "C:\Initial-1NodeType-UnmanagedDisks.json"
$parameterFilePath = "C:\Initial-1NodeType-UnmanagedDisks.parameters.json"
```

> [!NOTE]
> Assurez-vous que l’emplacement `certOutputFolder` existe sur votre ordinateur local avant d’exécuter la commande pour déployer un nouveau cluster Service Fabric.

Ensuite, ouvrez le fichier [*Initial-1NodeType-UnmanagedDisks.parameters.json*](https://github.com/erikadoyle/service-fabric-scripts-and-templates/blob/managed-disks/templates/nodetype-upgrade-no-outage/Initial-1NodeType-UnmanagedDisks.parameters.json), ajustez les valeurs de `clusterName` et `dnsName` pour qu’elles correspondent aux valeurs dynamiques que vous avez définies dans PowerShell et enregistrez vos modifications.

Déployez ensuite le cluster de test Service Fabric :

```powershell
# Deploy the initial test cluster
New-AzServiceFabricCluster `
    -ResourceGroupName $resourceGroupName `
    -CertificateOutputFolder $certOutputFolder `
    -CertificatePassword $certPassword `
    -CertificateSubjectName $certSubjectName `
    -TemplateFile $templateFilePath `
    -ParameterFile $parameterFilePath
```

Une fois le déploiement terminé, localisez le fichier *.pfx* (`$certPfx`) sur votre ordinateur local et importez-le dans votre magasin de certificats :

```powershell
cd c:\certificates
$certPfx = ".\sftestupgradegroup20200312121003.pfx"

Import-PfxCertificate `
     -FilePath $certPfx `
     -CertStoreLocation Cert:\CurrentUser\My `
     -Password (ConvertTo-SecureString Password!1 -AsPlainText -Force)
```

L’opération renverra l’empreinte du certificat, que vous utiliserez pour [vous connecter au nouveau cluster](#connect-to-the-new-cluster-and-check-health-status) et vérifier son état d’intégrité. (Ignorez la section suivante, qui est une autre approche du déploiement de cluster.)

### <a name="use-an-existing-certificate-to-deploy-the-cluster"></a>Utiliser un certificat existant pour déployer le cluster

Vous pouvez également utiliser un certificat Azure Key Vault existant pour déployer le cluster de test. Pour ce faire, vous devez [obtenir les références de votre Key Vault](#obtain-your-key-vault-references) et l’empreinte du certificat.

```powershell
# Key Vault variables
$certUrlValue = "https://sftestupgradegroup.vault.azure.net/secrets/sftestupgradegroup20200309235308/dac0e7b7f9d4414984ccaa72bfb2ea39"
$sourceVaultValue = "/subscriptions/########-####-####-####-############/resourceGroups/sftestupgradegroup/providers/Microsoft.KeyVault/vaults/sftestupgradegroup"
$thumb = "BB796AA33BD9767E7DA27FE5182CF8FDEE714A70"
```

Ouvrez le fichier [*Initial-1NodeType-UnmanagedDisks.parameters.json*](https://github.com/erikadoyle/service-fabric-scripts-and-templates/blob/managed-disks/templates/nodetype-upgrade-no-outage/Initial-1NodeType-UnmanagedDisks.parameters.json) et remplacez les valeurs de `clusterName` et `dnsName` par une valeur unique.

Enfin, désignez un nom de groupe de ressources pour le cluster et définissez les emplacements `templateFilePath` et `parameterFilePath` de vos fichiers *Initial-1NodeType-UnmanagedDisks* :

> [!NOTE]
> Le groupe de ressources désigné doit déjà exister et se trouver dans la même région que votre Key Vault.

```powershell
# Deploy the new scale set (upgraded to use managed disks) into the primary node type.
$resourceGroupName = "sftestupgradegroup"
$templateFilePath = "C:\Upgrade-1NodeType-2ScaleSets-ManagedDisks.json"
$parameterFilePath = "C:\Upgrade-1NodeType-2ScaleSets-ManagedDisks.parameters.json"
```

Enfin, exécutez la commande suivante pour déployer le cluster de test initial :

```powershell
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath `
    -CertificateThumbprint $thumb `
    -CertificateUrlValue $certUrlValue `
    -SourceVaultValue $sourceVaultValue `
    -Verbose
```

### <a name="connect-to-the-new-cluster-and-check-health-status"></a>Se connecter au nouveau cluster et vérifier l’état d’intégrité

Connectez-vous au cluster et assurez-vous que ses cinq nœuds sont intègres (en remplaçant les variables `clusterName` et `thumb` pour votre cluster) :

```powershell
# Connect to the cluster
$clusterName = "sftestupgrade.southcentralus.cloudapp.azure.com:19000"
$thumb = "BB796AA33BD9767E7DA27FE5182CF8FDEE714A70"

Connect-ServiceFabricCluster `
    -ConnectionEndpoint $clusterName `
    -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCertThumbprint $thumb  `
    -FindType FindByThumbprint `
    -FindValue $thumb `
    -StoreLocation CurrentUser `
    -StoreName My

# Check cluster health
Get-ServiceFabricClusterHealth
```

Avec cela, nous sommes prêts à commencer la procédure de mise à niveau.

## <a name="deploy-an-upgraded-scale-set-for-the-primary-node-type"></a>Déployer un groupe identique mis à niveau pour le type de nœud principal

Pour mettre à niveau, ou *mettre à l’échelle verticalement*, un type de nœud, nous devrons déployer une copie du groupe de machines virtuelles identiques de ce type de nœud, qui est autrement identique au groupe identique d’origine (y compris la référence aux mêmes `nodeTypeRef`, `subnet` et `loadBalancerBackendAddressPools`), à ceci près qu’elle inclut les mises à niveau/modifications souhaitées et ses propres sous-réseau distinct et pool d’adresses NAT entrantes. Étant donné que nous mettons à niveau un type de nœud principal, le nouveau groupe identique sera marqué comme principal (`isPrimary: true`), tout comme le groupe identique d’origine. (Pour les mises à niveau de type de nœud non principal, omettez-le simplement.)

Pour plus de commodité, les modifications requises ont déjà été apportées pour vous dans les fichiers *Upgrade-1NodeType-2ScaleSets-ManagedDisks*, de [modèle](https://github.com/erikadoyle/service-fabric-scripts-and-templates/blob/managed-disks/templates/nodetype-upgrade-no-outage/Upgrade-1NodeType-2ScaleSets-ManagedDisks.json) et de [paramètres](https://github.com/erikadoyle/service-fabric-scripts-and-templates/blob/managed-disks/templates/nodetype-upgrade-no-outage/Upgrade-1NodeType-2ScaleSets-ManagedDisks.parameters.json).

Les sections suivantes décrivent les modifications de modèle en détail. Si vous préférez, vous pouvez ignorer l’explication et passer à [l’étape suivante de la procédure de mise à niveau](#obtain-your-key-vault-references).

### <a name="update-the-cluster-template-with-the-upgraded-scale-set"></a>Mettre à jour le modèle de cluster avec le groupe identique mis à niveau

Voici les modifications section par section du modèle de déploiement du cluster d’origine pour l’ajout d’un groupe identique mis à niveau pour le type de nœud principal.

#### <a name="parameters"></a>Paramètres

Ajoutez des paramètres pour le nom de l’instance, le nombre et la taille du nouveau groupe identique. Notez que `vmNodeType1Name` est propre au nouveau groupe identique, tandis que les valeurs nombre et taille sont identiques à celles du groupe identique d’origine.

**Fichier de modèle**

```json
"vmNodeType1Name": {
    "type": "string",
    "defaultValue": "NTvm2",
    "maxLength": 9
},
"nt1InstanceCount": {
    "type": "int",
    "defaultValue": 5,
    "metadata": {
        "description": "Instance count for node type"
    }
},
"vmNodeType1Size": {
    "type": "string",
    "defaultValue": "Standard_D2_v2"
},
```

**Fichier de paramètres**

```json
"vmNodeType1Name": {
    "value": "NTvm2"
},
"nt1InstanceCount": {
    "value": 5
},
"vmNodeType1Size": {
    "value": "Standard_D2_v2"
}
```

### <a name="variables"></a>Variables

Dans la section `variables` du modèle de déploiement, ajoutez une entrée pour le pool d’adresses NAT entrantes du nouveau groupe identique.

**Fichier de modèle**

```json
"lbNatPoolID1": "[concat(variables('lbID0'),'/inboundNatPools/LoadBalancerBEAddressNatPool1')]", 
```

### <a name="resources"></a>Ressources

Dans la section *Ressources* du modèle de déploiement, ajoutez le nouveau groupe de machines virtuelles identiques en gardant à l’esprit les points suivants :

* Le nouveau groupe identique fait référence au même type de nœud que l’original :

    ```json
    "nodeTypeRef": "[parameters('vmNodeType0Name')]",
    ```

* Le nouveau groupe identique fait référence à la même adresse principale et au même sous-réseau de l’équilibreur de charge (mais utilise un pool NAT entrant d’équilibrage de charge différent) :

   ```json
    "loadBalancerBackendAddressPools": [
        {
            "id": "[variables('lbPoolID0')]"
        }
    ],
    "loadBalancerInboundNatPools": [
        {
            "id": "[variables('lbNatPoolID1')]"
        }
    ],
    "subnet": {
        "id": "[variables('subnet0Ref')]"
    }
   ```

* Comme pour le groupe identique d’origine, le nouveau groupe identique est marqué comme type de nœud principal. (Lors de la mise à niveau de types de nœuds non principaux, omettez cette modification.)

    ```json
    "isPrimary": true,
    ```

* Contrairement au groupe identique d’origine, le nouveau groupe identique est mis à niveau pour utiliser des disques managés.

    ```json
    "managedDisk": {
        "storageAccountType": "[parameters('storageAccountType')]"
    }
    ```

Une fois que vous avez implémenté toutes les modifications apportées à vos fichiers de modèle et de paramètres, passez à la section suivante pour obtenir vos références Key Vault et déployer les mises à jour sur votre cluster.

### <a name="obtain-your-key-vault-references"></a>Obtenir vos références Key Vault

Pour déployer la configuration mise à jour, vous devez d’abord obtenir plusieurs références à votre certificat de cluster stocké dans votre Key Vault. Le moyen le plus simple de rechercher ces valeurs consiste à utiliser le Portail Azure. Vous devez disposer des éléments suivants :

* **L’URL Key Vault de votre certificat de cluster.** À partir de votre Key Vault dans Portail Azure, sélectionnez **Certificats** > *Le certificat souhaité* > **Identificateur de secret** :

    ```powershell
    $certUrlValue="https://sftestupgradegroup.vault.azure.net/secrets/sftestupgradegroup20200309235308/dac0e7b7f9d4414984ccaa72bfb2ea39"
    ```

* **L’empreinte de votre certificat de cluster.** (Vous l’avez probablement déjà si vous [vous êtes connecté au cluster initial](#connect-to-the-new-cluster-and-check-health-status) pour vérifier son état d’intégrité.) À partir du même panneau de certificat (**Certificats** > *Le certificat souhaité*) dans Portail Azure, copiez **l’empreinte SHA-1 X.509 (en hexadécimal)**  :

    ```powershell
    $thumb = "BB796AA33BD9767E7DA27FE5182CF8FDEE714A70"
    ```

* **L’ID de ressource de votre Key Vault.** À partir de votre Key Vault dans Portail Azure, sélectionnez **Propriétés** > **ID de ressource** :

    ```powershell
    $sourceVaultValue = "/subscriptions/########-####-####-####-############/resourceGroups/sftestupgradegroup/providers/Microsoft.KeyVault/vaults/sftestupgradegroup"
    ```

### <a name="deploy-the-updated-template"></a>Déployer le modèle mis à jour

Ajustez `parameterFilePath` et `templateFilePath` selon les besoins, puis exécutez la commande suivante :

```powershell
# Deploy the new scale set (upgraded to use managed disks) into the primary node type.
$templateFilePath = "C:\Upgrade-1NodeType-2ScaleSets-ManagedDisks.json"
$parameterFilePath = "C:\Upgrade-1NodeType-2ScaleSets-ManagedDisks.parameters.json"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath `
    -CertificateThumbprint $thumb `
    -CertificateUrlValue $certUrlValue `
    -SourceVaultValue $sourceVaultValue `
    -Verbose
```

Une fois le déploiement terminé, vérifiez à nouveau l’intégrité du cluster et assurez-vous que les dix nœuds (cinq sur l’original et cinq sur le nouveau groupe identique) sont intègres.

```powershell
Get-ServiceFabricClusterHealth
```

## <a name="migrate-seed-nodes-to-the-new-scale-set"></a>Migrer les nœuds initiaux vers le nouveau groupe identique

Nous sommes maintenant prêts à désactiver les nœuds du groupe identique d’origine. À mesure que ces nœuds sont désactivés, les services système et les nœuds initiaux (seed) migrent vers les machines virtuelles du nouveau groupe identique, car ils sont également marqués comme type de nœud principal.

```powershell
# Disable the nodes in the original scale set.
$nodeNames = @("_NTvm1_0","_NTvm1_1","_NTvm1_2","_NTvm1_3","_NTvm1_4")

Write-Host "Disabling nodes..."
foreach($name in $nodeNames){
    Disable-ServiceFabricNode -NodeName $name -Intent RemoveNode -Force
}
```

Utilisez Service Fabric Explorer pour surveiller la migration des nœuds initiaux vers le nouveau groupe identique et la progression des nœuds dans le groupe identique d’origine de l’état *Désactivation* à *Désactivé*.

![Service Fabric Explorer présentant l’état de nœuds désactivés](./media/upgrade-managed-disks/service-fabric-explorer-node-status.png)

> [!NOTE]
> L’exécution de l’opération de désactivation sur tous les nœuds du groupe identique d’origine peut prendre un certain temps. Pour garantir la cohérence des données, un seul nœud initial peut changer à la fois. Chaque modification du nœud initial requiert une mise à jour du cluster ; par conséquent, le remplacement d’un nœud initial nécessite deux mises à niveau du cluster (une pour l’ajout et l’autre pour la suppression de nœuds). La mise à niveau des cinq nœuds initiaux dans cet exemple de scénario entraîne la mise à niveau de dix clusters.

## <a name="remove-the-original-scale-set"></a>Supprimer le groupe identique d’origine

Une fois l’opération de désactivation terminée, supprimez le groupe identique.

```powershell
# Remove the original scale set
$scaleSetName = "NTvm1"

Remove-AzVmss `
    -ResourceGroupName $resourceGroupName `
    -VMScaleSetName $scaleSetName `
    -Force

Write-Host "Removed scale set $scaleSetName"
```

Dans Service Fabric Explorer, les nœuds supprimés (et donc l’*état d’intégrité du cluster*) s’affichent désormais à l’état *Erreur*.

![Service Fabric Explorer présentant des nœuds désactivés à l’état Erreur](./media/upgrade-managed-disks/service-fabric-explorer-disabled-nodes-error-state.png)

Supprimez les nœuds obsolètes du cluster Service Fabric pour restaurer l’état d’intégrité du cluster sur *OK*.

```powershell
# Remove node states for the deleted scale set
foreach($name in $nodeNames){
    Remove-ServiceFabricNodeState -NodeName $name -TimeoutSec 300 -Force
    Write-Host "Removed node state for node $name"
}
```

![Service Fabric Explorer avec suppression des nœuds à l’état Erreur](./media/upgrade-managed-disks/service-fabric-explorer-healthy-cluster.png)

## <a name="next-steps"></a>Étapes suivantes

Dans cette procédure pas à pas, vous avez appris à mettre à niveau les groupes de machines virtuelles identiques d’un cluster Service Fabric pour utiliser des disques managés tout en évitant les interruptions de service pendant le processus. Pour plus d’informations sur les rubriques connexes, consultez les ressources suivantes.

Découvrez comment :

* [Monter en puissance un type de nœud principal de cluster Service Fabric](service-fabric-scale-up-primary-node-type.md)

* [Convertir un modèle de groupe identique pour utiliser des disques managés](../virtual-machine-scale-sets/virtual-machine-scale-sets-convert-template-to-md.md)

* [Supprimer un type de nœud Service Fabric](service-fabric-how-to-remove-node-type.md)

Voir aussi :

* [Exemple : Mettre à niveau des nœuds de cluster pour utiliser des disques managés Azure](https://github.com/microsoft/service-fabric-scripts-and-templates/tree/master/templates/nodetype-upgrade-no-outage)

* [Considérations relatives à la mise à l’échelle verticale](service-fabric-best-practices-capacity-scaling.md#vertical-scaling-considerations)