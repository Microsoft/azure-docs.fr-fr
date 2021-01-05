---
title: Démarrage rapide - WildFly sur CentOS
description: Déployer des applications Java sur WildFly sur une machine virtuelle CentOS
author: Theresa-Nguyen
ms.author: bicnguy
ms.topic: quickstart
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.assetid: 7aa21ef8-9cfb-43e0-bfda-3f10a2a2f3ef
ms.date: 10/23/2020
ms.openlocfilehash: 1140f765b21e68e206142cf341649af061ebdd33
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97705300"
---
# <a name="quickstart-wildfly-on-centos-8"></a>Démarrage rapide : WildFly on CentOS 8

Ce guide de démarrage rapide vous montre comment déployer le nœud autonome de WildFly d’une machine virtuelle CentOS 8. Il est idéal pour le développement et le test d’applications Java d’entreprise sur Azure. L’abonnement au serveur d’applications n’est pas nécessaire pour déployer ce guide de démarrage rapide.

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. Si vous n’avez pas d’abonnement Azure, vous pouvez activer vos [avantages abonné MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) ou [créer gratuitement un compte](https://azure.microsoft.com/pricing/free-trial).

## <a name="use-case"></a>Cas d’utilisation

WildFly est idéal pour le développement et le test d’applications Java d’entreprise sur Azure. Les listes des technologies disponibles dans les profils de configuration de serveur WildFly 18 sont disponibles dans le [Guide de prise en main de WildFly](https://docs.wildfly.org/18/Getting_Started_Guide.html#getting-started-with-wildfly).

Vous pouvez utiliser WildFly en mode autonome ou en mode cluster, selon votre cas d’utilisation. Vous pouvez garantir la haute disponibilité des applications Jakarta EE critiques par WildFly sur un cluster de nœuds. Apportez un petit nombre de modifications à la configuration de l’application, puis déployez l’application dans le cluster. Pour plus d’informations à ce sujet, consultez le [Guide de la haute disponibilité de WildFly](https://docs.wildfly.org/18/High_Availability_Guide.html).

## <a name="configuration-choice"></a>Choix de la configuration

WildFly peut être démarré en mode **serveur autonome** : une instance de serveur autonome est un processus indépendant, à l’instar d’une instance JBoss Application Server (AS) 3, 4, 5 ou 6. Les instances autonomes peuvent être lancées via des scripts de lancement standalone.sh ou standalone.bat. Pour plusieurs instances autonomes, il incombe à l’utilisateur de coordonner la gestion multiserveur entre les serveurs.

Vous pouvez aussi démarrer l’instance WildFly avec une autre configuration en utilisant des fichiers de configuration disponibles dans le dossier configuration.

Voici les fichiers de configuration du serveur autonome :

- standalone.xml (par défaut) : cette configuration est le fichier par défaut utilisé pour démarrer l’instance WildFly. Il contient la configuration certifiée de Jakarta Web Profile avec les technologies nécessaires.
   
- standalone-ha.xml : configuration certifiée de Jakarta EE Web Profile 8 avec haute disponibilité (ciblée sur les applications web).
   
- standalone-full.xml : configuration certifiée de Jakarta EE Plaform 8 incluant toutes les technologies nécessaires pour l’hébergement d’applications Jakarta EE.

- standalone-full-ha.xml : configuration certifiée de Jakarta EE Plaform 8 incluant avec la haute disponibilité pour l’hébergement d’applications Jakarta EE.

Pour démarrer votre serveur WildFly autonome avec une autre configuration fournie, utilisez l’argument --server-config avec le fichier server-config.

Par exemple, pour utiliser Jakarta EE Platform 8 avec les fonctionnalités de clustering, utilisez la commande suivante :

```
./standalone.sh --server-config=standalone-full-ha.xml
```

Pour plus d’informations sur les configurations, consultez le [Guide de prise en main de WildFly](https://docs.wildfly.org/18/Getting_Started_Guide.html#wildfly-10-configurations).

## <a name="licensing-support-and-subscription-notes"></a>Remarques sur les licences, le support et les abonnements

L’image Azure CentOS 8 est une image de machine virtuelle avec paiement à l’utilisation (PAYG) et ne demande pas à l’utilisateur d’obtenir une licence. La première fois que la machine virtuelle est lancée, la licence du système d’exploitation de la machine virtuelle est automatiquement activée et facturée à un tarif horaire. Ceci s’ajoute aux tarifs horaires des machines virtuelles Linux de Microsoft. Pour plus d’informations, cliquez sur [Tarification des machines virtuelles Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/#linux). Le téléchargement et l’utilisation de WildFly sont gratuits et ne nécessitent pas d’abonnement ou de licence Red Hat.

## <a name="how-to-consume"></a>Mode d’utilisation

Vous pouvez déployer le modèle des trois façons suivantes :

- Utiliser PowerShell - Déployez le modèle en exécutant les commandes suivantes : (Pour plus d’informations sur l’installation et la configuration d’Azure PowerShell, consultez [Azure PowerShell](/powershell/azure/).)

    ```
    New-AzResourceGroup -Name <resource-group-name> -Location <resource-group-location> #use this command when you need to create a new Resource Group for your deployment
    ```

    ```
    New-AzResourceGroupDeployment -ResourceGroupName <resource-group-name> -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/wildfly-standalone-centos8/azuredeploy.json
    ```
    
- Utiliser Azure CLI - Déployez le modèle en exécutant les commandes suivantes : (Pour plus d’informations sur l’installation et la configuration de l’interface de ligne de commande multiplateforme d’Azure, consultez [Ligne de commande multiplateforme d’Azure](/cli/azure/install-azure-cli).)

    ```
    az group create --name <resource-group-name> --location <resource-group-location> #use this command when you need to create a new Resource Group for your deployment
    ```

    ```
    az deployment group create --resource-group <my-resource-group> --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/wildfly-standalone-centos8/azuredeploy.json
    ```

- Utiliser le portail Azure - Déployez le modèle en cliquant <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fwildfly-standalone-centos8%2Fazuredeploy.json" target="_blank">ici</a> et connectez-vous à votre portail Azure.

## <a name="arm-template"></a>Modèle ARM

<a href="https://github.com/Azure/azure-quickstart-templates/tree/master/wildfly-standalone-centos8" target="_blank"> WildFly 18 sur CentOS 8 (machine virtuelle autonome)</a> - Il s’agit d’un modèle de démarrage rapide qui crée un nœud autonome de WildFly 18.0.1.Final sur une machine virtuelle CentOS 8 dans votre groupe de ressources, qui comprend une adresse IP privée pour la machine virtuelle, le réseau virtuel et le compte de stockage des diagnostics. Il déploie également un exemple d’application Java nommé JBoss-EAP sur Azure sur WildFly.

## <a name="resource-links"></a>Liens de ressources

* En savoir plus sur [WildFly 18](https://docs.wildfly.org/18/)
* En savoir plus sur les [distributions Linux sur Azure](../../linux/endorsed-distros.md)
* [Documentation Azure pour les développeurs Java](https://github.com/JasonFreeberg/jboss-on-app-service)

## <a name="next-steps"></a>Étapes suivantes

Pour un environnement de production, consultez les modèles ARM du guide de démarrage rapide pour Red Hat JBoss EAP Azure :

Machine virtuelle RHEL autonome avec un exemple d’application :

*  <a href="https://github.com/Azure/azure-quickstart-templates/tree/master/jboss-eap-standalone-rhel" target="_blank"> JBoss EAP sur RHEL (machine virtuelle autonome)</a>

Machines virtuelles RHEL en cluster avec un exemple d’application :

* <a href="https://github.com/Azure/azure-quickstart-templates/tree/master/jboss-eap-clustered-multivm-rhel" target="_blank"> JBoss EAP sur RHEL (machines virtuelles en cluster)</a>

Groupe de machines virtuelles identiques RHEL clusterisées avec un exemple d’application :

* <a href="https://github.com/Azure/azure-quickstart-templates/tree/master/jboss-eap-clustered-vmss-rhel" target="_blank"> JBoss EAP sur RHEL (groupe de machines virtuelles identiques clusterisées)</a>