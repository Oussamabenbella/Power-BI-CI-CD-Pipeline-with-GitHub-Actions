# Guide de Test - Power BI CI/CD Pipeline

## ✅ Étape 1 : Initialisation Git (COMPLÉTÉ)

Le repository Git a été initialisé avec succès.

## 📥 Étape 2 : Télécharger le Projet Power BI de Test

### Option A : Utiliser Contoso Sample (Recommandé pour tester)

1. **Télécharger les données Contoso** :
   - Aller sur : https://github.com/sql-bi/Contoso-Data-Generator-V2-Data/releases/tag/ready-to-use-data
   - Télécharger le fichier : `pbix-10k.7z` (environ 2 MB)
   - Extraire l'archive

2. **Ouvrir dans Power BI Desktop** :
   - Ouvrir le fichier `.pbix` extrait
   - Power BI Desktop va charger le modèle

3. **Sauvegarder en format .pbip** :
   - Aller dans **Fichier** → **Enregistrer sous**
   - Naviguer vers : `C:\Users\user\CascadeProjects\powerbi-cicd-github`
   - **IMPORTANT** : Dans "Type de fichier", sélectionner **"Fichiers de projet Power BI (*.pbip)"**
   - Nommer le fichier : `Contoso 10k`
   - Cliquer sur **Enregistrer**

### Option B : Utiliser Votre Propre Projet

Si vous préférez tester avec votre propre projet :

1. Ouvrir votre fichier `.pbix` dans Power BI Desktop
2. **Fichier** → **Enregistrer sous**
3. Naviguer vers : `C:\Users\user\CascadeProjects\powerbi-cicd-github`
4. Sauvegarder en format `.pbip`
5. **Important** : Notez le nom exact de votre projet

### Option C : Test Sans Projet Power BI (Validation de base)

Si vous n'avez pas Power BI Desktop, vous pouvez :
1. Créer une structure de dossier factice pour tester les workflows
2. Les workflows échoueront à l'étape BPA mais vous verrez qu'ils fonctionnent

## 📝 Étape 3 : Ajuster le Chemin du Projet (Si nécessaire)

Si votre projet ne s'appelle PAS "Contoso 10k", vous devez mettre à jour les workflows :

1. **Ouvrir** : `.github/workflows/Standard_BPA_Runner.yml`
2. **Trouver la ligne** (environ ligne 23) :
   ```yaml
   ./TabularEditor.exe "Contoso 10k.SemanticModel/definition/database.tmdl" -A ...
   ```
3. **Remplacer** `Contoso 10k` par le nom exact de votre projet
4. **Répéter** pour `.github/workflows/Custom_BPA_Runner.yml`

## 🌐 Étape 4 : Créer un Repository GitHub

### 4.1 Créer le Repository sur GitHub

1. Aller sur : https://github.com/new
2. Remplir les informations :
   - **Repository name** : `powerbi-cicd-github` (ou votre choix)
   - **Description** : "Power BI CI/CD Pipeline with GitHub Actions"
   - **Visibility** : Public ou Private (votre choix)
   - **NE PAS** cocher "Add a README file"
3. Cliquer sur **Create repository**

### 4.2 Connecter votre Repository Local

GitHub vous montrera des commandes. Utilisez celles-ci dans le terminal :

```bash
# Ajouter tous les fichiers
git add .

# Créer le premier commit
git commit -m "Initial commit: Power BI CI/CD pipeline setup"

# Ajouter le remote (remplacer YOUR_USERNAME)
git remote add origin https://github.com/YOUR_USERNAME/powerbi-cicd-github.git

# Pousser vers GitHub
git branch -M main
git push -u origin main
```

## 🧪 Étape 5 : Tester les Workflows

### Test 1 : Standard BPA Runner (Manuel)

1. Aller sur votre repository GitHub
2. Cliquer sur l'onglet **Actions**
3. Dans le panneau de gauche, sélectionner **Standard BPA Runner**
4. Cliquer sur le bouton **Run workflow** (à droite)
5. Sélectionner la branche `main`
6. Cliquer sur le bouton vert **Run workflow**

**Que va-t-il se passer ?**
- Le workflow va démarrer (environ 30 secondes)
- Il va télécharger Tabular Editor
- Il va exécuter BPA avec les règles Microsoft officielles
- Résultat attendu : ⚠️ Warnings (normal, le modèle Contoso a des violations)

### Test 2 : Custom BPA Runner (Automatique)

