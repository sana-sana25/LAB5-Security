# 📝 RAPPORT LAB5 — Analyse APK Android (UnCrackable Level 2)

## 🔷 1. Introduction

Dans ce laboratoire, nous avons analysé une application Android nommée UnCrackable Level 2.
L’objectif principal est de comprendre comment une application peut cacher sa logique critique dans une bibliothèque native, et comment un analyste peut malgré tout retrouver le secret attendu.

Cette analyse combine deux niveaux :

- Analyse du code Java avec JADX
- Analyse du code natif avec Ghidra

## 🎯 2. Objectif
- Comprendre le fonctionnement d’un APK Android
- Identifier où est effectuée la vérification
- Analyser le lien entre Java et code natif (JNI)
- Retrouver le mot de passe caché
- Valider le résultat dans l’application

## 🛠️ 3. Outils utilisés
- ADB : installation de l’APK
- JADX : décompilation du code Java
- Ghidra : analyse du code natif (.so)
- Émulateur Android

## 📱 4. Analyse dynamique (observation de l’application)

Après installation de l’application : adb install UnCrackable-Level2.apk

### 🔍 Observation :
Interface simple avec un champ de saisie
Bouton de validation
🧪 Test :
<img width="550" height="668" alt="1" src="https://github.com/user-attachments/assets/60acd4b4-bc3b-4bae-9f24-40ca33e76fc9" />

Entrée de valeurs incorrectes → message : “Nope… That’s not it. Try again.”

👉 L’application compare l’entrée utilisateur à une valeur secrète.

## 🔍 5. Analyse statique avec JADX

Ouverture de l’APK avec JADX : jadx-gui UnCrackable-Level2.apk
📌 Étape clé : MainActivity
<img width="1918" height="1005" alt="2" src="https://github.com/user-attachments/assets/af93957f-b819-4861-afd6-59ae0f429742" />

On trouve la fonction :

public void verify(View view) {
    String string = ((EditText) findViewById(R.id.edit_text)).getText().toString();
    if (this.m.a(string)) {
        // succès
    } else {
        // erreur
    }
}

👉 Conclusion :

L’entrée utilisateur est récupérée ici
La vérification est faite par : m.a(string)

## 🧩 6. Analyse de CodeCheck

Dans la classe CodeCheck :
<img width="1522" height="447" alt="image" src="https://github.com/user-attachments/assets/0d75634b-5715-4b88-8a78-9d3180492589" />

private native boolean bar(byte[] bArr);

public boolean a(String str) {
    return bar(str.getBytes());
}

- Utilisation du mot-clé native
- La fonction bar() n’est pas en Java
  
### Conclusion : La logique de vérification est dans une bibliothèque native.

##  7. Extraction de la bibliothèque native

L’APK est extrait : unzip UnCrackable-Level2.apk

On trouve : lib/x86/libfoo.so
<img width="1348" height="303" alt="image" src="https://github.com/user-attachments/assets/34daf618-e5f5-4127-a0a2-c22dca9b387a" />

## 8. Analyse avec Ghidra

Le fichier libfoo.so est importé dans Ghidra.
<img width="1648" height="997" alt="3" src="https://github.com/user-attachments/assets/d0c07c6c-2a10-40f6-a2c0-e4d291d996ad" />
<img width="1012" height="732" alt="4" src="https://github.com/user-attachments/assets/fe278cc5-46e6-4aa7-9db1-259a63fd07f4" />
<img width="1223" height="630" alt="5" src="https://github.com/user-attachments/assets/d7d5614e-868f-48ad-acb1-ca6e2ecff0af" />

### 🔍 Étape clé : Recherche de la fonction JNI : Java_sg_vantagepoint_uncrackable2_CodeCheck_bar
<img width="981" height="871" alt="image" src="https://github.com/user-attachments/assets/030d8872-4009-441e-b026-da57ed8f7f82" />

## 🧩 9. Analyse de la fonction native

Dans le pseudo-code :

builtin_strncpy(local_30, "Thanks for all the fish", ...);

iVar1 = strncmp(s1, local_30, 0x17);
<img width="1918" height="1031" alt="6" src="https://github.com/user-attachments/assets/e8d1f111-dce3-4e0b-80f3-2734f9e64267" />

--> Interprétation
- La chaîne "Thanks for all the fish" est copiée
- L’entrée utilisateur est comparée avec cette chaîne

## 🔐 10. Résultat (mot de passe)

Le secret attendu est :
### Thanks for all the fish

## 🧪 11. Validation

Après saisie dans l’application :
<img width="437" height="627" alt="7" src="https://github.com/user-attachments/assets/e097a53f-0695-4e4b-8a05-ccee8002c00f" />

👉 Résultat : Success!

## 🔁 12. Flux complet de l’application
Utilisateur
   ↓
MainActivity
   ↓
CodeCheck.a()
   ↓
CodeCheck.bar()
   ↓
libfoo.so
   ↓
strncmp(input, "Thanks for all the fish")
   ↓
Succès / Échec

##  13. Ce qu’il faut retenir
- Une application peut cacher sa logique dans du code natif
- Le mot-clé native est un indicateur clé
- JNI permet de relier Java au C/C++
- Ghidra permet d’analyser du code compilé
- Même le code natif peut être reverse

## ⚠️ 14. Difficultés rencontrées
- Problème de compatibilité Java avec Ghidra
- Installation et configuration de Java 17
- Compréhension du fonctionnement JNI
- Lecture du pseudo-code natif

## 15. Conclusion

Ce laboratoire démontre que la sécurité par obscurcissement (cacher la logique dans du code natif) n’est pas suffisante.
Grâce à des outils comme JADX et Ghidra, il est possible de remonter toute la logique et de retrouver les informations sensibles.

# Auteur
## ASSEKNOUR Sana
### ENSA Marrakech- Cybersecurity 
