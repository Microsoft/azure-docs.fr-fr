---
title: Intégrer Azure Key Vault à Azure Policy
description: Découvrir comment intégrer Azure Key Vault à Azure Policy
author: ShaneBala-keyvault
ms.author: sudbalas
ms.date: 10/15/2020
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.openlocfilehash: 6ac4d0e0744bfc82a686671234e013b2dd717146
ms.sourcegitcommit: dd45ae4fc54f8267cda2ddf4a92ccd123464d411
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92927751"
---
# <a name="integrate-azure-key-vault-with-azure-policy"></a>Intégrer Azure Key Vault à Azure Policy

[Azure Policy](../../governance/policy/index.yml) est un outil de gouvernance qui permet aux utilisateurs d’auditer et de gérer leur environnement Azure à grande échelle. Azure Policy offre la possibilité de placer des barrières sur les ressources Azure afin de s’assurer que celles-ci sont conformes aux règles de stratégie affectées. Il permet aux utilisateurs d’effectuer des audits, l’application en temps réel et la correction de leur environnement Azure. Les résultats des audits réalisés par la stratégie sont mis à disposition des utilisateurs dans un tableau de bord de conformité qui présente une vue détaillée des ressources et composants conformes et non conformes.  Pour plus d’informations, reportez-vous à la rubrique [Présentation du service Azure Policy](../../governance/policy/overview.md).

Exemples de scénarios d’utilisation :

- Vous souhaitez améliorer la position de sécurité de votre entreprise en implémentant des exigences en matière de taille de clé minimale et de durée de validité maximale des certificats présents dans les coffres de clés de votre entreprise, mais vous ne savez pas quelles équipes sont conformes et lesquelles ne le sont pas.
- Vous ne disposez actuellement d’aucune solution pour effectuer un audit au sein de votre organisation ou vous effectuez des audits manuels de votre environnement en demandant à chaque équipe de votre organisation d’établir un état de leur conformité. Vous recherchez un moyen d’automatiser cette tâche, d’effectuer des audits en temps réel et de garantir l’exactitude de l’audit.
- Vous voulez appliquer les stratégies de sécurité de votre entreprise et empêcher la création de certificats auto-signés, mais vous ne disposez d’aucun moyen automatisé pour bloquer leur création. 
- Vous souhaitez assouplir certaines exigences pour vos équipes de test, tout en conservant des contrôles étroits sur votre environnement de production. Vous recherchez une méthode automatisée simple pour séparer l’application de vos ressources.
- Vous voulez être sûr de pouvoir annuler l’application des nouvelles stratégies en cas de problème de site actif. Vous recherchez une solution en un clic pour désactiver l’application de la stratégie. 
- Vous faites appel à une solution tierce pour réaliser l’audit de votre environnement et vous souhaitez utiliser une offre Microsoft interne.

## <a name="types-of-policy-effects-and-guidance"></a>Types d’effets des stratégies et conseils

**Audit**  : Lorsque l’effet d’une stratégie est défini sur audit, celle-ci n’entraînera aucun changement cassant sur votre environnement. Elle vous informera simplement du fait que des composants tels que des certificats ne sont pas conformes aux définitions de la stratégie au sein d’une étendue spécifiée, en les marquant comme non conformes dans le tableau de bord de conformité aux stratégies. L’audit est le paramètre par défaut si aucun effet de stratégie n’est sélectionné.

**Refuser**  : Lorsque l’effet d’une stratégie est défini sur refuser, celle-ci bloque la création de nouveaux composants tels que des certificats et bloque les nouvelles versions de composants existants qui ne sont pas conformes à la définition de stratégie. Les ressources non conformes existantes au sein d’un coffre de clés ne sont pas affectées. Les capacités « audit » continuent de fonctionner.

## <a name="available-built-in-policy-definitions"></a>Définitions de stratégie « prédéfinies » disponibles

La solution Key Vault a créé un ensemble de stratégies qui peuvent être utilisées pour gérer les objets clés, certificats et secrets. Ces stratégies sont « prédéfinies », autrement dit elles peuvent être activées sans que vous n’ayez à écrire de code JSON personnalisé et elles sont prêtes à être attribuées dans le portail Azure. Vous pouvez cependant personnaliser certains paramètres pour répondre aux besoins de votre organisation.

