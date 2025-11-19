# 📌 TÂCHES PRIORITAIRES - SYSTÈME DE PERMISSIONS

**Créé**: 20 novembre 2025  
**Status**: Roadmap pour implémentation future  
**Dépend de**: Commit `e268044f` (retrobus-interne)  

---

## 🔥 TÂCHES CRITIQUES (FAIRE D'ABORD)

### T1: Tester API endpoints localement
**Priorité**: 🔴 CRITIQUE  
**Durée**: 30 min  
**Done By**: Dev  

**Checklist**:
- [ ] Démarrer le serveur local
- [ ] Tester GET /api/permissions/definitions
- [ ] Tester GET /api/permissions/my-permissions (avec auth)
- [ ] Tester POST /api/permissions/grant (admin)
- [ ] Vérifier réponses JSON
- [ ] Vérifier erreurs 403/401

**Commit After**: NONE (test only)

---

### T2: Prisma migration et test DB
**Priorité**: 🔴 CRITIQUE  
**Durée**: 15 min  
**Done By**: Dev  

**Checklist**:
```bash
cd api
# Générer migration
npx prisma migrate dev --name add_user_permissions_complete

# Vérifier table créée
psql $DATABASE_URL -c "SELECT * FROM user_permissions LIMIT 1"

# Vérifier relation
psql $DATABASE_URL -c "\d site_users" | grep permissions
```

- [ ] Migration réussie
- [ ] Table user_permissions existe
- [ ] Colonnes correctes
- [ ] Indexes créés
- [ ] Relation SiteUser existe

**Commit After**: NONE (DB only)

---

### T3: Tester React hooks localement
**Priorité**: 🔴 CRITIQUE  
**Durée**: 30 min  
**Done By**: Dev  

**Checklist**:
- [ ] Démarrer frontend (npm start)
- [ ] Ouvrir navigateur
- [ ] Login avec utilisateur test
- [ ] Inspecter useUnifiedPermissions hook
- [ ] Vérifier cache sessionStorage
- [ ] Tester PermissionGate masque/affiche
- [ ] Console: vérifier 0 erreurs

**Commit After**: NONE (test only)

---

### T4: Tests E2E complètes
**Priorité**: 🔴 CRITIQUE  
**Durée**: 1h  
**Done By**: QA  

**Checklist**:
```
Login avec chaque rôle:
- [ ] ADMIN: Accès tout
- [ ] MANAGER: Accès large
- [ ] PRESIDENT: Vision stratégique
- [ ] TRESORIER: Finances
- [ ] SECRETAIRE_GENERAL: Admin
- [ ] MEMBER: Lecture limitée

Vérifier pour chaque rôle:
- [ ] API /my-permissions retourne bonnes perms
- [ ] PermissionGate affiche bon contenu
- [ ] Impossible d'accéder UI non-autorisé
- [ ] Cache fonctionne
- [ ] Logout clear cache
- [ ] Relog rechage perms
```

**Commit After**: QA report

---

## ⏱️ TÂCHES À COURT TERME (1 SEMAINE)

### T5: Protéger toutes les routes REST
**Priorité**: 🟠 HAUTE  
**Durée**: 2-3 jours  
**Done By**: Dev  

**Détail**: Ajouter middleware `checkFunctionAccess` sur TOUTES les routes

**Routes à protéger**:
```javascript
// Véhicules
app.get('/api/vehicles', checkFunctionAccess('vehicles.view'), ...)
app.post('/api/vehicles', checkFunctionAccess('vehicles.create'), ...)
app.put('/api/vehicles/:id', checkFunctionAccess('vehicles.edit'), ...)
app.delete('/api/vehicles/:id', checkFunctionAccess('vehicles.delete'), ...)

// Finances
app.get('/api/finance/*', checkFunctionAccess('finance.view'), ...)
app.post('/api/finance/transactions', checkFunctionAccess('finance.create'), ...)

// Événements
app.get('/api/events', checkFunctionAccess('events.view'), ...)
app.post('/api/events', checkFunctionAccess('events.create'), ...)

// Membres
app.get('/api/members', checkFunctionAccess('members.view'), ...)
app.post('/api/members', checkFunctionAccess('members.create'), ...)

// ... Et tous les autres endpoints
```

**Résultat**: 0 route non protégée  
**Commit**: "🔐 Protect all REST routes with middleware"  

---

### T6: Migrer code frontend
**Priorité**: 🟠 HAUTE  
**Durée**: 1-2 jours  
**Done By**: Dev  

**Détail**: Remplacer tous les imports de permissionUtils.js par useUnifiedPermissions.js

