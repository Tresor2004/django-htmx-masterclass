# Module 7: Projet Complet - Bibliothèque de Lecture en Ligne 📚

## 🎯 Objectif Final
Construire une **Bibliothèque en Ligne complète** avec:
- ✅ Gestion des livres et auteurs
- ✅ Système de reviews et reactions (likes)
- ✅ Interface réactive avec HTMX (pas de JavaScript)
- ✅ Recherche en temps réel
- ✅ Export/Import de données (CSV)
- ✅ Dashboard avec statistiques

---

## 📂 Structure du Projet

```
library_project/
├── config/                 # Configuration Django
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── library/                # App principale
│   ├── migrations/
│   ├── templates/library/
│   │   ├── base.html
│   │   ├── books_list.html
│   │   ├── book_detail.html
│   │   ├── add_review.html
│   │   ├── search_results.html
│   │   └── dashboard.html
│   ├── models.py
│   ├── forms.py
│   ├── views.py
│   ├── urls.py
│   ├── admin.py
│   └── apps.py
├── static/
│   ├── css/style.css
│   └── js/htmx.min.js
└── manage.py
```

---

## 1️⃣ MODELS.PY - Définir la Structure de Données

```python
# library/models.py

from django.db import models
from django.contrib.auth.models import User

# ==================== MODEL 1: AUTEUR ====================
class Author(models.Model):
    """
    Modèle pour stocker les informations des auteurs.
    
    Champs:
    - name: Le nom de l'auteur (max 100 caractères)
    - email: L'email de l'auteur (unique)
    - bio: Une biographie courte
    - created_at: Date de création de l'enregistrement
    """
    
    name = models.CharField(
        max_length=100,
        help_text="Nom complet de l'auteur"
    )
    email = models.EmailField(
        unique=True,
        help_text="Email unique de l'auteur"
    )
    bio = models.TextField(
        blank=True,
        null=True,
        help_text="Biographie de l'auteur"
    )
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        ordering = ['name']  # Trier par nom par défaut
        verbose_name = 'Auteur'
        verbose_name_plural = 'Auteurs'
    
    def __str__(self):
        # Affiche le nom quand on utilise print(author)
        return self.name
    
    def get_books_count(self):
        # Méthode personnalisée: compter les livres de cet auteur
        return self.book_set.count()


# ==================== MODEL 2: LIVRE ====================
class Book(models.Model):
    """
    Modèle pour stocker les livres.
    
    Champs:
    - title: Le titre du livre
    - author: Clé étrangère vers Author (une livre a UN auteur)
    - isbn: International Standard Book Number (unique)
    - description: Description longue du livre
    - published_year: L'année de publication
    - rating: Note moyenne (calculée depuis les reviews)
    - created_at: Date d'ajout à la bibliothèque
    """
    
    title = models.CharField(
        max_length=200,
        help_text="Titre du livre"
    )
    author = models.ForeignKey(
        Author,  # Lien vers le modèle Author
        on_delete=models.CASCADE,  # Si l'auteur est supprimé, supprimer ses livres
        help_text="Auteur du livre"
    )
    isbn = models.CharField(
        max_length=13,
        unique=True,
        help_text="ISBN unique du livre"
    )
    description = models.TextField(
        blank=True,
        null=True,
        help_text="Description du livre"
    )
    published_year = models.IntegerField(
        help_text="Année de publication"
    )
    # Note: Cette note est stockée mais calculée depuis les reviews
    rating = models.DecimalField(
        max_digits=3,
        decimal_places=2,
        default=0,
        help_text="Note moyenne du livre"
    )
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        ordering = ['-created_at']  # Les plus récents en premier
        verbose_name = 'Livre'
        verbose_name_plural = 'Livres'
    
    def __str__(self):
        # Affiche: "Django 101 par Alex"
        return f"{self.title} par {self.author.name}"
    
    def get_reviews_count(self):
        # Méthode personnalisée: compter les reviews de ce livre
        return self.review_set.count()
    
    def get_likes_count(self):
        # Méthode personnalisée: compter les likes de ce livre
        return self.booklike_set.count()
    
    def get_average_rating(self):
        # Calculer la note moyenne depuis les reviews
        from django.db.models import Avg
        avg = self.review_set.aggregate(Avg('rating'))['rating__avg']
        return round(avg, 2) if avg else 0


# ==================== MODEL 3: REVIEW (Avis) ====================
class Review(models.Model):
    """
    Modèle pour les reviews/avis des utilisateurs sur les livres.
    
    Champs:
    - book: Le livre sur lequel porte la review
    - user: L'utilisateur qui a écrit la review
    - rating: Note de 1 à 5
    - comment: Le commentaire/avis
    - created_at: Date de création de la review
    """
    
    # Choix pour la note
    RATING_CHOICES = [
        (1, '⭐ Mauvais'),
        (2, '⭐⭐ Moyen'),
        (3, '⭐⭐⭐ Bon'),
        (4, '⭐⭐⭐⭐ Très bon'),
        (5, '⭐⭐⭐⭐⭐ Excellent'),
    ]
    
    book = models.ForeignKey(
        Book,
        on_delete=models.CASCADE,  # Si le livre est supprimé, supprimer ses reviews
        help_text="Le livre en question"
    )
    user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,  # Si l'utilisateur est supprimé, supprimer ses reviews
        help_text="L'utilisateur qui fait la review"
    )
    rating = models.IntegerField(
        choices=RATING_CHOICES,
        help_text="Note de 1 à 5"
    )
    comment = models.TextField(
        help_text="Le commentaire de l'utilisateur"
    )
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        # Un utilisateur ne peut faire qu'une review par livre
        unique_together = ('user', 'book')
        ordering = ['-created_at']  # Les plus récentes en premier
        verbose_name = 'Review'
        verbose_name_plural = 'Reviews'
    
    def __str__(self):
        # Affiche: "Review de Alex pour Django 101 (5/5)"
        return f"Review de {self.user.username} pour {self.book.title} ({self.rating}/5)"


# ==================== MODEL 4: LIKE ====================
class BookLike(models.Model):
    """
    Modèle pour les "likes" sur les livres (une réaction).
    
    Champs:
    - book: Le livre qui est aimé
    - user: L'utilisateur qui aime le livre
    - created_at: Date du like
    """
    
    book = models.ForeignKey(
        Book,
        on_delete=models.CASCADE,  # Si le livre est supprimé, supprimer ses likes
        help_text="Le livre aimé"
    )
    user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,  # Si l'utilisateur est supprimé, supprimer ses likes
        help_text="L'utilisateur qui aime"
    )
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        # Un utilisateur ne peut aimer qu'une fois le même livre
        unique_together = ('user', 'book')
        ordering = ['-created_at']
        verbose_name = 'Like'
        verbose_name_plural = 'Likes'
    
    def __str__(self):
        # Affiche: "Alex aime Django 101"
        return f"{self.user.username} aime {self.book.title}"
```

