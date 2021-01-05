---
title: Ajouter des métriques étendues pour les machines virtuelles Azure
description: Cet article vous aide à activer et à configurer les métriques de diagnostics étendues pour vos machines virtuelles Azure.
author: bandersmsft
ms.reviewer: vitavor
ms.author: banders
ms.date: 03/12/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.custom: seodec18
ROBOTS: NOINDEX
ms.openlocfilehash: 14ea98ecc4d9682353038088a124802d60a5dd5d
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92131443"
---
# <a name="add-extended-metrics-for-azure-virtual-machines"></a>Ajouter des métriques étendues pour les machines virtuelles Azure

Cloudyn utilise les données de métriques Azure issues de vos machines virtuelles Azure pour vous montrer des informations détaillées sur leurs ressources. Les données de métriques, également appelées compteurs de performances, permettent à Cloudyn de générer des rapports. Toutefois, Cloudyn ne collecte pas automatiquement toutes les données de métriques Azure à partir des machines virtuelles invitées : vous devez activer la collecte de métriques. Cet article vous aide à activer et à configurer des métriques de diagnostics supplémentaires pour vos machines virtuelles Azure.

Une fois que vous avez activé la collecte de métriques, vous pouvez :

- Savoir quand vos machines virtuelles atteignent leurs limites en termes de mémoire, disque et UC.
- Détecter les anomalies et les tendances d’utilisation.
- Contrôler les coûts en effectuant un redimensionnement en fonction de l’utilisation.
- Obtenir des recommandations d’optimisation du dimensionnement à moindre frais provenant de Cloudyn.

Par exemple, vous pouvez souhaiter surveiller les pourcentages d’utilisation de l’UC et de la mémoire de vos machines virtuelles Azure. Les métriques des machines virtuelles Azure correspondent à _Pourcentage d’UC_ et _\Mémoire\% octets validés en cours d’utilisation_ .

> [!NOTE]
> La collecte de données de métriques étendue est prise en charge uniquement avec la supervision d’invités Azure. Cloudyn n’est pas compatible avec [Log Analytics Agent](../../azure-monitor/platform/agents-overview.md).

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="determine-whether-extended-metrics-are-enabled"></a>Déterminer si les métriques étendues sont activées

1. Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).
2. Sous **Machines virtuelles** , sélectionnez une machine virtuelle, puis sous **Surveillance** , sélectionnez **Métriques** . Une liste de métriques disponibles s’affiche.
3. Sélectionnez certaines métriques ; un graphe affiche alors les données correspondantes.  
    ![Exemple de métrique : pourcentage d’UC (hôte)](./media/azure-vm-extended-metrics/metric01.png)

Dans l’exemple précédent, un ensemble limité de métriques standard est disponible pour vos hôtes, mais aucune métrique de mémoire n’est proposée. Les métriques de mémoire font partie des métriques étendues. Dans ce cas, les métriques étendues ne sont pas activées pour la machine virtuelle. Vous devez effectuer quelques étapes supplémentaires pour activer les métriques étendues. Les informations suivantes vous expliquent comment les activer.

## <a name="enable-extended-metrics-in-the-azure-portal"></a>Activer les métriques étendues dans le portail Azure

Les métriques standard sont des métriques d’hôte. La métrique _Pourcentage d’UC_ en est un exemple. Il existe également des métriques de base pour les machines virtuelles invitées ; elles sont également appelées « métriques étendues ». Voici quelques exemples de métriques étendues : _\Mémoire\% octets validés en cours d’utilisation_ et _\Mémoire\Octets disponibles_ .

L’activation des métriques étendues est simple. Pour chaque machine virtuelle, activez la surveillance d’invités. Quand vous activez la surveillance d’invités, l’agent de diagnostics Azure est installé sur la machine virtuelle. Par défaut, un ensemble de métriques étendues de base est ajouté. Le processus suivant est le même pour les machines virtuelles classiques et normales et le même pour les machines virtuelles Windows et Linux.