# <a name="certificate-policies"></a>[Stratégies du certificat](#tab/certificates)

### <a name="certificates-should-have-the-specified-maximum-validity-period-preview"></a>La période de validité maximale des certificats doit être spécifiée (préversion)

Cette stratégie vous permet de gérer la durée de validité maximale de vos certificats stockés dans le coffre de clés. La limitation de la durée de validité maximale de vos certificats est une pratique de sécurité recommandée. Si une clé privée de votre certificat devenait compromise sans que cela ne soit détecté, l’utilisation de certificats à durée de vie limitée réduit le délai d’exécution des dégâts en cours et également la valeur du certificat pour un attaquant.

### <a name="certificates-should-use-allowed-key-types-preview"></a>Les certificats doivent utiliser des types de clés autorisés (préversion)

Cette stratégie vous permet de restreindre le type de certificats acceptés dans votre coffre de clés. Vous pouvez utiliser cette stratégie pour vous assurer que les clés privées de vos certificats sont RSA, ECC ou sauvegardées par HSM. Vous pouvez choisir les types de certificats autorisés dans la liste suivante.

- RSA
- RSA - HSM
- ECC
- ECC - HSM

### <a name="certificates-should-have-the-specified-lifetime-action-triggers-preview"></a>Les certificats doivent avoir les déclencheurs d’action de durée de vie spécifiés (préversion)

Cette stratégie vous permet de gérer l’action de durée de vie spécifiée pour les certificats qui expirent dans un certain nombre de jours ou qui ont atteint un certain pourcentage de durée de vie.

### <a name="certificates-should-be-issued-by-the-specified-integrated-certificate-authority-preview"></a>Les certificats doivent être émis par l’autorité de certification intégrée spécifiée (préversion)

Si vous utilisez une autorité de certification intégrée Key Vault (Digicert ou GlobalSign) et que vous voulez que les utilisateurs fassent appel à l’un de ces deux fournisseurs, vous pouvez utiliser cette stratégie pour mener un audit ou appliquer votre sélection. Cette stratégie peut également être utilisée pour auditer ou refuser la création de certificats auto-signés dans le coffre de clés.

### <a name="certificates-should-be-issued-by-the-specified-non-integrated-certificate-authority-preview"></a>Les certificats doivent être émis par l’autorité de certification non intégrée spécifiée (préversion)

Si vous utilisez une autorité de certification interne ou non intégrée à Key Vault et que vous voulez que les utilisateurs fassent appel à une autorité de certification figurant dans une liste que vous fournissez, vous pouvez utiliser cette stratégie pour créer une liste autorisée d’autorités de certification par nom d’émetteur. Cette stratégie peut également être utilisée pour auditer ou refuser la création de certificats auto-signés dans le coffre de clés.

### <a name="certificates-using-elliptic-curve-cryptography-should-have-allowed-curve-names-preview"></a>Les certificats utilisant un chiffrement à courbe elliptique doivent avoir des noms de courbes autorisés (préversion)

Si vous utilisez des certificats à courbe elliptique ou ECC, vous pouvez personnaliser une liste autorisée de noms de courbes à partir de la liste ci-dessous. L’option par défaut autorise tous les noms de courbes suivants.

- P-256
- P-256K
- P-384
- P-521

## <a name="certificates-using-rsa-cryptography-manage-minimum-key-size-for-rsa-certificates-preview"></a>Les certificats utilisant le chiffrement RSA gèrent la taille de clé minimale pour les certificats RSA (préversion)

Si vous utilisez des certificats RSA, vous pouvez choisir une taille de clé minimale requise pour vos certificats. Vous pouvez sélectionner une option dans la liste ci-dessous.

- 2048 bits
- 3072 bits
- 4096 bits

## <a name="manage-certificates-that-are-within-a-specified-number-of-days-of-expiration-preview"></a>Gérer les certificats arrivés à un nombre de jours spécifié avant expiration (préversion)

