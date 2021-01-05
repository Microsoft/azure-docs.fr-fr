---
title: Utiliser les outils d’automatisation de l’infrastructure
description: Découvrez comment utiliser des outils d’automatisation d’infrastructure tels que Ansible, Chef, Puppet, Terraform et Packer pour créer et gérer des machines virtuelles dans Azure.
author: cynthn
ms.service: virtual-machines
ms.topic: conceptual
ms.workload: infrastructure
ms.date: 07/17/2020
ms.author: cynthn
ms.openlocfilehash: dfa0efaa49fbca0b927bfd32511348f3ded7e6c7
ms.sourcegitcommit: ab94795f9b8443eef47abae5bc6848bb9d8d8d01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96302803"
---
# <a name="use-infrastructure-automation-tools-with-virtual-machines-in-azure"></a>Utiliser les outils d’automatisation d’infrastructure avec les machines virtuelles dans Azure

Pour créer et gérer des machines virtuelles de façon cohérente à grande échelle, une forme d’automatisation est généralement souhaitée. De nombreux outils et solutions permettent d’automatiser le déploiement et la gestion du cycle de vie de toute l’infrastructure Azure. Cet article présente certains des outils d’automatisation d’infrastructure utilisables dans Azure. Ces outils s’intègrent généralement dans l’une des approches suivantes :

