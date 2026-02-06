---
paths:
  - "**/*.ts"
  - "**/*.tsx"
---

# Règles de gestion d'état

## Hiérarchie des solutions

Choisir la solution de state management la plus simple qui convient au besoin. Par ordre de préférence :

1. **État local du composant** (`useState`) : pour l'état qui ne concerne qu'un seul composant (toggle, input contrôlé, etc.).
2. **État de la route** (TanStack Router, search params) : pour l'état qui doit être reflété dans l'URL (filtres, pagination, onglet actif, etc.).
3. **Cache serveur** (TanStack Query) : pour toute donnée provenant de l'API ou de Supabase. Ce n'est pas du state management, c'est du cache.
4. **Store local isolé** (Zustand + Context API) : pour de l'état partagé entre plusieurs composants dans un sous-arbre limité. Le Context fournit la référence au store, seuls les composants dans ce sous-arbre y ont accès.
5. **Store global** (Zustand) : pour de l'état véritablement global (thème, utilisateur connecté, préférences). À utiliser en dernier recours.

## Architecture data/ et hooks façade

Toute donnée (API, cache, stores, persistence locale) est encapsulée dans le dossier `data/` de chaque module feature :

```
data/
├── api/          # Appels Supabase bruts
├── queries/      # Hooks TanStack Query (useQuery, useMutation)
├── stores/       # Stores Zustand (globaux ou locaux via Context)
└── index.ts      # Hooks façade
```

### Hooks façade

Le `data/index.ts` expose des **hooks façade** (un par entité) qui agrègent toutes les sources de données et actions. Les composants et hooks de `business/` consomment ces façades sans savoir quel outil est utilisé derrière.

```tsx
// data/index.ts
export const useUser = () => {
	const { data: user, isLoading } = useUserQuery();
	const preferences = useUserStore((s) => s.preferences);

	const { mutateAsync: updateUser } = useUpdateUserMutation();
	const setPreferences = useUserStore((s) => s.setPreferences);

	return {
		// Data
		user,
		isLoading,
		preferences,

		// Actions
		updateUser,
		setPreferences,
	};
};
```

Règles :
- Un hook façade par entité : `useUser`, `usePayment`, `useInvoice`.
- Le hook expose des **données** et des **actions**. Le consommateur ne sait jamais si la donnée vient de TanStack Query, Zustand, ou du localStorage.
- Si un module a plusieurs entités, créer plusieurs hooks façade, tous exportés depuis `data/index.ts`.

### Distinction data/ vs business/

- **`data/`** expose des données brutes et des actions atomiques (fetch, update, set).
- **`business/`** compose et dérive : états calculés (`isEligible`, `hasCompletedOnboarding`), orchestration de plusieurs actions, transformations.
- Règle simple : si le hook ne fait que retourner ce que l'API/store donne, c'est `data/`. Dès qu'il y a une décision, un calcul, ou une composition de sources, c'est `business/`.

## TanStack Query

- Chaque appel API est encapsulé dans un hook custom dans `data/queries/`.
- Les query keys suivent une convention hiérarchique : `["users"]`, `["users", userId]`, `["users", { filter }]`.
- Regrouper les query keys dans un objet centralisé au sein du module feature pour éviter les doublons.
- Toujours configurer `staleTime` de manière explicite plutôt que de se reposer sur le défaut.
- Les mutations utilisent `onSuccess` pour invalider les queries concernées via `queryClient.invalidateQueries()`.

## Zustand

- Les stores vivent dans `data/stores/` du module feature.
- Un store = un fichier, nommé `[nom].store.ts`.
- Ne pas mettre de logique async dans les stores. Les appels API passent par TanStack Query, pas par Zustand.
- Garder les stores plats (pas de nesting profond).
- Exposer des **actions nommées** plutôt que des setters génériques : `increaseQuantity()` plutôt que `setQuantity(quantity + 1)`.

## Store local avec Context

- Pattern : le Context Provider crée une instance de store Zustand à la volée et la fournit via `React.createContext`.
- Les composants consommateurs utilisent un hook custom qui lit le contexte.
- Ce pattern est adapté pour des formulaires multi-étapes, des modales complexes, ou des sections de page isolées.
- Le Provider, le hook d'accès, et le store factory sont dans le même fichier dans `data/stores/`.
