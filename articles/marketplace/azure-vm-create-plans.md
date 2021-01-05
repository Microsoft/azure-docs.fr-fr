---
title: Créer des plans pour une offre de machine virtuelle sur la Place de marché Azure
description: Découvrez comment créer des plans pour une offre de machine virtuelle sur la Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: mingshen-ms
ms.author: mingshen
ms.date: 10/19/2020
ms.openlocfilehash: 023dc877158c7074f46945893d40291e94ab2f09
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94629554"
---
# <a name="how-to-create-plans-for-a-virtual-machine-offer"></a>Comment créer des plans pour une offre de machine virtuelle

Dans la page **Vue d’ensemble du plan** (à partir du menu de navigation de gauche dans Espace partenaires), vous pouvez proposer différentes options de plan au sein d’une même offre. Une offre nécessite au moins un plan (précédemment appelé SKU), qui peut varier en termes de public de monétisation, de région Azure, de fonctionnalités ou d’images de machines virtuelles.

Vous pouvez créer jusqu’à 100 plans pour chaque offre : jusqu’à 45 peuvent être privés. En savoir plus sur les offres privées dans les [offres du marketplace commercial de Microsoft](private-offers.md).

Une fois que vous avez créé vos plans, sélectionnez l’onglet **Vue d’ensemble du plan** pour afficher :

- Noms des plans
- Modèles de licences
- Audience (publique ou privée)
- État actuel de la publication
- Actions disponibles

Les actions disponibles dans le volet **Vue d’ensemble du plan** varient en fonction de l’état actuel de votre plan.

- Si l’état du plan est Brouillon, sélectionnez **Supprimer le brouillon**.
- Si l’état du plan est Publié, sélectionnez **Arrêter la vente du plan** ou **Synchroniser le public privé**.

## <a name="create-a-new-plan"></a>Créer un plan

Sélectionnez **+ Créer un plan** en haut de la page.

Dans la boîte de dialogue **Nouveau plan**, entrez un **ID de plan** unique pour chaque plan contenu dans cette offre. Cet ID sera visible par les clients dans l’adresse web du produit. Utilisez uniquement des lettres minuscules et des chiffres, des tirets ou des traits de soulignement, dans la limite de 50 caractères.

> [!NOTE]
> L’ID de plan ne peut pas être changé une fois que vous avez sélectionné **Créer**.

Entrez un **nom de plan**. Les clients voient ce nom quand ils décident du plan à sélectionner dans votre offre. Créez un nom unique qui identifie clairement les différences entre chaque plan. Par exemple, vous pouvez entrer **Windows Server** avec des plans *Paiement à l’utilisation*, *BYOL (apportez votre propre licence)* , *Avancé* et *Entreprise*.

Sélectionnez **Create** (Créer). Cela ouvre la page **Configuration du plan**.

## <a name="plan-setup"></a>Configuration du plan

Définissez la configuration générale du type de plan, indiquez si le plan réutilisera la configuration technique d’un autre plan, et identifiez les régions Azure où le plan doit être disponible. Les options que vous choisissez ici déterminent les champs affichés dans les autres volets du même plan.

### <a name="reuse-technical-configuration"></a>Réutiliser la configuration technique

Si vous avez plusieurs plans du même type et que les packages qu’ils utilisent sont identiques, vous pouvez sélectionner l’option pour que **ce plan réutilise la configuration technique d’un autre plan**. Cette option vous permet de sélectionner l’un des autres plans du même type pour cette offre et de réutiliser sa configuration technique.

> [!NOTE]
> Si vous réutilisez la configuration technique d’un autre plan, l’onglet **Configuration technique** ne s’affiche plus pour ce plan. Les paramètres de la configuration technique de l’autre plan, y compris les mises à jour effectuées ultérieurement, seront également appliqués à ce plan. Ce paramètre n’est plus modifiable après la publication du plan.

### <a name="azure-regions"></a>Régions Azure

Votre plan doit être rendu disponible dans au moins une région Azure.

Sélectionnez **Azure Global** pour mettre votre plan à la disposition des clients de toutes les régions Azure qui disposent d’une intégration au marketplace commercial. Pour plus d’informations, consultez [Disponibilité géographique et prise en charge des devises](marketplace-geo-availability-currencies.md).

