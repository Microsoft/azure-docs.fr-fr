---
title: Résoudre les problèmes liés aux artefacts dans Azure DevTest Labs | Microsoft Docs
description: Découvrez comment résoudre les problèmes qui se produisent lors de l’application d’artefacts dans une machine virtuelle Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: a89b675a1b3bf134b98e09c7278f0eccb594c325
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85483191"
---
# <a name="troubleshoot-issues-when-applying-artifacts-in-an-azure-devtest-labs-virtual-machine"></a>Résoudre les problèmes liés à l’application d’artefacts dans une machine virtuelle Azure DevTest Labs
L’application d’artefacts sur une machine virtuelle peut échouer pour différentes raisons. Cet article décrit différentes méthodes pour vous aider à identifier les causes possibles.

Si vous avez besoin d’aide supplémentaire concernant n’importe quel point de cet article, contactez les experts Azure DevTest Labs (DTL) sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/), puis sélectionnez Obtenir de l’aide.   

> [!NOTE]
> Cet article s’applique aux machines virtuelles Windows et non Windows. Bien qu’il existe quelques différences, elles sont identifiées de manière explicite dans cet article.

## <a name="quick-troubleshooting-steps"></a>Étapes de dépannage rapide
Vérifiez que la machine virtuelle est en cours d’exécution. DevTest Labs exige que la machine virtuelle soit en cours d’exécution et que l’[agent de machine virtuelle Microsoft Azure](../virtual-machines/extensions/agent-windows.md) soit installé et prêt.

> [!TIP]
> Dans le **portail Azure**, accédez à la page **Gérer les artefacts** correspondant à la machine virtuelle afin de déterminer si celle-ci est prête pour l’application d’artefacts. Un message s’affiche tout en haut de cette page. 
> 
> À l’aide d’**Azure PowerShell**, examinez l’indicateur **canApplyArtifacts**, qui est retourné uniquement quand vous développez une opération Get. Consultez l’exemple de commande suivant :

```powershell
Select-AzSubscription -SubscriptionId $SubscriptionId | Out-Null
$vm = Get-AzResource `
        -Name "$LabName/$VmName" `
        -ResourceGroupName $LabRgName `
        -ResourceType 'microsoft.devtestlab/labs/virtualmachines' `
        -ApiVersion '2018-10-15-preview' `
        -ODataQuery '$expand=Properties($expand=ComputeVm)'
