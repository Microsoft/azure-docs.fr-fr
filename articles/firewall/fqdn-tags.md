---
title: Présentation des balises FQDN pour le pare-feu Azure
description: Une balise FQDN représente un groupe de noms de domaine complets (FQDN) associés à des services Microsoft bien connus.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: article
ms.date: 06/30/2020
ms.author: victorh
ms.openlocfilehash: c0bd2f6a021baae80dafcc3d544e1062d8e022e8
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94653332"
---
# <a name="fqdn-tags-overview"></a>Présentation des balises FQDN

Une balise FQDN représente un groupe de noms de domaine complets (FQDN) associés à des services Microsoft bien connus. Vous pouvez utiliser une balise FQDN dans les règles d’application pour autoriser le trafic sortant requis via votre pare-feu.

Par exemple, pour autoriser manuellement le trafic Windows Update via votre pare-feu, vous devez créer plusieurs règles d’application conformément à la documentation Microsoft. À l’aide des balises FQDN, vous pouvez créer une règle d’application et inclure la balise **Windows Update**. Le trafic vers les points de terminaison Microsoft Windows Update peut désormais transiter via votre pare-feu.

Vous ne pouvez pas créer vos propres balises FQDN, ni spécifier quels noms de domaine complets sont inclus dans une balise. Microsoft gère les noms de domaine complets inclus dans la balise FQDN et met à jour la balise en fonction de l’évolution des noms de domaine complets. 

<!--- screenshot of application rule with a FQDN tag.-->

Le tableau suivant présente les balises FQDN que vous pouvez actuellement utiliser. Microsoft gère ces balises, et vous pouvez vous attendre à ce que de nouvelles balises soient ajoutées régulièrement.

## <a name="current-fqdn-tags"></a>Balises FQDN actuelles

|Balise FQDN  |Description  |
|---------|---------|
|Windows Update     |Autoriser un accès sortant à Microsoft Update, comme décrit dans [Comment configurer un pare-feu pour les mises à jour logicielles](/mem/configmgr/sum/get-started/install-a-software-update-point).|
|Diagnostics Windows|Autoriser l’accès sortant à tous les [points de terminaison Windows Diagnostics](/windows/privacy/configure-windows-diagnostic-data-in-your-organization#endpoints).|
|Microsoft Active Protection Service (MAPS)|Autoriser l’accès sortant à [MAPS](https://cloudblogs.microsoft.com/enterprisemobility/2016/05/31/important-changes-to-microsoft-active-protection-service-maps-endpoint/).|
|App Service Environment (ASE)|Autoriser l’accès sortant au trafic de plateforme ASE. Cette balise ne concerne pas les points de terminaison SQL et de stockage propres au client créés par ASE. Ceux-ci doivent être activés via les [points de terminaison de service](../virtual-network/tutorial-restrict-network-access-to-resources.md) ou ajoutés manuellement.<br><br>Pour plus d’informations sur l’intégration du Pare-feu Azure avec l’ASE, consultez [Verrouillage d’un App Service Environment](../app-service/environment/firewall-integration.md#configuring-azure-firewall-with-your-ase).|
|Sauvegarde Azure|Autoriser l’accès sortant aux services de sauvegarde Azure.|
|Azure HDInsight|Autoriser l’accès sortant au trafic de plateforme HDInsight. Cette balise ne concerne pas le trafic SQL et de stockage propre au client provenant de HDInsight. Activer à l’aide de [points de terminaison de service](../virtual-network/tutorial-restrict-network-access-to-resources.md) ou ajouter manuellement.|
|WindowsVirtualDesktop (WVD)|Autorise le trafic sortant de la plateforme Windows Virtual Desktop. Cette étiquette ne concerne pas les points de terminaison Service Bus et de stockage propres au déploiement créés par WVD. En outre, des règles de réseau DNS et KMS sont requises. Pour plus d’informations sur l’intégration du Pare-feu Azure à WVD, consultez [Utiliser le Pare-feu Azure pour protéger les déploiements de Windows Virtual Desktop](protect-windows-virtual-desktop.md).|
|Azure Kubernetes Service (AKS)|Autorise l’accès sortant à AKS. Pour plus d’informations, consultez [Utiliser le Pare-feu Azure pour protéger des déploiements d’Azure Kubernetes Service (AKS)](protect-azure-kubernetes-service.md).|

> [!NOTE]
> Lorsque vous sélectionnez une balise FQDN dans une règle d’application, le champ protocol:port doit être défini sur **https**.

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment déployer un pare-feu Azure, consultez le [didacticiel : Déployer et configurer un pare-feu Azure à l’aide du portail Azure](tutorial-firewall-deploy-portal.md)