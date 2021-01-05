---
title: 'Procédure : Exécuter une application avec Fortanix Enclave Manager'
description: Apprenez à utiliser Fortanix Enclave Manager pour convertir vos images conteneurisées
services: virtual-machines
author: JBCook
ms.service: virtual-machines
ms.subservice: workloads
ms.workload: infrastructure
ms.topic: how-to
ms.date: 8/12/2020
ms.author: JenCook
ms.openlocfilehash: b2c266d31a3aacc5bc97434de2246bfb1285a1ee
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94565702"
---
# <a name="how-to-run-an-application-with-fortanix-enclave-manager"></a>Procédure : Exécuter une application avec Fortanix Enclave Manager 

Commencez à exécuter votre application dans l'informatique confidentielle Azure en utilisant les outils [Fortanix Enclave Manager](https://azuremarketplace.microsoft.com/marketplace/apps/fortanix.enclave_manager?tab=Overview) et [Fortanix Node Agent](https://azuremarketplace.microsoft.com/marketplace/apps/fortanix.rte_node_agent) de [Fortanix](https://www.fortanix.com/).


Fortanix est un fournisseur de logiciels tiers dont les produits et services reposent sur l'infrastructure Azure. D'autres fournisseurs tiers proposent des services informatiques confidentiels similaires sur Azure.

> [!Note] 
 > Microsoft n’a aucun contrôle sur les produits référencés dans ce document. Microsoft vous fournit ces informations uniquement à titre de commodité, et la référence à ces produits non Microsoft n’implique aucune approbation de la part de Microsoft.



