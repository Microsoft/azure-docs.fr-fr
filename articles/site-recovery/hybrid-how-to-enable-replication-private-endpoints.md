---
title: Activer la réplication pour les machines locales avec des points de terminaison privés
description: Cet article explique comment configurer la réplication pour les machines locales avec des points de terminaison privés dans Site Recovery.
author: Harsha-CS
ms.author: harshacs
ms.service: site-recovery
ms.topic: article
ms.date: 07/14/2020
ms.openlocfilehash: f23efa8d0439422fef685480ed270dce6e78a204
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92366851"
---
# <a name="replicate-on-premises-machines-by-using-private-endpoints"></a>Répliquer des machines locales à l’aide de points de terminaison privés

Azure Site Recovery vous permet d’utiliser des points de terminaison privés [Azure Private Link](../private-link/private-endpoint-overview.md) pour la réplication de vos machines locales vers un réseau virtuel dans Azure. Un accès de point de terminaison privé à un coffre de récupération est pris en charge dans toutes les régions Azure Commerce et Secteur public.

Cet article décrit comment effectuer les opérations suivantes :

- Créer un coffre Recovery Services Sauvegarde Azure pour protéger vos machines
- Activez une identité managée pour le coffre. Accordez les autorisations nécessaires pour accéder aux comptes de stockage afin d’activer la réplication du trafic à partir d’un emplacement local vers des emplacements cibles Azure. L’accès de l’identité managée au stockage est requis pour l’accès Azure Private Link au coffre.

- Apporter les modifications DNS nécessaires pour les points de terminaison privés.
- Créer et approuver des points de terminaison privés pour un coffre à l’intérieur d’un réseau virtuel.
- Créer des points de terminaison privés pour les comptes de stockage. Vous pouvez continuer à autoriser un accès public ou via un pare-feu au stockage en fonction des besoins. La création d’un point de terminaison privé pour accéder au stockage n’est pas nécessaire pour Azure Site Recovery.
  
Le schéma suivant montre le flux de travail de réplication pour la récupération d’urgence hybride avec des points de terminaison privés. Vous ne pouvez pas créer de points de terminaison privés sur votre réseau local. Pour pouvoir utiliser des liaisons privées, vous devez créer un réseau virtuel Azure (appelé *réseau de contournement* dans cet article), établir une connectivité privée entre le réseau local et le réseau de contournement, puis créer des points de terminaison privés dans le réseau de contournement. Vous pouvez choisir n’importe quelle forme de connectivité privée.

:::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/architecture.png" alt-text="Schéma représentant l’architecture des points de terminaison Azure Site Recovery et privés.":::

## <a name="prerequisites-and-caveats"></a>Prérequis et mises en garde

