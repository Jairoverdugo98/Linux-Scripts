## Descripción

En este Scrip se plantea un shell script que recibe como parámetro el nombre base para crear usuarios, el número de usuarios a generar y el grupo al que han de pertenecer. 
El número de usuarios es opcional siendo por defecto 5. 
El grupo al que han de pertenecer es opcional, siendo por defecto el nombre base que se da para generar los usuarios.

## Código Del Script
#!/bin/bash
# Author: Jairo Verdugo
# Version: 1.0
# Descripción: Creacion de usuarios en Linux.
#Funciones
mensaje_uso(){
    echo "ERROR: Número de parámetros incorrecto." >&2
    echo "USO: $0 <nombre_base> [-n <numero_de_usuarios>] [-g <nombre_de_grupo>]" >&2
    exit $salida  
}

crea_grupo(){
    sentencia_crea_grupo="groupadd $nombre_grupo"
    $sentencia_crea_grupo 2> /dev/null #Supuesto de error es porque ya existe
}

crea_usuario(){
    sentencia_crea_usuario="useradd -g $nombre_grupo $nombre_usuario"
    $sentencia_crea_usuario 2> /dev/null #Supuesto de error es porque ya existe
}

asigna_grupo(){
    sentencia_cambio_grupo="adduser $nombre_usuario $nombre_grupo"
    $sentencia_cambio_grupo 1>&2> /dev/null #Nos aseguramos de que formen parte del grupo exigido
}

asigna_passwd(){
    echo $nombre_usuario:$nombre_base | chpasswd    
}

#Variables
ENE="-n"
GE="-g"
nombre_grupo=$1
nombre_base=$1
numero_usuarios=5
salida=1

# Control simple de argumentos
case "$#" in
    1)  case $1 in
            -h | --help | -help )
            salida=0 
            mensaje_uso 
            ;;
        esac
        #Se mantiene los valores por defecto
    ;;    
    3)
        if [ "$2" = "$ENE" ]; then
            numero_usuarios=$3
        else
            if [ "$2" = "$GE" ]; then
            nombre_grupo=$3
            else
                mensaje_uso
            fi
        fi
    ;;
    5)
        if [ "$2" = "$ENE" ]; then
            if [ "$4" = "$GE" ]; then
                numero_usuarios=$3
                nombre_grupo=$5
            else
                mensaje_uso
            fi
        else
            mensaje_uso
        fi
    ;;
    *) mensaje_uso
    ;;
esac

#Bloque Principal
crea_grupo
 
for (( indice=1; indice <= $numero_usuarios; indice++ ))
    do
    nombre_usuario=$nombre_base$indice
    crea_usuario
    asigna_grupo
    asigna_passwd
done

exit 0
