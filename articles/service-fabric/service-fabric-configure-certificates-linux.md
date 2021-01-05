---
title: Configurer des certificats pour les applications sur Linux
description: Configurer des certificats pour votre application avec le runtime Service Fabric sur un cluster Linux
ms.topic: conceptual
ms.date: 09/06/2019
ms.author: pepogors
ms.openlocfilehash: a97c8b8315fe3be405aed9c6570004afb8fafd1d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86258665"
---
# <a name="certificates-and-security-on-linux-clusters"></a>Certificats et sécurité sur des clusters Linux

Cet article fournit des informations sur la configuration des certificats X.509 sur des clusters Linux.

## <a name="location-and-format-of-x509-certificates-on-linux-nodes"></a>Emplacement et format des certificats X.509 sur les nœuds Linux

Service Fabric s’attend généralement à la présence de certificats X.509 dans le répertoire */var/lib/sfcerts* sur les nœuds de cluster Linux. Cela est vrai pour les certificats de cluster, les certificats clients, etc. Dans certains cas, vous pouvez spécifier un emplacement autre que le dossier *var/lib/sfcerts* pour les certificats. Par exemple, avec les services Reliable Services générés à l’aide du kit SDK Java Service Fabric, vous pouvez spécifier un autre emplacement via le package de configuration (Settings.xml) pour certains certificats spécifiques à l’application. Pour en savoir plus, consultez [Certificats référencés dans le package de configuration (Settings.xml)](#certificates-referenced-in-the-configuration-package-settingsxml).

Pour les clusters Linux, Service Fabric s’attend à ce que les certificats soient présents sous la forme d’un fichier .pem qui contient à la fois le certificat et une clé privée ou d’un fichier .crt qui contient le certificat et d’un fichier .key qui contient la clé privée. Tous les fichiers doivent être au format PEM. 

Si vous installez votre certificat à partir d’Azure Key Vault à l’aide d’un [modèle Resource Manager](./service-fabric-cluster-creation-create-template.md) ou de commandes [PowerShell](/powershell/module/az.servicefabric/?view=azps-2.6.0), le certificat est installé au format correct dans le répertoire */var/lib/sfcerts* sur chaque nœud. Si vous installez un certificat via une autre méthode, vous devez vous assurer qu’il est correctement installé sur les nœuds de cluster.

## <a name="certificates-referenced-in-the-application-manifest"></a>Certificats référencés dans le manifeste de l’application

Les certificats spécifiés dans le manifeste de l’application, par exemple via les éléments [**SecretsCertificate**](./service-fabric-service-model-schema-elements.md#secretscertificate-element) ou [**EndpointCertificate**](./service-fabric-service-model-schema-elements.md#endpointcertificate-element), doivent être présents dans le répertoire */var/lib/sfcerts*. Les éléments qui sont utilisés pour spécifier des certificats dans le manifeste de l’application n’acceptant pas d’attribut de chemin, les certificats doivent être présents dans le répertoire par défaut. Ces éléments acceptent un attribut **X509StoreName** facultatif. Il s’agit par défaut de « My », qui pointe vers le répertoire */var/lib/sfcerts* sur les nœuds Linux. Toute autre valeur est non définie sur un cluster Linux. Nous vous recommandons d’omettre l’attribut **X509StoreName** pour les applications exécutées sur les clusters Linux. 

## <a name="certificates-referenced-in-the-configuration-package-settingsxml"></a>Certificats référencés dans le package de configuration (Settings.xml)

Pour certains services, vous pouvez configurer des certificats X.509 dans [ConfigPackage](./service-fabric-application-and-service-manifests.md) (par défaut, Settings.xml). C’est par exemple le cas quand vous déclarez des certificats utilisés pour sécuriser des canaux RPC pour les services Reliable Services générés avec les kits SDK Java ou .NET Core Service Fabric. Il existe deux façons de référencer les certificats dans le package de configuration. La prise en charge varie entre les kits SDK Java et .NET Core.

### <a name="using-x509-securitycredentialstype"></a>Utilisation du type SecurityCredentialsType X509

Avec les kits SDK Java ou .NET, vous pouvez spécifier **X509** pour **SecurityCredentialsType**. Cela correspond au type `X509Credentials` ([.NET](/previous-versions/azure/reference/mt124925(v=azure.100))/[Java](/java/api/system.fabric.x509credentials)) de `SecurityCredentials` ([.NET](/previous-versions/azure/reference/mt124894(v=azure.100))/[Java](/java/api/system.fabric.securitycredentials)).

La référence **X509** localise le certificat dans un magasin de certificats. Le code XML suivant affiche les paramètres utilisés pour spécifier l’emplacement du certificat :

```xml
    <Parameter Name="SecurityCredentialsType" Value="X509" />
    <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
    <Parameter Name="CertificateStoreName" Value="My" />
```

Pour un service exécuté sous Linux, **LocalMachine**/**My** pointe vers l’emplacement par défaut des certificats, le répertoire */var/lib/sfcerts*. Pour Linux, toutes les autres combinaisons de **CertificateStoreLocation** et **CertificateStoreName** sont non définies. 

Indiquez toujours **LocalMachine** pour le paramètre **CertificateStoreLocation**. Il est inutile de spécifier le paramètre **CertificateStoreName**, car il a pour valeur par défaut « My ». Avec une référence **X509**, les fichiers de certificat doivent être situés dans le répertoire */var/lib/sfcerts* sur le nœud de cluster.  

Le code XML suivant affiche une section **TransportSettings** basée sur ce style :

```xml
<Section Name="HelloWorldStatefulTransportSettings">
    <Parameter Name="MaxMessageSize" Value="10000000" />
    <Parameter Name="SecurityCredentialsType" Value="X509" />
    <Parameter Name="CertificateFindType" Value="FindByThumbprint" />
    <Parameter Name="CertificateFindValue" Value="4FEF3950642138446CC364A396E1E881DB76B48C" />
    <Parameter Name="CertificateRemoteThumbprints" Value="9FEF3950642138446CC364A396E1E881DB76B483" />
    <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
    <Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
    <Parameter Name="CertificateRemoteCommonNames" Value="ServiceFabric-Test-Cert" />
</Section>
```

### <a name="using-x509_2-securitycredentialstype"></a>Utilisation du type SecurityCredentialsType X509_2

Avec le kit SDK Java, vous pouvez spécifier **X509_2** pour **SecurityCredentialsType**. Cela correspond au type `X509Credentials2` ([Java](/java/api/system.fabric.x509credentials2)) de `SecurityCredentials` ([Java](/java/api/system.fabric.securitycredentials)). 

Avec une référence **X509_2**, vous spécifiez un paramètre de chemin et vous pouvez donc localiser le certificat dans un répertoire autre que */var/lib/sfcerts*.  Le code XML suivant affiche les paramètres utilisés pour spécifier l’emplacement du certificat : 

```xml
     <Parameter Name="SecurityCredentialsType" Value="X509_2" />
     <Parameter Name="CertificatePath" Value="/path/to/cert/BD1C71E248B8C6834C151174DECDBDC02DE1D954.crt" />
```

Le code XML suivant affiche une section **TransportSettings** basée sur ce style.

```xml
<!--Section name should always end with "TransportSettings".-->
<!--Here we are using a prefix "HelloWorldStateless".-->
<Section Name="HelloWorldStatelessTransportSettings">
    <Parameter Name="MaxMessageSize" Value="10000000" />
    <Parameter Name="SecurityCredentialsType" Value="X509_2" />
    <Parameter Name="CertificatePath" Value="/path/to/cert/BD1C71E248B8C6834C151174DECDBDC02DE1D954.crt" />
    <Parameter Name="CertificateProtectionLevel" Value="EncryptandSign" />
    <Parameter Name="CertificateRemoteThumbprints" Value="BD1C71E248B8C6834C151174DECDBDC02DE1D954" />
</Section>
```

> [!NOTE]
> Le certificat est spécifié comme un fichier .crt dans le code XML précédent. Cela implique qu’il existe également un fichier .key contenant la clé privée au même emplacement.

## <a name="configure-a-reliable-services-app-to-run-on-linux-clusters"></a>Configurer une application Reliable Services à exécuter sur les clusters Linux

Les kits SDK Service Fabric vous permettent de communiquer avec les API de runtime Service Fabric pour tirer parti de la plateforme. Quand vous exécutez une application qui utilise cette fonctionnalité sur des clusters Linux sécurisés, vous devez configurer votre application avec un certificat qui permet la validation auprès du runtime Service Fabric. Les applications qui contiennent des services Reliable Services Service Fabric écrits avec les kits SDK Java ou .NET Core nécessitent cette configuration. 

Pour configurer une application, ajoutez un élément [**SecretsCertificate**](./service-fabric-service-model-schema-elements.md#secretscertificate-element) sous la balise **Certificates**, qui est située sous la balise **ApplicationManifest** dans le fichier *ApplicationManifest.xml*. Le code XML suivant affiche un certificat référencé par son empreinte : 

```xml
   <Certificates>
       <SecretsCertificate X509FindType="FindByThumbprint" X509FindValue="0A00AA0AAAA0AAA00A000000A0AA00A0AAAA00" />
   </Certificates>   
```

Vous pouvez référencer le certificat de cluster ou un certificat que vous installez sur chaque nœud de cluster. Sur Linux, les fichiers de certificat doivent être présents dans le répertoire */var/lib/sfcerts*. Pour en savoir plus, consultez [Emplacement et format des certificats X.509 sur les nœuds Linux](#location-and-format-of-x509-certificates-on-linux-nodes).
