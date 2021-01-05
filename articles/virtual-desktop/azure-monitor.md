---
title: Utiliser Monitor Windows Virtual Desktop Monitor préversion - Azure
description: Comment utiliser Azure Monitor pour Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 12/01/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: e0be6decf28fcbb2edacd5019f567d26403b1f31
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465489"
---
# <a name="use-azure-monitor-for-windows-virtual-desktop-to-monitor-your-deployment-preview"></a>Utiliser Azure Monitor pour Windows Virtual Desktop pour surveiller votre déploiement (préversion)

>[!IMPORTANT]
>Azure Monitor pour Windows Virtual Desktop est en préversion publique. Cette préversion est fournie sans contrat de niveau de service, c’est pourquoi nous déconseillons son utilisation pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Monitor pour Windows Virtual Desktop (préversion) est un tableau de bord basé sur des classeurs Azure Monitor qui aide les professionnels de l’informatique à comprendre leurs environnements Windows Virtual Desktop. Cette rubrique vous guide tout au long de la configuration d’Azure Monitor pour Windows Virtual Desktop pour surveiller vos environnements Windows Virtual Desktop.

## <a name="requirements"></a>Spécifications

Avant de commencer à utiliser Azure Monitor pour Windows Virtual Desktop, vous devez configurer les éléments suivants :

- Tous les environnements Windows Virtual Desktop que vous analysez doivent être basés sur la dernière version de Windows Virtual Desktop compatible avec Azure Resource Manager.

- Au moins un espace de travail Log Analytics configuré.

- Activez la collecte de données pour les éléments suivants dans votre espace de travail Log Analytics :
    - Ajouter tous les compteurs de performance requis
    - Tous les compteurs de performances ou événements utilisés dans Azure Monitor pour Windows Virtual Desktop
    - Données de l’outil de diagnostic pour tous les objets de l’environnement que vous analyserez.
    - Machines virtuelles dans l’environnement que vous allez surveiller.

Toute personne surveillant Azure Monitor pour Windows Virtual Desktop pour votre environnement aura également besoin des autorisations d’accès en lecture suivantes :

- Accès en lecture au groupe de ressources où se trouvent les ressources de l’environnement.

- Accès en lecture au(x) groupe(s) de ressources où se trouvent les hôtes de session de l’environnement

>[!NOTE]
> L’accès en lecture permet uniquement aux administrateurs d’afficher les données. Ils ont besoin d’autorisations différentes pour gérer les ressources dans le portail Windows Virtual Desktop.

## <a name="open-azure-monitor-for-windows-virtual-desktop"></a>Ouvrir Azure Monitor pour Windows Virtual Desktop

Vous pouvez ouvrir Azure Monitor pour Windows Virtual Desktop avec l’une des méthodes suivantes :

