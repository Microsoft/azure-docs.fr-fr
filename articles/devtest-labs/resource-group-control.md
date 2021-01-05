---
title: Spécifier un groupe de ressources pour les machines virtuelles dans Azure DevTest Labs | Microsoft Docs
description: Découvrez comment spécifier un groupe de ressources pour les machines virtuelles dans Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 7b72048405d3025ca21b324b6ad3168dd0c9ac95
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85483361"
---
# <a name="specify-a-resource-group-for-lab-virtual-machines-in-azure-devtest-labs"></a>Spécifier un groupe de ressources pour les machines virtuelles de labo dans Azure DevTest Labs

En tant que propriétaire de labo, vous pouvez configurer les machines virtuelles de votre labo de manière à ce qu’elles soient créées dans un groupe de ressources en particulier. Cette fonctionnalité vous aide dans les scénarios suivants :

- Vous disposez de moins de groupes de ressources créés par les laboratoires dans votre abonnement.
- Vos laboratoires fonctionnent au sein d’un ensemble fixe de groupes de ressources configurés par vos soins.
- Vous contournez les restrictions et les approbations requises pour créer des groupes de ressources au sein de votre abonnement Azure.
- Vous consolidez toutes vos ressources de laboratoire au sein d’un seul groupe de ressources pour simplifier le suivi de ces ressources et l’application de [stratégies](../governance/policy/overview.md) afin de gérer les ressources au niveau du groupe de ressources.

Avec cette fonctionnalité, vous pouvez utiliser un script afin de spécifier un nouveau groupe de ressources ou un groupe de ressources existant dans votre abonnement Azure pour toutes les machines virtuelles de votre labo. Pour le moment, Azure DevTest Labs prend en charge cette fonctionnalité par le biais d’une API.

> [!NOTE]
> Toutes les limites d’abonnement s’appliquent lorsque vous créez des labos dans DevTest Labs. Considérez un labo comme n’importe quelle autre ressource dans votre abonnement. Dans le cas des groupes de ressources, la limite est de [980 groupes de ressources par abonnement](../azure-resource-manager/management/azure-subscription-service-limits.md#subscription-limits). 

## <a name="use-azure-portal"></a>Utiliser le portail Azure
Suivez ces étapes pour spécifier un groupe de ressources pour toutes les machines virtuelles créées dans le labo. 

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le menu de navigation de gauche, sélectionnez **Tous les services**. 
3. Sélectionnez **DevTest Labs** dans la liste.
4. Dans la liste de labos, sélectionnez votre **labo**.  
5. Sélectionnez **Configuration et stratégies** dans la section **Paramètres** du menu à gauche. 
6. Sélectionnez **Paramètres du laboratoire** dans le menu de gauche. 
7. Sélectionnez **Toutes les machines virtuelles dans un groupe de ressources**. 
8. Sélectionnez un groupe de ressources existant dans la liste déroulante ou sélectionnez **Créer**, donnez un **nom** au groupe de ressources puis sélectionnez **OK**. 

    ![Sélectionner le groupe de ressources pour toutes les machines virtuelles du laboratoire](./media/resource-group-control/select-resource-group.png)

## <a name="use-powershell"></a>Utiliser PowerShell 
L’exemple suivant montre comment utiliser un script PowerShell pour créer toutes les machines virtuelles du labo dans un nouveau groupe de ressources.

```powershell
[CmdletBinding()]
Param(
    $subId,
    $labRg,
    $labName,
    $vmRg
)

az login | out-null

az account set --subscription $subId | out-null

$rgId = "/subscriptions/"+$subId+"/resourceGroups/"+$vmRg

"Updating lab '$labName' with vm rg '$rgId'..."

az resource update -g $labRg -n $labName --resource-type "Microsoft.DevTestLab/labs" --api-version 2018-10-15-preview --set properties.vmCreationResourceGroupId=$rgId

"Done. New virtual machines will now be created in the resource group '$vmRg'."
```

Invoquez le script en utilisant la commande suivante. ResourceGroup.ps1 est le fichier qui contient le script précédent :

```powershell
.\ResourceGroup.ps1 -subId <subscriptionID> -labRg <labRGNAme> -labName <LanName> -vmRg <RGName> 
```

## <a name="use-an-azure-resource-manager-template"></a>Utiliser un modèle Azure Resource Manager
Si vous utilisez un modèle Azure Resource Manager pour créer un laboratoire, utilisez la propriété **vmCreationResourceResourceGroupId** dans la section propriétés du labo de votre modèle comme indiqué dans l’exemple suivant :

```json
        {
            "type": "microsoft.devtestlab/labs",
            "name": "[parameters('lab_name')]",
            "apiVersion": "2018_10_15_preview",
            "location": "eastus",
            "tags": {},
            "scale": null,
            "properties": {
                "vmCreationResourceGroupId": "/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>",
                "labStorageType": "Premium",
                "premiumDataDisks": "Disabled",
                "provisioningState": "Succeeded",
                "uniqueIdentifier": "000000000f-0000-0000-0000-00000000000000"
            },
            "dependsOn": []
        },
```


## <a name="api-to-configure-a-resource-group-for-lab-vms"></a>API de configuration d’un groupe de ressources pour des machines virtuelles de labo
Vous disposez des options suivantes en tant que propriétaire du labo lors de l’utilisation de cette API :

- Choisissez le **groupe de ressources du labo** pour toutes les machines virtuelles.
- Choisissez un **groupe de ressources existant** différent du groupe de ressources du labo pour toutes les machines virtuelles.
- Entrez un nom de **nouveau groupe de ressources** pour toutes les machines virtuelles.
- Continuez d’utiliser le comportement existant par défaut, dans lequel un groupe de ressources est créé pour chaque machine virtuelle du labo.
 
Ce paramètre s’applique aux nouvelles machines virtuelles créées dans le labo. Les anciennes machines virtuelles de votre labo qui ont été créées dans leurs propres groupes de ressources ne sont pas affectées par ces modifications. Les environnements créés dans votre labo demeurent dans leurs groupes de ressources.

Comment utiliser cette API :
- Utiliser l’API de version **2018_10_15_preview**.
- Si vous spécifiez un nouveau groupe de ressources, assurez-vous de disposer des **autorisations d’écriture sur les groupes de ressources** dans votre abonnement. Si vous manquez d’autorisations d’écriture, la création de nouvelles machines virtuelles dans le groupe de ressources spécifié entraîne un échec.
- Lors de l’utilisation de l’API, passez l’**ID complet du groupe de ressources**. Par exemple : `/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroupName>`. Assurez-vous que le groupe de ressources fait partie du même abonnement que le labo. 


## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants : 

- [Définir des stratégies pour un laboratoire](devtest-lab-set-lab-policy.md)
- [Forum Aux Questions (FAQ)](devtest-lab-faq.md)
