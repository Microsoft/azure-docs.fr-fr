---
title: Centre de sauvegarde - FAQ
description: Cet article répond aux questions fréquentes à propos du Centre de sauvegarde
ms.topic: conceptual
ms.date: 09/08/2020
ms.openlocfilehash: c5b23a32b60f651cd3ff91819155d83a7465491a
ms.sourcegitcommit: 2989396c328c70832dcadc8f435270522c113229
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92173827"
---
# <a name="backup-center---frequently-asked-questions"></a>Centre de sauvegarde - Forum Aux Questions

## <a name="management"></a>Gestion

### <a name="can-backup-center-be-used-across-tenants"></a>Le Centre de sauvegarde peut-il être utilisé sur plusieurs locataires ?

Oui, si vous utilisez [Azure Lighthouse](../lighthouse/overview.md) et que vous disposez d’un accès délégué à des abonnements sur différents locataires, vous pouvez utiliser le Centre de sauvegarde comme un volet unique pour gérer les sauvegardes entre les locataires.

### <a name="can-backup-center-be-used-to-manage-both-recovery-services-vaults-and-backup-vaults"></a>Le Centre de sauvegarde peut-il être utilisé pour gérer des coffres Recovery Services et des coffres de sauvegarde ?

Oui, le Centre de sauvegarde peut gérer les [coffres Recovery Services](./backup-azure-recovery-services-vault-overview.md) et les [coffres de sauvegarde](backup-vault-overview.md).

### <a name="is-there-a-delay-before-data-surfaces-in-backup-center"></a>Existe-t-il un délai avant que les données apparaissent dans le Centre de sauvegarde ?

Le Centre de sauvegarde est destiné à fournir des informations en temps réel. Il peut y avoir quelques secondes de décalage entre le moment où une entité s’affiche dans un écran de coffre individuel et le moment où la même entité s’affiche dans le Centre de sauvegarde.

## <a name="configuration"></a>Configuration

### <a name="do-i-need-to-configure-anything-to-see-data-in-backup-center"></a>Dois-je configurer quoi que ce soit pour afficher des données dans le Centre de sauvegarde ?

Non. Le Centre de sauvegarde est prêt à l’emploi. Toutefois, pour afficher les [rapports de sauvegarde](./configure-reports.md) dans le Centre de sauvegarde, vous devez configurer la création de rapports pour vos coffres.

### <a name="do-i-need-to-have-any-special-permissions-to-use-backup-center"></a>Ai-je besoin d’autorisations spéciales pour utiliser le Centre de sauvegarde ?

Le Centre de sauvegarde en tant que tel n’a pas besoin de nouvelles autorisations. Tant que vous disposez du niveau d’accès RBAC Azure approprié pour les ressources que vous gérez, vous pouvez utiliser le centre de sauvegarde pour ces ressources. Par exemple, pour afficher des informations sur vos sauvegardes, vous avez besoin de l’accès **Lecteur** à vos coffres. Pour configurer la sauvegarde et effectuer d’autres actions liées à la sauvegarde, vous devez avoir le rôle **Collaborateur de sauvegarde** ou **Opérateur de sauvegarde** . En savoir plus sur les rôles [Azure pour la sauvegarde Azure](./backup-rbac-rs-vault.md).

## <a name="pricing"></a>Tarifs

### <a name="do-i-need-to-pay-anything-extra-to-use-backup-explorer"></a>Dois-je payer un supplément pour utiliser le Centre de sauvegarde ?

Actuellement, il n’y a aucun coût supplémentaire (en dehors de vos coûts de sauvegarde) pour utiliser le Centre de sauvegarde. Toutefois, si vous utilisez des [rapports de sauvegarde](./configure-reports.md) dans le Centre de sauvegarde, il y a un [coût impliqué](https://azure.microsoft.com/pricing/details/monitor/) dans l’utilisation des journaux Azure Monitor pour les rapports de sauvegarde.

## <a name="next-steps"></a>Étapes suivantes

Lire les autres forums aux questions :

* [Questions courantes sur les coffres Recovery Services](./backup-azure-backup-faq.md)
* [Questions courantes sur la sauvegarde des machines virtuelles Azure](./backup-azure-vm-backup-faq.md)