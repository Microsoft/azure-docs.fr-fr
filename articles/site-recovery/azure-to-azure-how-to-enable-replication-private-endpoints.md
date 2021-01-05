---
title: Activer la réplication pour des points de terminaison privés dans Azure Site Recovery
description: Cet article décrit comment configurer la réplication pour des machines virtuelles disposant de points de terminaison privés d’une région Azure vers une autre à l’aide de Site Recovery.
author: Harsha-CS
ms.author: harshacs
ms.service: site-recovery
ms.topic: article
ms.date: 07/14/2020
ms.custom: references_regions
ms.openlocfilehash: 2db8d09511385039ea79ad1ed1f50177156f69c8
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92369928"
---
# <a name="replicate-machines-with-private-endpoints"></a>Répliquer des machines avec des points de terminaison privés

Azure Site Recovery vous permet d’utiliser des points de terminaison privés [Azure Private Link](../private-link/private-endpoint-overview.md) pour la réplication de vos machines depuis l’intérieur d’un réseau virtuel isolé. Un accès de point de terminaison privé à un coffre de récupération est pris en charge dans toutes les régions Azure Commerce et Secteur public.

Cet article fournit des instructions pour effectuer les étapes suivantes :

- Créer un coffre Recovery Services Sauvegarde Azure pour protéger vos machines
- Activer une identité managée pour le coffre et accorder les autorisations nécessaires pour accéder aux comptes de stockage client afin de répliquer le trafic des emplacements source à cible. L’accès de l’identité managée au stockage est nécessaire lors de la configuration de l’accès Azure Private Link au coffre.
- Apporter les modifications DNS nécessaires pour les points de terminaison privés
- Créer et approuver des points de terminaison privés pour un coffre à l’intérieur d’un réseau virtuel
- Créer des points de terminaison privés pour les comptes de stockage. Vous pouvez continuer à autoriser un accès public ou via un pare-feu au stockage en fonction des besoins. La création d’un point de terminaison privé pour l’accès au stockage n’est pas obligatoire pour Azure Site Recovery.
  
Vous trouverez ci-dessous une architecture de référence montrant dans quelle mesure le workflow de réplication change avec des points de terminaison privés.

:::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/architecture.png" alt-text="Architecture de référence pour Site Recovery avec des points de terminaison privés":::

## <a name="prerequisites-and-caveats"></a>Prérequis et mises en garde

