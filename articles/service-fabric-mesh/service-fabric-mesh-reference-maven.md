---
title: Informations de référence sur Maven Azure Service Fabric Mesh
description: Contient des informations de référence sur la façon d’utiliser le plug-in Maven pour Service Fabric Mesh
author: suhuruli
ms.author: suhuruli
ms.date: 11/26/2018
ms.topic: reference
ms.custom: devx-track-java
ms.openlocfilehash: 3a1aa004f47ba700ef4b96004dfe5b835788dcc7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87372465"
---
# <a name="maven-plugin-for-service-fabric-mesh"></a>Plug-in Maven pour Service Fabric Mesh

## <a name="prerequisites"></a>Conditions préalables requises

- Kit de développement logiciel (SDK) Java
- Maven
- Azure CLI avec extension Mesh
- CLI Service Fabric

## <a name="goals"></a>Objectifs

### `azure-sfmesh:init`
- Crée un dossier `servicefabric` qui contient un dossier `appresources` ayant le ficher `application.yaml`. 

### `azure-sfmesh:addservice`
- Crée un dossier dans un dossier `servicefabric` avec le nom du service et crée le fichier YAML du service. 

### `azure-sfmesh:addnetwork`
- Génère un fichier YAML `network` avec le nom de réseau fourni dans le dossier `appresources`. 

### `azure-sfmesh:addgateway`
- Génère un fichier YAML `gateway` avec le nom de passerelle fourni dans le dossier `appresources`. 

#### `azure-sfmesh:addvolume`
- Génère un fichier YAML `volume` avec le nom de volume fourni dans le dossier `appresources`.

### `azure-sfmesh:addsecret`
- Génère un fichier YAML `secret` avec le nom de secret fourni dans le dossier `appresources`. 

### `azure-sfmesh:addsecretvalue`
- Génère un fichier YAML `secretvalue` avec le nom de secret et de valeur de secret fourni dans le dossier `appresources`. 

### `azure-sfmesh:deploy`
- Fusionne les fichiers YAML à partir du dossier `servicefabric` et crée un JSON de modèle Azure Resource Manager dans le dossier actuel.
- Déploie toutes les ressources dans l’environnement Azure Service Fabric Mesh. 

### `azure-sfmesh:deploytocluster`
- Crée un dossier (`meshDeploy`) qui contient les fichiers JSON de déploiement générés à partir des fichiers YAML qui sont applicables aux clusters Service Fabric.
- Déploie toutes les ressources sur le cluster Service Fabric.
 

## <a name="usage"></a>Usage

Pour utiliser le plug-in Maven dans votre application Java Maven, ajoutez l’extrait de code suivant au fichier pom.xml :

```XML
<project>
  ...
  <build>
    ...
    <plugins>
      ...
      <plugin>
        <groupId>com.microsoft.azure</groupId>
          <artifactId>azure-sfmesh-maven-plugin</artifactId>
          <version>0.1.0</version>
      </plugin>
    </plugins>
  </build>
</project>
```

## <a name="common-configuration"></a>Configuration courante

Actuellement, le plug-in Maven ne prend pas en charge les configurations courantes des plug-ins Maven pour Azure.

## <a name="how-to"></a>Procédure

### <a name="initialize-maven-project-for-azure-service-fabric-mesh"></a>Initialiser le projet Maven pour Azure Service Fabric Mesh
Exécutez la commande suivante pour créer le fichier YAML de ressources d’application.

```cmd
mvn azure-sfmesh:init -DapplicationName=helloworldserver
```

- Crée un dossier appelé `servicefabric->appresources` dans votre dossier racine qui contient une application YAML appelée `app_helloworldserver`

### <a name="add-resource-to-your-application"></a>Ajouter une ressource à votre application

#### <a name="add-a-new-network-to-your-application"></a>Ajouter un nouveau réseau à votre application
Exécutez la commande ci-dessous pour créer un fichier YAML de ressources réseau. 

```cmd
mvn azure-sfmesh:addnetwork -DnetworkName=helloworldservicenetwork -DnetworkAddressPrefix=10.0.0.0/22
```

- Crée un fichier YAML de réseau dans le dossier `servicefabric->appresources` nommé `network_helloworldservicenetwork`

#### <a name="add-a-new-service-to-your-application"></a>Ajouter un nouveau service à votre application
Exécutez la commande ci-dessous pour créer un fichier YAML de service. 

```cmd
mvn azure-sfmesh:addservice -DapplicationName=helloworldserver -DserviceName=helloworldservice -DimageName=helloworldserver:latest -DlistenerPort=8080 -DnetworkRef=helloworldservicenetwork
```