---

## 2️⃣ FORMS.PY - Validation des Données

```python
# library/forms.py

from django import forms
from .models import Author, Book, Review

# ==================== FORM 1: FORMULAIRE D'AUTEUR ====================
class AuthorForm(forms.ModelForm):
    """
    Formulaire pour créer/modifier un auteur.
    
    ModelForm = génère le formulaire automatiquement depuis le modèle Author
    """
    
    class Meta:
        model = Author  # Lié au modèle Author
        fields = ['name', 'email', 'bio']  # Les champs à afficher
        
        # Personnaliser les labels
        labels = {
            'name': 'Nom de l\'auteur',
            'email': 'Email',
            'bio': 'Biographie',
        }
        
        # Personnaliser les widgets (éléments HTML)
        widgets = {
            'name': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Entrez le nom...',
            }),
            'email': forms.EmailInput(attrs={
                'class': 'form-control',
                'placeholder': 'email@exemple.com',
            }),
            'bio': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 4,
                'placeholder': 'Biographie...',
            }),
        }
    
    # Validation personnalisée: vérifier que l'email n'existe pas déjà
    def clean_email(self):
        """
        Cette méthode valide le champ email.
        Elle s'appelle automatiquement quand on appelle form.is_valid()
        """
        email = self.cleaned_data.get('email')
        
        # Si c'est un nouvel auteur OU l'email a changé
        if Author.objects.filter(email=email).exists():
            if not self.instance.pk or self.instance.email != email:
                raise forms.ValidationError('Cet email existe déjà!')
        
        return email


# ==================== FORM 2: FORMULAIRE DE LIVRE ====================
class BookForm(forms.ModelForm):
    """
    Formulaire pour créer/modifier un livre.
    """
    
    class Meta:
        model = Book
        fields = ['title', 'author', 'isbn', 'description', 'published_year']
        
        labels = {
            'title': 'Titre',
            'author': 'Auteur',
            'isbn': 'ISBN',
            'description': 'Description',
            'published_year': 'Année de publication',
        }
        
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Titre du livre...',
            }),
            'author': forms.Select(attrs={
                'class': 'form-control',
            }),
            'isbn': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': '978-3-16-148410-0',
            }),
            'description': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 5,
                'placeholder': 'Description du livre...',
            }),
            'published_year': forms.NumberInput(attrs={
                'class': 'form-control',
                'type': 'year',
            }),
        }
    
    # Validation personnalisée: vérifier l'ISBN
    def clean_isbn(self):
        """Valider que l'ISBN a 13 chiffres"""
        isbn = self.cleaned_data.get('isbn')
        
        # Enlever les tirets
        isbn_clean = isbn.replace('-', '')
        
        if not isbn_clean.isdigit() or len(isbn_clean) != 13:
            raise forms.ValidationError('ISBN invalide! Doit avoir 13 chiffres.')
        
        return isbn


# ==================== FORM 3: FORMULAIRE DE REVIEW ====================
class ReviewForm(forms.ModelForm):
    """
    Formulaire pour créer/modifier une review.
    """
    
    class Meta:
        model = Review
        fields = ['rating', 'comment']
        
        labels = {
            'rating': 'Votre note',
            'comment': 'Votre avis',
        }
        
        # Utiliser des RadioSelect pour les notes (plus visible)
        widgets = {
            'rating': forms.RadioSelect(attrs={
                'class': 'rating-option',
            }),
            'comment': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 5,
                'placeholder': 'Partagez votre avis...',
            }),
        }
    
    # Validation personnalisée: longueur minimum du commentaire
    def clean_comment(self):
        """Le commentaire doit avoir au least 10 caractères"""
        comment = self.cleaned_data.get('comment')
        
        if len(comment) < 10:
            raise forms.ValidationError('Le commentaire doit avoir au moins 10 caractères.')
        
        return comment


# ==================== FORM 4: FORMULAIRE DE RECHERCHE ====================
class SearchForm(forms.Form):
    """
    Formulaire simple (pas lié à un modèle) pour la recherche.
    """
    
    # Champ de recherche
    query = forms.CharField(
        max_length=100,
        required=False,  # Optionnel
        label='Chercher',
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Chercher un livre, un auteur...',
            'hx-get': '/search/',  # HTMX: chercher quand on tape
            'hx-target': '#search-results',  # Mettre les résultats ici
            'hx-trigger': 'keyup changed delay:300ms',  # Après un délai
        }),
    )
    
    # Filtrer par auteur
    author = forms.ModelChoiceField(
        queryset=Author.objects.all(),
        required=False,
        empty_label='Tous les auteurs',
        label='Filtre auteur',
    )
```

