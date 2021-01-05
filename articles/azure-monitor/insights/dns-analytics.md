---
title: Solution DNS Analytics dans Azure Monitor | Microsoft Docs
description: Configurez et utilisez la solution DNS Analytics dans Azure Monitor pour rassembler des informations sur une infrastructure DNS concernant la sécurité, les performances et les opérations.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/20/2018
ms.openlocfilehash: 7bdea9239faa4ec66fffa236bea40afd5e628e62
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96607141"
---
# <a name="gather-insights-about-your-dns-infrastructure-with-the-dns-analytics-preview-solution"></a>Rassembler des informations sur votre infrastructure DNS avec la solution DNS Analytics (préversion)

![Symbole DNS Analytics](./media/dns-analytics/dns-analytics-symbol.png)

Cet article explique comment configurer et utiliser la solution Azure DNS Analytics dans Azure Monitor pour rassembler des informations sur une infrastructure DNS concernant la sécurité, les performances et les opérations.

DNS Analytics vous aide à effectuer les tâches suivantes :

- Identifier les clients qui tentent de résoudre des noms de domaines malveillants
- Identifier les enregistrements de ressources obsolètes
- Identifier les noms de domaine fréquemment interrogés et les clients DNS communiquant
- Afficher la charge de la demande sur les serveurs DNS
- Afficher les échecs d’inscription DNS dynamique

La solution collecte, analyse et met en corrélation les journaux d’audit et d’analyse DNS Windows et d’autres données connexes de vos serveurs DNS.

## <a name="connected-sources"></a>Sources connectées

Le tableau suivant décrit les sources connectées qui sont prises en charge par cette solution :

