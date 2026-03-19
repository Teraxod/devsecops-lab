# 🔒 DevSecOps Lab

![Security](https://github.com/Teraxod/devsecops-lab/workflows/DevSecOps%20Pipeline/badge.svg)

Pipeline CI/CD sécurisé avec détection automatique des vulnérabilités.

## 🏗️ Architecture du pipeline

Sur chaque `push` ou `pull_request`, les jobs suivants s'exécutent en parallèle :

| Job | Outil | Rôle |
|-----|-------|------|
| 🏗️ Build | Docker | Construit et sauvegarde l'image |
| 🔍 SAST | Semgrep | Analyse statique du code source |
| 📦 SCA | npm audit | Vulnérabilités dans les dépendances |
| 🔐 Secrets | Gitleaks | Détecte les credentials dans Git |
| 🐳 Container | Trivy | CVE dans l'image Docker |
| ⚡ DAST | OWASP ZAP | Tests dynamiques sur l'appli lancée |
| 🔍 CodeQL | GitHub | Analyse statique native GitHub |
| 🚦 Security Gate | — | Bloque le merge si vulnérabilités critiques |
| 📊 Report | — | Génère le rapport JSON final |

## 🚨 Vulnérabilités trouvées (avant correction)

| Outil | Vulnérabilité | Sévérité |
|-------|--------------|----------|
| Semgrep | Secrets hardcodés (DB, Stripe, SendGrid) | 🔴 CRITICAL |
| Semgrep | JWT sans expiration | 🟠 HIGH |
| Semgrep | Endpoint `/debug` exposant tous les secrets | 🔴 CRITICAL |
| npm audit | express 4.17.1 → CVE-2022-24999 (ReDoS) | 🟠 HIGH |
| npm audit | jsonwebtoken 8.5.1 → CVE-2022-23529 | 🔴 CRITICAL |
| Gitleaks | Clé API Stripe `sk_live_*` dans le code | 🔴 CRITICAL |
| Gitleaks | Clé SendGrid dans le code | 🔴 CRITICAL |
| Trivy | node:14 → 50+ CVE critiques dans l'OS | 🔴 CRITICAL |
| Trivy | Processus lancé en root | 🟠 HIGH |

## ✅ Corrections appliquées (après)

**1. Secrets → Variables d'environnement**

```js
// Avant ❌
const JWT_SECRET = "secret";

// Après ✅
const SECRET = process.env.JWT_SECRET;
```

**2. Dépendances mises à jour**

```json
"express": "^4.18.2",
"jsonwebtoken": "^9.0.2"
```

**3. Sécurité applicative ajoutée**

- `helmet` → Headers HTTP sécurisés
- `express-rate-limit` → Anti-brute force (5 tentatives / 15 min)
- `express-validator` → Validation des entrées
- JWT avec expiration 1h

**4. Dockerfile sécurisé**

- `node:14` → `node:22-alpine` (surface d'attaque réduite)
- Utilisateur non-root (`nodejs:1001`)
- `npm ci --only=production`
- Healthcheck intégré

**5. Endpoint `/debug` supprimé en production**

## 🚀 Installation

```bash
git clone https://github.com/<username>/<repo>.git
cd devsecops-lab

cp .env.example .env
# Générer un JWT_SECRET fort :
openssl rand -base64 32

docker build -f Dockerfile.secure -t secure-app .
docker run -p 3000:3000 --env-file .env secure-app
```

## 🔧 GitHub Secrets à configurer

`Settings > Secrets and variables > Actions` :

| Secret | Commande pour générer |
|--------|----------------------|
| `JWT_SECRET` | `openssl rand -base64 32` |
| `ADMIN_USER` | `admin` |
| `ADMIN_PASS` | Mot de passe fort |

## 📊 Métriques avant / après

| Métrique | Avant | Après |
|----------|-------|-------|
| Vulnérabilités critiques | 9 | 0 |
| Secrets dans le code | 3 | 0 |
| CVE dans l'image Docker | 50+ | 0 |
| Rate limiting | ❌ | ✅ 5 req/15min |
| JWT expiration | ❌ | ✅ 1h |

## ✅ Checklist finale

- [x] Pipeline s'exécute sans erreur
- [x] Tous les secrets dans GitHub Secrets
- [x] Dépendances à jour (pas de CVE critiques)
- [x] Dockerfile sécurisé (non-root, alpine)
- [x] README complet avec instructions
- [x] Badge de build dans le README
- [x] `.gitignore` contient `.env`
- [x] Security Gate bloque en cas de vulnérabilité

## 📚 Ressources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Semgrep Rules](https://semgrep.dev/explore)
- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [Trivy](https://aquasecurity.github.io/trivy/)
- [Gitleaks](https://github.com/gitleaks/gitleaks)
