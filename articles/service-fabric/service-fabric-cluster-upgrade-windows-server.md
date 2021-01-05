---
title: Mettre à niveau la version d’un cluster autonome
description: Mettez à niveau le code d’Azure Service Fabric qui exécute un cluster Service Fabric autonome.
ms.topic: conceptual
ms.date: 11/09/2018
ms.openlocfilehash: 5831fcffe718e4966c3444a29d35c5bbd55e5f57
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91842935"
---
# <a name="upgrade-the-service-fabric-version-that-runs-on-your-cluster"></a>Mettre à niveau la version de Service Fabric qui s’exécute sur votre cluster 

Pour les systèmes modernes, la possibilité de mettre à niveau est essentielle à la réussite à long terme de votre produit. Un cluster Azure Service Fabric est une ressource que vous possédez. Cet article explique comment mettre à niveau la version de Service Fabric en cours d’exécution sur votre cluster autonome.

> [!NOTE]
> Veillez à ce que votre cluster exécute toujours une version de Service Fabric prise en charge. Lorsque Microsoft annonce le lancement d’une nouvelle version de Service Fabric, la fin de la prise en charge de la version précédente est signalée 60 jours minimum après la date de l’annonce. Les nouvelles versions sont annoncées [sur le blog de l’équipe Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric). La nouvelle version peut alors être sélectionnée.
>
>

Vous ne pouvez mettre à niveau votre cluster vers la nouvelle version que si vous utilisez une configuration de nœud de type production, où chaque nœud Service Fabric est alloué sur un ordinateur physique ou une machine virtuelle distincts. Si vous disposez d’un cluster de développement dans lequel plusieurs nœuds Service Fabric se trouvent sur un seul ordinateur physique ou une seule machine virtuelle, vous devez recréer le cluster avec la nouvelle version.

Deux workflows distincts permettent de mettre à niveau votre cluster vers la dernière version ou une version prise en charge de Service Fabric. L’un concerne les clusters disposant d’une connectivité suffisante pour télécharger automatiquement la version la plus récente. L’autre workflow est destiné aux clusters ne disposant pas d’une connectivité suffisante pour télécharger la version de Service Fabric la plus récente.

## <a name="enable-auto-upgrade-of-the-service-fabric-version-of-your-cluster"></a>Activer la mise à niveau automatique de la version de Service Fabric de votre cluster
Pour configurer votre cluster afin qu’il télécharge les mises à jour de Service Fabric quand Microsoft publie une nouvelle version, affectez la valeur *true* à la configuration de cluster `fabricClusterAutoupgradeEnabled`. Pour sélectionner manuellement une version prise en charge de Service Fabric pour votre cluster, affectez la valeur *false* à la configuration de cluster `fabricClusterAutoupgradeEnabled`.

