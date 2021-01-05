---
title: Mettre à niveau la version de Service Fabric d’un cluster
description: Mettez à niveau le code et/ou la configuration Service Fabric qui exécute un cluster Service Fabric. Cela inclut le mode de mise à jour du cluster, la mise à niveau de certificats, l’ajout de ports d’application, l’application des correctifs de système d’exploitation, etc. À quoi vous attendre lors de l'exécution des mises à niveau ?
ms.topic: conceptual
ms.date: 11/12/2018
ms.openlocfilehash: 01fe916f0ee78c8481ac6b17b8f7409b47c852ee
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90564285"
---
# <a name="upgrade-the-service-fabric-version-of-a-cluster"></a>Mettre à niveau la version de Service Fabric d’un cluster

Pour les systèmes modernes, la réussite à long terme d’un produit dépend de sa capacité à être mis à niveau. Un cluster Azure Service Fabric est une ressource que vous possédez mais qui est en partie gérée par Microsoft. Cet article explique comment mettre à niveau la version de Service Fabric en cours d’exécution dans votre cluster Azure.

Vous pouvez définir votre cluster de façon à recevoir les nouvelles mises à niveau de structure automatiques publiées par Microsoft, ou opter pour la sélection de la version de structure prise en charge que vous voulez associer à votre cluster.

Pour ce faire, définissez la configuration de cluster « upgradeMode » sur le portail, ou utilisez Resource Manager lors de la création ou ultérieurement, sur un cluster activé. 

> [!NOTE]
> Faites en sorte que votre cluster exécute systématiquement une version prise en charge de la structure. Lorsque nous annonçons le lancement d’une nouvelle version de Service Fabric, la fin de la prise en charge de la version précédente est signalée 60 jours minimum après la date de lancement. Les nouvelles versions sont annoncées [sur le blog de l’équipe Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric). La nouvelle version peut alors être sélectionnée. 
> 
> 

Un événement d’intégrité est généré 14 jours avant la date d’expiration de la version qu’exécute votre cluster. Il place le cluster dans un état d’avertissement. Ce dernier conserve cet état tant que vous n’avez pas effectué de mise à niveau vers une version prise en charge.

## <a name="set-the-upgrade-mode-in-the-azure-portal"></a>Définir le mode de mise à niveau dans le portail Azure
Vous pouvez définir le cluster sur le mode Manuel ou Automatique lors de la création du cluster.

![Capture d’écran montrant le volet Créer un cluster Service Fabric avec l’option 2 Configuration du cluster sélectionnée et le volet Configuration du cluster ouvert.][Create_Manualmode]

Vous pouvez définir le cluster sur le mode Automatique ou Manuel dans le cas d’un cluster activé, à l’aide de l’expérience de gestion. 

### <a name="upgrading-to-a-new-version-on-a-cluster-that-is-set-to-manual-mode-via-portal"></a>Mise à niveau vers une nouvelle version sur un cluster défini sur le mode Manuel, via le portail
Pour effectuer la mise à niveau vers une nouvelle version, il vous suffit de sélectionner la version disponible dans la liste déroulante, puis d’enregistrer. La mise à niveau de Service Fabric est lancée automatiquement. Les stratégies de contrôle d’intégrité du cluster (combinaison de l’intégrité des nœuds et de l’intégrité de toutes les applications en cours d’exécution dans le cluster) sont respectées pendant la mise à niveau.

Si les stratégies d'intégrité du cluster ne sont pas respectées, la mise à niveau est annulée. Faites dérouler ce document pour en savoir plus sur la définition de ces stratégies de contrôle d’intégrité personnalisées. 

Une fois que vous avez corrigé les problèmes entraînant la restauration, vous devez initier à nouveau la mise à niveau, en suivant la procédure décrite précédemment.

![Capture d’écran montrant la fenêtre Clusters Service Fabric avec le volet Mises à niveau de Fabric ouvert et les options de mise à niveau mises en surbrillance, dont Automatique et Manuel.][Manage_Automaticmode]

## <a name="set-the-upgrade-mode-using-a-resource-manager-template"></a>Définir le mode de mise à niveau à l’aide d’un modèle Resource Manager
Ajoutez la configuration « upgradeMode » dans la définition de ressource Microsoft.ServiceFabric/clusters et définissez le paramètre « clusterCodeVersion » sur l’une des versions de structure prises en charge, comme indiqué ci-dessous, puis déployez le modèle. Les valeurs valides pour l’élément « upgradeMode » sont « Manuel » ou « Automatique ».

![Capture d’écran montrant un modèle, qui est mis en retrait en texte clair pour refléter la structure et clusterCodeVersion et upgradeMode mis en surbrillance.][ARMUpgradeMode]

### <a name="upgrading-to-a-new-version-on-a-cluster-that-is-set-to-manual-mode-via-a-resource-manager-template"></a>Mise à niveau vers une nouvelle version sur un cluster défini sur le mode Manuel, via un modèle Resource Manager
Lorsque le cluster est en mode Manuel, si vous voulez effectuer une mise à niveau vers une nouvelle version, modifiez le paramètre « clusterCodeVersion » en indiquant une version prise en charge et en la déployant. Le déploiement du modèle lance la mise à niveau de la structure automatiquement. Les stratégies de contrôle d’intégrité du cluster (combinaison de l’intégrité des nœuds et de l’intégrité de toutes les applications en cours d’exécution dans le cluster) sont respectées pendant la mise à niveau.

