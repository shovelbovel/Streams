Gestion d’un magasin de vêtements avec Streams**

**1 - Créer une classe Vetement :**
Chaque vêtement est représenté par un objet ClothingItem contenant les informations suivantes :

- id : Identifiant unique (int).
- name : Nom du vêtement (String).
- category : Catégorie (par exemple, "T-shirt", "Pantalon") (String).
- price : Prix (double).
- stock : Quantité en stock (int).
- onSale : Indique si l’article est en promotion (boolean).

Ajouter à la classe un constructeur avec paramètre et une méthode toString

**2- Créer une classe magasin :**

- **Attributs** : une liste des vêtements
- **Constructeur** : sans paramètre et qui initialise la liste par une LinkedList
- **Méthode** : Ajouter une méthode pour chacune des tâches suivantes
o Lister les vêtements en promotion : Filtrer uniquement les articles avec onSale = true, puis afficher leur nom et prix.
o Trouver les 3 articles les moins chers : Trier les vêtements par prix croissant et afficher les 3 premiers.
o Calculer la valeur totale du stock : Multiplier le prix par la quantité en stock pour chaque article, puis additionner le tout.
o Regrouper les vêtements par catégorie : Utiliser les Collectors pour regrouper les articles dans une Map<String, List<ClothingItem>> selon leur catégorie.
o Lister les noms des vêtements avec une quantité supérieure à 20 : Filtrer les articles avec stock > 20 et collecter leurs noms dans une liste.
o Vérifier si tous les articles sont en stock : Utiliser allMatch pour vérifier si chaque article a un stock > 0.
o Appliquer une remise sur les articles en promotion : Créer un nouveau Stream où chaque article en promotion voit son prix réduit de 20%, puis collecter le résultat dans une nouvelle liste.
o Créer un affichage personnalisé pour chaque article : Utiliser map pour créer une chaîne de caractères formatée pour chaque article, comme :
“ID: 1 | Nom: T-shirt Rouge | Prix: 159dh | Stock: 50”

***

N’hésitez pas si vous souhaitez une version éditée, la traduction vers un autre format, ou d’autres informations !
<span style="display:none">[^1]</span>

<div align="center">⁂</div>

[^1]: WhatsApp-Image-2025-10-30-at-11.20.52.jpg

