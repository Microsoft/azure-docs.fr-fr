---
title: Prérequis d’OpenShift Container Platform 3.11 dans Azure
description: Conditions préalables au déploiement d’OpenShift Container Platform 3.11 dans Azure.
author: haroldwongms
manager: mdotson
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.workload: infrastructure
ms.date: 10/23/2019
ms.author: haroldw
ms.openlocfilehash: 68bd748e890659e4b79d76e4ccab038f251a937a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016027"
---
# <a name="common-prerequisites-for-deploying-openshift-container-platform-311-in-azure"></a>Conditions préalables courantes au déploiement d’OpenShift Container Platform 3.11 dans Azure

Cet article décrit les prérequis courants pour déployer OpenShift Container Platform ou OKD dans Azure.

L’installation d’OpenShift utilise des playbooks Ansible. Ansible utilise Secure Shell (SSH) pour se connecter à tous les hôtes de cluster pour effectuer les étapes d’installation.

Quand Ansible établit la connexion SSH avec les hôtes distants, il ne peut pas entrer de mot de passe. Pour cette raison, aucun mot de passe (phrase secrète) ne doit être associé à la clé privée, sous peine de provoquer l’échec du déploiement.

Puisque les machines virtuelles sont déployées via des modèles Azure Resource Manager, la même clé publique est utilisée pour accéder à l’ensemble des machines virtuelles. La clé privée correspondante doit se trouver dans la machine virtuelle qui exécute aussi tous les playbooks. Pour effectuer cette action en toute sécurité, un coffre de clés Azure est utilisé afin de transmettre la clé privée à la machine virtuelle.

Si un stockage persistant est nécessaire pour les conteneurs, alors des volumes persistants sont nécessaires. OpenShift prend en charge les disques durs virtuels (VHD) Azure pour les volumes persistants, mais Azure doit d'abord être configuré en tant que fournisseur de cloud.

Dans ce modèle, OpenShift :

- Crée un objet VHD sur un compte Stockage Azure ou un disque managé.
- Monte le disque dur virtuel (VHD) sur une machine virtuelle et formate le volume.
- Monte le volume sur le pod.

Pour que cette configuration fonctionne, OpenShift doit être autorisé à effectuer ces tâches dans Azure. Un principal de service est utilisé à cet effet. Le principal de service est un compte de sécurité dans Azure Active Directory qui dispose d’autorisations sur les ressources.

Le principal de service a besoin d’accéder aux comptes de stockage et aux machines virtuelles qui composent le cluster. Si toutes les ressources de cluster OpenShift sont déployées sur un seul groupe de ressources, des autorisations pour ce groupe de ressources peuvent être affectées au principal de service.

Ce guide décrit comment créer les artefacts associés aux prérequis.

> [!div class="checklist"]
> * Créez un coffre de clés pour gérer les clés SSH pour le cluster OpenShift.
> * Créez un principal de service, qui sera utilisé par le fournisseur de cloud Azure.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="sign-in-to-azure"></a>Connexion à Azure 
Connectez-vous à votre abonnement Azure avec la commande [az login](/cli/azure/reference-index) et suivez les instructions à l’écran ou cliquez sur **Essayer** pour utiliser Cloud Shell.

```azurecli
az login
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Vous devez utiliser un groupe de ressources dédié pour héberger le coffre de clés. Ce groupe est différent du groupe de ressources dans lequel se déploient les ressources de cluster OpenShift.

L’exemple suivant crée un groupe de ressources nommé *keyvaultrg* à l’emplacement *eastus* :

```azurecli
az group create --name keyvaultrg --location eastus
```

## <a name="create-a-key-vault"></a>Création d’un coffre de clés
Créez un coffre de clés pour stocker les clés SSH du cluster avec la commande [az keyvault create](/cli/azure/keyvault). Le nom du coffre de clés doit être unique à l’échelle globale et être activé pour le déploiement de modèle. Sinon, le déploiement échoue et le message d’erreur « KeyVaultParameterReferenceSecretRetrieveFailed » s’affiche.

L’exemple suivant crée un coffre de clés nommé *keyvault* dans le groupe de ressources *keyvaultrg* :

```azurecli 
az keyvault create --resource-group keyvaultrg --name keyvault \
       --enabled-for-template-deployment true \
       --location eastus