Votre service peut subir une panne si un certificat qui n’est pas surveillé correctement n’est pas pivoté avant son expiration. Cette stratégie est critique pour vous assurer que vos certificats stockés dans un coffre de clés sont sous surveillance. Nous vous recommandons d’appliquer cette stratégie plusieurs fois avec différents seuils d’expiration, par exemple 180, 90, 60 et 30 jours. Cette stratégie peut être utilisée pour surveiller et trier l’expiration des certificats dans votre organisation.

# <a name="key-policies"></a>[Stratégies de clé](#tab/keys)

### <a name="keys-should-not-be-active-for-longer-than-the-specified-number-of-days-preview"></a>Les clés ne doivent pas être actives pendant une durée supérieure au nombre de jours spécifié (préversion)

Si vous souhaitez vous assurer que vos clés n’ont pas été actives pendant plus qu’un nombre spécifié de jours, vous pouvez utiliser cette stratégie pour auditer la durée pendant laquelle votre clé a été active.

**Si votre clé a une date d’activation définie** , cette stratégie calcule le nombre de jours écoulés depuis la **date d’activation** de la clé jusqu’à la date actuelle. Si le nombre de jours dépasse le seuil que vous définissez, la clé est marquée comme non conforme à la stratégie.

**Si votre clé n’a pas de date d’activation définie** , cette stratégie calcule le nombre de jours écoulés depuis la **date de création** de la clé jusqu’à la date actuelle. Si le nombre de jours dépasse le seuil que vous définissez, la clé est marquée comme non conforme à la stratégie.

### <a name="keys-should-be-the-specified-cryptographic-type-rsa-or-ec-preview"></a>Les clés doivent être du type de chiffrement spécifié, RSA ou EC (préversion)

Cette stratégie vous permet de restreindre le type des certificats pouvant figurer dans votre coffre de clés. Vous pouvez utiliser cette stratégie pour vous assurer que vos clés sont de type RSA, ECC ou sauvegardées avec HSM. Vous pouvez choisir les types de certificats autorisés dans la liste suivante.

- RSA
- RSA - HSM
- ECC
- ECC - HSM

### <a name="keys-using-elliptic-curve-cryptography-should-have-the-specified-curve-names-preview"></a>Les clés utilisant un chiffrement à courbe elliptique doivent avoir les noms de courbes spécifiés (préversion)

Si vous utilisez un chiffrement à courbe elliptique ou des clés ECC, vous pouvez personnaliser une liste autorisée de noms de courbes à partir de la liste ci-dessous. L’option par défaut autorise tous les noms de courbes suivants.

- P-256
- P-256K
- P-384
- P-521

### <a name="keys-should-have-expirations-dates-set-preview"></a>Les clés doivent avoir des dates d’expiration définies (préversion)

Cette stratégie audite toutes les clés de vos coffres de clés et signale les clés qui n’ont pas de date d’expiration définie comme non conformes. Vous pouvez également utiliser cette stratégie pour bloquer la création de clés qui n’ont pas de date d’expiration définie.

### <a name="keys-should-have-more-than-the-specified-number-of-days-before-expiration-preview"></a>Les clés doivent avoir une durée de vie supérieure au nombre spécifié de jours avant l’expiration (préversion)

Si une clé a une durée de vie trop proche de l’expiration, un délai organisationnel pour la rotation de la clé peut occasionner une interruption. Les clés doivent faire l’objet d’une rotation un nombre spécifié de jours avant leur expiration, afin d’offrir suffisamment de temps pour réagir en cas de défaillance. Cette stratégie audite les clés dont la date d’expiration est proche, et vous permet de définir ce seuil en jours. Vous pouvez également l’utiliser pour empêcher la création de clés trop proches de leur date d’expiration.

### <a name="keys-should-be-backed-by-a-hardware-security-module-preview"></a>Les clés doivent être adossées à un module de sécurité matériel ou HSM (préversion)

Un HSM est un module de sécurité matériel qui stocke des clés. Un HSM fournit une couche physique de protection des clés de chiffrement. La clé de chiffrement ne peut pas quitter un HSM physique, ce qui offre un niveau de sécurité supérieur à celui d’une clé logicielle. Certaines organisations ont des exigences de conformité qui rendent obligatoire l’utilisation de clés HSM. Utilisez cette stratégie pour auditer les clés stockées dans votre coffre de clés qui ne sont pas sauvegardées par HSM. Vous pouvez également utiliser cette stratégie pour bloquer la création de clés non sauvegardées par HSM. Cette stratégie s’applique à tous les types de clés, tant RSA qu’ECC.

