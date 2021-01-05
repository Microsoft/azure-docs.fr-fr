---
title: Afficher les évaluations de mise à jour d’Azure Automation
description: Cet article explique comment voir les évaluations des déploiements d’Update Management.
services: automation
ms.subservice: update-management
ms.date: 09/17/2020
ms.topic: conceptual
ms.openlocfilehash: 39df5888a330a92ae043e34c3043da5b1f566345
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92221557"
---
# <a name="view-update-assessments-in-update-management"></a>Afficher les évaluations de mise à jour dans Update Management

Dans Update Management, vous pouvez voir des informations sur vos ordinateurs, les mises à jour manquantes, les déploiements de mises à jour et les déploiements de mises à jour planifiés. Vous pouvez afficher les informations d’évaluation relatives à la machine virtuelle Azure sélectionnée, au serveur Arc sélectionné ou au compte Automation pour l’ensemble des ordinateurs et serveurs configurés.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com)

## <a name="view-update-assessment"></a>Afficher l’évaluation des mises à jour

Pour afficher l’évaluation des mises à jour à partir d’une machine virtuelle Azure, accédez à **Machines virtuelles** et sélectionnez votre machine virtuelle dans la liste. Dans le menu de gauche, sélectionnez **Mises à jour de l’hôte et de l’invité**, puis sélectionnez **Accéder à Update Management** sur la page **Mises à jour de l’hôte et de l’invité**.

Dans Update Management, vous pouvez voir des informations sur votre ordinateur, les mises à jour manquantes, les déploiements de mises à jour et les déploiements de mises à jour planifiés.

