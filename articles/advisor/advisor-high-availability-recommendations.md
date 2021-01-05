---
title: Améliorer la fiabilité d’une application avec Advisor
description: Utilisez Azure Advisor pour garantir et améliorer la fiabilité de vos déploiements Azure vitaux pour l’entreprise.
ms.topic: article
ms.date: 09/27/2020
ms.openlocfilehash: 0ced690ae735a281fdf8b1c3a020ff8c63ce469b
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92078032"
---
# <a name="improve-the-reliability-of-your-application-by-using-azure-advisor"></a>Améliorer la fiabilité d’une application à l’aide d’Azure Advisor

Azure Advisor vous aide à garantir et à améliorer la continuité de vos applications stratégiques. Vous pouvez recevoir des recommandations en matière de fiabilité de la part d’Advisor dans l’onglet **Fiabilité** du tableau de bord Advisor.

## <a name="check-the-version-of-your-check-point-network-virtual-appliance-image"></a>Vérifier la version de l’image d’appliance virtuelle réseau Check Point

Advisor est capable d’identifier si votre machine virtuelle exécute une version de l’image Check Point qui est connue pour perdre la connectivité réseau pendant les opération de maintenance de la plateforme. Pour résoudre ce problème, suivez la recommandation d’Advisor afin de mettre à niveau l’image vers une version plus récente. Cette vérification permet de garantir la continuité des activités grâce à une meilleure connectivité réseau.

## <a name="ensure-application-gateway-fault-tolerance"></a>Assurer la tolérance de panne d’une passerelle

Cette recommandation garantit la continuité métier des applications stratégiques qui sont alimentées par des passerelles d’application. Advisor identifie les instances de passerelle d’application qui ne sont pas configurées pour la tolérance de panne. Il suggère ensuite des actions correctives que vous pouvez entreprendre. Advisor identifie les passerelles d’application de moyenne ou grande taille à instance unique et recommande d’ajouter au moins une instance supplémentaire. Il identifie également les petites passerelles d’application à une ou plusieurs instances et recommande de les migrer vers des références SKU de moyenne ou grande taille. Advisor recommande ces actions pour garantir que vos instances de passerelle d’application sont configurées de sorte à satisfaire les exigences actuelles du contrat de niveau de service pour ces ressources.

## <a name="protect-your-virtual-machine-data-from-accidental-deletion"></a>Protégez les données de vos machines virtuelles d’une suppression accidentelle

La configuration de la sauvegarde de machine virtuelle permet de garantir la disponibilité de vos données métier stratégiques et offre une protection contre une suppression accidentelle ou un endommagement des données. Advisor identifie les machines virtuelles sur lesquelles la sauvegarde n’est pas activée et recommande l’activation de la sauvegarde. 

## <a name="ensure-you-have-access-to-azure-experts-when-you-need-it"></a>S’assurer d’avoir accès aux experts Azure en cas de besoin

Quand vous exécutez une charge de travail vitale pour l’entreprise, il est essentiel d’avoir accès au support technique en cas de besoin. Advisor identifie les éventuels abonnements stratégiques dont les plans de support ne prévoient aucun support technique. Il recommande d’effectuer une mise à niveau pour choisir une option qui inclut un support technique.

## <a name="create-azure-service-health-alerts-to-be-notified-when-azure-problems-affect-you"></a>Créer des alertes Azure Service Health pour être averti des problèmes Azure qui vous concernent