### <a name="keys-using-rsa-cryptography-should-have-a-specified-minimum-key-size-preview"></a>Les clés utilisant le chiffrement RSA doivent avoir une taille minimale spécifiée (préversion)

L’utilisation de clés RSA de taille inférieure n’est pas une pratique de conception sécurisée. Il se peut que vous soyez soumis à des normes d’audit et de certification qui rendent obligatoire l’utilisation d’une taille de clé minimale. La stratégie suivante vous permet de définir une exigence de taille minimale de clé sur votre coffre de clés. Vous pouvez auditer les clés qui ne répondent pas à cette exigence. Vous pouvez également utiliser cette stratégie pour bloquer la création de clés qui ne satisfont pas à l’exigence de taille minimale de clé.

### <a name="keys-should-have-the-specified-maximum-validity-period-preview"></a>La période de validité maximale doit être spécifiée pour les clés (préversion)

Gérez les exigences en matière de conformité de votre organisation en spécifiant le nombre maximal de jours pendant lesquels une clé peut être valide au sein de votre coffre de clés. Les clés qui sont valides au-delà du seuil que vous définissez seront marquées comme non conformes. Vous pouvez également utiliser cette stratégie pour bloquer la création de clés dont la date d’expiration est postérieure à la période de validité maximale que vous spécifiez.

# <a name="secret-policies"></a>[Stratégies de secret](#tab/secrets)

### <a name="secrets-should-not-be-active-for-longer-than-the-specified-number-of-days-preview"></a>Les secrets ne doivent pas être actifs pendant une période plus longue que le nombre spécifié de jours (préversion)

Si vous souhaitez vous assurer que vos secrets n’ont pas été actifs pendant plus qu’un nombre spécifié de jours, vous pouvez utiliser cette stratégie pour auditer la durée pendant laquelle votre secret a été actif.

**Si votre secret a une date d’activation définie** , cette stratégie calcule le nombre de jours écoulés depuis la **date d’activation** du secret jusqu’à la date actuelle. Si le nombre de jours dépasse le seuil que vous définissez, le secret est marqué comme non conforme à la stratégie.

**Si votre secret n’a pas de date d’activation définie** , cette stratégie calcule le nombre de jours écoulés depuis la **date de création** du secret jusqu’à la date actuelle. Si le nombre de jours dépasse le seuil que vous définissez, le secret est marqué comme non conforme à la stratégie.

### <a name="secrets-should-have-content-type-set-preview"></a>Les secrets doivent avoir un type de contenu défini (préversion)

Tout fichier de texte brut ou encodé peut être stocké en tant que secret de coffre de clés. Toutefois, votre organisation peut définir différentes stratégies de rotation et restrictions sur les mots de passe, les chaînes de connexion ou les certificats stockés en tant que clés. Une balise de type de contenu peut aider un utilisateur à voir ce qui est stocké dans un objet secret sans lire la valeur du secret. Vous pouvez utiliser cette stratégie pour auditer des secrets qui n’ont pas de balise de type de contenu définie. Vous pouvez également utiliser cette stratégie pour empêcher la création de secrets si aucune balise de type de contenu n’est définie.

### <a name="secrets-should-have-expiration-date-set-preview"></a>Les secrets doivent avoir une date d’expiration définie (préversion)

Cette stratégie audite tous les secrets de vos coffres de clés et signale les secrets qui n’ont pas de date d’expiration définie comme non conformes. Vous pouvez également utiliser cette stratégie pour bloquer la création de secrets qui n’ont pas de date d’expiration définie.

### <a name="secrets-should-have-more-than-the-specified-number-of-days-before-expiration-preview"></a>Les secrets doivent avoir une durée de vie supérieure au nombre spécifié de jours avant l’expiration (préversion)

