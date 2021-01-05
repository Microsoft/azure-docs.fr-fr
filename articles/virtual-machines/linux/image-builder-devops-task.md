---
title: Tâche DevOps du service Azure Image Builder
description: Tâche Azure DevOps pour injecter des artefacts de build dans une image de machine virtuelle afin de pouvoir installer et configurer votre application et votre système d’exploitation.
author: danielsollondon
ms.author: danis
ms.date: 08/10/2020
ms.topic: article
ms.service: virtual-machines
ms.subservice: imaging
ms.openlocfilehash: a3016900b6265bfd56ad1a5a71f70efc01181af5
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499252"
---
# <a name="azure-image-builder-service-devops-task"></a>Tâche DevOps du service Azure Image Builder

Cet article vous explique comment utiliser une tâche Azure DevOps pour injecter des artefacts de build dans une image de machine virtuelle afin de pouvoir installer et configurer votre application et votre système d’exploitation.

## <a name="devops-task-versions"></a>Versions des tâches DevOps
Il existe deux tâches DevOps Azure VM Image Builder (AIB) :

* [La tâche « stable » AIB](https://marketplace.visualstudio.com/items?itemName=AzureImageBuilder.devOps-task-for-azure-image-builder), qui nous permet de placer les dernières mises à jour et fonctionnalités, d’autoriser les clients à les tester avant de les promouvoir à la tâche « stable », environ 1 semaine plus tard. 


* [La tâche « instable » AIB](https://marketplace.visualstudio.com/items?itemName=AzureImageBuilder.devOps-task-for-azure-image-builder-canary), qui nous permet de placer les dernières mises à jour et fonctionnalités, d’autoriser les clients à les tester avant de les promouvoir à la tâche « stable ». S’il n’y a aucun problème signalé et que notre télémétrie n’indique aucun problème, environ 1 semaine plus tard, nous allons promouvoir le code de tâche sur « stable ». 

## <a name="prerequisites"></a>Conditions préalables requises

* Installez la [tâche stable DevOps à partir de Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=AzureImageBuilder.devOps-task-for-azure-image-builder).
* Vous devez disposer d’un compte VSTS DevOps et d’un pipeline de build créé
* Inscrire et activer les exigences de la fonctionnalité Image Builder dans l’abonnement utilisé par les pipelines :
    * [Az PowerShell](../windows/image-builder-powershell.md#register-features)
    * [Az CLI](../windows/image-builder.md#register-the-features)
    
* Créez un compte de Stockage Azure standard dans le groupe de ressources d’images sources, vous pouvez utiliser d’autres comptes de groupe de ressources/stockage. Le compte de stockage est utilisé pour transférer les artefacts de build à partir de la tâche DevOps vers l’image.

    ```powerShell
    # Az PowerShell
    $timeInt=$(get-date -UFormat "%s")
    $storageAccName="aibstorage"+$timeInt
    $location=westus
    # create storage account and blob in resource group
    New-AzStorageAccount -ResourceGroupName $strResourceGroup -Name $storageAccName -Location $location -SkuName Standard_LRS
    ```

    ```bash
    # Az CLI
    location=westus
    scriptStorageAcc=aibstordot$(date +'%s')
    # create storage account and blob in resource group
    az storage account create -n $scriptStorageAcc -g $strResourceGroup -l $location --sku Standard_LRS
    ```

## <a name="add-task-to-release-pipeline"></a>Ajouter une tâche à un pipeline de mise en production

Sélectionner **Pipeline de mise en production** > **Modifier**

Sur l’agent utilisateur, sélectionnez *+* pour ajouter, puis recherchez **Image Builder**. Sélectionnez **Ajouter**.

Définissez les propriétés de la tâche suivantes :

### <a name="azure-subscription"></a>Abonnement Azure

Dans le menu déroulant l’abonnement que vous souhaitez que Image Builder exécute. Utilisez le même abonnement que celui où se trouvent vos images sources et où les images doivent être distribuées. Vous devez autoriser l’accès du contributeur du générateur d’images à l’abonnement ou au groupe de ressources.

### <a name="resource-group"></a>Groupe de ressources

Utilisez le groupe de ressources dans lequel l’artefact du modèle d’image temporaire sera stocké. Lors de la création d’un artefact du modèle, un groupe de ressources Image Builder temporaire supplémentaire`IT_<DestinationResourceGroup>_<TemplateName>_guid` est créé. Le groupe de ressources temporaire stocke les métadonnées de l’image, tels que les scripts. À la fin de la tâche, le groupe de ressources d’artefact du modèle d’image et Image Builder temporaire est supprimé.
 
### <a name="location"></a>Emplacement

L’emplacement est la région dans laquelle Image Builder sera exécuté. Seul un nombre défini de [régions](../windows/image-builder-overview.md#regions) sont prises en charge. Les images sources doivent être présentes à cet emplacement. Par exemple, si vous utilisez Shared Image Gallery, un réplica doit exister dans cette région.

### <a name="managed-identity-required"></a>Identité managée (obligatoire)
Image Builder nécessite une identité managée, qu’il utilise pour lire les images personnalisées sources, se connecter au Stockage Azure et créer des images personnalisées. Consultez [ce document](./image-builder-overview.md#permissions) pour plus d’informations.

### <a name="vnet-support"></a>Prise en charge du réseau virtuel

Actuellement, la tâche DevOps ne prend pas en charge la spécification d’un sous-réseau existant, elle se trouve sur la feuille de route, mais si vous souhaitez utiliser un réseau virtuel existant, vous pouvez utiliser un modèle Resource Manager, avec un modèle Image Builder imbriqué à l’intérieur, consultez les exemples de modèles Image Builder Windows pour savoir comment procéder ou utilisez [AZ AIB PowerShell](../windows/image-builder-powershell.md).

### <a name="source"></a>Source

Les images sources doivent être des systèmes d’exploitation Image Builder pris en charge. Vous pouvez choisir des images personnalisées existantes dans la même région que celle dans laquelle Image Builder s’exécute :
* Image managée : vous devez transmettre le resourceId, par exemple :
    ```json
    /subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/images/<imageName>
    ```
* Azure Shared Image Gallery : vous devez transmettre le resourceId de la version de l’image, par exemple :
    ```json
    /subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup/providers/Microsoft.Compute/galleries/$sigName/images/$imageDefName/versions/<versionNumber>
    ```

    Si vous avez besoin d’obtenir la dernière version de Shared Image Gallery, vous pouvez avoir une tâche AZ PowerShell ou AZ CLI pour obtenir la version la plus récente et définir une variable DevOps. Utilisez la variable dans la tâche AZ VM Image Builder DevOps. Pour plus d'informations, consultez les [exemples](https://github.com/danielsollondon/azvmimagebuilder/tree/master/solutions/8_Getting_Latest_SIG_Version_ResID#getting-the-latest-image-version-resourceid-from-shared-image-gallery).

* (Place de marché) Image de base : voici une liste déroulante d’images populaires, qui utilisent toujours la version « la plus récente » du système d’exploitation pris en charge. 

    Si l’image de base ne figure pas dans la liste, vous pouvez spécifier l’image exacte à l’aide de `Publisher:Offer:Sku`.

    Version de l’image de base (facultatif) : vous pouvez fournir la version de l’image que vous souhaitez utiliser, la valeur par défaut est `latest`.

### <a name="customize"></a>Personnaliser

#### <a name="provisioner"></a>Fournisseur

Initialement, deux personnalisations sont prises en charge : **Shell** et **PowerShell**. Seule la valeur inline est pris en charge. Si vous souhaitez télécharger des scripts, vous pouvez transmettre des commandes inlined pour ce faire.

Pour votre système d’exploitation, sélectionnez PowerShell ou Shell.

#### <a name="windows-update-task"></a>Tâche Windows Update

Pour Windows uniquement, la tâche exécute Windows Update à la fin des personnalisations. Il gère les redémarrages requis.

La configuration Windows Update suivante est exécutée :
```json
    "type": "WindowsUpdate",
    "searchCriteria": "IsInstalled=0",
    "filters": [
        "exclude:$_.Title -like '*Preview*'",
        "include:$true"
```
Il installe les mises à jour Windows Update importantes et recommandées qui ne sont pas une préversion.

#### <a name="handling-reboots"></a>Gestion des redémarrages
Actuellement, la tâche DevOps ne prend pas en charge le redémarrage des builds Windows. Si vous essayez de redémarrer avec du code PowerShell, la build échoue. Toutefois, vous pouvez utiliser du code pour redémarrer des builds Linux.

#### <a name="build-path"></a>Chemin d’accès à la build

La tâche est conçue pour être en mesure d’injecter des artefacts de version de build DevOps dans l’image. Pour que cela fonctionne, vous devez configurer un pipeline de build. Dans la configuration du pipeline de mise en version, vous devez ajouter le référentiel des artefacts de build.

:::image type="content" source="./media/image-builder-devops-task/add-artifact.png" alt-text="Sélection de l’option Ajouter un artefact dans le pipeline de mise en production.":::

Sélectionnez le bouton **Chemin d’accès à la build** pour choisir le dossier de build que vous souhaitez placer sur l’image. La tâche Image Builder copie tous les fichiers et répertoires qu’elle contient. Lorsque l’image est créée, Image Builder déploie les fichiers et les répertoires dans différents chemins d’accès, selon le système d’exploitation.

> [!IMPORTANT]
> Lorsque vous ajoutez un artefact de référentiel, vous pouvez constater que le répertoire est préfixé avec un trait de soulignement *_* . Le trait de soulignement peut provoquer des problèmes avec les commandes inlined. Utilisez les guillemets appropriés dans les commandes.
> 

L’exemple suivant explique son fonctionnement :

:::image type="content" source="./media/image-builder-devops-task/build-artifacts.png" alt-text="Une structure de répertoires qui présente la hiérarchie.":::


* Des fichiers Windows existent dans `C:\`. Un répertoire nommé `buildArtifacts` est créé et comprend le répertoire `webapp`.

* Des fichiers Linux existent dans `/tmp`. Le répertoire `webapp` est créé et inclut tous les fichiers et répertoires. Vous devez déplacer les fichiers de ce répertoire. Dans le cas contraire, ils seront supprimés car il se trouve dans le répertoire temporaire.

#### <a name="inline-customization-script"></a>Script de personnalisation inlined

* Windows : vous pouvez entrer des commandes inlined PowerShell séparées par des virgules. Si vous souhaitez exécuter un script dans votre répertoire de build, vous pouvez utiliser :

    ```PowerShell
    & 'c:\buildArtifacts\webapp\webconfig.ps1'
    ```

* Linux : sur les systèmes Linux, les artefacts de build sont placés dans le répertoire `/tmp`. Toutefois, sur un grand nombre de systèmes d’exploitation Linux, le contenu du répertoire /tmp est supprimé en cas de redémarrage. Si vous souhaitez que les artefacts existent dans l’image, vous devez créer un autre répertoire et les y copier.  Par exemple :

    ```bash
    sudo mkdir /lib/buildArtifacts
    sudo cp -r "/tmp/_ImageBuilding/webapp" /lib/buildArtifacts/.
    ```
    
    Si vous êtes d’accord pour utiliser le répertoire « /tmp », vous pouvez utiliser le code ci-dessous pour exécuter le script.
    
    ```bash
    # grant execute permissions to execute scripts
    sudo chmod +x "/tmp/_ImageBuilding/webapp/coreConfig.sh"
    echo "running script"
    sudo . "/tmp/AppsAndImageBuilderLinux/_WebApp/coreConfig.sh"
    ```
    
#### <a name="what-happens-to-the-build-artifacts-after-the-image-build"></a>Qu’advient-il des artefacts de build après la génération de l’image ?

> [!NOTE]
> Image Builder ne supprime pas automatiquement les artefacts de build, il est fortement recommandé de toujours disposer de code pour supprimer les artefacts de build.
> 

* Windows : Image Builder déploie les fichiers dans le répertoire `c:\buildArtifacts`. Le répertoire est persistant, vous devez supprimer le répertoire. Vous pouvez le supprimer dans le script que vous exécutez. Par exemple :

    ```PowerShell
    # Clean up buildArtifacts directory
    Remove-Item -Path "C:\buildArtifacts\*" -Force -Recurse
    
    # Delete the buildArtifacts directory
    Remove-Item -Path "C:\buildArtifacts" -Force 
    ```
    
* Linux : les artefacts de build sont placés dans le répertoire `/tmp`. Toutefois, sur un grand nombre de systèmes d’exploitation Linux, le contenu du répertoire `/tmp` est supprimé en cas de redémarrage. Il est recommandé de disposer de code pour supprimer le contenu et de ne pas compter sur le système d’exploitation pour supprimer le contenu. Par exemple :

    ```bash
    sudo rm -R "/tmp/AppsAndImageBuilderLinux"
    ```
    
#### <a name="total-length-of-image-build"></a>Longueur totale de la build d’image

La longueur totale ne peut pas encore être modifiée dans la tâche de pipeline DevOps. Il utilise la valeur par défaut de 240 minutes. Si vous souhaitez augmenter la [buildTimeoutInMinutes](./image-builder-json.md?bc=%2fazure%2fvirtual-machines%2fwindows%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#properties-buildtimeoutinminutes), vous pouvez utiliser une tâche AZ CLI dans le pipeline de mise en production. Configurez la tâche pour copier un modèle et l’envoyer. Pour obtenir un exemple, consultez cette [solution](https://github.com/danielsollondon/azvmimagebuilder/tree/master/solutions/4_Using_ENV_Variables#using-environment-variables-and-parameters-with-image-builder) ou utilisez AZ PowerShell.


#### <a name="storage-account"></a>Compte de stockage

Sélectionnez le compte de stockage que vous avez créé dans les composants requis. Si vous ne la voyez pas dans la liste, Image Builder ne dispose pas des autorisations appropriées.

Lorsque la build démarre, Image Builder crée un conteneur appelé `imagebuilder-vststask`. Le conteneur correspond à l’emplacement de stockage des artefacts de build du référentiel.

> [!NOTE]
> Vous devez supprimer manuellement le compte de stockage ou le conteneur après chaque build.
>

### <a name="distribute"></a>Distribuer

3 types de distribution sont pris en charge.

#### <a name="managed-image"></a>Image managée

* ResourceID :
    ```bash
    /subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/images/<imageName>
    ```

* Emplacements

#### <a name="azure-shared-image-gallery"></a>Azure Shared Image Gallery

Shared Image Gallery **doit** exister déjà.

* ResourceID : 
    ```bash
    /subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<imageDefName>
    ```

* Régions : liste des régions, séparées par des virgules. Par exemple, westus, eastus, centralus

#### <a name="vhd"></a>Disque dur virtuel (VHD)

Vous ne pouvez pas passer de valeurs à cette opération, Image Builder émettra le disque dur virtuel dans le groupe de ressources Image Builder temporaire, `IT_<DestinationResourceGroup>_<TemplateName>`, dans le conteneur *VHDS*. Quand vous démarrez la build de mise en production, le générateur d’images émet des journaux. Une fois l’opération terminée, l’URL du disque dur virtuel est émise.

### <a name="optional-settings"></a>Paramètres facultatifs

* [Taille de la machine virtuelle](image-builder-json.md#vmprofile) : vous pouvez remplacer la taille de la machine virtuelle à partir de la valeur par défaut *Standard_D1_v2*. Vous pouvez la remplacer pour réduire la durée totale de personnalisation ou parce que vous souhaitez créer les images qui dépendent de certaines tailles de machine virtuelle, telles que GPU / HPC, etc.

## <a name="how-it-works"></a>Fonctionnement

Lorsque vous créez la mise en production, la tâche crée un conteneur dans le compte de stockage, nommé *imagebuilder-vststask*. Il compresse et charge vos artefacts de build et crée un jeton SAP pour le fichier zip.

La tâche utilise les propriétés transmises à la tâche pour créer l’artefact du modèle Image Builder. Cette tâche effectue les actions suivantes :
* Elle télécharge le fichier zip de l’artefact de build et tous les autres scripts associés. Les fichiers sont enregistrés dans un compte de stockage dans le groupe de ressources Image Builder temporaire `IT_<DestinationResourceGroup>_<TemplateName>`.
* Elle crée un modèle dont le préfixe est *t_* et un entier monotone à 10 chiffres. Le modèle est enregistré dans le groupe de ressources que vous avez sélectionné. Le modèle existe pour la durée de la build dans le groupe de ressources. 

Exemple de sortie :

```text
start reading task parameters...
found build at:  /home/vsts/work/r1/a/_ImageBuilding/webapp
end reading parameters
getting storage account details for aibstordot1556933914
created archive /home/vsts/work/_temp/temp_web_package_21475337782320203.zip
Source for image:  { type: 'SharedImageVersion',
  imageVersionId: '/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<imageDefName>/versions/<imgVersionNumber>' }
template name:  t_1556938436xxx
starting put template...
```

Lors du démarrage de la génération de l’image, l’état d’exécution est signalé dans les journaux des mises en production :

```text
starting run template...
```

Une fois la tâche terminée, vous obtenez un résultat similaire au texte suivant :

```text
2019-05-06T12:49:52.0558229Z starting run template...
2019-05-06T13:36:33.8863094Z run template:  Succeeded
2019-05-06T13:36:33.8867768Z getting runOutput for  SharedImage_distribute
2019-05-06T13:36:34.6652541Z ==============================================================================
2019-05-06T13:36:34.6652925Z ## task output variables ##
2019-05-06T13:36:34.6658728Z $(imageUri) =  /subscriptions/<subscriptionID>/resourceGroups/aibwinsig/providers/Microsoft.Compute/galleries/my22stSIG/images/winWAppimages/versions/0.23760.13763
2019-05-06T13:36:34.6659989Z ==============================================================================
2019-05-06T13:36:34.6663500Z deleting template t_1557146959485...
2019-05-06T13:36:34.6673713Z deleting storage blob imagebuilder-vststask\webapp/18-1/webapp_1557146958741.zip
2019-05-06T13:36:34.9786039Z blob imagebuilder-vststask\webapp/18-1/webapp_1557146958741.zip is deleted
2019-05-06T13:38:37.4884068Z delete template:  Succeeded
```

Le modèle d’image et `IT_<DestinationResourceGroup>_<TemplateName>` sont supprimés.

Vous pouvez prendre la variable VSTS « $(imageUri) » et l’utiliser dans la tâche suivante ou simplement utiliser la valeur et générer une machine virtuelle.

## <a name="output-devops-variables"></a>Variables DevOps de sortie

Pub/offer/SKU/Version of the source marketplace image:
* $(pirPublisher)
* $(pirOffer)
* $(pirSku)
* $(pirVersion)

URI de l’image : le ResourceID de l’image distribuée :
* $(imageUri)

## <a name="faq"></a>Questions fréquentes (FAQ)

### <a name="can-i-use-an-existing-image-template-i-have-already-created-outside-of-devops"></a>Puis-je utiliser un modèle d’image existant que j’ai déjà créé, en dehors de DevOps ?

Pas pour l’instant.

### <a name="can-i-specify-the-image-template-name"></a>Puis-je spécifier le nom du modèle d’image ?

Non. Un nom de modèle unique est utilisé, puis supprimé.

### <a name="the-image-builder-failed-how-can-i-troubleshoot"></a>Échecs du générateur d’image. Comment puis-je résoudre ce problème ?

En cas d’échec d’une build, la tâche DevOps ne supprime pas le groupe de ressources de mise en lots. Vous pouvez accéder au groupe de ressources de mise en lots qui contient le journal de personnalisation de la build.

Vous verrez une erreur dans le journal DevOps pour la tâche Image Builder de machine virtuelle et vous verrez l’emplacement du fichier customization.log. Par exemple :

:::image type="content" source="./media/image-builder-devops-task/devops-task-error.png" alt-text="Exemple d’erreur de tâche DevOps qui indique une défaillance.":::

Pour plus d’informations sur la résolution des problèmes, consultez [Résoudre des problèmes liés au service Azure Image Builder](image-builder-troubleshoot.md). 

Après avoir examiné la défaillance, vous pouvez supprimer le groupe de ressources de mise en lots. Tout d’abord, supprimez l’artefact de ressource du modèle d’image. L’artefact a le préfix *t_* et se trouve dans le journal de génération des tâches DevOps :

```text
...
Source for image:  { type: 'SharedImageVersion',
  imageVersionId: '/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<imageDefName>/versions/<imgVersionNumber>' }
...
template name:  t_1556938436xxx
...

```

L’artefact de ressource du modèle d’image se trouve dans le groupe de ressources spécifié au début de la tâche. Une fois le résolution des problèmes terminée, supprimez l’artefact. Si vous supprimez à l’aide du Portail Azure, dans le groupe de ressources, sélectionnez **Afficher les types masqués** pour afficher l’artefact.


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Vue d’ensemble d’Azure Image Builder](image-builder-overview.md).