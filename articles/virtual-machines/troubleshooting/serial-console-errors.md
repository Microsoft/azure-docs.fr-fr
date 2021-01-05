---
title: Erreurs de la console série Azure | Microsoft Docs
description: Erreurs courantes au sein de la console série Azure
services: virtual-machines
documentationcenter: ''
author: asinn826
manager: borisb
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm
ms.workload: infrastructure-services
ms.date: 8/20/2019
ms.author: alsin
ms.openlocfilehash: cad12a55332a6c7898f9709776c58d7dba8dd81a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022834"
---
# <a name="common-errors-within-the-azure-serial-console"></a>Erreurs courantes au sein de la console série Azure
Il existe un ensemble d’erreurs connues au sein de la console série Azure. Voici une liste de ces erreurs et des étapes d’atténuation pour celles-ci.

## <a name="common-errors"></a>Erreurs courantes

Error                             |   Limitation des risques
:---------------------------------|:--------------------------------------------|
« La console série Azure requiert l’activation des diagnostics de démarrage. Cliquez ici pour configurer les diagnostics de démarrage pour votre machine virtuelle. » | Vérifiez que les [diagnostics de démarrage](boot-diagnostics.md) sont activés sur la machine virtuelle ou le groupe de machines virtuelles identiques. Si vous utilisez la console série sur une instance de groupe de machines virtuelles identiques, assurez-vous que votre instance possède le modèle le plus récent.
« La console série Azure requiert l’exécution d’une machine virtuelle. Utilisez le bouton Démarrer ci-dessus pour démarrer votre machine virtuelle. »  | La machine virtuelle ou l’instance de groupe de machines virtuelles identiques doit être à l’état Démarré pour accéder à la console série (votre machine ne doit pas être arrêtée ou libérée). Assurez-vous que votre machine virtuelle ou votre instance de groupe de machines virtuelles identiques soit en cours d’exécution, puis réessayez.
« La console série Azure n’est pas activée pour cet abonnement, contactez l’administrateur de votre abonnement pour l’activer. » | La console série Azure peut être désactivée au niveau de l’abonnement. Si vous êtes un administrateur d’abonnement, vous pouvez [activer et désactiver la console série Azure](./serial-console-enable-disable.md). Si vous n’êtes pas administrateur d’abonnement, vous devez contacter votre administrateur d’abonnement pour connaître les étapes suivantes.
Une réponse « Interdit » s’est produite lors de l’accès au compte de stockage des diagnostics de démarrage de cette machine virtuelle. | Assurez-vous que les diagnostics de démarrage n’ont pas un pare-feu de compte. Un compte de stockage des diagnostics de démarrage accessible est nécessaire au fonctionnement de la console série. Serial console, de par sa conception, ne peut pas fonctionner avec des pare-feu de compte de stockage activés sur le compte de stockage des diagnostics de démarrage.
You do not have the required permissions to use this VM with the serial console. Ensure you have at least Virtual Machine Contributor role permissions. (Vous ne disposez pas des autorisations nécessaires pour utiliser la console série sur cette machine virtuelle. Vous devez disposer des autorisations du rôle Contributeur de machine virtuelle au minimum.)| L’accès à la console série nécessite que vous disposiez d’un accès de niveau contributeur ou d’une version supérieure sur votre machine virtuelle ou votre groupe de machines virtuelles identiques. Pour plus d’informations, consultez la [page de présentation](serial-console-overview.md).
Le compte de stockage '' utilisé pour les diagnostics de démarrage sur cette machine virtuelle est introuvable. Vérifiez que les diagnostics de démarrage sont activés pour cette machine virtuelle, que ce compte de stockage n’a pas été supprimé et que vous avez accès à ce compte de stockage. | Vérifiez que vous n’avez pas supprimé le compte de stockage des diagnostics de démarrage pour votre machine virtuelle ou votre groupe de machines virtuelles identiques
Une erreur s’est produite lors de la connexion de la console série à la machine virtuelle : « Requête incorrecte » (400) | Cette erreur peut se produire si votre URI des diagnostics de démarrage est incorrect. Par exemple, « http:// » a été utilisé à la place de « https:// ». L’URI des diagnostics de démarrage peut être corrigée à l’aide de cette commande : `az vm boot-diagnostics enable --name vmName --resource-group rgName --storage https://<storageAccountUri>.blob.core.windows.net/`
Vous ne disposez pas des autorisations nécessaires pour écrire dans le compte de stockage des diagnostics de démarrage pour cette machine virtuelle. Veillez à avoir au moins des autorisations de contributeur de machine virtuelle | L’accès à la console série nécessite un accès de niveau contributeur sur le compte de stockage des diagnostics de démarrage. Pour plus d’informations, consultez la [page de présentation](serial-console-overview.md).
Unable to determine the resource group for the boot diagnostics storage account *&lt;STORAGEACCOUNTNAME&gt;* . Verify that boot diagnostics is enabled for this VM and you have access to this storage account (Impossible de déterminer le groupe de ressources pour le compte de stockage de diagnostic de démarrage. Vérifiez que les diagnostics de démarrage sont activés sur la machine virtuelle et que vous avez accès au compte de stockage) | L’accès à la console série nécessite un accès de niveau contributeur sur le compte de stockage des diagnostics de démarrage. Pour plus d’informations, consultez la [page de présentation](serial-console-overview.md).
L’approvisionnement pour cette machine virtuelle n’a pas encore réussi. Vérifiez que la machine virtuelle est entièrement déployée et recommencez la connexion à la console série. | Il se peut que votre machine virtuelle ou votre groupe de machines virtuelles identiques soit toujours en cours d’approvisionnement. Patientez quelques instants, puis réessayez.
WebSocket est fermé ou n’a pas pu être ouvert. | Vous devrez peut-être ajouter l’accès du pare-feu à `*.console.azure.com`. Une approche plus détaillée mais plus longue consiste à autoriser l’accès du pare-feu aux [plages d’adresses IP du centre de Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=41653), qui changent assez régulièrement.
La console série ne fonctionne pas avec un compte de stockage utilisant Azure Data Lake Storage Gen2 avec des espaces de noms hiérarchiques. | Il s’agit d’un problème connu avec les espaces de noms hiérarchiques. Pour atténuer ce problème, vérifiez que le compte de stockage des diagnostics de démarrage de la machine virtuelle n’est pas créé avec Azure Data Lake Storage Gen2. Cette option peut être définie seulement lors de la création du compte de stockage. Vous devrez peut-être créer un compte de stockage des diagnostics de démarrage distinct sans Azure Data Lake Storage Gen2 pour atténuer ce problème.
Une erreur s’est produite lors de la connexion de la console série à la machine virtuelle : ’Forbidden’(SubscriptionNotEnabled) - Nom d’abonnement non défini, l’id \<subscription id> est dans un état non activé non défini | Ce problème peut se produire si l’abonnement dans lequel un utilisateur a créé son compte de stockage Cloud Shell a été désactivé. Pour atténuer le problème, lancez Cloud Shell et [effectuez les étapes nécessaires](../../cloud-shell/persisting-shell-storage.md#unmount-clouddrive-1) afin de réapprovisionner un compte de stockage de sauvegarde pour Cloud Shell dans l’abonnement actuel.

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur la [console série Azure pour les machines virtuelles Linux](./serial-console-linux.md)
* En savoir plus sur la [console série Azure pour les machines virtuelles Windows](./serial-console-windows.md)