Ce tutoriel explique comment convertir l'image de votre application en image confidentielle protégée. Cet environnement utilise des logiciels [​​Fortanix](https://www.fortanix.com/), optimisés par des machines virtuelles Intel compatibles SGX de la série DCsv2 d'Azure. Cette solution orchestre les stratégies de sécurité critiques telles que la vérification d'identité et le contrôle d'accès aux données.

 Pour un support Fortanix spécifique, rejoignez la communauté [Fortanix Slack](https://fortanix.com/community/) et utilisez le canal #enclavemanager.


## <a name="prerequisites"></a>Prérequis

1. Si vous ne possédez pas de compte Fortanix Enclave Manager, [inscrivez-vous](https://em.fortanix.com/auth/sign-up) avant de commencer.
1. Un registre [Docker](https://docs.docker.com/) privé pour transmettre les images des applications converties.
1. Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/) avant de commencer.

> [!NOTE]
> Les comptes associés à un essai gratuit n’ont pas accès aux machines virtuelles utilisées dans ce tutoriel. Veuillez passer à un abonnement avec paiement à l’utilisation.

## <a name="add-an-application-to-fortanix-enclave-manager"></a>Ajouter une application à Fortanix Enclave Manager
1. Connectez-vous à [Fortanix enclave Manager (Fortanix EM)](https://em.fortanix.com)
1. Accédez à la page **Comptes** et sélectionnez **AJOUTER UN COMPTE** pour créer un compte. 
    
![Créer un compte](media/how-to-fortanix-enclave-manager/create-account.png)

1. Une fois votre compte créé, appuyez sur **SÉLECTIONNER** pour sélectionner le nouveau compte. Nous pouvons maintenant commencer à inscrire les nœuds de calcul et à créer des applications. 
1. Sélectionnez le bouton **+ APPLICATION** pour ajouter une application. Dans cet exemple, nous allons ajouter une application Flask Server Enclave OS. 

1. Sélectionnez le bouton **AJOUTER** pour l'application Enclave OS. 

    ![Ajouter l’application](media/how-to-fortanix-enclave-manager/add-enclave-application.png)

    > [!NOTE]
    > Ce tutoriel couvre uniquement l'ajout d'applications Enclave OS. [Découvrez-en plus](https://support.fortanix.com/hc/en-us/articles/360044746932-Bringing-EDP-Rust-Apps-to-Enclave-Manager) sur l'intégration d'applications EDP Rust dans Fortanix Enclave Manager. 

6. Dans ce tutoriel, nous allons utiliser le registre Docker de Fortanix pour l'exemple d'application. Renseignez les détails à partir des informations suivantes. Utilisez votre registre Docker privé pour conserver l'image de sortie. 
 
    - **Nom de l'application** : Python Application Server
    - **Description** : Serveur Python Flask
    - **Nom de l'image d'entrée** : fortanix/python-flask
    - **Nom de l'image de sortie** : fortanx-private/python-flask-sgx
    - **ISVPRODID** : 1
    - **ISVSVM** : 1
    - **Taille de la mémoire** : 1 Go
    - **Nombre de Threads** : 128

    *Facultatif* : Exécutez l'application.
    - **Docker Hub** : [https://hub.docker.com/u/fortanix](https://hub.docker.com/u/fortanix)
    - **Application** : fortanix/python-flask

        Exécutez la commande suivante :
         ```bash
            sudo docker run fortanix/python-flask
         ```

1. Ajoutez un certificat. Renseignez les informations à l'aide des détails ci-dessous, puis sélectionnez **SUIVANT** :
    - **Domaine** : myapp.domain.dom
    - **Type** : Certificat émis par Enclave Manager 
    - **Chemin de la clé** : /appkey.pem
    - **Type de clé** : RSA
    - **Chemin du certificat** : /appcert.pem
    - **Taille de clé RSA** : 2048 bits
    

## <a name="create-an-image"></a>Créer une image
Une image Fortanix EM est une version d'un logiciel ou d'une application. Chaque image est associée à un code de hachage d'enclave (MRENCLAVE). 
1. Sur la page **Ajouter une image**, entrez les **INFORMATIONS D'IDENTIFICATION DU REGISTRE** pour le **Nom de l'image de sortie**. Ces informations d'identification sont utilisées pour accéder au registre Docker privé dans lequel l'image sera envoyée. 

    ![Créer une image](media/how-to-fortanix-enclave-manager/create-image.png)
1. Fournissez l'étiquette de l'image et sélectionnez **Créer**.

    ![Ajouter une étiquette](media/how-to-fortanix-enclave-manager/add-tag.png)

## <a name="domain-and-image-allow-listing"></a>Liste verte de domaines et d'images 
Une application dont le domaine est ajouté à la liste verte reçoit un certificat TLS de Fortanix Enclave Manager. De même, lorsqu'une application s'exécute à partir de l'image convertie, elle essaie de contacter Fortanix Enclave Manager. L'application demande ensuite un certificat TLS. 

Accédez à l'onglet **Tâches**, à gauche, et approuvez les demandes en attente pour autoriser le domaine et l'image. 

## <a name="enroll-compute-node-agent-in-azure"></a>Inscrire l'agent du nœud de calcul dans Azure

### <a name="generate-and-copy-join-token"></a>Générer et copier un jeton Join
Dans Fortanix Enclave Manager, vous allez créer un jeton. Ce jeton permettra à un nœud de calcul Azure de s'authentifier. Vous devez fournir ce jeton à votre machine virtuelle Azure.
1. Dans la console de gestion, sélectionnez le bouton **+ INSCRIRE LE NŒUD**. 
1. Sélectionnez **GÉNÉRER LE JETON** pour générer le jeton Join. Copiez le jeton.

### <a name="enroll-nodes-into-fortanix-node-agent-in-azure-marketplace"></a>Inscrire des nœuds dans Fortanix Node Agent sur la Place de marché Azure

La création d'une instance de Fortanix Node Agent déploiera une machine virtuelle, une interface réseau, un réseau virtuel, un groupe de sécurité réseau et une adresse IP publique dans votre groupe de ressources Azure. Votre abonnement Azure sera facturé à l'heure pour la machine virtuelle. Avant de créer une instance de Fortanix Node Agent, consultez la [page de tarification des machines virtuelles](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) Azure de la série DCsv2. Supprimez les ressources Azure inutilisées. 

1. Accédez à la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/) et connectez-vous à l'aide de vos informations d'identification Azure.
1. Sur la barre de recherche, entrez **Fortanix Confidential Computing Node Agent**. Sélectionnez l'application **Fortanix Confidential Computing Node Agent** qui apparaît dans la zone de recherche pour accéder à la page d'accueil de l'offre. 
     ![Rechercher sur la Place de marché](media/how-to-fortanix-enclave-manager/search-fortanix-marketplace.png)
1. Sélectionnez **Obtenir maintenant**, si nécessaire entrez vos informations, puis sélectionnez **Continuer**. Vous êtes redirigé vers le portail Azure. 
1. Sélectionnez **Créer** pour accéder à la page de déploiement de Fortanix Confidential Computing Node Agent.
1. Sur cette page, vous allez entrer les informations nécessaires au déploiement d'une machine virtuelle. Plus précisément, il s'agit d'une machine virtuelle Intel compatible SGX de la série DCsv2 d'Azure sur laquelle le logiciel Fortanix Node Agent est installé. Node Agent permettra à votre image convertie de s'exécuter en toute sécurité sur les nœuds Intel SGX dans Azure.  Sélectionnez l'**abonnement** et le **groupe de ressources** dans lesquels vous souhaitez déployer la machine virtuelle et les ressources associées. 
 
    > [!NOTE]
    > Des contraintes s'appliquent au déploiement des machines virtuelles de la série DCsv2 dans Azure. Vous devrez peut-être demander un quota pour bénéficier de cœurs supplémentaires. Pour plus d'informations, consultez [Solutions informatiques confidentielles sur les machines virtuelles Azure](./virtual-machine-solutions.md). 

1. Sélectionnez une région disponible.
1. Dans le champ **Nom du nœud**, entrez un nom pour votre machine virtuelle. 
1. Entrez un nom d'utilisateur et un mot de passe (ou une clé SSH) pour vous authentifier sur la machine virtuelle.
1. Conservez la taille par défaut (200) du disque du système d'exploitation et sélectionnez une taille de machine virtuelle (Standard_DC4s_v2 suffira pour ce tutoriel).
1. Collez le jeton généré précédemment dans le champ **Jeton Join**.

     ![déployer des ressources](media/how-to-fortanix-enclave-manager/deploy-fortanix-node-agent.png)

1. Sélectionnez **Vérifier + créer**. Assurez-vous que la validation est réussie, puis sélectionnez **Créer**. Une fois toutes les ressources déployées, le nœud de calcul est inscrit dans Enclave Manager. 

## <a name="run-the-application-image-on-the-compute-node"></a>Exécuter l'image de l'application sur le nœud de calcul
Exécutez l'application à l'aide de la commande suivante. Veillez à transformer l'adresse IP du nœud, le port et le nom de l'image convertie en entrées pour votre application spécifique. 
 
Dans ce tutoriel, la commande à exécuter est la suivante :     

```bash
    sudo docker run `
        --device /dev/isgx:/dev/isgx `
        --device /dev/gsgx:/dev/gsgx `
        -v /var/run/aesmd/aesm.socket:/var/run/aesmd/aesm.socket `
        -e NODE_AGENT_BASE_URL=http://52.152.206.164:9092/v1/ fortanix-private/python-flask-sgx
```

où : 
- *52.152.206.164* correspond à l'adresse IP de l'hôte Node Agent
- *9092* correspond au port que Node Agent écoute
- *fortanix-private/python-flask-sgx* correspond à l'application convertie qui se trouve sous l'onglet Images, dans la colonne **Nom de l'image** du tableau **Images** sur le portail web de Fortanix Enclave Manager. 
    
## <a name="verify-and-monitor-the-running-application"></a>Vérifier et surveiller l'application en cours d'exécution
1. Revenez à [Fortanix Enclave Manager](https://em.fortanix.com/console).
1. Vérifiez que vous utilisez le **Compte** dans lequel vous avez inscrit le nœud.
1. Accédez à la **console de gestion** en sélectionnant l'icône supérieure dans le volet de navigation de gauche. 
1. Sélectionnez l'onglet **Application**.
1. Vérifiez qu'une application est en cours d'exécution avec un nœud de calcul associé.


## <a name="clean-up-resources"></a>Nettoyer les ressources

Dès que vous n'en avez plus besoin, vous pouvez supprimer le groupe de ressources, la machine virtuelle et les ressources associées. La suppression du groupe de ressources annulera l'inscription des nœuds associés à votre image convertie. 

Sélectionnez le groupe de ressources de la machine virtuelle, puis sélectionnez **Supprimer**. Confirmez le nom du groupe de ressources pour terminer la suppression des ressources.

Pour supprimer le compte Fortanix Enclave Manager que vous avez créé, accédez à la [page Comptes](https://em.fortanix.com/accounts) d'Enclave Manager. Pointez sur le compte que vous souhaitez supprimer. Sélectionnez les points noirs verticaux dans le coin supérieur droit, puis sélectionnez **Supprimer le compte**.

  ![supprimer](media/how-to-fortanix-enclave-manager/delete-account.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez utilisé les outils Fortanix pour convertir l'image de votre application afin qu'elle s'exécute sur une machine virtuelle de l'informatique confidentielle. Pour plus d’informations sur les machines virtuelles d’informatique confidentielle sur Azure, consultez [Solutions sur les machines virtuelles](virtual-machine-solutions.md). 

Pour en savoir plus sur les offres d'Azure en matière d'informatique confidentielle, consultez [Vue d'ensemble de l'informatique confidentielle Azure](overview.md).

 Apprenez à effectuer des tâches similaires à l'aide d'autres offres de tiers disponibles sur Azure, comme [Anjuna](https://azuremarketplace.microsoft.com/marketplace/apps/anjuna-5229812.aee-az-v1) et [Scone](https://sconedocs.github.io).