Sélectionnez **Azure Government** pour proposer votre plan dans la région [Azure Government](../azure-government/documentation-government-welcome.md). Cette région fournit un accès contrôlé aux clients d’entités gouvernementales fédérales, étatiques, locales ou tribales des États-Unis, ainsi qu’aux partenaires éligibles pour les servir. En tant qu’éditeur, il vous incombe de mettre en place l’ensemble des contrôles de conformité, des mesures de sécurité et des bonnes pratiques nécessaires. Azure Government utilise des réseaux et des centres de données qui sont physiquement isolés (situés aux États-Unis uniquement).

Avant de le publier dans [Azure Government](../azure-government/documentation-government-manage-marketplace-partners.md), testez et validez votre plan dans l’environnement, car certains points de terminaison peuvent être différents. Pour configurer et tester votre plan, demandez un compte d’essai à partir de la page [Microsoft Azure Government Trial](https://azure.microsoft.com/global-infrastructure/government/request/).

> [!NOTE]
> Une fois votre plan publié et disponible dans une région Azure spécifique, vous ne pouvez pas supprimer cette région.

### <a name="azure-government-certifications"></a>Certifications Azure Government

Cette option est visible uniquement si vous avez sélectionné **Azure Government** comme région Azure dans la section précédente.

Les services Azure Government gèrent des données soumises à certaines réglementations et exigences gouvernementales. Par exemple, FedRAMP, NIST 800.171 (DIB), ITAR, IRS 1075, DoD L4 et CJIS. Afin de renseigner les clients sur vos certifications pour ces programmes, vous pouvez fournir jusqu’à 100 liens décrivant ces certifications. Ces liens peuvent être des liens vers votre liste de plans directement sur le programme, ou des liens vers des descriptions de votre conformité avec ces certifications sur vos propres sites web. Ces liens sont visibles uniquement par les clients Azure Government.

Si vous effectuez des modifications, sélectionnez **Enregistrer le brouillon** avant de passer à l’onglet suivant du menu de navigation gauche **Référencement du plan**.

## <a name="plan-listing"></a>Liste des plans

Configurez les détails du plan. Ce volet affiche des informations spécifiques qui peuvent différer des autres plans de la même offre.

### <a name="plan-name"></a>Nom du plan

Ce champ est renseigné automatiquement avec le nom que vous avez donné à votre plan au moment de sa création. Ce nom apparaît sur la Place de marché Azure comme titre de ce plan. Il est limité à 100 caractères.

### <a name="plan-summary"></a>Résumé du plan

Fournissez un bref résumé de votre plan (pas de l’offre). Ce résumé est limité à 100 caractères.

### <a name="plan-description"></a>Description du plan

Décrivez ce qui rend ce plan logiciel unique, ainsi que les différences entre les plans de votre offre. Décrivez uniquement le plan (pas l’offre). La description du plan peut contenir jusqu’à 2 000 caractères.

Si vous effectuez des modifications, sélectionnez **Enregistrer le brouillon** avant de passer à l’onglet suivant du menu de navigation gauche **Tarification et disponibilité**.

## <a name="pricing-and-availability"></a>Tarification et disponibilité

Dans ce volet, vous configurez :

- Les marchés où ce plan est disponible. Chaque plan doit être disponible sur au moins un [marché](marketplace-geo-availability-currencies.md).
- Le tarif horaire.
- Si vous voulez que le plan soit visible à tous ou uniquement à des utilisateurs spécifiques (un public privé).

### <a name="markets"></a>Marchés

Chaque plan doit être disponible sur au moins un marché. La plupart des marchés sont sélectionnés par défaut. Pour modifier la liste, sélectionnez **Modifier les marchés** et cochez ou décochez les cases pour chaque emplacement de marché où ce plan doit (ou ne doit pas) être disponible à l’achat. Les utilisateurs des marchés sélectionnés peuvent toujours déployer l’offre dans toutes les régions Azure sélectionnées dans la section [« Configuration du plan »](#plan-setup).

Sélectionnez **Sélectionner uniquement les pays avec impôt versé par Microsoft** pour sélectionner les pays/régions dans lesquels Microsoft acquitte les taxes de vente et d’utilisation en votre nom. La publication en Chine se limite aux plans *gratuits* ou de type *Apportez votre propre licence* (BYOL).

Si vous avez déjà fixé des prix pour votre plan en dollar américain (USD) et ajouté un autre emplacement de marché, le prix de ce nouveau marché est calculé selon le taux de change en vigueur. Vérifiez toujours le prix de chaque Place de marché avant toute publication. Passez en revue la tarification en sélectionnant **Exporter les prix (xlsx)** après avoir enregistré vos modifications.

Quand vous supprimez un marché, les clients de ce marché utilisant des déploiements actifs ne pourront pas créer de nouveaux déploiements ou mettre à l’échelle leurs déploiements existants. Les déploiements existants ne sont pas affectés.

Cliquez sur **Save** (Enregistrer) pour continuer.

### <a name="pricing"></a>Tarifs

Pour le **modèle de licence**, sélectionnez **Plan avec facturation mensuelle à l’utilisation** afin de configurer les prix de ce plan, ou **BYOL (apportez votre propre licence)** pour permettre aux clients d’utiliser ce plan avec leur licence existante.

Pour un plan facturé mensuellement selon l’utilisation, utilisez l’une des trois options d’entrée de prix suivantes :

- **Par cœur** : indiquez le prix par cœur en USD. Microsoft calcule le prix par taille de cœur et le convertit dans les devises locales en appliquant le taux de change actuel.
- **Par taille de cœur** : indiquez les prix par taille de cœur en USD. Microsoft calcule le prix et le convertit dans les devises locales en appliquant le taux de change actuel.
- **Par marché et taille de cœur** : indiquez les prix de chaque taille de cœur pour tous les marchés. Vous pouvez importer les prix à partir d’une feuille de calcul.

Entrez un **prix par cœur**, puis sélectionnez **Prix par taille de cœur** pour afficher un tableau des calculs de prix/heure.

> [!NOTE]
> Enregistrez les modifications de la tarification pour autoriser l’exportation des données de tarification. Une fois qu’un prix pour un marché dans votre plan est publié, il ne peut pas être modifié ultérieurement. Pour vérifier que les prix sont corrects avant de les publier, exportez la feuille de calcul de prix et passez en revue les prix sur chaque marché.

### <a name="free-trial"></a>Version d’évaluation gratuite

Vous pouvez proposer à vos clients un **essai gratuit** d’un mois, de trois mois ou de six mois.

### <a name="plan-visibility"></a>Visibilité du plan

Vous pouvez concevoir chaque plan pour qu’il soit visible par tous les utilisateurs ou uniquement par un public présélectionné. Définissez l’appartenance à ce public restreint à l’aide des ID d’abonnement Azure.

**Public** : votre plan peut être consulté par tout le monde.

**Privé** : votre plan ne peut être consulté que par un public présélectionné. Une fois votre plan publié en tant que plan privé, vous pouvez mettre à jour le public visé ou rendre le plan public. Une fois que vous avez rendu un plan public, il doit rester public. Vous ne pouvez pas le reconvertir en plan privé.

Définissez le public qui aura accès à ce plan privé au moyen d’**ID d’abonnement Azure**. Le cas échéant, incluez une **description** de chaque ID d’abonnement Azure que vous attribuez. Ajoutez jusqu’à 10 ID d’abonnement manuellement ou jusqu’à 20 000 ID si vous importez une feuille de calcul CSV. Les ID d’abonnement Azure sont représentés par des GUID, où toutes les lettres doivent être en minuscules.

> [!NOTE]
> Un public privé ou restreint est différent du public de préversion que vous avez défini dans le volet **Préversion**. Un public de préversion peut accéder à votre offre *avant* sa publication sur la Place de marché Azure. La sélection d’un public privé ne s’applique qu’à un plan spécifique, tandis que le public de préversion peut voir tous les plans privés et publics dans le cadre de la validation.

Les offres privées ne sont pas prises en charge avec les abonnements Azure souscrits via un revendeur participant au programme des fournisseurs de solutions cloud (CSP).

### <a name="hide-plan"></a>Masquer le plan

Si vous comptez utiliser votre machine virtuelle uniquement de manière indirecte quand elle est référencée par un autre modèle de solution ou une application managée, cochez cette case pour publier la machine virtuelle tout en la masquant aux yeux des clients susceptibles de la rechercher ou d’y accéder directement.

Les plans masqués ne prennent pas en charge les liens de préversion.

Si vous effectuez des modifications, sélectionnez **Enregistrer le brouillon** avant de passer à l’onglet suivant du menu de navigation gauche **Configuration technique**.

## <a name="technical-configuration"></a>Configuration technique

Fournissez les images et autres propriétés techniques associées à ce plan.

> [!NOTE]
> Cet onglet ne s’affiche pas si vous avez configuré ce plan pour réutiliser les packages d’un autre plan dans l’onglet **Configuration du plan**.

### <a name="operating-system"></a>Système d'exploitation

Sélectionnez la famille de systèmes d’exploitation **Windows** ou **Linux**.

Sélectionnez la **version** Windows ou le **fournisseur** Linux.

Entrez un **nom convivial** pour le système d’exploitation. Ce nom est visible par les clients.

### <a name="recommended-vm-sizes"></a>Tailles de machines virtuelles recommandées

Sélectionnez le lien pour choisir jusqu’à six tailles recommandées de machine virtuelle à afficher sur Place de marché Azure.

### <a name="open-ports"></a>Ouvrir les ports

Ajoutez des ports publics ou privés sur une machine virtuelle déployée.

### <a name="properties"></a>Propriétés

Indiquez si votre machine virtuelle **prend en charge les performances réseau accélérées**. Pour plus d’informations, consultez [Performances réseau accélérées](https://go.microsoft.com/fwlink/?linkid=2124513).

### <a name="generations"></a>Générations

La génération d’une machine virtuelle définit le matériel virtuel qu’elle utilise. En fonction des besoins de votre client, vous pouvez publier une machine virtuelle de 1re génération, une machine virtuelle de 2e génération, ou les deux.

1. Lors de la création d’une offre, sélectionnez un **type de génération** et entrez les informations demandées :

    :::image type="content" source="./media/create-vm/azure-vm-generations-image-details-1.png" alt-text="Vue du menu déroulant Génération.":::

2. Pour ajouter une autre génération à un plan, sélectionnez **Ajouter une génération**…

    :::image type="content" source="./media/create-vm/azure-vm-generations-add.png" alt-text="Vue du lien « Ajouter une génération ».":::

    …et entrez les informations demandées :

    :::image type="content" source="./media/create-vm/azure-vm-generations-image-details-3.png" alt-text="Vue de la fenêtre des détails de la génération.":::

<!--    The **Generation ID** you choose will be visible to customers in places such as product URLs and ARM templates (if applicable). Use only lowercase, alphanumeric characters, dashes, or underscores; it cannot be modified once published.
-->
3. Pour mettre à jour une machine virtuelle existante pour laquelle une machine de 1re génération a déjà été publiée, modifiez les détails sur cette page **Configuration technique** :

    :::image type="content" source="./media/create-vm/azure-vm-generations-updating.png" alt-text="Vue de la page Configuration technique du plan.":::

Pour en savoir plus sur les différences entre les capacités de la 1re génération et de la 2e génération, consultez [Prise en charge des machines virtuelles de génération 2 dans Azure](../virtual-machines/generation-2.md).

### <a name="vm-images"></a>Images de machine virtuelle

Spécifiez une version de disque et l’URI de signature d’accès partagé (SAS) pour les images de machine virtuelle. Ajoutez jusqu’à 16 disques de données pour chaque image de machine virtuelle. Spécifiez une seule nouvelle version d’image par plan à chaque envoi. Quand une image a été publiée, vous ne pouvez pas la modifier, mais vous pouvez la supprimer. La suppression d’une version empêche les utilisateurs nouveaux et existants de déployer une nouvelle instance de la version supprimée.

Ces deux champs obligatoires sont affichés dans l’image précédente ci-dessus :

- **Version du disque** : version de l’image que vous fournissez.
- **Lien du disque dur virtuel du système d’exploitation** : emplacement dans votre compte de stockage Azure pour le disque dur virtuel du système d’exploitation. Pour obtenir l’URI SAS, consultez [Obtenir l’URI de la signature d’accès partagé pour vos images de machine virtuelle](azure-vm-get-sas-uri.md).

Les disques de données [sélectionnez **Ajouter un disque de données (16 maximum)** ] sont également des URI de signature d’accès partagé VHD qui sont stockés dans leurs comptes de stockage Azure. Ajoutez une seule image par envoi dans un plan.

Quel que soit le système d’exploitation que vous utilisez, ajoutez uniquement le nombre minimal de disques de données dont la solution a besoin. Pendant le déploiement, les clients ne peuvent pas supprimer les disques qui font partie d’une image, mais ils peuvent toujours ajouter des disques pendant ou après le déploiement.

Sélectionnez **Enregistrer le brouillon**, puis sélectionnez **← Vue d’ensemble du plan** en haut à gauche pour voir le plan que vous venez de créer.

## <a name="next-steps"></a>Étapes suivantes

- [Revendre via des fournisseurs de solutions cloud](azure-vm-create-resell-csp.md)
