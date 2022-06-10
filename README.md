# FileUploader
Service pour Symfony 6.* qui permet l'insertion d'un fichier, la modification et la suppression.

<?php

/*
 * Il prend un fichier, un chemin et renvoie un nom de fichier.
 *
 * (c) Valentin Potriquier <valentinpotriquier@gmail.com>
 */

namespace App\Service;

use Symfony\Component\Filesystem\Filesystem;
use Symfony\Component\HttpFoundation\File\UploadedFile;
use Symfony\Component\String\Slugger\SluggerInterface;

class FileUploaderService
{
    /* -------------------------------------------------------------------------- */
    /*                                Les Attributs                               */
    /* -------------------------------------------------------------------------- */

    private string $targetDirectory;
    private SluggerInterface $slugger;
    private Filesystem $fileSystem;

    /**
     * Le constructeur accueille un répertoire cible, un slugger et un système de fichiers.
     * 
     * @param string $targetDirectory Le répertoire où le fichier sera enregistré.
     * @param SluggerInterface $slugger Il s'agit d'un service qui sera utilisé pour générer un nom de fichier unique pour le fichier téléchargé.
     * @param Filesystem $fileSystem Il s'agit d'un service qui nous permet d'interagir avec le système de fichiers.
     */
    public function __construct(
        string $targetDirectory,
        SluggerInterface $slugger,
        Filesystem $fileSystem
    ) {
        $this->targetDirectory = $targetDirectory;
        $this->slugger = $slugger;
        $this->fileSystem = $fileSystem;
    }

    /* -------------------------------------------------------------------------- */
    /*                           Les méthodes publiques                           */
    /* -------------------------------------------------------------------------- */

    /**
     * Cette fonction prend un fichier téléchargé et un chemin, et renvoie un nom de fichier.
     * 
     * @param UploadedFile $file Le fichier qui a été téléchargé.
     * @param string $path Le chemin d'accès au répertoire où le fichier sera stocké.
     * 
     * @return string
     */
    public function add(UploadedFile $file, string $path): string
    {
        return $this->upload($file, $path);
    }

    /**
     * Si le fichier n'est pas nul, le télécharge et supprimez l'ancien fichier.
     * Si le fichier est nul, renvoyez l'ancien nom de fichier.
     * 
     * @param UploadedFile|null $file Le fichier qui a été téléchargé.
     * @param string $fileName Le nom du fichier à mettre à jour.
     * @param string $path Le chemin d'accès au répertoire où le fichier sera stocké.
     * 
     * @return string
     */
    public function update(
        ?UploadedFile $file,
        string $fileName,
        string $path
    ): string {
        if ($file !== null) {
            $name = $this->upload($file, $path);
            $this->remove($fileName, $path);
            return $name;
        } else {
            return $fileName;
        }
    }

    /**
     * Il supprime un fichier du système.
     * 
     * @param string $fileName Le nom du fichier à supprimer.
     * @param string $path Le chemin d'accès au fichier.
     */
    public function remove(string $fileName, string $path): void
    {
        $this->fileSystem->remove($this->targetDirectory . $path . $fileName);
    }

    /* -------------------------------------------------------------------------- */
    /*                            Les méthodes Privées                            */
    /* -------------------------------------------------------------------------- */

    /**
     * Il prend un fichier et un chemin, et renvoie un nom de fichier.
     * 
     * @param UploadedFile $file Le fichier à télécharger.
     * @param string $path Le chemin d'accès au répertoire où le fichier sera téléchargé.
     * 
     * @return string
     */
    private function upload(UploadedFile $file, string $path): string
    {
        $originalFilename = pathinfo($file->getClientOriginalName(), PATHINFO_FILENAME);
        $safeFilename = $this->slugger->slug($originalFilename);
        $fileName = $safeFilename . '-' . uniqid() . '.' . $file->guessExtension();
        $file->move($this->targetDirectory . $path, $fileName);
        return $fileName;
    }
}