$vm.Properties.canApplyArtifacts
```

## <a name="ways-to-troubleshoot"></a>Comment résoudre les problèmes 
Vous pouvez résoudre les problèmes affectant les machines virtuelles créées à l’aide de DevTest Labs et du modèle de déploiement Resource Manager en appliquant l’une des méthodes suivantes :

- **Portail Azure** : idéal si vous souhaitez obtenir rapidement une indication visuelle de ce qui peut être à l’origine du problème.
- **Azure PowerShell** : si vous êtes familiarisé avec les invites PowerShell, interrogez rapidement les ressources DevTest Labs à l’aide des applets de commande Azure PowerShell.

> [!TIP]
> Pour plus d’informations sur la façon de passer en revue l’exécution d’artefacts au sein d’une machine virtuelle, consultez [Diagnostiquer les échecs d’artefacts dans le labo](devtest-lab-troubleshoot-artifact-failure.md).

## <a name="symptoms-causes-and-potential-resolutions"></a>Symptômes, causes et solutions possibles 

### <a name="artifact-appears-to-stop-responding"></a>L'artefact semble cesser de répondre

Un artefact semble cesser de répondre jusqu'à l'expiration d'un délai prédéfini, et la mention **Échec** apparaît.

Quand un artefact semble se bloquer, commencez par déterminer où il est bloqué. Un artefact peut être bloqué à l’une des étapes suivantes lors de l’exécution :

- **Pendant la requête initiale**. DevTest Labs crée un modèle Azure Resource Manager pour demander l’utilisation de l’extension de script personnalisé (CSE). Par conséquent, en arrière-plan, un déploiement de groupe de ressources est déclenché. Quand une erreur à ce niveau se produit, vous pouvez obtenir des détails dans les **journaux d’activité** du groupe de ressources de la machine virtuelle en question.  
    - Vous pouvez accéder au journal d’activité à partir de la barre de navigation de la page de la machine virtuelle de labo. Quand vous sélectionnez cette option, vous voyez une entrée pour l’**application d’artefacts à la machine virtuelle** (si l’opération d’application des artefacts a été déclenchée directement) ou pour **Ajouter ou modifier des machines virtuelles** (si l’opération d’application des artefacts faisait partie du processus de création de la machine virtuelle).
    - Recherchez les erreurs dans ces entrées. Parfois, l’erreur n’est pas marquée de manière précise, et vous devez examiner chaque entrée.
    - Quand vous examinez les détails de chaque entrée, veillez à examiner le contenu de la charge utile JSON. Vous pouvez voir une erreur en bas de ce document.
- **Lors de la tentative d’exécution de l’artefact**. Cela peut être dû à des problèmes de réseau ou de stockage. Pour plus d’informations, consultez la section correspondante plus loin dans cet article. Cela peut également être dû à la façon dont le script est créé. Par exemple :
    - Un script PowerShell a des **paramètres obligatoires**, mais l’un d’eux ne lui transmet pas de valeur, soit parce que vous autorisez l’utilisateur à le laisser vide, soit parce que vous n’avez pas de valeur par défaut pour la propriété dans le fichier de définition artifactfile.json. Le script ne répond plus car il attend une entrée de l'utilisateur.
    - Un script PowerShell **exige une entrée utilisateur** dans le cadre de l’exécution. Les scripts doivent être écrits pour fonctionner en mode silencieux sans intervention de l’utilisateur.
- **Il faut beaucoup de temps à l’agent de machine virtuelle pour être prêt**. Quand la machine virtuelle est démarrée pour la première fois, ou quand l’extension de script personnalisé est installée initialement pour répondre à la requête d’application des artefacts, la machine virtuelle peut exiger une mise à niveau de l’agent de machine virtuelle ou attendre l’initialisation de l’agent de machine virtuelle. Il peut y avoir des services dont dépend l’agent de machine virtuelle qui prennent beaucoup de temps pour s’initialiser. Dans ce cas, consultez [Vue d’ensemble d’agent de machine virtuelle Azure](../virtual-machines/extensions/agent-windows.md) pour plus d’informations sur le dépannage.

### <a name="to-verify-if-the-artifact-appears-to-stop-responding-because-of-the-script"></a>Pour déterminer si l'artefact ne répond plus à cause du script

1. Connectez-vous à la machine virtuelle en question.
2. Copiez le script localement sur la machine virtuelle ou recherchez-le sur la machine virtuelle sous `C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\<version>`. Il s’agit de l’emplacement où les scripts d’artefact sont téléchargés.
3. À l’aide d’une invite de commandes avec élévation de privilèges, exécutez le script localement, en fournissant les mêmes valeurs de paramètres que celles ayant causé le problème.
4. Déterminez si le script présente un comportement indésirable. Si c’est le cas, demandez une mise à jour de l’artefact (s’il provient du référentiel public) ou effectuez les corrections vous-même (s’il provient de votre référentiel privé).

> [!TIP]
> Vous pouvez corriger les problèmes liés aux artefacts hébergés dans notre [référentiel public](https://github.com/Azure/azure-devtestlab) et soumettre les modifications pour notre examen et approbation. Consultez la section **Contributions** dans le document [README.md](https://github.com/Azure/azure-devtestlab/blob/master/Artifacts/README.md).
> 
> Pour plus d’informations sur l’écriture de vos propres artefacts, consultez le document [AUTHORING.md](https://github.com/Azure/azure-devtestlab/blob/master/Artifacts/AUTHORING.md).

### <a name="to-verify-if-the-artifact-appears-to-stop-responding-because-of-the-vm-agent"></a>Pour déterminer si l'artefact ne répond plus à cause de l'agent de machine virtuelle
1. Connectez-vous à la machine virtuelle en question.
2. À l’aide de l’Explorateur de fichiers, accédez à **C:\WindowsAzure\logs**.
3. Recherchez et ouvrez le fichier **WaAppAgent.log**.
4. Recherchez les entrées qui indiquent quand l’agent de machine virtuelle démarre et quand il termine l’initialisation (autrement dit, quand la première pulsation est envoyée). Privilégiez les entrées les plus récentes, ou plus précisément celles proches du moment où vous rencontrez le problème.

    ```
    [00000006] [11/14/2019 05:52:13.44] [INFO]  WindowsAzureGuestAgent starting. Version 2.7.41491.949
    ...
    [00000006] [11/14/2019 05:52:31.77] [WARN]  Waiting for OOBE to Complete ...
    ...
    [00000006] [11/14/2019 06:02:30.43] [WARN]  Waiting for OOBE to Complete ...
    [00000006] [11/14/2019 06:02:33.43] [INFO]  StateExecutor initialization completed.
    [00000020] [11/14/2019 06:02:33.43] [HEART] WindowsAzureGuestAgent Heartbeat.
    ```
    Dans cet exemple, vous pouvez voir que le démarrage de l’agent de machine virtuelle a duré 10 minutes et 20 secondes, car une pulsation a été envoyée. Ici, c’est le service OOBE qui a pris beaucoup de temps à démarrer.

> [!TIP]
> Pour obtenir des informations générales sur les extensions Azure, consultez [Extensions et fonctionnalités des machines virtuelles Azure](../virtual-machines/extensions/overview.md).

## <a name="storage-errors"></a>Erreurs de stockage
DevTest Labs exige l’accès au compte de stockage du laboratoire créé pour mettre en cache les artefacts. Quand DevTest Labs applique un artefact, il lit la configuration de l’artefact et ses fichiers à partir des référentiels configurés. Par défaut, DevTest Labs configure l’accès aux **référentiels d’artefacts publics**.

En fonction de sa configuration, il se peut que la machine virtuelle n’ait pas un accès direct à ce référentiel. Ainsi, par défaut, DevTest Labs met en cache les artefacts dans un compte de stockage créé lors de la première initialisation du laboratoire.

Si l’accès à ce compte de stockage est bloqué de quelque manière que ce soit, par exemple quand le trafic est bloqué de la machine virtuelle vers le service Stockage Azure, une erreur semblable à la suivante peut s’afficher :

```shell
CSE Error: Failed to download all specified files. Exiting. Exception: Microsoft.WindowsAzure.Storage.StorageException: The remote server returned an error: (403) Forbidden. ---> System.Net.WebException: The remote server returned an error: (403) Forbidden.
```

L’erreur ci-dessus apparaît dans la section **Message de déploiement** de la page **Résultats de l’artefact** sous **Gérer les artefacts**. Elle s’affiche également dans les **journaux d’activité** sous le groupe de ressources de la machine virtuelle en question.

### <a name="to-ensure-communication-to-the-azure-storage-service-isnt-being-blocked"></a>Pour garantir que la communication avec le service Stockage Azure n’est pas bloquée

- **Vérifiez si des groupes de sécurité réseau ont été ajoutés**. Il se peut qu’une stratégie d’abonnement ait été ajoutée, dans laquelle des groupes de sécurité réseau sont configurés automatiquement dans tous les réseaux virtuels. Cela affecterait également le réseau virtuel par défaut du laboratoire, s’il est utilisé, ou tout autre réseau virtuel configuré dans votre laboratoire et utilisé pour la création de machines virtuelles.
- **Vérifiez le compte de stockage du labo par défaut** (premier compte de stockage créé lors de la création du labo, dont le nom commence généralement par la lettre « a » et se termine par un nombre à plusieurs chiffres, autrement dit a\<labname\>#).
    1. Accédez au groupe de ressources du laboratoire.
    2. Recherchez la ressource de type **compte de stockage**, dont le nom correspond à la convention.
    3. Accédez à la page du compte de stockage nommée **Pare-feu et réseaux virtuels**.
    4. Vérifiez qu’il est défini sur **Tous les réseaux**. Si l’option **Réseaux sélectionnés** est sélectionnée, vérifiez que les réseaux virtuels du laboratoire utilisés pour créer des machines virtuelles sont ajoutés à la liste.

Pour obtenir des informations de dépannage détaillées, consultez [Configurer Pare-feu et réseaux virtuels dans Stockage Azure](../storage/common/storage-network-security.md).

> [!TIP]
> **Vérifiez les règles du groupe de sécurité réseau**. Utilisez la [vérification des flux IP](../network-watcher/diagnose-vm-network-traffic-filtering-problem.md#use-ip-flow-verify) pour confirmer qu’une règle d’un groupe de sécurité réseau bloque le trafic vers ou à partir d’une machine virtuelle. Vous pouvez également vérifier les règles de groupe de sécurité effectives pour vous assurer que la règle **Allow** entrante du groupe de sécurité réseau existe. Pour en savoir plus, voir [Utilisation de règles de sécurité effectives pour résoudre des problèmes de flux de trafic de machine virtuelle](../virtual-network/diagnose-network-traffic-filter-problem.md).

## <a name="other-sources-of-error"></a>Autres sources d’erreurs
Il existe d’autres sources d’erreurs moins fréquentes. Veillez à évaluer chacune d’elles pour déterminer si elle s’applique à votre cas. Voici l’une d’entre elles : 

- **Jeton d’accès personnel expiré pour le référentiel privé**. Une fois expiré, l’artefact n’est pas listé et les scripts qui font référence à des artefacts d’un référentiel avec un jeton d’accès privé expiré échouent en conséquence.

## <a name="next-steps"></a>Étapes suivantes
Si aucune de ces erreurs ne s’est produite et que vous ne pouvez toujours pas appliquer d’artefacts, vous pouvez soumettre un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.
