# 💪 EXERCICES PROGRESSIFS - Les Défis

## 🎯 Structure des Exercices

Chaque exercice:
- ✅ Construit sur les modules précédents
- ✅ Augmente graduellement en difficulté
- ✅ Inclut une solution complète
- ✅ Peut être fait seul ou avec un groupe

**Durée totale:** 20-30 heures (spread over a few days)

---

## 🏆 NIVEAU 1: LES FONDAMENTAUX (Modules 1-2)

### Exercice 1.1: Créer Ton Premier Model ⭐

**Difficulté:** ⭐ (Très facile)
**Durée:** 15 minutes
**Modules:** 1, 2

**Défi:**
Crée un model `Student` avec:
- `name` (CharField, max 100)
- `email` (EmailField, unique)
- `grade` (IntegerField, 1-12)
- `created_at` (DateTimeField, auto)

**Étapes:**
1. Crée le model dans `library/models.py`
2. Fais les migrations
3. Enregistre-le dans `admin.py`
4. Crée 3 étudiants dans l'admin

**Solution:**
```python
# models.py
class Student(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    grade = models.IntegerField(choices=[(i, i) for i in range(1, 13)])
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return f"{self.name} (Grade {self.grade})"

# admin.py
@admin.register(Student)
class StudentAdmin(admin.ModelAdmin):
    list_display = ('name', 'email', 'grade')
    search_fields = ('name', 'email')
```

---

### Exercice 1.2: Créer Une View Simple ⭐

**Difficulté:** ⭐ (Très facile)
**Durée:** 20 minutes
**Modules:** 2

**Défi:**
Crée une view qui:
1. Récupère tous les étudiants
2. Les trie par nom
3. Les passe au template

**Étapes:**
1. Crée la view `students_list()`
2. Crée l'URL
3. Crée le template HTML

**Solution:**
```python
# views.py
def students_list(request):
    students = Student.objects.all().order_by('name')
    return render(request, 'library/students_list.html', {
        'students': students,
        'count': students.count(),
    })

# urls.py
path('students/', views.students_list, name='students_list'),

# templates/library/students_list.html
{% extends "base.html" %}
{% block content %}
<h1>Étudiants ({{ count }})</h1>
<ul>
    {% for student in students %}
        <li>{{ student.name }} - Grade {{ student.grade }}</li>
    {% endfor %}
</ul>
{% endblock %}
```

---

### Exercice 1.3: Filtrer et Chercher ⭐⭐

**Difficulté:** ⭐⭐ (Facile)
**Durée:** 30 minutes
**Modules:** 2

**Défi:**
Ajoute à ta view `students_list`:
1. Filtrer par grade (?grade=10)
2. Chercher par nom (?q=alex)
3. Combiner les deux

**Étapes:**
1. Récupère les paramètres GET
2. Filtre la QuerySet
3. Affiche les résultats

**Solution:**
```python
def students_list(request):
    # Commencer avec tous les étudiants
    students = Student.objects.all()
    
    # Filtrer par grade
    grade = request.GET.get('grade')
    if grade:
        students = students.filter(grade=int(grade))
    
    # Chercher par nom
    query = request.GET.get('q')
    if query:
        students = students.filter(name__icontains=query)
    
    # Trier
    students = students.order_by('name')
    
    return render(request, 'library/students_list.html', {
        'students': students,
        'count': students.count(),
        'query': query,
        'grade': grade,
    })
```

---

## 🏆 NIVEAU 2: FORMULAIRES & VALIDATION (Module 3)

### Exercice 2.1: Créer Un Formulaire d'Étudiant ⭐⭐

**Difficulté:** ⭐⭐ (Facile)
**Durée:** 30 minutes
**Modules:** 3

**Défi:**
1. Crée une `StudentForm` avec ModelForm
2. Ajoute une validation personnalisée pour l'email
3. Crée une view pour ajouter un étudiant

**Étapes:**
1. Crée la form dans `forms.py`
2. Crée la view `add_student()`
3. Crée le template avec le formulaire
4. Teste l'ajout

