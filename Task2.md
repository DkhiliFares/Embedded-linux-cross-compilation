# Tache 2 : Construire une chaîne d'outils de compilation croisée #
1. Descriptions :
Les outils de développement habituels disponibles sur une station de travail GNU/Linux
constituent une chaîne de compilation native.
Cette chaîne de compilation fonctionne sur votre station de travail et génère du code pour
celle-ci, généralement pour l'architecture x86.
Pour le développement de systèmes embarqués, il est généralement impossible ou peu
intéressant d'utiliser une chaîne de compilation native.
► La cible est trop limitée en termes de stockage et/ou de mémoire.
►La cible est très lente comparée à votre station de travail.
►Vous ne souhaitez peut-être pas installer tous les outils de développement sur votre
cible.
C'est pourquoi les chaînes de compilation croisées sont généralement utilisées. Elles
s'exécutent sur votre station de travail mais génèrent du code pour votre cible.
---
![alt text](images/image2-1.png)
---

2. Composants d'une Chaîne de Compilation Croisée:

---
![alt text](images/image2-2.png)
---
• Binutils : Un ensemble d'outils pour la manipulation des fichiers binaires, incluant des utilitaires comme l'assembleur et l'éditeur de liens essentiels pour la gestion des fichiers objets.

• En-têtes du noyau : Fichiers indispensables pour établir l'interface entre les programmes utilisateur et le noyau, garantissant la compatibilité avec le système cible.

• Bibliothèques C/C++ : Les bibliothèques standards, telles que libc pour le C ou la bibliothèque standard C++, qui fournissent les fonctions de base nécessaires à l'exécution des programmes.

• Compilateur C/C++ : Le compilateur chargé de convertir le code source en C/C++ en exécutables binaires pour une architecture cible spécifique.

• Débogueur GDB (optionnel) : Un outil de débogage permettant d'analyser et de corriger les programmes compilés particulièrement utile pour le débogage sur la cible.

3. Objectifs spécifiques :

• ► Être capable de comprendre les fondements de la compilation croisée.

• ► Être capable d'installer et de configurer Crosstool-NG.

• ► Être capable de générer un compilateur croisé pour Raspberry Pi 3 (architecture aarch64).

• ► Être capable de configurer l'environnement d'exécution pour le compilateur croisé.

4. Étapes à suivre :
A.Installer les dépendances
```bash
sudo apt install build-essential git autoconf bison flex texinfo help2man gawk libtool-bin \
 libncurses5-dev unzip gettext python3
```
B. Installer et configurer Crosstool-NG
---
* Getting Crosstool-ng
```bash
git clone https://github.com/crosstool-ng/crosstool-ng
cd crosstool-ng/
git checkout crosstool-ng-1.28.0
```
---
* Building and installing Crosstool-ng
Comme nous ne construisons pas Crosstool-ng à partir d'une archive de version, mais à partir d'un dépôt git, nous devons d'abord générer un script configure et, plus généralement, tous les fichiers générés qui sont fournis dans l'archive source d'une version :
```bash
./bootstrap
```
Nous pouvons ensuite soit installer Crosstool-ng globalement sur le système, soit le garder localement dans son répertoire de téléchargement. Nous choisirons la dernière solution. Comme documenté à https://crosstool-ng.github.io/docs/install/#hackers-way

```bash
./configure --enable-local
make
./ct-ng help
```
C. Configurer Crosstool-NG pour Raspberry Pi 3

* Configurer la chaîne d'outils pour produire

Une seule installation de Crosstool-ng permet de produire autant de chaînes d'outils que vous le souhaitez, pour différentes architectures, avec différentes bibliothèques C et différentes versions des divers composants.

Crosstool-ng est livré avec un ensemble de fichiers de configuration préfabriqués pour différentes configurations typiques : Crosstool-ng les appelle des échantillons. Ils peuvent être listés en utilisant 
```bash 
./ct-ng list-samples 
```

Nous allons charger l'échantillon Cortex A8. Chargez-le avec la commande 
```bash 
./ct-ng  arm-cortex_a8-linux-gnueabi
```
Ensuite, pour affiner la configuration, lançons l'interface menuconfig :
```bash 
./ct-ng  menuconfig
```
Dans Path and misc options :

