#  LAB 4 --Audit statique – UnCrackable-Level1

## Informations générales

| Champ | Valeur |
|-------|--------|
| Date de l’analyse | 26 avril 2026 |
| Analyste | [Ton nom] |
| APK analysé | UnCrackable-Level1.apk |
| Version | 1.0 (versionCode=1) |
| Package | owasp.mstg.uncrackable1 |
| Outils utilisés | JADX-GUI, JD-GUI, dex2jar |

---

## Démarche suivie

### 1. Téléchargement de l’APK

L’APK `UnCrackable-Level1.apk` a été téléchargé depuis le site OWASP puis transféré sur le téléphone, puis récupéré sur le PC via `adb pull`.

<img width="720" height="1600" alt="image" src="https://github.com/user-attachments/assets/b0ffb196-3085-41d2-9ba7-208e44f21030" />


### 2. Vérification de l’intégrité

L’APK a été vérifié comme une archive ZIP valide (en-tête `50 4B 03 04`).

**Hash SHA256** :
1DA8BF57D266109F9A07C01BF7111A1975CE01F190BD9D14BCD3AE3DBEF96F21


<img width="911" height="261" alt="image" src="https://github.com/user-attachments/assets/6619cc9f-b10d-411b-9c4c-6dbc19fd434e" />

<img width="1301" height="699" alt="image" src="https://github.com/user-attachments/assets/46cdee85-85fa-4e93-a584-cc1f77aee344" />

<img width="1012" height="186" alt="image" src="https://github.com/user-attachments/assets/5bba1d3d-e806-44eb-be97-6cb749e89673" />


### 3. Décompilation avec JADX

L’APK a été ouvert dans JADX-GUI pour explorer :
- Le code source Java
- Le fichier `AndroidManifest.xml`
- Les ressources (`strings.xml`, etc.)

<img width="1366" height="711" alt="image" src="https://github.com/user-attachments/assets/a6ac3a38-9650-49ea-b14a-4e207a07bc70" />


### 4. Analyse du manifeste

Le fichier `AndroidManifest.xml` a été examiné pour identifier :
- Les permissions demandées (aucune)
- Les composants exportés (`MainActivity` avec intent-filter)
- Les configurations sensibles (`allowBackup="true"`)


### 5. Recherche de chaînes sensibles

Des recherches globales ont été effectuées sur les mots-clés :
- `secret` → découverte de `SecretKeySpec` (AES)
- `key` → découverte d’une clé de chiffrement
- `http`, `password`, `token` → aucun résultat

<img width="785" height="492" alt="image" src="https://github.com/user-attachments/assets/dec8116c-9daf-4464-a9c4-e2ccd95efbc2" />

<img width="785" height="492" alt="image" src="https://github.com/user-attachments/assets/305b5623-2197-4935-afd9-7bc211aabab7" />

<img width="784" height="490" alt="image" src="https://github.com/user-attachments/assets/1a8c9721-20f6-45c2-a4f2-43f0feb3fced" />


### 6. Conversion DEX → JAR

Le fichier `classes.dex` a été extrait de l’APK puis converti en `app.jar` avec **dex2jar**.

<img width="878" height="285" alt="image" src="https://github.com/user-attachments/assets/a40ca484-941a-4296-a9b0-cefe0d161af0" />

<img width="920" height="62" alt="image" src="https://github.com/user-attachments/assets/718849e2-c4c0-4c59-8e21-f9a998a6fa39" />

<img width="770" height="160" alt="image" src="https://github.com/user-attachments/assets/cc4889b1-cc2b-4f90-89a1-fe7df2da814b" />


### 7. Comparaison JADX vs JD-GUI

La classe `MainActivity` a été comparée entre les deux décompilateurs :

| Aspect | JADX | JD-GUI |
|--------|------|--------|
| Affichage des ressources | Résout les ID en noms (ex: `R.id.edit_text`) | Affiche des ID numériques |
| Commentaires | Ajoute des infos JADX | Absents |
| Lisibilité | Code bien formaté | Code plus brut |

---
<img width="1366" height="730" alt="image" src="https://github.com/user-attachments/assets/6606248e-52e7-482a-a9fd-83813982277c" />


<img width="1365" height="721" alt="image" src="https://github.com/user-attachments/assets/3bc0e748-c195-467b-94aa-36c4bf4272bb" />


## Constats de sécurité

### Constat #1 : allowBackup activé
- **Sévérité** : Élevée
- **Localisation** : `AndroidManifest.xml`
- **Impact** : Extraction des données de l’application via `adb backup`
- **Remédiation** : `android:allowBackup="false"`

### Constat #2 : Activité exportée
- **Sévérité** : Moyenne
- **Localisation** : `MainActivity` avec intent-filter
- **Impact** : Une autre application peut démarrer cette activité
- **Remédiation** : `android:exported="false"`

### Constat #3 : Utilisation de SecretKeySpec (AES)
- **Sévérité** : Moyenne
- **Localisation** : Classe `sg.vantagepoint.a.a`
- **Impact** : Clé de chiffrement potentiellement codée en dur
- **Remédiation** : Utiliser Android Keystore

### Constat #4 : Détection root/debug
- **Sévérité** : Faible
- **Localisation** : `MainActivity.onCreate()`
- **Impact** : Protection contournable facilement
- **Remédiation** : Renforcer les checks ou utiliser ProGuard avancé

---

## Annexes

### Fichiers générés
- `app.jar` – version JAR du bytecode
- `classes.dex` – bytecode Android extrait

### Outils utilisés
- JADX-GUI v1.5.5
- JD-GUI v1.6.6
- dex2jar v2.1

---

## Conclusion

Cette analyse statique a permis d’identifier plusieurs faiblesses de configuration et d’implémentation. Les recommandations proposées permettent d’améliorer la sécurité de l’application.


## Auteur
**H-oubane**
