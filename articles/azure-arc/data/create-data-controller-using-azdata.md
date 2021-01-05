---
title: Créer un contrôleur de données à l’aide de l’interface de ligne de commande Azure Data CLI (azdata)
description: Créez un contrôleur de données Azure Arc sur un cluster Kubernetes à nœuds multiples standard que vous avez déjà créé à l’aide d’Azure Data CLI (azdata).
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 0b4cf72622df78e13add723853d935fc97649b4a
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358994"
---
# <a name="create-azure-arc-data-controller-using-the-azure-data-cli-azdata"></a>Créer un contrôleur de données Azure Arc à l'aide de [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Prérequis

Pour obtenir des informations générales, consultez la rubrique [Créer le contrôleur de données Azure Arc](create-data-controller.md).

Pour créer le contrôleur de données Azure Arc à l'aide de [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)], [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] doit être installé.

   [Installer [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]](install-client-tools.md)

Quelle que soit la plateforme cible que vous choisissez, vous devez définir les variables d’environnement suivantes avant la création pour l’utilisateur administrateur du contrôleur de données. Vous pouvez fournir ces informations d’identification à d’autres personnes qui doivent disposer d’un accès administrateur au contrôleur de données si nécessaire.

**AZDATA_USERNAME** : nom d’utilisateur de votre choix pour l’utilisateur administrateur de contrôleur de données. Exemple : `arcadmin`

**AZDATA_PASSWORD** : mot de passe de votre choix pour l’utilisateur administrateur de contrôleur de données. Le mot de passe doit contenir au moins huit caractères appartenant à trois des quatre groupes suivants : lettres majuscules, lettres minuscules, chiffres et symboles.

### <a name="linux-or-macos"></a>Linux ou macOS

```console
export AZDATA_USERNAME="<your username of choice>"
export AZDATA_PASSWORD="<your password of choice>"
```

### <a name="windows-powershell"></a>Windows PowerShell

```console
$ENV:AZDATA_USERNAME="<your username of choice>"
$ENV:AZDATA_PASSWORD="<your password of choice>"
```

Vous devez vous connecter et vous authentifier auprès d’un cluster Kubernetes et avoir sélectionné un contexte Kubernetes existant avant de commencer la création du contrôleur de données Azure Arc. La façon dont vous vous connectez à un cluster ou un service Kubernetes varie. Consultez la documentation de la distribution ou du service Kubernetes que vous utilisez pour vous connecter au serveur de API de Kubernetes.

Vous pouvez vérifier que vous disposez d’une connexion Kubernetes actuelle et confirmer votre contexte actuel avec les commandes suivantes.

```console
kubectl get namespace
kubectl config current-context
```

### <a name="connectivity-modes"></a>Modes de connectivité

