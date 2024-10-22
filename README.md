
# Rapport de TP : Shell Bash

## 1. Exercice : Paramètres
Le script `analyse.sh` affiche des informations sur les paramètres passés.

```bash
#!/bin/bash

echo "Bonjour, vous avez rentré $# paramètres."
echo "Le nom du script est $0"
if [ $# -ge 3 ]; then
    echo "Le 3ème paramètre est $3"
else
    echo "Il n'y a pas de 3ème paramètre."
fi
echo "Voici la liste des paramètres : $@"
```

---

## 2. Exercice : Vérification du nombre de paramètres
Le script `concat.sh` concatène deux mots passés en paramètres, ou affiche une erreur s'il y a plus ou moins de deux paramètres.

```bash
#!/bin/bash

if [ $# -ne 2 ]; then
    echo "Erreur : vous devez rentrer exactement 2 paramètres."
    exit 1
fi

CONCAT="$1$2"
echo "La concaténation des deux mots est : $CONCAT"
```

---

## 3. Exercice : Argument type et droits
Le script `test-fichier.sh` vérifie le type d’un fichier, ses permissions, et s’il existe.

```bash
#!/bin/bash

if [ $# -ne 1 ]; then
    echo "Erreur : vous devez fournir un fichier en paramètre."
    exit 1
fi

FICHIER=$1

if [ ! -e "$FICHIER" ]; then
    echo "Le fichier $FICHIER n'existe pas."
    exit 1
fi

if [ -d "$FICHIER" ]; then
    echo "Le fichier $FICHIER est un répertoire."
elif [ -f "$FICHIER" ]; then
    if [ -s "$FICHIER" ]; then
        echo "Le fichier $FICHIER est un fichier ordinaire qui n'est pas vide."
    else
        echo "Le fichier $FICHIER est un fichier ordinaire qui est vide."
    fi
fi

if [ -r "$FICHIER" ]; then
    echo ""$FICHIER" est accessible en lecture."
fi

if [ -w "$FICHIER" ]; then
    echo ""$FICHIER" est accessible en écriture."
fi

if [ -x "$FICHIER" ]; then
    echo ""$FICHIER" est accessible en exécution."
fi
```

---

## 4. Exercice : Afficher le contenu d’un répertoire
Le script `listdir.sh` affiche le contenu d'un répertoire, en séparant les fichiers des répertoires.

```bash
#!/bin/bash

if [ $# -ne 1 ]; then
    echo "Erreur : vous devez fournir un répertoire en paramètre."
    exit 1
fi

DIR=$1

if [ ! -d "$DIR" ];then
    echo "Le chemin $DIR n'est pas un répertoire."
    exit 1
fi

echo "####### fichiers dans $DIR/"
for file in "$DIR"/*; do
    if [ -f "$file" ]; then
        echo "$file"
    fi
done

echo "####### répertoires dans $DIR/"
for file in "$DIR"/*; do
    if [ -d "$file" ]; then
        echo "$file"
    fi
done
```

---

## 5. Écrire un script bash affichant la liste des noms de login des utilisateurs définis dans `/etc/passwd` ayant un UID supérieur à 100
Ce script utilise `awk` pour afficher les utilisateurs ayant un UID supérieur à 100.

```bash
#!/bin/bash

awk -F : '{
    if($3 >= 100)
        print "Utilisateur: " $1 ", UID: " $3;
}' /etc/passwd
```

---

## 6. Mon utilisateur existe t-il ?

Ce script vérifie si un utilisateur existe deja en fonction d’un login passé en paramètres, en fonction d’un UID passé en paramètres
Si l’utilisateur existe le script va renvoyer son UID à l’affichage sinon il ne renverra rien.

```bash
#!/bin/bash

# Vérification de la présence d'un paramètre
if [ $# -ne 2 ]; then
    echo "Usage: $0 [login|uid] <valeur>"
    exit 1
fi

type=$1
valeur=$2

if [ "$type" == "login" ]; then
    # Vérification de l'existence de l'utilisateur par login
    if id "$valeur" &>/dev/null; then
        echo "L'utilisateur $valeur existe avec l'UID : $(id -u "$valeur")"
    fi
elif [ "$type" == "uid" ]; then
    # Vérification de l'existence de l'utilisateur par UID
    if getent passwd "$valeur" &>/dev/null; then
        echo "L'utilisateur avec l'UID $valeur existe."
    fi
else
    echo "Type non reconnu. Utilisez 'login' ou 'uid'."
    exit 1
fi
```
---

