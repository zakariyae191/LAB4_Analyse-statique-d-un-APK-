# LAB4_Analyse-statique-d-un-APK
Les applications Android sont distribuées sous forme de fichiers `.apk`. Même sans les exécuter, il est possible d’extraire une quantité surprenante d’informations sensibles. Ce document détaille chaque étape suivie : environnement isolé, extraction du bytecode, rétro-ingénierie du manifeste, recherche de secrets codés en dur.


---

## Étape 1 — Mise en place de l’environnement

Un dossier dédié `C:\RE-Mobile` a été créé pour isoler tous les artefacts. L’APK y a été déposé.

### Vérification du type de fichier

Un APK est une archive ZIP. Les bytes magiques `50 4B` (`PK` en ASCII) doivent être présents au début.

<img width="832" height="208" alt="11" src="https://github.com/user-attachments/assets/fa2c41af-b541-4e66-a6e3-cbd1e9a92f33" />

— Analyse du manifeste (AndroidManifest.xml)

Le manifeste a été ouvert directement dans JADX.

### Identité de l’application
<img width="577" height="184" alt="image" src="https://github.com/user-attachments/assets/427f9d8d-1b65-4684-a100-b0b2320b6304" />

<img width="351" height="228" alt="Screenshot 2026-05-07 163813" src="https://github.com/user-attachments/assets/b98c9c7c-0d7a-46fa-b3d6-7ef92ab38a37" />

### Permissions demandées

Aucune. Le fichier ne contient aucun `<uses-permission>`.

### Composants exposés

Une seule activité est déclarée :
```xml
<activity android:name="sg.vantagepoint.uncrackable1.MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```



### Empreinte numérique (SHA-256)

Avant toute manipulation, le hash a été calculé pour garantir l’intégrité.
<img width="1277" height="308" alt="Screenshot 2026-05-07 163907" src="https://github.com/user-attachments/assets/dce39eb3-19a1-451b-9e02-b668c9c6afe2" />


##Comparaison JADX vs JD-GUI

| Critère | JADX | JD-GUI |
|---------|------|--------|
| Lecture native d’APK | oui |  non (nécessite JAR) |
| Affichage du manifeste |  oui |  non |
| Désobfuscation partielle |  bonne |  limitée |
| Reconnaissance des ressources |  oui |  non |
| Cas d’échec | APK protégé (chargeur personnalisé) | rarement utilisé seul |

**Règle à retenir :**  
Toujours commencer par JADX. Passer à dex2jar + JD-GUI uniquement si JADX échoue (ex. : APK avec obscurcissement agressif ou multi-DEX corrompu).

---

## Étape 6 — Synthèse des vulnérabilités identifiées

### Profil général

| Champ | Valeur |
|-------|--------|
| Package | `owasp.mstg.uncrackable1` |
| Version | 1.0 |
| SDK minimum / cible | 19 / 28 |
| Permissions | aucune |
| Composants exportés | MainActivity (implicite) |

## dossier de travaill 
érification rapide avant de poursuivre. Le fichier se trouvait dans C:\APK-Analysis
<img width="812" height="490" alt="Screenshot 2026-05-12 132056" src="https://github.com/user-attachments/assets/0155dc4b-5a64-4099-b7dd-18be691be72a" />

### Tableau des failles

| ID | Découverte | Gravité | Emplacement |
|----|------------|---------|-------------|
| 1 | `allowBackup="true"` → extraction ADB possible | Moyenne | `AndroidManifest.xml` |
| 2 | Détection de débogueur (`isDebuggerConnected`) | Info | `MainActivity` (code Java) |
| 3 | Activité principale exportée implicitement | Faible | `AndroidManifest.xml` |

### Détails complémentaires

**[MOYENNE] Sauvegarde activée**  
Permet à tout utilisateur ayant un câble USB et le mode développeur activé de copier `/data/data/owasp.mstg.uncrackable1/`. Correction : `android:allowBackup="false"`.

**[INFO] Anti-debug**  
L’application vérifie si elle est exécutée sous débogueur. C’est une protection, pas une vulnérabilité. Cela montre que le développeur connaît les techniques d’analyse dynamique.

**[FAIBLE] Point d’entrée exporté**  
`MainActivity` est joignable par d’autres applications via des intents. Aucun traitement dangereux des extras n’a été observé, mais une vérification supplémentaire est recommandée.

---

## Étape 7 — Recommandations finales

1. Désactiver `allowBackup` en production.
2. Revoir tout traitement d’intent dans `MainActivity` (injection possible).
3. Conserver l’absence de permissions et l’interdiction du trafic HTTP clair.