| **Source connectée** | **Support** | **Description** |
| --- | --- | --- |
| [Agents Windows](../platform/agent-windows.md) | Oui | La solution collecte des informations DNS auprès d’agents Windows. |
| [Agents Linux](../learn/quick-collect-linux-computer.md) | Non | La solution ne collecte aucune information DNS auprès d’agents Linux directs. |
| [Groupe d’administration de Microsoft System Center Operations Manager](../platform/om-agents.md) | Oui | La solution collecte des informations DNS auprès d’agents dans un groupe d’administration d’Operations Manager connecté. Une connexion directe entre l’agent Operations Manager et Azure Monitor n’est pas obligatoire. Les données sont transférées du groupe d’administration à l’espace de travail Log Analytics. |
| [Compte Azure Storage](../platform/resource-logs.md#send-to-log-analytics-workspace) | Non | Le stockage Azure n’est pas utilisé par la solution. |

### <a name="data-collection-details"></a>Détails sur la collecte de données

La solution collecte des données liées aux inventaires DNS et aux événements DNS sur les serveurs DNS sur lesquels un agent Log Analytics est installé. Ces données sont ensuite chargées vers Azure Monitor, puis affichées dans le tableau de bord de la solution. Les données liées aux inventaires, comme le nombre de serveurs, de zones et d’enregistrements de ressources DNS, sont collectées en exécutant les applets de commande PowerShell de DNS. Les données sont mises à jour tous les deux jours. Les données liées aux événements sont collectées quasiment en temps réel à partir des [journaux d’analyse et d’audit](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn800669(v=ws.11)#enhanc) fournis par le biais de la fonctionnalité améliorée de diagnostics et de journalisation DNS de Windows Server 2012 R2.

## <a name="configuration"></a>Configuration

Utilisez les informations suivantes pour configurer la solution.

- Vous devez disposer d’un agent [Windows](../platform/agent-windows.md) ou [Operations Manager](../platform/om-agents.md) sur chaque serveur DNS que vous souhaitez surveiller.
- Vous pouvez ajouter la solution DNS Analytics à votre espace de travail Log Analytics à partir de la [Place de marché Microsoft Azure](https://aka.ms/dnsanalyticsazuremarketplace). Vous pouvez également procéder de la manière décrite dans [Ajouter des solutions Azure Monitor à partir de la Galerie Solutions](solutions.md).

La solution commence la collecte des données sans que vous ne deviez procéder à une configuration supplémentaire. Toutefois, vous pouvez utiliser la configuration suivante pour personnaliser la collecte de données.

### <a name="configure-the-solution"></a>Configuration de la solution

Dans le tableau de bord de la solution, cliquez sur **Configuration** pour ouvrir la page Configuration de DNS Analytics. Voici les deux types de modification de configuration que vous pouvez effectuer :

- **Noms de domaine de la liste d’autorisation**. La solution ne traite pas toutes les requêtes de recherche. Elle gère une liste d’autorisation des suffixes de nom de domaine. Les requêtes de recherche qui résolvent les noms de domaine qui correspondent à des suffixes de noms de domaine de cette liste d’autorisation ne sont pas traitées par la solution. Ne pas traiter les noms de domaine figurant dans la liste d’autorisation permet d’optimiser les données envoyées à Azure Monitor. La liste d’autorisation par défaut inclut des noms de domaine public populaires, tels que www.google.com et www.facebook.com. Vous pouvez afficher la liste par défaut complète à l’aide de la barre de défilement.

  Vous pouvez modifier la liste pour ajouter un suffixe de nom de domaine pour lequel vous souhaitez afficher les informations de recherche. Vous pouvez supprimer tout suffixe de nom de domaine pour lequel vous ne souhaitez pas afficher les informations de recherche.

- **Seuil de clients communiquant**. Les clients DNS qui dépassent le seuil correspondant au nombre de demandes de recherche sont mis en surbrillance dans le panneau **Clients DNS**. Par défaut, le seuil est défini sur 1 000 et vous pouvez le modifier.

    ![Noms de domaine de la liste d’autorisation](./media/dns-analytics/dns-config.png)

## <a name="management-packs"></a>Packs d’administration

Si vous utilisez Microsoft Monitoring Agent pour vous connecter à votre espace de travail Log Analytics, le pack d’administration suivant est installé :

- Microsoft DNS Data Collector Intelligence Pack (Microsoft.IntelligencePacks.Dns)

Si votre groupe d’administration Operations Manager est connecté à votre espace de travail Log Analytics, les packs d’administration suivants sont installés dans Operations Manager lorsque vous ajoutez cette solution. Ces packs d’administration ne nécessitent aucune opération de configuration ou de maintenance :

- Microsoft DNS Data Collector Intelligence Pack (Microsoft.IntelligencePacks.Dns)
- Microsoft System Center Advisor DNS Analytics Configuration (Microsoft.IntelligencePack.Dns.Configuration)

Pour plus d’informations sur la façon dont ces packs d’administration de solution sont mis à jour, consultez [Connecter Operations Manager à Log Analytics](../platform/om-agents.md).

## <a name="use-the-dns-analytics-solution"></a>Utilisation de la solution DNS Analytics

[!INCLUDE [azure-monitor-solutions-overview-page](../../../includes/azure-monitor-solutions-overview-page.md)]


La vignette DNS inclut le nombre de serveurs DNS où les données sont collectées. Il inclut également le nombre de demandes effectuées par les clients au cours des dernières 24 heures visant à résoudre des domaines malveillants. Lorsque vous cliquez sur la vignette, le tableau de bord de la solution s’ouvre.

![Vignette DNS Analytics](./media/dns-analytics/dns-tile.png)

### <a name="solution-dashboard"></a>Tableau de bord de solution

Le tableau de bord de la solution contient une synthèse des informations portant sur les différentes fonctionnalités de la solution. Il inclut également des liens vers la vue détaillée de diagnostic et d’analyse d’investigation. Par défaut, les données sont affichées pour les sept derniers jours. Vous pouvez modifier la plage de dates et d’heures avec le **contrôle de sélection de date/heure**, comme illustré dans l’image suivante :

![Contrôle de sélection de date/heure](./media/dns-analytics/dns-time.png)

Le tableau de bord de la solution affiche les panneaux suivants :

**Sécurité DNS**. Signale les clients DNS qui tentent de communiquer avec des domaines malveillants. À l’aide des flux d’évaluation des menaces de Microsoft, DNS Analytics peut détecter les adresses IP des clients qui essaient d’accéder aux domaines malveillants. Dans de nombreux cas, les appareils infectés par des programmes malveillants « composent un numéro » qui renvoie au centre de « commande et de contrôle » du domaine malveillant en résolvant le nom de domaine des programmes malveillants.

![Panneau Sécurité DNS](./media/dns-analytics/dns-security-blade.png)

Lorsque vous cliquez sur une adresse IP de client dans la liste, la fonctionnalité Recherche dans les journaux s’ouvre et affiche les détails de la recherche de la requête respective. Dans l’exemple suivant, DNS Analytics a détecté que la communication a été effectuée avec un [IRCbot](https://www.microsoft.com/en-us/wdsi/threats/malware-encyclopedia-description?Name=Backdoor:Win32/IRCbot&threatId=2621) :

![Résultats de recherche dans les journaux affichant un ircboot](./media/dns-analytics/ircbot.png)

Ces informations vous aident à identifier les éléments suivants :

- IP du client qui a initié la communication
- Nom de domaine résolvant l’adresse IP malveillante
- Adresses IP de résolution du nom de domaine
- Adresse IP malveillante
- Gravité du problème
- Motif du blocage de l’adresse IP malveillante
- Heure de détection

**Domaines interrogés**. Indique les noms de domaine les plus fréquemment interrogés par les clients DNS dans votre environnement. Vous pouvez afficher la liste de tous les noms de domaine interrogés. Vous pouvez également explorer les détails de demande de recherche d’un nom de domaine spécifique dans Recherche dans les journaux.

![Panneau Domaines interrogés](./media/dns-analytics/domains-queried-blade.png)

**Clients DNS**. Signale les clients qui *excèdent le seuil* correspondant au nombre de requêtes dans la période choisie. Vous pouvez afficher dans Recherche dans les journaux la liste de tous les clients DNS et les détails des requêtes qu’ils ont effectuées.

![Panneau Clients DNS](./media/dns-analytics/dns-clients-blade.png)

**Inscriptions DNS dynamiques**. Signale les échecs d’inscription de nom. Tous les échecs d’inscription pour les [enregistrements de ressources](https://en.wikipedia.org/wiki/List_of_DNS_record_types) d’adresse (type A et AAAA) sont mis en surbrillance avec les adresses IP des clients qui ont effectué les demandes d’inscription. Vous pouvez ensuite utiliser ces informations pour rechercher la cause principale de l’échec de l’inscription en procédant comme suit :

1. Recherchez dans la zone qui fait autorité le nom que le client essaie de mettre à jour.

1. Utilisez la solution pour vérifier les informations d’inventaire de cette zone.

1. Vérifiez que la mise à jour dynamique de la zone est activée.

1. Vérifiez si la zone est configurée pour la mise à jour dynamique sécurisée ou non.

    ![Panneau Inscriptions DNS dynamiques](./media/dns-analytics/dynamic-dns-reg-blade.png)

**Demandes d’inscriptions de noms**. La vignette supérieure indique une tendance du nombre de demandes de mise à jour dynamique DNS réussies et en échec. La vignette inférieure répertorie les 10 principaux clients ayant envoyé des demandes de mise à jour DNS en échec aux serveurs DNS, triés en fonction du nombre d’échecs.

![Panneau Demandes d’inscriptions de noms](./media/dns-analytics/name-reg-req-blade.png)

**Exemples de requêtes DDI Analytics**. Contient la liste des requêtes de recherche les plus courantes qui extraient directement les données d’analyse brutes.


![Exemples de requêtes](./media/dns-analytics/queries.png)

Vous pouvez utiliser ces requêtes comme point de départ pour créer vos propres requêtes de génération de rapports personnalisés. Les requêtes renvoient à la page Recherche dans les journaux de DNS Analytics où les résultats sont affichés :

- **Liste des serveurs DNS**. Affiche une liste de tous les serveurs DNS avec le nom de domaine complet, le nom de domaine, le nom de la forêt et les adresses IP de serveur associés.
- **Liste des zones DNS**. Affiche la liste de toutes les zones DNS avec le nom de zone, l’état de mise à jour dynamique, les serveurs de noms et l’état de signature DNSSEC associés.
- **Enregistrements de ressources non utilisés**. Affiche une liste de tous les enregistrements de ressources inutilisés/périmés. Cette liste contient le nom et le type d’enregistrement de ressource, le serveur DNS associé, l’heure de génération d’enregistrement et le nom de la zone. Cette liste permet d’identifier les enregistrements de ressources DNS qui ne sont plus utilisés. En fonction de ces informations, vous pouvez ensuite supprimer ces entrées des serveurs DNS.
- **Charge de requête des serveurs DNS**. Affiche des informations afin que vous puissiez obtenir une idée de la charge DNS sur vos serveurs DNS. Ces informations peuvent vous aider à planifier la capacité des serveurs. Vous pouvez accéder à l’onglet **Mesures** pour modifier l’affichage et le remplacer par une visualisation graphique. Cet affichage vous permet de comprendre de quelle façon la charge DNS est répartie entre vos serveurs DNS. Il indique les tendances des taux de requête DNS pour chaque serveur.

    ![Résultats de recherche dans les journaux des requêtes des serveurs DNS](./media/dns-analytics/dns-servers-query-load.png)

- **Charge de requête des zones DNS**. Affiche les statistiques (nombre de requêtes de zone DNS par seconde) de toutes les zones sur les serveurs DNS gérés par la solution. Cliquez sur l’onglet **Mesures** pour modifier l’affichage et passer des enregistrements détaillés à une visualisation graphique des résultats.
- **Événements de configuration**. Affiche tous les événements de changement de configuration DNS et les messages associés. Vous pouvez ensuite filtrer ces événements en fonction de l’heure de l’événement, de l’ID d’événement, du serveur DNS ou de la catégorie de la tâche. Les données peuvent vous aider à analyser les modifications apportées à des serveurs DNS spécifiques à des moments donnés.
- **Journal analytique DNS**. Affiche tous les événements analytiques sur tous les serveurs DNS gérés par la solution. Vous pouvez alors filtrer ces événements en fonction de l’heure de l’événement, de l’ID d’événement, du serveur DNS, de l’adresse IP du client qui a effectué la requête de recherche et de la catégorie du type de requête. Les événements d’analyse de serveur DNS permettent le suivi des activités sur le serveur DNS. Un événement d’analyse est enregistré chaque fois que le serveur envoie ou reçoit des informations DNS.

### <a name="search-by-using-dns-analytics-log-search"></a>Rechercher à l’aide de la recherche dans les journaux de DNS Analytics

Sur la page Recherche dans les journaux, vous pouvez créer une requête. Vous pouvez filtrer les résultats de la recherche à l’aide des contrôles de facette. Vous pouvez également créer des requêtes avancées pour la transformation, le filtrage et les rapports relatifs aux résultats. Commencez par utiliser les requêtes suivantes :

1. Dans la **zone de requête de recherche**, tapez `DnsEvents` pour afficher tous les événements DNS générés par les serveurs DNS gérés par la solution. Les résultats indiquent les données de journal pour tous les événements liés aux requêtes de recherche, aux inscriptions dynamiques et aux modifications de la configuration.

    ![Recherche dans les journaux des événements DNS](./media/dns-analytics/log-search-dnsevents.png)  

    a. Pour afficher les données du journal des requêtes de recherche, sélectionnez **LookUpQuery** comme filtre **Sous-type** à partir du contrôle de la facette sur la gauche. Un tableau contenant les événements de requête de recherche pour la période sélectionnée s’affiche.

    b. Pour afficher les données de journal correspondant aux inscriptions dynamiques, sélectionnez **DynamicRegistration** comme filtre **Sous-type** à partir du contrôle de facette sur la gauche. Un tableau contenant les événements d’inscription dynamique pour la période sélectionnée s’affiche.

    c. Pour afficher les données du journal correspondant aux modifications de configuration, sélectionnez **ConfigurationChange** comme filtre **Sous-type** à partir du contrôle de facette sur la gauche. Un tableau qui répertorie tous les événements de modification de configuration pour la période sélectionnée s’affiche.

1. Dans la **zone de requête de recherche**, tapez `DnsInventory` pour afficher toutes les données associées aux inventaires DNS pour les serveurs DNS gérés par la solution. Les résultats indiquent les données de journal pour les serveurs DNS, les zones DNS et les enregistrements de ressources.

    ![Recherche dans les journaux des inventaires DNS](./media/dns-analytics/log-search-dnsinventory.png)
    
## <a name="troubleshooting"></a>Dépannage

Étapes de résolution des problèmes courantes :

1. Données de recherches DNS manquantes : Pour résoudre ce problème, essayez de réinitialiser la configuration ou simplement de charger la page de configuration une fois dans le portail. Pour la réinitialisation, il vous suffit de modifier un paramètre en lui affectant une autre valeur, puis de rétablir la valeur d’origine et d’enregistrer la configuration.

## <a name="suggestions"></a>Suggestions

Pour nous faire part de vos commentaires, rendez-vous sur la [page UserVoice de User Analytics](https://aka.ms/dnsanalyticsuservoice) afin de publier des idées sur les fonctionnalités DNS Analytics. 

## <a name="next-steps"></a>Étapes suivantes

[Interrogez les journaux d’activité](../log-query/log-query-overview.md) pour afficher des enregistrements de journal d’activité DNS détaillés.