**Fichiers à migrer**:
- [ ] src/components/PermissionsManager.jsx
- [ ] src/components/RolePermissionsManager.jsx
- [ ] src/components/*.jsx (tous les composants)
- [ ] src/pages/*.jsx (toutes les pages)
- [ ] src/lib/permissionUtils.js → DELETE
- [ ] src/lib/permissions.js → ARCHIVE (backup)

**Avant**:
```javascript
import { hasPermission } from '../lib/permissions';
import { canUserAccess } from '../lib/permissionUtils';

export function MyComponent() {
  const { user } = useContext(UserContext);
  const can = canUserAccess(user.permissions, 'VEHICLES');
}
```

**Après**:
```javascript
import useUnifiedPermissions from '../hooks/useUnifiedPermissions';

export function MyComponent() {
  const { canAccess } = useUnifiedPermissions();
  const can = canAccess('vehicles.view');
}
```

**Résultat**: 0 reference à permissionUtils.js  
**Commit**: "Refactor: Migrate to unified permissions API"  

---

### T7: Supprimer ancien code
**Priorité**: 🟠 HAUTE  
**Durée**: 30 min  
**Done By**: Dev  

**Détail**: Nettoyer après migration

**À supprimer**:
- [ ] src/lib/permissionUtils.js
- [ ] src/lib/permissions.js (après vérifier 0 usage)
- [ ] src/hooks/usePermissions.js (si pas utilisé)
- [ ] src/components/OldPermissionGate.jsx
- [ ] Ancien code permissions-api.js (après vérifier)

**Resultat**: Codebase plus propre  
**Commit**: "Cleanup: Remove deprecated permission libraries"  

---

## 📋 TÂCHES MOYEN TERME (2-3 SEMAINES)

### T8: Déployer en production
**Priorité**: 🟠 HAUTE  
**Durée**: 1 jour  
**Done By**: DevOps  

**Suivre**: DEPLOYMENT_PERMISSIONS_QUICKSTART.md  

**Steps**:
1. Backup DB
2. Pull retrobus-interne
3. Prisma migrate deploy
4. Prisma generate
5. Restart serveur
6. Vérifier API endpoints
7. Monitorer logs
8. Test chaque rôle

**Rollback**: Voir guide si besoin  

---

### T9: Monitorer + Stabilisation
**Priorité**: 🟠 HAUTE  
**Durée**: 3-5 jours  
**Done By**: DevOps  

**Checklist**:
- [ ] 0 erreurs 500 sur /api/permissions/*
- [ ] < 5% 403 errors (expected)
- [ ] Temps réponse < 500ms
- [ ] Cache hit rate > 80%
- [ ] Audit trail enregistre tous les changes
- [ ] Aucun utilisateur bloqué

**If problems**: See troubleshooting guide  

---

### T10: Tests supplémentaires
**Priorité**: 🟡 MOYENNE  
**Durée**: 2 jours  
**Done By**: QA  

**Tests à faire**:
- [ ] Permissions expiration (expiresAt)
- [ ] Permission revocation
- [ ] Multiple tabs (cache sync)
- [ ] Offline behavior
- [ ] Large user lists
- [ ] Concurrent permission grants
- [ ] Audit trail correctness
- [ ] Role changes

---

## 🎯 TÂCHES FUTURE (1 MOIS+)

### T11: Optimisations performance
**Durée**: Flexible  
**Priorté**: 🟡 MOYENNE  

**Ideas**:
- [ ] Cache backend (Redis)
- [ ] Permission batch loading
- [ ] Audit pagination
- [ ] Permission search
- [ ] Admin dashboard pour permissions

---

### T12: UI Admin permissions
**Durée**: 3-5 jours  
**Priorité**: 🟡 MOYENNE  

**Créer**:
- [ ] Page admin: Grant permissions
- [ ] Page admin: Audit trail
- [ ] Page admin: Manage roles
- [ ] Dashboard: Permission usage
- [ ] Export: Permission reports

---

## 📊 STATUS TRACKING

| Tâche | Priorité | Durée | Status | Owner | Due |
|-------|----------|-------|--------|-------|-----|
| T1: API test local | 🔴 | 30m | ⏳ TODO | Dev | ASAP |
| T2: Prisma migration | 🔴 | 15m | ⏳ TODO | Dev | ASAP |
| T3: React test local | 🔴 | 30m | ⏳ TODO | Dev | ASAP |
| T4: E2E tests | 🔴 | 1h | ⏳ TODO | QA | ASAP |
| T5: Protect routes | 🟠 | 2-3j | ⏳ TODO | Dev | 1 week |
| T6: Migrate frontend | 🟠 | 1-2j | ⏳ TODO | Dev | 1 week |
| T7: Cleanup code | 🟠 | 30m | ⏳ TODO | Dev | 1 week |
| T8: Deploy prod | 🟠 | 1j | ⏳ TODO | DevOps | 2 weeks |
| T9: Monitor + Stabilize | 🟠 | 3-5j | ⏳ TODO | DevOps | 2-3 weeks |
| T10: Extra testing | 🟡 | 2j | ⏳ TODO | QA | 3 weeks |
| T11: Performance | 🟡 | Flex | 📌 BACKLOG | Dev | 1 month+ |
| T12: Admin UI | 🟡 | 3-5j | 📌 BACKLOG | Dev | 1 month+ |

---

## 🚨 RISQUES & MITIGATION

| Risque | Impact | Probabilité | Mitigation |
|--------|--------|-------------|-----------|
| Prisma migration fail | 🔴 HIGH | 🟡 MID | Backup DB, test local d'abord |
| Permissions stale cache | 🟡 MID | 🟡 MID | Logout force refresh, 5min timeout |
| Route unprotected | 🔴 HIGH | 🟡 MID | Checklist toutes routes |
| Perf degradation | 🟡 MID | 🟢 LOW | DB indexes, cache strategy |
| Backward compat break | 🔴 HIGH | 🟢 LOW | Keep old APIs running |

---

## 💡 NOTES IMPORTANTES

1. **Tester local d'abord** avant chaque déploiement
2. **Backup DB** avant chaque migration Prisma
3. **Monitor logs** après déploiement prod
4. **Rollback plan** ready si needed
5. **Documentation** mis à jour après chaque phase
6. **Commit messages** clairs et détaillés
7. **Tests E2E** complètes avant release

---

## 📞 CONTACTS

**Questions sur permissions system**:
- Voir documentation: PERMISSION_UNIFICATION_MIGRATION.md
- Code: api/src/core/FunctionPermissions.js
- API: api/src/unified-permissions-api.js

**Questions sur déploiement**:
- Voir: DEPLOYMENT_PERMISSIONS_QUICKSTART.md

**Questions sur problèmes**:
- Voir troubleshooting en guides
- Check logs: `/var/log/retrobus-api.log`