- Les points de terminaison privés ne peuvent être créés que pour les nouveaux coffres Recovery Services qui ne contiennent pas d’éléments. Ainsi, les points de terminaison privés **doivent être créés avant l’ajout d’éléments au coffre** . Passez en revue la structure des tarifs des [points de terminaison privés](https://azure.microsoft.com/pricing/details/private-link/).
- Quand un point de terminaison privé est créé pour un coffre, ce dernier est verrouillé et **n’est pas accessible à partir de réseaux autres que ceux qui ont des points de terminaison privés** .
- Azure Active Directory ne prend pas en charge les points de terminaison privés. Ainsi, les adresses IP et les noms de domaine complets nécessaires pour qu’Azure Active Directory fonctionne dans une région doivent se voir autoriser un accès sortant à partir du réseau sécurisé. Vous pouvez également utiliser l’étiquette de groupe de sécurité réseau « Azure Active Directory » et des étiquettes de Pare-feu Azure pour autoriser l’accès à Azure Active Directory, le cas échéant.
- **Au moins sept adresses IP sont requises** dans les sous-réseaux de vos machines source et de vos machines de récupération. Quand vous créez un point de terminaison privé pour le coffre, Site Recovery crée cinq liaisons privées pour accéder à ses microservices. En outre, lorsque vous activez la réplication, elle ajoute deux liaisons privées supplémentaires pour le jumelage des régions source et cible.
- **Une adresse IP supplémentaire est requise** dans les sous-réseaux source et de récupération. Cette adresse IP est nécessaire uniquement lorsque vous devez utiliser des points de terminaison privés se connectant aux comptes de stockage de cache.
  Les points de terminaison privés pour le stockage peuvent uniquement être créés sur un compte de stockage de type v2 universel. Passez en revue la structure des tarifs du [transfert de données sur GPv2](https://azure.microsoft.com/pricing/details/storage/page-blobs/).

 ## <a name="creating-and-using-private-endpoints-for-site-recovery"></a>Création et utilisation de points de terminaison privés pour Site Recovery

 Cette section décrit les étapes de création et d’utilisation des points de terminaison privés pour le Azure Site Recovery à l’intérieur de vos réseaux virtuels.

> [!NOTE]
> Nous vous recommandons de suivre ces étapes dans l’ordre indiqué. Si vous ne le faites pas, le coffre rendu risque de ne pas pouvoir utiliser des points de terminaison privés, ce qui vous obligerait à recommencer la procédure avec un nouveau coffre.

## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

Un coffre Recovery Services est une entité qui contient les informations de réplication des machines et qui est utilisée pour déclencher des opérations de Site Recovery. Pour plus d’informations, consultez [Créer un coffre Recovery Services](./azure-to-azure-tutorial-enable-replication.md#create-a-recovery-services-vault).

## <a name="enable-the-managed-identity-for-the-vault"></a>Activer l’identité managée pour le coffre

Une [identité managée](../active-directory/managed-identities-azure-resources/overview.md) autorise le coffre à accéder aux comptes de stockage du client. Site Recovery doit accéder au stockage source, au stockage cible et aux comptes de stockage de cache/journal en fonction de l’exigence du scénario.
L’accès de l’identité managée est essentiel quand vous utilisez le service de liaisons privées pour le coffre.

1. Accédez à votre coffre Recovery Services. Sélectionnez **Identité** sous _Paramètres_ .

   :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/enable-managed-identity-in-vault.png" alt-text="Architecture de référence pour Site Recovery avec des points de terminaison privés":::

1. Cliquez sur le bouton bascule sous **État** pour le définir sur la valeur _Activé_ , puis sélectionnez **Enregistrer** .

1. Un **ID d’objet** est généré, indiquant que le coffre est maintenant inscrit auprès d’Azure Active Directory.

## <a name="create-private-endpoints-for-the-recovery-services-vault"></a>Créer des points de terminaison privés pour le coffre Recovery Services

Pour activer le basculement et la restauration automatique pour les machines virtuelles Azure, vous avez besoin de deux points de terminaison privés pour le coffre. Un point de terminaison privé pour la protection des machines dans le réseau source et un autre pour la reprotection des machines basculées sur le réseau de récupération.

Vous devez également créer un réseau virtuel de récupération dans votre région cible au cours de ce processus de configuration.

Créez le premier point de terminaison privé de votre coffre à l’intérieur de votre réseau virtuel source à l’aide du centre de liaison privé dans le portail ou via [Azure PowerShell](../private-link/create-private-endpoint-powershell.md). Créez le deuxième point de terminaison privé pour le coffre à l’intérieur de votre réseau de récupération. Voici les étapes à suivre pour créer le point de terminaison privé dans le réseau source. Répétez les mêmes instructions pour créer le deuxième point de terminaison privé.

1. Depuis la barre de recherche du portail Azure, recherchez et sélectionnez « Liaison privée ». Vous accédez ainsi au Centre de liaisons privées.

   :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/search-private-links.png" alt-text="Architecture de référence pour Site Recovery avec des points de terminaison privés":::

1. Dans la barre de navigation de gauche, sélectionnez **points de terminaison privés** . Une fois dans le volet Points de terminaison privés, sélectionnez **\+Ajouter** pour démarrer le processus de création d’un point de terminaison privé pour votre coffre.

   :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/create-private-endpoints.png" alt-text="Architecture de référence pour Site Recovery avec des points de terminaison privés":::

1. Une fois dans l’expérience de création d’un point de terminaison privé, vous devez entrer des informations requises pour créer la connexion de votre point de terminaison privé.

   1. **Paramètres de base** : Entrez les informations de base de vos points de terminaison privés. La région doit être la même que celle des machines sources.

      :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/create-private-endpoints-basic-tab.png" alt-text="Architecture de référence pour Site Recovery avec des points de terminaison privés":::

   1. **Ressource**  : Dans cet onglet, vous devez mentionner la ressource PaaS (plateforme en tant que service) pour laquelle vous souhaitez créer votre connexion. Sélectionnez _Microsoft.RecoveryServices/vaults_ à partir du **Type de ressource** pour l’abonnement sélectionné. Ensuite, choisissez le nom de votre coffre Recovery Services en guise de **Ressource** et définissez _Azure Site Recovery_ comme **Sous-ressource cible** .

      :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/create-private-endpoints-resource-tab.png" alt-text="Architecture de référence pour Site Recovery avec des points de terminaison privés":::

   1. **Configuration**  : Dans Configuration, spécifiez le réseau virtuel et le sous-réseau où vous souhaitez que le point de terminaison privé soit créé. Ce réseau virtuel est le réseau sur lequel la machine virtuelle est présente. Activez l’intégration à la zone DNS privée en sélectionnant **Oui** . Choisissez une zone DNS déjà créée ou créez-en une. La sélection de l’option **Oui** associe automatiquement la zone au réseau au réseau virtuel source et ajoute les enregistrements DNS nécessaires à la résolution DNS des adresses IP et des noms de domaine complets créés pour le point de terminaison privé.

      Veillez à créer une zone DNS pour chaque nouveau point de terminaison privé qui se connecte au même coffre. Si vous choisissez une zone DNS privée existante, les enregistrements CNAME précédents sont remplacés. Reportez-vous à [Guide des points de terminaisons privés](../private-link/private-endpoint-overview.md#private-endpoint-properties) avant de continuer.

      Si votre environnement a un modèle hub-and-spoke, vous n’avez besoin que d’un seul point de terminaison privé et d’une seule zone DNS privée pour l’ensemble de l’installation, car l’appairage est déjà activé pour tous vos réseaux virtuels. Pour plus d’informations, consultez [Intégration DNS avec des points de terminaison privés](../private-link/private-endpoint-dns.md#virtual-network-workloads-without-custom-dns-server).

      Pour créer manuellement la zone DNS privée, suivez les étapes décrites dans [Créer des zones DNS privées et ajouter des enregistrements DNS manuellement](#create-private-dns-zones-and-add-dns-records-manually).

      :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/create-private-endpoints-configuration-tab.png" alt-text="Architecture de référence pour Site Recovery avec des points de terminaison privés":::

   1. **Étiquettes**  : Si vous le souhaitez, vous pouvez ajouter des étiquettes à votre point de terminaison privé.

   1. **Vérifier \+ créer**  : Une fois la validation terminée, sélectionnez **Créer** pour créer le point de terminaison privé.

Une fois le point de terminaison privé créé, cinq noms de domaine complets sont ajoutés à celui-ci. Ces liaisons permettent aux machines du réseau virtuel d’obtenir un accès à tous les microservices Site Recovery requis dans le contexte du coffre. Plus tard, lorsque vous activez la réplication, deux noms de domaine complets supplémentaires sont ajoutés au même point de terminaison privé.

Les cinq noms de domaine sont mis en forme selon le modèle suivant :

`{Vault-ID}-asr-pod01-{type}-.{target-geo-code}.siterecovery.windowsazure.com`

## <a name="approve-private-endpoints-for-site-recovery"></a>Approuver des points de terminaison privés pour Site Recovery

Si l’utilisateur qui crée le point de terminaison privé est également le propriétaire du coffre Recovery Services, le point de terminaison privé créé est approuvé automatiquement en quelques minutes. Dans le cas contraire, le propriétaire du coffre doit approuver le point de terminaison privé avant que vous ne l’utilisiez. Pour approuver ou rejeter une connexion de point de terminaison privée demandée, accédez à **Connexions des points de terminaison privés** sous « Paramètres » dans la page du coffre de récupération.

Vous pouvez accéder à la ressource de point de terminaison privé pour passer en revue l’état de la connexion avant de continuer.

:::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/vault-private-endpoint-connections.png" alt-text="Architecture de référence pour Site Recovery avec des points de terminaison privés":::

## <a name="optional-create-private-endpoints-for-the-cache-storage-account"></a><a name="create-private-endpoints-for-the-cache-storage-account"></a>(Facultatif) Créer des points de terminaison privés pour le compte de stockage de cache

Un point de terminaison privé vers le stockage Azure peut être utilisé. La création de points de terminaison privés pour l’accès au stockage est _facultative_ pour la réplication Azure Site Recovery. Lors de la création d’un point de terminaison privé pour le stockage, les conditions suivantes s’appliquent :

- Vous avez besoin d’un point de terminaison privé pour le compte de stockage de cache/journal dans votre réseau virtuel source.
- Vous avez besoin d’un deuxième point de terminaison privé au moment de la reprotection des machines basculées sur le réseau de récupération. Ce point de terminaison privé est destiné au nouveau compte de stockage créé dans la région cible.

> [!NOTE]
> Le point de terminaison privé pour le stockage peut uniquement être créé sur un compte de stockage **v2 universel** . Pour plus d’informations sur les tarifs, consultez les [prix des objets blob de pages Standard](https://azure.microsoft.com/pricing/details/storage/page-blobs/).

Suivez les [instructions pour la création d’un stockage privé](../private-link/tutorial-private-endpoint-storage-portal.md#create-storage-account-with-a-private-endpoint) afin de créer un compte de stockage avec un point de terminaison privé. Veillez à sélectionner **Oui** pour l’intégration à la zone DNS privée. Sélectionnez une zone DNS déjà créée ou créez-en une.

## <a name="grant-required-permissions-to-the-vault"></a>Accorder les autorisations nécessaires au coffre

Si vos machines virtuelles utilisent des disques managés, vous devez accorder les autorisations d’identité gérée uniquement aux comptes de stockage de cache. Si vos machines virtuelles utilisent des disques managés, vous devez accorder les autorisations d’identité gérée uniquement aux comptes de stockage source, de cache et cible. Dans ce cas, vous devez créer le compte de stockage cible à l’avance.

Avant d’activer la réplication des machines virtuelles, vous devez accorder à l’identité managée du coffre les autorisations de rôle suivantes en fonction du type de compte de stockage :

- Comptes de stockage basés sur Resource Manager (type Standard) :
  - [Contributeur](../role-based-access-control/built-in-roles.md#contributor)
  - [Contributeur aux données Blob du stockage](../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)
- Comptes de stockage basés sur Resource Manager (type Premium) :
  - [Contributeur](../role-based-access-control/built-in-roles.md#contributor)
  - [Propriétaire des données Blob du stockage](../role-based-access-control/built-in-roles.md#storage-blob-data-owner)
- Comptes de stockage Classic :
  - [Contributeur de compte de stockage classique](../role-based-access-control/built-in-roles.md#classic-storage-account-contributor)
  - [Rôle de service d’opérateur de clé de compte de stockage classique](../role-based-access-control/built-in-roles.md#classic-storage-account-key-operator-service-role)

Les étapes suivantes expliquent comment ajouter une attribution de rôle à vos comptes de stockage, une à la fois :

1. Accédez au compte de stockage et à **Contrôle d’accès (IAM)** sur le côté gauche de la page.

1. Une fois dans **Contrôle d’accès (IAM)** , dans la zone « Ajouter une attribution de rôle », sélectionnez **Ajouter** .

   :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/storage-role-assignment.png" alt-text="Architecture de référence pour Site Recovery avec des points de terminaison privés":::

1. Dans la page latérale « Ajouter une attribution de rôle », choisissez le rôle parmi la liste plus haut dans la liste déroulante **Rôle** . Entrez le **nom** du coffre, puis sélectionnez **Enregistrer** .

   :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/storage-role-assignment-select-role.png" alt-text="Architecture de référence pour Site Recovery avec des points de terminaison privés":::

En plus de ces autorisations, les services de confiance MS doivent se voir accorder l’accès. Accédez à « Pare-feux et réseaux virtuels » et cochez la case « Autoriser les services Microsoft approuvés à accéder à ce compte de stockage » dans **Exceptions** .

## <a name="protect-your-virtual-machines"></a>Protéger vos machines virtuelles

Une fois toutes les configurations ci-dessus terminées, continuez avec l’activation de la réplication pour vos machines virtuelles. Toutes les opérations de Site Recovery fonctionnent sans aucune étape supplémentaire si l’intégration DNS a été utilisée lors de la création de points de terminaison privés sur le coffre. Toutefois, si les zones DNS sont créées et configurées manuellement, vous devez effectuer des étapes supplémentaires pour ajouter des enregistrements DNS spécifiques dans les zones DNS source et cible après avoir activé la réplication. Pour plus d’informations et pour connaître les étapes à suivre, consultez [Créer des zones DNS privées et ajouter des enregistrements DNS manuellement](#create-private-dns-zones-and-add-dns-records-manually).

## <a name="create-private-dns-zones-and-add-dns-records-manually"></a>Créer des zones DNS privées et ajouter des enregistrements DNS manuellement

Si vous n’avez pas sélectionné l’option d’intégration à la zone DNS privée au moment de la création d’un point de terminaison privé pour le coffre, suivez les étapes de cette section.

Créez une zone DNS privée pour permettre à l’agent de mobilité de résoudre les noms de domaine complets de liaison privée en adresses IP privées.

1. Créer une zone DNS privée

   1. Dans la barre de recherche **Tous les services** , recherchez « Zone DNS privée », puis, dans la liste déroulante, sélectionnez « Zones DNS privées ».

      :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/search-private-dns-zone.png" alt-text="Architecture de référence pour Site Recovery avec des points de terminaison privés":::

   1. Une fois dans la page « Zones DNS privées », sélectionnez le bouton **\+Ajouter** pour commencer à créer une zone.

   1. Dans la page « Créer une zone DNS privée », entrez les informations requises. Entrez `privatelink.siterecovery.windowsazure.com` comme nom de la zone DNS privée. Vous pouvez choisir n’importe quel groupe de ressources et n’importe quel abonnement pour la créer.

      :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/create-private-dns-zone.png" alt-text="Architecture de référence pour Site Recovery avec des points de terminaison privés":::

   1. Passez à l’onglet **Vérifier \+ créer** pour vérifier et créer la zone DNS.

1. Lier la zone DNS privée à votre réseau virtuel

   Vous devez à présent lier les zones DNS privées créées précédemment au réseau virtuel sur lequel se trouvent actuellement les serveurs. Vous devez également lier la zone DNS privée au réseau virtuel cible à l’avance.

   1. Accédez à la zone DNS privée que vous avez créée à l’étape précédente, puis, sur le côté gauche de la page, accédez à **Liens de réseau virtuel** . Ensuite, sélectionnez le bouton **\+Ajouter** .

   1. Renseignez les champs obligatoires. Vous devez renseigner les champs **Subscription** (Abonnement) et **Virtual network** (Réseau virtuel) avec les paramètres correspondants du réseau virtuel sur lequel se trouvent vos serveurs. Ne modifiez pas les autres champs.

      :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/add-virtual-network-link.png" alt-text="Architecture de référence pour Site Recovery avec des points de terminaison privés":::

1. Ajoutez des enregistrements DNS.

   Une fois que vous avez créé les zones DNS privées requises et les points de terminaison privés, vous devez ajouter des enregistrements DNS à vos zones DNS.

   > [!NOTE]
   > Si vous utilisez une zone DNS privée personnalisée, assurez-vous que des entrées similaires sont créées, comme indiqué ci-dessous.

   Cette étape nécessite que vous créiez des entrées dans votre zone de DNS privée pour chaque nom de domaine complet dans votre point de terminaison privé.

   1. Accédez à votre zone DNS privée, puis, sur le côté gauche de la page, accédez à la section **Vue d’ensemble** . Cliquez ensuite sur **\+Jeu d’enregistrements** pour commencer à ajouter des enregistrements.

   1. Dans la page « Ajouter un jeu d’enregistrements » qui s’ouvre, ajoutez une entrée pour chaque nom de domaine complet et adresse IP privée en tant qu’enregistrement de type _A_ . La liste des noms de domaine complets et des adresses IP peut être obtenue à partir de la page « Point de terminaison privé » dans **Vue d’ensemble** . Comme indiqué dans l’exemple ci-dessous, le premier nom de domaine complet du point de terminaison privé est ajouté au jeu d’enregistrements dans la zone DNS privée.

      Ces noms de domaine complets correspondent au modèle : `{Vault-ID}-asr-pod01-{type}-.{target-geo-code}.siterecovery.windowsazure.com`

      :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/add-record-set.png" alt-text="Architecture de référence pour Site Recovery avec des points de terminaison privés":::

   > [!NOTE]
   > Une fois la réplication activée, deux noms de domaine complets supplémentaires sont créés sur les points de terminaison privés dans les deux régions. Veillez également à ajouter les enregistrements DNS pour ces noms de domaine complets nouvellement créés.

## <a name="next-steps"></a>Étapes suivantes

Des points de terminaison privés étant activés pour la réplication de vos machines virtuelles, consultez les pages ci-dessous pour obtenir des informations supplémentaires et connexes :

- [Répliquer des machines virtuelles Azure dans une autre région Azure](./azure-to-azure-how-to-enable-replication.md)
- [Tutoriel : Configurer la récupération d’urgence pour les machines virtuelles Azure](./azure-to-azure-tutorial-enable-replication.md)