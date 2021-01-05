---
title: Utiliser le portail Azure pour résoudre des erreurs d’activation liées à Azure Stack Edge Pro avec GPU | Microsoft Docs
description: Décrit comment résoudre les problèmes liés à l’activation d’Azure Stack Edge Pro avec GPU et au coffre de clés.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: troubleshooting
ms.date: 10/08/2020
ms.author: alkohli
ms.openlocfilehash: e93a7fd7aec5463a3d77bd9d6bb17d7072097870
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96447629"
---
# <a name="troubleshoot-activation-issues-on-your-azure-stack-edge-pro-gpu-device"></a>Résoudre les erreurs d’activation survenant sur votre appareil Azure Stack Edge Pro avec GPU 

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

Cet article explique comment résoudre des erreurs d’activation survenant sur votre appareil Azure Stack Edge Pro avec GPU. 


## <a name="activation-errors"></a>Erreurs d’activation

Le tableau suivant résume les erreurs liées à l’activation d’appareil et la résolution recommandée correspondante.

| Message d’erreur| Résolution recommandée |
|------------------------------------------------------|--------------------------------------|
| Si le coffre Azure Key Vault utilisé pour l’activation est supprimé avant l’activation de l’appareil avec la clé d’activation, vous recevez cette erreur. <br> ![Erreur de coffre de clés 1](./media/azure-stack-edge-gpu-troubleshoot-activation/key-vault-error-1.png)  | Si le coffre de clés a été supprimé, vous pouvez le récupérer si le coffre est dans une période de protection contre la suppression définitive. Suivez les étapes décrites dans [Récupérer un coffre de clés](../key-vault/general/key-vault-recovery.md#list-recover-or-purge-soft-deleted-secrets-keys-and-certificates). <br>Si la période de protection contre la suppression définitive s’est écoulée, le coffre de clés ne peut pas être récupéré. Contactez le Support Microsoft pour les étapes suivantes. |
| Si le coffre Azure Key Vault est supprimé après l’activation de l’appareil et que vous essayez d’effectuer une opération qui implique un chiffrement, par exemple : **Ajouter un utilisateur**, **Ajouter un partage**, **Configurer le calcul**, vous recevez cette erreur. <br> ![Erreur de coffre de clés 2](./media/azure-stack-edge-gpu-troubleshoot-activation/key-vault-error-2.png)    | Si le coffre de clés a été supprimé, vous pouvez le récupérer si le coffre est dans une période de protection contre la suppression définitive. Suivez les étapes décrites dans Récupérer un coffre de clés. <br>Si la période de protection contre la suppression définitive s’est écoulée, le coffre de clés ne peut pas être récupéré. Contactez le Support Microsoft pour les étapes suivantes. |
| Si la clé d’intégrité du canal dans le coffre Azure Key Vault est supprimée et que vous essayez d’effectuer des opérations qui impliquent un chiffrement, par exemple : **Ajouter un utilisateur**, **Ajouter un partage**, **Configurer le calcul**, vous recevrez cette erreur. <br> ![Erreur de coffre de clés 3](./media/azure-stack-edge-gpu-troubleshoot-activation/key-vault-error-3.png) | Si la clé d’intégrité du canal dans le coffre de clés est supprimée, mais qu’il est toujours dans la période de suppression définitive, suivez les étapes décrites dans [Annuler la suppression de clé du coffre de clés](/powershell/module/az.keyvault/undo-azkeyvaultkeyremoval). <br>Si la période de protection contre la suppression définitive s’est écoulée et que vous avez sauvegardé la clé, vous pouvez effectuer une restauration à partir de la sauvegarde. Sinon, vous ne pouvez pas récupérer la clé. Contactez le Support Microsoft pour les étapes suivantes. |
| Si la génération de la clé d’activation échoue en raison d’une erreur, vous recevez cette erreur. Des détails supplémentaires sont fournis dans la notification. <br> ![Erreur de coffre de clés 4](./media/azure-stack-edge-gpu-troubleshoot-activation/key-vault-error-4.png)   | Patientez quelques minutes, puis recommencez l’opération. Si le problème persiste, contactez le support technique Microsoft. |
| Si l’utilisateur dispose d’autorisations en lecture seule, il n’est pas autorisé à générer une clé d’activation et cette erreur est affichée. <br> ![Erreur de coffre de clés 5](./media/azure-stack-edge-gpu-troubleshoot-activation/key-vault-error-5.png) | Cela peut être dû au fait que vous ne disposez pas du droit d’accès ou que *Microsoft.KeyVault* n’est pas inscrit.<li>Vérifiez que vous disposez d’un accès propriétaire ou contributeur au niveau du groupe de ressources utilisé pour votre ressource Azure Stack Edge.</li><li>Assurez-vous que le fournisseur de ressources Microsoft.KeyVault est inscrit. Pour inscrire un fournisseur de ressources, accédez à l’abonnement utilisé pour la ressource Azure Stack Edge. Accédez à **Fournisseurs de ressources**, recherchez *Microsoft.KeyVault*, sélectionnez et cliquez sur **Inscrire**.</li> |

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur comment [Résoudre les problèmes d’appareil](azure-stack-edge-gpu-troubleshoot.md).