**Solution:**
```python
# forms.py
class StudentForm(forms.ModelForm):
    class Meta:
        model = Student
        fields = ['name', 'email', 'grade']
        widgets = {
            'name': forms.TextInput(attrs={'class': 'form-control'}),
            'email': forms.EmailInput(attrs={'class': 'form-control'}),
            'grade': forms.Select(attrs={'class': 'form-control'}),
        }
    
    # Validation personnalisée
    def clean_email(self):
        email = self.cleaned_data.get('email')
        # Vérifier que l'email n'existe pas déjà
        if Student.objects.filter(email=email).exists():
            raise forms.ValidationError('Cet email existe déjà!')
        return email

# views.py
def add_student(request):
    if request.method == 'POST':
        form = StudentForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('students_list')
    else:
        form = StudentForm()
    
    return render(request, 'library/add_student.html', {'form': form})

# urls.py
path('students/add/', views.add_student, name='add_student'),
```

---

### Exercice 2.2: Modifier et Supprimer ⭐⭐⭐

**Difficulté:** ⭐⭐⭐ (Moyen)
**Durée:** 45 minutes
**Modules:** 2, 3

**Défi:**
Crée deux views:
1. `edit_student(id)` - Modifier un étudiant
2. `delete_student(id)` - Supprimer un étudiant

**Étapes:**
1. Crée les views
2. Crée les templates
3. Ajoute les liens dans la liste

**Solution:**
```python
# views.py
def edit_student(request, student_id):
    student = get_object_or_404(Student, id=student_id)
    
    if request.method == 'POST':
        form = StudentForm(request.POST, instance=student)
        if form.is_valid():
            form.save()
            return redirect('students_list')
    else:
        form = StudentForm(instance=student)
    
    return render(request, 'library/edit_student.html', {
        'form': form,
        'student': student,
    })

def delete_student(request, student_id):
    student = get_object_or_404(Student, id=student_id)
    
    if request.method == 'POST':
        student.delete()
        return redirect('students_list')
    
    return render(request, 'library/confirm_delete.html', {
        'student': student,
    })

# urls.py
path('students/<int:student_id>/edit/', views.edit_student, name='edit_student'),
path('students/<int:student_id>/delete/', views.delete_student, name='delete_student'),
```

---

## 🏆 NIVEAU 3: TEMPLATES & AFFICHAGE (Module 4)

### Exercice 3.1: Template Complet ⭐⭐

**Difficulté:** ⭐⭐ (Facile)
**Durée:** 30 minutes
**Modules:** 4

**Défi:**
Crée un template `student_detail.html` qui affiche:
1. Tous les détails de l'étudiant
2. Une liste de tous les autres étudiants de même grade
3. Des boutons Modifier/Supprimer

**Étapes:**
1. Crée la view `student_detail()`
2. Crée le template complet
3. Ajoute les styles CSS

**Solution:**
```python
# views.py
def student_detail(request, student_id):
    student = get_object_or_404(Student, id=student_id)
    
    # Récupérer les autres étudiants du même grade
    classmates = Student.objects.filter(
        grade=student.grade
    ).exclude(id=student.id)
    
    return render(request, 'library/student_detail.html', {
        'student': student,
        'classmates': classmates,
    })

# urls.py
path('students/<int:student_id>/', views.student_detail, name='student_detail'),
```

```html
<!-- templates/library/student_detail.html -->
{% extends "base.html" %}

{% block content %}
<div class="student-detail">
    <a href="{% url 'students_list' %}">← Retour</a>
    
    <h1>{{ student.name }}</h1>
    <p>Email: {{ student.email }}</p>
    <p>Grade: {{ student.grade }}</p>
    <p>Créé: {{ student.created_at|date:"d/m/Y" }}</p>
    
    <div class="actions">
        <a href="{% url 'edit_student' student.id %}" class="btn">Modifier</a>
        <a href="{% url 'delete_student' student.id %}" class="btn btn-danger">Supprimer</a>
    </div>
    
    {% if classmates %}
    <h2>Camarades de classe ({{ classmates|length }})</h2>
    <ul>
        {% for mate in classmates %}
        <li>
            <a href="{% url 'student_detail' mate.id %}">
                {{ mate.name }}
            </a>
        </li>
        {% endfor %}
    </ul>
    {% endif %}
</div>
{% endblock %}
```

