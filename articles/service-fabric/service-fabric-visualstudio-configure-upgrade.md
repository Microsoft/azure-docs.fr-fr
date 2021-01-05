---
title: Configuration de la mise à niveau d’une application Service Fabric
description: Apprenez à configurer les paramètres de mise à niveau d'une application Service Fabric à l'aide de Microsoft Visual Studio.
ms.topic: conceptual
ms.date: 06/29/2017
ms.openlocfilehash: 11df0387db07669ebacdba2e3d850e32903da133
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96573678"
---
# <a name="configure-the-upgrade-of-a-service-fabric-application-in-visual-studio"></a>Configuration de la mise à niveau d’une application Service Fabric dans Visual Studio
Les outils Visual Studio pour Azure Service Fabric fournissent une prise en charge des mises à niveau pour la publication vers des clusters locaux ou distants. Voici les trois cas dans lesquels vous devriez mettre à niveau votre application vers une version plus récente au lieu de la remplacer durant les tests et le débogage :

* Les données d’application ne sont pas perdues lors de la mise à niveau.
* La disponibilité reste élevée, car le service n’est pas interrompu au cours de la mise à niveau s’il y a suffisamment d’instances de service réparties sur plusieurs domaines de mise à niveau.
* Une application peut faire l’objet de tests pendant sa mise à niveau.

## <a name="parameters-needed-to-upgrade"></a>Paramètres nécessaires à la mise à niveau
Vous avez le choix entre deux types de déploiement : standard ou mise à niveau. Un déploiement standard efface les informations relatives à tout déploiement précédent ainsi que les données du cluster, tandis qu’un déploiement de mise à niveau les conserve. Lorsque vous mettez à niveau une application Service Fabric dans Visual Studio, vous devez fournir les paramètres de mise à niveau de l'application et les stratégies de contrôle d’intégrité. Les paramètres de mise à niveau de l’application permettent de contrôler la mise à niveau, tandis que les stratégies de contrôle d’intégrité déterminent si la mise à niveau a réussi. Pour en savoir plus, consultez [Mise à niveau d’une application Service Fabric : paramètres de mise à niveau](service-fabric-application-upgrade-parameters.md) .

Il existe trois modes de mise à niveau : *Monitored*, *UnmonitoredAuto* et *UnmonitoredManual*.

* Une mise à niveau Monitored automatise la mise à niveau et le contrôle d’intégrité de l’application.
* Une mise à niveau UnmonitoredAuto automatise la mise à niveau, mais ignore le contrôle d’intégrité de l’application.
* Quand vous effectuez une mise à niveau UnmonitoredManual, vous devez mettre à niveau manuellement chaque domaine de mise à niveau.

Chaque mode de mise à niveau nécessite différents jeux de paramètres. Pour en savoir plus sur les options de mise à niveau disponibles, consultez [Paramètres de mise à niveau d’application](service-fabric-application-upgrade-parameters.md) .

## <a name="upgrade-a-service-fabric-application-in-visual-studio"></a>Mise à niveau d’une application Service Fabric dans Visual Studio
Si vous utilisez les outils Service Fabric de Visual Studio pour mettre à niveau une application Service Fabric, vous pouvez préciser si le processus de publication doit être une mise à niveau plutôt qu’un déploiement standard en cochant la case **Mettre à niveau l’application** .

### <a name="to-configure-the-upgrade-parameters"></a>Pour configurer les paramètres de mise à niveau
1. Cliquez sur le bouton **Paramètres** en regard de la case à cocher. La boîte de dialogue **Modifier les paramètres de mise à niveau** s’affiche. La boîte de dialogue **Modifier les paramètres de mise à niveau** prend en charge les modes de mise à niveau Monitored, UnmonitoredAuto et UnmonitoredManual.
2. Sélectionnez le mode de mise à niveau que vous souhaitez utiliser, puis remplissez la grille de paramètres.

    Chaque paramètre a des valeurs par défaut. Le paramètre facultatif *DefaultServiceTypeHealthPolicy* accepte une entrée de table de hachage. Voici un exemple du format d'entrée de table de hachage pour *DefaultServiceTypeHealthPolicy*:

    ```
    @{ ConsiderWarningAsError = "false"; MaxPercentUnhealthyDeployedApplications = 0; MaxPercentUnhealthyServices = 0; MaxPercentUnhealthyPartitionsPerService = 0; MaxPercentUnhealthyReplicasPerPartition = 0 }
    ```

    *ServiceTypeHealthPolicyMap* est un autre paramètre facultatif qui acceptant une entrée de table de hachage au format suivant :

    ```    
    @ {"ServiceTypeName" : "MaxPercentUnhealthyPartitionsPerService,MaxPercentUnhealthyReplicasPerPartition,MaxPercentUnhealthyServices"}
    ```

    Voici un exemple concret :

    ```
    @{ "ServiceTypeName01" = "5,10,5"; "ServiceTypeName02" = "5,5,5" }
    ```
3. Si vous sélectionnez le mode de mise à niveau UnmonitoredManual, vous devrez démarrer manuellement une console PowerShell pour continuer et terminer le processus de mise à niveau. Pour en savoir plus sur le fonctionnement de la mise à niveau manuelle, consultez [Mise à niveau d’une application Service Fabric : rubriques avancées](service-fabric-application-upgrade-advanced.md) .

## <a name="upgrade-an-application-by-using-powershell"></a>Mettre à niveau une application à l’aide de PowerShell
Vous pouvez utiliser les applets de commande PowerShell pour mettre à niveau une application Service Fabric. Pour plus d’informations, consultez [Didacticiel sur la mise à niveau d’une application Service Fabric](service-fabric-application-upgrade-tutorial.md) et [Start-ServiceFabricApplicationUpgrade](/powershell/module/servicefabric/start-servicefabricapplicationupgrade).

## <a name="specify-a-health-check-policy-in-the-application-manifest-file"></a>Spécifier une stratégie de contrôle d’intégrité dans le fichier manifeste d’application
Chaque service d’une application Service Fabric peut avoir ses propres paramètres de stratégie de contrôle d'intégrité, qui remplacent alors les valeurs par défaut. Vous pouvez définir les valeurs de ces paramètres dans le fichier manifeste d’application.

L'exemple suivant montre comment appliquer une stratégie de contrôle d'intégrité unique pour chaque service du manifeste d'application.

```xml
<Policies>
    <HealthPolicy ConsiderWarningAsError="false" MaxPercentUnhealthyDeployedApplications="20">
        <DefaultServiceTypeHealthPolicy MaxPercentUnhealthyServices="20"               
                MaxPercentUnhealthyPartitionsPerService="20"
                MaxPercentUnhealthyReplicasPerPartition="20" />
        <ServiceTypeHealthPolicy ServiceTypeName="ServiceTypeName1"
                MaxPercentUnhealthyServices="20"
                MaxPercentUnhealthyPartitionsPerService="20"
                MaxPercentUnhealthyReplicasPerPartition="20" />      
    </HealthPolicy>
</Policies>
```
## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur la mise à niveau d’une application, consultez [Mettre à niveau une application à l’aide de Visual Studio](service-fabric-application-upgrade-tutorial.md).