---

## 3️⃣ VIEWS.PY - La Logique Métier

```python
# library/views.py

from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.decorators import login_required
from django.http import JsonResponse
from django.db.models import Avg, Count, Q
from django.views.decorators.http import require_http_methods
import csv
import json

from .models import Author, Book, Review, BookLike
from .forms import AuthorForm, BookForm, ReviewForm, SearchForm

# ==================== VIEW 1: LISTE DE TOUS LES LIVRES ====================
def books_list(request):
    """
    Affiche la liste de tous les livres avec pagination et filtrage.
    
    Flux:
    1. Récupérer tous les livres
    2. Filtrer par auteur si demandé
    3. Passer au template
    """
    
    # Récupérer TOUS les livres, triés par création (plus récents d'abord)
    books = Book.objects.all().order_by('-created_at')
    
    # Récupérer le filtre d'auteur depuis l'URL (?author_id=1)
    author_id = request.GET.get('author_id')
    if author_id:
        # Filtrer par auteur
        books = books.filter(author_id=author_id)
    
    # Récupérer tous les auteurs pour le filtre
    authors = Author.objects.all()
    
    # Créer le contexte (dictionnaire de données)
    context = {
        'books': books,
        'authors': authors,
        'selected_author': author_id,
        'total_count': books.count(),
    }
    
    # Retourner le template avec le contexte
    return render(request, 'library/books_list.html', context)


# ==================== VIEW 2: DÉTAIL D'UN LIVRE ====================
def book_detail(request, book_id):
    """
    Affiche les détails complets d'un livre.
    
    Flux:
    1. Récupérer le livre (ou 404 si n'existe pas)
    2. Récupérer les reviews de ce livre
    3. Récupérer les autres livres de l'auteur
    4. Vérifier si l'utilisateur a liké
    """
    
    # Récupérer le livre ou retourner 404
    book = get_object_or_404(Book, id=book_id)
    
    # Récupérer les reviews de ce livre, triées par plus récentes
    reviews = book.review_set.all().order_by('-created_at')
    
    # Récupérer les autres livres du même auteur (exclure celui-ci)
    related_books = Book.objects.filter(
        author=book.author
    ).exclude(
        id=book.id
    )[:5]  # Seulement les 5 premiers
    
    # Vérifier si l'utilisateur connecté a liké ce livre
    is_liked = False
    if request.user.is_authenticated:
        is_liked = BookLike.objects.filter(
            book=book,
            user=request.user
        ).exists()
    
    context = {
        'book': book,
        'reviews': reviews,
        'related_books': related_books,
        'is_liked': is_liked,
        'reviews_count': reviews.count(),
        'likes_count': book.get_likes_count(),
    }
    
    return render(request, 'library/book_detail.html', context)


# ==================== VIEW 3: AJOUTER UN LIVRE ====================
@require_http_methods(['GET', 'POST'])  # Accepte GET et POST seulement
def add_book(request):
    """
    Ajouter un nouveau livre à la bibliothèque.
    
    Flux:
    - GET: Afficher le formulaire vide
    - POST: Valider et sauvegarder le livre
    """
    
    if request.method == 'POST':
        # L'utilisateur a soumis le formulaire
        form = BookForm(request.POST)  # Créer le form avec les données POST
        
        if form.is_valid():
            # Les données sont correctes! Sauvegarder
            book = form.save()
            
            # Rediriger vers le détail du livre créé
            return redirect('book_detail', book_id=book.id)
        # Si invalide, réafficher le formulaire avec les erreurs
    else:
        # GET: Afficher le formulaire vide
        form = BookForm()
    
    context = {'form': form}
    return render(request, 'library/add_book.html', context)


# ==================== VIEW 4: AJOUTER UNE REVIEW ====================
@login_required  # L'utilisateur DOIT être connecté
def add_review(request, book_id):
    """
    Ajouter une review pour un livre.
    
    Flux:
    1. Récupérer le livre
    2. Si POST: valider et sauvegarder la review
    3. Si GET: afficher le formulaire
    """
    
    # Récupérer le livre
    book = get_object_or_404(Book, id=book_id)
    
    if request.method == 'POST':
        form = ReviewForm(request.POST)
        
        if form.is_valid():
            # Créer l'instance mais ne PAS sauvegarder encore
            review = form.save(commit=False)
            
            # Ajouter les données qu'on ne peut pas mettre dans le form
            review.book = book
            review.user = request.user
            
            # Maintenant sauvegarder
            review.save()
            
            # Mettre à jour la note moyenne du livre
            avg_rating = book.get_average_rating()
            book.rating = avg_rating
            book.save()
            
            # HTMX: retourner juste la nouvelle review
            if request.headers.get('HX-Request'):
                return render(request, 'library/review_item.html', {'review': review})
            
            # Sinon, rediriger
            return redirect('book_detail', book_id=book.id)
    else:
        form = ReviewForm()
    
    context = {
        'form': form,
        'book': book,
    }
    
    return render(request, 'library/add_review.html', context)


# ==================== VIEW 5: RECHERCHE EN TEMPS RÉEL (HTMX) ====================
def search_books(request):
    """
    Chercher des livres en temps réel (HTMX).
    
    Flux:
    1. Récupérer le query depuis l'URL (?q=django)
    2. Filtrer les livres
    3. Retourner JUSTE les résultats (pas la page entière!)
    """
    
    # Récupérer la requête de recherche
    query = request.GET.get('q', '').strip()
    
    # Si pas de query, retourner rien
    if not query:
        return render(request, 'library/search_results.html', {'books': []})
    
    # Chercher dans les titres ET les auteurs
    books = Book.objects.filter(
        Q(title__icontains=query) |  # OR: titre contient query
        Q(author__name__icontains=query)  # OU: auteur contient query
    ).distinct()  # Éviter les doublons
    
    context = {
        'books': books,
        'query': query,
        'results_count': books.count(),
    }
    
    # Retourner JUSTE les résultats (HTMX!)
    return render(request, 'library/search_results.html', context)


# ==================== VIEW 6: LIKE/UNLIKE (HTMX) ====================
@login_required
def toggle_like(request, book_id):
    """
    Aimer ou ne pas aimer un livre (toggle).
    
    Flux:
    1. Récupérer le livre
    2. Chercher si le like existe
    3. Si existe: supprimer (unlike)
    4. Si n'existe pas: créer (like)
    5. Retourner le bouton mis à jour
    """
    
    book = get_object_or_404(Book, id=book_id)
    
    # Chercher le like
    like = BookLike.objects.filter(
        user=request.user,
        book=book
    ).first()
    
    if like:
        # Déjà liké, donc unlike
        like.delete()
        is_liked = False
    else:
        # Pas liké, donc créer le like
        BookLike.objects.create(
            user=request.user,
            book=book
        )
        is_liked = True
    
    # Récupérer le nombre de likes mise à jour
    likes_count = book.get_likes_count()
    
    context = {
        'book': book,
        'is_liked': is_liked,
        'likes_count': likes_count,
    }
    
    # Retourner JUSTE le bouton mis à jour (HTMX!)
    return render(request, 'library/like_button.html', context)


# ==================== VIEW 7: DASHBOARD AVEC STATS ====================
def dashboard(request):
    """
    Afficher le dashboard avec des statistiques.
    
    Flux:
    1. Récupérer des stats de la base de données
    2. Créer des dictionnaires et listes pour les afficher
    """
    
    # Récupérer tous les objets
    books = Book.objects.all()
    reviews = Review.objects.all()
    authors = Author.objects.all()
    
    # STATISTIQUES: créer un dictionnaire
    stats = {
        'total_books': books.count(),
        'total_authors': authors.count(),
        'total_reviews': reviews.count(),
        'avg_rating': books.aggregate(Avg('rating'))['rating__avg'] or 0,
    }
    
    # MEILLEURS LIVRES: créer une liste
    top_books = books.order_by('-rating')[:5]
    
    # LIVRES PAR AUTEUR: créer un dictionnaire de listes
    books_by_author = {}
    for author in authors:
        author_books = books.filter(author=author)
        books_by_author[author.name] = {
            'count': author_books.count(),
            'avg_rating': author_books.aggregate(Avg('rating'))['rating__avg'] or 0,
        }
    
    context = {
        'stats': stats,
        'top_books': top_books,
        'books_by_author': books_by_author,
    }
    
    return render(request, 'library/dashboard.html', context)


# ==================== VIEW 8: EXPORTER EN CSV ====================
def export_books_csv(request):
    """
    Exporter tous les livres en fichier CSV.
    
    Flux:
    1. Récupérer les livres
    2. Créer un fichier CSV
    3. Écrire les données
    4. Télécharger le fichier
    """
    
    # Récupérer les livres
    books = Book.objects.all()
    
    # Créer une réponse HTTP avec le type CSV
    response = HttpResponse(content_type='text/csv')
    response['Content-Disposition'] = 'attachment; filename="books.csv"'
    
    # Créer le writer CSV
    writer = csv.DictWriter(response, fieldnames=[
        'id', 'title', 'author', 'isbn', 'year', 'rating', 'reviews'
    ])
    
    # Écrire l'en-tête
    writer.writeheader()
    
    # Écrire les données
    for book in books:
        writer.writerow({
            'id': book.id,
            'title': book.title,
            'author': book.author.name,
            'isbn': book.isbn,
            'year': book.published_year,
            'rating': book.rating,
            'reviews': book.get_reviews_count(),
        })
    
    return response


# ==================== VIEW 9: IMPORTER UN CSV ====================
def import_books_csv(request):
    """
    Importer des livres depuis un fichier CSV.
    
    Flux:
    1. Si POST: lire le fichier CSV
    2. Parser chaque ligne
    3. Créer les auteurs et livres
    4. Afficher un résumé
    """
    
    if request.method == 'POST':
        csv_file = request.FILES.get('file')
        
        if not csv_file:
            return render(request, 'library/import_csv.html', {
                'error': 'Veuillez sélectionner un fichier'
            })
        
        # Lire le CSV
        decoded_file = csv_file.read().decode('utf-8').splitlines()
        reader = csv.DictReader(decoded_file)
        
        count = 0
        errors = []
        
        # Parcourir chaque ligne du CSV
        for row in reader:
            try:
                # Récupérer ou créer l'auteur
                author, _ = Author.objects.get_or_create(
                    name=row['author'],
                    defaults={'email': f"{row['author'].lower().replace(' ', '')}@example.com"}
                )
                
                # Créer le livre
                Book.objects.create(
                    title=row['title'],
                    author=author,
                    isbn=row['isbn'],
                    published_year=int(row['year']),
                )
                count += 1
            except Exception as e:
                errors.append(f"Ligne {count + 1}: {str(e)}")
        
        context = {
            'success': True,
            'imported_count': count,
            'errors': errors,
        }
    else:
        context = {'success': False}
    
    return render(request, 'library/import_csv.html', context)


# ==================== VIEW 10: API JSON (pour HTMX/Frontend) ====================
def api_books_json(request):
    """
    Retourner les livres en JSON (pour HTMX ou frontend).
    
    Flux:
    1. Récupérer les livres
    2. Les convertir en dictionnaire
    3. Retourner du JSON
    """
    
    # Récupérer les livres
    books = Book.objects.all()
    
    # Convertir en dictionnaire
    books_data = [
        {
            'id': book.id,
            'title': book.title,
            'author': book.author.name,
            'rating': float(book.rating),
            'reviews': book.get_reviews_count(),
            'likes': book.get_likes_count(),
        }
        for book in books
    ]
    
    # Retourner du JSON
    return JsonResponse({
        'status': 'success',
        'count': len(books_data),
        'books': books_data,
    })
```