## 7. Creation utilisateur

Le script crée un nouvel utilisateur en demandant des informations (login, nom, prénom, UID, GID, commentaires), en vérifiant que l'utilisateur est root et que l'utilisateur n'existe pas déjà, puis utilise `useradd` pour effectuer la création.

Pour lire une chaîne au clavier et l'affecter à une variable, j'ai utilisé la commande `read` de la manière suivante :

```bash
#!/bin/bash

# Vérification que l'utilisateur est root
if [ "$USER" != "root" ]; then
    echo "Erreur : Vous devez être root pour exécuter ce script."
    exit 1
fi

# Demande des informations à l'utilisateur
read -p "Entrez le login : " login
read -p "Entrez le nom : " nom
read -p "Entrez le prénom : " prenom
read -p "Entrez l'UID : " uid
read -p "Entrez le GID : " gid
read -p "Entrez des commentaires : " commentaires

# Vérification de l'existence de l'utilisateur
if id "$login" &>/dev/null; then
    echo "L'utilisateur $login existe déjà."
    exit 1
fi

# Création de l'utilisateur
useradd -m -u "$uid" -g "$gid" -c "$commentaires" "$login"

if [ $? -eq 0 ]; then
    echo "L'utilisateur $login a été créé avec succès."
else
    echo "Erreur lors de la création de l'utilisateur $login."
fi
```

---

## 8. Utiliser la commande `file`
La commande `file` permet d'afficher des informations sur le contenu d’un fichier. Voici les commandes à utiliser :

- Quitter `more` : Appuyer sur `q`.
- Avancer d’une ligne : Appuyer sur `Entrée`.
- Avancer d’une page : Appuyer sur `Espace`.
- Remonter d’une page : Appuyer sur `b`.
- Chercher une chaîne de caractères : Appuyer sur `/` suivi de la chaîne, puis appuyer sur `Entrée`. Pour passer à l’occurrence suivante, appuyer sur `n`.

Ce script Bash permet de visualiser page par page chaque fichier texte d'un répertoire spécifié par l'utilisateur. Il utilise la commande `file` pour vérifier si chaque fichier est un fichier texte, puis demande à l'utilisateur s'il souhaite visualiser ce fichier à l'aide de la commande `more`.

```bash
#!/bin/bash

# Vérification si un répertoire est passé en argument
if [ -z "$1" ]; then
    echo "Erreur : Veuillez spécifier un répertoire."
    exit 1
fi

# Parcourir tous les fichiers dans le répertoire
for fichier in "$1"/*; do
    # Vérifier si c'est un fichier texte
    if file "$fichier" | grep -q "text"; then
        # Demander à l'utilisateur s'il veut visualiser le fichier
        echo -n "Voulez-vous visualiser le fichier $fichier ? (o/n) : "
        read reponse

        if [ "$reponse" = "o" ]; then
            # Afficher le fichier avec more
            more "$fichier"
        fi
    fi
done


---

## 8. Écrire un script pour visualiser page par page chaque fichier texte
Voici un script qui permet de visualiser chaque fichier texte d'un répertoire page par page :

```bash
#!/bin/bash

if [ $# -eq 0 ]; then
    echo "Veuillez spécifier un répertoire."
    exit 1
fi

for file in "$1"/*.txt; do
    if [ -f "$file" ]; then
        echo -n "Voulez-vous visualiser le fichier $(basename "$file") ? (o/n) : "
        read reponse
        if [ "$reponse" = "o" ]; then
            more "$file"
        fi
    fi
done
```

---

## 9. Créer un script qui demande à l’utilisateur de saisir une note
Voici un script qui demande à l'utilisateur de saisir une note et affiche un message en fonction de celle-ci :

```bash
#!/bin/bash

while true; do
    read -p "Entrez une note (ou appuyez sur 'q' pour quitter) : " note
    if [[ "$note" == "q" ]]; then
        break
    elif [[ "$note" =~ ^[0-9]+(.[0-9]+)?$ ]]; then
        if (( $(echo "$note >= 16" | bc -l) )); then
            echo "Très bien"
        elif (( $(echo "$note >= 14" | bc -l) )); then
            echo "Bien"
        elif (( $(echo "$note >= 12" | bc -l) )); then
            echo "Assez bien"
        elif (( $(echo "$note >= 10" | bc -l) )); then
            echo "Moyen"
        else
            echo "Insuffisant"
        fi
    else
        echo "Veuillez entrer une note valide."
    fi
done
```
