# Setup TPs Spark MS BGD (2018-2019)

<!-- TOC -->

- [Setup TPs Spark MS BGD (2018-2019)](#setup-tps-spark-ms-bgd-2018-2019)
    - [Setup TP 1: Installation de Spark](#setup-tp-1-installation-de-spark)
        - [Installation de java](#installation-de-java)
            - [Sur les machines de TP](#sur-les-machines-de-tp)
            - [Linux (Ubuntu)](#linux-ubuntu)
            - [Linux (CentOS)](#linux-centos)
            - [Mac](#mac)
            - [Windows](#windows)
        - [Installation de Spark](#installation-de-spark)
            - [Utiliser le Spark-shell](#utiliser-le-spark-shell)
            - [Réduire la quantité des logs affichés par Spark](#réduire-la-quantité-des-logs-affichés-par-spark)
    - [Setup TP 2: Installation de SBT, IntelliJ et démarrage du projet](#setup-tp-2-installation-de-sbt-intellij-et-démarrage-du-projet)
        - [Installation de SBT](#installation-de-sbt)
            - [Sur les machines de TP](#sur-les-machines-de-tp-1)
            - [Sur Ubuntu (machine perso : besoin des droits root)](#sur-ubuntu-machine-perso--besoin-des-droits-root)
            - [Sur Mac](#sur-mac)
        - [Installation d'IntelliJ](#installation-dintellij)
            - [Sur les machines de TP](#sur-les-machines-de-tp-2)
            - [Sur Ubuntu (machines perso : besoin des droits root)](#sur-ubuntu-machines-perso--besoin-des-droits-root)
            - [Sur Mac](#sur-mac-1)
        - [Importer le projet (voir TP 2 pour télécharger le template de projet) dans IntelliJ](#importer-le-projet-voir-tp-2-pour-télécharger-le-template-de-projet-dans-intellij)
    - [HOW TO: lancer un job Spark](#how-to-lancer-un-job-spark)
        - [Compiler et construire le jar](#compiler-et-construire-le-jar)
        - [Démarrer un cluster Spark local (le driver et le worker seront sur la même machine)](#démarrer-un-cluster-spark-local-le-driver-et-le-worker-seront-sur-la-même-machine)
        - [Soumettre un Job à Spark](#soumettre-un-job-à-spark)

<!-- /TOC -->

## Setup TP 1: Installation de Spark

Spark utilise des machines virtuelles java, il faut donc commencer par l'installer.

NB : **Il faut installer java 1.8, les versions suivantes ne fonctionneront pas**.

### Installation de java

Selon votre machine (mac, Linux, machine de TP) reportez-vous à la section correspondante dans la suite.

#### Sur les machines de TP

Dans un terminal, entrez :
```
java -version
```

Cette commande affiche la version de java qui est installée : la version 1.7 ou 1.8 doit déjà être installée.

Il faut désormais setter notre java home. Dans un terminal :
```
update-alternatives --config java
```

Le path dans la colonne "Commande" est ce qui nous intéresse (en enlevant la partie */bin/java* à la fin). Ajouter ensuite la ligne suivante dans le fichier *.bash_profile* situé dans notre *home* :
```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b10-0.amzn2.0.1.x86_64/jre
```
(utiliser le path qui s'affiche dans votre terminal et qui n'est peut-être pas le même).

#### Linux (Ubuntu)

Vous devez avoir les droits root.

Option 1 : Dans un terminal

```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

NB : si vous n'arrivez pas à installer java 8 de cette façon, testez
```
sudo apt update
sudo apt install openjdk-8-jdk openjdk-8-jre
```

Si vous avez installé java de la seconde façon, il faut ajouter les lignes suivantes dans le fichier *.bash_profile* situé dans notre *home* :
```
export JAVA_HOME= /usr/lib/jvm/java-8-openjdk-amd64
export JRE_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre
```

Dans le cas contraire, setter uniquement la java home en suivant les explications plus bas.

Option 2 : Installation manuelle

Allez sur http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html. Dans *Java SE Development Kit 8u181* choisissez la version à dowloader qui vous correspond. Puis installer le paquet downloadé.

Il faut désormais setter notre java home. Dans un terminal :
```
update-alternatives --config java
```

Le path dans la colonne "Commande" est ce qui nous intéresse (en enlevant la partie */bin/java* à la fin). Ajouter ensuite la ligne suivante dans le fichier *.bash_profile* situé dans notre *home* :
```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b10-0.amzn2.0.1.x86_64/jre
```
(utiliser le path qui s'affiche dans votre terminal et qui n'est peut-être pas le même).

#### Linux (CentOS)

Dans un terminal :
```
sudo yum -y update
sudo yum -y install java-1.8.0-openjdk
update-alternatives --config java
```

Le path dans la colonne "Commande" est ce qui nous intéresse (en enlevant la partie */bin/java* à la fin). Ajouter ensuite la ligne suivante dans le fichier *.bash_profile* situé dans notre *home* :
```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b10-0.amzn2.0.1.x86_64/jre
```
(utiliser le path qui s'affiche dans votre terminal et qui n'est peut-être pas le même).

#### Mac

Option 1 : Dans un terminal

De façon générale, pour installer n'importe quel software via le terminal avec un Mac, il est préférable de passer par [*Brew*](https://brew.sh/) :
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew tap caskroom/cask
```

Pour installer java 1.8 :
```
brew tap adoptopenjdk/openjdk
brew cask install adoptopenjdk8
```

Option 2 : Installation manuelle

Allez sur http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html. Dans *Java SE Development Kit 8u181*, télécharger le fichier *.dmg* pour mac, puis l’installer.

Si la version de java n’est pas la bonne après installation (`java -version`). Dans le fichier .bash_profile ajouter la ligne :
```
export JAVA_HOME=`/usr/libexec/java_home -v 1.8`
```

Puis fermer et réouvrir le terminal pour que la modification soit effective.

#### Windows

Pour installer java 1.8 et Spark, suivre [cet article](https://medium.com/big-data-engineering/how-to-install-apache-spark-2-x-in-your-pc-e2047246ffc3).

### Installation de Spark

Option 1 : Dans le terminal

Allez dans la home, téléchargez spark, puis décompressez-le :
```
cd
wget http://ftp.tudelft.nl/apache/spark/spark-2.3.4/spark-2.3.4-bin-hadoop2.7.tgz
tar -xvzf spark-2.3.4-bin-hadoop2.7.tgz
```

Option 2 : Installation manuelle

Aller sur http://spark.apache.org/downloads.html puis :
- Spark release : 2.3.4
- Package type : pre-built for apache hadoop 2.7 and later
- cliquer sur le lien : spark-2.3.4-bin-hadoop2.7.tgz

Une fois téléchargé, copier le fichier *.tgz* dans votre répertoire *home* (dans un terminal entrez: `echo $HOME` pour savoir où est votre *home*). Puis décompresser le fichier *.tgz* via un `tar -xvzf <spark>` (remplacer `<spark>` par le bon nom).

#### Utiliser le Spark-shell

Dans le terminal, aller dans le répertoire où spark est installé, puis dans le répertoire *bin*, et lancer le *spark-shell*. Par exemple :
```
cd spark-2.3.4-bin-hadoop2.7/bin
./spark-shell
```

L’interface utilisateur est alors disponible dans un navigateur à l’adresse *localhost:4040*.

#### Réduire la quantité des logs affichés par Spark

Il faut tout d'abord copier le fichier de configuration des logs par défaut *log4j.properties.template* dans *log4j.properties*.

Dans un terminal :
```
cd spark-2.3.4-bin-hadoop2.7/conf
cp log4j.properties.template log4j.properties
```

Ouvrez le fichier *log4j.properties* dans un éditeur de texte et remplacez la ligne
```
log4j.rootCategory=INFO, console
```
par
```
log4j.rootCategory=WARN, console
```

## Setup TP 2: Installation de SBT, IntelliJ et démarrage du projet

Cette section est à faire pour le TP2 ! 

Selon votre machine (mac, Linux perso, machine de TP) reportez-vous aux sections correspondantes.

### Installation de SBT

#### Sur les machines de TP

SBT est déjà installé. Pour l’utiliser, ouvrir un terminal et faire :
```
sbt
```

#### Sur Ubuntu (machine perso : besoin des droits root)

Dans un terminal :
```
echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
sudo apt-get update
sudo apt-get install sbt
```

#### Sur Mac

Installer brew si ce n’est pas déjà fait : suivre les consignes sur https://brew.sh/index_fr.

Dans un terminal :
```
brew install sbt
```

### Installation d'IntelliJ

#### Sur les machines de TP

IntelliJ 2016 nécessite *java 1.8* (qui n’est pas installé sur les machines de TP).  
Donc télécharger la version 15 de IntelliJ Community pour linux ideaIC-15.0.6.tar.gz :
https://www.jetbrains.com/idea/download/other.html.

Décompresser IntelliJ via `tar -xvzf <idea>` (remplacer `<idea>` par le nom que vous voyez dans votre terminal).

Dans un terminal :
```
cd <idea>/bin
chmod +x idea.sh
./idea.sh
```
(encore une fois, remplacer `<idea>` par le bon nom).

Dans la fenêtre qui s’ouvre:
- I do not have previous installation => click OK
- Choisir un thème => click next
- create desktop entry : déselectionner "for all users" => click next
- tune Idea to your task : ne rien faire => click next
- scala : cliquer sur "Install" => start using IntelliJ

Une fois IntelliJ installé, aller dans les "préférences" puis dans "plugins". Dans la barre de recherche de plugins, chercher "scala" et installer le plugins (s'il ne l’est pas déjà).

#### Sur Ubuntu (machines perso : besoin des droits root)

Download : https://www.jetbrains.com/idea/.

Décompresser IntelliJ via `tar -xvzf <idea>` (remplacer `<idea>` par le nom que vous voyez dans votre terminal).

Dans un terminal :
```
cd <idea>/bin
./idea.sh
```
(encore une fois, remplacer `<idea>` par le bon nom).

Dans la fenêtre qui s’ouvre :
- I do not have previous installation => click OK
- Choisir un thème => click next
- create desktop entry => click next
- launcher script : ne rien faire => click next
- tune Idea to your task : ne rien faire => click next
- scala : cliquer sur "Install" => start using IntelliJ

Une fois IntelliJ installé, aller dans les "préférences" puis dans "plugins". Dans la barre de recherche de plugins, chercher "scala" et installer le plugins (s'il ne l’est pas déjà).

#### Sur Mac

Option 1 : Dans le terminal

```
brew cask install intellij-idea-ce
```

Option 2 : Installation manuelle

Download : https://www.jetbrains.com/idea/.

Télécharger le fichier *.dmg*, l’installer.

Lancer IntelliJ, dans la fenêtre qui s’ouvre faire:
- I do not have previous installation => click OK
- Choisir un thème => click next
- create desktop entry => click next
- laucher script : ne rien faire => click next
- tune Idea to your task : ne rien faire => click next
- scala : cliquer sur "Install" => start using IntelliJ

Une fois IntelliJ installé, aller dans les "préférences" puis dans "plugins". Dans la barre de recherche de plugins, chercher "scala" et installer le plugins (s'il ne l’est pas déjà)

### Importer le projet (voir TP 2 pour télécharger le template de projet) dans IntelliJ

Pour importer le projet, suivre la partie [Téléchargement du projet](TP_2_projet_et_pre_processings.md#téléchargement-du-projet) dans le TP 2.

Ouvrir IntelliJ puis :
- Import project
- Import project from external model, et choisir SBT
- Sélectionner le chemin vers le projet décompressé
- Sélectionner "use auto import" / project SDK cliquer sur "new" puis "JDK" sélectionner "java-8-oracle" dans l’arborescence / cliquer sur Finish.
- sbt data project to import, ne rien faire, cliquer sur OK
- Attendre

## HOW TO: lancer un job Spark

### Compiler et construire le jar

Dans un terminal :
```
cd <chemin/du/projet> # aller là où se trouve le fichier build.sbt du projet
sbt assembly
```

L’adresse du jar est donnée vers la fin du script :
```
[info] Packaging /Users/flo/Documents/github/spark_project_kickstarter_2019_2020/target/scala-2.11/spark_project_kickstarter_2019_2020-assembly-1.0.jar
```

### Démarrer un cluster Spark local (le driver et le worker seront sur la même machine)

Dans un terminal :
```
cd spark-2.3.4-bin-hadoop2.7/sbin # attention c’est bien "sbin"
./start-all.sh
```

NB : S'il y a une erreur *port 22 connection refused*, c’est que le worker ne trouve pas l’adresse du master, ils ne peuvent donc pas communiquer. Pour démarrer le cluster il faut alors taper dans le terminal et dans le dossier sbin) :
```
./start-master.sh
```

Allez à l’adresse *localhost:8080* dans un navigateur, repérez l’adresse en gras tout en haut (*spark://adresse_du_master:7077*). Notez qu’il n’y a pas de worker indiqué sous worker Id. Puis retournez dans le terminal et faîtes :
```
./start-slave.sh adresse_du_master:7077
```

Il devrait maintenant y avoir un worker indiqué sous worker Id !

Dans chrome, firefox, etc., aller à l’adresse *localhost:8080*. L’Interface Utilisateur (Spark UI) s’affiche si spark a bien démarré.

### Soumettre un Job à Spark

Soumettre le jar du script qui a été compilé:

Dans un terminal :
```
cd spark-2.3.4-bin-hadoop2.7/bin # !!!! Attention c’est bien "bin" maintenant

./spark-submit \
--driver-memory 3G \
--executor-memory 4G \
--class paristech.Job \
--master spark://$(hostname -i):7077 \
/Users/flo/Documents/github/spark_project_kickstarter_2019_2020/target/scala-2.11/spark_project_kickstarter_2019_2020-assembly-1.0.jar
```

NB : remplacer le chemin vers le jar, remplacer le nom de la classe s'il a été modifié par rapport au template donné en début de TP.

Un exemple de commande pour, en plus, sauvegarder les event logs et pouvoir lire et écrire depuis/sur AWS :
```
./spark-submit \
--conf spark.eventLog.enabled=true \
--conf spark.eventLog.dir="/tmp" \
--driver-memory 3G \
--executor-memory 4G \
--class paristech.Job \
--num-executors 2 \
--packages "com.amazonaws:aws-java-sdk:1.7.4,org.apache.hadoop:hadoop-aws:2.7.1" \
--master spark://$(hostname -i):7077 \
/Users/flo/Documents/github/spark_project_kickstarter_2019_2020/target/scala-2.11/spark_project_kickstarter_2019_2020-assembly-1.0.jar
```
