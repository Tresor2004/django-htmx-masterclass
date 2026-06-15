# Module 6: Gestion des Structures de Données (Listes, Dicts, JSON, CSV) 📊

## 🎯 Objectif du Module
Maîtriser la manipulation de **listes, dictionnaires, tuples, JSON, CSV** et leur circulation entre views et templates.

---

## 1️⃣ Listes - Les Fondamentaux

### Créer et Manipuler des Listes en Django

```python
from django.shortcuts import render
from .models import Book

def books_view(request):
    # Récupérer une liste de livres
    books = Book.objects.all()  # C'est une QuerySet (presque une liste)
    
    # Convertir en liste Python
    books_list = list(books)
    
    # Extraire juste les titres (list comprehension)
    titles = [book.title for book in books]
    # ['Django 101', 'Python Pro', 'Web Dev']
    
    # Filtrer les livres
    popular_books = [b for b in books if b.rating > 4]
    
    # Trier les livres
    sorted_books = sorted(books, key=lambda b: b.title)
    
    # Grouper les livres par auteur
    books_by_author = {}
    for book in books:
        if book.author not in books_by_author:
            books_by_author[book.author] = []
        books_by_author[book.author].append(book)
    
    context = {
        'books': books,
        'titles': titles,
        'popular_books': popular_books,
        'sorted_books': sorted_books,
        'books_by_author': books_by_author,
    }
    
    return render(request, 'library/books.html', context)
```

### Afficher les Listes dans les Templates

```html
<!-- Boucle simple -->
<ul>
    {% for book in books %}
        <li>{{ book.title }}</li>
    {% endfor %}
</ul>

<!-- Accéder à un index -->
<p>Premier livre: {{ books.0 }}</p>
<p>Deuxième livre: {{ books.1 }}</p>

<!-- Longueur -->
<p>Total: {{ books|length }} livres</p>

<!-- Première et dernière -->
<p>Premier: {{ books|first }}</p>
<p>Dernier: {{ books|last }}</p>

<!-- Slicing -->
<p>Les 3 premiers: {{ books|slice:":3" }}</p>

<!-- Rejoindre avec un séparateur -->
<p>Titres: {{ titles|join:", " }}</p>

<!-- Vérifier si vide -->
{% if books %}
    <p>Il y a des livres!</p>
{% empty %}
    <p>Aucun livre.</p>
{% endif %}
```

---

## 2️⃣ Dictionnaires - Les Fondamentaux

### Créer et Manipuler des Dictionnaires

```python
def library_stats(request):
    books = Book.objects.all()
    
    # Créer un dictionnaire simple
    stats = {
        'total_books': books.count(),
        'total_authors': Book.objects.values('author').distinct().count(),
        'average_rating': books.aggregate(models.Avg('rating'))['rating__avg'],
    }
    
    # Créer un dictionnaire avec des listes comme valeurs
    books_by_author = {}
    for book in books:
        author_name = book.author.name
        if author_name not in books_by_author:
            books_by_author[author_name] = []
        books_by_author[author_name].append(book)
    
    # Compter les occurrences
    rating_counts = {}
    for book in books:
        rating = book.rating
        if rating not in rating_counts:
            rating_counts[rating] = 0
        rating_counts[rating] += 1
    # {5: 10, 4: 15, 3: 8}
    
    # Extraire des clés et valeurs
    all_ratings = list(rating_counts.keys())
    all_counts = list(rating_counts.values())
    
    context = {
        'stats': stats,
        'books_by_author': books_by_author,
        'rating_counts': rating_counts,
    }
    
    return render(request, 'library/stats.html', context)
```

### Afficher les Dictionnaires dans les Templates

```html
<!-- Accéder à une clé -->
<p>Total: {{ stats.total_books }}</p>
<p>Moyenne: {{ stats.average_rating|floatformat:2 }}</p>

<!-- Boucler sur les clés/valeurs -->
<h2>Livres par Auteur</h2>
<ul>
    {% for author, books in books_by_author.items %}
        <li>
            <strong>{{ author }}</strong>: {{ books|length }} livres
            <ul>
                {% for book in books %}
                    <li>{{ book.title }}</li>
                {% endfor %}
            </ul>
        </li>
    {% endfor %}
</ul>

<!-- Boucler sur les clés seulement -->
<p>Notations: {{ rating_counts.keys|join:", " }}</p>

<!-- Boucler sur les valeurs seulement -->
<p>Comptes: {{ rating_counts.values|join:", " }}</p>

<!-- Vérifier si une clé existe -->
{% if 'special_key' in stats %}
    <p>Clé trouvée!</p>
{% endif %}
```

---

## 3️⃣ Tuples - Les Fondamentaux

### Créer et Manipuler des Tuples

