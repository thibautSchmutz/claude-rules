---
paths:
  - "**/*.tsx"
  - "**/*.jsx"
---

# Règles React

## Structure des composants

- Les composants vivent dans le dossier `ui/` de leur module feature, ou dans `shared/ui/` s'ils font partie du design system.
- Un fichier = un composant exporté. Les sous-composants internes (non exportés) sont tolérés s'ils sont courts.
- Les composants sont des **arrow functions** avec des named exports.
- L'ordre dans un fichier de composant est : (1) imports, (2) types/props, (3) composant, (4) sous-composants internes si nécessaire.

```tsx
// ✅ Correct
type UserCardProps = {
	name: string;
	email: string;
};

export const UserCard = ({ name, email }: UserCardProps) => {
	return (
		<div>
			<p>{name}</p>
			<p>{email}</p>
		</div>
	);
};
```

## Séparation des responsabilités

- **`ui/`** : composants purs. Ils reçoivent des données et des callbacks via les props. Aucune logique métier, aucun appel à `data/` directement.
- **`business/`** : hooks qui orchestrent la logique métier. Ils consomment les hooks façade de `data/` et exposent des données dérivées et des actions à `ui/`.
- Un composant dans `ui/` consomme un hook de `business/`, jamais un hook de `data/` directement.

## Hooks

- Les hooks de logique métier sont dans `business/` au sein du module feature.
- Les hooks façade d'accès aux données sont dans `data/` (voir les règles de state management).
- Les hooks utilitaires partagés sont dans `shared/hooks/`.
- Un hook custom ne doit faire qu'une seule chose. Si un hook devient trop complexe (> 50 lignes), le découper.
- Ne jamais appeler un hook conditionnellement.

## Props

- Destructurer les props dans la signature de la fonction.
- Éviter le prop drilling au-delà de 2 niveaux. Utiliser un store Zustand local avec Context API ou de la composition de composants à la place.
- Les children typés utilisent `React.ReactNode`.
- Les event handlers en props sont préfixés par `on` : `onSubmit`, `onClick`, `onChange`.

## Patterns privilégiés

- **Composition plutôt que configuration** : préférer les composants composables (pattern compound components) plutôt qu'un gros composant avec beaucoup de props conditionnelles.
- **Render props et children functions** : éviter. Préférer les hooks custom.
- **Forwarded refs** : utiliser `React.forwardRef` uniquement pour les composants UI primitifs (boutons, inputs).

## Performance

- Utiliser `React.memo()` uniquement quand un problème de performance est mesuré, pas de manière préventive.
- `useMemo` et `useCallback` sont utilisés pour stabiliser les références passées en props à des composants mémoïsés, ou pour des calculs coûteux. Ne pas en abuser.

## Clés dans les listes

- Ne jamais utiliser l'index comme clé dans un `map()`. Utiliser un identifiant stable et unique provenant des données.
