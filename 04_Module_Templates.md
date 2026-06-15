# Module 4: Les Templates - Affichage et Passage de Données 🎨

## 🎯 Objectif du Module
Maîtriser les **templates Django** et comment **afficher les données** avec des tags spéciaux.

---

## 1️⃣ Qu'est-ce qu'un Template ?

Un **template** est un fichier **HTML + Django Tags** qui:
1. **Reçoit des données** depuis la view (le `context`)
2. **Les affiche** avec des tags spéciaux `{{ }}` et `{% %}`
3. **Génère du HTML** final pour le navigateur

```html
<!-- Exemple simple -->
<h1>Bienvenue {{ username }}!</h1>
<!-- Si context = {'username': 'Tresor'}, Django affiche: -->
<!-- <h1>Bienvenue Tresor!</h1> -->
```

---

## 2️⃣ Les Tags Django Essentiels

### Tag 1: Variables `{{ }}`
```html
<!-- Afficher une variable simple -->
<p>{{ book.title }}</p>
<p>{{ user.email }}</p>

<!-- Afficher un dictionnaire -->
<p>{{ context_dict.key }}</p>

<!-- Afficher un index de liste -->
<p>{{ books.0 }}</p>  <!-- Premier élément -->
```

### Tag 2: Boucles `{% for %}`
```html
<!-- Boucler sur une liste -->
<ul>
    {% for book in books %}
        <li>{{ book.title }} par {{ book.author }}</li>
    {% empty %}
        <li>Aucun livre trouvé.</li>
    {% endfor %}
</ul>

<!-- Afficher l'index -->
<ul>
    {% for book in books %}
        <li>#{{ forloop.counter }}: {{ book.title }}</li>
    {% endfor %}
</ul>
<!-- Affiche: #1: Django, #2: Python, etc. -->

<!-- Boucler sur un dictionnaire -->
<ul>
    {% for key, value in my_dict.items %}
        <li>{{ key }}: {{ value }}</li>
    {% endfor %}
</ul>
```

### Tag 3: Conditions `{% if %}`
```html
<!-- If simple -->
{% if user.is_authenticated %}
    <p>Bonjour {{ user.username }}!</p>
{% endif %}

<!-- If/Else -->
{% if books.count > 0 %}
    <p>Nous avons {{ books.count }} livres.</p>
{% else %}
    <p>Aucun livre trouvé.</p>
{% endif %}

<!-- If/Elif/Else -->
{% if rating >= 4 %}
    <p>Excellent!</p>
{% elif rating >= 3 %}
    <p>Bon</p>
{% else %}
    <p>À améliorer</p>
{% endif %}

<!-- Conditions complexes -->
{% if user.is_authenticated and user.is_staff %}
    <p>Admin trouvé!</p>
{% endif %}

{% if 'django' in books or 'python' in books %}
    <p>Livres de programmation trouvés!</p>
{% endif %}
```

### Tag 4: Filtres `|`
```html
<!-- Convertir en majuscules -->
<p>{{ book.title|upper }}</p>

<!-- Convertir en minuscules -->
<p>{{ author.name|lower }}</p>

<!-- Longueur -->
<p>Nous avons {{ books|length }} livres</p>

<!-- Formater une date -->
<p>{{ book.created_at|date:"d/m/Y" }}</p>

<!-- Valeur par défaut si vide -->
<p>{{ book.description|default:"Pas de description" }}</p>

<!-- Filtres chaînés -->
<p>{{ book.title|upper|truncatewords:"3" }}</p>
```

### Tag 5: URLs dynamiques
```html
<!-- Créer un lien vers une view -->
<a href="{% url 'book_detail' book.id %}">{{ book.title }}</a>
<!-- Génère: <a href="/books/5/">Django</a> -->

<!-- Avec paramètres -->
<a href="{% url 'author_detail' author_id=author.id %}">
    {{ author.name }}
</a>

<!-- Stocker l'URL dans une variable -->
{% url 'book_detail' book.id as book_url %}
<a href="{{ book_url }}">Voir le livre</a>
```

