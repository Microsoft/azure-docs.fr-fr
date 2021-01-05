---
title: Déployer un cluster OpenShift Container Platform 3.11 autogéré à l’aide de l’offre de la Place de marché dans Azure
description: Déployez un cluster OpenShift Container Platform 3.11 autogéré à l’aide de l’offre de la Place de marché dans Azure.
author: haroldwongms
manager: mdotson
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 10/14/2019
ms.author: haroldw
ms.openlocfilehash: 36adf35c5fbfc3e88b7d9af425ebabc852707e69
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87374097"
---
# <a name="configure-prerequisites"></a>Configuration préalable requise

Avant d’utiliser l’offre de la Place de marché pour déployer un cluster OpenShift Container Platform 3.11 autogéré dans Azure, vous devez configurer certaines conditions préalables.  Consultez l’article [Conditions préalables de OpenShift](./openshift-container-platform-3x-prerequisites.md) pour obtenir des instructions sur la création d’une clé SSH (sans phrase secrète), d’un coffre de clés Azure, d’un secret de coffre de clés et d’un principal de service.

 
## <a name="deploy-using-the-marketplace-offer"></a>Déployer à l’aide de l’offre Place de marché

La façon la plus simple de déployer un cluster OpenShift Container Platform 3.11 autogéré dans Azure consiste à utiliser l’[offre de la Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/osatesting.open-shift-azure-proxy).

Cette option est la plus simple, mais les possibilités de personnalisation sont limitées. L’offre de la place de marché déploie OpenShift Container Platform 3.11.82 et comprend les options de configuration suivantes :

- **Nœuds master** : trois (3) nœuds master avec le type d’instance configurable.
- **Nœuds infrastructure** : trois (3) nœuds infrastructure avec le type d’instance configurable.
- **Nœuds** : le nombre de nœuds (entre 1 et 9) et le type d’instance sont configurables.
- **Type de disque** : La fonctionnalité Disques managés est utilisée.
- **Réseau** : prise en charge d’un réseau nouveau ou existant, ainsi que de la plage CIDR personnalisée.
- **CNS** : peut être activé.
- **Métriques** : les métriques Hawkular peuvent être activées.
- **Journalisation** : la journalisation EFK peut être activée.
- **Fournisseur de cloud Azure** : activé par défaut, peut être désactivé.

Dans le coin supérieur gauche du portail Azure, cliquez sur **Créer une ressource**, entrez « openshift container platform » dans la zone de recherche et appuyez sur Entrée.

   ![Recherche de nouvelles ressources](media/openshift-marketplace-self-managed/ocp-search.png)  
<br>

La page Résultats s’affichera avec une liste contenant **Red Hat OpenShift Container Platform 3.11 autogéré**. 

   ![Résultats de recherche de nouvelles ressources](media/openshift-marketplace-self-managed/ocp-searchresult.png)  
<br>

Cliquez sur l’offre pour afficher ses détails. Pour déployer cette offre, cliquez sur **Créer**. L’interface utilisateur permettant d’entrer les paramètres nécessaires s’affichera. Le premier écran est le panneau **Concepts de base**.

   ![Page du titre de l’offre](media/openshift-marketplace-self-managed/ocp-titlepage.png)  
<br>

**Concepts de base**

Pour obtenir de l’aide sur les paramètres d’entrée, placez votre curseur sur l’icône ***i*** en regard du nom du paramètre.

Entrez les valeurs des paramètres d’entrée et cliquez sur **OK**.

| Paramètre d’entrée | Description du paramètre |
|-----------------------|-----------------|
| Nom d’utilisateur administrateur de la machine virtuelle | Utilisateur administrateur à créer sur toutes les instances de machine virtuelle |
| Clé publique SSH pour l’utilisateur administrateur | Clé publique SSH utilisée pour se connecter à la machine virtuelle, ne doit pas utiliser de phrase secrète |
| Abonnement | Abonnement Azure dans lequel déployer un cluster |
| Groupe de ressources | Pour les ressources de cluster, créez un groupe de ressources ou sélectionnez un groupe de ressources vide existant |
| Emplacement | Région Azure dans laquelle déployer le cluster |

   ![Panneau des concepts de base de l’offre](media/openshift-marketplace-self-managed/ocp-basics.png)  
