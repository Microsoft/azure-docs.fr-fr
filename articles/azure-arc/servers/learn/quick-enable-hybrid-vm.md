---
title: Connecter une machine hybride avec Azure Arc enabled servers
description: Découvrez comment connecter et inscrire votre machine hybride à l’aide d’Azure Arc enabled servers.
ms.topic: quickstart
ms.date: 12/15/2020
ms.openlocfilehash: 68869854cbfcf6d7297137e6239b2229a20c04a1
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516791"
---
# <a name="quickstart-connect-hybrid-machine-with-azure-arc-enabled-servers"></a>Démarrage rapide : Connecter une machine hybride avec Azure Arc enabled servers

[Azure Arc enabled servers](../overview.md) vous permet de gérer et contrôler vos machines Windows et Linux hébergées dans des environnements locaux, périphériques et multiclouds. Dans ce guide de démarrage rapide, vous allez déployer et configurer l’agent Connected Machine sur votre machine Windows ou Linux hébergée en dehors d’Azure pour permettre la gestion par Arc enabled servers.

## <a name="prerequisites"></a>Prérequis

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

* Pour déployer l’agent Hybrid Connected Machine d’Arc enabled servers, vous devez disposer d’autorisations d’administrateur sur la machine où installer et configurer l’agent. Vous utilisez le compte racine sous Linux et, sous Windows, un compte membre du groupe Administrateurs local.