- Automatiser la configuration des machines virtuelles
    - Les outils incluent [Ansible](#ansible), [Chef](#chef), [Puppet](#puppet) et [Modèle Azure Resource Manager](#azure-resource-manager-template).
    - Les outils spécifiques à la personnalisation de la machine virtuelle incluent [cloud-init](#cloud-init) pour les machines virtuelles Linux, la [Configuration d’état souhaité (DSC) PowerShell](#powershell-dsc)et l’[Extension de Script personnalisé Azure](#azure-custom-script-extension) pour toutes les machines virtuelles Azure.

- Automatiser la gestion de l’infrastructure
    - Parmi les outils, on retrouve [Packer](#packer) qui automatise les builds d’image de machine virtuelle personnalisée et [Terraform](#terraform) qui automatise les processus de génération de l’infrastructure.
    - [Azure Automation](#azure-automation) peut effectuer des actions dans votre infrastructure Azure et votre infrastructure locale.

- Automatiser le déploiement et la diffusion d’applications
    - Les exemples incluent [Azure DevOps Services](#azure-devops-services) et [Jenkins](#jenkins).

## <a name="ansible"></a>Ansible
[Ansible](https://www.ansible.com/) est un moteur d’automatisation de la gestion de la configuration, de la création de machine virtuelle ou du déploiement d’application. Ansible utilise un modèle sans agent, généralement avec des clés SSH, pour authentifier et gérer les machines cibles. Les tâches de configuration sont définies dans les playbooks, avec un certain nombre de modules Ansible permettant d’effectuer des tâches spécifiques. Pour plus d’informations, consultez [Comment fonctionne Ansible](https://www.ansible.com/how-ansible-works).

Découvrez comment :

- [Installer et configurer Ansible sur Linux pour une utilisation avec Azure](/azure/developer/ansible/install-on-linux-vm).
- [Créer une machine virtuelle Linux](/azure/developer/ansible/vm-configure).
- [Gérer une machine virtuelle Linux](/azure/developer/ansible/vm-manage).


## <a name="chef"></a>Chef
[Chef](https://www.chef.io/) est une plate-forme d’automatisation qui permet de définir la configuration, le déploiement et la gestion de votre infrastructure. Les composants supplémentaires incluaient Chef Habitat pour l’automatisation du cycle de vie de l’application plutôt que de l’infrastructure, ainsi que Chef InSpec qui permet d’automatiser la conformité avec les exigences de sécurité et de stratégie. Les clients Chef sont installés sur les machines cibles, avec un ou plusieurs serveurs Chef centraux qui stockent et gèrent les configurations. Pour plus d’informations, consultez [Une vue d’ensemble de Chef](https://docs.chef.io/chef_overview.html).

Découvrez comment :

- [Déployer Chef Automate à partir d’Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/chef-software.chef-automate?tab=Overview).
- [Installer Chef sur Windows et créer des machines virtuelles Azure](/azure/developer/chef/windows-vm-configure).


## <a name="puppet"></a>Puppet
[Puppet](https://www.puppet.com) est une plate-forme d’automatisation de l’entreprise qui gère le processus de diffusion et de déploiement d’application. Les agents sont installés sur les machines cibles pour permettre à Puppet Master d’exécuter des manifestes qui définissent la configuration souhaitée de l’infrastructure Azure et des machines virtuelles. Puppet peut s’intégrer à d’autres solutions telles que Jenkins et GitHub pour un flux de travail devops amélioré. Pour plus d’informations, consultez [Comment fonctionne Puppet](https://puppet.com/products/how-puppet-works).

Découvrez comment :

- [Déployer Puppet](https://puppet.com/docs/puppet/5.5/install_windows.html).


## <a name="cloud-init"></a>Cloud-Init
[Cloud-init](https://cloudinit.readthedocs.io) est une méthode largement utilisée pour personnaliser une machine virtuelle Linux lors de son premier démarrage. Vous pouvez utiliser cloud-init pour installer des packages et écrire des fichiers, ou encore pour configurer des utilisateurs ou des paramètres de sécurité. cloud-init étant appelé pendant le processus de démarrage initial, aucune autre étape ni aucun agent ne sont nécessaires pour appliquer votre configuration.  Pour plus d’informations sur la façon de mettre correctement en forme vos fichiers `#cloud-config`, consultez le [site de documentation cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/format.html#cloud-config-data).  Les fichiers `#cloud-config` sont des fichiers texte encodés en base64.

Cloud-init fonctionne aussi sur les différentes distributions. Par exemple, vous n’utilisez pas **apt-get install** ou **yum install** pour installer un package. Au lieu de cela, vous pouvez définir une liste des packages à installer, après quoi cloud-init se charge d’utiliser automatiquement l’outil de gestion de package natif correspondant à la distribution que vous sélectionnez.

Nous travaillons activement avec nos partenaires de distribution Linux afin de mettre des images compatibles cloud-init à disposition sur Place de marché Azure. Ces images permettent à vos déploiements et configurations cloud-init de fonctionner de manière fluide avec des machines virtuelles et des groupes de machines virtuelles identiques.
Pour en savoir plus sur cloud-init sur Azure :

- [Prise en charge de cloud-init pour les machines virtuelles Linux dans Azure](./linux/using-cloud-init.md)
- [Essayez un didacticiel sur la configuration automatisée de machine virtuelle à l’aide de cloud-init](./linux/tutorial-automate-vm-deployment.md).


## <a name="powershell-dsc"></a>DSC PowerShell
La [Configuration d’état souhaité (DSC) PowerShell](/powershell/scripting/dsc/overview/overview) est une plateforme de gestion qui définit la configuration des machines cibles. Elle peut également être utilisée sur Linux via le [serveur Open Management Infrastructure (OMI)](https://collaboration.opengroup.org/omi/).

Les configurations d’état souhaité définissent les éléments à installer sur une machine et la procédure à suivre pour configurer l’hôte. Un moteur du Gestionnaire de configuration local (LCM) s’exécute sur chaque nœud cible qui traite les actions demandées en fonction des configurations envoyées. Un serveur collecteur est un service web qui s’exécute sur un hôte central pour stocker les configurations d’état souhaité et les ressources associées. Le serveur collecteur communique avec le moteur du Gestionnaire de configuration local sur chaque hôte cible pour fournir les configurations requises et signaler la conformité.

Découvrez comment :

- [Créer une configuration d’état souhaité de base](/powershell/scripting/dsc/quickstarts/website-quickstart).
- [Configurer un serveur collecteur de configuration d’état souhaité](/powershell/scripting/dsc/pull-server/pullserver).
- [Utiliser une configuration d’état souhaité pour Linux](/powershell/scripting/dsc/getting-started/lnxgettingstarted).


## <a name="azure-custom-script-extension"></a>Extension de Script personnalisé Azure
L’Extension de Script personnalisé Azure pour [Linux](./extensions/custom-script-linux.md) ou [Windows](./extensions/custom-script-windows.md) télécharge et exécute des scripts sur les machines virtuelles Azure. Vous pouvez utiliser l’extension lors de la création d’une machine virtuelle, ou à tout moment après l’exécution de la machine virtuelle.

Les scripts peuvent être téléchargés à partir du stockage Azure ou de n’importe quel emplacement public tel qu’un référentiel GitHub. Avec l’Extension de Script personnalisé, vous pouvez écrire des scripts dans n’importe quel langage qui s’exécute sur la machine virtuelle source. Ces scripts peuvent être utilisés pour installer des applications ou configurer la machine virtuelle comme vous le souhaitez. Pour sécuriser les informations d’identification, des informations sensibles telles que les mots de passe peuvent être stockées dans une configuration protégée. Ces informations d’identification ne sont déchiffrées qu’à l’intérieur de la machine virtuelle.

Découvrez comment :

- [Créer une machine virtuelle Linux avec Azure CLI et utiliser l’Extension de Script personnalisé](./scripts/virtual-machines-linux-cli-sample-create-vm-nginx.md?toc=%2fcli%2fazure%2ftoc.json).
- [Créer une machine virtuelle Windows avec Azure PowerShell et utiliser l’Extension de Script personnalisé](./scripts/virtual-machines-windows-powershell-sample-create-vm-iis.md?toc=%2fpowershell%2fmodule%2ftoc.json).


## <a name="packer"></a>Packer
[Packer](https://www.packer.io) automatise le processus de génération lorsque vous créez une image personnalisée de la machine virtuelle dans Azure. Grâce à Packer, vous pouvez définir le système d’exploitation et exécuter des scripts de post-configuration qui permettent de personnaliser la machine virtuelle pour vos besoins spécifiques. Une fois configurée, la machine virtuelle est ensuite capturée en tant qu’image d’un disque géré. Packer automatise le processus de création de la machine virtuelle source, les ressources réseau et de stockage, exécute des scripts de configuration, puis crée l’image de la machine virtuelle.

Découvrez comment :

- [Utiliser Packer pour créer une image de machine virtuelle Linux dans Azure](./linux/build-image-with-packer.md).
- [Utiliser Packer pour créer une image de machine virtuelle Windows dans Azure](./windows/build-image-with-packer.md).


## <a name="terraform"></a>Terraform
[Terraform](https://www.terraform.io) est un outil d’automatisation qui vous permet de définir et de créer une infrastructure Azure entière avec un format langage de modèle unique - le langage de configuration HashiCorp (HCL). Avec Terraform, vous définissez des modèles qui automatisent le processus de création du réseau, du stockage et des ressources de machine virtuelle pour une solution d’application donnée. Vous pouvez utiliser vos modèles Terraform existants pour d’autres plateformes avec Azure pour assurer la cohérence et simplifier le déploiement de l’infrastructure sans avoir à la convertir en modèle Azure Resource Manager.

Découvrez comment :

- [Installer et configurer Terraform avec Azure](/azure/developer/terraform/getting-started-cloud-shell).
- [Créer une infrastructure Azure avec Terraform](/azure/developer/terraform/create-linux-virtual-machine-with-infrastructure).


## <a name="azure-automation"></a>Azure Automation
[Azure Automation](https://azure.microsoft.com/services/automation/) utilise des runbooks pour traiter un ensemble de tâches sur les machines virtuelles que vous ciblez. Azure Automation est utilisée pour gérer des machines virtuelles existantes plutôt que pour créer une infrastructure. Azure Automation peut s’exécuter sur des machines virtuelles Linux et Windows, ainsi que localement sur des machines physiques ou virtuelles avec un Runbook Worker hybride. Vous pouvez stocker des runbooks dans un référentiel de contrôle de code source, tel que GitHub. Ces runbooks peuvent ensuite être exécutés manuellement ou selon une planification définie.

Azure Automation fournit également un service de Configuration d’état souhaité (DSC) qui vous permet de créer des définitions sur les configurations d’un ensemble donné de machines virtuelles. La configuration d’état souhaité vérifie ensuite que la configuration requise est en application et que la machine virtuelle reste cohérente. Azure Automation DSC s’exécute sur les machines Windows et Linux.

Découvrez comment :

- [Créer un runbook PowerShell](../automation/learn/automation-tutorial-runbook-textual-powershell.md).
- [Utiliser un Runbook Worker hybride pour gérer les ressources locales](../automation/automation-hybrid-runbook-worker.md).
- [Utiliser Azure Automation DSC](../automation/automation-dsc-getting-started.md).


## <a name="azure-devops-services"></a>Azure DevOps Services
[Azure DevOps Services](https://www.visualstudio.com/team-services/) est une suite d’outils qui permet de partager et de suivre le code, d’utiliser des builds automatisées et de créer un pipeline d’intégration continue/de développement continu. Azure DevOps s’intègre avec Visual Studio et d’autres éditeurs pour simplifier son utilisation. Azure DevOps peut également créer et configurer des machines virtuelles Azure, puis déployer le code vers celles-ci.

Pour en savoir plus :

- [Azure DevOps Services](/azure/devops/user-guide/index?view=vsts).


## <a name="jenkins"></a>Jenkins
[Jenkins](https://jenkins.io) est un serveur d’intégration continue qui aide à déployer et à tester des applications. Il permet également de créer des pipelines automatisés pour la diffusion de code. Des centaines de plug-ins permettent d’étendre la plate-forme principale de Jenkins. Vous pouvez également l’intégrer à de nombreux autres produits et solutions via les webhooks. Vous pouvez installer Jenkins manuellement sur une machine virtuelle Azure, exécuter Jenkins à partir d’un conteneur Docker ou utiliser une image préconstruite d’Azure Marketplace.

Découvrez comment :

- [Créer une infrastructure de développement sur une machine virtuelle Linux dans Azure avec Jenkins, GitHub et Docker](/azure/developer/jenkins/pipeline-with-github-and-docker).


## <a name="azure-resource-manager-template"></a>Modèle Azure Resource Manager
[Azure Resource Manager](../azure-resource-manager/templates/overview.md) est le service de déploiement et de gestion d’Azure. Il fournit une couche de gestion qui vous permet de créer, mettre à jour et supprimer des ressources dans votre abonnement Azure. Vous utilisez des fonctionnalités de gestion, telles que le contrôle d’accès, les verrous et les étiquettes, pour sécuriser et organiser vos ressources après le déploiement.

Découvrez comment :

- [Déployer des machines virtuelles Spot en utilisant un modèle Resource Manager](./linux/spot-template.md).
- [Créer une machine virtuelle Windows à partir d’un modèle Resource Manager](./windows/ps-template.md).
- [Télécharger le modèle d’une machine virtuelle](./windows/download-template.md).
- [Créer un modèle Azure Image Builder](./linux/image-builder-json.md).

## <a name="next-steps"></a>Étapes suivantes
De nombreuses options permettent d’utiliser les outils d’automatisation d’infrastructure dans Azure. Vous avez la possibilité d’utiliser la solution qui répond le mieux à vos besoins et à votre environnement. Pour commencer à essayer certains des outils intégrés à Azure, renseignez vous sur l’automatisation de la personnalisation d’une machine virtuelle [Linux](./linux/tutorial-automate-vm-deployment.md) ou [Windows](./windows/tutorial-automate-vm-deployment.md).