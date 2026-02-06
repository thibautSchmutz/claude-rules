# 🤖 Claude Code Rules

Set de règles et de guidelines pour Claude Code, à copier dans chaque projet.

## Installation

```bash
# Cloner le repo
git clone https://github.com/thibautSchmutz/claude-rules.git /tmp/claude-rules

# Copier les fichiers dans ton projet
cp -r /tmp/claude-rules/.claude ./.claude

# Nettoyer
rm -rf /tmp/claude-rules
```

Ou via un script one-liner :

```bash
npx degit TON_USERNAME/claude-rules /tmp/claude-rules && cp -r /tmp/claude-rules/.claude ./.claude && rm -rf /tmp/claude-rules
```

## Structure

```
└── .claude/
    ├── CLAUDE.md                       # Mémoire projet principale (à personnaliser)
    ├── settings.json                      # Permissions et hooks
    ├── rules/
    │   ├── general.md                     # Principes, conventions universelles
    │   ├── typescript.md                  # Règles TypeScript
    │   ├── frontend/
    │   │   ├── react.md                   # Règles React / composants
    │   │   ├── styles.md                  # Règles Tailwind CSS
    │   │   └── state.md                   # Règles state management
    │   ├── backend/
    │   │   ├── api.md                     # Règles API REST / Hono
    │   │   └── supabase.md                # Règles Supabase / base de données
    │   ├── mobile/
    │   │   └── expo.md                    # Règles React Native + Expo
    │   └── testing/
    │       └── testing.md                 # Règles de tests (Playwright, Maestro, Vitest)
    └── skills/
        ├── code-review/
        │   └── SKILL.md                   # Skill de revue de code
        ├── create-feature/
        │   └── SKILL.md                   # Scaffolding module feature frontend/mobile
        └── create-backend-feature/
            └── SKILL.md                   # Scaffolding module feature backend

```

## Après installation

1. **Personnaliser `CLAUDE.md`** : remplacer les placeholders `[...]` par les infos spécifiques au projet.
2. **Supprimer les rules inutiles** :
   - Projet web-only → supprimer `.claude/rules/mobile/` et `.claude/skills/create-backend-feature/`
   - Projet mobile-only → supprimer `.claude/rules/frontend/` et `.claude/rules/backend/`
   - Projet backend-only → supprimer `.claude/rules/frontend/`, `.claude/rules/mobile/` et `.claude/skills/create-feature/`
3. **Adapter `settings.json`** : ajuster les permissions selon les besoins du projet.

## Mise à jour

Ces fichiers sont copiés, pas liés. Pour intégrer des évolutions :

```bash
git clone https://github.com/thibautSchmutz/claude-rules.git /tmp/claude-rules
diff -r ./.claude /tmp/claude-rules/.claude
```
