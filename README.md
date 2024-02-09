# Demo Sylius para ITBN

Esta demo parte del template original [Sylius
Standard](https://github.com/Sylius/Sylius-Standard). Al iniciar el ejemplo que
por defecto parte de la versión 1.12 de sylius (porque la 1.13 está en alpha
aún), nos encontramos con una serie de cambioa que se debieron realizar y que
deben considerarse antes de pasar a producción.

## `composer.lock` no se versiona

Al ver el proyecto template original, vemos que propone versionar
`composer.lock`. Sin embargo, en el [ticket
665](https://github.com/Sylius/Sylius-Standard/issues/665) y con el [PR
855](https://github.com/Sylius/Sylius-Standard/pull/855) se da solución a este
problema, cuando se trabaja con:

```bash
 php composer.phar create-project sylius/sylius-standard project
```

Sin embargo, si se sigue la [documentación para desarrollar con
docker](https://docs.sylius.com/en/1.12/book/installation/installation_with_docker.html),
este paso  es omitido, y el archivo de locks queda fuera del versionado.

Nuestro criterio, basado en [la documentación de
composer](https://getcomposer.org/doc/01-basic-usage.md#commit-your-composer-lock-file-to-version-control)
es la de **versionar este archivo**. Por esta razón, manualmente cambiamos el
`.gitignore` y aprovechamos para crear el [ticket
947](https://github.com/Sylius/Sylius-Standard/issues/947) para proponer una
solución al tema.

## Dockerfile

Modificamos únicamnte el agregado de una variable en el multistage `base` para
que composer pueda usar más de 1.5GB de RAM (valor limitado por defecto), usando
la variable `COMPOSER_MEMORY_LIMIT=-1`.

## Uso de direnv mediante `.envrc`

Proponemos un refactor del docker-compose para ser usado en linux, y considerar
que el usuario usado por el contenedor coincida con el del user que corre
docker. Para ello, podemo usar o no [direnv](https://direnv.net/).

### Uso con direnv

Setea automáticamente el valor de `DOCKER_USER` a partir de `.envrc`, por tanto,
al ingresar al directorio del proyecto, y habiendo **permitido a direnv setear
esta variable**, el valor es seteado automáticamente. Podemos chequear si el
valor está seteado con:

```bash
echo $DOCKER_USER
```
> Debe recibir algún valor entero

### Uso sin direnv

Debe setear esta variable como lo hace el archivo `.envrc`. Para ello, podemos
en una sesión del shell usar:

```bash
source .envrc
```

Y verificar como en el paso antes mencionado

> Es importante destacar que el comando `source` funciona en la misma sesión
> donde se corrió. Si inicia una nueva solapa o terminal, debe correr el mismo
> comando cada vez.

## docker-compose adaptado

El `docker-compose.yml` provisto por el repositorio template, parte
considerando que el desarrolaldor usará Mac / Windows / Linux. Por tanto,
algunos volúmenes están comentados para explicar como se usan en cada SO. Hemos
modifcado estos volúmenes para que funcionen directamente en Linux.

Además, en los contenedores de **php** y **node** hemos agregado la instrucción:

```yaml
...
  php:
    ... 
    user: $DOCKER_USER
    ...
  node:
    ...
    user: $DOCKER_USER
    ...
```

Con el objeto que corran con el usuario del sistema y no cree archivos como root
en el directorio local.

# Iniciando el stack

Simplemente correr:

```bash
docker-compose build
docker-compose up
```

Una vez todo levantado, podemos ingresar al navegador usando:

```bash
xdg-open http://localhost
```

El usuario es **sylius** con igual password

## Instalar algun plugin

Si queremos instalar algo nuevo, podemos modificar `composer.json` y luego
correr `composer instalall` o `composer update`. Pero estas tareas deben
correrse dentro del contenedor:

```bash
docker-compose exec php ash
```
> Nos permite ingresar al contenedor de php

Luego, corremos el comando necearios:

```bash
composer up
```

De igual forma, podemos usar la consola de symfony o correr aquellos comandos
que necesitemos de node, usando el contenedor de nodejs.

