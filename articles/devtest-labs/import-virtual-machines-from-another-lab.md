---
title: Importer des machines virtuelles depuis un autre labo dans Azure DevTest Labs
description: Cet article explique comment importer des machines virtuelles à partir d’un autre lab vers le lab actuel dans Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 0f664a0ae399575ee936565adaf7364fd1c5ce5c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85475932"
---
# <a name="import-virtual-machines-from-another-lab-in-azure-devtest-labs"></a>Importer des machines virtuelles depuis un autre labo dans Azure DevTest Labs
Cet article fournit des informations sur la façon d’importer des machines virtuelles depuis un autre labo dans votre labo.

## <a name="scenarios"></a>Scénarios
Voici quelques scénarios où vous avez besoin d’importer des machines virtuelles depuis un labo dans un autre labo :

- Une personne de l’équipe passe à un autre groupe au sein de l’entreprise et veut apporter le poste de travail de développement aux DevTest Labs de la nouvelle équipe.
- Le groupe a atteint un [quota au niveau de l’abonnement](../azure-resource-manager/management/azure-subscription-service-limits.md) et veut diviser les équipes en plusieurs abonnements.
- L’entreprise passe à Express Route (ou à une autre topologie réseau) et l’équipe veut déplacer les machines virtuelles afin d’utiliser cette nouvelle infrastructure.

## <a name="solution-and-constraints"></a>Solution et contraintes
Cette fonctionnalité vous permet d’importer des machines virtuelles depuis un labo (source) dans un autre labo (destination). Vous pouvez donner un nouveau nom pour la machine virtuelle de destination dans le processus. Le processus d’importation inclut toutes les dépendances, comme les disques, les planifications, les paramètres réseau, etc.

Le processus prend un certain temps et est impacté par les facteurs suivants :

- Nombre/taille des disques attachés à la machine source (car il s’agit d’une opération de copie et non pas d’une opération de déplacement)
- Distance à la destination (par exemple, de la région USA Est vers la région Asie Sud-est).

Une fois le processus terminé, la machine virtuelle source reste arrêtée et la nouvelle est en cours d’exécution dans le labo de destination.

Vous devez avoir deux contraintes principales en tête quand vous planifiez l’importation de machines virtuelles depuis un labo vers un autre labo :

- Les importations de machines virtuelles entre des abonnements et entre des régions sont prises en charge, mais les abonnements doivent être associés au même locataire Azure Active Directory.
- Les machines virtuelles ne doivent pas être dans un état revendicable dans le labo source.
- Vous êtes propriétaire de la machine virtuelle dans le labo source et propriétaire du labo dans le labo de destination.
- Actuellement, cette fonctionnalité est prise en charge seulement via Powershell et l’API REST.

## <a name="use-powershell"></a>Utiliser PowerShell
Téléchargez le fichier ImportVirtualMachines.ps1 depuis [GitHub](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/Scripts/ImportVirtualMachines). Vous pouvez utiliser le script pour importer une seule machine virtuelle ou toutes les machines virtuelles du labo source dans le labo de destination.

### <a name="use-powershell-to-import-a-single-vm"></a>Utiliser PowerShell pour importer une seule machine virtuelle
L’exécution de ce script PowerShell nécessite l’identification de la machine virtuelle source et du labo de destination, et vous pouvez aussi spécifier un nouveau nom à utiliser pour la machine de destination :

```powershell
./ImportVirtualMachines.ps1 -SourceSubscriptionId "<ID of the subscription that contains the source lab>" `
                            -SourceDevTestLabName "<Name of the source lab>" `
                            -SourceVirtualMachineName "<Name of the VM to be imported from the source lab> " `
                            -DestinationSubscriptionId "<ID of the subscription that contians the destination lab>" `
                            -DestinationDevTestLabName "<Name of the destination lab>" `
                            -DestinationVirtualMachineName "<Optional: specify a new name for the imported VM in the destination lab>"
```

### <a name="use-powershell-to-import-all-vms-in-the-source-lab"></a>Utiliser PowerShell pour importer toutes les machines virtuelles du labo source
Si la machine virtuelle source n’est pas spécifiée, le script importe automatiquement toutes les machines virtuelles dans les DevTest Labs.  Par exemple :

```powershell
./ImportVirtualMachines.ps1 -SourceSubscriptionId "<ID of the subscription that contains the source lab>" `
                            -SourceDevTestLabName "<Name of the source lab>" `
                            -DestinationSubscriptionId "<ID of the subscription that contians the destination lab>" `
                            -DestinationDevTestLabName "<Name of the destination lab>"
```

## <a name="use-http-rest-to-import-a-vm"></a>Utiliser HTTP REST pour importer une machine virtuelle
L’appel REST est simple. Vous donnez suffisamment d’informations pour identifier les ressources sources et de destination. Rappelez-vous que l’opération s’effectue sur la ressource du labo de destination.

```REST
POST https://management.azure.com/subscriptions/<DestinationSubscriptionID>/resourceGroups/<DestinationResourceGroup>/providers/Microsoft.DevTestLab/labs/<DestinationLab>/ImportVirtualMachine?api-version=2017-04-26-preview
{
   sourceVirtualMachineResourceId: "/subscriptions/<SourceSubscriptionID>/resourcegroups/<SourceResourceGroup>/providers/microsoft.devtestlab/labs/<SourceLab>/virtualmachines/<NameofVMTobeImported>",
   destinationVirtualMachineName: "<NewNameForImportedVM>"
}
```

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- [Définir des stratégies pour un laboratoire](devtest-lab-set-lab-policy.md)
- [Forum Aux Questions (FAQ)](devtest-lab-faq.md)
