---
title: Filtres de sécurité pour filtrer les résultats avec Active Directory
titleSuffix: Azure Cognitive Search
description: Les privilèges de sécurité au niveau du document pour Azure Recherche cognitive les résultats de la recherche, à l’aide de filtres de sécurité et d’identités Azure Active Directory (AAD).
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/04/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 87337cf22bdb388c5873a2811bb9913c3e7f4d4e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994959"
---
# <a name="security-filters-for-trimming-azure-cognitive-search-results-using-active-directory-identities"></a>Utilisation de filtres de sécurité pour filtrer les résultats de Recherche cognitive Azure à l’aide d’identités Active Directory

Cet article explique comment utiliser les identités de sécurité Azure Active Directory (AAD) avec des filtres dans la Recherche cognitive Azure pour tronquer les résultats de recherche en fonction de l’appartenance à des groupes d’utilisateurs.

Cet article décrit les tâches suivantes :
> [!div class="checklist"]
> - Créer des utilisateurs et des groupes AAD
> - Associer l’utilisateur au groupe que vous avez créé
> - Mettre en cache les nouveaux groupes
> - Indexer les documents avec les groupes associés
> - Émettre une demande de recherche avec un filtre d’identificateurs de groupe
> 
> [!NOTE]
> Dans cet article, les exemples d’extraits de code sont écrits en C#. L’intégralité du code source est disponible [sur GitHub](https://github.com/Azure-Samples/search-dotnet-getting-started). 

## <a name="prerequisites"></a>Prérequis

Votre index dans Recherche cognitive Azure doit avoir un [champ de sécurité](search-security-trimming-for-azure-search.md) pour stocker la liste des identités de groupe disposant d’un accès en lecture pour le document. Ce cas d’usage implique une correspondance exacte entre un élément sécurisable (par exemple l’application d’un établissement scolaire) et un champ de sécurité spécifiant qui a accès à cet élément (personnel en charge des admissions).

Vous devez disposer des autorisations d’administrateur AAD requises dans cette procédure pour créer des utilisateurs, des groupes et des associations dans AAD.

Votre application doit également être inscrite auprès d’AAD, comme décrit dans la procédure suivante.

### <a name="register-your-application-with-aad"></a>Inscrire votre application auprès d’AAD

Cette étape intègre votre application avec AAD pour pouvoir accepter les connexions des comptes d’utilisateur et de groupe. Si vous n’êtes pas administrateur AAD dans votre organisation, vous devrez peut-être [créer un nouveau locataire](../active-directory/develop/quickstart-create-new-tenant.md) pour effectuer les étapes suivantes.