- Les liens privés sont pris en charge dans Site Recovery 9.35 et versions ultérieures.
- Vous pouvez créer des points de terminaison privés uniquement pour les nouveaux coffres Recovery Services qui ne contiennent pas d’éléments. Ainsi, vous devez créer les points de terminaison privés avant que des éléments soient ajoutés au coffre. Consultez [Tarifs Azure Private Link](https://azure.microsoft.com/pricing/details/private-link/) pour plus d’informations sur les prix.
- Lorsque vous créez un point de terminaison privé à un coffre, ce dernier est verrouillé. Il est accessible uniquement à partir des réseaux possédant des points de terminaison privés.
- Azure Active Directory ne prend pas en charge les points de terminaison privés pour le moment. Vous devez donc autoriser l’accès sortant du réseau virtuel sécurisé Azure aux adresses IP et aux noms de domaine complets qui sont nécessaires au fonctionnement d’Azure Active Directory dans une région.
  Le cas échéant, vous pouvez également utiliser l’étiquette de groupe de sécurité réseau « Azure Active Directory » et des étiquettes de Pare-feu Azure pour autoriser l’accès à Azure Active Directory.
- Cinq adresses IP sont nécessaires dans le réseau de contournement où vous créez votre point de terminaison privé. Quand vous créez un point de terminaison privé pour le coffre, Site Recovery crée cinq liaisons privées pour accéder à ses microservices.
- Une adresse IP supplémentaire est nécessaire dans le réseau de contournement pour la connectivité des points de terminaison privés à un compte de stockage de cache. Vous pouvez utiliser n’importe quelle méthode de connectivité entre les sites locaux et le point de terminaison de votre compte de stockage. Par exemple, vous pouvez utiliser Internet ou Azure [ExpressRoute](../expressroute/index.yml). L’établissement d’une liaison privée est facultatif. Vous pouvez créer des points de terminaison privés pour le stockage uniquement sur les comptes v2 à usage général. Consultez la [tarification Azure Page Blobs](https://azure.microsoft.com/pricing/details/storage/page-blobs/) pour plus d’informations sur le transfert de données sur les comptes v2 à usage général.

 ## <a name="create-and-use-private-endpoints-for-site-recovery"></a>Créer et utiliser des points de terminaison privés pour Site Recovery

 Les sections suivantes décrivent les étapes à suivre pour créer et utiliser des points de terminaison privés pour Site Recovery dans vos réseaux virtuels.

> [!NOTE]
> Nous vous recommandons de suivre ces étapes dans l’ordre indiqué. Si vous ne le faites pas, vous risquez de ne pas pouvoir utiliser les points de terminaison privés dans le coffre et vous devrez peut-être redémarrer le processus avec un nouveau coffre.

### <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

Un coffre Recovery Services contient les informations de réplication des machines. Il permet de déclencher des opérations Site Recovery. Pour savoir comment créer un coffre-fort Recovery Services dans la région Azure où vous souhaitez effectuer un basculement en cas de sinistre, consultez [Créer un coffre-fort Recovery Services](./azure-to-azure-tutorial-enable-replication.md#create-a-recovery-services-vault).

### <a name="enable-the-managed-identity-for-the-vault"></a>Activer l’identité managée pour le coffre

Une [identité managée](../active-directory/managed-identities-azure-resources/overview.md) permet au coffre d’accéder à vos comptes de stockage. Site Recovery peut avoir à accéder au stockage cible et aux comptes de stockage de cache/journal en fonction de vos exigences. L’accès de l’identité managée est requis quand vous utilisez le service Azure Private Link pour le coffre.

1. Accédez à votre coffre Recovery Services. Sélectionnez **Identité** sous **Paramètres**  :

   :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/enable-managed-identity-in-vault.png" alt-text="Schéma représentant l’architecture des points de terminaison Azure Site Recovery et privés.":::

1. Cliquez sur le bouton bascule sous **État** pour le définir sur la valeur **Activé** , puis sélectionnez **Enregistrer** .

   Un ID d’objet est généré. Le coffre est à présent inscrit auprès d’Azure Active Directory

### <a name="create-private-endpoints-for-the-recovery-services-vault"></a>Créer des points de terminaison privés pour le coffre Recovery Services

Pour protéger les machines du réseau source local, vous aurez besoin d’un point de terminaison privé pour le coffre du réseau de contournement. Créez le point de terminaison privé à l’aide du centre Azure Private Link dans le Portail Azure ou par le biais d’[Azure PowerShell](../private-link/create-private-endpoint-powershell.md).

1. Depuis la barre de recherche du Portail Microsoft Azure, recherchez « liaison privée ». Sélectionnez **Liaison privée** pour accéder au Centre de liaisons privées :

   :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/search-private-links.png" alt-text="Schéma représentant l’architecture des points de terminaison Azure Site Recovery et privés.":::

1. Dans le volet gauche, sélectionnez **Points de terminaison privés** . Une fois dans le volet **Points de terminaison privés** , sélectionnez **Ajouter** pour démarrer le processus de création d’un point de terminaison privé pour votre coffre :

   :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/create-private-endpoints.png" alt-text="Schéma représentant l’architecture des points de terminaison Azure Site Recovery et privés.":::

1. Dans la page **Créer un point de terminaison privé** , indiquez les détails pour créer votre connexion de point de terminaison privé.

   1. **Concepts de base** . Fournissez les informations de base de vos points de terminaison privés. Utilisez la région que vous avez utilisée pour le réseau de contournement :

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/create-private-endpoints-basic-tab.png" alt-text="Schéma représentant l’architecture des points de terminaison Azure Site Recovery et privés.":::

   1. **Ressource** . Dans cet onglet, vous devez indiquer la ressource PaaS (plateforme en tant que service) pour laquelle vous souhaitez créer votre connexion. Sous **Type de ressource** pour l’abonnement sélectionné, sélectionnez **Microsoft.RecoveryServices/vaults** . Choisissez le nom de votre coffre Recovery Services sous **Ressource** . Sélectionnez **Azure Site Recovery** en tant que **Sous-ressource cible** .

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/create-private-endpoints-resource-tab.png" alt-text="Schéma représentant l’architecture des points de terminaison Azure Site Recovery et privés.":::

   1. **Configuration** . Dans cet onglet, indiquez le réseau de contournement et le sous-réseau où vous souhaitez que le point de terminaison privé soit créé. 

      Activez l’intégration à la zone DNS privée en sélectionnant **Oui** .
      Choisir une zone DNS existante ou en créer une. La sélection de l’option **Oui** associe automatiquement la zone au réseau de contournement. Cette action ajoute également les enregistrements DNS nécessaires à la résolution DNS des adresses IP et des noms de domaine complets créés pour le point de terminaison privé.

      Veillez à créer une zone DNS pour chaque nouveau point de terminaison privé qui se connecte au même coffre. Si vous choisissez une zone DNS privée existante, les enregistrements CNAME précédents sont remplacés. Reportez-vous au [Guide des points de terminaisons privés](../private-link/private-endpoint-overview.md#private-endpoint-properties) avant de continuer.

      Si votre environnement a un modèle hub-and-spoke, vous n’avez besoin que d’un seul point de terminaison privé et d’une seule zone DNS privée pour l’ensemble de l’installation. En effet, l’appairage est déjà activé pour tous vos réseaux virtuels. Pour plus d’informations, consultez [Intégration DNS avec des points de terminaison privés](../private-link/private-endpoint-dns.md#virtual-network-workloads-without-custom-dns-server).

      Pour créer manuellement la zone DNS privée, suivez les étapes décrites dans [Créer des zones DNS privées et ajouter des enregistrements DNS manuellement](#create-private-dns-zones-and-add-dns-records-manually).

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/create-private-endpoints-configuration-tab.png" alt-text="Schéma représentant l’architecture des points de terminaison Azure Site Recovery et privés.":::

   1. **Tags** (balises). Si vous le souhaitez, vous pouvez ajouter des étiquettes à votre point de terminaison privé.

   1. **Vérifier \+ créer** . Une fois la validation terminée, sélectionnez **Créer** pour créer le point de terminaison privé.

Une fois le point de terminaison privé créé, cinq noms de domaine complets (FQDN) sont ajoutés à celui-ci. Ces liaisons permettent aux machines du réseau local d’accéder, via le réseau de contournement, à tous les microservices Site Recovery requis dans le contexte du coffre. Vous pouvez utiliser le même point de terminaison privé pour la protection de n’importe quelle machine Azure dans le réseau de contournement et tous les réseaux appairés.

Les cinq noms de domaine sont mis en forme dans ce modèle :

`{Vault-ID}-asr-pod01-{type}-.{target-geo-code}.siterecovery.windowsazure.com`

### <a name="approve-private-endpoints-for-site-recovery"></a>Approuver des points de terminaison privés pour Site Recovery

Si vous créez le point de terminaison privé et que vous êtes également le propriétaire du coffre-fort Recovery Services, le point de terminaison privé que vous avez créé précédemment est approuvé automatiquement en quelques minutes. Dans le cas contraire, le propriétaire du coffre doit approuver le point de terminaison privé avant que vous ne puissiez l’utiliser. Pour approuver ou rejeter une connexion de point de terminaison privée demandée, accédez à **Connexions des points de terminaison privés** sous **Paramètres** dans la page du coffre de récupération.

Vous pouvez accéder à la ressource de point de terminaison privé pour passer en revue l’état de la connexion avant de continuer :

:::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/vault-private-endpoint-connections.png" alt-text="Schéma représentant l’architecture des points de terminaison Azure Site Recovery et privés.":::

### <a name="optional-create-private-endpoints-for-the-cache-storage-account"></a><a name="create-private-endpoints-for-the-cache-storage-account"></a>(Facultatif) Créer des points de terminaison privés pour le compte de stockage de cache

Vous pouvez utiliser un point de terminaison privé vers le stockage Azure. La création de points de terminaison privés pour l’accès au stockage est facultative pour la réplication Azure Site Recovery. Si vous créez un point de terminaison privé pour le stockage, vous avez besoin d’un point de terminaison privé pour le compte de stockage de cache/journal dans votre réseau virtuel de contournement.

> [!NOTE]
> Les points de terminaison privés pour le stockage peuvent être créés uniquement sur un compte de stockage v2 à usage général. Pour obtenir des informations sur les prix, consultez la [tarification Azure Page Blobs](https://azure.microsoft.com/pricing/details/storage/page-blobs/).

Suivez les [instructions pour la création d’un stockage privé](../private-link/tutorial-private-endpoint-storage-portal.md#create-storage-account-with-a-private-endpoint) afin de créer un compte de stockage avec un point de terminaison privé. Veillez à sélectionner **Oui** sous **Intégrer à une zone DNS privée** . Sélectionnez une zone DNS existante ou créez-en une.

### <a name="grant-required-permissions-to-the-vault"></a>Accorder les autorisations nécessaires au coffre

Selon votre configuration, vous pouvez avoir besoin d’un ou de plusieurs comptes de stockage dans la région Azure cible. Ensuite, accordez les autorisations d’identité managée pour tous les comptes de stockage de cache/journal requis par Site Recovery. Dans ce cas, vous devez créer les comptes de stockage requis à l’avance.

Avant d’activer la réplication des machines virtuelles, vous devez accorder à l’identité managée du coffre les autorisations de rôle suivantes en fonction du type de compte de stockage.

- Comptes de stockage basés sur Resource Manager (type Standard) :
  - [Contributeur](../role-based-access-control/built-in-roles.md#contributor)
  - [Contributeur aux données Blob du stockage](../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)
- Comptes de stockage basés sur Resource Manager (type Premium) :
  - [Contributeur](../role-based-access-control/built-in-roles.md#contributor)
  - [Propriétaire des données Blob du stockage](../role-based-access-control/built-in-roles.md#storage-blob-data-owner)
- Comptes de stockage Classic :
  - [Contributeur de compte de stockage classique](../role-based-access-control/built-in-roles.md#classic-storage-account-contributor)
  - [Rôle de service d’opérateur de clé de compte de stockage classique](../role-based-access-control/built-in-roles.md#classic-storage-account-key-operator-service-role)

Ces étapes expliquent comment ajouter une attribution de rôle à vos comptes de stockage :

1. Accédez au compte de stockage. Sélectionnez **Contrôle d’accès (IAM)** dans le volet gauche.

1. Dans la section **Ajouter une attribution de rôle** , sélectionnez **Ajouter**  :

   :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/storage-role-assignment.png" alt-text="Schéma représentant l’architecture des points de terminaison Azure Site Recovery et privés.":::

1. Dans la page **Ajouter une attribution de rôle** , dans la liste **Rôle** , sélectionnez le rôle indiqué au début de cette section. Entrez le nom du coffre, puis sélectionnez **Enregistrer** .

   :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/storage-role-assignment-select-role.png" alt-text="Schéma représentant l’architecture des points de terminaison Azure Site Recovery et privés.":::

Après avoir ajouté ces autorisations, vous devez autoriser l’accès aux services de confiance Microsoft. Accédez à **Pare-feux et réseaux virtuels** et sélectionnez **Autoriser les services Microsoft approuvés à accéder à ce compte de stockage** dans **Exceptions** .

### <a name="protect-your-virtual-machines"></a>Protéger vos machines virtuelles

Une fois que vous avez terminé les tâches précédentes, poursuivez la configuration de votre infrastructure locale. Poursuivez par l’une des tâches suivantes : 

- [Déployer le serveur de configuration pour VMware et les machines physiques](./vmware-azure-deploy-configuration-server.md)
- [Configurer l’environnement Hyper-V pour la réplication](./hyper-v-azure-tutorial.md#set-up-the-source-environment)

Une fois l’installation terminée, activez la réplication pour vos machines sources. Ne configurez pas l’infrastructure tant que les points de terminaison privés du coffre n’ont pas été créés dans le réseau de contournement.

### <a name="create-private-dns-zones-and-add-dns-records-manually"></a>Créer des zones DNS privées et ajouter des enregistrements DNS manuellement

Si vous n’avez pas sélectionné l’option d’intégration à la zone DNS privée lorsque vous avez créé le point de terminaison privé pour le coffre, suivez les étapes de cette section.

Créez une zone DNS privée afin de permettre au fournisseur Site Recovery (pour les machines Hyper-V) ou au serveur de processus (pour les machines VMware/physiques) de résoudre les FQDN de liaison privée en adresses IP privées.

1. Créez une zone DNS privée.

   1. Dans la barre de recherche **Tous les services** , recherchez « zone DNS privée », puis sélectionnez **Zone DNS privées** dans les résultats :

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/search-private-dns-zone.png" alt-text="Schéma représentant l’architecture des points de terminaison Azure Site Recovery et privés.":::

   1. Dans la page **Zones DNS privées** , sélectionnez le bouton **Ajouter** pour commencer à créer une zone.

   1. Dans la page **Créer une zone DNS privée** , entrez les informations requises. Entrez **privatelink.siterecovery.windowsazure.com** comme nom de la zone DNS privée. Vous pouvez choisir n’importe quel groupe de ressources et n’importe quel abonnement.

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/create-private-dns-zone.png" alt-text="Schéma représentant l’architecture des points de terminaison Azure Site Recovery et privés.":::

   1. Passez à l’onglet **Vérifier \+ créer** pour vérifier et créer la zone DNS.

1. Liez la zone DNS privée à votre réseau virtuel.

   Vous devez maintenant lier la zone DNS privée que vous avez créée au contournement.

   1. Accédez à la zone DNS privée que vous avez créée à l’étape précédente, puis accédez à **Liens de réseau virtuel** dans le volet gauche. Sélectionnez **Ajouter** .

   1. Entrez les informations requises. Dans les listes **Abonnement** et **Réseau virtuel** , sélectionnez les détails qui correspondent au réseau de contournement. Laissez les valeurs par défaut dans les autres champs.

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/add-virtual-network-link.png" alt-text="Schéma représentant l’architecture des points de terminaison Azure Site Recovery et privés.":::

1. Ajoutez des enregistrements DNS.

   À présent que vous avez créé la zone DNS privée requise et le point de terminaison privé, vous devez ajouter des enregistrements DNS à votre zone DNS.

   > [!NOTE]
   > Si vous utilisez une zone DNS privée personnalisée, veillez à créer des entrées similaires, comme décrit à l’étape suivante.

   À cette étape, vous devez créer des entrées pour chaque FQDN (nom de domaine complet) de votre point de terminaison privé dans votre zone de DNS privée.

   1. Accédez à votre zone DNS privée, puis, dans le volet de gauche, accédez à **Vue d’ensemble** . Sélectionnez **Jeu d’enregistrements** pour commencer à ajouter des enregistrements.

   1. Dans la page **Ajouter un jeu d’enregistrements** qui s’ouvre, ajoutez une entrée pour chaque nom de domaine complet et adresse IP privée en tant qu’enregistrement de type **A** . Vous pouvez obtenir la liste des noms de domaine complets et des adresses IP dans la page **Point de terminaison privé** dans **Vue d’ensemble** . Comme indiqué dans la capture d’écran ci-dessous, le premier nom de domaine complet du point de terminaison privé est ajouté au jeu d’enregistrements dans la zone DNS privée.

      Ces noms de domaine complets correspondent à ce modèle : `{Vault-ID}-asr-pod01-{type}-.{target-geo-code}.siterecovery.windowsazure.com`

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/add-record-set.png" alt-text="Schéma représentant l’architecture des points de terminaison Azure Site Recovery et privés.":::

## <a name="next-steps"></a>Étapes suivantes

Des points de terminaison privés étant activés pour la réplication de vos machines virtuelles, consultez les articles ci-dessous pour obtenir des informations supplémentaires et connexes :

- [Déployer un serveur de configuration local](./vmware-azure-deploy-configuration-server.md)
- [Configurer la reprise d’activité après sinistre de machines virtuelles Hyper-V locales vers Azure](./hyper-v-azure-tutorial.md)