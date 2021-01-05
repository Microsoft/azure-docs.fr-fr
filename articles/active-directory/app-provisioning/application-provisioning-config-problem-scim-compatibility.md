---
title: Problèmes connus avec la conformité du protocole SCIM (System for Cross-domain Identity Management) 2.0 – Azure AD
description: Comment résoudre les problèmes courants de compatibilité des protocoles rencontrés lors de l’ajout d’une application ne figurant pas dans la galerie qui prend en charge SCIM 2.0 dans Azure AD
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: reference
ms.date: 08/05/2020
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: 7f400d6959a40361ea3beff8bd21c2fa9ef2996a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90052628"
---
# <a name="known-issues-and-resolutions-with-scim-20-protocol-compliance-of-the-azure-ad-user-provisioning-service"></a>Problèmes et solutions connus relatifs à la conformité au protocole SCIM 2.0 du service de provisionnement des utilisateurs Azure AD

Azure Active Directory (Azure AD) peut affecter automatiquement des utilisateurs et des groupes à une application ou un système avec en façade un service Web avec l’interface définie dans la [Spécification du protocole SCIM (System for Cross-Domain Identity Management) 2.0](https://tools.ietf.org/html/draft-ietf-scim-api-19). 

La prise en charge Azure AD du protocole SCIM 2.0 est décrite dans l’article [Utilisation du protocole SCIM (System for Cross-Domain Identity Management) pour configurer automatiquement des utilisateurs et groupes d’Azure Active Directory dans des applications](use-scim-to-provision-users-and-groups.md), qui dresse la liste des parties spécifiques du protocole qu’il implémente afin d’approvisionner automatiquement les utilisateurs et groupes Azure AD aux applications qui prennent en charge le protocole SCIM 2.0.

Cet article décrit les problèmes actuels et passés concernant le respect du service d’approvisionnement d’utilisateurs Azure AD au protocole SCIM 2.0, et comment les résoudre.

## <a name="understanding-the-provisioning-job"></a>Compréhension des étapes d’approvisionnement
Le service d’approvisionnement utilise le concept de travail pour opérer sur une application. Le jobID se trouve dans la [barre de progression](application-provisioning-when-will-provisioning-finish-specific-user.md#view-the-provisioning-progress-bar). Toutes les nouvelles applications d’approvisionnement sont créées avec un jobID commençant par « scim ». Le travail scim représente l’état actuel du service. Les travaux plus anciens ont l’ID « customappsso ». Ce travail représente l’état du service en 2018. 

Si vous utilisez une application de la galerie, le travail contient généralement le nom de l’application (par exemple, zoom snowFrake, dataBricks, etc.). Vous pouvez ignorer cette documentation lors de l’utilisation d’une application de la galerie. Cela s’applique principalement aux applications qui ne sont pas de la galerie avec jobID SCIM ou customAppSSO.

## <a name="scim-20-compliance-issues-and-status"></a>Problèmes de conformité et état SCIM 2.0
Dans le tableau ci-dessous, tout élément marqué comme corrigé signifie que le comportement correct peut être trouvé sur le travail SCIM. Nous avons travaillé pour garantir la compatibilité descendante des modifications que nous avons apportées. Toutefois, nous vous déconseillons d’implémenter l’ancien comportement. Nous vous recommandons d’utiliser le nouveau comportement pour les nouvelles implémentations et la mise à jour des implémentations existantes.

> [!NOTE]
> Pour les modifications apportées en 2018, vous pouvez revenir au comportement customappsso. Pour les modifications apportées depuis 2018, vous pouvez utiliser les URL pour revenir à l’ancien comportement. Nous avons travaillé pour garantir la compatibilité descendante des modifications que nous avons apportées en vous permettant de revenir à l’ancien jobID ou en utilisant un indicateur. Toutefois, comme mentionné précédemment, nous vous déconseillons d’implémenter l’ancien comportement. Nous vous recommandons d’utiliser le nouveau comportement pour les nouvelles implémentations et la mise à jour des implémentations existantes.

| **Problèmes de conformité à SCIM 2.0** |  **Résolution ?** | **Date du correctif**  |  **Compatibilité descendante** |
|---|---|---|
| Azure AD nécessite que « /scim » soit à la racine de l’URL du point de terminaison SCIM de l’application  | Oui  |  18 décembre 2018 | Passage à la version antérieure customappSSO |
| Les attributs d’extension utilisent la notation point « . » avant les noms d’attribut, et non pas la notation deux-points « : » |  Oui  | 18 décembre 2018  | Passage à la version antérieure customappSSO |
| Les demandes de correctif pour les attributs multivaleurs contiennent une syntaxe de filtre de chemin d’accès non valide | Oui  |  18 décembre 2018  | passage à la version antérieure customappSSO |
| Les demandes de création de groupe contiennent une URI de schéma non valide | Oui  |  18 décembre 2018  |  passage à la version antérieure customappSSO |
| Mise à jour du comportement des PATCH pour garantir la conformité (par exemple, actif en tant que valeur booléenne et suppressions appropriées d’appartenance à un groupe) | Non | TBD| utiliser un indicateur d’aperçu |

## <a name="flags-to-alter-the-scim-behavior"></a>Indicateurs permettant de modifier le comportement de SCIM
Utilisez les indicateurs ci-dessous dans l’URL du locataire de votre application afin de modifier le comportement de client SCIM par défaut.

:::image type="content" source="media/application-provisioning-config-problem-scim-compatibility/scim-flags.jpg" alt-text="Indicateurs SCIM pour comportement ultérieur.&quot;:::

* Utilisez l’URL suivante pour mettre à jour le comportement des PATCH et garantir la conformité SCIM (par exemple, actif en tant que valeur booléenne et suppressions appropriées d’appartenance à un groupe). Ce comportement est actuellement disponible uniquement lorsque vous utilisez l’indicateur, mais il deviendra le comportement par défaut au cours des prochains mois. Notez que cet indicateur de préversion ne fonctionne actuellement pas avec l’approvisionnement à la demande. 
  * **URL (conforme à SCIM) :** AzureAdScimPatch062020
  * **Références RFC SCIM :** 
    * https://tools.ietf.org/html/rfc7644#section-3.5.2
  * **Comportement :**
  ```json
   PATCH https://[...]/Groups/ac56b4e5-e079-46d0-810e-85ddbd223b09
   {
    &quot;schemas&quot;: [
        &quot;urn:ietf:params:scim:api:messages:2.0:PatchOp&quot;
    ],
    &quot;Operations&quot;: [
        {
            &quot;op&quot;: &quot;remove&quot;,
            &quot;path&quot;: &quot;members[value eq \&quot;16b083c0-f1e8-4544-b6ee-27a28dc98761\&quot;]&quot;
        }
    ]
   }

    PATCH https://[...]/Groups/ac56b4e5-e079-46d0-810e-85ddbd223b09
    {
    &quot;schemas&quot;: [
        &quot;urn:ietf:params:scim:api:messages:2.0:PatchOp&quot;
    ],
    &quot;Operations&quot;: [
        {
            &quot;op&quot;: &quot;add&quot;,
            &quot;path&quot;: &quot;members&quot;,
            &quot;value&quot;: [
                {
                    &quot;value&quot;: &quot;10263a6910a84ef9a581dd9b8dcc0eae&quot;
                }
            ]
        }
    ]
    } 

    PATCH https://[...]/Users/ac56b4e5-e079-46d0-810e-85ddbd223b09
    {
    &quot;schemas&quot;: [
        &quot;urn:ietf:params:scim:api:messages:2.0:PatchOp&quot;
    ],
    &quot;Operations&quot;: [
        {
            &quot;op&quot;: &quot;replace&quot;,
            &quot;path&quot;: &quot;emails[type eq \&quot;work\&quot;].value&quot;,
            &quot;value&quot;: &quot;someone@contoso.com&quot;
        },
        {
            &quot;op&quot;: &quot;replace&quot;,
            &quot;path&quot;: &quot;emails[type eq \&quot;work\&quot;].primary&quot;,
            &quot;value&quot;: true
        },
        {
            &quot;op&quot;: &quot;replace&quot;,
            &quot;value&quot;: {
                &quot;active&quot;: false,
                &quot;userName&quot;: &quot;someone&quot;
            }
        }
    ]
    }

    PATCH https://[...]/Users/ac56b4e5-e079-46d0-810e-85ddbd223b09
    {
    &quot;schemas&quot;: [
        &quot;urn:ietf:params:scim:api:messages:2.0:PatchOp&quot;
    ],
    &quot;Operations&quot;: [
        {
            &quot;op&quot;: &quot;replace&quot;,
            &quot;path&quot;: &quot;active&quot;,
            &quot;value&quot;: false
        }
    ]
    }

    PATCH https://[...]/Users/ac56b4e5-e079-46d0-810e-85ddbd223b09
    {
    &quot;schemas&quot;: [
        &quot;urn:ietf:params:scim:api:messages:2.0:PatchOp&quot;
    ],
    &quot;Operations&quot;: [
        {
            &quot;op&quot;: &quot;add&quot;,
            &quot;path&quot;: &quot;urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department&quot;,
            &quot;value&quot;: &quot;Tech Infrastructure"
        }
    ]
    }
   
  ```

  * **URL de rétrogradation :** Une fois que le nouveau comportement conforme à SCIM devient celui par défaut sur l’application ne figurant pas dans la galerie, vous pouvez utiliser l’URL suivante pour revenir à l’ancien comportement conforme non SCIM : AzureAdScimPatch2017
  


## <a name="upgrading-from-the-older-customappsso-job-to-the-scim-job"></a>Mise à niveau de l’ancien travail customappsso en travail SCIM
Les étapes ci-dessous ont pour effet de supprimer votre travail customappsso existant et créer un nouveau travail scim. 
 
1. Connectez-vous au portail Azure à l’adresse https://portal.azure.com.
2. Dans la section **Azure Active Directory > Applications d’entreprise** du portail Azure, localisez et sélectionnez votre application SCIM existante.
3. Dans la section **Propriétés** de votre application SCIM existante, copiez l’**ID objet**.
4. Dans une nouvelle fenêtre de navigateur web, accédez à https://developer.microsoft.com/graph/graph-explorer et connectez-vous en tant qu’administrateur pour le locataire Azure AD où votre application est ajoutée.
5. Dans l’Explorateur graphique, exécutez la commande suivante pour rechercher l’ID de votre travail d’approvisionnement. Remplacez « [object-id] » par l’ID du principal de service (ID d’objet) copié à partir de la troisième étape.
 
   `GET https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs` 

   ![Obtenir les travaux](media/application-provisioning-config-problem-scim-compatibility/get-jobs.PNG "Obtenir les travaux") 


6. Dans les résultats, copiez la chaîne « ID » complète qui commence par « customappsso » ou « scim ».
7. Exécutez la commande ci-dessous pour récupérer la configuration du mappage des attributs, afin de pouvoir effectuer une sauvegarde. Utilisez le même [object-id] qu’auparavant et remplacez [job-id] par l’ID du travail approvisionnement copié à partir de la dernière étape.
 
   `GET https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs/[job-id]/schema`
 
   ![Obtenir le schéma](media/application-provisioning-config-problem-scim-compatibility/get-schema.PNG "Obtenir le schéma") 

8. Copiez la sortie JSON de la dernière étape et enregistrez-la dans un fichier texte. Le code JSON contient les mappages d’attributs personnalisés que vous avez ajoutés à votre ancienne application et doit contenir quelques milliers de lignes.
9. Exécutez la commande ci-dessous pour supprimer le travail d’approvisionnement :
 
   `DELETE https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs/[job-id]`

10. Exécutez la commande ci-dessous pour créer un nouveau travail d’approvisionnement avec les derniers correctifs de service.

 `POST https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs`
 `{   templateId: "scim"   }`
   
11. Dans les résultats de la dernière étape, copiez la chaîne « ID » complète qui commence par « scim ». Si vous le souhaitez, réappliquez vos anciens mappages d’attributs en exécutant la commande ci-dessous, en remplaçant [new-job-id] par le nouvel ID de travail que vous avez copié, et en entrant la sortie JSON de l’étape n° 7 en tant que corps de la demande.

 `POST https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs/[new-job-id]/schema`
 `{   <your-schema-json-here>   }`

12. Revenez à la première fenêtre du navigateur Web et sélectionnez l’onglet **Approvisionnement** de votre application.
13. Vérifiez votre configuration, puis démarrez la tâche d’approvisionnement. 

## <a name="downgrading-from-the-scim-job-to-the-customappsso-job-not-recommended"></a>passage fu travail SCIM à la tâche version antérieure du travail customappsso (non recommandé)
 Nous vous autorisons à revenir à l’ancien comportement mais ne le recommandons pas, car le travail customappsso ne bénéficie pas de certaines mises à jour que nous faisons, et risque ne pas être pris en charge indéfiniment. 

1. Connectez-vous au portail Azure à l’adresse https://portal.azure.com.
2. Dans la section **Azure Active Directory > Applications d’entreprise > Créer une application** du portail Azure, créez une application **ne figurant pas dans la galerie**.
3. Dans la section **Propriétés** de votre nouvelle application personnalisée, copiez l’**ID objet**.
4. Dans une nouvelle fenêtre de navigateur web, accédez à https://developer.microsoft.com/graph/graph-explorer et connectez-vous en tant qu’administrateur pour le locataire Azure AD où votre application est ajoutée.
5. Dans l’Explorateur graphique, exécutez la commande suivante pour initialiser la configuration d’approvisionnement de votre application.
   Remplacez « [object-id] » par l’ID du principal de service (ID d’objet) copié à partir de la troisième étape.

   `POST https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs`
   `{   templateId: "customappsso"   }`
 
6. Revenez à la première fenêtre du navigateur Web et sélectionnez l’onglet **Approvisionnement** de votre application.
7. Complétez la configuration d’approvisionnement utilisateur comme vous le feriez normalement.


## <a name="next-steps"></a>Étapes suivantes
[En savoir plus sur l’approvisionnement et l’annulation de l’approvisionnement pour les applications SaaS](user-provisioning.md)