---

## 4️⃣ URLS.PY - Routes

```python
# library/urls.py

from django.urls import path
from . import views

# Nommer les patterns pour les références dans les templates
urlpatterns = [
    # Pages principales
    path('', views.books_list, name='books_list'),
    path('book/<int:book_id>/', views.book_detail, name='book_detail'),
    path('add-book/', views.add_book, name='add_book'),
    
    # Reviews et réactions
    path('book/<int:book_id>/review/', views.add_review, name='add_review'),
    path('book/<int:book_id>/like/', views.toggle_like, name='toggle_like'),
    
    # Recherche et filtres
    path('search/', views.search_books, name='search_books'),
    
    # Dashboard
    path('dashboard/', views.dashboard, name='dashboard'),
    
    # Import/Export
    path('export/csv/', views.export_books_csv, name='export_csv'),
    path('import/csv/', views.import_books_csv, name='import_csv'),
    
    # API JSON
    path('api/books/', views.api_books_json, name='api_books'),
]
```

---

## 5️⃣ TEMPLATES - L'Interface

### base.html
```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Bibliothèque en Ligne{% endblock %}</title>
    
    <!-- HTMX -->
    <script src="https://unpkg.com/htmx.org@1.9.10"></script>
    
    <!-- CSS personnalisé -->
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
</head>
<body>
    <!-- Navigation -->
    <nav class="navbar">
        <div class="navbar-brand">
            <a href="{% url 'books_list' %}">📚 Bibliothèque</a>
        </div>
        
        <div class="navbar-menu">
            <a href="{% url 'books_list' %}">Accueil</a>
            <a href="{% url 'dashboard' %}">Dashboard</a>
            <a href="{% url 'add_book' %}">Ajouter</a>
            
            {% if user.is_authenticated %}
                <span>Connecté: {{ user.username }}</span>
                <a href="{% url 'logout' %}">Déconnexion</a>
            {% else %}
                <a href="{% url 'login' %}">Connexion</a>
            {% endif %}
        </div>
    </nav>
    
    <!-- Contenu principal -->
    <main class="container">
        {% block content %}{% endblock %}
    </main>
    
    <!-- Pied de page -->
    <footer class="footer">
        <p>&copy; 2024 Bibliothèque en Ligne | Django + HTMX</p>
    </footer>
</body>
</html>
```

