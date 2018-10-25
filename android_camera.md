# Utiliser l'appareil photo et les enregistrer dans le cache

Dans cette quête, tu apprendras à capturer une photo sur ton terminal Android, tout en déléguant le travail à l'application de caméra.

## Objectifs

* Prendre une photo avec l'application par défaut
* Enregistrer une miniature
* Enregistrer la photo en taille réelle

## Etapes

### Prendre une photo avec l'application par défaut

Dans ce chapitre, tu apprendras à prendre une photo en utilisant l'application prévu par défaut sur ton terminal.

Pour cela, tu vas devoir indiquer ton besoin d'utiliser la caméra de ton téléphone dans ton fichier `Manifest` .

``` java
<uses-feature
    android:name="android.hardware.camera"
    android:required="true"/>
```

Pour déléguer des actions à d'autres applications, Android utilise un `Intent`, qui décrit l'action à réaliser :

``` java
public static final int REQUEST_IMAGE_CAPTURE = 1234;

private void dispatchTakePictureIntent() {
    // ouvrir l'application de prise de photo
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    
    // lors de la validation de la photo
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        // déclenche l'appel de onActivityResult
        startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
    }
}
```

L'`Intent` sur `MediaStore.ACTION_IMAGE_CAPTURE` va te permettre d'accéder à la caméra du téléphone.

La valeur de la constante `REQUEST_IMAGE_CAPTURE` est importante : elle servira de "clé" de réponse quand la photo sera prise : il ne faut pas qu'une autre constante ait la même valeur dans ta classe !

#### Ressources

* [Comment gérer startActivityForResult sur Android ?](https://code.i-harness.com/fr/q/9eccf7)

### Obtenir la vignette de la photo

Une fois la photo prise, tu voudras probablement la récupérer afin de l'afficher quelque part.

L'application caméra de ton téléphone va encoder la photo au format *Bitmap*, et la retourner à l'`Intent` dans la méthode `onActivityResult()` . 

Grâce à cela, tu vas pouvoir récupérer cette image et l'afficher dans un `ImageView` :

``` java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
        Bundle extras = data.getExtras();
        Bitmap imageBitmap = (Bitmap) extras.get("data");
        mImageView.setImageBitmap(imageBitmap);
    }
}
```

#### Ressources

* [Getting a Result from an Activity](https://developer.android.com/training/basics/intents/result)

### Enregistrer la photo en taille réelle

Dans le chapitre précédent tu as réussi à récupérer la miniature de la photo grâce au `onActivityResult()`.

Si tu veux que Android sauvegarde une photo en taille réelle, il faut lui préciser dans quel répertoire l'enregistrer.

#### 1. Autorisations en lecture et écriture

En général, toutes les photos tu captures avec ta caméra doivent être enregistrées dans le stockage externe, public, afin qu'elles soient accessibles à toutes les applications.

Le répertoire dans lequel elles sont partagées requiert les autorisations `READ_EXTERNAL_STORAGE` et `WRITE_EXTERNAL_STORAGE`. 
L'autorisation d'écriture permet implicitement la lecture. Par conséquent, tu n'auras besoin de demander qu'une seule autorisation dans le `Manifest` :

``` java
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" android:maxSdkVersion="18" />
```

> Tu peux remarquer l'utilisation du paramètre *android:maxSdkVersion="18"*.
En effet, pour les version d'Android antérieures à la 4.4, cette autorisation est obligatoire !
Elle ne sera donc réellement requise que pour les téléphones avec des version d'Android 4.3 (API 18) et antérieure.

#### 2. Création de l'image

Afin d'enregistrer ton image dans le répertoire, tu vas devoir préciser le fichier dans laquelle elle sera créée : tu vas donc donner un nom à ton image.

Voici un exemple de méthode qui renvoie un nom de fichier unique pour une nouvelle photo, en utilisant l'heure et la date de la prise de vue :


``` java
private File createImageFile() throws IOException {
    String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
    String imgFileName = "JPEG_" + timeStamp + "_";
    File storageDir = getExternalFilesDir(Environment.DIRECTORY_PICTURES);
    File image = File.createTempFile(imgFileName, ".jpg", storageDir);
    return image;
}
```

`storageDir` est le répertoire dans lequel la photo va être sauvegardée.

#### 3. Enregistrement de l'image

Maintenant que tu as renommé ton image, tu vas pouvoir l'enregister.

Pour cela, tu vas modifier la méthode `dispatchTakePictureIntent()`, que tu avais précédemment créée afin de récupérer la vignette :

``` java
// chemin de la photo dans le téléphone
private Uri mFileUri = null;

private void dispatchTakePictureIntent() {
    // ouvrir l'application de prise de photo
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    
    // lors de la validation de la photo
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        // créer le fichier contenant la photo
        File photoFile = null;
        try {
            photoFile = createImageFile();
        } catch (IOException e) {
            // TODO : gérer l'erreur
        }
        
        if (photoFile != null) {
            // récupèrer le chemin de la photo
            mFileUri = FileProvider.getUriForFile(this,
                "fr.wildcodeschool.photo.fileprovider",
                photoFile);
            // déclenche l'appel de onActivityResult
            takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, mFileUri);
            startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
        }
    }
}
```

Tu reçois désormais un lien *Uri* contenant le chemin vers ton image, et non plus l'image en *Bitmap* !

Il te faut donc replacer ta méthode `onActivityResult` par celle-ci :

``` java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
        mImageView.setImageURI(mFileUri);
    }
}
```

Tu vas maintenant devoir configurer le `FileProvider` dans ton `Manifest` :

``` java
<application>
    ...
    <provider
        android:name="android.support.v4.content.FileProvider"
        android:authorities="fr.wildcodeschool.photo.fileprovider"
        android:exported="false"
        android:grantUriPermissions="true">
        <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            android:resource="@xml/file_paths" />
    </provider>
    ...
</application>
```

Dans `provider`, tu peux voir que le répertoire est configuré dans un fichier de ressources : *res/xml/file_paths.xml* qu'il te faudra créer, avec le contenu :

``` java
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://shemas.android.com/apk/res/android">
    <external-path
        name="my-images"
        path="Android/data/fr.wildcodeschool.photo/files/Pictures" />
</paths>
```

Le composant `path` renvoie au chemin où seront stockées tes images sur ton téléphone.

> Attention : penses à remplacer "fr.wildcodeschool.photo" par le nom du package de ton application

#### Ressources

* [Save the full-size photo](https://developer.android.com/training/camera/photobasics#TaskPath)
* [What is an URI ?](https://stackoverflow.com/a/31600191)

## Challenge

### Enregistrer et afficher une photo en taille réelle

1. Dans une nouvelle application, dans `MainActivity` implémentes un bouton qui va te permettre de prendre une photo.
2. Affiche la photo dans une `ImageView`, en utilisant le code du chapitre "Enregistrer la photo en taille réelle"
3. Héberges l'application sur ton dépôt *GitHub* et partage le lien en solution

### Critères de validation

* La photo qui s'affiche n'est pas la miniature
* La photo est bien enregistrée dans le téléphone dans le répertoire défini