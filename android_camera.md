# Utiliser l'appareil photo et les enregistrer dans le cache

Dans cette quête, tu apprendras à capture une photo en déléguant le travail à une autre application de caméra du téléphone

## Objectifs

* Prendre une photo avec l'application caméra et l'enregistrer en taille réelle dans le téléphone

## Etapes

* Prendre une photo avec une application caméra
* Obtenir la vignette de la photo
* Enregistrer la photo en taille réelle dans le téléphone - Partie 1
* Enregistrer la photo en taille réelle dans le téléphone - Partie 2

### Prendre une photo avec une application caméra

Dans ce chapitre, tu apprendras à prendre une photo pour toi.

Pour cela, il faut indiquer dans ton fichier `Manifest` que tu vas avoir besoin d'utiliser la caméra de ton téléphone.

``` java
	<uses-feature
		android:name="android.hardware.camera"
		android:required="true"/>
```

Pour déléguer des actions à d'autres applications, Android utilise une méthode `Intent` qui décrit ce que tu veux faire.

``` java
    public static final int REQUEST_IMAGE_CAPTURE = 1;
    
    private void dispatchTakePictureIntent() {
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
            startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
        }
    }
```

L'Intent sur `MediaStore.ACTION_IMAGE_CAPTURE` va te permettre d'accéder à la caméra du téléphone.

### Obtenir la vignette de la photo

Une fois la photo prise, tu voudras probablement la récupérer et en faire quelque chose.

L'application caméra de ton téléphone va encoder la photo dans le retour du `Intent` grâce à la méthode `onActivityResult()` sous le format *Bitmap*. 
Grâce à cela, tu vas pouvoir récupérer cette image et l'afficher dans un `ImageView`.

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

### Enregistrer la photo en taille réelle dans le téléphone - Partie 1

Dans le chapitre précédent tu as récupérer la miniature de la photo prise avec le téléphone grâce au `onActivityResult()`.
Si tu veux que Android enregistre une photo en taille réelle, il faut lui donner un répertoire dans laquelle l'enregistrer.

En général, toutes les photo tu captures avec ta caméra doivent être enregistrées sur le périphérique dans le stockage externe public afin qu'elles soient accessibles à toutes les applications.

Le répertoire dans lequel elles sont partagées requiert les autorisations `READ_EXTERNAL_STORAGE` et `WRITE_EXTERNAL_STORAGE`. 
L'autorisation d'écriture autorise implicitement la lecture. Par conséquent, tu n'auras besoin de demander qu'une seule autorisation dans le `Manifest`.

``` java
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

Pour pouvoir enregistrer ton image dans le répertoire, tu vas devoir donner un nom à tes images. 
Voici un exemple de méthode qui renvoie un nom de fichier unique pour une nouvelle photo en utilisant l'heure et la date de prise de vue.


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

#### Ressources

* [Save the full-size photo](https://developer.android.com/training/camera/photobasics#TaskPath)

### Enregistrer la photo en taille réelle dans le téléphone - Partie 2

Maintenant que tu as renommée ton image, tu vas vouloir enregister celle-ci.
Tu vas donc pouvoir modifier la méthode `dispatchTakePictureIntent()` que tu appelleras pour activer la caméra.

``` java
    private Uri mFileUri = null;
    
    private void dispatchTakePictureIntent() {
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
            File photoFile = null;
            try {
                photoFile = createImageFile();
            } catch (IOException) {
                
            }
            
            if (photoFile != null) {
                mFileUri = FileProvider.getUriForFile(this,
                    "fr.wildcodeschool.photo.fileprovider",
                    photoFile);
                takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, mFileUri);
                startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
            }
        }
    }
```

*Attention : tu reçois désormais une image au format Uri*

Tu vas maintenant devoir configurer le `FileProvider` dans ton `Manifest`.

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

Dans `provider`, tu peux voir que le répertoire est configuré dans un fichier de ressources : *res/xml/file_paths.xml*

``` java
    <?xml version="1.0" encoding="utf-8"?>
    <paths xmlns:android="http://shemas.android.com/apk/res/android">
        <external-path
            name="my-images"
            path="Android/data/fr.wildcodeschool.photo/files/Pictures" />
    </paths>
```

Le composant `path` renvoie au chemin où seront stockées tes images sur ton téléphone.

*Penses à remplacer "fr.wildcodeschool.photo" par le nom du package de ton application*

#### Ressources

* [Save the full-size photo](https://developer.android.com/training/camera/photobasics#TaskPath)

## Challenge

### Enregistrer et afficher une photo en taille réelle

1. Dans une nouvelle application, dans `MainActivity` implémentes un bouton qui va te permettre de prendre une photo.
3. Héberges l'application sur ton dépôt *GitHub* et partage le lien en solution

### Critères de validation

* La photo qui s'affiche n'est pas la miniature
* La photo est enregistrée dans le téléphone dans le répertoire défini