- Crée un fichier YAML de service dans le dossier `servicefabric->helloworldservice` nommé `service_helloworldservice` qui fait référence à `helloworldservicenetwork` et à l’application `helloworldserver`
- Le service devrait normalement écouter sur le port 8080
- et utiliser ***helloworldserver:latest*** comme image conteneur.

#### <a name="add-a-new-gateway-resource-to-your-application"></a>Ajouter une nouvelle ressource de passerelle à votre application
Exécutez la commande ci-dessous pour créer un fichier YAML de ressources de passerelle. 

```cmd
mvn azure-sfmesh:addgateway -DapplicationName=helloworldserver -DdestinationNetwork=helloworldservicenetwork -DgatewayName=helloworldgateway -DlistenerName=helloworldserviceListener -DserviceName=helloworldservice -DsourceNetwork=open -DtcpPort=80
```

- Crée un fichier YAML de passerelle dans le dossier `servicefabric->appresources` nommé `gateway_helloworldgateway`.
- Référence `helloworldservicelistener` comme écouteur du service qui écoute les appels provenant de cette passerelle. Référence aussi `helloworldservice` comme service, `helloworldservicenetwork` comme réseau et `helloworldserver` comme application. 
- Écoute les demandes sur le port 80.

#### <a name="add-a-new-volume-to-your-application"></a>Ajouter un nouveau volume à votre application
Exécutez la commande ci-dessous pour créer un fichier YAML de ressource de volume. 

```cmd
mvn azure-sfmesh:addvolume -DvolumeAccountKey=key -DvolumeAccountName=name -DvolumeName=vol1 -DvolumeShareName=share
```

- Crée un fichier YAML de volume dans le dossier `servicefabric->appresources` nommé `volume_vol1`.
- Définit les propriétés des paramètres obligatoires, `volumeAccountKey` et `volumeShareName`, comme ci-dessus.
- Pour plus d’informations sur la façon de référencer ce volume créé, consultez [Déployer une application avec Azure Files Volume](service-fabric-mesh-howto-deploy-app-azurefiles-volume.md).

#### <a name="add-a-new-secret-resource-to-your-application"></a>Ajouter une nouvelle ressource de secret à votre application
Exécutez la commande ci-dessous pour créer un fichier YAML de ressource de secret. 

```cmd
mvn azure-sfmesh:addsecret -DsecretName=secret1
```

- Crée un fichier YAML de secret dans le dossier `servicefabric->appresources` nommé `secret_secret1`.
- Pour plus d’informations sur la façon de référencer ce secret créé, consultez [Gérer les secrets](service-fabric-mesh-howto-manage-secrets.md).

#### <a name="add-a-new-secretvalue-resource-to-your-application"></a>Ajouter une nouvelle ressource secretvalue à votre application
Exécutez la commande ci-dessous pour créer un fichier YAML de ressource secretvalue. 

```cmd
mvn azure-sfmesh:addsecretvalue -DsecretValue=someVal -DsecretValueName=secret1/v1
```

- Crée un fichier YAML de secretvalue dans le dossier `servicefabric->appresources` nommé `secretvalue_secret1_v1`.

### <a name="run-the-application-locally"></a>Exécuter l’application localement

À l’aide de l’objectif `azure-sfmesh:deploytocluster`, vous pouvez exécuter l’application localement à l’aide de la commande suivante :

```cmd
mvn azure-sfmesh:deploytocluster
```

Par défaut, cet objectif déploie des ressources sur un cluster local. Si vous déployez sur un cluster local, celui-ci suppose que vous avez un cluster Service Fabric local opérationnel. Le cluster Service Fabric local pour les ressources est actuellement pris en charge sur [Windows](service-fabric-mesh-howto-setup-developer-environment-sdk.md) uniquement.

- Crée des fichiers JSON à partir des fichiers YAML qui sont applicables aux clusters Service Fabric.
- Déploie ensuite sur le point de terminaison du cluster.

### <a name="deploy-application-to-azure-service-fabric-mesh"></a>Déployer une application dans Azure Service Fabric Mesh

À l’aide de l’objectif `azure-sfmesh:deploy`, vous pouvez déployer dans l’environnement Service Fabric Mesh en exécutant la commande ci-dessous :

```cmd
mvn azure-sfmesh:deploy -DresourceGroup=todoapprg -Dlocation=eastus
```

- Crée un groupe de ressources appelé `todoapprg` s’il n’existe pas.
- Crée un fichier JSON de modèle Azure Resource Manager en fusionnant les fichiers YAML. 
- Déploie le fichier JSON dans l’environnement Azure Service Fabric Mesh.