* Avant de commencer, veillez à passer en revue les [conditions préalables](../agent-overview.md#prerequisites) de l’agent et à vérifier les éléments suivants :

    * Votre machine cible exécute un [système d’exploitation](../agent-overview.md#supported-operating-systems) pris en charge.

    * Votre compte est affecté aux [rôles Azure requis](../agent-overview.md#required-permissions).

    * Si la machine se connecte via un pare-feu ou un serveur proxy pour communiquer sur Internet, assurez-vous que les URL [répertoriées](../agent-overview.md#networking-configuration) ne sont pas bloquées.

    * Azure Arc enabled servers prend uniquement en charge les régions spécifiées [ici](../overview.md#supported-regions).

> [!WARNING]
> Le nom d’hôte Linux ou le nom de l’ordinateur Windows ne peuvent pas contenir de mots réservés ou de marques dans le nom. Dans le cas contraire, la tentative d’inscription de la machine connectée auprès d’Azure se solde par un échec. Consultez [Résoudre les erreurs de nom de ressource réservé](../../../azure-resource-manager/templates/error-reserved-resource-name.md) pour obtenir la liste des mots réservés.

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

## <a name="register-azure-resource-providers"></a>Inscrire des fournisseurs de ressources Azure

Pour utiliser ce service, les serveurs activés par Azure Arc dépendent des fournisseurs de ressources Azure suivants dans votre abonnement :

* Microsoft.HybridCompute
* Microsoft.GuestConfiguration

Enregistrez-les à l’aide des commandes suivantes :

```azurecli-interactive
az account set --subscription "{Your Subscription Name}"
az provider register --namespace 'Microsoft.HybridCompute'
az provider register --namespace 'Microsoft.GuestConfiguration'
```

## <a name="generate-installation-script"></a>Générer un script d’installation

Le script permettant d’automatiser le téléchargement et l’installation, ainsi que d’établir la connexion avec Azure Arc, est disponible sur le portail Azure. Pour effectuer le processus, procédez comme suit :

1. Lancez le service Azure Arc dans le portail Azure en cliquant sur **Tous les services**, puis en recherchant et sélectionnant **Serveurs - Azure Arc**.

    :::image type="content" source="./media/quick-enable-hybrid-vm/search-machines.png" alt-text="Rechercher des serveurs avec Azure Arc dans Tous les services" border="false":::

1. Dans la page **Serveurs - Azure Arc**, sélectionnez **Ajouter** en haut à gauche.

1. Dans la page **Sélectionner une méthode**, sélectionnez la vignette **Ajouter des serveurs à l’aide d’un script interactif**, puis sélectionnez **Générer un script**.

1. Dans la page **Générer un script**, sélectionnez l’abonnement et le groupe de ressources où vous souhaitez que la machine soit gérée dans Azure. Sélectionnez une localisation Azure destinée au stockage des métadonnées de la machine. Cet emplacement peut être identique ou différent de l’emplacement du groupe de ressources.

1. Dans la page **Prérequis**, passez en revue les informations, puis sélectionnez **Suivant : Détails des ressources**.

1. Dans la page **Détails des ressources**, spécifiez les informations suivantes :

    1. Dans la liste déroulante **Groupe de ressources**, sélectionnez le groupe de ressources à partir duquel sera gérée la machine.
    1. Dans la liste déroulante **Région**, sélectionnez la région Azure dans laquelle seront stockées les métadonnées des serveurs.
    1. Dans la liste déroulante **Système d’exploitation**, sélectionnez le système d’exploitation sur lequel le script doit s’exécuter.
    1. Si la machine passe par un serveur proxy pour se connecter à Internet, spécifiez l’adresse IP du serveur proxy ou le nom et le numéro de port que la machine doit utiliser pour communiquer avec le serveur proxy. Saisissez la valeur au format `http://<proxyURL>:<proxyport>`.
    1. Sélectionnez **Suivant : Balises**.

1. Dans la page **Balises**, passez en revue les suggestions de **balises d’emplacement physique** par défaut et entrez une valeur, ou spécifiez une ou plusieurs **Balises personnalisées** en fonction de vos standards.

1. Sélectionnez **Suivant : Télécharger et exécuter le script**.

1. Dans la page **Télécharger et exécuter le script**, passez en revue les informations de résumé, puis sélectionnez **Télécharger**. Si vous avez encore besoin d’apporter des modifications, sélectionnez **Précédent**.

## <a name="install-the-agent-using-the-script"></a>Installer l’agent à l’aide du script

### <a name="windows-agent"></a>Agent Windows

1. Connectez-vous au serveur.

1. Ouvrez une invite de commandes PowerShell 64 bits avec élévation de privilèges.

1. Accédez au dossier ou au partage sur lequel vous avez copié le script et exécutez-le sur le serveur en exécutant le script `./OnboardingScript.ps1`.

### <a name="linux-agent"></a>Agent Linux

1. Pour installer l’agent Linux sur la machine cible pouvant directement communiquer avec Azure, exécutez les commandes suivantes :

    ```bash
    bash ~/Install_linux_azcmagent.sh
    ```

    * Si la machine cible communique par le biais d’un serveur proxy, exécutez la commande suivante :

        ```bash
        bash ~/Install_linux_azcmagent.sh --proxy "{proxy-url}:{proxy-port}"
        ```

## <a name="verify-the-connection-with-azure-arc"></a>Vérifier la connexion avec Azure Arc

Une fois que vous avez installé l’agent et que vous l’avez configuré pour qu’il se connecte à Azure Arc enabled servers, accédez au portail Azure pour vérifier que le serveur s’est correctement connecté. Affichez vos machines dans le [portail Azure](https://aka.ms/hybridmachineportal).

:::image type="content" source="./media/quick-enable-hybrid-vm/enabled-machine.png" alt-text="Une connexion de machine réussie" border="false":::

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez activé votre machine hybride Linux ou Windows et que vous vous êtes connecté au service avec succès, vous pouvez activer Azure Policy pour comprendre la conformité dans Azure.

Pour savoir comment identifier la machine activée Azure Arc enabled servers sur laquelle l’agent Log Analytics n’est pas installé, suivez le tutoriel :

> [!div class="nextstepaction"]
> [Créer une affectation de stratégie pour identifier les ressources non conformes](tutorial-assign-policy-portal.md)
