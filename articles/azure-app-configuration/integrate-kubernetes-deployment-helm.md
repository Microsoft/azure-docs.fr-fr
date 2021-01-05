---
title: Intégrer Azure App Configuration avec le déploiement Kubernetes en utilisant Helm
description: Découvrez comment utiliser des configurations dynamiques dans le déploiement Kubernetes avec Helm.
services: azure-app-configuration
author: shenmuxiaosen
manager: zhenlan
ms.service: azure-app-configuration
ms.topic: tutorial
ms.date: 04/14/2020
ms.author: shuawan
ms.openlocfilehash: c388bd22ba20dd681997064496a90a81dabb292f
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92426715"
---
# <a name="integrate-with-kubernetes-deployment-using-helm"></a>Intégrer au déploiement Kubernetes en utilisant Helm

Helm fournit un moyen de définir, d’installer et de mettre à niveau des applications qui s’exécutent dans Kubernetes. Un chart Helm contient les informations nécessaires à la création d’une instance d’une application Kubernetes. La configuration est stockée en dehors du chart lui-même, dans un fichier nommé *values.yaml*. 

Pendant le processus de publication, Helm fusionne le chart avec la configuration appropriée pour exécuter l’application. Par exemple, les variables définies dans *values.yaml* peuvent être référencées en tant que variables d’environnement dans les conteneurs en cours d’exécution. Helm prend également en charge la création de secrets Kubernetes, qui peuvent être montés en tant que volumes de données ou exposés en tant que variables d’environnement.

Vous pouvez remplacer les valeurs stockées dans *values.yaml* en fournissant des fichiers de configuration YAML supplémentaires sur la ligne de commande lors de l’exécution de Helm. Azure App configuration prend en charge l’exportation des valeurs de configuration vers des fichiers YAML. L’intégration de cette fonctionnalité d’exportation à votre déploiement permet à vos applications Kubernetes de tirer parti des valeurs de configuration stockées dans App Configuration.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Utiliser des valeurs d’App Configuration lors du déploiement d’une application sur Kubernetes avec Helm.
> * Créer un secret Kubernetes basé sur une référence Key Vault dans App Configuration.

Ce tutoriel suppose une compréhension de base de la gestion de Kubernetes avec Helm. Découvrez plus d’informations sur l’installation d’applications avec Helm dans [Azure Kubernetes Service](../aks/kubernetes-helm.md).

## <a name="prerequisites"></a>Prérequis

