# Vinyl — lecteur de musique Android

Application Vanilla JavaScript emballée avec Capacitor. En navigateur, les morceaux et playlists sont choisis manuellement pour le test. Sur Android, le pont natif interroge `MediaStore` pour retrouver les MP3 indexés, les métadonnées et les pochettes; la lecture utilise leurs URI `content://`.

## Pré-requis

- Node.js LTS et npm

## Tester le site en local

Pour ouvrir une version de test sans installer Node.js ni npm, ouvrez directement le fichier `test-local.html` dans Opera GX (double-clic depuis l'Explorateur Windows). Ajoutez les MP3 et fichiers M3U avec les boutons de l'application. Les MP3 contenant des tags ID3v2 peuvent afficher leur titre, artiste et pochette intégrée.

Cette version de test n'a pas besoin de serveur. Elle permet d'ajouter des MP3 directement depuis chaque playlist, de consulter les titres « À suivre » et d'activer la lecture aléatoire. Comme tout site local, le navigateur ne peut pas parcourir vos dossiers sans sélection manuelle. Les fichiers audio ne sont pas conservés après fermeture/rechargement de l'onglet.

### Variante avec serveur de développement

Dans PowerShell, depuis le dossier du projet :

```powershell
npm install
npm run start
```

Ouvrez l'adresse locale affichée par Vite (par défaut `http://localhost:5173`). Les options de test local sont les mêmes que sur `test-local.html`.

## Pré-requis Android

- Android Studio avec Android SDK et JDK 17
- Un téléphone Android ou un émulateur

## Création et compilation de l'APK

Depuis ce dossier, dans PowerShell :

```powershell
npm install
npx cap add android
```

Copiez ensuite les deux sources natives Java de `android-plugin` vers :

```text
android/app/src/main/java/com/example/vinylplayer/
```

Dans `android/app/src/main/AndroidManifest.xml`, ajoutez comme enfants de `<manifest>` (et avant `<application>`) :

```xml
<uses-permission android:name="android.permission.READ_MEDIA_AUDIO" />
<uses-permission
    android:name="android.permission.READ_EXTERNAL_STORAGE"
    android:maxSdkVersion="32" />
```

Remplacez le `MainActivity.java` généré par celui fourni dans `android-plugin` (ou fusionnez l'appel `registerPlugin` dans cette activité). Si le chemin généré diffère, conservez la déclaration `package com.example.vinylplayer` cohérente avec `appId`.

Puis compilez :

```powershell
npm run build
npx cap sync android
cd android
.\gradlew.bat assembleDebug
```

APK debug : `android\app\build\outputs\apk\debug\app-debug.apk`.
Pour l'installer sur un appareil connecté avec le débogage USB activé :

```powershell
.\gradlew.bat installDebug
```

Après toute modification HTML/CSS/JS, depuis la racine du projet, relancez `npm run build` puis `npx cap sync android`.

## Accès au stockage et playlists M3U

Android moderne ne permet pas à une application ordinaire de parcourir librement `/Music` ni de lire tous les fichiers `.m3u` avec la seule permission audio. Capacitor Filesystem ne contourne pas ces restrictions de stockage cloisonné. Ce projet utilise donc le pont natif Capacitor/MediaStore pour scanner les audios autorisés, et le sélecteur Android (Storage Access Framework) quand l'utilisateur importe un `.m3u`. Une playlist importée est rapprochée des morceaux trouvés par nom de fichier; les playlists créées et favoris sont conservés dans `localStorage`.

Le scan automatique des audios ne peut retourner que les fichiers indexés par MediaStore et accessibles avec la permission accordée. Un `.m3u` déjà présent dans le téléphone ne peut pas être lu silencieusement au lancement sans accès explicite au fichier/dossier; demander `MANAGE_EXTERNAL_STORAGE` pour éviter ce choix utilisateur serait une permission générale intrusive, soumise aux règles Play Store, et n'est volontairement pas utilisée.
