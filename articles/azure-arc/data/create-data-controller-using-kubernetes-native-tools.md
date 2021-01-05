---
title: Créer un contrôleur de données à l’aide des outils Kubernetes
description: Créer un contrôleur de données à l’aide des outils Kubernetes
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 051a7f506d351a17764e38c760ffba06d224cc38
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422567"
---
# <a name="create-azure-arc-data-controller-using-kubernetes-tools"></a>Créer un contrôleur de données Azure Arc à l’aide des outils Kubernetes

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Prérequis

Pour obtenir des informations générales, consultez la rubrique [Créer le contrôleur de données Azure Arc](create-data-controller.md).

Pour créer le contrôleur de données Azure Arc à l’aide des outils Kubernetes, vous devez avoir installé ceux-ci.  Les exemples de cet article utilisent `kubectl`, mais des approches similaires peuvent être utilisées avec d’autres outils Kubernetes tels que le tableau de bord Kubernetes, `oc`ou `helm` si vous êtes familiarisé avec ces outils et Kubernetes yaml/json.

[Installer l’outil kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

> [!NOTE]
> Certaines étapes de création du contrôleur de données Azure Arc indiqué ci-après nécessitent des autorisations d’administrateur de cluster Kubernetes.  Si vous n’êtes pas administrateur de cluster Kubernetes, vous devez faire en sorte que l’administrateur de cluster Kubernetes effectue ces étapes à votre place.

### <a name="cleanup-from-past-installations"></a>Nettoyage des installations précédentes

Si vous avez installé un contrôleur de données Azure Arc dans le passé, sur le même cluster et que vous l’avez supprimé à l’aide de la commande `azdata arc dc delete`, il se peut que des objets de niveau cluster doivent encore être supprimés. Exécutez les commandes suivantes pour supprimer les objets de niveau cluster du contrôleur de données Azure Arc :

```console
# Cleanup azure arc data service artifacts
kubectl delete crd datacontrollers.arcdata.microsoft.com 
kubectl delete sqlmanagedinstances.sql.arcdata.microsoft.com 
kubectl delete postgresql-11s.arcdata.microsoft.com 
kubectl delete postgresql-12s.arcdata.microsoft.com
kubectl delete clusterroles azure-arc-data:cr-arc-metricsdc-reader
kubectl delete clusterrolebindings azure-arc-data:crb-arc-metricsdc-reader
```

## <a name="overview"></a>Vue d’ensemble

La création du contrôleur de données Azure Arc présente les étapes générales suivantes :
1. Créez les définitions de ressource personnalisées pour le contrôleur de données Arc, l’instance gérée Azure SQL et PostgreSQL Hyperscale. **[Requiert des autorisations d’administrateur de cluster Kubernetes]**
2. Créez un espace de noms dans lequel le contrôleur de données sera créé. **[Requiert des autorisations d’administrateur de cluster Kubernetes]**
3. Créez le service du programme d’amorçage, y compris le jeu de réplicas, le compte de service, le rôle et la liaison de rôle.
4. Créez un secret pour le nom d’utilisateur et le mot de passe de l’administrateur du contrôleur de données.
5. Créez le contrôleur de données.
   
## <a name="create-the-custom-resource-definitions"></a>Créer les définitions de ressource personnalisées

Exécutez la commande suivante pour créer les définitions de ressources personnalisées.  **[Requiert des autorisations d’administrateur de cluster Kubernetes]**

```console
kubectl create -f https://raw.githubusercontent.com/microsoft/azure_arc/master/arc_data_services/deploy/yaml/custom-resource-definitions.yaml
```

## <a name="create-a-namespace-in-which-the-data-controller-will-be-created"></a>Créer un espace de noms dans lequel le contrôleur de données sera créé

Exécutez une commande semblable à la suivante pour créer un espace de noms dédié dans lequel le contrôleur de données sera créé.  Dans cet exemple et dans les autres exemples de cet article, un nom d’espace de noms de `arc` sera utilisé. Si vous choisissez d’utiliser un autre nom, utilisez le même nom partout.

```console
kubectl create namespace arc
```

Si d’autres personnes utilisant cet espace de noms ne sont pas des administrateurs de cluster, nous vous recommandons de créer un rôle d’administrateur d’espace de noms et d’accorder ce rôle à ces utilisateurs via une liaison de rôle.  L’administrateur de l’espace de noms doit avoir les autorisations complètes sur l’espace de noms.  De plus amples informations seront fournies ultérieurement sur la manière de proposer aux utilisateurs un accès plus granulaire en fonction du rôle.

## <a name="create-the-bootstrapper-service"></a>Créer le service du programme d’amorçage

Le service du programme d’amorçage gère les demandes entrantes pour la création, la modification et la suppression de ressources personnalisées telles qu’un contrôleur de données, une instance gérée SQL ou un groupe de serveurs PostgreSQL Hyperscale.

Exécutez la commande suivante pour créer un service de programme d’amorçage, un compte de service pour le service du programme d’amorçage, ainsi qu’un rôle et une liaison de rôle pour le compte de service du programme d’amorçage.

```console
kubectl create --namespace arc -f https://raw.githubusercontent.com/microsoft/azure_arc/master/arc_data_services/deploy/yaml/bootstrapper.yaml
```

Vérifiez que le pod du programme d’amorçage est en cours d’exécution à l’aide de la commande suivante.  Vous devrez peut-être l’exécuter plusieurs fois jusqu’à ce que l’état passe à `Running`.

```console
kubectl get pod --namespace arc
```

Le fichier de modèle bootstrapper.yaml est utilisé par défaut pour extraire l’image de conteneur du programme d’amorçage de Microsoft Container Registry (MCR).  Si votre environnement n’a pas accès directement à Microsoft Container Registry, vous pouvez effectuer les opérations suivantes :
- Suivez les étapes pour [extraire les images de conteneur de Microsoft Container Registry et les transmettre à un registre de conteneurs privé](offline-deployment.md).
- [Créez un secret d’extraction d’image](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-by-providing-credentials-on-the-command-lin) pour votre registre de conteneurs privé.
- Ajoutez une clé secrète d’extraction d’image au conteneur du programme d’amorçage. Voir l'exemple ci-dessous.
- Modifiez l’emplacement de l’image pour l’image du programme d’amorçage. Voir l'exemple ci-dessous.

L’exemple ci-dessous suppose que vous avez créé un nom de secret d’extraction d’image `regcred` comme indiqué dans la documentation Kubernetes.

```yaml
#just showing only the relevant part of the bootstrapper.yaml template file here
containers:
      - env:
        - name: ACCEPT_EULA
          value: "Y"
        #image: mcr.microsoft.com/arcdata/arc-bootstrapper:public-preview-oct-2020  <-- template value to change
        image: <your registry DNS name or IP address>/<your repo>/arc-bootstrapper:<your tag>
        imagePullPolicy: IfNotPresent
        name: bootstrapper
        resources: {}
        securityContext:
          runAsUser: 21006
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: regcred
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      serviceAccount: sa-mssql-controller
      serviceAccountName: sa-mssql-controller
      terminationGracePeriodSeconds: 30

```

## <a name="create-a-secret-for-the-data-controller-administrator"></a>Créer un secret pour l’administrateur du contrôleur de données

Le nom d’utilisateur et le mot de passe de l’administrateur du contrôleur de données permettent de s’authentifier auprès de l’API du contrôleur de données pour exécuter des fonctions d’administration.  Choisissez un mot de passe sécurisé et partagez-le uniquement avec ceux qui doivent disposer de privilèges d’administrateur de cluster.

Une clé secrète Kubernetes est stockée sous la forme d’une chaîne encodée en base64, une pour le nom d’utilisateur et l’autre pour le mot de passe.

Vous pouvez utiliser un outil en ligne pour coder en base64 le nom d’utilisateur et le mot de passe souhaités, ou vous pouvez utiliser des outils d’interface de ligne de commande intégrés en fonction de votre plateforme.

PowerShell

```console
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('<your string to encode here>'))

#Example
#[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('example'))

```

Linux/macOS

```console
echo '<your string to encode here>' | base64

#Example
# echo 'example' | base64
```

Une fois que vous avez encodé le nom d’utilisateur et le mot de passe, vous pouvez créer un fichier basé sur le [fichier modèle ](https://raw.githubusercontent.com/microsoft/azure_arc/master/arc_data_services/deploy/yaml/controller-login-secret.yaml) et remplacer les valeurs de nom d’utilisateur et de mot de passe par les vôtres.

Exécutez ensuite la commande ci-dessous pour créer la clé secrète :

```console
kubectl create --namespace arc -f <path to your data controller secret file>

#Example
kubectl create --namespace arc -f C:\arc-data-services\controller-login-secret.yaml
```

## <a name="create-the-data-controller"></a>Créer le contrôleur de données

Vous êtes maintenant prêt à créer le contrôleur de données lui-même.

Tout d'abord, créez une copie du [fichier modèle](https://raw.githubusercontent.com/microsoft/azure_arc/master/arc_data_services/deploy/yaml/data-controller.yaml) localement sur votre ordinateur afin de pouvoir modifier certains des paramètres.

Modifiez les éléments suivants selon les besoins :

**OBLIGATOIRE**
- **location**  : Modifiez-le pour qu’il corresponde à l’emplacement Azure dans lequel les _métadonnées_ relatives au contrôleur de données sont stockées.  Vous pouvez consulter la liste des emplacements Azure disponibles dans l’article [Vue d’ensemble de la création d’un contrôleur de données](create-data-controller.md).
- **resourceGroup**  : groupe de ressources Azure dans lequel vous souhaitez créer la ressource Azure Data Controller dans Azure Resource Manager.  En général, ce groupe de ressources doit déjà exister, mais il n’est pas nécessaire tant que vous n’avez pas chargé les données vers Azure.
- **subscription**  : GUID de l’abonnement Azure dans lequel vous souhaitez créer les ressources Azure.

**RECOMMANDÉ POUR LA RÉVISION ET ÉVENTUELLEMENT LA MODIFICATION DES VALEURS PAR DÉFAUT**
- **storage..className**  : classe de stockage à utiliser pour les fichiers de données et les fichiers journaux du contrôleur de données.  En cas de doute sur les classes de stockage disponibles dans votre cluster Kubernetes, vous pouvez exécuter la commande suivante : `kubectl get storageclass`.  La valeur par défaut est `default`, ce qui suppose qu'il existe une classe de stockage nommée `default` et non pas qu’une classe de stockage _est_ utilisée par défaut.  Remarque : Il existe deux paramètres className à définir pour la classe de stockage souhaitée, l’un pour les données et l’autre pour les journaux.
- **serviceType**  : Modifiez le type de service en `NodePort` si vous n’utilisez pas de programme d’équilibrage de charge.  Remarque : Deux paramètres serviceType doivent être modifiés.

**FACULTATIF**
- **nom**  : Le nom par défaut du contrôleur de données est `arc`, mais vous pouvez le modifier si vous le souhaitez.
- **displayName**  : Affectez-lui la même valeur que l’attribut Name en haut du fichier.
- **registry**  : Microsoft Container Registry est le registre par défaut.  Si vous extrayez les images de Microsoft Container Registry et [les transférez vers un registre de conteneurs privé](offline-deployment.md), entrez l’adresse IP ou le nom DNS de votre registre ici.
- **dockerRegistry**  : secret d’extraction d’image à utiliser pour extraire les images à partir d’un registre de conteneurs privé, si nécessaire.
- **repository**  : `arcdata` est le registre par défaut sur Microsoft Container Registry.  Si vous utilisez un registre de conteneurs privé, entrez le chemin d’accès au dossier/référentiel contenant les images de conteneur des services de données Azure Arc activées.
- **imageTag**  : l’étiquette de la dernière version actuelle est utilisée par défaut dans le modèle mais vous pouvez la modifier si vous souhaitez utiliser une version antérieure.

Exemple de fichier yaml de contrôleur de données terminé :
```yaml
apiVersion: arcdata.microsoft.com/v1alpha1
kind: datacontroller
metadata:
  generation: 1
  name: arc
spec:
  credentials:
    controllerAdmin: controller-login-secret
    #dockerRegistry: mssql-private-registry - optional if you are using a private container registry that requires authentication using an image pull secret
    serviceAccount: sa-mssql-controller
  docker:
    imagePullPolicy: Always
    imageTag: public-preview-oct-2020 
    registry: mcr.microsoft.com
    repository: arcdata
  security:
    allowDumps: true
    allowNodeMetricsCollection: true
    allowPodMetricsCollection: true
    allowRunAsRoot: false
  services:
  - name: controller
    port: 30080
    serviceType: LoadBalancer
  - name: serviceProxy
    port: 30777
    serviceType: LoadBalancer
  settings:
    ElasticSearch:
      vm.max_map_count: "-1"
    azure:
      connectionMode: Indirect
      location: eastus
      resourceGroup: myresourcegroup
      subscription: c82c901a-129a-435d-86e4-cc6b294590ae
    controller:
      displayName: arc
      enableBilling: "True"
      logs.rotation.days: "7"
      logs.rotation.size: "5000"
  storage:
    data:
      accessMode: ReadWriteOnce
      className: default
      size: 15Gi
    logs:
      accessMode: ReadWriteOnce
      className: default
      size: 10Gi
```

enregistrez le fichier modifié sur votre ordinateur local et exécutez la commande suivante pour créer le contrôleur de données :

```console
kubectl create --namespace arc -f <path to your data controller file>

#Example
kubectl create --namespace arc -f C:\arc-data-services\data-controller.yaml
```

## <a name="monitoring-the-creation-status"></a>Surveillance de l’état de la création

La création du contrôleur prend plusieurs minutes. Vous pouvez superviser la progression dans une autre fenêtre de terminal avec les commandes suivantes :

> [!NOTE]
>  Les exemples de commandes ci-dessous supposent que vous avez créé un contrôleur de données et un espace de noms Kubernetes avec le nom `arc`.  Si vous avez utilisé un autre nom pour le contrôleur de données/espace de noms, vous pouvez remplacer `arc` par ce nom.

```console
kubectl get datacontroller/arc --namespace arc
```

```console
kubectl get pods --namespace arc
```

Vous pouvez également vérifier l’état de la création de n’importe quel pod en exécutant une commande comme celle qui figure ci-dessous.  C’est particulièrement utile pour résoudre les problèmes.

```console
kubectl describe po/<pod name> --namespace arc

#Example:
#kubectl describe po/control-2g7bl --namespace arc
```

L’extension Azure Arc pour Azure Data Studio fournit un notebook pour vous guider au fil de l’installation de Kubernetes activé par Azure Arc et de sa configuration pour surveiller un dépôt git contenant un exemple de fichier yaml de SQL Managed Instance. Lorsque tout est connecté, un nouveau service Managed Instance SQL est déployé sur votre cluster Kubernetes.

Reportez-vous au notebook **Déployer un service SQL Managed Instance à l’aide de Kubernetes activé par Azure Arc et de Flux** dans l’extension Azure Arc pour Azure Data Studio.

## <a name="troubleshooting-creation-problems"></a>Résolution des problèmes de création

Si vous rencontrez des problèmes avec la création, consultez le [Guide de résolution des problèmes](troubleshoot-guide.md).

## <a name="next-steps"></a>Étapes suivantes

- [Créer une instance gérée SQL à l’aide des outils Kubernetes natifs](./create-sql-managed-instance-using-kubernetes-native-tools.md)
- [Créer un groupe de serveurs PostgreSQL Hyperscale à l’aide des outils Kubernetes natifs](./create-postgresql-hyperscale-server-group-kubernetes-native-tools.md)
