# Module 1: Les Fondamentaux - Cycle Request/Response 🔄

## 🎯 Objectif du Module
Comprendre **comment les données circulent** dans Django, du client au serveur et retour.

---

## 1️⃣ Le Cycle Fondamental (THE MAGIC!)

```
┌─────────────┐
│   CLIENT    │ (Navigateur, formulaire HTML)
│  (Browser)  │
└──────┬──────┘
       │
       │ 1. Utilisateur envoie une requête
       │    (clic sur un lien, soumission de formulaire)
       │
       ▼
┌─────────────────────────────────────┐
│   urls.py (Routage)                 │
│  "Quelle view doit traiter cela?"   │
└──────┬──────────────────────────────┘
       │
       │ 2. Django trouve la bonne view
       │
       ▼
┌─────────────────────────────────────┐
│   views.py (Logique Métier)         │
│  "Je traite la requête et je fais   │
│   quelque chose avec les données"   │
└──────┬──────────────────────────────┘
       │
       │ 3. La view récupère des données
       │    (base de données, formulaires, etc.)
       │
       ▼
┌─────────────────────────────────────┐
│   templates/ (Présentation)         │
│  "J'affiche les données au client"  │
└──────┬──────────────────────────────┘
       │
       │ 4. Django crée une page HTML
       │
       ▼
┌─────────────┐
│   CLIENT    │ (Navigateur affiche la page)
│  (Browser)  │
└─────────────┘
```

**C'est ça le cycle! Maintenant, décortiquons chaque partie.**

---

## 2️⃣ Comprendre REQUEST et RESPONSE

### Qu'est-ce qu'un REQUEST ?
Un **REQUEST** est un message que le navigateur envoie au serveur:

```python
# Exemple de ce qu'est un REQUEST
REQUEST = {
    'method': 'GET',  # ou POST, PUT, DELETE, etc.
    'path': '/books/5/',  # l'URL
    'GET': {'search': 'django'},  # données de l'URL (?search=django)
    'POST': {'title': 'Django for Beginners'},  # données du formulaire
    'user': User(username='tresor'),  # l'utilisateur connecté
    'FILES': {},  # fichiers uploadés
    # ... et beaucoup d'autres infos
}
```

### Qu'est-ce qu'une RESPONSE ?
Une **RESPONSE** est le message que Django envoie au navigateur:

```python
# Exemple de ce qu'est une RESPONSE
RESPONSE = {
    'status_code': 200,  # 200 = OK, 404 = pas trouvé, 500 = erreur
    'headers': {
        'Content-Type': 'text/html; charset=utf-8',
    },
    'body': '<html>...</html>'  # le contenu HTML
}
```

---

## 3️⃣ Les Trois Méthodes HTTP Principales

### GET - "Récupérer des données"
```
Utilisateur clique sur un lien → Django affiche une page
```

**Exemple:**
```html
<!-- Utilisateur clique sur ce lien -->
<a href="/books/">Voir tous les livres</a>

<!-- Django reçoit: REQUEST.method = 'GET' et REQUEST.path = '/books/' -->
```

### POST - "Envoyer des données"
```
Utilisateur remplit un formulaire → Django traite les données
```

**Exemple:**
```html
<!-- Utilisateur remplitce formulaire et clique sur "Envoyer" -->
<form method="post" action="/books/add/">
    <input name="title" value="Django Masterclass">
    <button type="submit">Ajouter</button>
</form>

<!-- Django reçoit: REQUEST.method = 'POST' et REQUEST.POST = {'title': 'Django Masterclass'} -->
```

### HTMX - "Mise à jour partielle sans recharger la page"
```
Utilisateur clique sur un bouton → Django retourne un petit bout HTML
```

(Tu comprendras mieux en Module 5)

---

## 4️⃣ Exemple Concret: Afficher les Livres

Imaginons qu'un utilisateur clique sur "Voir tous les livres".

### Étape 1: `urls.py` - Définir la route
```python
from django.urls import path
from . import views

urlpatterns = [
    # Quand quelqu'un va à /books/, appelle la fonction books_list
    path('books/', views.books_list, name='books_list'),
]
```

### Étape 2: `views.py` - Traiter la requête
```python
from django.shortcuts import render
from .models import Book

def books_list(request):
    # REQUEST: Django nous donne l'objet 'request'
    # On récupère tous les livres de la base de données
    books = Book.objects.all()  # Liste de tous les livres
    
    # RESPONSE: On cr��e une réponse
    context = {
        'books': books,  # On passe les livres au template
        'title': 'Tous les livres',
    }
    
    # Django rend (render) le template avec les données
    return render(request, 'library/books_list.html', context)
```

### Étape 3: `books_list.html` - Afficher les données
```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ title }}</title>
</head>
<body>
    <h1>{{ title }}</h1>
    
    <!-- books est une LISTE de livres -->
    {% for book in books %}
        <div class="book">
            <h2>{{ book.title }}</h2>
            <p>Auteur: {{ book.author }}</p>
        </div>
    {% empty %}
        <p>Aucun livre trouvé.</p>
    {% endfor %}
</body>
</html>
```

### Le Flux Complet:
```
1. Utilisateur clique: /books/
2. Django cherche dans urls.py → trouve views.books_list
3. views.books_list récupère les livres: books = Book.objects.all()
4. context = {'books': books, ...} (C'est un DICTIONNAIRE!)
5. render() mélange le template + context
6. Template affiche: {% for book in books %} ... {% endfor %}
7. Navigateur reçoit une belle page HTML
```

---

## 5️⃣ Les Structures de Données Que Tu Vas Rencontrer

### Listes
```python
# Dans la view
books = [
    {'title': 'Django 101', 'author': 'Alex'},
    {'title': 'Python Pro', 'author': 'Bob'},
]

# Dans le template
{% for book in books %}
    {{ book.title }}
{% endfor %}
```

### Dictionnaires
```python
# Dans la view
context = {
    'user': 'Tresor',
    'books_count': 42,
}

# Dans le template
{{ user }} a {{ books_count }} livres
```

### Tuples
```python
# Dans la view (rarement utilisé directement)
choices = ('Français', 'Anglais', 'Espagnol')

# Dans le template
{% for choice in choices %}
    {{ choice }}
{% endfor %}
```

---

## 6️⃣ Résumé - À Retenir Absolument!

| Partie | Rôle | Données |
|--------|------|---------|
| **urls.py** | Router les requêtes | `path()` |
| **views.py** | Logique métier | `request` → traitement → `context` |
| **templates/** | Affichage HTML | `context` → HTML → navigateur |
| **models.py** | Base de données | Requêtes ORM comme `Book.objects.all()` |

**La Règle d'Or:**
```
request (du client)
    ↓
urlpatterns (trouve la bonne view)
    ↓
view (traite + crée context)
    ↓
template (affiche avec context)
    ↓
response (au client)
```

---

## 🧪 Exercice 1: Créer Ta Première Vue

### Ton Défi:
1. Crée un model `Author` avec `name` et `email`
2. Crée une vue qui récupère tous les auteurs
3. Affiche-les dans un template

**Solution au Module 2!** 👀

---

## 📌 Points Clés À Retenir
- ✅ REQUEST = message du navigateur vers Django
- ✅ RESPONSE = message de Django vers le navigateur  
- ✅ urls.py = routeur (trouvela bonne view)
- ✅ views.py = moteur (traite les données)
- ✅ templates = présentation (affiche HTML)
- ✅ context = dictionnaire (passe les données au template)

**Prêt pour Module 2? C'est là qu'on rentre dans les détails des VIEWS! 🚀**
