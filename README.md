# 🔬 Rapport de Lab — Analyse Dynamique Android avec MobSF & DIVA

> **Module :** Sécurité des Applications Mobiles  
> **Niveau :** Master / Ingénierie  
> **Durée estimée :** 2–3 heures  
> **Auteur :** *(Ton nom)*  
> **Date :** *(Date du lab)*

---

## 📋 Table des matières

1. [Objectifs du lab](#1-objectifs-du-lab)
2. [Prérequis & environnement](#2-prérequis--environnement)
3. [Étape 1 — Création de l'émulateur AVD](#3-étape-1--création-de-lémulateur-avd-sans-play-store)
4. [Étape 2 — Clonage de MobSF](#4-étape-2--clonage-de-mobsf)
5. [Étape 3 — Lancement de l'émulateur via MobSF](#5-étape-3--lancement-de-lémulateur-via-le-script-mobsf)
6. [Étape 4 — Installation de MobSF via Docker](#6-étape-4--installation-et-lancement-de-mobsf-via-docker)
7. [Étape 5 — Téléchargement de l'APK DIVA](#7-étape-5--téléchargement-de-lapk-diva)
8. [Étape 6 — Analyse Statique + Dynamique de DIVA](#8-étape-6--analyse-statique--dynamique-de-diva)
9. [Étape 7 — Tests avancés avec Frida](#9-étape-7--tests-avancés--exploration-avec-frida)
10. [Résultats & observations](#10-résultats--observations)
11. [Tableau des boutons Dynamic Analyzer](#11-tableau-des-boutons-dynamic-analyzer)
12. [Dépannage rapide](#12-dépannage-rapide)
13. [Conclusion](#13-conclusion)

---

## 1. Objectifs du lab

Ce lab a pour objectif de réaliser une **analyse dynamique (runtime)** complète d'une application Android vulnérable en utilisant **MobSF (Mobile Security Framework)** combiné à **Frida** pour l'instrumentation.

| Objectif | Description |
|---|---|
| 🧪 Analyse dynamique | Comprendre le comportement runtime d'une APK Android |
| 📱 Émulateur propre | Configurer un AVD sans Play Store pour éviter le bruit réseau |
| 🐳 Docker | Installer et lancer MobSF via Docker |
| 🎯 DIVA APK | Tester les 13 challenges de vulnérabilités en dynamique |
| 🔍 Détection temps réel | Logs runtime, trafic réseau, proxy HTTPS, instrumentation Frida |

---

## 2. Prérequis & environnement

### Logiciels requis

- **Android Studio** (dernière version) + SDK Platform-Tools (ADB inclus)
- **Docker Desktop** (installé et en cours d'exécution)
- **Git**
- **Minimum 8 Go RAM + processeur 64-bit**
- Connexion internet (pour les premiers downloads)

### Pourquoi un AVD sans Play Store ?

L'émulateur utilisé est un **Android Virtual Device (AVD)** créé sans Google Play Store pour les raisons suivantes :

- ✅ **Pas de bruit de fond** (Google Play Services, Firebase, etc.) → logs et trafic réseau purs
- ✅ **Proxy HTTPS global** applicable sans conflit
- ✅ **Compatible avec le rooting/Frida** (MobSF exige un émulateur rooté)
- ✅ **Performances optimales** sur x86_64
- ⚠️ Limité à Android API ≤ 30 (MobSF ne supporte pas les versions plus récentes car `/system` n'est plus writable)

---

## 3. Étape 1 — Création de l'émulateur AVD sans Play Store

### Procédure

1. Ouvrez **Android Studio** → `Tools` → `AVD Manager` → `Create Virtual Device`
2. Choisissez un téléphone (ex. : **Pixel 5** ou **Pixel 6**)
3. Dans **System Image** :
   - Sélectionnez **Android** ou **Google APIs** — **SANS** l'option « Google Play »
   - Choisissez **API 28 à 30** (recommandé : **API 29 ou 30, x86_64**)
   - Cliquez **Download** si besoin → **Next** → **Finish**
4. Nommez l'AVD : `MobSF_DIVA_API_30`

> ✅ **Vérification :** L'image sélectionnée ne contient aucune icône Play Store.

---

**📸 Screenshot 1 — AVD Manager : sélection du System Image (sans Play Store)**

```
[ Insérer ici screenshot de la fenêtre AVD Manager avec l'image API 29/30 sélectionnée ]
Nom suggéré du fichier : screen_01_avd_system_image.png
```

![Screenshot 1 - AVD System Image](./screenshots/screen_01_avd_system_image.png)

---

**📸 Screenshot 2 — AVD créé et visible dans la liste (nom : MobSF_DIVA_API_30)**

```
[ Insérer ici screenshot de la liste AVD Manager avec votre AVD créé ]
Nom suggéré du fichier : screen_02_avd_created.png
```

![Screenshot 2 - AVD créé](./screenshots/screen_02_avd_created.png)

---

## 4. Étape 2 — Clonage de MobSF

Ouvrez un terminal et exécutez :

```bash
git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF.git
cd Mobile-Security-Framework-MobSF
```

---

**📸 Screenshot 3 — Terminal : clonage de MobSF réussi**

```
[ Insérer ici screenshot du terminal après git clone (affichage "Cloning into..." terminé) ]
Nom suggéré du fichier : screen_03_git_clone.png
```

![Screenshot 3 - Git clone MobSF](./screenshots/screen_03_git_clone.png)

---

## 5. Étape 3 — Lancement de l'émulateur via le script MobSF

Le script MobSF root automatiquement l'émulateur et le configure pour l'analyse dynamique.

### Sur Windows (PowerShell) :

```powershell
scripts\start_avd.ps1
```

### Sur Linux / Mac :

```bash
./scripts/start_avd.sh MobSF_DIVA_API_30
```

Sélectionnez `MobSF_DIVA_API_30` dans la liste proposée.

### Vérification — Ouvrez un nouveau terminal :

```bash
adb devices
```

Résultat attendu :
```
List of devices attached
emulator-5554   device
```

> ⚠️ **Notez l'identifiant** (ex. : `emulator-5554`) — il sera utilisé à l'étape suivante.

---

**📸 Screenshot 4 — Terminal : résultat de `adb devices` (emulator-5554 device)**

```
[ Insérer ici screenshot du terminal montrant "emulator-5554   device" ]
Nom suggéré du fichier : screen_04_adb_devices.png
```

![Screenshot 4 - ADB devices](./screenshots/screen_04_adb_devices.png)

---

**📸 Screenshot 5 — L'émulateur Android démarré (écran d'accueil visible)**

```
[ Insérer ici screenshot de la fenêtre émulateur Android ouverte ]
Nom suggéré du fichier : screen_05_emulator_running.png
```

![Screenshot 5 - Émulateur démarré](./screenshots/screen_05_emulator_running.png)

---

## 6. Étape 4 — Installation et lancement de MobSF via Docker

> ⚠️ **Important : L'émulateur doit être lancé AVANT MobSF.**

### Téléchargement de l'image Docker :

```bash
docker pull opensecurity/mobile-security-framework-mobsf:latest
```

### Lancement de MobSF :

```bash
docker run -it --rm \
  -p 8000:8000 \
  -e MOBSF_ANALYZER_IDENTIFIER=emulator-5554 \
  opensecurity/mobile-security-framework-mobsf:latest
```

> 🔁 Remplacez `emulator-5554` par votre identifiant exact !

### Accès à l'interface :

Ouvrez votre navigateur : **http://127.0.0.1:8000**

| Champ | Valeur |
|---|---|
| Login | `mobsf` |
| Mot de passe | `mobsf` |

---

**📸 Screenshot 6 — Docker : lancement de MobSF (logs de démarrage dans le terminal)**

```
[ Insérer ici screenshot du terminal Docker avec les logs de démarrage MobSF ]
Nom suggéré du fichier : screen_06_docker_mobsf_start.png
```

![Screenshot 6 - Docker MobSF](./screenshots/screen_06_docker_mobsf_start.png)

---

**📸 Screenshot 7 — Interface MobSF accessible dans le navigateur (http://127.0.0.1:8000)**

```
[ Insérer ici screenshot de la page d'accueil MobSF dans le navigateur ]
Nom suggéré du fichier : screen_07_mobsf_homepage.png
```

![Screenshot 7 - MobSF Homepage](./screenshots/screen_07_mobsf_homepage.png)

---

## 7. Étape 5 — Téléchargement de l'APK DIVA

**DIVA (Damn Insecure and Vulnerable App)** est une application Android volontairement vulnérable proposant **13 challenges** de sécurité.

| Challenge | Type de vulnérabilité |
|---|---|
| 1 | Insecure Logging |
| 2 | Hardcoded Credentials (Part 1) |
| 3 | Insecure Data Storage (Part 1) |
| 4 | Insecure Data Storage (Part 2) |
| 5 | Insecure Data Storage (Part 3) |
| 6 | Insecure Data Storage (Part 4) |
| 7 | Input Validation (Part 1) |
| 8 | Input Validation (Part 2) |
| 9 | Access Control (Part 1) |
| 10 | Access Control (Part 2) |
| 11 | Access Control (Part 3) |
| 12 | Hardcoded Credentials (Part 2) |
| 13 | Input Validation (Part 3) |

### Sources de téléchargement :

- 🔗 Site officiel : http://www.payatu.com/damn-insecure-and-vulnerable-app/
- 🔗 GitHub : https://github.com/payatu/diva-android

Gardez le fichier `diva.apk` (ou `DIVA-debug.apk`) sur votre bureau.

---

## 8. Étape 6 — Analyse Statique + Dynamique de DIVA

### 8.1 Upload & Analyse Statique

1. Dans MobSF → **Upload & Analyze** → sélectionnez votre `diva.apk`
2. Attendez la fin de l'**analyse statique** (manifest, permissions, code source, etc.)

---

**📸 Screenshot 8 — MobSF : upload de l'APK DIVA en cours**

```
[ Insérer ici screenshot de la page Upload de MobSF avec l'APK DIVA ]
Nom suggéré du fichier : screen_08_mobsf_upload.png
```

![Screenshot 8 - Upload APK](./screenshots/screen_08_mobsf_upload.png)

---

**📸 Screenshot 9 — Rapport d'analyse statique (Security Score, permissions, activities exportées)**

```
[ Insérer ici screenshot du rapport statique MobSF (score, APP SCORES, FILE INFORMATION) ]
Nom suggéré du fichier : screen_09_static_report.png
```

![Screenshot 9 - Rapport statique](./screenshots/screen_09_static_report.png)

---

### 8.2 Lancement de l'Analyse Dynamique

3. Dans le rapport de scan → cliquez sur **Start Dynamic Analysis**

MobSF va automatiquement :
- ✅ Installer DIVA sur l'émulateur
- ✅ Lancer Frida Server
- ✅ Configurer le proxy HTTPS global
- ✅ Ouvrir l'interface Dynamic Analyzer

---

**📸 Screenshot 10 — Bouton "Start Dynamic Analysis" dans le rapport MobSF**

```
[ Insérer ici screenshot du rapport avec le bouton "Start Dynamic Analysis" visible ]
Nom suggéré du fichier : screen_10_start_dynamic.png
```

![Screenshot 10 - Start Dynamic Analysis](./screenshots/screen_10_start_dynamic.png)

---

**📸 Screenshot 11 — Interface Dynamic Analyzer ouverte avec DIVA sur l'émulateur**

```
[ Insérer ici screenshot de l'interface Dynamic Analyzer MobSF avec l'émulateur visible ]
Nom suggéré du fichier : screen_11_dynamic_analyzer.png
```

![Screenshot 11 - Dynamic Analyzer](./screenshots/screen_11_dynamic_analyzer.png)

---

### 8.3 Exploration dans l'émulateur

Dans l'émulateur :
- Lancez l'application **DIVA**
- Explorez les **13 challenges** (Insecure Logging, Hardcoded, Insecure Storage, etc.)

Dans **MobSF Dynamic Analyzer** :

| Section | Ce que vous observez |
|---|---|
| **Runtime Logs** | Tous les logs de l'application en live |
| **Network Traffic** | Tout le trafic HTTP/HTTPS intercepté (même SSL !) |
| **Frida** | Hooker des méthodes Java en temps réel |
| **File Monitor** | Fichiers créés/modifiés par l'app |
| **Intent Monitor** | Intents envoyés/reçus |

---

**📸 Screenshot 12 — Runtime Logs : logs en direct de DIVA**

```
[ Insérer ici screenshot de l'onglet Logcat/Runtime Logs de MobSF ]
Nom suggéré du fichier : screen_12_runtime_logs.png
```

![Screenshot 12 - Runtime Logs](./screenshots/screen_12_runtime_logs.png)

---

**📸 Screenshot 13 — Network Traffic : trafic HTTP/HTTPS intercepté**

```
[ Insérer ici screenshot de l'onglet Network Traffic MobSF ]
Nom suggéré du fichier : screen_13_network_traffic.png
```

![Screenshot 13 - Network Traffic](./screenshots/screen_13_network_traffic.png)

---

### 8.4 Exemples concrets observés avec DIVA

- **Insecure Data Storage** → MobSF capture les fichiers écrits en clair sur le stockage
- **Access Control** → Les intents non sécurisés sont visibles dans l'Intent Monitor
- **Insecure Logging** → Les mots de passe apparaissent en clair dans les Runtime Logs

---

**📸 Screenshot 14 — Challenge "Insecure Data Storage" : fichiers captés en clair**

```
[ Insérer ici screenshot montrant un fichier insecure détecté dans MobSF ]
Nom suggéré du fichier : screen_14_insecure_storage.png
```

![Screenshot 14 - Insecure Storage](./screenshots/screen_14_insecure_storage.png)

---

## 9. Étape 7 — Tests avancés & Exploration avec Frida

### Instrumentation Frida

Dans l'onglet **Frida Code Editor** de MobSF :

1. Cliquez **Spawn & Inject** pour hooker l'application
2. Sélectionnez les **Default Frida Scripts** :
   - ✅ API Monitoring
   - ✅ SSL Pinning Bypass
   - ✅ Root Detection Bypass
   - ✅ Debugger Check Bypass
   - ✅ Clipboard Monitor

---

**📸 Screenshot 15 — Frida Code Editor avec scripts activés**

```
[ Insérer ici screenshot du panneau Frida Code Editor avec les scripts cochés ]
Nom suggéré du fichier : screen_15_frida_scripts.png
```

![Screenshot 15 - Frida Scripts](./screenshots/screen_15_frida_scripts.png)

---

### Actions supplémentaires

- Modifiez les inputs dans DIVA et observez les changements en live
- Exportez le **rapport dynamique complet** (PDF ou JSON)
- Arrêtez/re-démarrez l'analyse pour tester plusieurs scénarios

---

**📸 Screenshot 16 — Génération du rapport dynamique final (Generate Report)**

```
[ Insérer ici screenshot du bouton Generate Report ou du rapport généré ]
Nom suggéré du fichier : screen_16_generate_report.png
```

![Screenshot 16 - Generate Report](./screenshots/screen_16_generate_report.png)

---

## 10. Résultats & Observations

> *(Complétez cette section avec vos propres observations après avoir fait le lab)*

### Vulnérabilités détectées

| # | Challenge DIVA | Vulnérabilité | Observée dans MobSF |
|---|---|---|---|
| 1 | Insecure Logging | Credentials en clair dans Logcat | ☐ Oui / ☐ Non |
| 2 | Hardcoded Credentials | Mot de passe en dur dans l'APK | ☐ Oui / ☐ Non |
| 3 | Insecure Data Storage | Fichier SharedPreferences en clair | ☐ Oui / ☐ Non |
| 9 | Access Control | Activity exportée accessible sans auth | ☐ Oui / ☐ Non |

### Score de sécurité statique

| Métrique | Valeur observée |
|---|---|
| Security Score | *(ex. : 36/100)* |
| Trackers détectés | *(ex. : 0/432)* |
| Exported Activities | *(ex. : 2/17)* |
| Exported Providers | *(ex. : 1/1)* |

---

## 11. Tableau des boutons Dynamic Analyzer

| Bouton | Rôle principal | Utilité en analyse dynamique |
|---|---|---|
| **Stop Screen** | Arrêter l'affichage de l'écran distant | Interrompre le mirroring de l'écran de l'émulateur |
| **Remove Root CA** | Supprimer le certificat racine MobSF | Nettoyer l'environnement après interception HTTPS |
| **Unset HTTP(S) Proxy** | Désactiver le proxy HTTP/HTTPS | Arrêter la redirection du trafic réseau |
| **TLS/SSL Security Tester** | Tester la sécurité TLS/SSL | Vérifier la validation des certificats |
| **Exported Activity Tester** | Tester les activities exportées | Détecter les activités exploitables depuis l'extérieur |
| **Activity Tester** | Tester les activités de l'app | Observer les écrans internes |
| **Get Dependencies** | Récupérer les dépendances | Identifier bibliothèques et composants |
| **Take a Screenshot** | Capturer l'écran courant | Documenter une étape de test |
| **Logcat Stream** | Afficher les logs Android en temps réel | Détecter erreurs, exceptions, fuites d'informations |
| **Generate Report** | Générer le rapport final | Regrouper les résultats de l'analyse dynamique |

---

## 12. Dépannage rapide

| Problème | Solution |
|---|---|
| **"Dynamic Analysis Failed"** | Vérifiez que l'émulateur est lancé **avant** MobSF + `adb devices` montre bien le device |
| **Problème de connexion Docker** | Ajoutez `--net=host` à la commande docker (surtout sur Linux) |
| **Émulateur lent** | Utilisez une API 29 x86_64 |
| **Sur Windows** | Utilisez `start_avd.ps1` et le même identifiant exact |
| **MobSF ne voit pas l'émulateur** | Relancez Docker avec la bonne variable `MOBSF_ANALYZER_IDENTIFIER` |

---

## 13. Conclusion

Ce lab a permis de réaliser une **analyse dynamique complète** d'une application Android vulnérable en conditions réelles, en combinant :

- 🐳 **MobSF** comme plateforme d'analyse tout-en-un (statique + dynamique)
- 📱 **AVD rooté** sans Play Store pour un environnement propre et contrôlé
- 🔧 **Frida** pour l'instrumentation et le bypass de protections
- 🎯 **DIVA** comme application cible avec 13 challenges de vulnérabilités

### Compétences acquises

- Configuration d'un environnement d'analyse mobile sécurisé
- Interception du trafic HTTPS (même SSL) via proxy MobSF
- Instrumentation d'une application Android avec Frida
- Détection de vulnérabilités OWASP MASVS en runtime

### Ressources officielles

- 📖 Doc MobSF Dynamic : https://github.com/MobSF/docs
- 📱 DIVA challenges : https://github.com/payatu/diva-android
- 📚 OWASP MASTG : https://mas.owasp.org/MASTG/

---

*Rapport généré dans le cadre du lab MobSF + DIVA — Analyse Dynamique Android*