---

## 🏆 NIVEAU 4: HTMX & INTERACTIVITÉ (Module 5)

### Exercice 4.1: Recherche en Temps Réel ⭐⭐⭐

**Difficulté:** ⭐⭐⭐ (Moyen)
**Durée:** 45 minutes
**Modules:** 5

**Défi:**
Crée une recherche qui:
1. Se déclenche à chaque caractère tapé
2. Affiche les résultats en temps réel (HTMX)
3. Pas de recharge de page

**Étapes:**
1. Crée une view `search_students()` qui retourne JUSTE les résultats
2. Crée un template pour les résultats
3. Ajoute HTMX dans l'input
4. Teste avec plusieurs requêtes

**Solution:**
```python
# views.py
def search_students(request):
    query = request.GET.get('q', '').strip()
    
    if query:
        students = Student.objects.filter(
            name__icontains=query
        ).order_by('name')
    else:
        students = Student.objects.none()
    
    # Retourner JUSTE les résultats (pas la page entière!)
    return render(request, 'library/search_results.html', {
        'students': students,
        'query': query,
    })

# urls.py
path('students/search/', views.search_students, name='search_students'),
```

```html
<!-- templates/library/search_input.html -->
<input type="text"
       placeholder="Chercher un étudiant..."
       hx-get="{% url 'search_students' %}"
       hx-target="#search-results"
       hx-trigger="keyup changed delay:300ms"
       class="search-input">

<div id="search-results"></div>

<!-- templates/library/search_results.html -->
{% if students %}
    <ul class="results">
    {% for student in students %}
        <li>
            <a href="{% url 'student_detail' student.id %}">
                {{ student.name }} - Grade {{ student.grade }}
            </a>
        </li>
    {% endfor %}
    </ul>
{% elif query %}
    <p>Aucun résultat pour "{{ query }}"</p>
{% else %}
    <p>Commence à taper...</p>
{% endif %}
```

---

### Exercice 4.2: Like/Vote avec HTMX ⭐⭐⭐

**Difficulté:** ⭐⭐⭐ (Moyen)
**Durée:** 1 heure
**Modules:** 5, 2

**Défi:**
Crée un système de "votes" pour les étudiants:
1. Chaque utilisateur peut voter pour un étudiant
2. Afficher le nombre de votes
3. Mettre à jour en temps réel avec HTMX

**Étapes:**
1. Crée un model `StudentVote`
2. Crée une view `toggle_vote()`
3. Crée un template pour le bouton
4. Teste avec plusieurs votes

**Solution:**
```python
# models.py
class StudentVote(models.Model):
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        unique_together = ('student', 'user')

# views.py
@login_required
def toggle_vote(request, student_id):
    student = get_object_or_404(Student, id=student_id)
    
    vote = StudentVote.objects.filter(
        student=student,
        user=request.user
    ).first()
    
    if vote:
        vote.delete()
        voted = False
    else:
        StudentVote.objects.create(
            student=student,
            user=request.user
        )
        voted = True
    
    votes_count = student.studentvote_set.count()
    
    return render(request, 'library/vote_button.html', {
        'student': student,
        'voted': voted,
        'votes_count': votes_count,
    })

# urls.py
path('students/<int:student_id>/vote/', views.toggle_vote, name='toggle_vote'),
```

