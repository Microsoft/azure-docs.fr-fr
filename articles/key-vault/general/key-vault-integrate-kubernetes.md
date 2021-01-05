---
title: Intégrer Azure Key Vault à Kubernetes
description: Dans ce tutoriel, vous allez accéder à des secrets dans votre coffre de clés Azure et les récupérer en utilisant le pilote CSI (Container Storage Interface) du magasin des secrets pour les monter dans des pods Kubernetes.
author: ShaneBala-keyvault
ms.author: sudbalas
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 09/25/2020
ms.openlocfilehash: c628ba780ae64fceb32322fdb2004d69e2ebf24b
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452751"
---
# <a name="tutorial-configure-and-run-the-azure-key-vault-provider-for-the-secrets-store-csi-driver-on-kubernetes"></a>Tutoriel : Configurer et exécuter le fournisseur Azure Key Vault pour le pilote CSI du magasin des secrets sur Kubernetes

> [!IMPORTANT]
> Le pilote CSI constitue un projet open source qui n’est pas pris charge par le support technique Azure. Signalez tous les commentaires et problèmes liés à l’intégration Key Vault du pilote CSI sur le lien GitHub situé en bas de la page. Cet outil est fourni pour permettre aux utilisateurs d’effectuer une auto-installation dans des clusters et recueillir les commentaires de notre communauté.

Dans ce tutoriel, vous allez accéder à des secrets dans votre coffre de clés Azure et les récupérer en utilisant le pilote CSI (Container Storage Interface) du magasin des secrets pour monter ces secrets dans des pods Kubernetes.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un principal de service ou utiliser des identités managées.
> * Déployer un cluster AKS (Azure Kubernetes Service) à l’aide d’Azure CLI.
> * Installer Helm et le pilote CSI du magasin des secrets.
> * Créer un coffre de clés Azure et définir vos secrets.
> * Créer votre propre objet SecretProviderClass.
> * Affecter votre principal de service ou utiliser des identités managées.
> * Déployer votre pod avec les secrets montés de votre coffre de clés.

## <a name="prerequisites"></a>Prérequis

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

* Avant de commencer ce tutoriel, installez l’[interface de ligne de commande Azure](/cli/azure/install-azure-cli-windows?view=azure-cli-latest).

## <a name="create-a-service-principal-or-use-managed-identities"></a>Créer un principal du service ou utiliser des identités managées

Si vous envisagez d’utiliser des identités managées, vous pouvez passer à la section suivante.

Créez un principal de service pour contrôler les ressources qui sont accessibles à partir de votre coffre de clés Azure. L’accès de ce principal du service est limité par les rôles qui lui sont attribués. Cette fonctionnalité vous permet de contrôler la façon dont le principal du service peut gérer vos secrets. Dans l’exemple suivant, le nom du principal de service est *contosoServicePrincipal*.

```azurecli
az ad sp create-for-rbac --name contosoServicePrincipal --skip-assignment
```
Cette opération retourne une série de paires clé/valeur :

![Capture d’écran montrant les éléments appId et password pour contosoServicePrincipal](../media/kubernetes-key-vault-1.png)

Copiez les informations d’identification **appId** et **password** pour une utilisation ultérieure.

## <a name="flow-for-using-managed-identity"></a>Flux pour l’utilisation d’une identité managée

Ce diagramme montre le flux d’intégration AKS – Key Vault pour l’identité managée :

![Diagramme montrant le flux d’intégration AKS – Key Vault pour l’identité managée](../media/aks-key-vault-integration-flow.png)

## <a name="deploy-an-azure-kubernetes-service-aks-cluster-by-using-the-azure-cli"></a>Déployer un cluster AKS (Azure Kubernetes Service) à l’aide d’Azure CLI

Vous n’avez pas besoin d’utiliser Azure Cloud Shell. Votre invite de commandes (terminal) couplée à l’interface installée Azure CLI suffira. 

Suivez les instructions des sections « Créer un groupe de ressources », « Créer un cluster AKS » et « Se connecter au cluster » dans l’article [Déployer un cluster Azure Kubernetes Service à l’aide d’Azure CLI](../../aks/kubernetes-walkthrough.md). 

> [!NOTE] 
> Si vous prévoyez d’utiliser une identité de pod plutôt qu’un principal de service, veillez à l’activer au moment de créer le cluster Kubernetes, comme dans la commande suivante :
>
> ```azurecli
> az aks create -n contosoAKSCluster -g contosoResourceGroup --kubernetes-version 1.16.9 --node-count 1 --enable-managed-identity
> ```