- Accédez à [aka.ms/azmonwvdi](https://portal.azure.com/#blade/Microsoft_Azure_WVD/WvdManagerMenuBlade/workbooks).

- Recherchez et sélectionnez **Windows Virtual Desktop** dans le Portail Azure, puis sélectionnez **Insights**.

- Recherchez et sélectionnez **Azure Monitor** dans le Portail Azure. Sélectionnez **Hub Insights** sous **Insights**, puis, sous **Autres** sélectionnez **Windows Virtual Desktop** pour ouvrir le tableau de bord dans la page Azure Monitor.

Une fois que vous avez ouvert Azure Monitor pour Windows Virtual Desktop, activez une ou plusieurs des cases à cocher intitulées **Abonnement**, **Groupe de ressources**, **Pool d’hôtes** et **Intervalle de temps** en fonction de l’environnement que vous souhaitez analyser.

>[!NOTE]
>Actuellement, Windows Virtual Desktop prend uniquement en charge l’analyse d’un abonnement, d’un groupe de ressources et d’un pool d’hôtes à la fois. Si vous ne trouvez pas l’environnement que vous souhaitez analyser, consultez [notre documentation de dépannage](troubleshoot-azure-monitor.md) pour connaître les demandes de fonctionnalités et problèmes connus.

## <a name="set-up-with-the-configuration-workbook"></a>Configurer avec le classeur de configuration

Si c’est la première fois que vous ouvrez Azure Monitor pour Windows Virtual Desktop, vous devez configurer Azure Monitor pour vos ressources Windows Virtual Desktop. Pour configurer vos ressources :

1. Ouvrez votre classeur dans le Portail Azure.
2. Sélectionnez **Ouvrir le classeur de configuration**.

Le classeur de configuration configure votre environnement d’analyse et vous permet de vérifier la configuration une fois que vous avez terminé le processus d’installation. Il est important de vérifier votre configuration si les éléments du tableau de bord ne s’affichent pas correctement, ou lorsque le groupe de produits publie des mises à jour qui nécessitent des points de données supplémentaires.

## <a name="host-pool-diagnostic-settings"></a>Paramètres de diagnostic du pool d’hôtes

Vous devez activer les paramètres de diagnostic Azure Monitor sur tous les objets au sein de l’environnement Windows Virtual Desktop qui prennent en charge cette fonctionnalité.

1. Ouvrez Azure Monitor pour Windows Virtual Desktop à l’adresse [aka.ms/azmonwvdi](https://portal.azure.com/#blade/Microsoft_Azure_WVD/WvdManagerMenuBlade/workbooks), puis sélectionnez **Classeur de configuration**.

2. Sélectionnez un environnement à surveiller sous **Abonnement**, **Groupe de ressources** et **Pool d’hôtes**.

3. Sous **Paramètres de diagnostic du pool d’hôtes**, vérifiez si les diagnostics Windows Virtual Desktop sont activés pour le pool d’hôtes. Si ce n’est pas le cas, un message d’erreur s’affiche indiquant « Aucune configuration de diagnostics existante n’a été trouvée pour le pool d’hôtes sélectionné ». 
   
   Les tableaux suivants doivent être activés :

    - Point de contrôle
    - Error
    - Gestion
    - Connexion
    - HostRegistration

    >[!NOTE]
    > Si vous ne voyez pas le message d’erreur, vous n’avez pas besoin de l’étape 4.

4. Sélectionnez **Configurer le pool d’hôtes**.

5. Sélectionnez **Déployer**.

6. Actualiser le classeur.

Pour en savoir plus sur l’activation des diagnostics sur tous les objets dans l’environnement Windows Virtual Desktop ou la manière d’accéder à l’espace de travail Log Analytics, consultez [Envoyer des diagnostics Windows Virtual Desktop à Log Analytics](diagnostics-log-analytics.md).

## <a name="configure-log-analytics"></a>Configurer Log Analytics

Pour commencer à utiliser Azure Monitor pour Windows Virtual Desktop, vous avez également besoin d’au moins un espace de travail Log Analytics pour collecter les données de l’environnement que vous envisagez d’analyser et les fournir au classeur. Si vous en avez déjà un de configuré, passez directement à [Configurer les compteurs de performances](#set-up-performance-counters). Pour configurer un nouvel espace de travail Log Analytics pour l’abonnement Azure contenant votre environnement Windows Virtual Desktop, consultez [Créer un espace de travail Log Analytics dans le Portail Azure](../azure-monitor/learn/quick-create-workspace.md).

>[!NOTE]
>Des frais de stockage de données standard pour Log Analytics s’appliqueront. Pour commencer, nous vous recommandons de choisir le modèle de paiement à l’utilisation et de l’ajuster à mesure que vous mettez à l’échelle votre déploiement et que vous prenez plus de données. Pour en savoir plus, consultez la section [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

### <a name="set-up-performance-counters"></a>Configurer les compteurs de performances

Vous devez activer des compteurs de performances spécifiques pour la collecte à l’intervalle d’échantillonnage correspondant dans l’espace de travail Log Analytics. Ces compteurs de performances sont les seuls compteurs dont vous aurez besoin pour surveiller Windows Virtual Desktop. Vous pouvez désactiver tous les autres pour réduire les coûts.

Si vous avez déjà activé des compteurs de performances et que vous souhaitez les supprimer, suivez les instructions de [Configuration des compteurs de performances](../azure-monitor/platform/data-sources-performance-counters.md) pour reconfigurer vos compteurs de performances. Bien que l’article explique comment ajouter des compteurs, vous pouvez également les supprimer au même emplacement.

Si vous n’avez pas encore configuré les compteurs de performances, voici comment les configurer pour Azure Monitor pour Windows Virtual Desktop :

1. Accédez à [aka.ms/azmonwvdi](https://portal.azure.com/#blade/Microsoft_Azure_WVD/WvdManagerMenuBlade/workbooks), puis sélectionnez **Classeur de configuration** en bas de la fenêtre .

2. Sous **Configuration Log Analytics**, sélectionnez l’espace de travail que vous avez configuré pour votre abonnement.

3. Dans **Compteurs de performance de l’espace de travail**, vous verrez la liste des compteurs nécessaires à l’analyse. Sur le côté droit de cette liste, vérifiez les éléments dans la liste **Compteurs manquants** pour activer les compteurs dont vous avez besoin pour commencer à analyser votre espace de travail.

4. Sélectionnez **Configurer les compteurs de performances**.

5. Sélectionnez **Appliquer la configuration**.

6. Actualisez le classeur de configuration et poursuivez la configuration de votre environnement.

Vous pouvez également ajouter de nouveaux compteurs de performances après la configuration initiale chaque fois que le service est mis à jour et nécessite de nouveaux outils d’analyse. Vous pouvez vérifier que tous les compteurs requis sont activés en les sélectionnant dans la liste **Compteurs manquants**.

>[!NOTE]
>Les compteurs de performances de délai d’entrée sont uniquement compatibles avec Windows 10 RS5 et versions ultérieures ou Windows Server 2019 et versions ultérieures.

Pour en savoir plus sur la façon d’ajouter manuellement des compteurs de performances qui ne sont pas déjà activés pour la collecte, consultez [Configuration des compteurs de performances](../azure-monitor/platform/data-sources-performance-counters.md).

### <a name="set-up-windows-events"></a>Configurer des événements Windows

Ensuite, vous devez activer des événements Windows spécifiques pour la collecte dans l’espace de travail Log Analytics. Les événements décrits dans cette section sont les seuls dont Azure Monitor a besoin pour Windows Virtual Desktop. Vous pouvez désactiver tous les autres pour réduire les coûts.

Pour configurer des événements Windows :

1. Si vous avez déjà activé des événements Windows et que vous souhaitez les supprimer, supprimez ceux que vous ne souhaitez pas avant d’utiliser le classeur de configuration pour activer l’ensemble requis à des fins d’analyse.

2. Accédez à Azure Monitor pour Windows Virtual Desktop à l’adresse [aka.ms/azmonwvdi](https://portal.azure.com/#blade/Microsoft_Azure_WVD/WvdManagerMenuBlade/workbooks), puis sélectionnez **Classeur de configuration** en bas de la fenêtre.

3. Dans **Configuration des événements Windows** se trouve une liste des événements Windows requis à des fins d’analyse. Sur le côté droit de cette liste se trouve la liste **Événements manquants**, où vous trouverez les noms d’événements et les types d’événements requis qui ne sont pas actuellement activés pour votre espace de travail. Enregistrez chacun de ces noms en vue d’une utilisation ultérieure.

4. Sélectionnez **Ouvrir la configuration des espaces de travail**.

5. Sélectionnez **Données**.

6. Sélectionnez **Journaux d’événements Windows**.

7. Ajoutez les noms d’événements manquants de l’étape 3 et le type d’événement requis pour chacun d’entre eux.

8. Actualisez le classeur de configuration et poursuivez la configuration de votre environnement.

Vous pouvez ajouter de nouveaux événements Windows après la configuration initiale, si les mises à jour de l’outil d’analyse requièrent l’activation de nouveaux événements. Pour vous assurer que tous les événements requis sont activés, revenez à la liste **Événements manquants** et activez tous les événements manquants que vous y voyez.

## <a name="install-the-log-analytics-agent-on-all-hosts"></a>Installer l’agent Log Analytics sur tous les hôtes

Enfin, vous devez installer l’agent Log Analytics sur tous les hôtes du pool d’hôtes pour envoyer des données à partir des hôtes vers l’espace de travail sélectionné.

Pour installer l'agent Log Analytics :

1. Accédez à Azure Monitor pour Windows Virtual Desktop à l’adresse [aka.ms/azmonwvdi](https://portal.azure.com/#blade/Microsoft_Azure_WVD/WvdManagerMenuBlade/workbooks), puis sélectionnez **Classeur de configuration** en bas de la fenêtre.

2. Si Log Analytics n’est pas configuré pour tous les hôtes du pool d’hôtes, une erreur s’affiche au bas de la section de configuration Log Analytics avec le message « Certains hôtes dans le pool d’hôtes n’envoient pas de données à l’espace de travail Log Analytics sélectionné. » Sélectionnez **Ajouter des hôtes à l’espace de travail** pour ajouter les hôtes sélectionnés. Si vous ne voyez pas le message d’erreur, arrêtez-vous ici.

3. Actualisez le classeur de configuration.

>[!NOTE]
>L’ordinateur hôte doit être en cours d’exécution pour installer l’extension de Log Analytics. Si le déploiement automatique échoue sur un hôte, vous pouvez toujours installer manuellement l’extension sur un hôte. Pour savoir comment installer l’extension manuellement, consultez [Extension de machine virtuelle Log Analytics pour Windows](../virtual-machines/extensions/oms-windows.md).

## <a name="optional-configure-alerts"></a>Facultatif : configurer des alertes

Vous pouvez configurer Azure Monitor pour Windows Virtual Desktop pour vous avertir si des alertes Azure Monitor graves se produisent dans votre abonnement sélectionné. Pour ce faire, suivez les instructions figurant dans [Répondre à des événements avec des alertes Azure Monitor](../azure-monitor/learn/tutorial-response.md).

## <a name="diagnostic-and-usage-data"></a>Données relatives aux diagnostics et à l’utilisation

Microsoft collecte automatiquement les données sur l’utilisation et les performances via votre utilisation du service Azure Monitor. Microsoft utilise ces données pour fournir et améliorer la qualité, la sécurité et l’intégrité du service.

Pour fournir des fonctionnalités de résolution des problèmes précises et efficaces, les données collectées incluent l’ID de session du portail, l’ID d’utilisateur Azure Active Directory et le nom de l’onglet du portail dans lequel l’événement s’est produit. Microsoft ne collecte pas de nom, d’adresse ou d’autres coordonnées.

Pour plus d’informations sur l’utilisation et la collecte de données, voir la [Déclaration de confidentialité des Services en ligne de Microsoft](https://privacy.microsoft.com/privacystatement).

>[!NOTE]
>Pour en savoir plus sur l’affichage ou la suppression des données personnelles collectées par le service, consultez [Requêtes DSR (droits de la personne concernée) Azure pour le RGPD](/microsoft-365/compliance/gdpr-dsr-azure). Pour plus d’informations sur le RGPD, consultez la [section RGPD du portail Service Trust](https://servicetrust.microsoft.com/ViewPage/GDPRGetStarted).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez configuré votre Portail Azure Windows Virtual Desktop, voici quelques ressources qui peuvent vous aider :

- Consultez notre [glossaire](azure-monitor-glossary.md) pour en savoir plus sur les termes et les concepts liés à Azure Monitor pour Windows Virtual Desktop.
- Si vous rencontrez un problème, consultez notre [guide de résolution des problèmes](troubleshoot-azure-monitor.md) pour obtenir de l’aide.