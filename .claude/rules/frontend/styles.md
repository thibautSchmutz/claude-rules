---
paths:
  - "**/*.tsx"
  - "**/*.jsx"
  - "**/*.css"
---

# Règles de styling (Tailwind CSS)

## Principes généraux

- Utiliser **Tailwind CSS** pour tout le styling. Ne pas écrire de CSS custom sauf cas exceptionnel (animations complexes, styles impossibles via Tailwind).
- Ne jamais utiliser de styles inline (`style={{ ... }}`), sauf pour des valeurs dynamiques calculées qui ne peuvent pas être exprimées en classes Tailwind.
- Ne pas utiliser de classes CSS globales. Si un style global est nécessaire, le définir dans le fichier `globals.css` avec les directives `@layer`.

## Organisation des classes

- Quand la liste de classes Tailwind devient longue (> 5 classes), utiliser `clsx` ou `cn` (utility inspirée de shadcn/ui) pour la lisibilité.
- Regrouper les classes par catégorie logique : layout, spacing, typography, colors, states.

```tsx
// ✅ Lisible avec cn()
<div
	className={cn(
		"flex items-center gap-4 p-4",
		"rounded-lg border border-gray-200",
		"text-sm text-gray-700",
		isActive && "border-blue-500 bg-blue-50"
	)}
/>
```

## Design system

- Les composants UI de base (Button, Input, Card, etc.) sont construits en **custom**, inspirés de shadcn/ui en termes de patterns. Ils vivent dans `shared/ui/`.
- Utiliser des **variants** pour les composants UI via des props explicites (`variant="primary"`, `size="sm"`), pas en combinant des classes Tailwind à l'extérieur du composant.
- Les couleurs, espacements et typographies suivent le thème défini dans `tailwind.config.ts`. Ne pas utiliser de valeurs arbitraires (`text-[13px]`) sauf exception justifiée.

## Responsive design

- Approche **mobile-first** : les classes de base s'appliquent au mobile, les breakpoints (`sm:`, `md:`, `lg:`) ajoutent les adaptations pour les écrans plus grands.
- Tester systématiquement sur les breakpoints `sm` (640px), `md` (768px), et `lg` (1024px) au minimum.

## Dark mode

- Si le projet supporte le dark mode, utiliser les classes `dark:` de Tailwind.
- Les couleurs doivent toujours avoir une variante dark définie. Ne jamais hardcoder une couleur sans penser à son équivalent dark.