## <a name="upgrade-clusters-that-have-connectivity-to-download-the-latest-code-and-configuration"></a>Mettre à niveau les clusters disposant d’une connectivité suffisante pour télécharger le code et la configuration les plus récents
Si vos nœuds de cluster disposent d’une connectivité Internet au [Centre de téléchargement Microsoft](https://download.microsoft.com), suivez les étapes ci-après pour mettre à niveau votre cluster vers une version prise en charge.

Pour les clusters qui disposent d’une connectivité au [Centre de téléchargement Microsoft](https://download.microsoft.com), Microsoft vérifie régulièrement la disponibilité de nouvelles versions de Service Fabric.

Quand une nouvelle version de Service Fabric est disponible, le package est téléchargé localement dans le cluster et configuré pour la mise à niveau. De plus, pour informer l’utilisateur de cette nouvelle version, le système montre un avertissement explicite concernant l’intégrité du cluster, semblable à ce qui suit :

« La prise en charge de la version actuelle du cluster [numéro de version] se terminera le [date]. »

Une fois que le cluster exécute la version la plus récente, l’avertissement est retiré.

Quand l’avertissement concernant l’intégrité du cluster s’affiche, mettez à niveau le cluster :

1. Connectez-vous au cluster à partir d’une machine virtuelle disposant d’un accès administrateur à toutes les machines qui sont répertoriées en tant que nœuds dans le cluster. L’ordinateur sur lequel ce script est exécuté ne doit pas nécessairement faire partie du cluster.

    ```powershell
    ###### connect to the secure cluster using certs
    $ClusterName= "mysecurecluster.something.com:19000"
    $CertThumbprint= "70EF5E22ADB649799DA3C8B6A6BF7FG2D630F8F3"
    Connect-serviceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
        -X509Credential `
        -ServerCertThumbprint $CertThumbprint  `
        -FindType FindByThumbprint `
        -FindValue $CertThumbprint `
        -StoreLocation CurrentUser `
        -StoreName My
    ```

2. Obtenez la liste des versions de Service Fabric vers lesquelles vous pouvez effectuer une mise à niveau.

    ```powershell
    ###### Get the list of available Service Fabric versions
    Get-ServiceFabricRegisteredClusterCodeVersion
    ```

    La sortie est normalement de ce type :

    ![Obtenir les versions de Service Fabric prises en charge][getfabversions]
3. Démarrez une mise à niveau du cluster vers une version disponible avec la commande Windows PowerShell [Start-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/start-servicefabricclusterupgrade).

    ```powershell
    Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion <codeversion#> -Monitored -FailureAction Rollback

    ###### Here is a filled-out example

    Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion 5.3.301.9590 -Monitored -FailureAction Rollback
    ```
   Pour surveiller la progression de la mise à niveau, vous pouvez utiliser Service Fabric Explorer ou exécuter la commande PowerShell suivante :

    ```powershell
    Get-ServiceFabricClusterUpgrade
    ```

    Si les stratégies d’intégrité du cluster ne sont pas respectées, la mise à niveau est annulée. Pour spécifier des stratégies d’intégrité personnalisées pour la commande ServiceFabricClusterUpgrade, consultez la documentation de [Start-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/start-servicefabricclusterupgrade).

    Après avoir corrigé les problèmes ayant entraîné la restauration, relancez la mise à niveau, en suivant les étapes décrites précédemment.

## <a name="upgrade-clusters-that-have-no-connectivity-to-download-the-latest-code-and-configuration"></a>Mettre à niveau les clusters *ne disposant pas d’une connectivité* suffisante pour télécharger le code et la configuration les plus récents
Si vos nœuds de cluster disposent d’une connectivité Internet au [Centre de téléchargement Microsoft](https://download.microsoft.com), suivez les étapes ci-après pour mettre à niveau votre cluster vers une version prise en charge.

> [!NOTE]
> Si vous utilisez un cluster qui n’est pas connecté à Internet, vous devez consulter régulièrement le [blog de l’équipe Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) pour être averti de la disponibilité des nouvelles versions. Le système n’affiche pas d’avertissement concernant l’intégrité du cluster vous alertant en cas de disponibilité d’une nouvelle version.  
>
>

### <a name="auto-provisioning-vs-manual-provisioning"></a>Provisionnement automatique vs. provisionnement manuel
Pour activer le téléchargement et l’inscription automatiques pour la dernière version du code, configurez le service de mise à jour de Service Fabric. Pour obtenir des instructions, consultez *Tools\ServiceFabricUpdateService.zip\Readme_InstructionsAndHowTos.txt* dans le [package autonome](service-fabric-cluster-standalone-package-contents.md).

Pour le processus manuel, procédez comme suit.

Modifiez la configuration du cluster pour affecter la valeur *false* à la propriété suivante avant de commencer la mise à niveau d’une configuration :

```json
"fabricClusterAutoupgradeEnabled": false,
```

Pour plus d’informations sur l’utilisation, reportez-vous à la commande PowerShell [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade). Veillez à mettre à jour « clusterConfigurationVersion » dans votre code JSON avant de commencer la mise à niveau de la configuration.

```powershell
    Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>
```

### <a name="cluster-upgrade-workflow"></a>Flux de travail de mise à niveau de cluster

1. Exécutez [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade) à partir de l’un des nœuds du cluster et notez la valeur de *TargetCodeVersion*.

2. Exécutez la commande suivante à partir d’un ordinateur connecté à Internet pour répertorier toutes les versions de mise à niveau compatibles avec la version actuelle et téléchargez le package correspondant à l’aide des liens de téléchargement associés :

    ```powershell
    ###### Get list of all upgrade compatible packages  
    Get-ServiceFabricRuntimeUpgradeVersion -BaseVersion <TargetCodeVersion as noted in Step 1> 
    ```

3. Connectez-vous au cluster à partir d’une machine virtuelle disposant d’un accès administrateur à toutes les machines qui sont répertoriées en tant que nœuds dans le cluster. L’ordinateur sur lequel ce script est exécuté ne doit pas nécessairement faire partie du cluster.

    ```powershell
    ###### Get the list of available Service Fabric versions
    Copy-ServiceFabricClusterPackage -Code -CodePackagePath <name of the .cab file including the path to it> -ImageStoreConnectionString "fabric:ImageStore"

    ###### Here is a filled-out example
    Copy-ServiceFabricClusterPackage -Code -CodePackagePath .\MicrosoftAzureServiceFabric.5.3.301.9590.cab -ImageStoreConnectionString "fabric:ImageStore"
    ```
4. Copiez le package téléchargé dans le magasin d’images du cluster.

5. Inscrivez le package copié.

    ```powershell
    ###### Get the list of available Service Fabric versions
    Register-ServiceFabricClusterPackage -Code -CodePackagePath <name of the .cab file>

    ###### Here is a filled-out example
    Register-ServiceFabricClusterPackage -Code -CodePackagePath MicrosoftAzureServiceFabric.5.3.301.9590.cab
    ```
6. Commencez une mise à niveau de cluster vers une version disponible.

    ```powershell
    Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion <codeversion#> -Monitored -FailureAction Rollback

    ###### Here is a filled-out example
    Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion 5.3.301.9590 -Monitored -FailureAction Rollback
    ```
    Vous pouvez surveiller la progression de la mise à niveau avec Service Fabric Explorer ou exécuter la commande PowerShell suivante :

    ```powershell
    Get-ServiceFabricClusterUpgrade
    ```

    Si les stratégies d’intégrité du cluster ne sont pas respectées, la mise à niveau est annulée. Pour spécifier des stratégies d’intégrité personnalisées pour la commande ServiceFabricClusterUpgrade, consultez la documentation de [Start-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/start-servicefabricclusterupgrade).

    Après avoir corrigé les problèmes ayant entraîné la restauration, relancez la mise à niveau, en suivant les étapes décrites précédemment.

## <a name="next-steps"></a>Étapes suivantes
* [Mettre à niveau la configuration d’un cluster autonome](service-fabric-cluster-config-upgrade-windows-server.md)
* Personnalisez certains [paramètres de cluster Service Fabric](service-fabric-cluster-fabric-settings.md).
* [Effectuez un scale-in et un scale-out de votre cluster](service-fabric-cluster-scale-in-out.md).

<!--Image references-->
[getfabversions]: ./media/service-fabric-cluster-upgrade-windows-server/getfabversions.PNG
