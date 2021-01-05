---
title: Aide sur la limitation de requêtes Azure Key Vault
description: La limitation Key Vault permet de réduire le nombre d’appels simultanés afin d’empêcher toute surexploitation des ressources.
services: key-vault
author: msmbaldwin
manager: rkarlin
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 12/02/2019
ms.author: mbaldwin
ms.openlocfilehash: 5b60f290f6d3ca184e25edd2984ad5b2d1ff2bdf
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93289681"
---
# <a name="azure-key-vault-throttling-guidance"></a>Aide sur la limitation de requêtes Azure Key Vault

La limitation de requêtes est le processus permettant de réduire le nombre d’appels simultanés au service Azure afin d’empêcher toute surexploitation des ressources. Azure Key Vault (AKV) est conçu pour gérer un volume élevé de requêtes. En cas d’affluence, la limitation des requêtes du client permet de maintenir des performances et une fiabilité optimales pour le service AKV.

Les limites varient selon les scénarios. Par exemple, si vous réalisez un volume important d’écritures, la possibilité de limitation est plus élevée que si vous effectuez uniquement des lectures.

## <a name="how-does-key-vault-handle-its-limits"></a>Comment Key Vault gère-t-il ses limites ?

Les limites de service dans Key Vault empêchent l’usage abusif des ressources et garantissent la qualité du service pour tous les clients de Key Vault. En cas de dépassement d’un seuil de service, Key Vault limite toutes les autres requêtes de ce client sur une période donnée et retourne le code d’état HTTP 429 (Trop de requêtes), indiquant l’échec de la requête. Les requêtes qui ont échoué et retourné une erreur 429 sont comptabilisées dans les limites appliquées par Key Vault. 

Key Vault a été conçu à l’origine pour stocker et récupérer vos secrets au moment du déploiement.  Le monde a évolué et Key Vault est maintenant utilisé au moment de l’exécution pour stocker et récupérer les secrets ; par ailleurs, les applications et les services cherchent souvent à utiliser Key Vault comme une base de données.  Les limites actuelles ne sont pas adaptées aux débits élevés.

Key Vault a initialement été créé avec les limites décrites dans [Limites du service Azure Key Vault](service-limits.md).  Pour optimiser votre débit Key Vault, voici quelques conseils et bonnes pratiques :
1. Assurez-vous que des limitations sont mises en place.  Le client doit respecter les stratégies de backoff exponentiel pour les erreurs 429. Veillez à procéder à de nouvelles tentatives conformément aux instructions ci-dessous.
1. Répartissez le trafic de votre Key Vault entre plusieurs coffres et différentes régions.   Utilisez un coffre distinct pour chaque domaine de sécurité/disponibilité.   Par exemple, si vous avez cinq applications, chacune dans deux régions, nous vous conseillons d’utiliser dix coffres contenant chacun les secrets propres à l’application et à la région.  La limite d’abonnement pour tous les types de transactions est fixée à cinq fois la limite d’un coffre de clés. Par exemple, le nombre de transactions autres que HSM par abonnement est limité à 5 000 en 10 secondes. Envisagez de mettre en cache le secret dans votre service ou votre application afin de réduire également le RPS directement dans le coffre de clés et/ou de gérer le trafic en rafale.  Vous pouvez également répartir le trafic entre différentes régions pour réduire la latence et utiliser un autre abonnement/coffre.  N’envoyez pas plus de transactions que la limite d’abonnement au service Key Vault dans une seule région Azure.
1. Mettez en cache les secrets que vous récupérez d’Azure Key Vault et réutilisez ces secrets du cache mémoire autant que possible.  Récupérez-les d’Azure Key Vault uniquement si la copie en cache n’est plus disponible (par exemple, si elle a été changée dans la source). 
1. Utilisez Key Vault uniquement pour vos propres secrets de services.   Si vous stockez les secrets de vos clients (surtout dans les scénarios de stockage de clés à haut débit), placez les clés dans un compte de base de données ou de stockage avec chiffrement et stockez seulement la clé principale dans Azure Key Vault.
1. Les opérations de chiffrement, d’inclusion dans un wrapper et de vérification de clé publique peuvent être effectuées sans accès à Key Vault, ce qui non seulement réduit le risque de limitation, mais améliore également la fiabilité (à condition d’avoir bien mis en cache les éléments de clé publique).
1. Si vous utilisez Key Vault pour stocker les informations d’identification d’un service, assurez-vous que ce service prend en charge Azure AD Authentication pour s’authentifier directement. Cela réduit la charge sur Key Vault, améliore la fiabilité et simplifie votre code puisque Key Vault peut maintenant utiliser le jeton Azure AD.  De nombreux services ont été mis à jour pour utiliser l’authentification Azure AD.  Consultez la liste actualisée des [services qui prennent en charge les identités managées pour les ressources Azure](../../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-managed-identities-for-azure-resources).
1. Essayez d’échelonner votre charge/déploiement sur une période plus longue pour rester sous les limites RPS actuelles.
1. Si votre application comprend plusieurs nœuds devant lire le ou les mêmes secrets, utilisez un modèle de distribution ramifiée, où une seule entité récupère un secret de Key Vault et le distribue ensuite vers tous les nœuds.   Mettez en cache les secrets récupérés uniquement en mémoire.
Si les conseils ou bonnes pratiques ci-dessus ne vous permettent pas de résoudre votre problème, renseignez ce tableau, puis contactez-nous pour déterminer si une capacité supplémentaire peut être ajoutée (l’exemple ci-dessous est donné à titre indicatif uniquement).