1. Accédez à [**Portail d’inscription des applications**](https://apps.dev.microsoft.com) >  **Application convergée** > **Ajouter une application**.
2. Entrez un nom pour votre application, puis cliquez sur **Créer**. 
3. Sélectionnez votre application nouvellement inscrite sur la page Mes Applications.
4. Sur la page d’inscription des applications > **Plateformes** > **Ajouter une plateforme** > **API Web**.
5. Toujours sur la page d’inscription des applications, accédez à > **Autorisations pour Microsoft Graph** > **Ajouter**.
6. Sous Sélectionner les autorisations, ajoutez les autorisations déléguées suivantes, puis cliquez sur **OK** :

   + **Directory.ReadWrite.All**
   + **Group.ReadWrite.All**
   + **User.ReadWrite.All**

Microsoft Graph fournit une API qui permet l’accès par programme à AAD via une API REST. L’exemple de code pour cette procédure utilise les autorisations pour appeler l’API Microsoft Graph pour la création de groupes, d’utilisateurs et d’associations. Les API sont également utilisées pour mettre en cache les identificateurs de groupe afin d’accélérer le filtrage.

## <a name="create-users-and-groups"></a>Créer des utilisateurs et des groupes

Si vous ajoutez une fonctionnalité de recherche à une application établie, il est possible que vous disposiez déjà d’identificateurs d’utilisateur et de groupe dans AAD. Dans ce cas, vous pouvez ignorer les trois étapes suivantes. 

Toutefois, si vous n’avez pas d’utilisateurs existants, vous pouvez utiliser les API Microsoft Graph pour créer des entités de sécurité. Les extraits de code suivants montrent comment générer des identificateurs qui deviennent des valeurs de données pour le champ de sécurité dans votre index Recherche cognitive Azure. Dans l’exemple de notre application d’admission scolaire, il s’agirait des identificateurs de sécurité pour le personnel en charge des admissions.

La gestion des groupes et des utilisateurs peut s’avérer très fluide, en particulier dans les grandes organisations. Le code qui génère les identités d’utilisateur et de groupe doit s’exécuter assez souvent pour tenir compte des modifications apportées aux groupes de l’organisation. De même, votre index Recherche cognitive Azure requiert une planification de mise à jour similaire pour refléter l’état actuel des utilisateurs et des ressources autorisés.

### <a name="step-1-create-aad-group"></a>Étape 1 : Créer un [groupe AAD](/graph/api/group-post-groups?view=graph-rest-1.0) 
```csharp
// Instantiate graph client 
GraphServiceClient graph = new GraphServiceClient(new DelegateAuthenticationProvider(...));
Group group = new Group()
{
    DisplayName = "My First Prog Group",
    SecurityEnabled = true,
    MailEnabled = false,
    MailNickname = "group1"
}; 
Group newGroup = await graph.Groups.Request().AddAsync(group);
```
   
### <a name="step-2-create-aad-user"></a>Étape 2 : Créer un [utilisateur AAD](/graph/api/user-post-users?view=graph-rest-1.0)
```csharp
User user = new User()
{
    GivenName = "First User",
    Surname = "User1",
    MailNickname = "User1",
    DisplayName = "First User",
    UserPrincipalName = "User1@FirstUser.com",
    PasswordProfile = new PasswordProfile() { Password = "********" },
    AccountEnabled = true
};
User newUser = await graph.Users.Request().AddAsync(user);
```

### <a name="step-3-associate-user-and-group"></a>Étape 3 : Associer les utilisateurs et les groupes
```csharp
await graph.Groups[newGroup.Id].Members.References.Request().AddAsync(newUser);
```

### <a name="step-4-cache-the-groups-identifiers"></a>Étape 4 : Mettre en cache les identificateurs de groupe
Si vous le souhaitez, pour réduire la latence du réseau, vous pouvez mettre en cache les associations utilisateurs-groupes. Ainsi, lorsqu’une demande de recherche est émise, les groupes sont renvoyés à partir du cache, ce qui évite un aller-retour dans AAD. Vous pouvez utiliser l'[API de Batch AAD](/graph/json-batching) pour envoyer une requête Http unique avec plusieurs utilisateurs et générer le cache.

Microsoft Graph est conçu pour gérer un volume élevé de demandes. Si un trop grand nombre de demandes sont émises, Microsoft Graph génère une erreur avec le code d’état HTTP 429. Pour plus d’informations, consultez le document [Limitation dans Microsoft Graph](/graph/throttling).

## <a name="index-document-with-their-permitted-groups"></a>Indexer les documents avec leurs groupes autorisés

Les opérations de demande dans Recherche cognitive Azure sont exécutées via un index Recherche cognitive Azure. Dans cette étape, une opération d’indexation importe les données interrogeables sous forme d’index, y compris les identificateurs utilisés comme filtres de sécurité. 

Le service Recherche cognitive Azure n’authentifie pas les identités des utilisateurs et ne fournit aucune logique pour définir le contenu qu’un utilisateur est autorisé à consulter. Le cas d’usage pour le filtrage de sécurité implique que vous fournissiez l’association entre un document sensible et l’identificateur de groupe ayant accès à ce document importé sans modification dans un index de recherche. 

Dans notre exemple hypothétique, le corps de la demande PUT sur un index Recherche cognitive Azure inclurait une épreuve ou une transcription d’un candidat, ainsi que l’identificateur de groupe ayant l’autorisation d’afficher ce contenu. 

Dans l’exemple générique utilisé dans l’exemple de code pour cette procédure, l’action d’indexation peut se présenter comme suit :

```csharp
var actions = new IndexAction<SecuredFiles>[]
              {
                  IndexAction.Upload(
                  new SecuredFiles()
                  {
                      FileId = "1",
                      Name = "secured_file_a",
                      GroupIds = new[] { groups[0] }
                  }),
              ...
             };

var batch = IndexBatch.New(actions);

_indexClient.Documents.Index(batch);  
```

## <a name="issue-a-search-request"></a>Émettre une demande de recherche

Pour des raisons de filtrage de sécurité, les valeurs du champ de sécurité dans l’index sont des valeurs statiques utilisées pour inclure ou exclure des documents dans les résultats de la recherche. Par exemple, si l’identificateur de groupe pour les admissions est « A11B22C33D44-E55F66G77-H88I99JKK », tous les documents dans un index Recherche cognitive Azure ayant cet identificateur dans le champ de sécurité sont inclus (ou exclus) dans les résultats de la recherche renvoyés au demandeur.

Pour filtrer les documents renvoyés dans les résultats de la recherche en fonction des groupes de l’utilisateur qui émet la demande, procédez comme suit.

### <a name="step-1-retrieve-users-group-identifiers"></a>Étape 1 : Récupérer les identificateurs de groupe de l’utilisateur

Si les groupes de l’utilisateur n’ont pas encore été mis en cache, ou si le cache a expiré, exécutez la demande [groupes](/graph/api/directoryobject-getmembergroups?view=graph-rest-1.0).
```csharp
private static void RefreshCacheIfRequired(string user)
{
    if (!_groupsCache.ContainsKey(user))
    {
        var groups = GetGroupIdsForUser(user).Result;
        _groupsCache[user] = groups;
    }
}

private static async Task<List<string>> GetGroupIdsForUser(string userPrincipalName)
{
    List<string> groups = new List<string>();
    var allUserGroupsRequest = graph.Users[userPrincipalName].GetMemberGroups(true).Request();

    while (allUserGroupsRequest != null) 
    {
        IDirectoryObjectGetMemberGroupsRequestBuilder allUserGroups = await allUserGroupsRequest.PostAsync();
        groups = allUserGroups.ToList();
        allUserGroupsRequest = allUserGroups.NextPageRequest;
    }
    return groups;
}
``` 

### <a name="step-2-compose-the-search-request"></a>Étape 2 : Composer la requête de recherche

En supposant que vous connaissez les groupes de l’utilisateur, vous pouvez émettre la demande de recherche avec les valeurs de filtre appropriées.

```csharp
string filter = String.Format("groupIds/any(p:search.in(p, '{0}'))", string.Join(",", groups.Select(g => g.ToString())));
SearchParameters parameters = new SearchParameters()
             {
                 Filter = filter,
                 Select = new[] { "application essays" }
             };

DocumentSearchResult<SecuredFiles> results = _indexClient.Documents.Search<SecuredFiles>("*", parameters);
```
### <a name="step-3-handle-the-results"></a>Étape 3 : Gérer les résultats

La réponse inclut une liste répertoriant uniquement les documents que l’utilisateur est autorisé à afficher. Selon la façon dont vous construisez la page de résultats, vous pouvez inclure des repères visuels pour matérialiser le jeu de résultats obtenu.

## <a name="conclusion"></a>Conclusion

Dans cette procédure pas à pas, vous avez découvert comment utiliser les connexions AAD pour filtrer des documents dans les résultats Recherche cognitive Azure et ignorer les documents ne correspondant pas au filtre défini dans la demande.

## <a name="see-also"></a>Voir aussi

+ [Contrôle d’accès basé sur l’identité à l’aide des filtres Recherche cognitive Azure](search-security-trimming-for-azure-search.md)
+ [Filtres dans la Recherche cognitive Azure](search-filters.md)
+ [Sécurité des données et contrôle d'accès dans les opérations de Recherche cognitive Azure](search-security-overview.md)