Nous recommandons de configurer des alertes Azure Service Health pour être averti lorsque des problèmes de service Azure vous concernent. [Azure Service Health](https://azure.microsoft.com/features/service-health/) est un service gratuit qui offre des conseils et un support personnalisés lorsque vous êtes concerné par un problème de service Azure. Advisor identifie les abonnements qui n’ont pas d’alertes configurées et recommande de les configurer.

## <a name="configure-traffic-manager-endpoints-for-resiliency"></a>Configurer des points de terminaison Traffic Manager à des fins de résilience

Les profils Azure Traffic Manager avec plusieurs points de terminaison offrent une meilleure disponibilité en cas d’échec d’un point de terminaison donné. Le placement de points de terminaison dans différentes régions améliore la fiabilité du service. Advisor identifie les profils Traffic Manager où il n’existe qu’un seul point de terminaison et recommande d’ajouter au moins un point de terminaison supplémentaire dans une autre région.

Si tous les points de terminaison d’un profil Traffic Manager configuré pour le routage de proximité se trouvent dans la même région, les utilisateurs des autres régions peuvent subir des retards de connexion. L’ajout ou le déplacement d’un point de terminaison vers une autre région améliorera les performances globales et offrira une meilleure disponibilité si tous les points de terminaison d’une région échouent. Advisor identifie les profils Traffic Manager configurés pour le routage de proximité où les points de terminaison se trouvent dans la même région. Il vous recommande d’ajouter ou de déplacer un point de terminaison dans une autre région Azure.

Si un profil Traffic Manager est configuré pour le routage géographique, le trafic est acheminé vers les points de terminaison selon les régions définies. En cas d’échec d’une région, il n’existe aucun basculement prédéfini. Si vous avez un point de terminaison où le regroupement régional est configuré pour **Tous (monde)** , vous pouvez éviter les pertes de trafic et améliorer la disponibilité du service. Advisor identifie les profils Traffic Manager configurés pour le routage géographique où il n’existe aucun point de terminaison configuré avec le regroupement régional **Tous (monde)** . Il recommande cette modification de configuration.

## <a name="use-soft-delete-on-your-azure-storage-account-to-save-and-recover-data-after-accidental-overwrite-or-deletion"></a>Utiliser la suppression réversible sur le compte de stockage Azure pour enregistrer et récupérer des données après une suppression ou un remplacement accidentel

Activez la [suppression réversible](../storage/blobs/soft-delete-blob-overview.md) sur votre compte de stockage afin que les blobs supprimés passent dans un état de suppression réversible au lieu d’être supprimés de façon permanente. Quand les données sont remplacées, un instantané du blob supprimé de manière réversible est généré pour enregistrer l’état des données remplacées. La suppression réversible vous permet de récupérer les données en cas de suppression ou de remplacement accidentel. Advisor identifie les comptes de stockage Azure pour lesquels la suppression réversible n’est pas activée et vous suggère de l’activer.

## <a name="configure-your-vpn-gateway-to-active-active-for-connection-resiliency"></a>Configurer votre passerelle VPN en mode actif/actif à des fins de résilience de connexion

En configuration active/active, les deux instances d’une passerelle VPN établissent des tunnels S2S VPN vers votre périphérique VPN local. Lorsqu’un événement de maintenant planifié ou un événement non planifié se produit sur une instance de passerelle, le trafic bascule automatiquement vers l’autre tunnel IPsec actif. Azure Advisor identifie les passerelles VPN non configurées en mode actif/actif et vous suggère de les configurer pour la haute disponibilité.

## <a name="use-production-vpn-gateways-to-run-your-production-workloads"></a>Utiliser des passerelles VPN de production pour exécuter vos charges de travail de production

Azure Advisor recherche les passerelles VPN qui utilisent une référence SKU de base et vous recommande d’utiliser une référence SKU de production à la place. La référence SKU de base est conçue à des fins de développement et de test. Les SKU de production proposent :
- Plus de tunnels. 
- Une prise en charge de BGP. 
- Des options de configuration active/active. 
- Une stratégie IPsec/IKE personnalisée. 
- Une stabilité et disponibilité accrues.

## <a name="ensure-virtual-machine-fault-tolerance-temporarily-disabled"></a>Assurer la tolérance de panne des machines virtuelles (temporairement désactivée)

Pour assurer la redondance de votre application, nous vous recommandons de regrouper au moins deux machines virtuelles dans un groupe à haute disponibilité. Advisor identifie les machines virtuelles qui ne font pas partie d’un groupe à haute disponibilité et recommande de les y déplacer. Cette configuration garantit que lors d’une maintenance planifiée ou non, au moins une machine virtuelle est disponible et répond au SLA de machine virtuelle Azure. Vous pouvez choisir de créer un groupe à haute disponibilité pour la machine virtuelle ou d’ajouter la machine virtuelle à un groupe à haute disponibilité existant.

> [!NOTE]
> Si vous choisissez de créer un groupe à haute disponibilité, vous devez y ajouter au moins une machine virtuelle supplémentaire. Nous vous recommandons de regrouper au moins deux machines virtuelles dans un groupe à haute disponibilité pour garantir qu’au moins une d’elles reste disponible en cas de panne.

## <a name="ensure-availability-set-fault-tolerance-temporarily-disabled"></a>Assurer la tolérance de panne d’un groupe à haute disponibilité (temporairement désactivée)

Pour assurer la redondance de votre application, nous vous recommandons de regrouper au moins deux machines virtuelles dans un groupe à haute disponibilité. Advisor identifie les groupes à haute disponibilité contenant une seule machine virtuelle et recommande d’y ajouter au moins une machine virtuelle. Cette configuration garantit que lors d’une maintenance planifiée ou non, au moins une machine virtuelle est disponible et répond au SLA de machine virtuelle Azure. Vous pouvez créer une machine virtuelle ou ajouter une machine virtuelle existante au groupe à haute disponibilité.  

## <a name="use-managed-disks-to-improve-data-reliability-temporarily-disabled"></a>Utiliser des disques managés pour améliorer la fiabilité des données (temporairement désactivé)

Les machines virtuelles situées dans un groupe à haute disponibilité avec des disques partageant des comptes de stockage ou des unités d’échelle de stockage ne sont pas résilientes face aux échecs des unités d’échelle de stockage en cas de pannes. Advisor identifie ces groupes à haute disponibilité et conseille leur migration vers des disques managés Azure. Cette migration permet de s’assurer que les disques des machines virtuelles du groupe à haute disponibilité sont suffisamment isolés pour éviter un point de défaillance unique. 

## <a name="repair-invalid-log-alert-rules"></a>Réparer les règles d’alerte de journal invalides

Azure Advisor détecte les règles d’alerte de journal dont la section des conditions contient des requêtes non valides. Les règles d’alerte Azure Monitor exécutent des requêtes à la fréquence spécifiée et déclenchent des alertes en fonction des résultats. Les requêtes peuvent devenir non valides au fil du temps en raison de modifications effectuées dans les ressources, les tables ou les commandes référencées. Advisor recommande des corrections pour les requêtes d’alerte afin d’éviter que les règles soient automatiquement désactivées et de garantir la couverture de la surveillance. Pour plus d’informations, consultez [Résolution des problèmes liés aux règles d’alerte](../azure-monitor/platform/alerts-troubleshoot-log.md#query-used-in-a-log-alert-isnt-valid)

## <a name="configure-consistent-indexing-mode-on-your-azure-cosmos-db-collection"></a>Configurer le mode d’indexation cohérent sur une collection Azure Cosmos DB

La configuration de conteneurs Azure Cosmos DB avec le mode d’indexation différée peut perturber l’actualisation des résultats de requête. Advisor détecte les conteneurs configurés de cette façon et vous recommande de basculer en mode cohérent. [En savoir plus sur les stratégies d’indexation dans Azure Cosmos DB.](../cosmos-db/how-to-manage-indexing-policy.md)

## <a name="configure-your-azure-cosmos-db-containers-with-a-partition-key"></a>Configurer vos conteneurs Azure Cosmos DB avec une clé de partition

Azure Advisor identifie les collections Azure Cosmos DB non partitionnées qui ont presque atteint leur quota de stockage approvisionné. Il vous recommande de les migrer vers de nouvelles collections avec une définition de clé de partition de manière à ce que le service puisse automatiquement effectuer un scale-out. [En savoir plus sur le choix d’une clé de partition.](../cosmos-db/partitioning-overview.md)

## <a name="upgrade-your-azure-cosmos-db-net-sdk-to-the-latest-version-from-nuget"></a>Mettre à niveau le Kit de développement logiciel (SDK) .NET Azure Cosmos DB avec la version la plus récente de NuGet

Azure Advisor identifie les comptes Azure Cosmos DB qui utilisent d’anciennes versions du Kit de développement logiciel (SDK) .NET. Il vous recommande de le mettre à niveau avec la version la plus récente de NuGet pour bénéficier des derniers correctifs, de performances accrues et de nouvelles capacités. [En savoir plus sur le Kit de développement logiciel (SDK) .NET Azure Cosmos DB.](../cosmos-db/sql-api-sdk-dotnet-standard.md)

## <a name="upgrade-your-azure-cosmos-db-java-sdk-to-the-latest-version-from-maven"></a>Mettre à niveau votre SDK Java Azure Cosmos DB avec la dernière version de Maven

Azure Advisor identifie les comptes Azure Cosmos DB qui utilisent d’anciennes versions du Kit de développement logiciel (SDK) Java. Il vous recommande de le mettre à niveau avec la version la plus récente de Maven pour bénéficier des derniers correctifs, de performances accrues et de nouvelles capacités. [En savoir plus sur le Kit de développement logiciel (SDK) Java Azure Cosmos DB.](../cosmos-db/sql-api-sdk-java-v4.md)

## <a name="upgrade-your-azure-cosmos-db-spark-connector-to-the-latest-version-from-maven"></a>Mettre à niveau le connecteur Spark Azure Cosmos DB avec la version la plus récente de Maven

Azure Advisor identifie les comptes Azure Cosmos DB qui utilisent d’anciennes versions du connecteur Spark Azure Cosmos DB. Il vous recommande de le mettre à niveau avec la version la plus récente de Maven pour bénéficier des derniers correctifs, de performances accrues et de nouvelles capacités. [En savoir plus sur le connecteur Spark Azure Cosmos DB.](../cosmos-db/spark-connector.md)

## <a name="consider-moving-to-kafka-21-on-hdinsight-40"></a>Envisager de passer à Kafka 2.1 sur HDInsight 4.0

À partir du 1er juillet 2020, vous ne pourrez plus créer de nouveaux clusters Kafka en utilisant Kafka 1.1 sur Azure HDInsight 4.0. Les clusters existants s’exécuteront en l’état sans support de Microsoft. Envisagez de migrer vers Kafka 2.1 sur HDInsight 4.0 d’ici le 30 juin 2020 pour éviter une éventuelle interruption du système ou du support.

## <a name="consider-upgrading-older-spark-versions-in-hdinsight-spark-clusters"></a>Envisager de mettre à niveau les anciennes versions de Spark dans les clusters HDInsight Spark

À partir du 1er juillet 2020, vous ne pourrez plus créer de nouveaux clusters Spark en utilisant Spark 2.1 ou 2.2 sur HDInsight 3.6. Vous ne pourrez plus créer de nouveaux clusters Spark en utilisant Spark 2.3 sur HDInsight 4.0. Les clusters existants s’exécuteront en l’état sans support de Microsoft. 

## <a name="enable-virtual-machine-replication"></a>Activer la réplication des machines virtuelles
Les machines virtuelles pour lesquelles la réplication n’est pas activée vers une autre région ne sont pas résilientes face aux pannes régionales. La réplication des machines virtuelles réduit les impacts négatifs sur l’activité en cas de panne d’une région Azure. Advisor détecte les machines virtuelles sur lesquelles la réplication n’est pas activée et recommande de l’activer. Lorsque vous activez la réplication, en cas de panne, vous pouvez rapidement faire apparaître vos machines virtuelles dans une région Azure distante. [En savoir plus sur la réplication de machines virtuelles.](../site-recovery/azure-to-azure-quickstart.md)

## <a name="upgrade-to-the-latest-version-of-the-azure-connected-machine-agent"></a>Effectuer une mise à niveau vers la dernière version d’Azure Connected Machine Agent
[Azure Connected Machine Agent](../azure-arc/servers/manage-agent.md) est régulièrement mis à jour avec des correctifs de bogues, des améliorations en termes de stabilité et de nouvelles fonctionnalités. Nous avons identifié des ressources qui ne travaillent pas sur la dernière version de l’agent de machine, et cette recommandation d’Advisor vous suggérera de mettre à niveau votre agent vers la dernière version pour bénéficier de la meilleure expérience Azure Arc.

## <a name="do-not-override-hostname-to-ensure-website-integrity"></a>Ne pas remplacer le nom d’hôte pour garantir l’intégrité du site web
Advisor recommande d’éviter, dans la mesure du possible, de remplacer le nom d’hôte lors de la configuration d’Application Gateway. Le fait d’avoir un domaine différent sur le serveur frontal d’Application Gateway que celui qui est utilisé pour accéder au serveur principal peut entraîner des ruptures de cookies ou d’URL de redirection. Cela peut ne pas être le cas dans toutes les situations et certaines catégories de backends (comme les API REST) y sont en général moins sensibles. Assurez-vous que le serveur principal peut traiter ce problème ou mettez à jour la configuration d’Application Gateway pour que le nom d’hôte ne doive pas être remplacé par le serveur principal. Lorsqu’il est utilisé avec App Service, associez un nom de domaine personnalisé à l’application web et évitez d’utiliser le *nom d’hôte .azurewebsites.net vers le serveur principal.* [En savoir plus sur le domaine personnalisé](../application-gateway/troubleshoot-app-service-redirection-app-service-url.md).

## <a name="how-to-access-high-availability-recommendations-in-advisor"></a>Comment accéder aux recommandations en matière de haute disponibilité dans Advisor

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis ouvrez [Advisor](https://aka.ms/azureadvisordashboard).

2.  Dans le tableau de bord Advisor, sélectionnez l’onglet **Haute disponibilité**.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les recommandations d’Advisor, consultez :
* [Présentation du conseiller](advisor-overview.md)
* [Prise en main d’Advisor](advisor-get-started.md)
* [Score Advisor](azure-advisor-score.md)
* [Recommandations d’Advisor en matière de coûts](advisor-cost-recommendations.md)
* [Recommandations d’Advisor en matière de performances](advisor-performance-recommendations.md)
* [Recommandations d’Advisor en matière de sécurité](advisor-security-recommendations.md)
* [Recommandations d’Advisor en matière d’excellence opérationnelle](advisor-operational-excellence-recommendations.md)