[ ![Update Management assessment view for Azure VM](./media/view-update-assessments/update-assessment-azure-vm.png)](./media/view-update-assessments/update-assessment-azure-vm-expanded.png#lightbox)

Pour afficher l’évaluation des mises à jour à partir d’un serveur Arc, accédez à **Serveurs – Azure Arc** et sélectionnez votre serveur dans la liste. Dans le menu de gauche, sélectionnez **Mises à jour de l’hôte et de l’invité**. Sur la page **Mises à jour de l’hôte et de l’invité**, sélectionnez **Accéder à Update Management**.

Dans Update Management, vous pouvez voir des informations sur votre ordinateur Arc, les mises à jour manquantes, les déploiements de mises à jour et les déploiements de mises à jour planifiés.

[ ![Update Management assessment view for Arc enabled servers](./media/view-update-assessments/update-assessment-arc-server.png)](./media/view-update-assessments/update-assessment-arc-server-expanded.png#lightbox)

Pour afficher l’évaluation des mises à jour sur tous les ordinateurs, notamment les serveurs Arc à partir de votre compte Automation, accédez à **Comptes Automation** et sélectionnez dans la liste le compte Automation pour lequel Update Management est activé. Dans votre compte Automation, sélectionnez **Update Management** dans le menu de gauche.

Les mises à jour pour votre environnement sont listées dans la page **Gestion des mises à jour**. Si des mises à jour sont identifiées comme manquantes, une liste de celles-ci s’affiche sous l’onglet **Mises à jour manquantes**.

[ ![Vue par défaut de Update Management](./media/overview/update-management-view.png)](./media/overview/update-management-view-expanded.png#lightbox)

Sous la colonne **CONFORMITÉ**, vous pouvez voir quand l’ordinateur a été évalué pour la dernière fois. Sous la colonne **METTRE À JOUR LA DISPONIBILITÉ DE L’AGENT**, vous pouvez voir l’intégrité de l’agent de mise à jour. En cas de problème, sélectionnez le lien pour accéder à la documentation de résolution des problèmes, qui peut vous aider à remédier au problème.

Sous **Lien d’information**, sélectionnez le lien d’une mise à jour pour ouvrir l’article de support qui vous donne des informations importantes sur la mise à jour.

[ ![Afficher l’état des mises à jour](./media/view-update-assessments/missing-updates.png)](./media/view-update-assessments/missing-updates-expanded.png#lightbox)

Cliquez n’importe où ailleurs sur la mise à jour pour ouvrir le volet Recherche dans les journaux. La requête pour la recherche dans les journaux est prédéfinie pour cette mise à jour spécifique. Vous pouvez modifier cette requête ou créer votre propre requête pour afficher des informations détaillées.

[ ![Afficher les résultats d’une requête de journal](./media/view-update-assessments/logsearch-results.png)](./media/view-update-assessments/logsearch-results-expanded.png#lightbox)

## <a name="view-missing-updates"></a>Afficher les mises à jour manquantes

Sélectionnez **Mises à jour manquantes** pour afficher la liste des mises à jour qui manquent à vos ordinateurs. Chaque mise à jour est répertoriée et peut être sélectionnée. Les informations sur le nombre de machines qui nécessitent la mise à jour, les détails sur le système d’exploitation et un lien permettant d’obtenir plus d’informations sont tous affichés. Le volet Recherche dans les journaux présente également d’autres informations sur les mises à jour.

![Mises à jour manquantes](./media/view-update-assessments/automation-view-update-assessments-missing-updates.png)

## <a name="work-with-update-classifications"></a>Utiliser des classifications de mise à jour

Les tableaux suivants répertorient les classifications des mises à jour prises en charge dans Update Management, avec une définition de chaque classification.

### <a name="windows"></a>Windows

|classification ;  |Description  |
|---------|---------|
|Mises à jour critiques     | Mises à jour relatives à des problèmes spécifiques concernant des bogues critiques non liés à la sécurité.        |
|Mises à jour de sécurité     | Mises à jour relatives à des problèmes de sécurité propres à un produit.        |
|Correctifs cumulatifs     | Ensembles de correctifs logiciels regroupés pour faciliter leur déploiement.        |
|Packs de fonctionnalités     | Nouvelles fonctionnalités de produit distribuées en dehors d’une version de produit.        |
|Service Packs     | Ensembles de correctifs logiciels appliqués à une application.        |
|Mises à jour de définitions     | Mises à jour de fichiers de définition de virus ou autre.        |
|Outils     | Utilitaires ou fonctionnalités permettant d’effectuer une ou plusieurs tâches.        |
|Mises à jour     | Mises à jour d’applications ou de fichiers actuellement installés.        |

### <a name="linux"></a>Linux

|classification ;  |Description  |
|---------|---------|
|Mises à jour critiques et de sécurité     | Mises à jour pour un problème spécifique ou un problème de sécurité propre à un produit.         |
|Autres mises à jour     | Toutes les autres mises à jour qui ne sont ni critiques par nature, ni des mises à jour de sécurité.        |

Pour Linux, Update Management peut faire la différence entre les mises à jour critiques et les mises à jour de sécurité dans le cloud, tout en affichant les données d’évaluation. (Cette précision est possible en raison de l’enrichissement des données dans le cloud.) Pour la mise à jour corrective, Update Management s’appuie sur les données de classification disponibles sur l’ordinateur. Contrairement à d’autres distributions, CentOS n’a pas accès à ces informations dans les versions RTM du produit. Si vous disposez de machines CentOS configurées pour retourner les données de sécurité de la commande suivante, Update Management peut appliquer la mise à jour corrective en fonction des classifications :

```bash
sudo yum -q --security check-update
```

Il n’existe actuellement aucune méthode prise en charge permettant d’activer la disponibilité des données de classification natives sur CentOS. Pour le moment, seule une prise en charge optimale est proposée aux clients qui ont activé eux-mêmes cette fonctionnalité.

Pour classifier les mises à jour sur Red Hat Enterprise version 6, vous devez installer le plug-in yum-security. Sur Red Hat Enterprise Linux 7, le plug-in faisant déjà partie de yum lui-même, il est inutile d'installer quoi que ce soit. Pour plus d'informations, consultez l'[article de base de connaissances](https://access.redhat.com/solutions/10021) Red Hat suivant.

## <a name="next-steps"></a>Étapes suivantes

La phase suivante du processus consiste à [déployer des mises à jour](deploy-updates.md) sur des machines non conformes et à passer en revue les résultats du déploiement.