Si les stratégies d'intégrité du cluster ne sont pas respectées, la mise à niveau est annulée.  

Une fois que vous avez corrigé les problèmes entraînant la restauration, vous devez initier à nouveau la mise à niveau, en suivant la procédure décrite précédemment.

## <a name="set-custom-health-polices-for-upgrades"></a>Définir des stratégies d’intégrité personnalisées pour les mises à niveau
Vous pouvez spécifier des stratégies de contrôle d’intégrité personnalisées pour la mise à niveau de la structure. Si vous avez défini votre cluster sur une mise à niveau automatique de la structure, ces stratégies sont appliquées lors de la [phase 1 de cette mise à niveau](service-fabric-cluster-upgrade.md#fabric-upgrade-behavior-during-automatic-upgrades).
Si vous avez défini votre cluster sur une mise à niveau manuelle de la structure, ces stratégies sont appliquées chaque fois que vous sélectionnez une nouvelle version, ce qui amène le système à déclencher la mise à niveau de la structure dans votre cluster. Si vous ne remplacez pas les stratégies, les valeurs par défaut sont utilisées.

Vous pouvez spécifier des stratégies de contrôle d’intégrité personnalisées ou passer en revue les paramètres actuels dans le panneau Fabric Upgrade (Mise à niveau de la structure), en sélectionnant les paramètres de mise à niveau avancés. L’image suivante vous explique comment procéder. 

![Gérer les stratégies de contrôle d’intégrité personnalisées][HealthPolices]

## <a name="list-all-available-versions-for-all-environments-for-a-given-subscription"></a>Lister toutes les versions disponibles de tous les environnements pour un abonnement donné
Exécutez la commande suivante. La sortie doit ressembler à ce qui suit.

Le paramètre « supportExpiryUtc » vous indique lorsqu’une version donnée arrive à expiration ou a expiré. La version la plus récente n’est pas associée à une date valide. Elle a donc la valeur « 9999-12-31T23:59:59.9999999 », qui signifie simplement que la date d’expiration n’est pas encore définie.

```REST
GET https://<endpoint>/subscriptions/{{subscriptionId}}/providers/Microsoft.ServiceFabric/locations/{{location}}/clusterVersions?api-version=2016-09-01

Example: https://management.azure.com/subscriptions/1857f442-3bce-4b96-ad95-627f76437a67/providers/Microsoft.ServiceFabric/locations/eastus/clusterVersions?api-version=2016-09-01

Output:
{
                  "value": [
                    {
                      "id": "subscriptions/35349203-a0b3-405e-8a23-9f1450984307/providers/Microsoft.ServiceFabric/environments/Windows/clusterVersions/5.0.1427.9490",
                      "name": "5.0.1427.9490",
                      "type": "Microsoft.ServiceFabric/environments/clusterVersions",
                      "properties": {
                        "codeVersion": "5.0.1427.9490",
                        "supportExpiryUtc": "2016-11-26T23:59:59.9999999",
                        "environment": "Windows"
                      }
                    },
                    {
                      "id": "subscriptions/35349203-a0b3-405e-8a23-9f1450984307/providers/Microsoft.ServiceFabric/environments/Windows/clusterVersions/4.0.1427.9490",
                      "name": "5.1.1427.9490",
                      "type": " Microsoft.ServiceFabric/environments/clusterVersions",
                      "properties": {
                        "codeVersion": "5.1.1427.9490",
                        "supportExpiryUtc": "9999-12-31T23:59:59.9999999",
                        "environment": "Windows"
                      }
                    },
                    {
                      "id": "subscriptions/35349203-a0b3-405e-8a23-9f1450984307/providers/Microsoft.ServiceFabric/environments/Windows/clusterVersions/4.4.1427.9490",
                      "name": "4.4.1427.9490",
                      "type": " Microsoft.ServiceFabric/environments/clusterVersions",
                      "properties": {
                        "codeVersion": "4.4.1427.9490",
                        "supportExpiryUtc": "9999-12-31T23:59:59.9999999",
                        "environment": "Linux"
                      }
                    }
                  ]
                }
```

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment personnaliser certains [paramètres de la structure du cluster Service Fabric](service-fabric-cluster-fabric-settings.md)
* Découvrez comment [mettre votre cluster à l’échelle](service-fabric-cluster-scale-in-out.md)
* Découvrez les [mises à niveau de l’application](service-fabric-application-upgrade.md)

<!--Image references-->
[CertificateUpgrade]: ./media/service-fabric-cluster-upgrade/CertificateUpgrade2.png
[AddingProbes]: ./media/service-fabric-cluster-upgrade/addingProbes2.PNG
[AddingLBRules]: ./media/service-fabric-cluster-upgrade/addingLBRules.png
[HealthPolices]: ./media/service-fabric-cluster-upgrade/Manage_AutomodeWadvSettings.PNG
[ARMUpgradeMode]: ./media/service-fabric-cluster-upgrade/ARMUpgradeMode.PNG
[Create_Manualmode]: ./media/service-fabric-cluster-upgrade/Create_Manualmode.PNG
[Manage_Automaticmode]: ./media/service-fabric-cluster-upgrade/Manage_Automaticmode.PNG
