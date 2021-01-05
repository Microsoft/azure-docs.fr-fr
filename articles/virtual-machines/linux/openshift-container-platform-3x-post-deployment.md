---
title: Tâches post-déploiement d’OpenShift Container Platform 3.11 dans Azure
description: Tâches supplémentaires à effectuer après le déploiement d’un cluster OpenShift Container Platform 3.11.
author: haroldwongms
manager: mdotson
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.workload: infrastructure
ms.date: 10/14/2019
ms.author: haroldw
ms.custom: devx-track-ansible, devx-track-azurecli
ms.openlocfilehash: dd967ad08b628f9073edfe548033f7e97845d047
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016061"
---
# <a name="post-deployment-tasks"></a>Tâches de post-déploiement

Une fois que vous avez déployé un cluster OpenShift, vous pouvez configurer d’autres éléments. Cet article couvre les points suivants :

- Configurer l’authentification unique à l’aide d’Azure Active Directory (Azure AD)
- Configurer les journaux Azure Monitor pour superviser OpenShift
- Configurer les métriques et la journalisation
- Installer Open Service Broker pour Azure (OSBA)

## <a name="configure-single-sign-on-by-using-azure-active-directory"></a>Configurer l’authentification unique à l’aide d’Azure Active Directory

Pour utiliser Azure Active Directory à des fins d’authentification, vous devez d’abord créer une inscription d’application Azure AD. Ce processus implique deux étapes : création de l’inscription d’application et configuration d’autorisations.

### <a name="create-an-app-registration"></a>Créer une inscription d’application

Ces étapes utilisent Azure CLI pour créer l’inscription d’application et l’interface graphique utilisateur (portail) pour définir les autorisations. Pour créer l’inscription d’application, cinq éléments d’information sont nécessaires :

- Nom complet : nom d’inscription de l’application (par exemple : OCPAzureAD)
- Page d’accueil : URL de la console OpenShift (par exemple, `https://masterdns343khhde.westus.cloudapp.azure.com/console`)
- URL d’identificateur : URL de la console OpenShift (par exemple, `https://masterdns343khhde.westus.cloudapp.azure.com/console`)
- URL de réponse : URL publique maître et nom d’inscription de l’application (par exemple, `https://masterdns343khhde.westus.cloudapp.azure.com/oauth2callback/OCPAzureAD`)
- Mot de passe : mot de passe sécurisé (utilisez un mot de passe fort)

L’exemple suivant crée une inscription d’application à l’aide des informations précédentes :

```azurecli
az ad app create --display-name OCPAzureAD --homepage https://masterdns343khhde.westus.cloudapp.azure.com/console --reply-urls https://masterdns343khhde.westus.cloudapp.azure.com/oauth2callback/hwocpadint --identifier-uris https://masterdns343khhde.westus.cloudapp.azure.com/console --password {Strong Password}
```

Si la commande aboutit, vous obtenez une sortie JSON similaire à la suivante :

```json
{
  "appId": "12345678-ca3c-427b-9a04-ab12345cd678",
  "appPermissions": null,
  "availableToOtherTenants": false,
  "displayName": "OCPAzureAD",
  "homepage": "https://masterdns343khhde.westus.cloudapp.azure.com/console",
  "identifierUris": [
    "https://masterdns343khhde.westus.cloudapp.azure.com/console"
  ],
  "objectId": "62cd74c9-42bb-4b9f-b2b5-b6ee88991c80",
  "objectType": "Application",
  "replyUrls": [
    "https://masterdns343khhde.westus.cloudapp.azure.com/oauth2callback/OCPAzureAD"
  ]
}
```

Prenez note de la propriété appId renvoyée par la commande. Vous en aurez besoin pour une étape suivante.

Dans le portail Azure :

1. Sélectionnez **Azure Active Directory** > **Inscription d’application**.
2. Recherchez votre inscription d’application (par exemple : OCPAzureAD).
3. Dans les résultats, cliquez sur l’inscription d’application.
4. Sous **Paramètres**, sélectionnez **Autorisations requises**.
5. Sous **Autorisations requises**, sélectionnez **Ajouter**.

   ![Inscription d’application](media/openshift-post-deployment/app-registration.png)

6. Cliquez sur l’étape 1 : Sélectionner une API, puis cliquez sur **Windows Azure Active Directory (Microsoft.Azure.ActiveDirectory)** . Cliquez sur **Sélectionner** en bas.

   ![Inscription d’application - Sélectionner une API](media/openshift-post-deployment/app-registration-select-api.png)

7. Sur l’étape 2 : Sélectionner des autorisations, sélectionnez **Activer la connexion et lire le profil utilisateur** sous **Autorisations déléguées**, puis cliquez sur **Sélectionner**.

   ![Accès à l’inscription d’application](media/openshift-post-deployment/app-registration-access.png)

8. Sélectionnez **Terminé**.

### <a name="configure-openshift-for-azure-ad-authentication"></a>Configurer OpenShift pour l’authentification Azure AD