| Nom du coffre | Région du coffre | Type d’objet (secret, clé ou certificat) | Opération(s)* | Type de clé | Courbe ou longueur de clé | Clé HSM ?| État stable de RPS requis | Pic de RPS requis |
|--|--|--|--|--|--|--|--|--|
| https://mykeyvault.vault.azure.net/ | | Clé | Sign (Signer) | EC | P-256 | Non | 200 | 1 000 |

\* Pour obtenir une liste complète des valeurs possibles, consultez [Opérations Azure Key Vault](/rest/api/keyvault/key-operations).

Si une capacité supplémentaire est approuvée, notez les conséquences suivantes d’une augmentation de la capacité :
1. Modifications du modèle de cohérence des données. Une fois qu’un coffre est autorisé avec une capacité de débit supplémentaire, la cohérence des données du service Key Vault garantit l’application des modifications (ce qui est nécessaire pour traiter un volume RPS plus élevé qui ne peut pas l’être par le service Stockage Azure sous-jacent).  En résumé :
  1. **Sans autorisation**  : le service Key Vault reflètera les résultats d’une opération d’écriture (par exemple, SecretSet, CreateKey) immédiatement dans les appels suivants (par exemple, SecretGet, KeySign).
  1. **Avec autorisation**  : le service Key Vault reflètera les résultats d’une opération d’écriture (par exemple, SecretSet, CreateKey) dans un délai de 60 secondes dans les appels suivants (par exemple, SecretGet, KeySign).
1. Le code client doit respecter la stratégie de backoff entre les nouvelles tentatives (code de réponse 429). Le code client qui appelle le service Key Vault ne doit pas réessayer les requêtes de Key Vault immédiatement après avoir reçu un code de réponse 429.  La présente aide sur la limitation de requêtes Azure Key Vault vous conseille d’appliquer un backoff exponentiel pour les codes de réponse HTTP 429 reçus.

Si vous avez un scénario valide justifiant une limitation supérieure, contactez-nous.

## <a name="how-to-throttle-your-app-in-response-to-service-limits"></a>Guide pratique pour limiter une application en réponse à des limites de service

Les **bonnes pratiques** suivantes doivent être implémentées lorsque votre service est limité :
- Réduisez le nombre d’opérations par requête.
- Réduisez la fréquence des requêtes.
- Évitez les nouvelles tentatives immédiates. 
    - Toutes les requêtes sont comptabilisées dans le cadre de vos limites d’utilisation.

Lorsque vous implémentez la gestion des erreurs de votre application, utilisez le code d’erreur HTTP 429 pour détecter si une limitation côté client est nécessaire. Si la requête échoue à nouveau avec un code d’erreur HTTP 429, cela signifie que vous rencontrez toujours une limite de service Azure. Continuez à utiliser la méthode de limitation côté client recommandée, en réessayant la requête jusqu’à ce qu’elle aboutisse.

Le code qui implémente un backoff exponentiel est montré ci-dessous. 
```
SecretClientOptions options = new SecretClientOptions()
    {
        Retry =
        {
            Delay= TimeSpan.FromSeconds(2),
            MaxDelay = TimeSpan.FromSeconds(16),
            MaxRetries = 5,
            Mode = RetryMode.Exponential
         }
    };
    var client = new SecretClient(new Uri("https://keyVaultName.vault.azure.net"), new DefaultAzureCredential(),options);
                                 
    //Retrieve Secret
    secret = client.GetSecret(secretName);
```


L'utilisation de ce code dans une application cliente C# est simple. 

### <a name="recommended-client-side-throttling-method"></a>Méthode de limitation côté client recommandée

En cas de code d’erreur HTTP 429, commencez à limiter votre client suivant une approche d’interruption exponentielle :

1. Patientez une seconde, relancez la requête ;
2. Si la limitation persiste, patientez deux secondes, relancez la requête ;
3. Si la limitation persiste, patientez quatre secondes, relancez la requête ;
4. Si la limitation persiste, patientez huit secondes, relancez la requête ;
5. Si la limitation persiste, patientez seize secondes, relancez la requête.

À ce stade, vous ne devriez pas recevoir de code de réponse HTTP 429.

## <a name="see-also"></a>Voir aussi

Pour orienter la limitation de façon plus approfondie sur Microsoft Cloud, consultez la page [Modèle de limitation](/azure/architecture/patterns/throttling).