```python
def book_choices(request):
    # Les tuples sont souvent utilisés pour les choix
    rating_choices = (
        (1, '⭐ Mauvais'),
        (2, '⭐⭐ Moyen'),
        (3, '⭐⭐⭐ Bon'),
        (4, '⭐⭐⭐⭐ Très bon'),
        (5, '⭐⭐⭐⭐⭐ Excellent'),
    )
    
    # Créer un tuple de données
    book_data = ('Django 101', 'Alex', 2020)
    title, author, year = book_data  # Unpacking
    
    context = {
        'rating_choices': rating_choices,
        'book_data': book_data,
    }
    
    return render(request, 'library/choices.html', context)
```

### Afficher les Tuples dans les Templates

```html
<!-- Boucler sur les tuples -->
<select name="rating">
    {% for value, label in rating_choices %}
        <option value="{{ value }}">{{ label }}</option>
    {% endfor %}
</select>

<!-- Accéder à un index dans un tuple -->
<p>Titre: {{ book_data.0 }}</p>
<p>Auteur: {{ book_data.1 }}</p>
<p>Année: {{ book_data.2 }}</p>
```

---

## 4️⃣ JSON - L'Échange de Données

### Créer et Manipuler du JSON en Python

```python
import json
from django.http import JsonResponse

def api_books(request):
    books = Book.objects.all()
    
    # Créer une liste de dictionnaires
    books_data = [
        {
            'id': book.id,
            'title': book.title,
            'author': book.author.name,
            'rating': float(book.rating),
        }
        for book in books
    ]
    
    # Retourner du JSON (pour HTMX ou API)
    return JsonResponse({
        'status': 'success',
        'count': len(books_data),
        'books': books_data,
    })

def api_book_detail(request, book_id):
    book = get_object_or_404(Book, id=book_id)
    
    data = {
        'id': book.id,
        'title': book.title,
        'author': {
            'id': book.author.id,
            'name': book.author.name,
        },
        'reviews': [
            {
                'user': review.user.username,
                'rating': review.rating,
                'comment': review.comment,
            }
            for review in book.review_set.all()
        ]
    }
    
    return JsonResponse(data)

# Pour une liste de JSON (API)
from django.http import JsonResponse
from django.views.decorators.http import require_GET

@require_GET
def api_search(request):
    query = request.GET.get('q', '')
    
    if query:
        books = Book.objects.filter(title__icontains=query)
    else:
        books = []
    
    results = [{'id': b.id, 'title': b.title} for b in books]
    
    return JsonResponse({'results': results})
```

### Recevoir du JSON en Django

```python
import json
from django.views.decorators.http import require_POST

@require_POST
def create_book_api(request):
    # Recevoir du JSON
    data = json.loads(request.body)
    
    # Extraire les données
    title = data.get('title')
    author_id = data.get('author_id')
    
    # Créer le livre
    book = Book.objects.create(title=title, author_id=author_id)
    
    # Retourner du JSON
    return JsonResponse({
        'status': 'created',
        'book': {
            'id': book.id,
            'title': book.title,
        }
    })
```

### Utiliser du JSON avec HTMX

```html
<!-- Envoyer du JSON -->
<button hx-post="/api/books/"
        hx-headers='{"Content-Type": "application/json"}'
        hx-vals='{"title": "Mon Livre", "author_id": 1}'>
    Créer
</button>

<!-- Récupérer du JSON et l'afficher -->
<div hx-get="/api/books/"
     hx-trigger="load"
     hx-select=".book"
     hx-target="#books-list">
</div>
```

---

## 5️⃣ CSV - Exporter et Importer des Données

### Exporter en CSV

```python
import csv
from django.http import HttpResponse

def export_books_csv(request):
    # Récupérer les livres
    books = Book.objects.all()
    
    # Créer une réponse HTTP avec le type CSV
    response = HttpResponse(content_type='text/csv')
    response['Content-Disposition'] = 'attachment; filename="books.csv"'
    
    # Créer le writer CSV
    writer = csv.writer(response)
    
    # Écrire l'en-tête
    writer.writerow(['ID', 'Titre', 'Auteur', 'Année', 'Note'])
    
    # Écrire les données
    for book in books:
        writer.writerow([
            book.id,
            book.title,
            book.author.name,
            book.published_year,
            book.rating,
        ])
    
    return response

# Version avec DictWriter (plus propre)
def export_books_csv_v2(request):
    books = Book.objects.all()
    
    response = HttpResponse(content_type='text/csv')
    response['Content-Disposition'] = 'attachment; filename="books.csv"'
    
    writer = csv.DictWriter(response, fieldnames=['id', 'title', 'author', 'year', 'rating'])
    writer.writeheader()
    
    for book in books:
        writer.writerow({
            'id': book.id,
            'title': book.title,
            'author': book.author.name,
            'year': book.published_year,
            'rating': book.rating,
        })
    
    return response
```

