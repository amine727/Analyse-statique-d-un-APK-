# Analyse-statique-d-un-APK-
# 🔍 Analyse Statique – UnCrackable-Level1.apk

## 📌 Objectif

Réaliser une analyse statique d’une application Android afin d’identifier :

- La structure interne de l’application
- Les mécanismes de sécurité implémentés
- Les éventuelles faiblesses
- Comparer les outils JADX et JD-GUI

---

# 🧩 Task 1 – Vérification de l’APK

## 📂 Présence du fichier

L’APK a été placé dans le dossier :

C:\APK-Analysis


## 🔎 Vérification du header ZIP

Commande utilisée :

```powershell
Get-Content .\UnCrackable-Level1.apk -TotalCount 4 | Format-Hex
```

Résultat observé :

50 4B

✔ Cela confirme que l’APK est une archive ZIP valide.

### 📸 Capture
<img width="1143" height="1098" alt="1" src="https://github.com/user-attachments/assets/ee7a0148-0d80-411f-8b47-4d542b483746" />


---

## 📦 Listing du contenu de l’APK

Commande utilisée :

```powershell
[System.IO.Compression.ZipFile]::OpenRead($apk).Entries | Select-Object -ExpandProperty FullName
```

Fichiers principaux détectés :

- AndroidManifest.xml
- classes.dex
- META-INF/
- res/
- resources.arsc

### 📸 Capture
<img width="1568" height="445" alt="2" src="https://github.com/user-attachments/assets/1864911a-b121-4388-874b-42fbb57458d2" />

---

# 🧩 Task 2 – Provenance

L’APK UnCrackable-Level1.apk a été fourni par l’enseignant dans le cadre du laboratoire.

Objectif pédagogique :
Analyser les mécanismes anti-tampering et anti-reverse engineering.

---

# 🧩 Task 3 – Analyse avec JADX

## 📦 Structure détectée

Package principal :

sg.vantagepoint.uncrackable1

Classes principales :

- MainActivity
- a (vérification du secret)
- b (détection debug)
- c (détection root)

### 📸 Capture – Structure JADX
<img width="1483" height="1089" alt="6" src="https://github.com/user-attachments/assets/135c1613-7356-4fdd-8059-739ad21262c6" />

---

## 🔐 Mécanismes de sécurité identifiés

### 1️⃣ Détection Root

Implémentation dans :

sg.vantagepoint.a.c

Vérifications effectuées :

- Recherche de l’exécutable `su`
- Vérification de Build.TAGS contenant "test-keys"
- Recherche de fichiers liés à SuperSU

---

### 2️⃣ Détection Debuggable

Implémentation dans :

sg.vantagepoint.a.b

Vérification du flag :

ApplicationInfo.FLAG_DEBUGGABLE

---

### 3️⃣ Vérification du Secret

Localisation :

sg.vantagepoint.uncrackable1.a

Clé AES codée en dur :

8d127684cbc37c17616d806cf50473cc

Ciphertext Base64 :

5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=

Le secret est :
- Déchiffré en AES
- Comparé avec l’entrée utilisateur



---

# 🧩 Task 4 – Recherche de chaînes sensibles

| Recherche | Résultat | Niveau de risque |
|------------|-----------|------------------|
| http:// | Aucun endpoint API externe trouvé | Faible |
| https:// | Présent uniquement dans bibliothèques Android | Faible |
| su | Détection root implémentée | Faible |
| debug | Vérification FLAG_DEBUGGABLE | Moyen |
| AES | Clé cryptographique codée en dur | Moyen |

---

# 🧩 Task 5 – dex2jar

## 📦 Extraction de classes.dex

Extraction effectuée avec PowerShell.

### 📸 Capture
<img width="1386" height="343" alt="3" src="https://github.com/user-attachments/assets/f64f7b5e-8c45-4abc-a8e1-bb7b3ecc7517" />

---

## 🔄 Conversion DEX → JAR

Commande utilisée :

```powershell
.\d2j-dex2jar.bat C:\APK-Analysis\dex_out\classes.dex -o C:\APK-Analysis\app.jar
```

Résultat :

app.jar

### 📸 Capture
<img width="1730" height="425" alt="4" src="https://github.com/user-attachments/assets/cc36ed13-1ca2-4035-9faa-d099055b7ba6" />

---

# 🧩 Task 6 – Comparaison JADX vs JD-GUI

| Aspect | JADX | JD-GUI |
|--------|------|--------|
| Ouverture APK | Directe | Nécessite conversion |
| Ressources Android | Oui | Non |
| Lisibilité | Meilleure reconstruction | Plus brute |
| Structure | Android complète | Java uniquement |
| Gestion Kotlin | Meilleure | Moins lisible |

### 📸 Capture – JD-GUI
<img width="1293" height="1129" alt="5" src="https://github.com/user-attachments/assets/4882f7cb-cb8f-4a3f-920d-09ad968fc230" />

