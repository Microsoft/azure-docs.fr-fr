---
title: Tutoriel – Se connecter à un cluster Azure Red Hat OpenShift 4
description: Découvrez comment vous connecter à un cluster Microsoft Azure Red Hat OpenShift
author: sakthi-vetrivel
ms.author: suvetriv
ms.topic: tutorial
ms.service: container-service
ms.date: 04/24/2020
ms.openlocfilehash: 132a9b7ad3eef17f4ca27fb76a0c05f94c82fd37
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92428281"
---
# <a name="tutorial-connect-to-an-azure-red-hat-openshift-4-cluster"></a>Tutoriel : Se connecter à un cluster Azure Red Hat OpenShift 4

Dans la deuxième partie de ce tutoriel (qui en compte trois), vous allez vous connecter à un cluster Azure Red Hat OpenShift en exécutant OpenShift 4 en tant qu’utilisateur kubeadmin via la console web OpenShift. Vous allez apprendre à effectuer les actions suivantes :
> [!div class="checklist"]
> * Obtenir les informations d’identification `kubeadmin` pour votre cluster
> * Installer l’interface CLI OpenShift
> * Se connecter à un cluster Azure Red Hat OpenShift en utilisant l’interface CLI OpenShift

## <a name="before-you-begin"></a>Avant de commencer

Dans les tutoriels précédents, un cluster Azure Red Hat OpenShift a été créé. Si vous n’avez pas effectué ces étapes et si vous souhaitez suivre cette procédure, commencez par le [Tutoriel 1 - Créer un cluster Azure Red Hat OpenShift 4](tutorial-create-cluster.md).

Si vous choisissez d’installer et d’utiliser l’interface CLI localement, ce tutoriel demande au minimum la version 2.6.0 d’Azure CLI. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="connect-to-the-cluster"></a>Se connecter au cluster

Vous pouvez vous connecter au cluster à l’aide de l’utilisateur `kubeadmin`.  Exécutez la commande suivante pour rechercher le mot de passe de l’utilisateur `kubeadmin`.

```azurecli-interactive
az aro list-credentials \
  --name $CLUSTER \
  --resource-group $RESOURCEGROUP
```

L’exemple de sortie suivant montre ce que sera le mot de passe dans `kubeadminPassword`.

```json
{
  "kubeadminPassword": "<generated password>",
  "kubeadminUsername": "kubeadmin"
}
```

Vous trouverez l’URL de la console de cluster en exécutant la commande suivante, qui se présente comme ceci : `https://console-openshift-console.apps.<random>.<region>.aroapp.io/`.

```azurecli-interactive
 az aro show \
    --name $CLUSTER \
    --resource-group $RESOURCEGROUP \
    --query "consoleProfile.url" -o tsv
```

Lancez l’URL de la console dans un navigateur et connectez-vous à l’aide des informations d’identification `kubeadmin`.

![Écran de connexion Azure Red Hat OpenShift](media/aro4-login.png)

## <a name="install-the-openshift-cli"></a>Installer l’interface CLI OpenShift

Une fois que vous êtes connecté à la console web OpenShift, cliquez sur **?** en haut à droite, puis sur **Command Line Tools** (Outils en ligne de commande). Téléchargez la version qui convient pour votre ordinateur.

![Capture d’écran qui met en évidence l’option Outils en ligne de commande dans la liste lorsque vous sélectionnez l’icône ? .](media/aro4-download-cli.png)

Vous pouvez aussi télécharger la dernière version de l’interface CLI adaptée à votre ordinateur à partir de <https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/>.

Si vous exécutez les commandes sur Azure Cloud Shell, téléchargez la dernière version de l’interface CLI OpenShift 4 pour Linux.

```azurecli-interactive
cd ~
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz

mkdir openshift
tar -zxvf openshift-client-linux.tar.gz -C openshift
echo 'export PATH=$PATH:~/openshift' >> ~/.bashrc && source ~/.bashrc
```

## <a name="connect-using-the-openshift-cli"></a>Se connecter à partir de l’interface CLI OpenShift

Récupérez l’adresse du serveur d’API.

```azurecli-interactive
apiServer=$(az aro show -g $RESOURCEGROUP -n $CLUSTER --query apiserverProfile.url -o tsv)
```

Connectez-vous au serveur d’API du cluster OpenShift à l’aide de la commande suivante. Remplacez **\<kubeadmin password>** par le mot de passe que vous venez de récupérer.

```azurecli-interactive
oc login $apiServer -u kubeadmin -p <kubeadmin password>
```

## <a name="next-steps"></a>Étapes suivantes

Dans cette partie du tutoriel, vous avez appris à :
> [!div class="checklist"]
> * Obtenir les informations d’identification `kubeadmin` pour votre cluster
> * Installer l’interface CLI OpenShift
> * Vous connecter à un cluster Azure Red Hat OpenShift en utilisant l’interface CLI OpenShift

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Supprimer un cluster Azure Red Hat OpenShift](tutorial-delete-cluster.md)