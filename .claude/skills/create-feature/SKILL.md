---
name: create-feature
description: Créer la structure d'un nouveau module feature côté frontend ou mobile avec tous les fichiers nécessaires. Utiliser quand on démarre un nouveau module, une nouvelle fonctionnalité, ou un nouveau domaine métier dans une application React (web) ou React Native (mobile). Ne PAS utiliser pour du code backend, des edge functions, ou de la logique purement serveur.
---

# Créer un module feature (frontend / mobile)

## Contexte

Ce skill s'applique uniquement aux applications **frontend (React / TanStack)** et **mobile (React Native / Expo)**. Pour le backend, utiliser le skill `create-backend-feature`.

## Instructions

Quand l'utilisateur demande de créer un nouveau module feature, générer la structure suivante sous `src/features/[nom-du-module]/` :

```
src/features/[nom-du-module]/
├── ui/                   # Composants React purs (rendu)
├── business/             # Hooks de logique métier (orchestration, états dérivés)
├── data/                 # Accès aux données
│   ├── api/              # Appels Supabase bruts
│   ├── queries/          # Hooks TanStack Query (useQuery, useMutation)
│   ├── stores/           # Stores Zustand (globaux ou locaux via Context)
│   └── index.ts          # Hooks façade (useUser, etc.)
├── types/                # Schémas Zod modèle + types dérivés
├── utils/                # Fonctions utilitaires pures
├── constants/            # Valeurs constantes
└── index.ts              # Barrel export — API publique du module
```

## Règles de génération

1. **Nommage** : le nom du module est en kebab-case (`user-profile`, `payment`, `notification`).
2. **index.ts** : ne réexporte que ce qui est nécessaire aux autres modules. Les fichiers internes restent privés.
3. **types/** : créer au minimum un schéma Zod pour l'entité principale du module, et en dériver le type avec `z.infer`.
4. **data/api/** : créer les fonctions d'appel Supabase bruts si le module correspond à une ressource.
5. **data/queries/** : créer les hooks TanStack Query qui encapsulent les appels API.
6. **data/stores/** : si le module a besoin d'état partagé, créer un store Zustand ici.
7. **data/index.ts** : créer le hook façade qui agrège toutes les sources de données.
8. **Ne créer que les dossiers nécessaires.** Si le module n'a pas besoin de store, ne pas créer `data/stores/`. Si pas de logique métier complexe, ne pas créer `business/`.

## Flux de dépendances

```
ui/ → business/ → data/ → types/
                     ↘       ↗
                     utils/
```

Les imports ne doivent jamais remonter ce flux.

## Exemple : hook façade

```tsx
// data/index.ts
import { useUserQuery, useUpdateUserMutation } from "./queries/user.queries";
import { useUserStore } from "./stores/user.store";

export const useUser = () => {
	// Sources de données
	const { data: user, isLoading } = useUserQuery();
	const preferences = useUserStore((s) => s.preferences);

	// Actions
	const { mutateAsync: updateUser } = useUpdateUserMutation();
	const setPreferences = useUserStore((s) => s.setPreferences);

	return {
		user,
		isLoading,
		preferences,
		updateUser,
		setPreferences,
	};
};
```

## Exemple : store local avec Context

```tsx
// data/stores/payment-form.store.ts
import { createContext, useContext } from "react";
import { createStore, useStore } from "zustand";

type PaymentFormState = {
	step: number;
	amount: number;
	nextStep: () => void;
	previousStep: () => void;
	setAmount: (amount: number) => void;
};

export const createPaymentFormStore = () =>
	createStore<PaymentFormState>((set) => ({
		step: 0,
		amount: 0,
		nextStep: () => set((state) => ({ step: state.step + 1 })),
		previousStep: () => set((state) => ({ step: Math.max(0, state.step - 1) })),
		setAmount: (amount) => set({ amount }),
	}));

const PaymentFormStoreContext = createContext<ReturnType<typeof createPaymentFormStore> | null>(null);

export const PaymentFormStoreProvider = PaymentFormStoreContext.Provider;

export const usePaymentFormStore = <T,>(selector: (state: PaymentFormState) => T): T => {
	const store = useContext(PaymentFormStoreContext);

	if (!store) {
		throw new Error("usePaymentFormStore doit être utilisé dans un PaymentFormStoreProvider");
	}

	return useStore(store, selector);
};
```

## Exemple : schéma modèle

```tsx
// types/payment.schema.ts
import { z } from "zod";

export const paymentSchema = z.object({
	id: z.string().uuid(),
	amount: z.number().positive(),
	currency: z.string().length(3),
	status: z.enum(["pending", "completed", "failed"]),
	createdAt: z.string().datetime(),
});

export type Payment = z.infer<typeof paymentSchema>;
```

## Exemple : barrel export

```tsx
// index.ts
export { paymentSchema, type Payment } from "./types/payment.schema";
export { PaymentList } from "./ui/payment-list";
export { usePayment } from "./data";
```

## Checklist après création

- [ ] Le barrel export `index.ts` est en place
- [ ] Au moins un schéma Zod est défini pour l'entité principale
- [ ] Les types sont dérivés du schéma Zod (pas de duplication)
- [ ] Le module n'importe rien depuis les fichiers internes d'un autre module feature
- [ ] Le hook façade dans `data/index.ts` est la seule interface d'accès aux données
- [ ] Les stores éventuels suivent le bon pattern (global simple ou local avec Context)
- [ ] Le flux de dépendances est unidirectionnel