Comme décrit dans [Modes et exigences de connectivité](https://docs.microsoft.com/azure/azure-arc/data/connectivity), le contrôleur de données Azure Arc peut être déployé en mode de connectivité `direct` ou `indirect`. En mode de connectivité `direct`, les données d’utilisation sont envoyées automatiquement et en continu à Azure. Dans cet article, les exemples spécifient le mode de connectivité `direct` comme suit :

   ```console
   --connectivity-mode direct
   ```

   Pour créer le contrôleur en mode de connectivité `indirect`, mettez à jour les scripts dans l’exemple comme indiqué ci-dessous :

   ```console
   --connectivity-mode indirect
   ```

#### <a name="create-service-principal"></a>Créer un principal du service

Si vous déployez le contrôleur de données Azure Arc en mode de connectivité `direct`, les informations d’identification du principal de service sont nécessaires pour la connectivité Azure. Le principal du service permet de charger les données d’utilisation et des métriques. 

Suivez ces commandes pour créer le principal du service de chargement de vos métriques :

> [!NOTE]
> La création d’un principal de service nécessite [certaines autorisations dans Azure](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

Pour créer un principal de service, mettez à jour l’exemple de commande suivant. Remplacez `<ServicePrincipalName>` par le nom de votre principal de service et exécutez la commande :

```azurecli
az ad sp create-for-rbac --name <ServicePrincipalName>
``` 

Si vous avez créé le principal de service précédemment et que vous devez simplement récupérer les informations d’identification actuelles, exécutez la commande suivante pour réinitialiser les informations d’identification.

```azurecli
az ad sp credential reset --name <ServicePrincipalName>
```

Par exemple, pour créer un principal de service nommé `azure-arc-metrics`, exécutez la commande suivante

```console
az ad sp create-for-rbac --name azure-arc-metrics
```

Exemple de sortie :

```output
"appId": "2e72adbf-de57-4c25-b90d-2f73f126e123",
"displayName": "azure-arc-metrics",
"name": "http://azure-arc-metrics",
"password": "5039d676-23f9-416c-9534-3bd6afc78123",
"tenant": "72f988bf-85f1-41af-91ab-2d7cd01ad1234"
```

Enregistrez les valeurs `appId`, `password` et `tenant` dans une variable d’environnement pour une utilisation ultérieure. 

#### <a name="save-environment-variables-in-windows"></a>Enregistrer les variables d’environnement dans Windows

```console
SET SPN_CLIENT_ID=<appId>
SET SPN_CLIENT_SECRET=<password>
SET SPN_TENANT_ID=<tenant>
SET SPN_AUTHORITY=https://login.microsoftonline.com
```

#### <a name="save-environment-variables-in-linux-or-macos"></a>Enregistrer les variables d’environnement dans Linux ou macOS

```console
export SPN_CLIENT_ID='<appId>'
export SPN_CLIENT_SECRET='<password>'
export SPN_TENANT_ID='<tenant>'
export SPN_AUTHORITY='https://login.microsoftonline.com'
```

#### <a name="save-environment-variables-in-powershell"></a>Enregistrer les variables d’environnement dans PowerShell

```console
$Env:SPN_CLIENT_ID="<appId>"
$Env:SPN_CLIENT_SECRET="<password>"
$Env:SPN_TENANT_ID="<tenant>"
$Env:SPN_AUTHORITY="https://login.microsoftonline.com"
```

Une fois que vous avez créé le principal de service, affectez le principal de service au rôle approprié. 

### <a name="assign-roles-to-the-service-principal"></a>Affecter des rôles au principal de service

Exécutez cette commande pour affecter le principal de service au rôle `Monitoring Metrics Publisher` sur l’abonnement où se trouvent les ressources de votre instance de base de données :

#### <a name="run-the-command-on-windows"></a>Exécuter la commande sur Windows

> [!NOTE]
> Vous devez utiliser des guillemets doubles pour les noms de rôle lors de l’exécution à partir d’un environnement Windows.

```azurecli
az role assignment create --assignee <appId> --role "Monitoring Metrics Publisher" --scope subscriptions/<Subscription ID>
az role assignment create --assignee <appId> --role "Contributor" --scope subscriptions/<Subscription ID>
```

#### <a name="run-the-command-on-linux-or-macos"></a>Exécuter la commande sur Linux ou macOS.

```azurecli
az role assignment create --assignee <appId> --role 'Monitoring Metrics Publisher' --scope subscriptions/<Subscription ID>
az role assignment create --assignee <appId> --role 'Contributor' --scope subscriptions/<Subscription ID>
```

#### <a name="run-the-command-in-powershell"></a>Exécuter la commande dans PowerShell

```powershell
az role assignment create --assignee <appId> --role 'Monitoring Metrics Publisher' --scope subscriptions/<Subscription ID>
az role assignment create --assignee <appId> --role 'Contributor' --scope subscriptions/<Subscription ID>
```

```output
{
  "canDelegate": null,
  "id": "/subscriptions/<Subscription ID>/providers/Microsoft.Authorization/roleAssignments/f82b7dc6-17bd-4e78-93a1-3fb733b912d",
  "name": "f82b7dc6-17bd-4e78-93a1-3fb733b9d123",
  "principalId": "5901025f-0353-4e33-aeb1-d814dbc5d123",
  "principalType": "ServicePrincipal",
  "roleDefinitionId": "/subscriptions/<Subscription ID>/providers/Microsoft.Authorization/roleDefinitions/3913510d-42f4-4e42-8a64-420c39005123",
  "scope": "/subscriptions/<Subscription ID>",
  "type": "Microsoft.Authorization/roleAssignments"
}
```

Une fois le principal de service affecté au rôle approprié et les variables d’environnement définies, vous pouvez procéder à la création du contrôleur de données. 

## <a name="create-the-azure-arc-data-controller"></a>Créer le contrôleur de données Azure Arc

> [!NOTE]
> Vous pouvez utiliser une valeur différente pour le paramètre `--namespace` de la commande azdata arc dc create dans les exemples ci-dessous, mais veillez à utiliser ce nom d’espace de noms pour `--namespace parameter` dans toutes les autres commandes ci-dessous.

- [Créer Azure Kubernetes Service (AKS)](#create-on-azure-kubernetes-service-aks)
- [Créer sur le moteur AKS sur Azure Stack Hub](#create-on-aks-engine-on-azure-stack-hub)
- [Créer sur AKS sur Azure Stack HCI](#create-on-aks-on-azure-stack-hci)
- [Créer sur Azure Red Hat OpenShift (ARO)](#create-on-azure-red-hat-openshift-aro)
- [Créer sur Red Hat OpenShift Container Platform (OCP)](#create-on-red-hat-openshift-container-platform-ocp)
- [Créer sur open source, amont Kubernetes (kubeadm)](#create-on-open-source-upstream-kubernetes-kubeadm)
- [Créer sur AWS Elastic Kubernetes Service (EKS)](#create-on-aws-elastic-kubernetes-service-eks)
- [Créer sur le service Google Cloud Kubernetes Engine (GKE)](#create-on-google-cloud-kubernetes-engine-service-gke)

### <a name="create-on-azure-kubernetes-service-aks"></a>Créer Azure Kubernetes Service (AKS)

Par défaut, le profil de déploiement AKS utilise la classe de stockage `managed-premium`. La classe de stockage `managed-premium` ne fonctionnera que si vous avez des machines virtuelles déployées à l’aide d’images de machine virtuelle qui ont des disques Premium.

Si vous envisagez d’utiliser `managed-premium` comme classe de stockage, vous pouvez exécuter la commande suivante pour créer le contrôleur de données. Remplacez les espaces réservés dans la commande par le nom de votre groupe de ressources, votre ID d’abonnement et l’emplacement Azure.

```console
azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode direct

#Example:
#azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode direct
```

Si vous n’êtes pas sûr de la classe de stockage à utiliser, vous devez utiliser la classe de stockage `default` qui est prise en charge, quel que soit le type de machine virtuelle que vous utilisez. Elle n’offre cependant pas les performances les plus rapides.

Si vous souhaitez utiliser la classe de stockage `default`, vous pouvez alors exécuter cette commande :

```console
azdata arc dc create --profile-name azure-arc-aks-default-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode direct

#Example:
#azdata arc dc create --profile-name azure-arc-aks-default-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode direct
```

Une fois que vous avez exécuté la commande, passez à [Contrôle de l’état de la création](#monitoring-the-creation-status).

### <a name="create-on-aks-engine-on-azure-stack-hub"></a>Créer sur le moteur AKS sur Azure Stack Hub

Par défaut, le profil de déploiement utilise la classe de stockage `managed-premium`. La classe de stockage `managed-premium` ne fonctionnera que si vous avez des machines virtuelles de travail déployées à l’aide d’images de machine virtuelle qui ont des disques Premium sur Azure Stack Hub.

Vous pouvez exécuter la commande suivante pour créer le contrôleur de données au moyen de la classe de stockage managed-premium :

```console
azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode direct

#Example:
#azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode direct
```

Si vous n’êtes pas sûr de la classe de stockage à utiliser, vous devez utiliser la classe de stockage `default` qui est prise en charge, quel que soit le type de machine virtuelle que vous utilisez. Dans Azure Stack Hub, les disques Premium et les disques standard sont sauvegardés par la même infrastructure de stockage. Par conséquent, ils sont censés fournir les mêmes performances générales, mais avec des limites d’E/S par seconde différentes.

Si vous souhaitez utiliser la classe de stockage `default`, vous pouvez alors exécuter cette commande.

```console
azdata arc dc create --profile-name azure-arc-aks-default-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode direct

#Example:
#azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode direct
```

Une fois que vous avez exécuté la commande, passez à [Contrôle de l’état de la création](#monitoring-the-creation-status).

### <a name="create-on-aks-on-azure-stack-hci"></a>Créer sur AKS sur Azure Stack HCI

Par défaut, le profil de déploiement utilise une classe de stockage nommée `default` et le type de service `LoadBalancer`.

Vous pouvez exécuter la commande suivante pour créer le contrôleur de données au moyen de la classe de stockage `default` et le type de service `LoadBalancer`.

```console
azdata arc dc create --profile-name azure-arc-aks-hci --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode direct

#Example:
#azdata arc dc create --profile-name azure-arc-aks-hci --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode direct
```

Une fois que vous avez exécuté la commande, passez à [Contrôle de l’état de la création](#monitoring-the-creation-status).


### <a name="create-on-azure-red-hat-openshift-aro"></a>Créer sur Azure Red Hat OpenShift (ARO)

#### <a name="apply-the-scc"></a>Appliquer les contraintes de contexte de sécurité (SCC)

Avant de créer le contrôleur de données sur Azure Red Hat OpenShift, vous devez appliquer des contraintes de contexte de sécurité (SCC) spécifiques. Pour la préversion, ces dernières assouplissent les contraintes de sécurité. Les versions à venir fourniront des contraintes de contexte de sécurité mises à jour.

1. Téléchargez la contrainte de contexte de sécurité (SCC) personnalisée. Utilisez l’une des valeurs suivantes : 
   - [GitHub](https://github.com/microsoft/azure_arc/tree/master/arc_data_services/deploy/yaml/arc-data-scc.yaml) 
   - ([Brut](https://raw.githubusercontent.com/microsoft/azure_arc/master/arc_data_services/deploy/yaml/arc-data-scc.yaml))
   - La commande `curl` suivante télécharge arc-data-scc.yaml :

      ```console
      curl https://raw.githubusercontent.com/microsoft/azure_arc/master/arc_data_services/deploy/yaml/arc-data-scc.yaml -o arc-data-scc.yaml
      ```

1. Créez la contrainte de contexte de sécurité.

   ```console
   oc create -f arc-data-scc.yaml
   ```

1. Appliquez la contrainte de contexte de sécurité au compte de service.

   > [!NOTE]
   > Utilisez le même espace de noms ici et dans la commande `azdata arc dc create` ci-dessous. Par exemple, `arc`.

   ```console
   oc adm policy add-scc-to-user arc-data-scc --serviceaccount default --namespace arc
   ```


#### <a name="create-custom-deployment-profile"></a>Créer un profil de déploiement personnalisé

Utilisez le profil `azure-arc-azure-openshift` pour Azure RedHat Open Shift.

```console
azdata arc dc config init --source azure-arc-azure-openshift --path ./custom
```

#### <a name="create-data-controller"></a>Créer un contrôleur de données

Vous pouvez exécuter la commande suivante pour créer le contrôleur de données :

> [!NOTE]
> Utilisez le même espace de noms ici et dans la commande `oc adm policy add-scc-to-user` ci-dessus. Par exemple, `arc`.

```console
azdata arc dc create --profile-name azure-arc-azure-openshift --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode direct

#Example
#azdata arc dc create --profile-name azure-arc-azure-openshift --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode direct
```

Une fois que vous avez exécuté la commande, passez à [Contrôle de l’état de la création](#monitoring-the-creation-status).

### <a name="create-on-red-hat-openshift-container-platform-ocp"></a>Créer sur Red Hat OpenShift Container Platform (OCP)

> [!NOTE]
> Si vous utilisez Red Hat OpenShift Container Platform sur Azure, nous vous recommandons d'utiliser la dernière version disponible.

#### <a name="apply-the-scc"></a>Appliquer les contraintes de contexte de sécurité (SCC)

Avant de créer le contrôleur de données sur Red Hat OCP, vous devez appliquer des contraintes de contexte de sécurité (SCC) spécifiques. Pour la préversion, ces dernières assouplissent les contraintes de sécurité. Les versions à venir fourniront des contraintes de contexte de sécurité mises à jour.

1. Téléchargez la contrainte de contexte de sécurité (SCC) personnalisée. Utilisez l’une des valeurs suivantes : 
   - [GitHub](https://github.com/microsoft/azure_arc/tree/master/arc_data_services/deploy/yaml/arc-data-scc.yaml) 
   - ([Brut](https://raw.githubusercontent.com/microsoft/azure_arc/master/arc_data_services/deploy/yaml/arc-data-scc.yaml))
   - La commande `curl` suivante télécharge arc-data-scc.yaml :

      ```console
      curl https://raw.githubusercontent.com/microsoft/azure_arc/master/arc_data_services/deploy/yaml/arc-data-scc.yaml -o arc-data-scc.yaml
      ```

1. Créez la contrainte de contexte de sécurité.

   ```console
   oc create -f arc-data-scc.yaml
   ```

1. Appliquez la contrainte de contexte de sécurité au compte de service.

   > [!NOTE]
   > Utilisez le même espace de noms ici et dans la commande `azdata arc dc create` ci-dessous. Par exemple, `arc`.

   ```console
   oc adm policy add-scc-to-user arc-data-scc --serviceaccount default --namespace arc
   ```

#### <a name="determine-storage-class"></a>Déterminer la classe de stockage

Vous devrez également déterminer la classe de stockage à utiliser en exécutant la commande suivante.

```console
kubectl get storageclass
```

#### <a name="create-custom-deployment-profile"></a>Créer un profil de déploiement personnalisé

Créez un fichier de profil de déploiement personnalisé basé sur le profil de déploiement `azure-arc-openshift` en exécutant la commande suivante. Cette commande crée un répertoire `custom` dans votre répertoire de travail actuel et un fichier de profil de déploiement personnalisé `control.json` dans ce répertoire.

Utilisez le profil `azure-arc-openshift` pour OpenShift Container Platform.

```console
azdata arc dc config init --source azure-arc-openshift --path ./custom
```

#### <a name="set-storage-class"></a>Définir la classe de stockage 

À présent, définissez la classe de stockage souhaitée en remplaçant `<storageclassname>` dans la commande ci-dessous par le nom de la classe de stockage que vous souhaitez utiliser, qui a été déterminée en exécutant la commande `kubectl get storageclass` ci-dessus.

```console
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=<storageclassname>"
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=<storageclassname>"

#Example:
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=mystorageclass"
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=mystorageclass"
```

#### <a name="set-loadbalancer-optional"></a>Définir LoadBalancer (facultatif)

Par défaut, le profil de déploiement `azure-arc-openshift` utilise `NodePort` comme type de service. Si vous utilisez un cluster OpenShift intégré à un équilibreur de charge, vous pouvez changer la configuration pour utiliser le type de service `LoadBalancer` à l’aide de la commande suivante :

```console
azdata arc dc config replace --path ./custom/control.json --json-values "$.spec.services[*].serviceType=LoadBalancer"
```

#### <a name="verify-security-policies"></a>Vérifier les stratégies de sécurité

Quand vous utilisez OpenShift, vous voulez peut-être exécuter les stratégies de sécurité par défaut dans OpenShift ou verrouiller l’environnement plus que d’habitude. Vous pouvez choisir de désactiver certaines fonctionnalités afin de réduire les autorisations nécessaires au moment du déploiement et au moment de l’exécution en exécutant les commandes suivantes.

Cette commande désactive les collectes de métriques relatives aux pods. Si vous désactivez cette fonctionnalité, vous ne pourrez pas voir les métriques relatives aux pods dans les tableaux de bord Grafana. La valeur par défaut est true.

```console
azdata arc dc config replace -p ./custom/control.json --json-values spec.security.allowPodMetricsCollection=false
```

Cette commande désactive les collectes de métriques relatives aux nœuds. Si vous désactivez cette fonctionnalité, vous ne pourrez pas voir les métriques relatives aux pods dans les tableaux de bord Grafana. La valeur par défaut est true.

```console
azdata arc dc config replace --path ./custom/control.json --json-values spec.security.allowNodeMetricsCollection=false
```

Cette commande désactive la capacité à effectuer des images mémoire à des fins de dépannage.
```console
azdata arc dc config replace --path ./custom/control.json --json-values spec.security.allowDumps=false
```

#### <a name="create-data-controller"></a>Créer un contrôleur de données

Vous êtes maintenant prêt à créer le contrôleur de données à l’aide de la commande suivante.

> [!NOTE]
>   Utilisez le même espace de noms ici et dans la commande `oc adm policy add-scc-to-user` ci-dessus. Par exemple, `arc`.

> [!NOTE]
>   Le paramètre `--path` doit pointer vers le _répertoire_ contenant le fichier control.json et non vers le fichier control.json lui-même.


```console
azdata arc dc create --path ./custom --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode direct

#Example:
#azdata arc dc create --path ./custom --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode direct
```

Une fois que vous avez exécuté la commande, passez à [Contrôle de l’état de la création](#monitoring-the-creation-status).

### <a name="create-on-open-source-upstream-kubernetes-kubeadm"></a>Créer sur open source, amont Kubernetes (kubeadm)

Par défaut, le profil de déploiement kubeadm utilise une classe de stockage appelée `local-storage` et le type de service `NodePort`. Si cela est acceptable, vous pouvez ignorer les instructions ci-dessous qui définissent la classe de stockage et le type de service souhaités, puis exécuter immédiatement la commande `azdata arc dc create` ci-dessous.

Si vous souhaitez personnaliser votre profil de déploiement pour spécifier une classe de stockage et/ou un type de service spécifique, commencez par créer un fichier de profil de déploiement personnalisé basé sur le profil de déploiement kubeadm en exécutant la commande suivante. Cette commande va créer un répertoire `custom` dans votre répertoire de travail actuel et un fichier de profil de déploiement personnalisé `control.json` dans ce répertoire.

```console
azdata arc dc config init --source azure-arc-kubeadm --path ./custom
```

Vous pouvez rechercher les classes de stockage disponibles en exécutant la commande suivante.

```console
kubectl get storageclass
```

À présent, définissez la classe de stockage souhaitée en remplaçant `<storageclassname>` dans la commande ci-dessous par le nom de la classe de stockage que vous souhaitez utiliser, qui a été déterminée en exécutant la commande `kubectl get storageclass` ci-dessus.

```console
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=<storageclassname>"
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=<storageclassname>"

#Example:
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=mystorageclass"
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=mystorageclass"
```

Par défaut, le profil de déploiement kubeadm utilise `NodePort` comme type de service. Si vous utilisez un cluster Kubernetes intégré à un équilibreur de charge, vous pouvez modifier la configuration à l’aide de la commande suivante.

```console
azdata arc dc config replace --path ./custom/control.json --json-values "$.spec.services[*].serviceType=LoadBalancer"
```

Vous êtes maintenant prêt à créer le contrôleur de données à l’aide de la commande suivante.

```console
azdata arc dc create --path ./custom --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode direct

#Example:
#azdata arc dc create --path ./custom --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode direct
```

Une fois que vous avez exécuté la commande, passez à [Contrôle de l’état de la création](#monitoring-the-creation-status).

### <a name="create-on-aws-elastic-kubernetes-service-eks"></a>Créer sur AWS Elastic Kubernetes Service (EKS)

Par défaut, la classe de stockage EKS est `gp2` et le type de service est `LoadBalancer`.

Exécutez la commande suivante pour créer le contrôleur de données à l’aide du profil de déploiement EKS fourni.

```console
azdata arc dc create --profile-name azure-arc-eks --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode direct

#Example:
#azdata arc dc create --profile-name azure-arc-eks --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode direct
```

Une fois que vous avez exécuté la commande, passez à [Contrôle de l’état de la création](#monitoring-the-creation-status).

### <a name="create-on-google-cloud-kubernetes-engine-service-gke"></a>Créer sur le service Google Cloud Kubernetes Engine (GKE)

Par défaut, la classe de stockage GKE est `standard` et le type de service est `LoadBalancer`.

Exécutez la commande suivante pour créer le contrôleur de données à l’aide du profil de déploiement GKE fourni.

```console
azdata arc dc create --profile-name azure-arc-gke --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode direct

#Example:
#azdata arc dc create --profile-name azure-arc-gke --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode direct
```

Une fois que vous avez exécuté la commande, passez à [Contrôle de l’état de la création](#monitoring-the-creation-status).

## <a name="monitoring-the-creation-status"></a>Surveillance de l’état de la création

La création du contrôleur prend plusieurs minutes. Vous pouvez superviser la progression dans une autre fenêtre de terminal avec les commandes suivantes :

> [!NOTE]
>  Les exemples de commandes ci-dessous supposent que vous avez créé un contrôleur de données et un espace de noms Kubernetes avec le nom `arc`. Si vous avez utilisé un autre nom pour le contrôleur de données/espace de noms, vous pouvez remplacer `arc` par ce nom.

```console
kubectl get datacontroller/arc --namespace arc
```

```console
kubectl get pods --namespace arc
```

Vous pouvez également vérifier l’état de la création de n’importe quel pod en exécutant une commande comme celle qui figure ci-dessous. C’est particulièrement pratique pour résoudre les éventuels problèmes.

```console
kubectl describe po/<pod name> --namespace arc

#Example:
#kubectl describe po/control-2g7bl --namespace arc
```

## <a name="troubleshooting-creation-problems"></a>Résolution des problèmes de création

Si vous rencontrez des problèmes avec la création, consultez le [Guide de résolution des problèmes](troubleshoot-guide.md).
