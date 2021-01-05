---
title: Configurer l’interface de ligne de commande Azure Service Fabric Mesh
description: 'L’interface de ligne de commande Service Fabric Mesh est nécessaire pour déployer et gérer des ressources localement et dans Azure Service Fabric Mesh. Voici comment la configurer :'
author: georgewallace
ms.author: gwallace
ms.date: 11/28/2018
ms.topic: conceptual
ms.openlocfilehash: ea4a7764cf1ede1cfaf53b1097034c5894660376
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660676"
---
# <a name="set-up-service-fabric-mesh-cli"></a>Configurer l’interface de ligne de commande Service Fabric Mesh
L’interface de ligne de commande Service Fabric Mesh est nécessaire pour déployer et gérer des ressources localement et dans Azure Service Fabric Mesh. Voici comment la configurer :

Les trois types d’interfaces CLI qui peuvent être utilisés sont mentionnés dans le tableau ci-dessous.

| Module CLI | Environnement cible |  Description | 
|---|---|---|
| az mesh | Azure Service Fabric mesh | Interface CLI principale qui vous permet de déployer vos applications et de gérer des ressources dans l’environnement Azure Service Fabric Mesh. 
| sfctl | Clusters locaux | Interface CLI Service Fabric qui permet de déployer et de tester des ressources Service Fabric sur des clusters locaux.  
| Interface de ligne de commande Maven | Clusters locaux et Azure Service Fabric Mesh | Wrapper pour `az mesh` et `sfctl` qui permet aux développeurs Java d’utiliser une ligne de commande familière pour le développement local et Azure.  

Pour la préversion, Azure Service Fabric Mesh CLI est écrit en tant qu’extension à Azure CLI. Vous pouvez l’installer dans Azure Cloud Shell ou dans une instance locale d’Azure CLI. 

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.67 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="install-the-azure-service-fabric-mesh-cli"></a>Installer l’interface CLI Azure Service Fabric Mesh

Si vous ne l’avez pas déjà fait, installez le module d’extension CLI Azure Service Fabric Mesh à l’aide de la commande suivante : 
 
```azurecli-interactive
az extension add --name mesh
```

S’il est déjà installé, mettez à jour votre module CLI Azure Service Fabric Mesh à l’aide de la commande suivante :

```azurecli-interactive
az extension update --name mesh
```

## <a name="install-the-service-fabric-cli-sfctl"></a>Installer l’interface CLI Service Fabric (sfctl) 

Suivez les instructions fournies dans [Configurer l’interface CLI Service Fabric](../service-fabric/service-fabric-cli.md). Le module **sfctl** peut être utilisé pour déployer des applications basées sur un modèle de ressource sur des clusters Service Fabric de votre ordinateur local. 

## <a name="install-the-maven-cli"></a>Installer l’interface CLI Maven 

Pour utiliser l’interface CLI Maven, vous devez installer les éléments suivants sur votre ordinateur : 

* [Java](https://www.azul.com/downloads/zulu/)
* [Maven](https://maven.apache.org/download.cgi)
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* L’interface CLI Azure Mesh (az mesh) - Pour cibler Azure Service Fabric Mesh 
* SFCTL (sfctl) - Pour cibler les clusters locaux 

L’interface CLI de Maven pour Service Fabric est encore en préversion. 

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
          <configuration>
            ...
          </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

Pour plus d’informations sur son utilisation, lisez la section [Référence sur l’interface CLI Maven](service-fabric-mesh-reference-maven.md).

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également configurer votre [environnement de développement Windows](service-fabric-mesh-howto-setup-developer-environment-sdk.md).

Trouvez les réponses aux [questions et problèmes courants](service-fabric-mesh-faq.md).

[azure-cli-install]: /cli/azure/install-azure-cli
