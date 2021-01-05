---
title: Azure Service Fabric - Service DNS
description: Utilisez le service DNS de Service Fabric pour détecter des microservices depuis le cluster.
ms.topic: conceptual
ms.date: 7/20/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: f7f06920820cdc73f8d3101ab24ee46625931ee4
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91268041"
---
# <a name="dns-service-in-azure-service-fabric"></a>Service DNS dans Azure Service Fabric
Le service DNS est un service système facultatif que vous pouvez activer dans votre cluster pour détecter d’autres services utilisant le protocole DNS. 

De nombreux services, en particulier les services en conteneur, sont adressables via une URL préexistante. Il est souhaitable d’être en mesure de résoudre ces services à l’aide du protocole DNS standard, plutôt qu’avec le protocole Service Fabric Naming Service. Le service DNS vous permet de mapper les noms DNS à un nom de service et, par conséquent, de résoudre les adresses IP des points de terminaison. Cette fonctionnalité conserve la portabilité des services en conteneur sur différentes plateformes et peut simplifier les scénarios de type « lift and shift », en vous laissant utiliser des URL de service existantes sans avoir à réécrire le code pour tirer parti du Naming Service. 

Le service DNS mappe les noms DNS aux noms de service, qui sont à leur tour résolus par le service de nommage pour retourner le point de terminaison de service. Le nom DNS du service est fourni au moment de la création. Le schéma suivant illustre le mode de fonctionnement du service DNS pour des services sans état.

![Diagramme montrant comment les noms DNS sont mappés aux noms de service par le service DNS pour les services sans état.](./media/service-fabric-dnsservice/stateless-dns.png)

À compter de la version 6.3 de Service Fabric, le protocole DNS de Service Fabric a été étendu pour inclure un schéma d’adressage des services partitionnés avec état. Ces extensions permettent de résoudre des adresses IP de partition spécifiques à l’aide d’une combinaison composée à la fois du nom DNS du service avec état et du nom de la partition. Les trois schémas de partitionnement sont pris en charge :

- Partitionnement nommé
- Partitionnement par plages de valeurs
- Partitionnement singleton

Le schéma suivant montre le fonctionnement du service DNS pour des services partitionnés avec état.

![Diagramme montrant comment les noms DNS sont mappés aux noms de service par le service DNS pour les services sans état partitionnés.](./media/service-fabric-dnsservice/stateful-dns.png)

Les ports dynamiques ne sont pas pris en charge par le service DNS. Pour résoudre des services exposés sur des ports dynamiques, utilisez le [service de proxy inverse](./service-fabric-reverseproxy.md).

## <a name="enabling-the-dns-service"></a>Activation du service DNS
> [!NOTE]
> Le service DNS pour les services Service Fabric n’est pas encore pris en charge sur Linux.

Quand vous créez un cluster à l’aide du portail, le service DNS est activé par défaut dans la case à cocher **Inclure le service DNS** du menu **Configuration du cluster** :

![Activation du service DNS via le portail](./media/service-fabric-dnsservice/enable-dns-service.png)

Si vous n’utilisez pas le portail pour créer votre cluster ou si vous mettez à jour un cluster existant, vous devez activer le service DNS dans un modèle :

- Pour déployer un nouveau cluster, vous pouvez utiliser les [exemples de modèles](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype) ou créer votre propre modèle Resource Manager. 
- Pour mettre à jour un cluster existant, vous pouvez accéder au groupe de ressources du cluster dans le portail et cliquer sur **Script d’automatisation** pour travailler avec un modèle qui reflète l’état actuel du cluster et d’autres ressources du groupe. Pour plus d’informations, consultez [Exportation du modèle à partir d’un groupe de ressources](../azure-resource-manager/templates/export-template-portal.md).

Une fois que vous avez un modèle, vous pouvez activer le service DNS en effectuant les étapes suivantes :

1. Vérifiez que `apiversion` a la valeur `2017-07-01-preview` ou plus pour la ressource `Microsoft.ServiceFabric/clusters` ; dans le cas contraire, procédez à une mise à jour comme indiqué dans l’exemple suivant :

    ```json
    {
        "apiVersion": "2017-07-01-preview",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
    }
    ```

