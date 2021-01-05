---
title: Configurer GitHub Enterprise Server sur votre cloud privé Azure VMware Solution
description: Découvrez comment configurer GitHub Enterprise Server sur votre cloud privé Azure VMware Solution.
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 00b3acf721dd7f7a1a15bcd0d24eccf3ca27ff58
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96326911"
---
# <a name="set-up-github-enterprise-server-on-your-azure-vmware-solution-private-cloud"></a>Configurer GitHub Enterprise Server sur votre cloud privé Azure VMware Solution

Dans cet article, nous allons suivre les étapes de configuration de GitHub Enterprise Server, la version « locale » de [GitHub.com](https://github.com/), sur votre cloud privé Azure VMware Solution. Le scénario abordé dans cette procédure pas à pas est destiné à une instance GitHub Enterprise Server pouvant servir jusqu’à 3 000 développeurs exécutant jusqu’à 25 travaux par minute sur GitHub Actions. Il comprend l’installation (au moment de la rédaction de cet article) des *fonctionnalités d’évaluation*, telles que GitHub Actions. Pour personnaliser la configuration en fonction de vos besoins spécifiques, consultez la configuration requise indiquée dans [Installing GitHub Enterprise Server on VMware](https://docs.github.com/en/enterprise/admin/installation/installing-github-enterprise-server-on-vmware#hardware-considerations) (Installation de GitHub Enterprise Server sur VMware).

## <a name="before-you-begin"></a>Avant de commencer

GitHub Enterprise Server nécessite une clé de licence valide. Vous pouvez vous inscrire pour obtenir une [licence d’essai](https://enterprise.github.com/trial). Si vous souhaitez étendre les capacités de GitHub Enterprise Server par le biais d’une intégration, vous pouvez bénéficier d’une licence de développement gratuite pour cinq postes. Appliquez cette licence par le biais du [programme partenaire de GitHub](https://partner.github.com/).

## <a name="installing-github-enterprise-server-on-vmware"></a>Installation de GitHub Enterprise Server sur VMware

Téléchargez [la version actuelle de GitHub Enterprise Server](https://enterprise.github.com/releases/2.19.0/download) pour VMware ESXi/vSphere (OVA) et [déployez le modèle OVA](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-17BEDA21-43F6-41F4-8FB2-E01D275FE9B4.html) que vous avez téléchargé.

:::image type="content" source="media/github-enterprise-server/github-options.png" alt-text="Choisir d’exécuter GitHub localement ou dans le cloud.":::  

:::image type="content" source="media/github-enterprise-server/deploy-ova-template.png" alt-text="Déployer le modèle OVA.":::  

Donnez un nom reconnaissable à votre nouvelle machine virtuelle, par exemple GitHubEnterpriseServer. Vous n’avez pas besoin d’inclure les détails de la version dans le nom de la machine virtuelle, car ces détails deviennent obsolètes lorsque l’instance est mise à niveau. Sélectionnez toutes les valeurs par défaut pour l’instant (nous modifierons ces détails prochainement) et attendez que l’OVA soit importé.

Une fois l’importation terminée, [ajustez la configuration matérielle](https://docs.github.com/en/enterprise/admin/installation/installing-github-enterprise-server-on-vmware#creating-the-github-enterprise-server-instance) en fonction de vos besoins. Dans notre exemple de scénario, nous avons besoin de la configuration suivante.

| Ressource | Installation standard | Installation standard + « fonctionnalités bêta » (Actions) |
| --- | --- | --- |
| Processeurs virtuels | 4 | 8 |
| Mémoire | 32 Go | 61 Go |
| Stockage associé | 250 Go | 300 Go |
| Stockage racine | 200 Go | 200 Go |

Toutefois, vos besoins peuvent varier. Reportez-vous aux conseils relatifs aux considérations matérielles [Installing GitHub Enterprise Server on VMware](https://docs.github.com/en/enterprise/admin/installation/installing-github-enterprise-server-on-vmware#hardware-considerations) (Installation de GitHub Enterprise Server sur VMware). Consultez également [Adding CPU or memory resources for VMware](https://docs.github.com/en/enterprise/admin/enterprise-management/increasing-cpu-or-memory-resources#adding-cpu-or-memory-resources-for-vmware) (Ajout de ressources de processeur ou mémoire pour VMware) pour personnaliser la configuration matérielle en fonction de votre situation.

## <a name="configuring-the-github-enterprise-server-instance"></a>Configuration de l’instance GitHub Enterprise Server

:::image type="content" source="media/github-enterprise-server/install-github-enterprise.png" alt-text="Installer GitHub Enterprise.":::  

Une fois que la machine virtuelle nouvellement approvisionnée a été mise sous tension, [configurez-la via votre navigateur](https://docs.github.com/en/enterprise/admin/installation/installing-github-enterprise-server-on-vmware#configuring-the-github-enterprise-server-instance). Il vous sera demandé de charger votre fichier de licence et de définir un mot de passe pour la console de gestion. Veillez à noter ce mot de passe dans un endroit sûr.

:::image type="content" source="media/github-enterprise-server/ssh-access.png" alt-text="Accéder à l’interpréteur de commandes d’administration via SSH.":::    

Nous vous recommandons d’effectuer au moins les étapes suivantes :

1. Téléchargez une clé SSH publique dans la console de gestion pour pouvoir [accéder à l’interpréteur de commandes d’administration via SSH](https://docs.github.com/en/enterprise/admin/configuration/accessing-the-administrative-shell-ssh). 

2. [Configurez le protocole TLS sur votre instance](https://docs.github.com/en/enterprise/admin/configuration/configuring-tls) pour pouvoir utiliser un certificat signé par une autorité de certification approuvée.

:::image type="content" source="media/github-enterprise-server/configuring-your-instance.png" alt-text="Configuration de votre instance.":::

Appliquez vos paramètres.  Pendant le redémarrage de l’instance, vous pouvez passer à l’étape suivante, à savoir **Configuration de Stockage Blob pour GitHub Actions**.

:::image type="content" source="media/github-enterprise-server/create-admin-account.png" alt-text="Créer votre compte administrateur.":::

Une fois l’instance redémarrée, créez un nouveau compte administrateur sur l’instance. Veillez à noter également le mot de passe de cet utilisateur.

### <a name="additional-configuration-steps"></a>Étapes de configuration supplémentaires

Pour renforcer votre instance à des fins de production, il est recommandé d’effectuer les étapes de configuration facultatives suivantes :

1. Configurez [la haute disponibilité](https://help.github.com/enterprise/admin/guides/installation/configuring-github-enterprise-for-high-availability/) pour la protection contre les incidents suivants :

    - Pannes logicielles (au niveau du système d’exploitation ou de l’application)
    - Défaillances matérielles (stockage, UC, RAM, etc.)
    - Défaillances du système hôte de virtualisation
    - Réseau rompu logiquement ou physiquement

2. [Configurez](https://docs.github.com/en/enterprise/admin/configuration/configuring-backups-on-your-appliance) [des utilitaires de sauvegarde](https://github.com/github/backup-utils) en fournissant des instantanés de version pour la récupération d’urgence hébergés dans une disponibilité distincte de l’instance principale.
3. [Configurez l’isolement de sous-domaine](https://docs.github.com/en/enterprise/admin/configuration/enabling-subdomain-isolation) en utilisant un certificat TLS valide pour limiter le scripting inter-site et autres vulnérabilités associées.

## <a name="configuring-blob-storage-for-github-actions"></a>Configuration de Stockage Blob pour GitHub Actions

> [!NOTE]
> GitHub Actions est [actuellement disponible en tant que version bêta limitée sur GitHub Enterprise Server version 2.22](https://docs.github.com/en/enterprise/admin/github-actions).

Le stockage blob externe est nécessaire pour activer GitHub Actions sur GitHub Enterprise Server (actuellement disponible en tant que fonctionnalité « bêta »). Ce stockage blob externe est utilisé par Actions pour stocker des artefacts et des journaux. Actions sur GitHub Enterprise Server [prend en charge Stockage Blob Azure en tant que fournisseur de stockage](https://docs.github.com/en/enterprise/admin/github-actions/enabling-github-actions-and-configuring-storage#about-external-storage-requirements) (et d’autres). Nous allons donc configurer un nouveau compte de stockage Azure avec le [type de compte de stockage](../storage/common/storage-account-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#types-of-storage-accounts) BlobStorage :

:::image type="content" source="media/github-enterprise-server/storage-account.png" alt-text="Configurer un compte Stockage Blob Azure.":::

Une fois le déploiement de la nouvelle ressource BlobStorage terminé, copiez et notez la chaîne de connexion (disponible sous Clés d’accès). Nous aurons besoin de cette chaîne dans un instant.

## <a name="setting-up-the-github-actions-runner"></a>Configuration de l’exécuteur GitHub Actions

> [!NOTE]
> GitHub Actions est [actuellement disponible en tant que version bêta limitée sur GitHub Enterprise Server version 2.22](https://docs.github.com/en/enterprise/admin/github-actions).

À ce stade, vous devez avoir une instance GitHub Enterprise Server en état de fonctionnement, avec un compte administrateur créé. Vous devez également disposer d’un stockage blob externe qui sera utilisé par GitHub Actions pour la persistance.

Créons à présent un emplacement où exécuter GitHub Actions. Là encore, nous allons utiliser Azure VMware Solution.

Tout d’abord, nous allons configurer une nouvelle machine virtuelle sur le cluster. Nous allons baser notre machine virtuelle sur [une version récente de Ubuntu Server](http://releases.ubuntu.com/20.04.1/).

:::image type="content" source="media/github-enterprise-server/provision-new-vm.png" alt-text="Configurer une nouvelle machine virtuelle.":::

:::image type="content" source="media/github-enterprise-server/provision-new-vm-2.png" alt-text="Configurer une nouvelle machine virtuelle 2.":::

Une fois la machine virtuelle créée, mettez-la sous tension et connectez-vous via SSH.

Ensuite, installez l’application de [l’exécuteur Actions](https://github.com/actions/runner), qui exécute un travail à partir d’un workflow GitHub Actions. Identifiez et téléchargez la version Linux x64 la plus récente de l’exécuteur Actions, soit à partir de [la page des versions](https://github.com/actions/runner/releases), soit en exécutant le script rapide suivant. Ce script nécessite la présence de curl et [jq](https://stedolan.github.io/jq/) sur votre machine virtuelle.

`LATEST\_RELEASE\_ASSET\_URL=$( curl https://api.github.com/repos/actions/runner/releases/latest | \`

`  jq -r '.assets | .[] | select(.name | match("actions-runner-linux-arm64")) | .url' )`

`DOWNLOAD\_URL=$( curl $LATEST\_RELEASE\_ASSET\_URL | \`

`  jq -r '.browser\_download\_url' )`

`curl -OL $DOWNLOAD\_URL`

Vous devez maintenant disposer d’un fichier local sur votre machine virtuelle, actions-runner-linux-arm64-\*.tar.gz. Extrayez ce tarball localement :

`tar xzf actions-runner-linux-arm64-\*.tar.gz`

Cette extraction décompresse quelques fichiers localement, notamment les scripts `config.sh` et `run.sh`, sur lesquels nous reviendrons bientôt.

## <a name="enabling-github-actions"></a>Activation de GitHub Actions

> [!NOTE]
> GitHub Actions est [actuellement disponible en tant que version bêta limitée sur GitHub Enterprise Server version 2.22](https://docs.github.com/en/enterprise/admin/github-actions).

Vous avez presque fini ! Configurons et activons GitHub Actions sur l’instance GitHub Enterprise Server. Nous devrons [accéder à l’interpréteur de commandes d’administration de l’instance GitHub Enterprise Server via SSH](https://docs.github.com/en/enterprise/admin/configuration/accessing-the-administrative-shell-ssh), puis exécuter les commandes suivantes :

`# set an environment variable containing your Blob storage connection string`

`export CONNECTION\_STRING="<your connection string from the blob storage step>"`

`# configure actions storage`

`ghe-config secrets.actions.storage.blob-provider azure`

`ghe-config secrets.actions.storage.azure.connection-string "$CONNECTION\_STRING"`

`# apply these settings`

`ghe-config-apply`

`# execute a precheck, this install additional software required by Actions on GitHub Enterprise Server`

`ghe-actions-precheck -p azure -cs "$CONNECTION\_STRING"`

`# enable actions, and re-apply the config`

`ghe-config app.actions.enabled true`

`ghe-config-apply`

Exécution suivante :

`ghe-actions-check -s blob`

Cette sortie doit s’afficher : « Stockage Blob est sain ».

Maintenant que la fonctionnalité GitHub Actions est configurée, activez-la pour vos utilisateurs. Connectez-vous à votre instance GitHub Enterprise Server en tant qu’administrateur, puis sélectionnez l’icône ![Rocket icon](media/github-enterprise-server/rocket-icon.png) dans l’angle supérieur droit de n’importe quelle page. Dans la barre latérale gauche, sélectionnez **Vue d’ensemble de l’entreprise**, puis **Stratégies**, **Actions**, et sélectionnez l’option permettant d’**activer Actions pour toutes les organisations**.

Ensuite, configurez votre exécuteur à partir de l’onglet **Exécuteurs auto-hébergés**. Sélectionnez **Ajouter nouveau** puis **Nouvel exécuteur** dans la liste déroulante.

Sur la page suivante, vous verrez un ensemble de commandes à exécuter. Il suffit de copier la commande pour **configurer** l’exécuteur, par exemple :

`./config.sh --url https://10.1.1.26/enterprises/octo-org --token AAAAAA5RHF34QLYBDCHWLJC7L73MA`

Copiez la commande `config.sh` et collez-la dans une session sur votre exécuteur Actions (créé précédemment).

:::image type="content" source="media/github-enterprise-server/actions-runner.png" alt-text="Exécuteur Actions.":::

Utilisez la commande run.sh pour *exécuter* l’exécuteur :

:::image type="content" source="media/github-enterprise-server/run-runner.png" alt-text="Exécuter l’exécuteur.":::

Pour mettre cet exécuteur à la disposition des organisations de votre entreprise, modifiez son accès aux organisations :

:::image type="content" source="media/github-enterprise-server/edit-runner-access.png" alt-text="Modifier l’accès de l’exécuteur.":::

Nous le mettrons ici à la disposition de toutes les organisations, mais vous pouvez également limiter l’accès à un sous-ensemble d’organisations, voire à des référentiels spécifiques.

## <a name="optional-configuring-github-connect"></a>(Facultatif) Configuration de GitHub Connect

Bien que cette étape soit facultative, nous vous recommandons de l’utiliser si vous envisagez de consommer des actions open source disponibles sur GitHub.com. Cela vous permet de vous appuyer sur le travail des autres en référençant ces actions réutilisables dans vos workflows.

Pour activer GitHub Connect, suivez les étapes décrites dans [Enabling automatic access to GitHub.com actions using GitHub Connect](https://docs.github.com/en/enterprise/admin/github-actions/enabling-automatic-access-to-githubcom-actions-using-github-connect) (Activation de l’accès automatique aux actions de GitHub.com en utilisant GitHub Connect).

Une fois que GitHub Connect est activé, sélectionnez l’option **Serveur permettant d’utiliser des actions de GitHub.com dans des exécutions de workflow**.

:::image type="content" source="media/github-enterprise-server/enable-using-actions.png" alt-text="Activer l’utilisation des actions de GitHub.com dans les exécutions de workflow.":::

## <a name="setting-up-and-running-your-first-workflow"></a>Configuration et exécution de votre premier workflow

Maintenant que les fonctionnalités GitHub Actions et GitHub Connect sont configurées, mettons tout ce travail à profit. Voici un exemple de workflow qui fait référence à l’excellent [octokit/request-action](https://github.com/octokit/request-action), qui nous permet de « scripter » GitHub par le biais d’interactions à l’aide de l’API GitHub, alimentée par GitHub Actions.

Dans ce workflow de base, nous allons utiliser `octokit/request-action` pour simplement soumettre un problème sur GitHub à l’aide de l’API.

:::image type="content" source="media/github-enterprise-server/workflow-example.png" alt-text="Exemple de workflow.":::

>[!NOTE]
>GitHub.com héberge l’action, mais lorsque celle-ci s’exécute sur GitHub Enterprise Server, elle utilise *automatiquement* l’API de GitHub Enterprise Server.

Si vous choisissez de ne pas activer GitHub Connect, vous pouvez utiliser l’autre workflow suivant.

:::image type="content" source="media/github-enterprise-server/workflow-example-2.png" alt-text="Autre exemple de workflow.":::

Accédez à un référentiel sur votre instance et ajoutez le workflow ci-dessus en tant que : `.github/workflows/hello-world.yml`.

:::image type="content" source="media/github-enterprise-server/workflow-example-3.png" alt-text="Autre exemple de workflow.":::

Dans l’onglet **Actions** pour votre référentiel, attendez que le workflow s’exécute.

:::image type="content" source="media/github-enterprise-server/executed-example-workflow.png" alt-text="Exemple de workflow exécuté.":::

Vous pouvez également observer son traitement par l’exécuteur.

:::image type="content" source="media/github-enterprise-server/workflow-processed-by-runner.png" alt-text="Workflow traité par un exécuteur.":::

Si tout s’est bien passé, vous devriez voir un nouveau problème dans votre référentiel, intitulé « Hello world ».

:::image type="content" source="media/github-enterprise-server/example-in-repo.png" alt-text="Exemple dans référentiel.":::

Félicitations ! Vous venez de terminer votre premier workflow Actions sur GitHub Enterprise Server, qui s’exécute sur votre cloud privé Azure VMware Solution.

Nous n’avons fait qu’effleurer la surface de ce que vous pouvez faire avec GitHub Actions. Pour plus d’inspiration, consultez la liste des actions sur le [marketplace de GitHub](https://github.com/marketplace) ou [créez vos propres actions](https://docs.github.com/en/actions/creating-actions).

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, nous avons configuré une nouvelle instance GitHub Enterprise Server, l’équivalent auto-hébergé de GitHub.com, sur votre cloud privé Azure VMware Solution. Cette instance comprend la prise en charge de GitHub Actions et utilise Stockage Blob Azure pour la persistance des journaux et des artefacts. Il s’agit d’une combinaison parfaite pour une expérience moderne, collaborative et sûre de développement de logiciels. Elle s’appuie sur les bases solides d’Azure VMware Solution, ce qui vous permet de tirer parti des ressources cloud dans un cadre familier.

Pour plus d’informations, consultez les ressources suivantes :

- [Prise en main de GitHub Actions](https://docs.github.com/en/actions)
- [Participer au programme bêta](https://resources.github.com/beta-signup/)
- [En savoir plus sur l’administration de GitHub Enterprise Server](https://githubtraining.github.io/admin-training/#/00_getting_started)