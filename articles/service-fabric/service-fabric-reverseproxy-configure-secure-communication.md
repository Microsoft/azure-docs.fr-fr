---
title: Communication sécurisée avec le proxy inverse Azure Service Fabric
description: Configurez le proxy inverse pour permettre la communication de bout en bout sécurisée dans une application Azure Service Fabric.
ms.topic: conceptual
ms.date: 08/10/2017
ms.openlocfilehash: b01ce559b3c790164992d6618149afa9df069466
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86256133"
---
# <a name="connect-to-a-secure-service-with-the-reverse-proxy"></a>Se connecter à un service sécurisé avec le proxy inverse

Cet article explique comment établir une connexion sécurisée entre le proxy inverse et les services, permettant ainsi un canal sécurisé de bout en bout. Pour plus d’informations sur le proxy inverse, consultez [Proxy inverse dans Azure Service Fabric](service-fabric-reverseproxy.md)

> [!IMPORTANT]
> La connexion à des services sécurisés est prise en charge uniquement quand le proxy inverse est configuré pour écouter sur le protocole HTTPS. Cet article suppose que c’est le cas. Pour configurer le proxy inverse dans Service Fabric, reportez-vous à [Configurer le proxy inverse dans Azure Service Fabric](service-fabric-reverseproxy-setup.md).

## <a name="secure-connection-establishment-between-the-reverse-proxy-and-services"></a>Établissement d’une connexion sécurisée entre le proxy inverse et les services 