### books_list.html
```html
{% extends "base.html" %}

{% block title %}Tous les Livres{% endblock %}

{% block content %}

<div class="books-container">
    <h1>📚 Bibliothèque</h1>
    
    <!-- Barre de recherche (HTMX) -->
    <div class="search-bar">
        <input type="text"
               name="q"
               placeholder="Chercher un livre..."
               hx-get="{% url 'search_books' %}"
               hx-target="#search-results"
               hx-trigger="keyup changed delay:300ms"
               class="search-input">
    </div>
    
    <!-- Résultats de recherche (remplacés par HTMX) -->
    <div id="search-results"></div>
    
    <!-- Filtre par auteur -->
    <div class="filter-bar">
        {% for author in authors %}
            <a href="?author_id={{ author.id }}" class="filter-btn">
                {{ author.name }}
            </a>
        {% endfor %}
        <a href="{% url 'books_list' %}" class="filter-btn">Tous</a>
    </div>
    
    <!-- Bouton Ajouter -->
    <a href="{% url 'add_book' %}" class="btn btn-primary">
        ➕ Ajouter un Livre
    </a>
    
    <!-- Liste des livres -->
    <div class="books-grid">
        {% for book in books %}
            <div class="book-card">
                <h3>{{ book.title }}</h3>
                <p class="author">{{ book.author.name }}</p>
                <p class="year">📅 {{ book.published_year }}</p>
                
                <!-- Note -->
                <div class="rating">
                    ⭐ {{ book.rating }}/5
                </div>
                
                <!-- Boutons -->
                <div class="actions">
                    <a href="{% url 'book_detail' book.id %}" class="btn btn-small">
                        Détails
                    </a>
                </div>
            </div>
        {% empty %}
            <p>Aucun livre trouvé.</p>
        {% endfor %}
    </div>
    
    <!-- Statistiques -->
    <div class="stats">
        <p>Total: <strong>{{ total_count }}</strong> livre{{ total_count|pluralize }}</p>
    </div>
</div>

{% endblock %}
```