### Tag 6: Inclusion de templates
```html
<!-- Inclure un autre template -->
{% include 'header.html' %}

<!-- Inclure avec contexte -->
{% include 'book_card.html' with book=current_book %}

<!-- Inclure sans contexte actuel -->
{% include 'book_card.html' with book=current_book only %}
```

### Tag 7: Héritage de templates (IMPORTANT!)
```html
<!-- base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
</head>
<body>
    <header>Mon Site</header>
    
    {% block content %}
    {% endblock %}
    
    <footer>© 2024</footer>
</body>
</html>

<!-- books_list.html (hérité de base.html) -->
{% extends "base.html" %}

{% block title %}Tous les Livres{% endblock %}

{% block content %}
    <h1>Livres</h1>
    {% for book in books %}
        <p>{{ book.title }}</p>
    {% endfor %}
{% endblock %}
```

---

## 3️⃣ Exemple Complet: Affichage d'une Bibliothèque

### views.py
```python
from django.shortcuts import render
from .models import Book, Author

def library_view(request):
    books = Book.objects.all()
    popular_books = Book.objects.filter(rating__gte=4)[:3]
    authors = Author.objects.all()
    
    context = {
        'books': books,
        'popular_books': popular_books,
        'authors': authors,
        'total_books': books.count(),
        'user': request.user,
    }
    
    return render(request, 'library/library.html', context)
```

### library.html
```html
{% extends "base.html" %}

{% block title %}Bibliothèque{% endblock %}

{% block content %}

<div class="library">
    <!-- Header avec les stats -->
    <div class="stats">
        {% if user.is_authenticated %}
            <p>Bienvenue {{ user.username }}!</p>
        {% else %}
            <p><a href="{% url 'login' %}">Se connecter</a></p>
        {% endif %}
        
        <p>Total: {{ total_books }} livre{{ total_books|pluralize }}</p>
    </div>
    
    <!-- Livres populaires -->
    {% if popular_books %}
        <section class="popular">
            <h2>Livres Populaires</h2>
            <div class="books-grid">
                {% for book in popular_books %}
                    <div class="book-card">
                        <h3>{{ book.title|truncatewords:"5" }}</h3>
                        <p>Auteur: {{ book.author.name|title }}</p>
                        <p>⭐ {{ book.rating|default:"N/A" }}</p>
                        <a href="{% url 'book_detail' book.id %}">
                            Voir plus →
                        </a>
                    </div>
                {% endfor %}
            </div>
        </section>
    {% endif %}
    
    <!-- Tous les livres -->
    <section class="all-books">
        <h2>Tous les Livres</h2>
        
        {% if books %}
            <table>
                <thead>
                    <tr>
                        <th>#</th>
                        <th>Titre</th>
                        <th>Auteur</th>
                        <th>Année</th>
                        <th>Action</th>
                    </tr>
                </thead>
                <tbody>
                    {% for book in books %}
                        <tr class="{% cycle 'odd' 'even' %}">
                            <td>{{ forloop.counter }}</td>
                            <td>{{ book.title }}</td>
                            <td>{{ book.author.name }}</td>
                            <td>{{ book.published_year }}</td>
                            <td>
                                <a href="{% url 'book_detail' book.id %}">
                                    Détails
                                </a>
                            </td>
                        </tr>
                    {% endfor %}
                </tbody>
            </table>
        {% else %}
            <p>Aucun livre dans la bibliothèque.</p>
        {% endif %}
    </section>
    
    <!-- Liste des auteurs -->
    <section class="authors">
        <h2>Auteurs ({{ authors|length }})</h2>
        <ul>
            {% for author in authors %}
                <li>
                    <a href="{% url 'author_detail' author.id %}">
                        {{ author.name }}
                    </a>
                </li>
            {% empty %}
                <li>Aucun auteur.</li>
            {% endfor %}
        </ul>
    </section>

</div>

{% endblock %}
```

---

## 4️⃣ Pièges Courants