Pour configurer OpenShift pour utiliser Azure AD en tant que fournisseur d’authentification, le fichier /etc/origin/master/master-config.yaml doit être modifié sur tous les nœuds master.

Trouvez l’ID de locataire à l’aide de la commande d’interface de ligne de commande suivante :

```azurecli
az account show
```

Dans le fichier yaml, recherchez les lignes suivantes :

```yaml
oauthConfig:
  assetPublicURL: https://masterdns343khhde.westus.cloudapp.azure.com/console/
  grantConfig:
    method: auto
  identityProviders:
  - challenge: true
    login: true
    mappingMethod: claim
    name: htpasswd_auth
    provider:
      apiVersion: v1
      file: /etc/origin/master/htpasswd
      kind: HTPasswdPasswordIdentityProvider
```

Immédiatement après les lignes précédentes, insérez les lignes suivantes :

```yaml
  - name: <App Registration Name>
    challenge: false
    login: true
    mappingMethod: claim
    provider:
      apiVersion: v1
      kind: OpenIDIdentityProvider
      clientID: <appId>
      clientSecret: <Strong Password>
      claims:
        id:
        - sub
        preferredUsername:
        - unique_name
        name:
        - name
        email:
        - email
      urls:
        authorize: https://login.microsoftonline.com/<tenant Id>/oauth2/authorize
        token: https://login.microsoftonline.com/<tenant Id>/oauth2/token
```

Assurez-vous que le texte est bien aligné sous identityProviders. Trouvez l’ID de locataire à l’aide de la commande d’interface de ligne de commande suivante : ```az account show```

Redémarrez les services maître OpenShift sur tous les nœuds master :

```bash
sudo /usr/local/bin/master-restart api
sudo /usr/local/bin/master-restart controllers
```

Dans la console OpenShift, vous voyez maintenant deux options pour l’authentification : htpasswd_auth et [Inscription d’application].

## <a name="monitor-openshift-with-azure-monitor-logs"></a>Superviser OpenShift avec les journaux Azure Monitor

Il y a trois façons d’ajouter l’agent Log Analytics dans OpenShift.
- Installer l’agent Log Analytics pour Linux directement sur chaque nœud OpenShift
- Activer l’extension de machine virtuelle Azure Monitor sur chaque nœud OpenShift
- Installer l’agent Log Analytics en tant que daemon-set OpenShift

Pour plus d’informations, lisez les [instructions](../../azure-monitor/insights/containers.md#configure-a-log-analytics-agent-for-red-hat-openshift) complètes.

## <a name="configure-metrics-and-logging"></a>Configurer les métriques et la journalisation

En fonction de la branche utilisée, les modèles Azure Resource Manager pour OpenShift Container Platform et OKD peuvent fournir des paramètres d’entrée pour l’activation des métriques et une journalisation au moment de l’installation.

L’offre de la Place de marché pour OpenShift Container Platform fournit également une option permettant d’activer les métriques et la journalisation pendant l’installation du cluster.

Si les métriques ou la journalisation n’ont pas été activées durant l’installation du cluster, elles peuvent l’être facilement après.

### <a name="azure-cloud-provider-in-use"></a>Fournisseur de cloud Azure utilisé

Établissez une connexion SSH au nœud Bastion ou au premier nœud master (en fonction du modèle et de la branche utilisés), à l’aide des informations d’identification fournies durant le déploiement. Émettez les commandes suivantes :

```bash
ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/openshift-metrics/config.yml \
-e openshift_metrics_install_metrics=True \
-e openshift_metrics_cassandra_storage_type=dynamic

ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/openshift-logging/config.yml \
-e openshift_logging_install_logging=True \
-e openshift_logging_es_pvc_dynamic=true
```

### <a name="azure-cloud-provider-not-in-use"></a>Fournisseur de cloud Azure non utilisé

```bash
ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/openshift-metrics/config.yml \
-e openshift_metrics_install_metrics=True

ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/openshift-logging/config.yml \
-e openshift_logging_install_logging=True
```

## <a name="install-open-service-broker-for-azure-osba"></a>Installer Open Service Broker pour Azure (OSBA)

Open Service Broker pour Azure (OSBA) vous permet de provisionner des services cloud Azure directement à partir d’OpenShift. OSBA est une implémentation de l’API Open Service Broker pour Azure. L’API Open Service Broker est une spécification qui définit un langage commun pour les fournisseurs de cloud que les applications cloud natives peuvent utiliser pour manager les services cloud sans verrou.

Pour installer OSBA sur OpenShift, suivez les instructions fournies ici : https://github.com/Azure/open-service-broker-azure#openshift-project-template. 
> [!NOTE]
> Effectuez uniquement les étapes décrites dans la section Modèle de projet OpenShift et non celles de la section Installation entière.

## <a name="next-steps"></a>Étapes suivantes

- [Prise en main d’OpenShift Container Platform](https://docs.openshift.com)