```html
<!-- templates/library/vote_button.html -->
<div id="vote-btn-{{ student.id }}">
    {% if voted %}
    <button hx-post="{% url 'toggle_vote' student.id %}"
            hx-target="#vote-btn-{{ student.id }}"
            hx-swap="innerHTML"
            class="btn-voted">
        👍 {{ votes_count }}
    </button>
    {% else %}
    <button hx-post="{% url 'toggle_vote' student.id %}"
            hx-target="#vote-btn-{{ student.id }}"
            hx-swap="innerHTML"
            class="btn-not-voted">
        👋 {{ votes_count }}
    </button>
    {% endif %}
</div>
```

---

## 🏆 NIVEAU 5: DONNÉES AVANCÉES (Module 6)

### Exercice 5.1: Exporter en CSV ⭐⭐

**Difficulté:** ⭐⭐ (Facile)
**Durée:** 30 minutes
**Modules:** 6

**Défi:**
Crée un bouton qui exporte tous les étudiants en CSV avec:
- ID, Nom, Email, Grade, Date de création

**Étapes:**
1. Crée la view `export_students_csv()`
2. Ajoute un lien dans le template
3. Teste le téléchargement

**Solution:**
```python
# views.py
import csv
from django.http import HttpResponse

def export_students_csv(request):
    students = Student.objects.all()
    
    response = HttpResponse(content_type='text/csv')
    response['Content-Disposition'] = 'attachment; filename="students.csv"'
    
    writer = csv.DictWriter(response, fieldnames=['id', 'name', 'email', 'grade', 'created_at'])
    writer.writeheader()
    
    for student in students:
        writer.writerow({
            'id': student.id,
            'name': student.name,
            'email': student.email,
            'grade': student.grade,
            'created_at': student.created_at,
        })
    
    return response

# urls.py
path('students/export/csv/', views.export_students_csv, name='export_students_csv'),
```

```html
<!-- Dans students_list.html -->
<a href="{% url 'export_students_csv' %}" class="btn btn-primary">
    📥 Exporter CSV
</a>
```

---

### Exercice 5.2: Importer un CSV ⭐⭐⭐

**Difficulté:** ⭐⭐⭐ (Moyen)
**Durée:** 1 heure
**Modules:** 6, 3

**Défi:**
Crée une page qui:
1. Accepte un fichier CSV
2. Importe les étudiants
3. Affiche un résumé (nombre importés, erreurs)

**Étapes:**
1. Crée un formulaire d'upload
2. Crée la view `import_students_csv()`
3. Parse le CSV et crée les étudiants
4. Affiche les résultats

**Solution:**
```python
# views.py
def import_students_csv(request):
    if request.method == 'POST':
        csv_file = request.FILES.get('file')
        
        decoded_file = csv_file.read().decode('utf-8').splitlines()
        reader = csv.DictReader(decoded_file)
        
        count = 0
        errors = []
        
        for row in reader:
            try:
                Student.objects.create(
                    name=row['name'],
                    email=row['email'],
                    grade=int(row['grade']),
                )
                count += 1
            except Exception as e:
                errors.append(f"Ligne {count + 1}: {str(e)}")
        
        return render(request, 'library/import_result.html', {
            'imported': count,
            'errors': errors,
        })
    
    return render(request, 'library/import_form.html')

# urls.py
path('students/import/csv/', views.import_students_csv, name='import_students_csv'),
```

---

## 🏆 NIVEAU 6: PROJET FINAL (All Modules)

### Exercice 6.1: Tableau de Bord (Dashboard) ⭐⭐⭐⭐

**Difficulté:** ⭐⭐⭐⭐ (Difficile)
**Durée:** 2-3 heures
**Modules:** 1-7

**Défi:**
Crée un dashboard complet qui affiche:
1. Nombre total d'étudiants
2. Nombre par grade (graphique data)
3. Les 5 étudiants les plus votés
4. Statistiques par grade (moyennes)
5. Recherche rapide (HTMX)

**Étapes:**
1. Crée la view `dashboard()`
2. Récupère les statistiques
3. Crée le template avec graphique
4. Ajoute du CSS pour le design

