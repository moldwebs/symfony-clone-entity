<?php

namespace App\AdminBundle\Services;

use Symfony\Component\HttpFoundation\File\File;

class CloneEntity
{
    public static function createClone($entity, $em)
    {
        $clone = clone $entity;

        $metadata = $em->getClassMetadata(get_class($clone));
        foreach ($metadata->reflClass->getProperties() as $property) {
            $docComment = $property->getDocComment();
            if (strpos($docComment, '@Vich\UploadableField') !== false && preg_match('/fileNameProperty="(.*?)"/', $docComment, $match)) {

                $fileNameProperty = $match[1];

                /** @var File $file */
                $file = $clone->{'get'.$property->name}();
                if($file instanceof File){
                    if(file_exists($file->getRealPath())){
                        $uniqId = md5(uniqid());
                        $newFileName = $uniqId . '.' . $file->getExtension();
                        copy($file->getRealPath(), $file->getPath() . DIRECTORY_SEPARATOR . $newFileName);
                        $clone->{'set'.$fileNameProperty}($newFileName);
                    } else {
                        $clone->{'set'.$fileNameProperty}(null);
                    }
                    $clone->{'set'.$property->name}(null);
                }

            } elseif (strpos($docComment, 'ArrayCollection') !== false) {
                $_arrayCollection = $clone->{'get'.$property->name}();
                foreach ($_arrayCollection as $_key => $_entity) {
                    $_arrayCollection[$_key] = self::createClone($_entity, $em);
                }
                $property->setAccessible(true);
                $property->setValue($clone, $_arrayCollection);
            }

        }

        return $clone;
    }
}