### book_detail.html
```html
{% extends "base.html" %}

{% block title %}{{ book.title }}{% endblock %}

{% block content %}

<div class="book-detail">
    <a href="{% url 'books_list' %}" class="btn-back">← Retour</a>
    
    <!-- Détails du livre -->
    <div class="book-info">
        <h1>{{ book.title }}</h1>
        <p class="author">
            Par <a href="?author_id={{ book.author.id }}">{{ book.author.name }}</a>
        </p>
        
        <!-- Métadonnées -->
        <div class="metadata">
            <p><strong>ISBN:</strong> {{ book.isbn }}</p>
            <p><strong>Année:</strong> {{ book.published_year }}</p>
        </div>
        
        <!-- Description -->
        {% if book.description %}
            <div class="description">
                <h2>Description</h2>
                <p>{{ book.description }}</p>
            </div>
        {% endif %}
        
        <!-- Bouton Like (HTMX) -->
        <div id="like-container">
            {% include 'library/like_button.html' %}
        </div>
        
        <!-- Bouton Ajouter Review -->
        {% if user.is_authenticated %}
            <a href="{% url 'add_review' book.id %}" class="btn btn-primary">
                ✍️ Laisser une Review
            </a>
        {% else %}
            <p><a href="{% url 'login' %}">Se connecter</a> pour laisser une review</p>
        {% endif %}
    </div>
    
    <!-- Reviews -->
    <section class="reviews">
        <h2>Reviews ({{ reviews_count }})</h2>
        
        {% if reviews %}
            <div id="reviews-container">
                {% for review in reviews %}
                    {% include 'library/review_item.html' %}
                {% endfor %}
            </div>
        {% else %}
            <p>Aucune review encore.</p>
        {% endif %}
    </section>
    
    <!-- Livres associés -->
    {% if related_books %}
        <section class="related">
            <h2>Autres Livres de {{ book.author.name }}</h2>
            <ul>
                {% for rel_book in related_books %}
                    <li>
                        <a href="{% url 'book_detail' rel_book.id %}">
                            {{ rel_book.title }}
                        </a>
                    </li>
                {% endfor %}
            </ul>
        </section>
    {% endif %}
</div>

{% endblock %}
```