**Solution:**
```python
# views.py
from django.db.models import Count, Avg

def dashboard(request):
    students = Student.objects.all()
    
    # Statistiques générales
    stats = {
        'total': students.count(),
        'avg_grade': students.aggregate(Avg('grade'))['grade__avg'],
    }
    
    # Compter par grade
    by_grade = {}
    for i in range(1, 13):
        by_grade[i] = students.filter(grade=i).count()
    
    # Étudiants les plus votés
    top_students = students.annotate(
        votes=Count('studentvote')
    ).order_by('-votes')[:5]
    
    context = {
        'stats': stats,
        'by_grade': by_grade,
        'top_students': top_students,
    }
    
    return render(request, 'library/dashboard.html', context)

# urls.py
path('dashboard/', views.dashboard, name='dashboard'),
```

```html
<!-- templates/library/dashboard.html -->
{% extends "base.html" %}

{% block content %}
<div class="dashboard">
    <h1>📊 Dashboard</h1>
    
    <!-- Stats Cards -->
    <div class="stats-cards">
        <div class="card">
            <h3>Total Étudiants</h3>
            <p class="big-number">{{ stats.total }}</p>
        </div>
        <div class="card">
            <h3>Moyenne Grade</h3>
            <p class="big-number">{{ stats.avg_grade|floatformat:1 }}</p>
        </div>
    </div>
    
    <!-- Top Voted -->
    <section class="top-students">
        <h2>⭐ Top Étudiants</h2>
        <ol>
        {% for student in top_students %}
            <li>
                {{ student.name }} - {{ student.votes }} votes
            </li>
        {% endfor %}
        </ol>
    </section>
</div>
{% endblock %}
```

---

## 📝 Réponse Aux Questions Fréquentes

### "Comment déboguer ma view?"
```python
def my_view(request):
    data = Student.objects.all()
    print(f"Data: {data}")  # Affiche dans la console
    print(f"Type: {type(data)}")
    print(f"Count: {data.count()}")
    return render(request, 'template.html', {'data': data})
```

### "Comment vérifier si HTMX fonctionne?"
```python
def my_view(request):
    # Vérifier si c'est une requête HTMX
    if request.headers.get('HX-Request'):
        # Retourner juste le fragment
        return render(request, 'fragment.html', {})
    else:
        # Retourner la page entière
        return render(request, 'full_page.html', {})
```

### "Comment déboguer un template?"
```html
<!-- Afficher une variable -->
{{ student }}

<!-- Afficher le type -->
{{ student|default:"N/A" }}

<!-- Afficher tous les contextes -->
{% for key, value in context.items %}
    {{ key }}: {{ value }}
{% endfor %}
```

---

## 🎯 Checklist de Complétion

### Niveau 1
- ⬜ Exercice 1.1: Model créé
- ⬜ Exercice 1.2: View simple
- ⬜ Exercice 1.3: Filtrage et recherche

### Niveau 2
- ⬜ Exercice 2.1: Formulaire créé
- ⬜ Exercice 2.2: Modifier/Supprimer

### Niveau 3
- ⬜ Exercice 3.1: Template complet

### Niveau 4
- ⬜ Exercice 4.1: Recherche HTMX
- ⬜ Exercice 4.2: Vote/Like HTMX

### Niveau 5
- ⬜ Exercice 5.1: Export CSV
- ⬜ Exercice 5.2: Import CSV

### Niveau 6
- ⬜ Exercice 6.1: Dashboard complet

---

## 🚀 Prochaines Étapes Après Ces Exercices

1. **Ajouter une authentification avancée**
   - Système de permissions
   - Groupes d'utilisateurs

2. **Ajouter des images**
   - Upload de photos d'étudiants
   - Stockage des fichiers

3. **Ajouter une API**
   - Django REST Framework
   - Endpoints JSON

4. **Ajouter des tests**
   - Tests unitaires
   - Tests d'intégration

5. **Déployer en production**
   - Heroku, DigitalOcean, AWS
   - Configuration de la base de données

---

**Bravo d'avoir complété les exercices! Tu es maintenant un développeur Django! 🏆**