### <a name="reverse-proxy-authenticating-to-services"></a>Authentification du proxy inverse auprès des services :
Le proxy inverse s’identifie lui-même auprès des services avec son certificat. Pour les clusters Azure, le certificat est spécifié avec la propriété ***reverseProxyCertificate*** dans la section [Type de ressource](../azure-resource-manager/templates/template-syntax.md) de [**Microsoft.ServiceFabric/clusters**](/azure/templates/microsoft.servicefabric/clusters) du Modèle Resource Manager. Pour les clusters autonomes, le certificat est spécifié avec la propriété ***ReverseProxyCertificate*** ou ***ReverseProxyCertificateCommonNames*** dans la section **Security** du fichier ClusterConfig.json. Pour plus d’informations, consultez [Activer le proxy inverse sur des clusters autonomes](service-fabric-reverseproxy-setup.md#enable-reverse-proxy-on-standalone-clusters). 

Les services peuvent implémenter la logique qui vérifie le certificat présenté par le serveur proxy inverse. Les services peuvent spécifier les détails du certificat client accepté en tant que paramètres de configuration dans le package de configuration. Celui-ci peut être lu au moment de l’exécution et utilisé pour valider le certificat présenté par le proxy inverse. Reportez-vous à [Gestion des paramètres d’application](service-fabric-manage-multiple-environment-app-configuration.md) pour ajouter les paramètres de configuration. 

### <a name="reverse-proxy-verifying-the-services-identity-via-the-certificate-presented-by-the-service"></a>Vérification par le proxy inverse de l’identité du service via le certificat présenté par le service :
Le proxy inverse prend en charge les stratégies suivantes pour effectuer la validation de certificat de serveur des certificats présentés par les services : Aucun, ServiceCommonNameAndIssuer et ServiceCertificateThumbprints.
Pour sélectionner la stratégie à utiliser pour le proxy inverse, spécifiez le paramètre **ApplicationCertificateValidationPolicy** dans la section **ApplicationGateway/Http** sous [fabricSettings](service-fabric-cluster-fabric-settings.md).

La section suivante montre les détails de configuration de chacune de ces options.

### <a name="service-certificate-validation-options"></a>Options de validation du certificat de service 

- **Aucun** : le proxy inverse ignore la vérification du certificat de service proxy et établit une connexion sécurisée. Il s'agit du comportement par défaut.
Attribuez à **ApplicationCertificateValidationPolicy** la valeur **None** dans la section [**ApplicationGateway/Http**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttp).

   ```json
   {
   "fabricSettings": [
             ...
             {
               "name": "ApplicationGateway/Http",
               "parameters": [
                 {
                   "name": "ApplicationCertificateValidationPolicy",
                   "value": "None"
                 }
               ]
             }
           ],
           ...
   }
   ```

- **ServiceCommonNameAndIssuer** : le proxy inverse vérifie le certificat présenté par le service en fonction du nom commun du certificat et de l’empreinte de l’émetteur immédiat : Attribuez à **ApplicationCertificateValidationPolicy** la valeur **ServiceCommonNameAndIssuer** dans la section [**ApplicationGateway/Http**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttp).

   ```json
   {
   "fabricSettings": [
             ...
             {
               "name": "ApplicationGateway/Http",
               "parameters": [
                 {
                   "name": "ApplicationCertificateValidationPolicy",
                   "value": "ServiceCommonNameAndIssuer"
                 }
               ]
             }
           ],
           ...
   }
   ```

   Pour spécifier la liste des empreintes de l’émetteur et des noms communs de service, ajoutez une section [**ApplicationGateway/Http/ServiceCommonNameAndIssuer**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttpservicecommonnameandissuer) sous **fabricSettings**, comme ci-dessous. Il est possible d’ajouter plusieurs paires nom commun du certificat/empreinte d’émetteur au tableau **parameters**. 

   Si le proxy inverse du point de terminaison se connecte pour présenter un certificat dont le nom commun et l’empreinte de l’émetteur correspondent à l’une des valeurs spécifiées ici, un canal TLS est établi.
   Si aucune des valeurs ne correspond aux détails du certificat, le proxy inverse fait échouer la demande du client avec un code d’état 502 (passerelle incorrecte). La ligne d’état HTTP contient également l’expression « Certificat SSL non valide ». 

   ```json
   {
   "fabricSettings": [
             ...
             {
               "name": "ApplicationGateway/Http/ServiceCommonNameAndIssuer",
               "parameters": [
                 {
                   "name": "WinFabric-Test-Certificate-CN1",
                   "value": "b3 44 9b 01 8d 0f 68 39 a2 c5 d6 2b 5b 6c 6a c8 22 b4 22 11"
                 },
                 {
                   "name": "WinFabric-Test-Certificate-CN2",
                   "value": "b3 44 9b 01 8d 0f 68 39 a2 c5 d6 2b 5b 6c 6a c8 22 11 33 44"
                 }
               ]
             }
           ],
           ...
   }
   ```

- **ServiceCertificateThumbprints** : le proxy inverse vérifie le certificat de service proxy en fonction de son empreinte. Vous pouvez opter pour cette approche si les services sont configurés avec des certificats auto-signés : Attribuez à **ApplicationCertificateValidationPolicy** la valeur **ServiceCertificateThumbprints** dans la section [**ApplicationGateway/Http**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttp).

   ```json
   {
   "fabricSettings": [
             ...
             {
               "name": "ApplicationGateway/Http",
               "parameters": [
                 {
                   "name": "ApplicationCertificateValidationPolicy",
                   "value": "ServiceCertificateThumbprints"
                 }
               ]
             }
           ],
           ...
   }
   ```

   Spécifiez également les empreintes avec une entrée **ServiceCertificateThumbprints** dans la section **ApplicationGateway/Http**. Plusieurs empreintes peuvent être spécifiées sous forme de liste séparée par des virgules dans le champ value, comme indiqué ci-dessous :

   ```json
   {
   "fabricSettings": [
             ...
             {
               "name": "ApplicationGateway/Http",
               "parameters": [
                   ...
                 {
                   "name": "ServiceCertificateThumbprints",
                   "value": "78 12 20 5a 39 d2 23 76 da a0 37 f0 5a ed e3 60 1a 7e 64 bf,78 12 20 5a 39 d2 23 76 da a0 37 f0 5a ed e3 60 1a 7e 64 b9"
                 }
               ]
             }
           ],
           ...
   }
   ```

   Si l’empreinte du certificat de serveur est listée dans cette entrée de configuration, le proxy inverse fait réussir la connexion TLS. Dans le cas contraire, il met fin à la connexion et fait échouer la demande du client avec un code 502 (passerelle incorrecte). La ligne d’état HTTP contient également l’expression « Certificat SSL non valide ».

## <a name="endpoint-selection-logic-when-services-expose-secure-as-well-as-unsecured-endpoints"></a>Logique de sélection du point de terminaison quand les services exposent des points de terminaison sécurisés et non sécurisés
Service Fabric prend en charge la configuration de plusieurs points de terminaison pour un service. Pour plus d’informations, voir [Spécifier des ressources dans un manifeste de service](service-fabric-service-manifest-resources.md).

Le proxy inverse sélectionne un des points de terminaison pour transférer la demande en fonction du paramètre de requête **ListenerName** dans [l’URI de service](./service-fabric-reverseproxy.md#uri-format-for-addressing-services-by-using-the-reverse-proxy). Si le paramètre **ListenerName** n’est pas spécifié, le proxy inverse peut choisir n’importe quel point de terminaison de la liste des points de terminaison. Il peut s’agir d’un point de terminaison HTTP ou HTTPS selon les points de terminaison configurés pour le service. Certains scénarios ou prérequis exigent que le proxy inverse fonctionne en « mode sécurisé exclusivement », autrement dit, que le proxy inverse sécurisé ne transfère pas de requêtes à des points de terminaison non sécurisés. Pour configurer ce mode, attribuez à l’entrée de configuration **SecureOnlyMode** la valeur **true** dans la section [**ApplicationGateway/Http**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttp).   

```json
{
"fabricSettings": [
          ...
          {
            "name": "ApplicationGateway/Http",
            "parameters": [
                ...
              {
                "name": "SecureOnlyMode",
                "value": true
              }
            ]
          }
        ],
        ...
}
```

> [!NOTE]
> En mode **SecureOnlyMode**, si le client a spécifié un **ListenerName** correspondant à un point de terminaison HTTP (non sécurisé), le proxy inverse fait échouer la requête avec un code d’état HTTP 404 (introuvable).

## <a name="setting-up-client-certificate-authentication-through-the-reverse-proxy"></a>Configuration de l’authentification de certificat client par le biais du proxy inverse
L’arrêt TLSL se produit au niveau du proxy inverse et toutes les données de certificat client sont perdues. Pour que les services effectuent l’authentification par certificat client, définissez le paramètre **ForwardClientCertificate** de la section [**ApplicationGateway/Http**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttp).

1. Quand **ForwardClientCertificate** est défini sur **false**, le proxy inverse ne demande pas le certificat client pendant sa négociation TLS avec le client.
Il s'agit du comportement par défaut.

2. Quand **ForwardClientCertificate** est défini sur **true**, le proxy inverse demande le certificat du client pendant sa négociation TLS avec le client.
Ensuite, il envoie les données du certificat client dans un en-tête HTTP personnalisé nommé **X-Client-Certificate**. La valeur d’en-tête est la chaîne de format PEM encodée en base 64 du certificat du client. Le service peut faire réussir ou échouer la demande avec le code d’état approprié après avoir inspecté les données du certificat.
Si le client ne présente pas un certificat, le proxy inverse transfère un en-tête vide et laisse le service gérer le cas.

> [!NOTE]
> Le proxy inverse agit uniquement comme un service de transfert. Il n’effectue aucune validation du certificat du client.


## <a name="next-steps"></a>Étapes suivantes
* [Installer et configurer un proxy inverse sur un cluster](service-fabric-reverseproxy-setup.md).
* Voir [Configurer le proxy inverse pour se connecter à des services sécurisés](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/Reverse-Proxy-Sample#configure-reverse-proxy-to-connect-to-secure-services).
* Pour obtenir un exemple de communication HTTP entre services, consultez cet [exemple de projet sur GitHub](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started).
* [Appels de procédure distante avec Reliable Services à distance](service-fabric-reliable-services-communication-remoting.md)
* [API Web qui utilise OWIN dans Reliable Services](./service-fabric-reliable-services-communication-aspnetcore.md)
* [Gérer les certificats de cluster](service-fabric-cluster-security-update-certs-azure.md)