---

# 🧩 Task 7 – Mini Rapport

# Rapport d'analyse statique - UnCrackable-Level1

---

## A) Informations générales

- **Date d'analyse :** 02/03/2026  
- **APK analysé :** UnCrackable-Level1.apk  
- **Version :** 1.0 (versionCode 1)  
- **Provenance :** Fournie par l’enseignant dans le cadre du laboratoire  
- **Outils utilisés :**
  - JADX GUI
  - dex2jar v2.4
  - JD-GUI
  - PowerShell

---

## B) Résumé exécutif

Cette analyse statique a permis d’identifier plusieurs mécanismes de protection intégrés dans l’application.

L’application implémente :

- Une détection d’appareil rooté
- Une détection du mode debug
- Une vérification d’un secret chiffré en AES

Aucune communication réseau externe n’a été identifiée.

Cependant, une faiblesse importante a été détectée :  
la clé AES utilisée pour vérifier le secret est stockée en dur dans le code source.

### Niveau de risque global : **Moyen**

### Actions prioritaires recommandées :

1. Ne jamais stocker de clé cryptographique en dur dans l’application.
2. Déplacer la logique de validation côté serveur.
3. Remplacer AES/ECB par un mode sécurisé (ex : AES-GCM).

---

## C) Constats détaillés

---

### Constat #1 : Détection Root

**Sévérité :** Faible  

**Description :**  
L’application vérifie si l’appareil est rooté en recherchant :
- La présence de l’exécutable `su`
- La présence de fichiers liés à SuperSU
- La présence de "test-keys" dans Build.TAGS

**Localisation :**  
sg.vantagepoint.a.c  

**Impact potentiel :**  
Empêche l’exécution sur appareil rooté mais peut être contourné par patching.

**Remédiation recommandée :**  
Utiliser des mécanismes serveur ou attestation matérielle (ex : SafetyNet / Play Integrity API).

---

### Constat #2 : Détection du mode Debug

**Sévérité :** Faible  

**Description :**  
L’application vérifie si le flag `FLAG_DEBUGGABLE` est activé.

**Localisation :**  
sg.vantagepoint.a.b  

**Impact potentiel :**  
Empêche l’analyse simple via debug mais peut être contourné facilement.

**Remédiation recommandée :**  
Combiner avec des protections anti-debug plus avancées.

---

### Constat #3 : Clé AES codée en dur

**Sévérité :** Moyenne  

**Description :**  
La clé AES utilisée pour vérifier le secret est codée en dur dans le code source :

```
8d127684cbc37c17616d806cf50473cc
```

Le texte chiffré est également stocké dans l’application.

**Localisation :**  
sg.vantagepoint.uncrackable1.a  

**Impact potentiel :**  
Un attaquant peut :
- Extraire la clé
- Déchiffrer le secret
- Contourner totalement la protection

**Remédiation recommandée :**  
Ne jamais stocker de secret côté client.  
Implémenter la vérification côté serveur.

---

### Constat #4 : Utilisation de AES/ECB

**Sévérité :** Moyenne  

**Description :**  
Le mode AES utilisé est ECB, qui n’est pas recommandé en production.

**Localisation :**  
sg.vantagepoint.a.a  

**Impact potentiel :**  
Faible dans ce contexte pédagogique, mais non conforme aux bonnes pratiques cryptographiques.

**Remédiation recommandée :**  
Utiliser AES-GCM ou AES-CBC avec IV aléatoire.

---

## D) Annexes

### Permissions détectées

Aucune permission sensible particulière détectée.

---

### Composants exportés

- MainActivity (exported = true)

---

### Éléments pertinents

- Application marquée comme debuggable
- Logique de sécurité entièrement côté client
- Application pédagogique destinée à l’apprentissage du reverse engineering

---

# Conclusion

L’application UnCrackable-Level1 met en œuvre plusieurs mécanismes anti-analyse :

- Anti-root
- Anti-debug
- Vérification d’un secret chiffré

Cependant, la présence d’une clé cryptographique codée en dur constitue une faiblesse importante.

Dans un contexte réel, cette architecture serait vulnérable à une analyse statique et à une rétro-ingénierie.


## 🔓 Preuve de validation technique

Après analyse statique du code, il a été identifié que :

- La clé AES est codée en dur dans le code.
- Le texte chiffré est stocké en Base64.
- L’application déchiffre le ciphertext en AES/ECB.
- Le résultat est comparé à l’entrée utilisateur.

En reproduisant le mécanisme de déchiffrement hors de l’application, il est possible de retrouver la valeur attendue.

Cette valeur a été saisie dans l’application exécutée sur un émulateur Android Studio.

### Résultat

L’application affiche :

"Success! This is the correct secret."

📸 Voir capture : <img width="320" height="580" alt="7" src="https://github.com/user-attachments/assets/204e0c0f-f5d3-4d67-ada6-7347b2d9a4c2" />