Ce workflow se déclenche automatiquement sur les push/PR. Pour le tester :

#### Option A : Déclencher manuellement
1. Onglet **Actions**
2. Sélectionner **Custom BPA Runner**
3. **Run workflow**

#### Option B : Déclencher par un commit
1. Ouvrir Power BI Desktop avec votre projet
2. Ajouter une mesure qui viole la règle :
   ```dax
   Test Division = [Margin] / [Total Cost]
   ```
3. Sauvegarder le projet
4. Dans le terminal :
   ```bash
   git add .
   git commit -m "Test: Add measure with division operator"
   git push
   ```
5. Le workflow se déclenchera automatiquement

## 📊 Étape 6 : Voir les Résultats

### Pendant l'Exécution
1. Cliquer sur le workflow en cours
2. Cliquer sur le job (ex: "run-bpa")
3. Voir les logs en temps réel

### Après l'Exécution
1. Scroller vers le bas jusqu'à **Artifacts**
2. Télécharger **bpa-results** ou **bpa-results-custom**
3. Extraire et ouvrir `BPA_output.txt`

### Interpréter les Résultats

**Succès (✅)** : Aucune violation Severity 3
**Warning (⚠️)** : Violations Severity 1 ou 2 (workflow continue)
**Échec (❌)** : Violations Severity 3 (workflow échoue)

## 🔍 Tests Supplémentaires

### Test 3 : Modifier les Règles BPA

1. Éditer `BPA_Rule.json`
2. Changer `"Severity": 2` en `"Severity": 3`
3. Commit et push
4. Le workflow devrait maintenant échouer si la règle est violée

### Test 4 : Ajouter une Nouvelle Règle

1. Ouvrir `BPA_Rule.json`
2. Ajouter une règle (voir `docs/BPA_RULES_REFERENCE.md` pour exemples)
3. Commit et push
4. Vérifier que la nouvelle règle est appliquée

### Test 5 : Pull Request Workflow

1. Créer une nouvelle branche :
   ```bash
   git checkout -b feature/test-pr
   ```
2. Faire des modifications
3. Push la branche :
   ```bash
   git push -u origin feature/test-pr
   ```
4. Sur GitHub, créer une Pull Request vers `main`
5. Le workflow Custom BPA Runner devrait se déclencher

## ❓ Dépannage

### Le workflow échoue avec "Path not found"
- Vérifier que le chemin dans le workflow correspond à votre projet
- Le format doit être : `"NomProjet.SemanticModel/definition/database.tmdl"`

### Le workflow échoue avec "Exit code 1"
- C'est normal si vous avez des violations Severity 3
- Regarder les logs pour voir quelle règle a échoué
- Corriger la violation ou réduire la sévérité à 2

### Tabular Editor ne se télécharge pas
- Problème réseau temporaire
- Re-lancer le workflow

### Pas de fichier .pbip généré
- Vérifier que vous utilisez Power BI Desktop récent (2023+)
- Le format .pbip n'est pas disponible dans les anciennes versions

## 📋 Checklist de Test Complet

- [ ] Repository Git initialisé
- [ ] Projet Power BI ajouté en format .pbip
- [ ] Chemins dans les workflows mis à jour (si nécessaire)
- [ ] Repository GitHub créé
- [ ] Code poussé vers GitHub
- [ ] Standard BPA Runner exécuté manuellement
- [ ] Custom BPA Runner testé (manuel ou auto)
- [ ] Résultats BPA téléchargés et vérifiés
- [ ] Règle personnalisée testée
- [ ] Pull Request workflow testé (optionnel)

## 🎯 Prochaines Étapes

Une fois les tests réussis :

1. **Personnaliser les règles** selon vos besoins
2. **Configurer les notifications** (Slack, Email)
3. **Ajouter des workflows avancés** (voir `docs/ADVANCED_WORKFLOWS.md`)
4. **Intégrer dans votre processus de développement**
5. **Former votre équipe** sur l'utilisation

## 📚 Ressources

- `README.md` - Documentation principale
- `QUICK_START.md` - Démarrage rapide
- `docs/TROUBLESHOOTING.md` - Solutions aux problèmes
- `docs/BPA_RULES_REFERENCE.md` - Référence des règles
- `docs/ADVANCED_WORKFLOWS.md` - Workflows avancés

---

**Besoin d'aide ?** Consultez `docs/TROUBLESHOOTING.md` ou ouvrez une issue sur GitHub.