Si ce n’est pas encore fait, activez Try features marked as EXPERIMENTAL.

Dans Target options :

Définissez Use specific FPU (ARCH_FPU) sur vfpv3.

Définissez Floating point sur hardware (FPU).

Dans Toolchain options :

Définissez Tuple's vendor string (TARGET_VENDOR) sur training.

Définissez Tuple's alias (TARGET_ALIAS) sur arm-linux. Ainsi, nous pourrons utiliser le compilateur avec arm-linux-gcc, un nom plus court que celui basé sur le tuple complet de la toolchain.

Dans Operating System :

Définissez Version of linux sur la version la plus proche mais antérieure à 6.6. Il est important que les headers du noyau utilisés dans la toolchain ne soient pas plus récents que le noyau qui sera exécuté sur la carte (v6.6).

Dans C-library :

Si ce n’est pas encore fait, définissez C library sur musl (LIBC_MUSL).

Conservez la version par défaut proposée.

Dans C compiler :

Définissez Version of gcc sur 14.2.0.

Assurez-vous que C++ (CC_LANG_CXX) est activé.

Dans Debug facilities :

Supprimez toutes les options ici. Certains outils de débogage peuvent être fournis dans la toolchain, mais ils peuvent également être construits via des outils de construction du système de fichiers.
 Produce the toolchain
 Nothing is simpler:
 ```bash
 $ ./ct-ng build
 ```
 The toolchain will be installed by default in $HOME/x-tools/. That’s something you could have changed in
 Crosstool-ng’s configuration.

D. Installer QEMU

vous pouvez quand même exécuter ce binaire depuis votre machine hôte x86 ?
Pour cela, installez l’émulateur utilisateur QEMU, qui n’émule que le jeu d’instructions cible, et non tout un système avec ses périphériques :
```bash
sudo apt install qemu-user
 ```
E. Compiler et tester un programme:

Vous pouvez maintenant tester votre toolchain en ajoutant $HOME/x-tools/arm-training-linux-musleabihf/bin/ à votre variable d’environnement PATH et en compilant le simple programme hello.c dans votre répertoire principal du labo avec arm-linux-gcc :
```bash
arm-linux-gcc -o hello hello.c
 ```
 Vous pouvez utiliser la commande file sur votre binaire pour vérifier qu’il a bien été compilé pour l’architecture ARM.
```bash
file hello
 --- 
hello: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-armhf.so.1, not stripped
---
```
Ensuite, essayez de lancer l’émulateur utilisateur QEMU pour ARM :
```bash
qemu-arm hello
```

Vous obtiendrez probablement une erreur :

qemu-arm: Could not open '/lib/ld-musl-armhf.so.1': No such file or directory
---

Ce qui se passe, c’est que qemu-arm ne trouve pas le chargeur de bibliothèques partagées (compilé pour ARM) dont dépend ce binaire.

Localiser le chargeur de bibliothèques dans la toolchain
```bash
find ~/x-tools-name -name ld-musl-armhf.so.1
```


Vous devriez obtenir quelque chose comme :

---
/home/tux/x-tools/arm-training-linux-musleabihf/arm-training-linux-musleabihf/sysroot/lib/ld-musl-armhf.so.1

---
Indiquer à QEMU où trouver les bibliothèques

Vous pouvez maintenant utiliser l’option -L de qemu-arm pour lui indiquer où se trouvent les bibliothèques partagées :
```bash
qemu-arm -L ~/x-tools/arm-training-linux-musleabihf/arm-training-linux-musleabihf/sysroot hello
```
on obtient:

---
Hello, ARM world!
---
5. Documentation:
Pour plus d'informations détaillées sur chaque étape et les outils utilisés, vous pouvez vous
référer à la documentation Bootlin. Bootlin propose des ressources et formations complètes
sur la compilation croisée, l'utilisation de Crosstool-NG et d'autres outils pour le
développement sur des architectures embarquées.

. Bootlin Documentation : https://bootlin.com/doc/training/embedded-linux-bbb/embedded-linux-bbb-labs.pdf