```

## <a name="create-an-ssh-key"></a>Création d’une clé SSH 
Une clé SSH est nécessaire pour sécuriser l’accès au cluster OpenShift. Créez une paire de clés SSH à l’aide de la commande `ssh-keygen` (sur Linux ou macOS) :
 
```bash
ssh-keygen -f ~/.ssh/openshift_rsa -t rsa -N ''
```

> [!NOTE]
> Votre paire de clés SSH ne peut pas avoir de mot de passe/phrase secrète.

Pour plus d’informations sur les clés SSH sur Windows, consultez [Guide pratique pour créer des clés SSH sur Windows](./ssh-from-windows.md). Assurez-vous d’exporter la clé privée au format OpenSSH.

## <a name="store-the-ssh-private-key-in-azure-key-vault"></a>Stocker la clé privée SSH dans Azure Key Vault
Le déploiement OpenShift utilise la clé SSH que vous avez créée pour sécuriser l’accès au maître OpenShift. Pour permettre au déploiement de récupérer la clé SSH en toute sécurité, stockez la clé dans Key Vault à l’aide de la commande suivante :

```azurecli
az keyvault secret set --vault-name keyvault --name keysecret --file ~/.ssh/openshift_rsa
```

## <a name="create-a-service-principal"></a>Créer un principal du service 
OpenShift communique avec Azure à l’aide d’un nom d’utilisateur et d’un mot de passe ou d’un principal de service. Un principal de service Azure est une identité de sécurité que vous pouvez utiliser avec des applications, des services et des outils d’automatisation comme OpenShift. Vous contrôlez et vous définissez les opérations que le principal du service est autorisé à effectuer dans Azure. Il est préférable d'étendre les autorisations du principal du service à des groupes de ressources spécifiques plutôt qu'à l'ensemble de l'abonnement.

Créez un principal de service avec la commande [az ad sp create-for-rbac](/cli/azure/ad/sp) et affichez les informations d’identification requises par OpenShift.

L’exemple suivant crée un principal de service et lui assigne des autorisations de contributeur sur un groupe de ressources nommé *openshiftrg*.

Commencez par créer le groupe de ressources appelé *openshiftrg* :

```azurecli
az group create -l eastus -n openshiftrg
```

Créez un principal de service :

```azurecli
az group show --name openshiftrg --query id
```

Enregistrez la sortie de la commande et utilisez-la à la place de $scope dans la commande suivante.

```azurecli
az ad sp create-for-rbac --name openshiftsp \
      --role Contributor --scopes $scope \
```

Prenez note de la propriété appId et du mot de passe renvoyés par la commande :

```json
{
  "appId": "11111111-abcd-1234-efgh-111111111111",
  "displayName": "openshiftsp",
  "name": "http://openshiftsp",
  "password": {Strong Password},
  "tenant": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
}
```

 > [!WARNING] 
 > Veillez à noter le mot de passe sécurisé, car il ne sera pas possible de le récupérer une nouvelle fois.

Pour plus d’informations sur les principaux de service, consultez [Créer un principal du service avec Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).

## <a name="prerequisites-applicable-only-to-resource-manager-template"></a>Conditions préalables uniquement applicables aux modèles Resource Manager

Des secrets doivent être créés pour la clé privée SSH (**sshPrivateKey**), la clé secrète client Azure AD (**aadClientSecret**), le mot de passe de l'administrateur OpenShift (**openshiftPassword**), et le mot de passe ou la clé d'activation du Gestionnaire d'abonnements Red Hat (**rhsmPasswordOrActivationKey**).  De plus, si des certificats TLS/SSL personnalisés sont utilisés, six secrets supplémentaires devront être créés : **routingcafile**, **routingcertfile**, **routingkeyfile**, **mastercafile**, **mastercertfile** et **masterkeyfile**.  Ces paramètres seront décrits en détail.

Le modèle fait référence à des noms de secret spécifiques. Par conséquent, vous **devez** utiliser les noms en gras répertoriés ci-dessus (en respectant la casse).

### <a name="custom-certificates"></a>Certificats personnalisés

Par défaut, le modèle déploie un cluster OpenShift en utilisant des certificats auto-signés pour la console web OpenShift et le domaine de routage. Si vous souhaitez utiliser des certificats TLS/SSL personnalisés, définissez « routingCertType » sur « custom » et « masterCertType » sur « custom ».  Vous aurez besoin des fichiers CA, Cert et Key au format .pem pour les certificats.  Il est possible d'utiliser des certificats personnalisés pour l'un mais pas pour l'autre.

Vous devrez stocker ces fichiers dans des secrets du coffre de clés Key Vault.  Utilisez le même coffre que celui utilisé pour la clé privée.  Plutôt que d’exiger six entrées supplémentaires pour les noms secrets, le modèle est codé en dur afin d’utiliser des noms secrets spécifiques pour chacun des fichiers de certificat TLS/SSL.  Stockez les données de certificat à l'aide des informations du tableau suivant.

| Nom du secret      | Fichier de certificat   |
|------------------|--------------------|
| mastercafile     | fichier d'autorité de certification (CA) principal     |
| mastercertfile   | fichier de certificat (CERT) principal   |
| masterkeyfile    | fichier de clé (Key) principal    |
| routingcafile    | fichier d'autorité de certification (CA) de routage    |
| routingcertfile  | fichier de certificat (CERT) de routage  |
| routingkeyfile   | fichier de clé (Key) de routage   |

Créez les secrets à l’aide de l'interface de ligne de commande Azure (Azure CLI). Voici un exemple.

```azurecli
az keyvault secret set --vault-name KeyVaultName -n mastercafile --file ~/certificates/masterca.pem
```

## <a name="next-steps"></a>Étapes suivantes

Cet article a abordé les thèmes suivants :
> [!div class="checklist"]
> * Créez un coffre de clés pour gérer les clés SSH pour le cluster OpenShift.
> * Créez un principal de service pour que le fournisseur de solution cloud Azure l’utilise.

Ensuite, déployez un cluster OpenShift :

- [Déployer OpenShift Container Platform](./openshift-container-platform-3x.md)
- [Déployer un cluster OpenShift Container Platform autogéré à l'aide de l'offre de la Place de marché](./openshift-container-platform-3x-marketplace-self-managed.md)