2. Vous pouvez maintenant activer le service DNS en procédant de l’une des manières suivantes :

   - Pour activer le service DNS avec les paramètres par défaut, ajoutez-le à la section `addonFeatures` à l’intérieur de la section `properties`, comme indiqué dans l’exemple suivant :

        ```json
          "properties": {
            ...
            "addonFeatures": [
              "DnsService"
              ],
            ...
          }
        ```

   - Pour activer le service avec d’autres paramètres que les paramètres par défaut, ajoutez une section `DnsService` à la section `fabricSettings` à l’intérieur de la section `properties`. Dans ce cas, vous n’avez pas besoin d’ajouter le DnsService à `addonFeatures`. Pour en savoir plus sur les propriétés que vous pouvez définir pour le service DNS, consultez les [Paramètres du service DNS](./service-fabric-cluster-fabric-settings.md#dnsservice).

       ```json
           "properties": {
             ...  
             "fabricSettings": [
               ...
               {
                 "name": "DnsService",
                 "parameters": [
                   {
                     "name": "IsEnabled",
                     "value": "true"
                   },
                   {
                     "name": "PartitionSuffix",
                     "value": "--"
                   },
                   {
                     "name": "PartitionPrefix",
                     "value": "--"
                   }
                 ]
               },
               ...
              ]
            }
       ```
3. Après avoir mis à jour le modèle de cluster avec ces modifications, appliquez-les et laissez la mise à niveau s’accomplir. Une fois la mise à niveau terminée, le service DNS démarre votre cluster. Le nom du service est `fabric:/System/DnsService`. Il est mentionné dans la section **Système** de Service Fabric explorer. 

> [!NOTE]
> Lors de la mise à niveau du DNS de désactivé à activé, Service Fabric Explorer peut ne pas refléter le nouvel état. Pour résoudre ce problème, redémarrez les nœuds en modifiant UpgradePolicy dans votre modèle Azure Resource Manager. Pour plus d’informations, consultez la [Référence du modèle Service Fabric](/azure/templates/microsoft.servicefabric/2019-03-01/clusters/applications).

> [!NOTE]
> L’activation du service DNS lors du développement sur un ordinateur local remplace certains paramètres DNS. Si vous rencontrez des problèmes lors de la connexion à Internet, vérifiez vos paramètres DNS.

## <a name="setting-the-dns-name-for-your-service"></a>Configuration du nom DNS de votre service
Vous pouvez définir un nom DNS pour vos services, soit de manière déclarative pour les services par défaut dans le fichier ApplicationManifest.xml, soit à l’aide de commandes PowerShell.

Le nom DNS de votre service ne peut pas être résolu au sein du cluster, c’est pourquoi il est important de garantir l’unicité du nom DNS au sein du cluster. 

Nous vous recommandons vivement d’utiliser un schéma de nommage `<ServiceDnsName>.<AppInstanceName>`, par exemple `service1.application1`. Si une application est déployée à l’aide de Docker Compose, des noms DNS sont affectés automatiquement aux services à l’aide de ce schéma de nommage.

### <a name="setting-the-dns-name-for-a-default-service-in-the-applicationmanifestxml"></a>Définition du nom DNS pour un service par défaut du paramètre dans le fichier ApplicationManifest.xml
Ouvrez votre projet dans Visual Studio ou dans votre éditeur favori, puis ouvrez le fichier ApplicationManifest.xml. Accédez à la section des services par défaut puis, pour chaque service, ajoutez l’attribut `ServiceDnsName`. L’exemple suivant montre comment définir le nom DNS du service sur `service1.application1`

```xml
    <Service Name="Stateless1" ServiceDnsName="service1.application1">
      <StatelessService ServiceTypeName="Stateless1Type" InstanceCount="[Stateless1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
```
Une fois l’application déployée, l’instance de service dans Service Fabric Explorer indique le nom DNS de cette instance, comme illustré dans la figure suivante : 

![points de terminaison de service](./media/service-fabric-dnsservice/service-fabric-explorer-dns.png)

L’exemple suivant définit sur `statefulsvc.app` le nom DNS d’un service avec état. Le service utilise un schéma de partitionnement nommé. Notez que les noms de partition sont en minuscules. Il est nécessaire de respecter la casse pour les partitions qui seront ciblés dans les requêtes DNS ; pour plus d’informations, consultez la section [Exécution de requêtes DNS sur une partition de service avec état](#preview-making-dns-queries-on-a-stateful-service-partition).

```xml
    <Service Name="Stateful1" ServiceDnsName="statefulsvc.app" />
      <StatefulService ServiceTypeName="ProcessingType" TargetReplicaSetSize="2" MinReplicaSetSize="2">
        <NamedPartition>
         <Partition Name="partition1" />
         <Partition Name="partition2" />
        </NamedPartition>
      </StatefulService>
    </Service>
```

### <a name="setting-the-dns-name-for-a-service-using-powershell"></a>Définition du nom DNS d’un service à l’aide de Powershell
Vous pouvez définir le nom DNS d’un service lors de sa création à l’aide de la commande Powershell `New-ServiceFabricService`. L’exemple suivant permet de créer un service sans état avec le nom DNS `service1.application1`

```powershell
    New-ServiceFabricService `
    -Stateless `
    -PartitionSchemeSingleton `
    -ApplicationName `fabric:/application1 `
    -ServiceName fabric:/application1/service1 `
    -ServiceTypeName Service1Type `
    -InstanceCount 1 `
    -ServiceDnsName service1.application1
```

## <a name="preview-making-dns-queries-on-a-stateful-service-partition"></a>[Préversion] Exécution de requêtes DNS sur une partition de service avec état
À partir de Service Fabric version 6.3, le service DNS de Service Fabric prend en charge les requêtes pour les partitions de service.

Pour les partitions qui seront utilisées dans les requêtes DNS, les restrictions d’affectation de noms suivantes s’appliquent :

   - Les noms de partition doivent être compatibles avec DNS.
   - Les noms de partition à libellés multiples (dont le nom comporte un point, « . ») ne doivent pas être utilisés.
   - Les noms de partition doivent être en minuscules.

Les requêtes DNS qui ciblent une partition sont formatées comme suit :

```
    <First-Label-Of-Partitioned-Service-DNSName><PartitionPrefix><Target-Partition-Name>< PartitionSuffix>.<Remaining- Partitioned-Service-DNSName>
```
Où :

- *First-Label-Of-Partitioned-Service-DNSName* est la première partie du nom de votre service DNS.
- *PartitionPrefix* est une valeur qui peut être définie dans la section DnsService du manifeste du cluster ou via le modèle Resource Manager du cluster. La valeur par défaut est « -- ». Pour plus d’informations, consultez les [paramètres du service DNS](./service-fabric-cluster-fabric-settings.md#dnsservice).
- *Target-Partition-Name* est le nom de la partition. 
- *PartitionSuffix* est une valeur qui peut être définie dans la section DnsService du manifeste du cluster ou via le modèle Resource Manager du cluster. La valeur par défaut est une chaîne vide. Pour plus d’informations, consultez les [paramètres du service DNS](./service-fabric-cluster-fabric-settings.md#dnsservice).
- *Remaining-Partitioned-Service-DNSName* est la partie restante du nom DNS de votre service.

Les exemples suivants illustrent des requêtes DNS pour des services partitionnés exécutés sur un cluster qui utilise les paramètres par défaut pour `PartitionPrefix` et `PartitionSuffix` : 

- Pour résoudre la partition « 0 » d’un service avec le nom DNS `backendrangedschemesvc.application` qui utilise un schéma de partitionnement par plages de valeurs, utilisez `backendrangedschemesvc-0.application`.
- Pour résoudre la partition « first » d’un service avec le nom DNS `backendnamedschemesvc.application` qui utilise un schéma de partitionnement nommé, utilisez `backendnamedschemesvc-first.application`.

Le service DNS renvoie l’adresse IP du réplica principal de la partition. Si aucune partition n’est spécifiée, le service retourne l’adresse IP du réplica principal d’une partition sélectionnée de façon aléatoire.

## <a name="using-dns-in-your-services"></a>Utilisation de DNS dans vos services
Si vous déployez plusieurs services, vous pouvez utiliser un nom DNS pour trouver les points de terminaison d’autres services avec lesquels communiquer. Le service DNS fonctionne pour les services sans état et, à compter de la version 6.3 de Service Fabric, pour les services avec état. Pour les services avec état exécutés sur des versions de Service Fabric antérieures à la version 6.3, vous pouvez utiliser le [service de proxy inverse](./service-fabric-reverseproxy.md) intégré pour les appels http afin d’appeler une partition de service particulière. 

Les ports dynamiques ne sont pas pris en charge par le service DNS. Vous pouvez utiliser le service de proxy inverse pour résoudre des services qui utilisent des ports dynamiques.

Le code suivant montre comment appeler un service sans état via DNS. Il s’agit d’un simple appel http standard dans lequel vous fournissez le nom DNS, le port et un chemin d’accès facultatif dans le cadre de l’URL.

```csharp
public class ValuesController : Controller
{
    // GET api
    [HttpGet]
    public async Task<string> Get()
    {
        string result = "";
        try
        {
            Uri uri = new Uri("http://service1.application1:8080/api/values");
            HttpClient client = new HttpClient();
            var response = await client.GetAsync(uri);
            result = await response.Content.ReadAsStringAsync();
            
        }
        catch (Exception e)
        {
            Console.Write(e.Message);
        }

        return result;
    }
}
```

Le code suivant montre un appel sur une partition spécifique d’un service avec état. Dans ce cas, le nom DNS contient le nom de la partition (partition1). L’appel suppose que le cluster utilise les valeurs par défaut pour `PartitionPrefix` et `PartitionSuffix`.

```csharp
public class ValuesController : Controller
{
    // GET api
    [HttpGet]
    public async Task<string> Get()
    {
        string result = "";
        try
        {
            Uri uri = new Uri("http://service2-partition1.application1:8080/api/values");
            HttpClient client = new HttpClient();
            var response = await client.GetAsync(uri);
            result = await response.Content.ReadAsStringAsync();
            
        }
        catch (Exception e)
        {
            Console.Write(e.Message);
        }

        return result;
    }
}
```

## <a name="known-issues"></a>Problèmes connus
* Pour les versions 6.3 et ultérieures de Service Fabric, il y a un problème avec les recherches DNS pour les noms de service contenant un trait d’union dans le nom DNS. Pour plus d’informations sur ce problème, veuillez suivre le [Problème GitHub](https://github.com/Azure/service-fabric-issues/issues/1197) suivant. Une solution à ce problème est prévue dans la prochaine mise à jour 6.3. 

* Le service DNS pour les services Service Fabric n’est pas encore pris en charge sur Linux. Le service DNS est pris en charge pour les conteneurs sur Linux. L’alternative possible est la résolution manuelle à l’aide de Fabric Client/ServicePartitionResolver.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur la communication de service au sein du cluster, consultez l’article [Se connecter aux services et communiquer avec eux dans Service Fabric](service-fabric-connect-and-communicate-with-services.md)