<br>

**Paramètres de l’infrastructure**

Entrez les valeurs des paramètres d’entrée et cliquez sur **OK**.

| Paramètre d’entrée | Description du paramètre |
|-----------------------|-----------------|
| Préfixe du nom du cluster OCP | Préfixe de cluster utilisé pour configurer les noms d’hôtes de tous les nœuds. Il doit contenir entre 1 et 20 caractères |
| Taille du nœud principal | Acceptez la taille de machine virtuelle par défaut ou cliquez sur **Modifier la taille** pour sélectionner une autre taille de machine virtuelle.  Sélectionnez la taille de machine virtuelle appropriée pour votre charge de travail |
| Taille de nœud d’infrastructure | Acceptez la taille de machine virtuelle par défaut ou cliquez sur **Modifier la taille** pour sélectionner une autre taille de machine virtuelle.  Sélectionnez la taille de machine virtuelle appropriée pour votre charge de travail |
| Nombre de nœuds d’application | Acceptez la taille de machine virtuelle par défaut ou cliquez sur **Modifier la taille** pour sélectionner une autre taille de machine virtuelle.  Sélectionnez la taille de machine virtuelle appropriée pour votre charge de travail |
| Taille de nœud d’application | Acceptez la taille de machine virtuelle par défaut ou cliquez sur **Modifier la taille** pour sélectionner une autre taille de machine virtuelle.  Sélectionnez la taille de machine virtuelle appropriée pour votre charge de travail |
| Taille d’hôte Bastion | Acceptez la taille de machine virtuelle par défaut ou cliquez sur **Modifier la taille** pour sélectionner une autre taille de machine virtuelle.  Sélectionnez la taille de machine virtuelle appropriée pour votre charge de travail |
| Réseau virtuel nouveau ou existant | Créez un réseau virtuel (par défaut) ou utilisez un réseau virtuel existant |
| Choisissez les paramètres CIDR par défaut ou personnalisez la plage d’adresses IP (CIDR) | Acceptez les plages CIDR par défaut ou sélectionnez **Plage d’adresses IP personnalisée** et entrez les informations CIDR personnalisées.  Les paramètres par défaut créent un réseau virtuel avec un CIDR de 10.0.0.0/14, un sous-réseau principal avec 10.1.0.0/16, un sous-réseau d’infrastructure avec 10.2.0.0/16 et un sous-réseau pour le calcul et le CNS avec 10.3.0.0/16 |
| Nom du groupe de ressources de coffre de clés | Nom du groupe de ressources qui contient le Key Vault |
| Nom du coffre de clés | Le nom du coffre de clés qui contient le secret avec la clé privée SSH.  Seuls les caractères alphanumériques et les traits d’union sont autorisés et la clé doit contenir entre 3 et 24 caractères |
| Nom du secret | Le nom du secret qui contient le secret avec la clé privée SSH.  Seuls les caractères alphanumériques et les traits d’union sont autorisés |

   ![Panneau de l’infrastructure de l’offre](media/openshift-marketplace-self-managed/ocp-inframain.png)  
<br>

**Modifier la taille**

Pour sélectionner une autre taille de machine virtuelle, cliquez sur ***Modifier la taille***.  La fenêtre de sélection de la machine virtuelle s’affiche.  Sélectionnez la taille de machine virtuelle de votre choix, puis cliquez sur **Sélectionner**.

   ![Sélectionner la taille de machine virtuelle](media/openshift-marketplace-self-managed/ocp-selectvmsize.png)  
<br>

**Réseau virtuel existant**

| Paramètre d’entrée | Description du paramètre |
|-----------------------|-----------------|
| Nom de réseau virtuel existant | Nom du réseau virtuel existant |
| Nom de sous-réseau pour les nœuds principaux | Nom du sous-réseau existant pour les nœuds principaux.  Doit contenir au moins 16 adresses IP et être conforme au document RFC 1918 |
| Nom de sous-réseau pour les nœuds d’infrastructure | Nom du sous-réseau existant pour les nœuds d’infrastructure.  Doit contenir au moins 32 adresses IP et être conforme au document RFC 1918 |
| Nom de sous-réseau pour les nœuds de calcul et de CNS | Nom du sous-réseau existant pour les nœuds de calcul et de CNS.  Doit contenir au moins 32 adresses IP et être conforme au document RFC 1918 |
| Groupe de ressources pour le réseau virtuel existant | Nom du groupe de ressources qui contient le sous-réseau existant |

   ![Réseau virtuel existant de l’infrastructure de l’offre](media/openshift-marketplace-self-managed/ocp-existingvnet.png)  
