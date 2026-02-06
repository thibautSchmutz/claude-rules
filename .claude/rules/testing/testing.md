---
paths:
  - "**/*.test.*"
  - "**/*.spec.*"
  - "**/tests/**"
  - "**/e2e/**"
---

# Règles de tests

## Philosophie

- Les tests servent de **documentation produit**. L'arborescence des fichiers de test doit permettre de comprendre les fonctionnalités de l'application d'un coup d'œil.
- **Privilégier les tests end-to-end** qui valident des parcours utilisateur complets. Ils offrent le meilleur retour sur investissement en testant le système dans son ensemble.
- Les **tests unitaires** sont réservés aux parties critiques : logique métier complexe, fonctions utilitaires avec des cas limites, transformations de données.
- Ne pas écrire de tests pour des composants simples, du code trivial, ou du styling.

## Structure des dossiers de tests

La structure est identique pour les trois contextes (frontend, mobile, backend) :

```
tests/
├── utils/                          # Fixtures, mocks, helpers partagés
├── [domaine]/                      # auth/, profile/, payment/...
│   └── [action]/                   # login/, register/... (quand plusieurs variantes)
│       └── [use-case].spec.ts      # Scénario précis
```

La hiérarchie se lit comme une phrase : `auth → login → email-and-password` = "l'application permet de se connecter avec email et mot de passe".

- **Dossiers** = domaine métier (`auth/`, `profile/`, `payment/`)
- **Sous-dossiers** = action, quand elle a plusieurs variantes (`login/`, `register/`)
- **Fichiers** = use case précis en kebab-case, du point de vue utilisateur

### Dossier utils/

Chaque dossier `tests/` contient un dossier `utils/` avec :

- **Mocks** : données de test réutilisables
- **Helpers** : fonctions utilitaires (login, seed, cleanup, factory functions)
- **Fixtures** : configurations de test partagées (Playwright fixtures, instance Hono de test, etc.)

## Tests frontend (Playwright)

```
tests/
├── utils/
│   ├── fixtures.ts               # Fixtures Playwright custom (user connecté, etc.)
│   ├── mocks.ts                  # Données de test
│   └── helpers.ts                # Helpers (login, seed, cleanup)
├── auth/
│   ├── login/
│   │   ├── email-and-password.spec.ts
│   │   └── google-oauth.spec.ts
│   ├── register/
│   │   └── email-and-password.spec.ts
│   └── logout.spec.ts
└── profile/
    ├── update-display-name.spec.ts
    └── upload-avatar.spec.ts
```

Règles Playwright :

- Utiliser les **locators accessibles** en priorité : `getByRole`, `getByLabel`, `getByText`. Éviter les sélecteurs CSS ou les `data-testid` sauf si aucun locator accessible n'est possible.
- Les données de test sont isolées : chaque test crée ses propres données et les nettoie après.
- Pas de `sleep()` ou de délais fixes. Utiliser les mécanismes d'attente de Playwright (`waitForSelector`, `expect().toBeVisible()`).
- Nommer les tests en français de manière descriptive.

```tsx
// ✅ Test e2e lisible
test("l'utilisateur peut se connecter avec son email et mot de passe", async ({ page }) => {
	await page.goto("/login");
	await page.getByLabel("Email").fill("test@example.com");
	await page.getByLabel("Mot de passe").fill("password123");
	await page.getByRole("button", { name: "Se connecter" }).click();

	await expect(page.getByRole("heading", { name: "Dashboard" })).toBeVisible();
});
```

## Tests mobile (Maestro)

```
tests/
├── utils/
│   ├── mocks/                    # Données mockées (JSON)
│   └── scripts/                  # Sous-flows réutilisables (YAML)
│       ├── login.yaml            # Flow de login réutilisable
│       └── clear-state.yaml      # Nettoyage d'état
├── auth/
│   ├── login/
│   │   ├── email-and-password.yaml
│   │   └── google-oauth.yaml
│   └── register/
│       └── email-and-password.yaml
└── onboarding/
    ├── complete-onboarding.yaml
    └── skip-onboarding.yaml
```

Règles Maestro :

- Les flows réutilisables (login, nettoyage) vivent dans `utils/scripts/` et sont appelés via `runFlow`.
- Les données mockées en JSON vivent dans `utils/mocks/`.
- Chaque fichier YAML teste un parcours utilisateur complet.

## Tests backend (Vitest)

```
tests/
├── utils/
│   ├── setup.ts                  # Setup global (seed, cleanup)
│   ├── mocks.ts                  # Données de test
│   ├── helpers.ts                # Helpers (créer un user, générer un token)
│   └── test-app.ts               # Instance Hono configurée pour les tests
├── auth/
│   ├── login/
│   │   ├── email-and-password.test.ts
│   │   └── invalid-credentials.test.ts
│   └── register/
│       ├── email-and-password.test.ts
│       └── duplicate-email.test.ts
└── profile/
    ├── get-own-profile.test.ts
    └── update-profile.test.ts
```

Règles Vitest backend :

- Utiliser `app.request()` de Hono pour tester les routes sans serveur HTTP.
- Tester à la fois les **cas positifs** (parcours normal) et les **cas d'erreur** (credentials invalides, payload malformé, ressource introuvable) — ils documentent les limites de l'API.
- Nommer les fichiers de test d'erreur explicitement : `invalid-credentials.test.ts`, `duplicate-email.test.ts`.
- Nommer les tests en français de manière descriptive.

```tsx
// ✅ Test backend lisible
describe("PATCH /profile/:id", () => {
	test("met à jour le profil avec un payload valide", async () => {
		const user = await seedUser();
		const res = await app.request(`/profile/${user.id}`, {
			method: "PATCH",
			headers: createAuthHeaders(user),
			body: JSON.stringify({ displayName: "Nouveau nom" }),
		});

		expect(res.status).toBe(200);
		const json = await res.json();
		expect(json.data.displayName).toBe("Nouveau nom");
	});
});
```

## Ce qu'il ne faut PAS tester

- Les composants purement présentationnels sans logique.
- Les wrappers triviaux autour de bibliothèques tierces.
- Les types TypeScript (le compilateur s'en charge).
- Le styling (sauf si un comportement conditionnel en dépend).