Si un secret a une durée de vie trop proche de l’expiration, un délai organisationnel pour la rotation du secret peut occasionner une interruption. Les secrets doivent faire l’objet d’une rotation un nombre spécifié de jours avant leur expiration, afin d’offrir suffisamment de temps pour réagir en cas de défaillance. Cette stratégie audite les secrets dont la date d’expiration est proche, et vous permet de définir ce seuil en jours. Vous pouvez également l’utiliser pour empêcher la création de secrets trop proches de leur date d’expiration.

### <a name="secrets-should-have-the-specified-maximum-validity-period-preview"></a>La période de validité maximale doit être spécifiée pour les secrets (préversion)

Gérez les exigences en matière de conformité de votre organisation en spécifiant le nombre maximal de jours pendant lesquels un secret peut être valide au sein de votre coffre de clés. Les secrets qui sont valides au-delà du seuil que vous définissez seront marqués comme non conformes. Vous pouvez également utiliser cette stratégie pour bloquer la création de secrets dont la date d’expiration est postérieure à la période de validité maximale que vous spécifiez.

---

## <a name="example-scenario"></a>Exemple de scénario

Vous gérez un coffre de clés utilisé par plusieurs équipes qui contient 100 certificats et vous voulez être sûr qu’aucun des certificats du coffre de clés n’est valide plus de 2 ans.