- [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
- Installer [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest) (version 2.4.0 ou ultérieure)
- Installer [Helm](https://helm.sh/docs/intro/install/) (version 2.14.0 ou ultérieure)
- Un cluster Kubernetes.

## <a name="create-an-app-configuration-store"></a>Créer un magasin App Configuration

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

7. Sélectionnez **Explorateur de configurations** > **Créer** pour ajouter les paires clé-valeur suivantes :

    | Clé | Valeur |
    |---|---|
    | settings.color | White |
    | settings.message | Data from Azure App Configuration |

    Laissez **Étiquette** et **Type de contenu** vides pour l’instant.

## <a name="add-a-key-vault-reference-to-app-configuration"></a>Ajouter une référence Key Vault à App Configuration
1. Connectez-vous au [portail Azure](https://portal.azure.com) et ajoutez un secret à [Key Vault](../key-vault/secrets/quick-create-portal.md#add-a-secret-to-key-vault) avec comme nom **Password** et comme valeur **myPassword**. 
2. Sélectionnez l’instance de magasin App Configuration que vous avez créée dans la section précédente.

3. Sélectionnez **Explorateur de configuration**.

4. Sélectionnez **+ Créer** > **Référence Key Vault** , puis choisissez les valeurs suivantes :
    - **Clé**  : Sélectionnez **secrets.password**.
    - **Étiquette**  : Laissez cette valeur vide.
    - **Abonnement** , **Groupe de ressources** et **Key Vault** : Entrez les valeurs correspondant à ces éléments dans le coffre de clés que vous avez créé dans la section précédente.
    - **Secret**  : Sélectionnez le secret nommé **Password** que vous avez créé dans la section précédente.

## <a name="create-helm-chart"></a>Créer un chart Helm ##
Tout d’abord, créez un exemple de chart Helm avec la commande suivante.
```console
helm create mychart
```

Helm crée un répertoire appelé mychart avec la structure ci-dessous. 

> [!TIP]
> Pour plus d’informations, consultez ce [guide des charts](https://helm.sh/docs/chart_template_guide/getting_started/).

```
mychart
|-- Chart.yaml
|-- charts
|-- templates
|   |-- NOTES.txt
|   |-- _helpers.tpl
|   |-- deployment.yaml
|   |-- ingress.yaml
|   `-- service.yaml
`-- values.yaml
```

Ensuite, mettez à jour la section **spec:template:spec:containers** du fichier *deployment.yaml*. L’extrait de code suivant ajoute deux variables d’environnement au conteneur. Vous définirez leurs valeurs dynamiquement au moment du déploiement.

```yaml
env:
- name: Color
    value: {{ .Values.settings.color }}
- name: Message
    value: {{ .Values.settings.message }}
``` 

Le fichier *deployment.yaml* complet après la mise à jour doit ressembler à ce qui suit.

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    app.kubernetes.io/name: {{ include "mychart.name" . }}
    helm.sh/chart: {{ include "mychart.chart" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app.kubernetes.io/name: {{ include "mychart.name" . }}
      app.kubernetes.io/instance: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ include "mychart.name" . }}
        app.kubernetes.io/instance: {{ .Release.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          env:
            - name: Color
              value: {{ .Values.settings.color }}
            - name: Message
              value: {{ .Values.settings.message }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
{{ toYaml .Values.resources | indent 12 }}
    {{- with .Values.nodeSelector }}
      nodeSelector:
{{ toYaml . | indent 8 }}
    {{- end }}
    {{- with .Values.affinity }}
      affinity:
{{ toYaml . | indent 8 }}
    {{- end }}
    {{- with .Values.tolerations }}
      tolerations:
{{ toYaml . | indent 8 }}
    {{- end }}
```

Pour stocker des données sensibles en tant que secrets Kubernetes, ajoutez un fichier *secrets.yaml* dans le dossier templates.

> [!TIP]
> Découvrez plus d’informations sur la façon d’utiliser les [secrets Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets).

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: {{ .Values.secrets.password | b64enc }}
```

Enfin, mettez à jour le fichier *values.yaml* avec le contenu suivant pour fournir si nécessaire les valeurs par défaut des paramètres de configuration et des secrets référencés dans les fichiers *deployment.yaml* et *secrets.yaml*. Leurs valeurs réelles seront remplacées par la configuration extraite d’App configuration.

```yaml
# settings will be overwritten by App Configuration
settings:
    color: red
    message: myMessage
```

## <a name="pass-configuration-from-app-configuration-in-helm-install"></a>Passer la configuration d’App Configuration dans l’installation Helm ##
Tout d’abord, téléchargez la configuration depuis App Configuration dans un fichier *myConfig.yaml*. Utilisez un filtre de clé pour télécharger seulement les clés qui commencent par **settings.** . Si dans votre cas, le filtre de clé n’est pas suffisant pour exclure les clés des références Key Vault, vous pouvez utiliser l’argument **--skip-keyvault** pour les exclure. 

> [!TIP]
> Découvrez plus d’informations sur la [commande export](/cli/azure/appconfig/kv?view=azure-cli-latest#az-appconfig-kv-export). 

```azurecli-interactive
az appconfig kv export -n myAppConfiguration -d file --path myConfig.yaml --key "settings.*"  --separator "." --format yaml
```

Ensuite, téléchargez les secrets dans un fichier appelé *mySecrets.yaml*. L’argument de ligne de commande **--resolve-keyvault** résout les références Key Vault en extrayant les valeurs réelles dans Key Vault. Vous devez exécuter cette commande avec des informations d’identification qui ont des autorisations d’accès au coffre de clés correspondant.

> [!WARNING]
> Comme ce fichier contient des informations sensibles, conservez soigneusement le fichier et effacez-le quand il n’est plus nécessaire.

```azurecli-interactive
az appconfig kv export -n myAppConfiguration -d file --path mySecrets.yaml --key "secrets.*" --separator "." --resolve-keyvault --format yaml
```

Utilisez l’argument **-f** de mise à niveau de Helm pour passer les deux fichiers de configuration que vous avez créés. Elles remplacent les valeurs de configuration définies dans *values.yaml* avec les valeurs exportées depuis App Configuration.

```console
helm upgrade --install -f myConfig.yaml -f mySecrets.yaml "example" ./mychart 
```

Vous pouvez également utiliser l’argument **--set** pour la mise à niveau de Helm afin de passer des valeurs de clés littérales. L’utilisation de l’argument **--set** est un bon moyen d’éviter de conserver des données sensibles sur le disque. 

```powershell
$secrets = az appconfig kv list -n myAppConfiguration --key "secrets.*" --resolve-keyvault --query "[*].{name:key, value:value}" | ConvertFrom-Json

foreach ($secret in $secrets) {
  $keyvalues += $secret.name + "=" + $secret.value + ","
}

if ($keyvalues){
  $keyvalues = $keyvalues.TrimEnd(',')
  helm upgrade --install --set $keyvalues "example" ./mychart 
}
else{
  helm upgrade --install "example" ./mychart 
}

```

Vérifiez que les configurations et les secrets ont été définis correctement en accédant au [Tableau de bord Kubernetes](../aks/kubernetes-dashboard.md). Vous voyez que les valeurs pour **color** et **message** provenant d’App Configuration ont été renseignées dans les variables d’environnement du conteneur.

![Démarrage rapide du lancement d’application local](./media/kubernetes-dashboard-env-variables.png)

Un secret, **password** , stocké en tant que référence Key Vault dans App Configuration, a également été ajouté aux secrets Kubernetes. 

![Capture d’écran mettant en évidence le mot de passe dans la section Data.](./media/kubernetes-dashboard-secrets.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez exporté des données d’App Configuration à utiliser dans un déploiement Kubernetes avec Helm. Pour en savoir plus sur l’utilisation d’App Configuration, passez aux exemples Azure CLI.

> [!div class="nextstepaction"]
> [Azure CLI](/cli/azure/appconfig?view=azure-cli-latest)