1. [Définissez votre variable d’environnement PATH](https://www.java.com/en/download/help/path.xml) sur le fichier *kubectl.exe* que vous avez téléchargé.
1. Vérifiez votre version de Kubernetes à l’aide de la commande suivante, qui indique la version du client et du serveur. La version du client est le fichier *kubectl.exe* que vous avez installé, et la version du serveur est l’environnement AKS (Azure Kubernetes Service) sur lequel votre cluster s’exécute.
    ```azurecli
    kubectl version
    ```
1. Vérifiez que votre version de Kubernetes est la version 1.1.16.0 ou ultérieure. Pour les clusters Windows, vérifiez que votre version de Kubernetes est 1.18.0 ou une version ultérieure. La commande suivante met à niveau le cluster Kubernetes et le pool de nœuds. L’exécution de la commande peut prendre quelques minutes. Dans cet exemple, le groupe de ressources est *contosoResourceGroup* et le cluster Kubernetes est *contosoAKSCluster*.
    ```azurecli
    az aks upgrade --kubernetes-version 1.16.9 --name contosoAKSCluster --resource-group contosoResourceGroup
    ```
1. Pour afficher les métadonnées du cluster AKS que vous avez créé, utilisez la commande suivante. Copiez les valeurs de **principalId**, **clientId**, **subscriptionId** et **nodeResourceGroup** pour une utilisation ultérieure. Si le cluster AKS n’a pas été créé avec les identités managées activées, les paramètres **principalId** et **clientId** sont Null. 

    ```azurecli
    az aks show --name contosoAKSCluster --resource-group contosoResourceGroup
    ```

    Dans la sortie, deux paramètres sont mis en évidence :
    
    ![Capture d’écran d’Azure CLI avec les valeurs de principalId et clientId mises en évidence](../media/kubernetes-key-vault-2.png) ![Capture d’écran d’Azure CLI avec les valeurs de subscriptionId et nodeResourceGroup mises en évidence](../media/kubernetes-key-vault-3.png)
    
## <a name="install-helm-and-the-secrets-store-csi-driver"></a>Installer Helm et le pilote CSI du magasin des secrets

Pour installer le pilote CSI du magasin des secrets, vous devez d’abord installer [Helm](https://helm.sh/docs/intro/install/).

Avec l’interface du pilote [CSI du magasin des secrets](https://github.com/Azure/secrets-store-csi-driver-provider-azure/blob/master/charts/csi-secrets-store-provider-azure/README.md), vous pouvez obtenir les secrets qui sont stockés dans votre instance de coffre de clés Azure et utiliser ensuite l’interface du pilote pour monter le contenu des secrets dans des pods Kubernetes.

1. Vérifiez que la version de Helm est v3 ou ultérieure :
    ```azurecli
    helm version
    ```
1. Installez le pilote CSI du magasin des secrets et le fournisseur Azure Key Vault pour le pilote :
    ```azurecli
    helm repo add csi-secrets-store-provider-azure https://raw.githubusercontent.com/Azure/secrets-store-csi-driver-provider-azure/master/charts

    helm install csi-secrets-store-provider-azure/csi-secrets-store-provider-azure --generate-name
    ```

## <a name="create-an-azure-key-vault-and-set-your-secrets"></a>Créer un coffre de clés Azure et définir vos secrets

Pour créer votre propre coffre de clés et définir vos secrets, suivez les instructions fournies dans [Définir et récupérer un secret depuis Azure Key Vault à l’aide d’Azure CLI](../secrets/quick-create-cli.md).

> [!NOTE] 
> Vous n’avez pas besoin d’utiliser Azure Cloud Shell ni de créer un groupe de ressources. Vous pouvez utiliser le groupe de ressources que vous avez créé précédemment pour le cluster Kubernetes.

## <a name="create-your-own-secretproviderclass-object"></a>Créer votre propre objet SecretProviderClass

Pour créer votre propre objet SecretProviderClass personnalisé avec des paramètres propres au fournisseur pour le pilote CSI du magasin des secrets, [utilisez ce modèle](https://github.com/Azure/secrets-store-csi-driver-provider-azure/blob/master/examples/v1alpha1_secretproviderclass_service_principal.yaml). Cet objet permet à l’identité d’accéder à votre coffre de clés.

Dans l’exemple de fichier YAML SecretProviderClass, spécifiez les paramètres manquants. Les paramètres suivants sont requis :

* **userAssignedIdentityID** : # [OBLIGATOIRE] Si vous employez un principal du service, servez-vous de l’ID client pour désigner l’identité managée affectée par l’utilisateur à utiliser. Si vous utilisez une identité affectée par l’utilisateur comme identité managée de la machine virtuelle, spécifiez l’ID client de l’identité. Si la valeur est vide, elle utilise par défaut l’identité affectée par le système sur la machine virtuelle 
* **keyvaultName** : nom de votre coffre de clés
* **objects** : conteneur de l’ensemble du contenu de secret à monter
    * **objectName** : nom du contenu de secret
    * **objectType** : type d’objet (secret, clé, certificat)
* **resourceGroup** : Nom du groupe de ressources # [OBLIGATOIRE pour toute version < 0.0.4]. Groupe de ressources du coffre de clés
* **subscriptionId** : ID d’abonnement de votre coffre de clés # [OBLIGATOIRE pour toute version < 0.0.4]. ID d’abonnement du coffre de clés
* **tenantID** : ID de locataire ou ID de répertoire de votre coffre de clés

La documentation de tous les champs obligatoires est disponible ici : [Lien](https://github.com/Azure/secrets-store-csi-driver-provider-azure#create-a-new-azure-key-vault-resource-or-use-an-existing-one)

Le modèle mis à jour figure dans l’exemple suivant. Téléchargez-le sous forme de fichier YAML, puis complétez les champs obligatoires. Dans cet exemple, le coffre de clés est **contosoKeyVault5**. Il comporte deux secrets : **secret1** et **secret2**.

> [!NOTE] 
> Si vous utilisez des identités managées, définissez la valeur de **usePodIdentity** sur *true*, puis attribuez à **userAssignedIdentityID** une paire de guillemets droits ( **""** ) en guise de valeur. 

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1alpha1
kind: SecretProviderClass
metadata:
  name: azure-kvname
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"                   # [REQUIRED] Set to "true" if using managed identities
    useVMManagedIdentity: "false"             # [OPTIONAL] if not provided, will default to "false"
    userAssignedIdentityID: "servicePrincipalClientID"       # [REQUIRED] If you're using a service principal, use the client id to specify which user-assigned managed identity to use. If you're using a user-assigned identity as the VM's managed identity, specify the identity's client id. If the value is empty, it defaults to use the system-assigned identity on the VM
                                                             #     az ad sp show --id http://contosoServicePrincipal --query appId -o tsv
                                                             #     the preceding command will return the client ID of your service principal
    keyvaultName: "contosoKeyVault5"          # [REQUIRED] the name of the key vault
                                              #     az keyvault show --name contosoKeyVault5
                                              #     the preceding command will display the key vault metadata, which includes the subscription ID, resource group name, key vault 
    cloudName: ""                                # [OPTIONAL for Azure] if not provided, Azure environment will default to AzurePublicCloud
    objects:  |
      array:
        - |
          objectName: secret1                 # [REQUIRED] object name
                                              #     az keyvault secret list --vault-name "contosoKeyVault5"
                                              #     the above command will display a list of secret names from your key vault
          objectType: secret                  # [REQUIRED] object types: secret, key, or cert
          objectVersion: ""                   # [OPTIONAL] object versions, default to latest if empty
        - |
          objectName: secret2
          objectType: secret
          objectVersion: ""
    resourceGroup: "contosoResourceGroup"     # [REQUIRED] the resource group name of the key vault
    subscriptionId: "subscriptionID"          # [REQUIRED] the subscription ID of the key vault
    tenantId: "tenantID"                      # [REQUIRED] the tenant ID of the key vault
```
L’image suivante montre la sortie de la console pour **az keyvault show --name contosoKeyVault5** avec les métadonnées importantes mises en évidence :

![Capture d’écran montrant la sortie de la console pour « az keyvault show --name contosoKeyVault5 »](../media/kubernetes-key-vault-4.png)

## <a name="assign-your-service-principal-or-use-managed-identities"></a>Affecter votre principal du service ou utiliser des identités managées

### <a name="assign-a-service-principal"></a>Affecter un principal de service

Si vous utilisez un principal de service, accordez-lui des autorisations pour accéder à votre coffre de clés et récupérer les secrets. Attribuez le rôle *Lecteur* et accordez au principal du service des autorisations pour *obtenir* les secrets de votre coffre de clés à l’aide de la commande suivante :

1. Affectez votre principal de service à votre coffre de clés existant. Le paramètre **$AZURE_CLIENT_ID** correspond à la valeur de **appId** que vous avez copiée après avoir créé votre principal de service.
    ```azurecli
    az role assignment create --role Reader --assignee $AZURE_CLIENT_ID --scope /subscriptions/$SUBID/resourcegroups/$KEYVAULT_RESOURCE_GROUP/providers/Microsoft.KeyVault/vaults/$KEYVAULT_NAME
    ```

    La sortie de la commande est illustrée dans l’image suivante : 

    ![Capture d’écran montrant la valeur de principalId](../media/kubernetes-key-vault-5.png)

1. Accordez au principal de service des autorisations pour obtenir les secrets :
    ```azurecli
    az keyvault set-policy -n $KEYVAULT_NAME --secret-permissions get --spn $AZURE_CLIENT_ID
    az keyvault set-policy -n $KEYVAULT_NAME --key-permissions get --spn $AZURE_CLIENT_ID
    ```

1. Vous avez maintenant configuré votre principal de service avec les autorisations lui permettant de lire les secrets de votre coffre de clés. **$AZURE_CLIENT_SECRET** est le mot de passe de votre principal de service. Ajoutez les informations d’identification de votre principal de service sous la forme d’un secret Kubernetes accessible au pilote CSI du magasin des secrets :
    ```azurecli
    kubectl create secret generic secrets-store-creds --from-literal clientid=$AZURE_CLIENT_ID --from-literal clientsecret=$AZURE_CLIENT_SECRET
    ```

> [!NOTE] 
> Si vous déployez le pod Kubernetes et que vous obtenez une erreur indiquant que l’ID de secret client n’est pas valide, il est possible que vous disposiez d’un ID de secret client ancien qui a expiré ou a été réinitialisé. Pour résoudre ce problème, supprimez votre secret *secrets-store-creds* et créez-en un nouveau avec l’ID de secret client actuel. Pour supprimer votre secret *secrets-store-creds*, exécutez la commande ci-dessous :
>
> ```azurecli
> kubectl delete secrets secrets-store-creds
> ```

Si vous avez oublié l’ID de secret client de votre principal de service, vous pouvez le réinitialiser à l’aide de la commande suivante :

```azurecli
az ad sp credential reset --name contosoServicePrincipal --credential-description "APClientSecret" --query password -o tsv
```

### <a name="use-managed-identities"></a>Utiliser des identités managées

Si vous utilisez des identités managées, attribuez des rôles spécifiques au cluster AKS que vous avez créé. 

1. Pour créer, lister ou lire une identité managée affectée par l’utilisateur, le rôle [Opérateur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-operator) doit être affecté à votre cluster AKS. Vérifiez que **$clientId** correspond au clientId du cluster Kubernetes. Pour l’étendue, il se trouve sous votre service d’abonnement Azure, notamment le groupe de ressources de nœud mis en place lors de la création du cluster AKS. Cette étendue permet de s’assurer que seules les ressources de ce groupe sont affectées par les rôles attribués ci-dessous. 

    ```azurecli
    RESOURCE_GROUP=contosoResourceGroup
    az role assignment create --role "Managed Identity Operator" --assignee $clientId --scope /subscriptions/$SUBID/resourcegroups/$RESOURCE_GROUP
    
    az role assignment create --role "Managed Identity Operator" --assignee $clientId --scope /subscriptions/$SUBID/resourcegroups/$NODE_RESOURCE_GROUP
    
    az role assignment create --role "Virtual Machine Contributor" --assignee $clientId --scope /subscriptions/$SUBID/resourcegroups/$NODE_RESOURCE_GROUP
    ```

1. Installez l’identité Azure Active Directory (Azure AD) dans AKS.
    ```azurecli
    helm repo add aad-pod-identity https://raw.githubusercontent.com/Azure/aad-pod-identity/master/charts

    helm install pod-identity aad-pod-identity/aad-pod-identity
    ```

1. Créez une identité Azure AD. Dans la sortie, copiez la valeur de **clientId** et **principalId** pour une utilisation ultérieure.
    ```azurecli
    az identity create -g $resourceGroupName -n $identityName
    ```

1. Attribuez le rôle *Lecteur* à l’identité Azure AD que vous avez créée à l’étape précédente pour votre coffre de clés, puis accordez les autorisations d’identité pour obtenir les secrets de votre coffre de clés. Utilisez les valeurs de **clientId** et **principalId** de l’identité Azure AD.
    ```azurecli
    az role assignment create --role "Reader" --assignee $principalId --scope /subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/contosoResourceGroup/providers/Microsoft.KeyVault/vaults/contosoKeyVault5

    az keyvault set-policy -n contosoKeyVault5 --secret-permissions get --spn $clientId
    az keyvault set-policy -n contosoKeyVault5 --key-permissions get --spn $clientId
    ```

## <a name="deploy-your-pod-with-mounted-secrets-from-your-key-vault"></a>Déployer votre pod avec les secrets montés de votre coffre de clés

Pour configurer votre objet SecretProviderClass, exécutez la commande suivante :
```azurecli
kubectl apply -f secretProviderClass.yaml
```

### <a name="use-a-service-principal"></a>Utiliser un principal de service

Si vous utilisez un principal de service, utilisez la commande suivante pour déployer vos pods Kubernetes avec l’objet SecretProviderClass et le secret secrets-store-creds que vous avez configurés précédemment. Voici les modèles de déploiement :
* Pour [Linux](https://github.com/Azure/secrets-store-csi-driver-provider-azure/blob/master/examples/nginx-pod-inline-volume-service-principal.yaml)
* Pour [Windows](https://github.com/Azure/secrets-store-csi-driver-provider-azure/blob/master/examples/windows-pod-secrets-store-inline-volume-secret-providerclass.yaml)

```azurecli
kubectl apply -f updateDeployment.yaml
```

### <a name="use-managed-identities"></a>Utiliser des identités managées

Si vous utilisez des identités managées, créez un objet *AzureIdentity* dans votre cluster qui fait référence à l’identité que vous avez créée précédemment. Ensuite, créez un objet *AzureIdentityBinding* qui fait référence à l’objet AzureIdentity que vous avez créé. Spécifiez les paramètres du modèle suivant, puis enregistrez-le sous le nom *podIdentityAndBinding.yaml*.  

```yml
apiVersion: aadpodidentity.k8s.io/v1
kind: AzureIdentity
metadata:
    name: "azureIdentityName"               # The name of your Azure identity
spec:
    type: 0                                 # Set type: 0 for managed service identity
    resourceID: /subscriptions/<SUBID>/resourcegroups/<RESOURCEGROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<AZUREIDENTITYNAME>
    clientID: "managedIdentityClientId"     # The clientId of the Azure AD identity that you created earlier
---
apiVersion: aadpodidentity.k8s.io/v1
kind: AzureIdentityBinding
metadata:
    name: azure-pod-identity-binding
spec:
    azureIdentity: "azureIdentityName"      # The name of your Azure identity
    selector: azure-pod-identity-binding-selector
```
    
Exécutez la commande suivante pour exécuter la liaison :

```azurecli
kubectl apply -f podIdentityAndBinding.yaml
```

Ensuite, déployez le pod. Le code suivant est le fichier YAML de déploiement qui utilise la liaison d’identité du pod de l’étape précédente. Enregistrez ce fichier en tant que *podBindingDeployment.yaml*.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-secrets-store-inline
  labels:
    aadpodidbinding: azure-pod-identity-binding-selector
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - name: secrets-store-inline
          mountPath: "/mnt/secrets-store"
          readOnly: true
  volumes:
    - name: secrets-store-inline
      csi:
        driver: secrets-store.csi.k8s.io
        readOnly: true
        volumeAttributes:
          secretProviderClass: azure-kvname
        nodePublishSecretRef:           # Only required when using service principal mode
          name: secrets-store-creds     # Only required when using service principal mode
```

Exécutez la commande suivante pour déployer votre pod :

```azurecli
kubectl apply -f podBindingDeployment.yaml
```

### <a name="check-the-pod-status-and-secret-content"></a>Vérifier l’état du pod et le contenu du secret 

Pour afficher les pods que vous avez déployés, exécutez la commande suivante :
```azurecli
kubectl get pods
```

Pour vérifier l’état de votre pod, exécutez la commande suivante :
```azurecli
kubectl describe pod/nginx-secrets-store-inline
```

![Capture d’écran de la sortie Azure CLI indiquant l’état « Running » (En cours d’exécution) et présentant tous les événements comme étant de type « Normal » ](../media/kubernetes-key-vault-6.png)

Dans la fenêtre de sortie, le pod déployé doit être à l’état *Running* (En cours d’exécution). Dans la section **Events** au bas, tous les événements sont affichés comme étant de type *Normal*.

Après avoir vérifié que le pod est bien en cours d’exécution, vous pouvez vérifier qu’il contient les secrets de votre coffre de clés.

Pour afficher tous les secrets contenus dans le pod, exécutez la commande suivante :
```azurecli
kubectl exec -it nginx-secrets-store-inline -- ls /mnt/secrets-store/
```

Pour afficher le contenu d’un secret spécifique, exécutez la commande suivante :
```azurecli
kubectl exec -it nginx-secrets-store-inline -- cat /mnt/secrets-store/secret1
```

Vérifiez que le contenu du secret est affiché.

## <a name="next-steps"></a>Étapes suivantes

Pour vérifier que votre coffre de clés est récupérable, consultez :
> [!div class="nextstepaction"]
> [Activer la suppression réversible](./key-vault-recovery.md)