1. Vous attribuez la stratégie **La période de validité maximale des certificats doit être spécifiée** , spécifiez que la durée de validité maximale d’un certificat est 24 mois, puis définissez l’effet de la stratégie sur « audit ». 
1. En consultant le [rapport de conformité sur le portail Azure](#view-compliance-results), vous constatez que 20 certificats sont non conformes et valides plus de 2 ans, et que les autres certificats sont conformes. 
1. Vous contactez les propriétaires de ces certificats et les informez de la nouvelle exigence de sécurité spécifiant que les certificats ne doivent pas être valides plus de 2 ans. Certaines équipes répondent et 15 des certificats sont renouvelés avec une durée de validité maximale de 2 ans ou moins. Les autres équipes ne répondent pas et il reste 5 certificats non conformes dans votre coffre de clés.
1. Vous remplacez l’effet de la stratégie attribuée par « refuser ». Les 5 certificats non conformes ne sont pas révoqués et continuent à fonctionner. Toutefois, ils ne peuvent pas être renouvelés avec une durée de validité supérieure à 2 ans. 

## <a name="enabling-and-managing-a-key-vault-policy-through-the-azure-portal"></a>Activer et gérer une stratégie Key Vault via le portail Azure

### <a name="select-a-policy-definition"></a>Sélectionner une définition de stratégie

1. Connectez-vous au portail Azure. 
1. Recherchez « Stratégie » dans la barre de recherche et sélectionnez **Stratégie**.

    ![Capture d’écran montrant la barre de recherche.](../media/policy-img1.png)

1. Dans la fenêtre Stratégie, sélectionnez **Définitions**.

    ![Capture d’écran mettant en évidence l’option Définitions.](../media/policy-img2.png)

1. Dans le filtre Catégorie, désélectionnez **Sélectionner tout** et sélectionnez **Key Vault**. 

    ![Capture d’écran montrant le filtre Catégorie et la catégorie Key Vault sélectionnée.](../media/policy-img3.png)

1. Vous devriez maintenant voir toutes les stratégies disponibles pour la préversion publique, pour Azure Key Vault. Veillez à lire et comprendre la section sur les conseils de stratégie ci-dessus et sélectionnez la stratégie que vous voulez attribuer à une étendue.  

    ![Capture d’écran montrant les stratégies disponibles pour la préversion publique.](../media/policy-img4.png)

### <a name="assign-a-policy-to-a-scope"></a>Attribuer une stratégie à une étendue 

1. Sélectionnez une stratégie à appliquer. Dans cet exemple, il s’agit de la stratégie **Gérer la durée de validité des certificats**. Cliquez sur le bouton Attribuer dans le coin supérieur gauche.

    ![Capture d’écran qui montre la stratégie Gérer la période de validité de certificat.](../media/policy-img5.png)
  
1. Sélectionnez l’abonnement dans lequel vous voulez appliquer la stratégie. Vous pouvez choisir de restreindre l’étendue à un seul groupe de ressources au sein d’un abonnement. Si vous souhaitez appliquer la stratégie à l’ensemble de l’abonnement et exclure certains groupes de ressources, vous pouvez également configurer une liste d’exclusions. Définissez le sélecteur d’application de stratégie sur **Activé** si vous voulez que l’effet de la stratégie (audit ou refuser) se produise ou sur **Désactivé** pour désactiver l’effet (audit ou refuser). 

    ![Capture d’écran montrant où vous pouvez choisir de restreindre l’étendue à un seul groupe de ressources au sein d’un abonnement.](../media/policy-img6.png)

1. Cliquez sur l’onglet des paramètres en haut de l’écran pour indiquer la durée de validité maximale en mois souhaitée. Sélectionnez **audit** ou **refuser** pour l’effet de la stratégie selon les conseils fournis dans les sections précédentes. Sélectionnez ensuite le bouton Vérifier + Créer. 

    ![Capture d’écran montrant l’onglet Paramètres où vous pouvez spécifier la durée de validité maximale en mois souhaitée.](../media/policy-img7.png)

### <a name="view-compliance-results"></a>Afficher les résultats de conformité

1. Revenez au panneau Stratégie et sélectionnez l’onglet Conformité. Cliquez sur l’attribution de stratégie dont vous voulez afficher les résultats de conformité.

    ![Capture d’écran montrant l’onglet Conformité où vous pouvez sélectionner l’attribution de stratégie pour laquelle vous souhaitez afficher les résultats de compatibilité.](../media/policy-img8.png)

1. Sur cette page, vous pouvez filtrer les résultats par coffres conformes et non conformes. Vous pouvez consulter ici une liste des coffres de clés non conformes dans l’étendue de l’attribution de stratégie. Un coffre est jugé non conforme si un ou plusieurs de ses composants (certificats) sont non conformes. Vous pouvez sélectionner un coffre particulier pour afficher les composants (certificats) non conformes. 


    ![Capture d’écran montrant la liste des coffres de clés non conformes dans l’étendue de l’attribution de stratégie.](../media/policy-img9.png)

1. Afficher le nom des composants non conformes au sein d’un coffre


    ![Capture d’écran montrant où vous pouvez afficher le nom des composants non conformes au sein d’un coffre.](../media/policy-img10.png)

1. Si vous devez vérifier si les utilisateurs se voient refuser la possibilité de créer des ressources dans le coffre de clés, vous pouvez cliquer sur l’onglet **Événements des composants (préversion)** pour afficher une synthèse des opérations de refus de certificat indiquant le demandeur et le timestamp des requêtes. 


    ![Vue d'ensemble du fonctionnement d'Azure Key Vault](../media/policy-img11.png)

## <a name="feature-limitations"></a>Limitations des fonctionnalités

Après avoir attribué une stratégie avec un effet « refuser », la prise d’effet du refus de créer des ressources non conformes peut prendre entre 30 minutes (en moyenne) et 1 heure (pire des cas). Une fois l’évaluation de stratégie des composants existants d’un coffre effectuée, l’affichage des résultats de conformité dans l’interface utilisateur du portail peut prendre entre 1 heure (en moyenne) et 2 heures (pire des cas). Si les résultats de conformité présentent le statut « Non démarré », cela peut être dû aux raisons suivantes :
- L’évaluation des stratégies n’est pas encore terminée. La latence de l’évaluation initiale peut atteindre 2 heures dans le pire des cas. 
- Il n’y a pas de coffres de clés dans l’étendue de l’attribution de stratégie.
- Il n’y a pas de coffres de clés avec des certificats dans l’étendue de l’attribution de stratégie.

> [!NOTE]
> Les [modes de fournisseur de ressources](../../governance/policy/concepts/definition-structure.md#resource-provider-modes) d’Azure Policy, tels que ceux destinées à la solution Azure Key Vault, fournissent des informations relatives à la conformité dans la page [Conformité des composants](../../governance/policy/how-to/get-compliance-data.md#component-compliance).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur le [service Azure Policy](../../governance/policy/overview.md)
- Consultez des exemples Key Vault : [Définitions de stratégies prédéfinies Key Vault](../../governance/policy/samples/built-in-policies.md#key-vault)