N’oubliez pas que la supervision d’invités Windows et Linux nécessite un compte de stockage. Quand vous activez la supervision d’invités, si vous ne choisissez pas un compte de stockage existant, un compte de stockage est automatiquement créé.

### <a name="enable-guest-level-monitoring-on-existing-vms"></a>Activer la surveillance d’invités sur les machines virtuelles existantes

1. Dans **Machines virtuelles** , affichez la liste de vos machines virtuelles, puis sélectionnez une machine virtuelle.
2. Sous **Supervision** , sélectionnez **Paramètres de diagnostic** .
3. Dans la page Paramètres de diagnostic, cliquez sur **Activer la surveillance d’invités** .  
    ![Activer la surveillance d’invités dans la page Vue d’ensemble](./media/azure-vm-extended-metrics/enable-guest-monitoring.png)
4. Après quelques minutes, l’agent de diagnostics Azure est installé sur la machine virtuelle. Un ensemble de métriques de base est ajouté. Actualisez la page. Les compteurs de performances ajoutés s’affichent sous l’onglet Vue d’ensemble.
5. Sous Supervision, sélectionnez **Métriques** .
6. Dans le graphique des métriques, sous **Espace de noms de la métrique** , sélectionnez **Invité (classique)** .
7. Dans la liste Métriques, vous pouvez afficher tous les compteurs de performances disponibles pour la machine virtuelle invitée.  
    ![liste d’exemples de métriques étendues](./media/azure-vm-extended-metrics/extended-metrics.png)

### <a name="enable-guest-level-monitoring-on-new-vms"></a>Activer la surveillance d’invités sur des machines virtuelles nouvelles

Quand vous créez des machines virtuelles, sous l’onglet Gestion, sélectionnez **Activé** pour **Diagnostic du système d’exploitation invité** .

![définition de Diagnostics du système d’exploitation invité sur Activé](./media/azure-vm-extended-metrics/new-enable-diag.png)

Pour plus d’informations sur l’activation des métriques étendues pour les machines virtuelles, consultez [Présentation et utilisation de l’agent Linux Azure](../../virtual-machines/extensions/agent-linux.md) et [Vue d’ensemble d’agent de machine virtuelle Azure](../../virtual-machines/extensions/agent-windows.md).

## <a name="resource-manager-credentials"></a>Informations d’identification Resource Manager

Après avoir activé les métriques étendues, vérifiez que Cloudyn a accès à vos [Informations d’identification Resource Manager](./activate-subs-accounts.md). Cloudyn a besoin de vos informations d’identification pour collecter et afficher les données de performances de vos machines virtuelles. Elles lui permettent également de créer des recommandations d’optimisation des coûts. Cloudyn a besoin d’au moins trois jours de données de performances d’une instance pour déterminer si une diminution des coûts peut être recommandée.

## <a name="enable-vm-metrics-with-a-script"></a>Activer les métriques de machine virtuelle avec un script

Vous pouvez activer les métriques de machine virtuelle avec des scripts Azure PowerShell. Si le nombre de machines virtuelles sur lesquelles vous souhaitez activer les métriques est élevé, vous pouvez utiliser un script pour automatiser le processus. Vous trouverez des exemples de scripts sur GitHub à la page [Azure Enable Diagnostics](https://github.com/Cloudyn/azure-enable-diagnostics) (Azure - Activer les diagnostics).

## <a name="view-azure-performance-metrics"></a>Afficher les métriques de performances Azure

Pour afficher les métriques de performances sur vos instances Azure dans le portail Cloudyn, accédez à **Ressources** > **Compute** > **Explorateur d’instances** . Dans la liste des instances de machine virtuelle, développez une instance, puis développez une ressource pour afficher les détails.

![exemples d’informations affichées dans l’Explorateur d’instances](./media/azure-vm-extended-metrics/instance-explorer.png)

## <a name="next-steps"></a>Étapes suivantes

- Si vous n’avez pas déjà activé l’accès de l’API Azure Resource Manager pour vos comptes, consultez [Activer des abonnements et comptes Azure](./activate-subs-accounts.md).