# Conventions générales

## Principes de développement

Ces principes guident toutes les décisions de conception et d'implémentation :

- **KISS (Keep It Simple, Stupid)** : la solution la plus simple qui fonctionne est toujours préférable. Éviter la complexité accidentelle, les abstractions inutiles, et le code "clever" difficile à lire.
- **YAGNI (You Aren't Gonna Need It)** : ne pas implémenter une fonctionnalité "au cas où". Coder uniquement ce qui est nécessaire maintenant. Le refactoring futur est moins coûteux que le code mort.
- **SRP (Single Responsibility Principle)** : chaque fichier, fonction, composant et module a une responsabilité unique et clairement définie. Si on ne peut pas décrire le rôle en une phrase, il faut découper.
- **DRY avec discernement** : la duplication est préférable à une mauvaise abstraction. Dupliquer du code 2 fois est acceptable. À la 3ème occurrence, extraire une abstraction. C'est la "Rule of Three".
- **Colocation** : placer le code le plus près possible de là où il est utilisé. Un schéma utilisé uniquement dans `data/` reste dans `data/`. Il remonte dans `types/` seulement quand il est partagé entre plusieurs couches.
- **Composition over Inheritance** : composer des petites fonctions, des petits hooks, et des petits composants plutôt que de créer des hiérarchies d'héritage.

## Architecture feature-based

Le code est organisé par **domaine métier** (features) plutôt que par type technique. Chaque feature est un module autonome avec ses propres couches.

### Flux de dépendances (frontend / mobile)

Les dépendances sont **unidirectionnelles**. Aucune flèche ne doit remonter :

```
ui/ → business/ → data/ → types/
                     ↘       ↗
                     utils/
```

- `ui/` consomme `business/`
- `business/` consomme `data/`
- `types/`, `utils/` et `constants/` sont consommés par toutes les couches
- **Interdit** : `data/` ne doit jamais importer depuis `business/` ou `ui/`

### Flux de dépendances (backend)

```
Route → Service → Repository → Supabase
```

- Une route n'appelle jamais un repository directement (sauf CRUD trivial sans logique métier)
- Un repository ne contient pas de logique métier
- Un service peut appeler plusieurs repositories

### Communication entre features

- Ne jamais importer directement depuis les fichiers internes d'un autre module feature. Toujours passer par le barrel export (`index.ts`).
- Si deux features ont besoin de partager du code, l'extraire dans `shared/`.

## Langue

- Les commentaires dans le code sont rédigés en français.
- Les messages de commit suivent le format Conventional Commits en français : `feat: ajouter la page de profil`, `fix: corriger le calcul du total`.
- Les noms de variables, fonctions, types et fichiers restent en anglais.
- Les descriptions de PR sont rédigées en français.

## Nommage des fichiers

- Tous les fichiers sont nommés en **kebab-case** : `user-profile.tsx`, `use-auth.ts`, `payment.schema.ts`.
- Les fichiers de test suivent le pattern `[use-case].spec.ts` (e2e) ou `[use-case].test.ts` (unitaire/intégration).
- Les fichiers de types dédiés sont nommés `[entité].schema.ts` dans leur dossier `types/`.

## Structure du code

- Un fichier ne doit **jamais dépasser 300 lignes**. Au-delà, découper en sous-modules.
- Chaque fonction doit avoir une **responsabilité unique**.
- Privilégier les **early returns** pour réduire l'imbrication.
- Éviter les fonctions avec plus de 3 paramètres. Au-delà, utiliser un objet de paramètres typé.

## Exports

- Utiliser exclusivement des **named exports**. Les default exports sont interdits sauf quand le framework l'impose (ex : configuration de route, `export default` requis par TanStack Router / Expo Router / Hono).
- Chaque module (feature) expose un fichier `index.ts` qui sert de barrel export pour son API publique.

## Gestion des erreurs

- Ne jamais laisser un `catch` vide. Toujours logger ou remonter l'erreur.
- Utiliser un try/catch centralisé au niveau des couches d'entrée (handlers de route, event handlers de composants).
- Les erreurs utilisateur (formulaires, validation) sont gérées via Zod et affichées dans l'UI.
- Les erreurs système (réseau, base de données) sont loggées et affichent un message générique à l'utilisateur.
- Côté backend, les réponses d'erreur suivent le format standardisé `{ data: null, error: { code, message }, meta? }`.

## Git

- Workflow : **feature branches + Pull Requests**.
- Les branches suivent le format : `feat/description-courte`, `fix/description-courte`, `chore/description-courte`.
- Ne jamais commiter directement sur `main`.
- Ne jamais commiter de secrets, clés API, tokens, ou fichiers `.env`.

## Documentation

- Les fonctions utilitaires complexes doivent avoir un commentaire JSDoc expliquant leur rôle, leurs paramètres, et ce qu'elles retournent.
- Les composants React n'ont pas besoin de JSDoc si leur nom et leurs props sont explicites.
- Préférer un code lisible à un code commenté : si un commentaire est nécessaire pour comprendre, c'est souvent un signe que le code doit être refactorisé.
- Les tests servent de documentation produit : l'arborescence des fichiers de test doit permettre de comprendre les fonctionnalités de l'application d'un coup d'œil.
