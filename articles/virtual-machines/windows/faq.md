---
title: FAQ sur les machines virtuelles Windows dans Azure
description: Fournit des réponses à certaines questions courantes sur les machines virtuelles Azure créées avec le modèle de déploiement Resource Manager.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 05/08/2019
ms.author: cynthn
ms.openlocfilehash: d5dda567d0cfa89863731a3e8e651aede4ab5957
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87825189"
---
# <a name="frequently-asked-question-about-windows-virtual-machines"></a>Questions fréquentes sur les machines virtuelles Azure
Cet article traite certaines questions courantes concernant les machines virtuelles Windows créées dans Azure avec le modèle de déploiement Resource Manager. Pour la version Linux de cette rubrique, consultez [Forum aux questions sur les machines virtuelles Linux](../linux/faq.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

## <a name="what-can-i-run-on-an-azure-vm"></a>Qu’est-il possible d’exécuter sur une machine virtuelle Azure ?
Tous les abonnés peuvent exécuter des logiciels serveur sur une machine virtuelle Azure. Pour plus d’informations sur la stratégie de prise en charge des logiciels serveur Microsoft en cours d’exécution dans Azure, consultez [Prise en charge des logiciels serveur Microsoft pour les machines virtuelles Windows Azure](https://support.microsoft.com/kb/2721672).

Certaines versions de Windows 7, Windows 8.1 et Windows 10 sont disponibles pour les abonnés MSDN Azure et les abonnés Développement et test MSDN avec paiement à l’utilisation (pour les tâches de test et de développement). Pour plus d’informations, notamment des instructions et des limitations, voir [Images de client Windows pour les abonnés MSDN](https://azure.microsoft.com/blog/2014/05/29/windows-client-images-on-azure/). 

## <a name="how-much-storage-can-i-use-with-a-virtual-machine"></a>Quelle quantité de stockage puis-je utiliser avec une machine virtuelle ?
Chaque disque de données peut avoir une capacité allant jusqu’à 32 767 Gio. Le nombre de disques de données que vous pouvez utiliser dépend de la taille de la machine virtuelle. Pour en savoir plus, voir la rubrique [Tailles de machines virtuelles](../sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

Les disques managés Azure sont les offres de stockage sur disque recommandées à utiliser avec les machines virtuelles Azure pour le stockage persistant de données. Vous pouvez utiliser plusieurs disques managés avec chaque machine virtuelle. Ces disques offrent deux types d’options de stockage durable : les disques managés Premium et les disques managés Standard. Pour obtenir des informations sur la tarification, consultez [Tarification de la fonctionnalité Disques managés](https://azure.microsoft.com/pricing/details/managed-disks).

Les comptes de stockage Azure peuvent aussi fournir le stockage pour le disque du système d’exploitation et tout disque de données. Chaque disque est un fichier .vhd stocké sous la forme d’un objet blob de pages. Pour plus d’informations sur la tarification, voir [Tarification – Stockage](https://azure.microsoft.com/pricing/details/storage/).

## <a name="how-can-i-access-my-virtual-machine"></a>Comment puis-je accéder à ma machine virtuelle ?
Établissez une connexion à distance à l’aide du protocole RDP (Remote Desktop Protocol) pour une machine virtuelle Windows. Pour plus d’informations, voir [Connexion à une machine virtuelle Azure exécutant Windows](connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Deux connexions simultanées maximum sont prises en charge, sauf si le serveur est configuré en tant qu’hôte de session Services Bureau à distance.  

En cas de problème avec le Bureau à distance, consultez [Résoudre les problèmes de connexion Bureau à distance à une machine virtuelle Azure Windows](../troubleshooting/troubleshoot-rdp-connection.md?toc=/azure/virtual-machines/windows/toc.json). 

Si vous connaissez bien Hyper-V, vous pouvez rechercher un outil semblable à VMConnect. Azure n’offre pas d’outil similaire car l’accès console à une machine virtuelle n’est pas pris en charge.

## <a name="can-i-use-the-temporary-disk-the-d-drive-by-default-to-store-data"></a>Puis-je utiliser le disque temporaire (lecteur D: par défaut) pour stocker des données ?
N’utilisez pas le disque temporaire pour stocker des données. Il ne sert qu’au stockage temporaire. Vous risqueriez donc de perdre des données sans pouvoir les récupérer. Une perte de données peut se produire si la machine virtuelle est déplacée vers un autre hôte. après le redimensionnement d’une machine virtuelle, la mise à jour de l’hôte ou une panne matérielle sur l’hôte, par exemple.

Si vous avez une application qui doit utiliser le lecteur D:, vous pouvez réaffecter les lettres de lecteur afin que le disque temporaire utilise une autre lettre que D. Pour obtenir des instructions, consultez la page [Modification de la lettre de lecteur du disque temporaire Windows](change-drive-letter.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json).


## <a name="how-can-i-change-the-drive-letter-of-the-temporary-disk"></a>Comment puis-je modifier la lettre de lecteur d’un disque temporaire ?
Vous pouvez changer la lettre de lecteur en déplaçant le fichier d’échange et en réaffectant les lettres de lecteur. Toutefois, vous devrez veiller à effectuer les étapes dans un ordre spécifique. Pour obtenir des instructions, consultez la page [Modification de la lettre de lecteur du disque temporaire Windows](change-drive-letter.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json).

## <a name="can-i-add-an-existing-vm-to-an-availability-set"></a>Puis-je ajouter une machine virtuelle à un groupe à haute disponibilité ?
Non. Si vous souhaitez que votre machine virtuelle fasse partie d’un groupe à haute disponibilité, vous devez la créer dans le groupe. Il n’existe actuellement aucun moyen d’ajouter une machine virtuelle dans un groupe à haute disponibilité, une fois celle-ci créée.

## <a name="can-i-upload-a-virtual-machine-to-azure"></a>Puis-je télécharger une machine virtuelle dans Azure ?
Oui. Pour obtenir des instructions, consultez [Migration de machines virtuelles sur site vers Azure](on-prem-to-azure.md).

## <a name="can-i-resize-the-os-disk"></a>Puis-je redimensionner le disque du système d’exploitation ?
Oui. Pour obtenir des instructions, consultez [Extension du lecteur de système d’exploitation d’une machine virtuelle dans un groupe de ressources Azure](expand-os-disk.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="can-i-copy-or-clone-an-existing-azure-vm"></a>Puis-je copier ou cloner une machine virtuelle Azure ?
Oui. À l’aide d’images gérées, vous pouvez créer une image d’une machine virtuelle, puis utiliser l’image pour générer plusieurs nouvelles machines virtuelles. Pour obtenir des instructions, consultez [Créer une image personnalisée d’une machine virtuelle](tutorial-custom-images.md).

## <a name="why-am-i-not-seeing-canada-central-and-canada-east-regions-through-azure-resource-manager"></a>Pourquoi ne vois-je pas les régions Centre et Est du Canada dans Azure Resource Manager ?

Les deux régions Centre et Est du Canada ne sont pas enregistrées automatiquement lors de la création de machines virtuelles pour des abonnements Azure existants. Cet enregistrement s’effectue automatiquement lorsqu’une machine virtuelle est déployée par le biais du portail Azure dans n’importe quelle autre région à l’aide d’Azure Resource Manager. Une fois une machine virtuelle déployée dans toute autre région Azure, les nouvelles régions doivent être disponibles pour les machines virtuelles suivantes.

## <a name="does-azure-support-linux-vms"></a>Azure prend-il en charge les machines virtuelles Linux ?
Oui. Pour créer rapidement une machine virtuelle Linux de test, consultez [Création d’une machine virtuelle Linux sur Azure à l’aide du portail](../linux/quick-create-portal.md).

## <a name="can-i-add-a-nic-to-my-vm-after-its-created"></a>Puis-je ajouter une carte réseau à ma machine virtuelle après sa création ?
Oui, c’est maintenant possible. La machine virtuelle doit d’abord être arrêtée et libérée. Ensuite, vous pouvez ajouter ou supprimer une carte réseau (sauf si elle est la dernière carte réseau sur la machine virtuelle). 

## <a name="are-there-any-computer-name-requirements"></a>Existe-t-il des exigences en matière de nom d’ordinateur ?
Oui. Le nom d’ordinateur peut avoir une longueur maximale de 15 caractères. Consultez l’article [Naming conventions rules and restrictions](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging) (Restrictions et règles de conventions d’affectation de noms) pour en savoir plus sur la dénomination de vos ressources.

## <a name="are-there-any-resource-group-name-requirements"></a>Existe-t-il des exigences pour le nom du groupe de ressources ?
Oui. Le nom du groupe de ressources peut avoir une longueur maximale de 90 caractères. Consultez l’article [Naming conventions rules and restrictions](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging#resource-naming) (Restrictions et règles de conventions d’affectation de noms) pour en savoir plus sur les groupes de ressources.

## <a name="what-are-the-username-requirements-when-creating-a-vm"></a>Quelles sont les exigences en matière de nom d’utilisateur lors de la création d’une machine virtuelle ?

Les noms d’utilisateur peuvent comporter un maximum de 20 caractères et ne doivent pas se terminer par un point («. »). 

Les noms d’utilisateur suivants ne sont pas autorisés :

- `1`
- `123`
- `a`
- `actuser`
- `adm`
- `admin`
- `admin1`
- `admin2`
- `administrator`
- `aspnet`
- `backup`
- `console`
- `david`
- `guest`
- `john`
- `owner`
- `root`
- `server`
- `sql`
- `support_388945a0`
- `support`
- `sys`
- `test`
- `test1`
- `test2`
- `test3`
- `user`
- `user1`
- `user2`

## <a name="what-are-the-password-requirements-when-creating-a-vm"></a>Quelles sont les exigences en matière de mot de passe lors de la création d’une machine virtuelle ?

Il existe différentes exigences en termes de longueur du mot de passe, selon l’outil que vous utilisez :
 - Portail : entre 12 et 72 caractères
 - PowerShell : entre 8 et 123 caractères
 - Interface CLI : entre 12 et 123 caractères

* Avoir des minuscules
* Avoir des majuscules
* Avoir un chiffre
* Avoir un caractère spécial (correspondances Regex [\W_])

Les noms mots de passe suivants ne sont pas autorisés :

<table>
    <tr>
        <td>abc@123</td>
        <td>iloveyou!</td>
        <td>P@$$w0rd</td>
        <td>P@ssw0rd</td>
        <td>P@ssword123</td>
    </tr>
    <tr>
        <td>Pa$$word</td>
        <td>pass@word1</td>
        <td>Password!</td>
        <td>Password1</td>
        <td>Password22</td>
    </tr>
</table>