---

## 6️⃣ ADMIN.PY - Interface d'Administration

```python
# library/admin.py

from django.contrib import admin
from .models import Author, Book, Review, BookLike

# Enregistrer les modèles dans l'admin

@admin.register(Author)
class AuthorAdmin(admin.ModelAdmin):
    """Configuration de l'admin pour les auteurs"""
    list_display = ('name', 'email', 'created_at')
    search_fields = ('name', 'email')
    readonly_fields = ('created_at',)

@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    """Configuration de l'admin pour les livres"""
    list_display = ('title', 'author', 'isbn', 'published_year', 'rating')
    list_filter = ('author', 'published_year')
    search_fields = ('title', 'isbn')
    readonly_fields = ('created_at',)

@admin.register(Review)
class ReviewAdmin(admin.ModelAdmin):
    """Configuration de l'admin pour les reviews"""
    list_display = ('user', 'book', 'rating', 'created_at')
    list_filter = ('rating', 'created_at')
    search_fields = ('user__username', 'book__title')
    readonly_fields = ('created_at',)

@admin.register(BookLike)
class BookLikeAdmin(admin.ModelAdmin):
    """Configuration de l'admin pour les likes"""
    list_display = ('user', 'book', 'created_at')
    search_fields = ('user__username', 'book__title')
    readonly_fields = ('created_at',)
```

---

## ✅ Résumé du Projet

```
REQUEST CLIENT
    ↓
urls.py (Routage)
    ↓
views.py (Logique métier)
    ├─ Récupère les données du REQUEST
    ├─ Consulte la BD (models.py)
    ├─ Valide les données (forms.py)
    ├─ Crée le contexte (dict)
    └─ Appelle le template
    ↓
templates/ (Présentation)
    ├─ Reçoit le contexte
    ├─ Affiche les données
    ├─ Utilise les tags Django
    └─ Génère du HTML
    ↓
RESPONSE HTML/JSON au navigateur
```

---

## 🎓 Checklist de Maîtrise

- ✅ Comprendre le cycle Request → View → Template → Response
- ✅ Créer et manipuler les models Django
- ✅ Utiliser les Django Forms pour la validation
- ✅ Afficher les données dans les templates
- ✅ Utiliser HTMX pour les interactions sans JavaScript
- ✅ Gérer les structures de données (listes, dicts, JSON, CSV)
- ✅ Construire une API JSON
- ✅ Exporter/Importer des données

**Tu es maintenant EXPERT en Django! 🚀**