### Importer un CSV

```python
def import_books_csv(request):
    if request.method == 'POST':
        csv_file = request.FILES['file']
        
        # Lire le CSV
        decoded_file = csv_file.read().decode('utf-8').splitlines()
        reader = csv.DictReader(decoded_file)
        
        count = 0
        errors = []
        
        for row in reader:
            try:
                # Récupérer ou créer l'auteur
                author, _ = Author.objects.get_or_create(
                    name=row['author']
                )
                
                # Créer le livre
                Book.objects.create(
                    title=row['title'],
                    author=author,
                    published_year=int(row['year']),
                    rating=float(row['rating']),
                )
                count += 1
            except Exception as e:
                errors.append(f"Erreur ligne {count + 1}: {str(e)}")
        
        return render(request, 'library/import_result.html', {
            'count': count,
            'errors': errors,
        })
    
    return render(request, 'library/import_form.html')
```

### Template pour l'Import

```html
{% extends "base.html" %}

{% block content %}

<div class="import-container">
    <h1>Importer des Livres (CSV)</h1>
    
    <form method="post" enctype="multipart/form-data">
        {% csrf_token %}
        
        <input type="file" name="file" accept=".csv" required>
        <button type="submit">Importer</button>
    </form>
    
    <p class="info">
        Format CSV attendu:
        <code>title,author,year,rating</code>
    </p>
</div>

{% endblock %}
```

---

## 6️⃣ Flux Complet: Exemple Avancé

### Scénario: Afficher les Statistiques Complètes

```python
from django.db.models import Avg, Count, Q
import json

def library_dashboard(request):
    books = Book.objects.all()
    reviews = Review.objects.all()
    
    # LISTE: Les 10 meilleurs livres
    top_books = books.order_by('-rating')[:10]
    
    # DICTIONNAIRE: Statistiques générales
    stats = {
        'total_books': books.count(),
        'total_reviews': reviews.count(),
        'avg_rating': books.aggregate(Avg('rating'))['rating__avg'],
    }
    
    # DICTIONNAIRE: Livres par auteur
    books_by_author = {}
    for book in books:
        author = book.author.name
        if author not in books_by_author:
            books_by_author[author] = {
                'count': 0,
                'books': [],
            }
        books_by_author[author]['count'] += 1
        books_by_author[author]['books'].append(book)
    
    # JSON pour HTMX
    books_json = json.dumps([
        {
            'id': book.id,
            'title': book.title,
            'rating': float(book.rating),
        }
        for book in top_books
    ])
    
    context = {
        'stats': stats,
        'top_books': top_books,
        'books_by_author': books_by_author,
        'books_json': books_json,
    }
    
    return render(request, 'library/dashboard.html', context)
```

### Template

```html
{% extends "base.html" %}

{% block content %}

<div class="dashboard">
    <!-- Afficher les statistiques (DICT) -->
    <div class="stats-panel">
        <h2>Statistiques</h2>
        <p>Total: {{ stats.total_books }} livres</p>
        <p>Reviews: {{ stats.total_reviews }}</p>
        <p>Moyenne: {{ stats.avg_rating|floatformat:2 }}/5</p>
    </div>
    
    <!-- Meilleurs livres (LISTE) -->
    <div class="top-books">
        <h2>Top 10 Livres</h2>
        <ol>
            {% for book in top_books %}
                <li>{{ book.title }} ({{ book.rating }}/5)</li>
            {% endfor %}
        </ol>
    </div>
    
    <!-- Livres par auteur (DICT de LISTES) -->
    <div class="by-author">
        <h2>Livres par Auteur</h2>
        {% for author, data in books_by_author.items %}
            <div class="author-section">
                <h3>{{ author }} ({{ data.count }} livre{{ data.count|pluralize }})</h3>
                <ul>
                    {% for book in data.books %}
                        <li>{{ book.title }}</li>
                    {% endfor %}
                </ul>
            </div>
        {% endfor %}
    </div>
    
    <!-- Données JSON stockées -->
    <script>
        const booksData = {{ books_json|safe }};
        console.log(booksData);
    </script>
</div>

{% endblock %}
```

---

## ✅ Points Clés À Retenir

- ✅ **Listes**: boucles `{% for %}`, indexing `[0]`, filters `|length`
- ✅ **Dictionnaires**: accès `.keys()/.values()`, iteration `.items()`
- ✅ **Tuples**: unpacking, utilisation dans les choix
- ✅ **JSON**: `JsonResponse()`, `json.loads()`, `json.dumps()`
- ✅ **CSV**: `csv.writer()`, `csv.DictWriter()` pour export/import
- ✅ **Circulation**: views crée structure → template affiche
- ✅ **Transformation**: list comprehension, dictionnaires, agrégation

**Prêt pour Module 7? Le PROJET COMPLET! 🚀**