### ❌ Piège 1: Accéder à une clé inexistante
```html
<!-- MAUVAIS: affiche rien si key n'existe pas -->
<p>{{ my_dict.nonexistent_key }}</p>

<!-- BON: utiliser default -->
<p>{{ my_dict.nonexistent_key|default:"N/A" }}</p>
```

### ❌ Piège 2: Oublier les guillemets dans les conditions
```html
<!-- MAUVAIS -->
{% if status == active %}  <!-- active est une variable, pas une string! -->

<!-- BON -->
{% if status == "active" %}
```

### ❌ Piège 3: Appeler des méthodes sans ()
```html
<!-- MAUVAIS -->
<p>{{ book.get_full_title }}</p>  <!-- Affiche le texte de la fonction! -->

<!-- BON -->
{{ book.get_full_title }}  <!-- Django appelle la méthode automatiquement! -->
```

### ❌ Piège 4: Ne pas passer la variable au template
```python
# MAUVAIS: view.py
def my_view(request):
    books = Book.objects.all()
    return render(request, 'template.html')  # books n'est pas dans le context!

# BON: view.py
def my_view(request):
    books = Book.objects.all()
    return render(request, 'template.html', {'books': books})
```

---

## 5️⃣ Exercice 4: Créer Un Template Complet

### Ton Défi:
1. Crée un template `book_detail.html` qui affiche un livre
2. Affiche aussi tous les autres livres du même auteur
3. Ajoute un lien "Retour à la liste"
4. Gère le cas où l'utilisateur n'est pas connecté

### Hints:
```html
{% extends "base.html" %}

{% block content %}

<!-- À faire: afficher le livre -->
<h1>{{ book.title }}</h1>

<!-- À faire: afficher les autres livres de l'auteur -->
{% if related_books %}
    <h2>Autres livres de {{ book.author }}</h2>
    <!-- À faire: boucle -->
{% endif %}

<!-- À faire: afficher un bouton si l'utilisateur est connecté -->
{% if user.is_authenticated %}
    <a href="#">Laisser une review</a>
{% endif %}

{% endblock %}
```

**Solution plus bas!**

---

## ✅ Points Clés À Retenir

- ✅ `{{ variable }}` = afficher une variable
- ✅ `{% for %}...{% endfor %}` = boucler
- ✅ `{% if %}...{% endif %}` = condition
- ✅ `|` = appliquer un filtre
- ✅ `{% url 'name' param %}` = générer une URL
- ✅ `{% extends %}` = hériter d'un template
- ✅ `{% include %}` = réutiliser un morceau de template

---

## 🎁 SOLUTION EXERCICE 4

```html
{% extends "base.html" %}

{% block title %}{{ book.title }}{% endblock %}

{% block content %}

<div class="book-detail">
    <a href="{% url 'books_list' %}">← Retour</a>
    
    <h1>{{ book.title }}</h1>
    <p class="author">
        Par <a href="{% url 'author_detail' book.author.id %}">
            {{ book.author.name }}
        </a>
    </p>
    
    <div class="book-info">
        <p><strong>ISBN:</strong> {{ book.isbn }}</p>
        <p><strong>Année:</strong> {{ book.published_year }}</p>
        <p><strong>Description:</strong> {{ book.description|default:"Pas de description" }}</p>
    </div>
    
    {% if user.is_authenticated %}
        <a href="{% url 'add_review' book.id %}" class="btn-primary">
            ✍️ Laisser une review
        </a>
    {% else %}
        <p><a href="{% url 'login' %}">Se connecter</a> pour laisser une review</p>
    {% endif %}
    
    {% if related_books %}
        <section class="related-books">
            <h2>Autres livres de {{ book.author.name }}</h2>
            <ul>
                {% for related_book in related_books %}
                    <li>
                        <a href="{% url 'book_detail' related_book.id %}">
                            {{ related_book.title }}
                        </a>
                    </li>
                {% endfor %}
            </ul>
        </section>
    {% endif %}
    
</div>

{% endblock %}
```

**Prêt pour Module 5? C'est là que la MAGIE arrive avec HTMX! ✨**