<br>

**Plage d’adresses IP personnalisée**

| Paramètre d’entrée | Description du paramètre |
|-----------------------|-----------------|
| Plage d’adresses pour le réseau virtuel | Plage CIDR personnalisée pour le réseau virtuel |
| Plage d’adresses pour le sous-réseau contenant les nœuds principaux | Plage CIDR personnalisée pour le sous-réseau principal |
| Plage d’adresses pour le sous-réseau contenant les nœuds d’infrastructure | Plage CIDR personnalisée pour le sous-réseau d’infrastructure |
| Plage d’adresses pour le sous-réseau contenant les nœuds de calcul et de CNS | Plage CIDR personnalisée pour les nœuds de calcul et de CNS |

   ![Plage d’adresses IP personnalisée de l’infrastructure de l’offre](media/openshift-marketplace-self-managed/ocp-customiprange.png)  
<br>

**OpenShift Container Platform 3.11**

Entrez les valeurs des paramètres d’entrée et cliquez sur **OK**

| Paramètre d’entrée | Description du paramètre |
|-----------------------|-----------------|
| Mot de passe de l’utilisateur administrateur OpenShift | Mot de passe pour l’utilisateur OpenShift initial.  Cet utilisateur sera également l’administrateur du cluster |
| Confirmez le mot de passe de l’utilisateur administrateur OpenShift | Saisissez à nouveau le mot de passe de l’utilisateur administrateur OpenShift |
| Nom d’utilisateur du gestionnaire d’abonnements Red Hat | Nom d’utilisateur permettant d’accéder à votre abonnement ou votre ID d’organisation Red Hat.  Ces informations d’identification sont utilisées pour inscrire l’instance RHEL à votre abonnement et elles ne seront ne sont pas stockées par Microsoft ou Red Hat |
| Mot de passe utilisateur du gestionnaire d’abonnements Red Hat | Mot de passe permettant d’accéder à votre abonnement ou votre clé d’activation Red Hat.  Ces informations d’identification sont utilisées pour inscrire l’instance RHEL à votre abonnement et elles ne seront ne sont pas stockées par Microsoft ou Red Hat |
| ID de pool OpenShift du gestionnaire d’abonnements Red Hat | ID du pool contenant les droits d’utilisation d’OpenShift Container Platform. Assurez-vous de disposer d’un nombre suffisant de droits sur OpenShift Container Platform pour installer le cluster |
| ID de pool OpenShift du gestionnaire d’abonnements Red Hat pour les nœuds de répartiteur/principaux | ID du pool contenant les droits d’utilisation d’OpenShift Container Platform pour les nœuds de répartiteur/principaux. Assurez-vous de disposer d’un nombre suffisant de droits sur OpenShift Container Platform pour installer le cluster. Si vous n’utilisez pas l’ID de pool pour les nœuds de répartiteur/principaux, entrez l’ID de pool pour les nœuds d’application |
| Configurer le fournisseur de cloud Azure | Configurez OpenShift pour utiliser le fournisseur de cloud Azure. Nécessaire si vous utilisez l’attachement de disque Azure pour les volumes persistants.  La valeur par défaut est Oui |
| GUID de l’ID client du principal de service Azure AD | GUID de l’ID client du principal de service Azure AD, également appelé AppID. Nécessaire si l’option Configurer le fournisseur de cloud Azure est défini sur **Oui** uniquement |
| Secret de l’ID client du principal de service Azure AD | Secret de l’ID client du principal de service Azure AD. Nécessaire si l’option Configurer le fournisseur de cloud Azure est défini sur **Oui** uniquement |
 
   ![Panneau OpenShift de l’offre](media/openshift-marketplace-self-managed/ocp-ocpmain.png)  
