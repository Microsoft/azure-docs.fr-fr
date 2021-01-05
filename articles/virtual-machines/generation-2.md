---
title: Support Azure pour les machines virtuelles de 2e génération
description: Vue d’ensemble de la prise en charge Azure pour les machines virtuelles de 2e génération
author: ju-shim
ms.service: virtual-machines
ms.subservice: sizes
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 08/28/2020
ms.author: jushiman
ms.openlocfilehash: 56a360f461c89d7c60e973bf2d6258d63113ab47
ms.sourcegitcommit: 6b16e7cc62b29968ad9f3a58f1ea5f0baa568f02
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/28/2020
ms.locfileid: "96303783"
---
# <a name="support-for-generation-2-vms-on-azure"></a>Support des machines virtuelles de 2e génération dans Azure

Le support des machines virtuelles de 2e génération est désormais disponible dans Azure. Vous ne pouvez pas modifier la génération d’une machine virtuelle après l’avoir créée. Passez bien en revue les considérations relatives sur cette page avant de choisir une génération.

Les machines virtuelles de 2e génération prennent en charge des fonctionnalités clés qui ne sont pas prises en charge par des machines virtuelles de 1ère génération. Ces fonctionnalités incluent la mémoire augmentée, Intel Software Guard Extensions (Intel SGX) et mémoire persistante virtualisée (vPMEM). Les machines virtuelles locales de 2e génération présentent des fonctionnalités qui ne sont pas encore prises en charge dans Azure. Pour en savoir, consultez la section [Fonctionnalités et capacités](#features-and-capabilities).

Les machines virtuelles de 2e génération utilisent la nouvelle architecture de démarrage basée sur UEFI alors que les machines virtuelles de 1ère génération utilisaient l’architecture basée sur le BIOS. Comparées aux machines virtuelles de 1ère génération, les machines virtuelles de 2e génération peuvent avoir des temps d’installation et de démarrage améliorés. Pour obtenir une vue d’ensemble des machines virtuelles de 2e génération et de quelques-unes des différences entre 1ère et 2e génération, consultez [Dois-je créer une machine virtuelle de génération 1 ou 2 dans Hyper-V ?](/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v).

## <a name="generation-2-vm-sizes"></a>Tailles des machines virtuelles de 2e génération

Les machines virtuelles de 1ère génération sont prises en charge dans toutes les tailles de machines virtuelles dans Azure (sauf les machines virtuelles de série Mv2). Azure prend désormais en charge la 2e génération de machines virtuelles des séries sélectionnées suivantes :


* [Série B](sizes-b-series-burstable.md)
* [Série DCsv2](dcv2-series.md)
* [Séries DSv2](dv2-dsv2-series.md)
* [Dsv3-series](dv3-dsv3-series.md)
* [Série Dsv4](dv4-dsv4-series.md)
* [Dasv4-series](dav4-dasv4-series.md)
* [Série Ddsv4](ddv4-ddsv4-series.md)
* [Série Esv3](ev3-esv3-series.md)
* [Easv4-series](eav4-easv4-series.md)
* [Série Fsv2](fsv2-series.md)
* [Série GS](sizes-previous-gen.md#gs-series)
* [Série HB](hb-series.md)
* [Série HC](hc-series.md)
* [Série Ls](sizes-previous-gen.md#ls-series) 
* [Série Lsv2](lsv2-series.md)
* [Série M](m-series.md)
* [Série Mv2](mv2-series.md)<sup>1</sup>
* [Série NCv2](ncv2-series.md) 
* [Série NCv3](ncv3-series.md)
* [Série ND](nd-series.md)
* [Série NVv3](nvv3-series.md)
* [Série NVv4](nvv4-series.md)
* [NCasT4_v3-series](nct4-v3-series.md)

<sup>1</sup> La série MV2 ne prend pas en charge les images de machine virtuelle de 1re génération et ne prend en charge qu’un sous-ensemble d’images de 2e génération. Pour plus d’informations, consultez la [documentation de la série MV2](mv2-series.md).


## <a name="generation-2-vm-images-in-azure-marketplace"></a>Images de machine virtuelle de 2e génération dans la Place de marché Azure

Les machines virtuelles de 2e génération prennent en charge les images de la Place de marché suivantes :

* Windows Server 2019, 2016, 2012 R2, 2012
* Windows 10 Professionnel, Windows 10 Entreprise
* SUSE Linux Enterprise Server 15 SP1
* SUSE Linux Enterprise Server 12 SP4
* Ubuntu Server 16.04, 18.04, 19.04, 19.10 
* RHEL 8.1, 8.0, 7.7, 7.6, 7.5, 7.4, 7.0
* Cent OS 8.1, 8.0, 7.7, 7.6, 7.5, 7.4
* Oracle Linux 7.7, 7.7-CI

> [!NOTE]
> Les tailles de machine virtuelle spécifiques telles que la série MV2 ne peuvent prendre en charge qu’un sous-ensemble de ces images. Pour plus d’informations, consultez la documentation appropriée sur la taille de machine virtuelle.

## <a name="on-premises-vs-azure-generation-2-vms"></a>Local Machines virtuelles de 2e génération Azure

Azure ne prend actuellement pas en charge certaines fonctionnalités qui sont prises en charge par Hyper-V local pour les machines virtuelles de 2e génération.

| Fonctionnalités pour la 2e génération                | Hyper-V local | Azure |
|-------------------------------------|---------------------|-------|
| Démarrage sécurisé                         | :heavy_check_mark:  | :x:   |
| Machine virtuelle protégée                         | :heavy_check_mark:  | :x:   |
| vTPM                                | :heavy_check_mark:  | :x:   |
| Sécurité basée sur la virtualisation (VBS) | :heavy_check_mark:  | :x:   |
| Format VHDX                         | :heavy_check_mark:  | :x:   |

## <a name="features-and-capabilities"></a>Fonctionnalités et capacités

### <a name="generation-1-vs-generation-2-features"></a>Fonctionnalités de 1ère et 2e générations

| Fonctionnalité | Génération 1 | Génération 2 |
|---------|--------------|--------------|
| Démarrage             | PCAT                      | UEFI                               |
| Contrôleurs de disque | IDE                       | SCSI                               |
| Tailles de machine virtuelle         | Toutes les tailles de machine virtuelle | [Voir les tailles disponibles](#generation-2-vm-sizes) |

### <a name="generation-1-vs-generation-2-capabilities"></a>Capacités de 1ère et 2e générations

| Fonctionnalité | Génération 1 | Génération 2 |
|------------|--------------|--------------|
| Disque OS > 2 To                    | :x:                | :heavy_check_mark: |
| Disque/image/système d’exploitation d’échange personnalisé         | :heavy_check_mark: | :heavy_check_mark: |
| Prise en charge de groupes de machines virtuelles identiques | :heavy_check_mark: | :heavy_check_mark: |
| Azure Site Recovery               | :heavy_check_mark: | :heavy_check_mark: |
| Sauvegarde/restauration                    | :heavy_check_mark: | :heavy_check_mark: |
| Galerie d’images partagées              | :heavy_check_mark: | :heavy_check_mark: |
| [Azure Disk Encryption](../security/fundamentals/azure-disk-encryption-vms-vmss.md)             | :heavy_check_mark: | :x:                |
| [Chiffrement côté serveur](disk-encryption.md)            | :heavy_check_mark: | :heavy_check_mark: |

## <a name="creating-a-generation-2-vm"></a>Création d’une machine virtuelle de 2e génération

### <a name="marketplace-image"></a>Image de la Place de marché

Dans le portail Azure ou Azure CLI, vous pouvez créer des machines virtuelles de 2e génération à partir d’une image de la Place de marché qui prend en charge le démarrage UEFI.

#### <a name="azure-portal"></a>Portail Azure

Vous trouverez ci-dessous les étapes de création d’une machine virtuelle de génération 2 (Gen2) dans le portail Azure.

1. Connectez-vous au portail Azure sur https://portal.azure.com.
1. Sélectionnez **Créer une ressource**.
1. Sélectionnez **Afficher tout** dans la Place de Marché Azure à gauche.
1. Sélectionnez une image qui prend en charge Gen2.
1. Cliquez sur **Créer**.
1. Dans l’onglet **Avancé**, sous la section **Génération de machine virtuelle**, sélectionnez l’option **Gen 2**.
1. Dans l’onglet **De base**, sous **Détails de l’instance**, accédez à **Taille** et ouvrez le panneau **Sélectionner une taille de machine virtuelle**.
1. Sélectionnez une [machine virtuelle de génération 2 prise en charge](#generation-2-vm-sizes).
1. Parcourez le reste des pages pour terminer la création de la machine virtuelle.

![Sélectionner une machine virtuelle de génération 1 ou de génération 2](./media/generation-2/gen1-gen2-select.png)

#### <a name="powershell"></a>PowerShell

Vous pouvez également utiliser PowerShell pour créer une machine virtuelle en référençant directement le SKU de génération 1 ou de génération 2.

Par exemple, utilisez l’applet de commande PowerShell suivante pour obtenir une liste des SKU dans l’offre `WindowsServer`.

```powershell
Get-AzVMImageSku -Location westus2 -PublisherName MicrosoftWindowsServer -Offer WindowsServer
```

Si vous créez une machine virtuelle avec le système d’exploitation Windows Server 2012, vous devez sélectionner le SKU de machine virtuelle génération 1 (BIOS) ou génération 2 (UEFI), comme suit :

```powershell
2012-Datacenter
2012-datacenter-gensecond
```

Consultez la section [ Fonctionnalités et capacités](#features-and-capabilities) pour obtenir une liste actuelle des images de la Place de marché prises en charge.

#### <a name="azure-cli"></a>Azure CLI

Vous pouvez également utiliser Azure CLI pour voir les images de génération 2 disponibles, listées par **Éditeur**.

```azurecli
az vm image list --publisher Canonical --sku gen2 --output table --all
```

### <a name="managed-image-or-managed-disk"></a>Image ou disque managé

Vous pouvez créer une machine virtuelle de 2e génération à partir d’une image managée ou d’un disque managé, de la même façon que pour une machine virtuelle de 1ère génération.

### <a name="virtual-machine-scale-sets"></a>Groupes identiques de machines virtuelles

Vous pouvez également créer des machines virtuelles de 2e génération à l’aide de groupes de machines virtuelles identiques. Dans Azure CLI, utilisez les groupes identiques Azure pour créer des machines virtuelles de 2e génération.

## <a name="frequently-asked-questions"></a>Forum aux questions

* **Les machines virtuelles de 2e génération sont-elles disponibles dans toutes les régions Azure ?**  
    Oui. Par contre, toutes les [tailles de machines virtuelles de 2e génération](#generation-2-vm-sizes) ne sont pas disponibles dans chaque région. La disponibilité des machines virtuelles de 2e génération dépend des disponibilités en matière de taille.

* **Y a-t-il une différence de prix entre des machines virtuelles de 1ère et 2e générations ?**  
   Non.

* **J’ai un fichier .vhd de ma machine virtuelle de 2e génération locale. Puis-je utiliser ce fichier .vhd pour créer une machine virtuelle de 2e génération dans Azure ?**
  Oui, vous pouvez placer votre fichier .vhd de 2e génération sur Azure et l’utiliser pour créer une machine virtuelle de 2e génération. Pour ce faire, procédez comme suit :
    1. Chargez le fichier .vhd sur un compte de stockage situé dans la région où vous souhaitez créer votre machine virtuelle.
    1. Créez un disque managé à partir du fichier .vhd. Affectez à la propriété Génération Hyper-V la valeur V2. Les commandes PowerShell suivantes définissent la propriété Génération Hyper-V au moment de la création d’un disque managé.

        ```powershell
        $sourceUri = 'https://xyzstorage.blob.core.windows.net/vhd/abcd.vhd'. #<Provide location to your uploaded .vhd file>
        $osDiskName = 'gen2Diskfrmgenvhd'  #<Provide a name for your disk>
        $diskconfig = New-AzDiskConfig -Location '<location>' -DiskSizeGB 127 -AccountType Standard_LRS -OsType Windows -HyperVGeneration "V2" -SourceUri $sourceUri -CreateOption 'Import'
        New-AzDisk -DiskName $osDiskName -ResourceGroupName '<Your Resource Group>' -Disk $diskconfig
        ```

    1. Une fois le disque disponible, créez une machine virtuelle en y attachant ce disque. La machine virtuelle créée est alors une machine virtuelle de 2e génération.
    Une fois la machine virtuelle de 2e génération créée, vous pouvez généraliser l’image de cette machine virtuelle. En généralisant l’image, vous pouvez l’utiliser pour créer plusieurs machines virtuelles.

* **Comment augmenter la taille du disque du système d’exploitation ?**  

  Les disques d’une taille supérieure à 2 Tio sont nouveaux pour les machines virtuelles de 2e génération. Par défaut, les disques de système d’exploitation sont d’une taille inférieure à 2 Tio pour les machines virtuelles de 2e génération. Vous pouvez augmenter la taille de disque jusqu’à la taille maximale recommandée de 4 Tio. Utilisez Azure CLI ou le portail Azure pour augmenter la taille du disque. Pour en savoir plus sur la manière d’étendre des disques par programme, consultez **Redimensionner un disque** pour [Windows](./windows/expand-os-disk.md) ou [Linux](./linux/resize-os-disk-gpt-partition.md).

  Pour augmenter la taille du disque du système d’exploitation à partir du portail Azure :

  1. Accédez à la page des propriétés de la machine virtuelle dans le portail Azure.
  1. Pour arrêter et libérer la machine virtuelle, sélectionnez le bouton **Arrêter**.
  1. Dans la section **Disques**, sélectionnez le disque du système d’exploitation que vous souhaitez augmenter.
  1. Dans la section **Disques**, sélectionnez **Configuration**, et mettez à jour la **Taille** à la valeur souhaitée.
  1. Revenez à la page des propriétés de la machine virtuelle et **démarrez**-la.
  
  Vous verrez peut-être s’afficher un avertissement pour les disques de système d’exploitation d’une taille supérieure à 2 Tio. L’avertissement ne s’applique pas aux machines virtuelles de 2e génération. Toutefois, les tailles de disque de système d’exploitation supérieures à 4 Tio ne sont pas prises en charge.

* **Les machines virtuelles de 2e génération prennent-elles en charge la mise en réseau accélérée ?**  
    Oui. Pour en savoir plus, consultez [Créer une machine virtuelle avec mise en réseau accélérée](../virtual-network/create-vm-accelerated-networking-cli.md).

* **Les machines virtuelles de 2e génération prennent-elles en charge le démarrage sécurisé ou vTPM dans Azure ?**
    Les machines virtuelles de 1ère et 2e générations dans Azure ne prennent pas en charge le démarrage sécurisé ou vTPM. 
    
* **La 2e génération prend-elle en charge VHDX ?**  
    Non, les machines virtuelles de 2e génération prennent en charge uniquement un disque dur virtuel.

* **Les machines virtuelles de 2e génération prennent-elles en charge les disques de stockage Ultra Azure ?**  
    Oui.

* **Puis-je migrer une machine virtuelle à partir de la 1ère génération à la 2e génération ?**  
    Non, vous ne pouvez pas modifier la génération d’une machine virtuelle après sa création. Si vous avez besoin de passer d’une génération à une autre, créez une machine virtuelle de la génération souhaitée.

* **Pourquoi la taille de la machine virtuelle n’est-elle pas activée dans le sélecteur de taille quand j’essaie de créer une machine virtuelle Gen2 ?**

    Vous pouvez résoudre ce problème en effectuant les étapes suivantes :

    1. Vérifiez que la propriété **Génération de machine virtuelle** est définie sur **Gen 2** dans l’onglet **Avancé**.
    1. Vérifiez que vous recherchez une [taille de machine virtuelle qui prend en charge les machines virtuelles Gen2](#generation-2-vm-sizes).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [machines virtuelles de 2e génération dans Hyper-V](/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v).