<br>

**Paramètres supplémentaires**

Le panneau Paramètres supplémentaires permet de configurer le CNS pour le stockage, la journalisation, les métriques et le sous-domaine de routeur glusterfs.  La configuration par défaut n’installera pas ces options et utilisera nip.io comme sous-domaine de routeur à des fins de test. Activer le CNS installera trois nœuds de calcul supplémentaires ainsi que trois disques attachés supplémentaires qui hébergeront les pods glusterfs.  

Entrez les valeurs des paramètres d’entrée et cliquez sur **OK**

| Paramètre d’entrée | Description du paramètre |
|-----------------------|-----------------|
| Configurer le CNS (Stockage natif de conteneurs) | Installe le CNS dans le cluster OpenShift et l’active en tant que stockage. Sera configuré par défaut si le fournisseur Azure est désactivé |
| Configurer la journalisation de cluster | Installe les fonctionnalités de journalisation EFK dans le cluster.  Dimensionnez correctement les nœuds d’infrastructure de façon à héberger les pods EFK |
| Configurer les métriques pour le cluster | Installe les métriques Hawkular dans le cluster OpenShift.  Dimensionnez correctement les nœuds d’infrastructure de façon à héberger les pods des métriques Hawkular |
| Sous-domaine du routeur par défaut | Sélectionnez nipio pour effectuer des tests ou entrez votre propre sous-domaine personnalisé à des fins de production |
 
   ![Panneau supplémentaire de l’offre](media/openshift-marketplace-self-managed/ocp-additionalmain.png)  
<br>

**Paramètres supplémentaires**

| Paramètre d’entrée | Description du paramètre |
|-----------------------|-----------------|
| Taille de nœud (CNS) | Acceptez la taille de nœud par défaut ou sélectionnez **Modifier la taille** pour sélectionner une nouvelle taille de machine virtuelle |
| Entrer votre sous-domaine personnalisé | Le domaine de routage personnalisé à utiliser pour exposer les applications via le routeur sur le cluster OpenShift.  Assurez-vous de créer l’entrée DNS de caractère générique appropriée |
 
   ![Installation du CNS supplémentaire de l’offre](media/openshift-marketplace-self-managed/ocp-additionalcnsall.png)  
<br>

**Résumé**

La validation se produit à ce stade afin de vérifier que vous disposez d’un quota de cœurs suffisant pour déployer le nombre total de machines virtuelles sélectionnées pour le cluster.  Vérifiez tous les paramètres qui ont été entrés.  Si les entrées sont acceptables, cliquez sur **OK** pour continuer.

   ![Panneau du résumé de l’offre](media/openshift-marketplace-self-managed/ocp-summary.png)  
<br>

**Acheter**

Vérifiez les informations de contact sur la page Acheter et cliquez sur **Achat** pour accepter les conditions d’utilisation et commencer le déploiement du cluster OpenShift Container Platform.

   ![Panneau d’achat de l’offre](media/openshift-marketplace-self-managed/ocp-purchase.png)  
<br>


## <a name="connect-to-the-openshift-cluster"></a>Se connecter au cluster OpenShift

Une fois le déploiement terminé, récupérez la connexion à partir de la section de sortie du déploiement. Connectez-vous à la console OpenShift dans un navigateur à l’aide de l’**URL de la console OpenShift**. Vous pouvez également établir une connexion SSH à l’hôte Bastion. Dans l’exemple suivant, le nom d’utilisateur administrateur est clusteradmin et le nom de domaine complet du DNS d’adresses IP publiques Bastion est bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com :

```bash
$ ssh clusteradmin@bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Utilisez la commande [az group delete](/cli/azure/group) pour supprimer le groupe de ressources, le cluster OpenShift et toutes les ressources associées quand vous n’en avez plus besoin.

```azurecli 
az group delete --name openshiftrg
```

## <a name="next-steps"></a>Étapes suivantes

- [Tâches de post-déploiement](./openshift-container-platform-3x-post-deployment.md)
- [Résoudre les problèmes de déploiement d’OpenShift dans Azure](./openshift-container-platform-3x-troubleshooting.md)
- [Prise en main d’OpenShift Container Platform](https://docs.openshift.com)
- 
