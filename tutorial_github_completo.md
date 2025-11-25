# Tutorial Completo de GitHub: Flujo de Trabajo Profesional

## Tabla de Contenidos
1. [Configuración Inicial](#configuración-inicial)
2. [Trabajando con Ramas (Branches)](#trabajando-con-ramas)
3. [Políticas de Protección de Ramas](#políticas-de-protección)
4. [Pull Requests (PRs)](#pull-requests)
5. [Code Review y Aprobaciones](#code-review-y-aprobaciones)
6. [Mejores Prácticas](#mejores-prácticas)
7. [Flujos de Trabajo Comunes](#flujos-de-trabajo)

---

## 1. Configuración Inicial

### Configurar Git localmente

```bash
# Configurar nombre y email
git config --global user.name "Tu Nombre"
git config --global user.email "tu.email@ejemplo.com"

# Configurar editor predeterminado
git config --global core.editor "code --wait"  # Para VS Code

# Ver configuración
git config --list
```

### Clonar un repositorio

```bash
# HTTPS
git clone https://github.com/usuario/repositorio.git

# SSH (recomendado)
git clone git@github.com:usuario/repositorio.git
```

### Configurar SSH (recomendado para autenticación)

```bash
# Generar llave SSH
ssh-keygen -t ed25519 -C "tu.email@ejemplo.com"

# Agregar la llave al ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Copiar llave pública y agregarla en GitHub Settings > SSH Keys
cat ~/.ssh/id_ed25519.pub
```

---

## 2. Trabajando con Ramas (Branches)

### Estrategia de Branching

**Ramas principales:**
- `main` o `master`: Código en producción
- `develop`: Rama de desarrollo activo
- `staging`: Rama de pre-producción (opcional)

**Ramas de trabajo:**
- `feature/*`: Nuevas funcionalidades
- `bugfix/*`: Corrección de bugs
- `hotfix/*`: Correcciones urgentes en producción
- `release/*`: Preparación de releases

### Comandos Básicos de Ramas

```bash
# Ver todas las ramas
git branch -a

# Crear una nueva rama
git branch nombre-rama

# Cambiar a una rama
git checkout nombre-rama

# Crear y cambiar a una rama (atajo)
git checkout -b feature/nueva-funcionalidad

# Crear rama desde una rama específica
git checkout -b feature/mi-feature develop

# Renombrar rama actual
git branch -m nuevo-nombre

# Eliminar rama local
git branch -d nombre-rama

# Eliminar rama remota
git push origin --delete nombre-rama

# Ver ramas remotas
git branch -r
```

### Convención de Nombres de Ramas

```bash
# Ejemplos de buenos nombres
feature/user-authentication
feature/add-payment-gateway
bugfix/fix-login-error
hotfix/critical-security-patch
release/v1.2.0

# Evitar nombres como:
nueva-cosa
fix
temp
test123
```

### Sincronizar con el Remoto

```bash
# Actualizar referencias remotas
git fetch origin

# Traer cambios de la rama remota
git pull origin develop

# Subir tu rama al remoto
git push origin feature/mi-feature

# Establecer upstream (primera vez)
git push -u origin feature/mi-feature
```

---

## 3. Políticas de Protección de Ramas

### Configurar Protección en GitHub

**Pasos en la UI de GitHub:**
1. Ve a tu repositorio
2. Settings → Branches
3. Add branch protection rule
4. Especifica el patrón de rama (ej: `main`, `develop`)

### Reglas Recomendadas de Protección

#### Para la rama `main`:

**Requerimientos básicos:**
- ✅ Require a pull request before merging
  - Require approvals: 2 revisores mínimo
  - Dismiss stale pull request approvals when new commits are pushed
  - Require review from Code Owners

**Verificaciones de estado:**
- ✅ Require status checks to pass before merging
  - Require branches to be up to date before merging
  - Status checks específicos:
    - CI/CD pipeline
    - Tests unitarios
    - Linting
    - Security scans

**Restricciones adicionales:**
- ✅ Require conversation resolution before merging
- ✅ Require signed commits (opcional pero recomendado)
- ✅ Require linear history (evita merge commits)
- ✅ Restrict who can push to matching branches
  - Solo administradores o usuarios específicos

**Otras opciones:**
- ✅ Do not allow bypassing the above settings
- ✅ Allow force pushes: Never
- ✅ Allow deletions: Never

#### Para la rama `develop`:

Similar a `main` pero con menos restricciones:
- Require approvals: 1 revisor
- Menos estricto con force pushes (solo para administradores)

### Configurar CODEOWNERS

Crea un archivo `.github/CODEOWNERS`:

```bash
# Ejemplo de CODEOWNERS

# Usuarios por defecto para todo el repo
* @owner-principal @equipo-dev

# Directorio específico
/docs/ @equipo-documentacion

# Tipo de archivo específico
*.py @equipo-backend
*.js @equipo-frontend
*.md @tech-writer

# Múltiples archivos
package.json @senior-dev @tech-lead
requirements.txt @senior-dev @tech-lead

# Carpetas críticas
/src/auth/ @security-team
/src/payments/ @senior-dev @security-team
```

---

## 4. Pull Requests (PRs)

### Crear un Pull Request

#### Desde la línea de comandos:

```bash
# 1. Asegúrate de estar en tu rama de feature
git checkout feature/mi-feature

# 2. Actualiza tu rama con los últimos cambios
git fetch origin
git rebase origin/develop  # o git merge origin/develop

# 3. Resuelve conflictos si existen
git add .
git rebase --continue

# 4. Push de tu rama
git push origin feature/mi-feature

# 5. Crea el PR desde GitHub UI o usando GitHub CLI
gh pr create --base develop --head feature/mi-feature --title "Título del PR" --body "Descripción"
```

#### Desde GitHub UI:

1. Ve a tu repositorio en GitHub
2. Click en "Pull requests"
3. Click en "New pull request"
4. Selecciona base: `develop` y compare: `feature/mi-feature`
5. Click en "Create pull request"

### Template de Pull Request

Crea `.github/pull_request_template.md`:

```markdown
## 📋 Descripción

<!-- Describe qué cambios introduce este PR -->

## 🎯 Tipo de Cambio

- [ ] 🐛 Bug fix (cambio que corrige un issue)
- [ ] ✨ Nueva feature (cambio que añade funcionalidad)
- [ ] 💥 Breaking change (fix o feature que causa que funcionalidad existente cambie)
- [ ] 📝 Documentación
- [ ] 🎨 Cambios de estilo (formato, punto y coma, etc)
- [ ] ♻️ Refactoring
- [ ] ⚡ Performance
- [ ] ✅ Tests

## 🔗 Issues Relacionados

Closes #(issue)

## 📸 Screenshots (si aplica)

## ✅ Checklist

- [ ] Mi código sigue las guías de estilo del proyecto
- [ ] He realizado una auto-revisión de mi código
- [ ] He comentado mi código, especialmente en áreas difíciles
- [ ] He actualizado la documentación correspondiente
- [ ] Mis cambios no generan nuevos warnings
- [ ] He añadido tests que prueban que mi fix funciona o que mi feature funciona
- [ ] Tests unitarios nuevos y existentes pasan localmente
- [ ] Cambios dependientes han sido merged y publicados

## 🧪 Cómo Probar

<!-- Pasos para probar los cambios -->

1. 
2. 
3. 

## 📝 Notas Adicionales

<!-- Cualquier información adicional relevante -->
```

### Mejores Prácticas para PRs

**Tamaño del PR:**
- **Ideal:** 200-400 líneas de código
- **Máximo recomendado:** 500 líneas
- Si es más grande, considera dividirlo en múltiples PRs

**Commits:**
```bash
# Commits atómicos y descriptivos
git commit -m "feat: add user authentication endpoint"
git commit -m "test: add unit tests for auth service"
git commit -m "docs: update API documentation"

# Convención de commits (Conventional Commits)
# <tipo>(<scope>): <descripción>
#
# Tipos comunes:
# feat: nueva funcionalidad
# fix: corrección de bug
# docs: cambios en documentación
# style: formato, punto y coma, etc
# refactor: refactorización de código
# test: añadir o corregir tests
# chore: cambios en build, CI, etc
```

**Squash commits antes de merge (opcional):**
```bash
# Combinar últimos 3 commits
git rebase -i HEAD~3

# En el editor, cambiar 'pick' a 'squash' para los commits que quieres combinar
```

---

## 5. Code Review y Aprobaciones

### Proceso de Code Review

**Para el Autor del PR:**

1. **Auto-revisión primero**
   - Lee tu propio código en GitHub antes de pedir review
   - Verifica que todo está limpio y documentado

2. **Asignar reviewers**
   - Asigna a 1-2 personas específicas
   - Considera experiencia y carga de trabajo

3. **Responder a comentarios**
   - Responde a todos los comentarios
   - Haz cambios solicitados
   - Explica si no estás de acuerdo (respetuosamente)

4. **Re-solicitar review**
   - Después de hacer cambios, pide nueva revisión

**Para el Reviewer:**

```markdown
### Qué revisar:

✅ **Funcionalidad:**
- ¿El código hace lo que dice que hace?
- ¿Hay casos edge no considerados?
- ¿Funcionará en producción?

✅ **Código:**
- ¿Es legible y mantenible?
- ¿Sigue las convenciones del proyecto?
- ¿Hay código duplicado?
- ¿Es eficiente?

✅ **Tests:**
- ¿Hay tests adecuados?
- ¿Cubren casos importantes?
- ¿Los tests son claros?

✅ **Seguridad:**
- ¿Hay vulnerabilidades obvias?
- ¿Se validan inputs?
- ¿Se manejan errores correctamente?

✅ **Documentación:**
- ¿Está documentado lo complejo?
- ¿Se actualizó la documentación externa?
```

### Tipos de Comentarios

**Comentarios constructivos:**

```markdown
# ❌ Mal ejemplo
"Esto está mal"

# ✅ Buen ejemplo
"Considera usar un try-catch aquí para manejar el caso cuando la API falle. 
Ejemplo: 
```python
try:
    response = api.call()
except APIError as e:
    logger.error(f"API call failed: {e}")
    return default_value
```
```

**Niveles de comentarios:**

```markdown
# 🔴 Bloqueante (Must fix)
**[BLOCKER]** Este código tiene un memory leak. Necesita ser corregido antes de merge.

# 🟡 Importante (Should fix)
**[IMPORTANT]** Esta validación debería estar aquí para prevenir errores futuros.

# 🟢 Sugerencia (Nice to have)
**[NIT]** Considera renombrar esta variable a algo más descriptivo.

# 💡 Pregunta
**[QUESTION]** ¿Por qué elegiste este approach? ¿Consideraste usar X?

# 👍 Aprobación
**[LGTM]** Looks good to me! Buen trabajo con los tests.
```

### GitHub CLI para Reviews

```bash
# Instalar GitHub CLI
# brew install gh  (macOS)
# apt install gh   (Ubuntu)

# Autenticarse
gh auth login

# Ver PRs pendientes de review
gh pr list --search "review-requested:@me"

# Ver un PR específico
gh pr view 123

# Checkout a un PR localmente
gh pr checkout 123

# Hacer review
gh pr review 123 --approve -b "LGTM! Buen trabajo"
gh pr review 123 --request-changes -b "Por favor corrige X antes de merge"
gh pr review 123 --comment -b "Algunos comentarios adicionales"

# Mergear PR
gh pr merge 123 --squash --delete-branch
```

---

## 6. Mejores Prácticas

### Estrategia de Branching: Git Flow Simplificado

```
main (producción)
  ↑
  │ merge solo vía PR
  │
develop (desarrollo activo)
  ↑
  ├── feature/user-auth
  ├── feature/payment-integration
  ├── bugfix/login-error
  └── hotfix/critical-security-fix → merge directo a main + develop
```

### Workflow Diario Recomendado

```bash
# Inicio del día
git checkout develop
git pull origin develop

# Crear nueva feature
git checkout -b feature/nombre-descriptivo

# Trabajar en tu feature
# ... hacer cambios ...
git add .
git commit -m "feat: descripción clara del cambio"

# Actualizar con cambios del equipo (hazlo frecuentemente)
git fetch origin
git rebase origin/develop  # o git merge origin/develop

# Resolver conflictos si existen
# ... resolver conflictos ...
git add .
git rebase --continue

# Push a tu rama
git push origin feature/nombre-descriptivo

# Cuando termines, crear PR en GitHub
```

### Manejo de Conflictos

```bash
# Método 1: Rebase (historia más limpia)
git fetch origin
git rebase origin/develop

# Si hay conflictos
# 1. Resolver conflictos en los archivos
# 2. Marcar como resueltos
git add archivo-resuelto.py
# 3. Continuar rebase
git rebase --continue

# Método 2: Merge (más seguro)
git fetch origin
git merge origin/develop

# Resolver conflictos y commit
git add .
git commit -m "Merge develop into feature/mi-feature"
```

### Commits Semánticos (Conventional Commits)

```bash
# Formato: <tipo>(<scope>): <descripción>

# Ejemplos:
feat(auth): add OAuth2 authentication
fix(api): resolve timeout issue in user endpoint
docs(readme): update installation instructions
style(css): format according to style guide
refactor(database): optimize query performance
test(auth): add integration tests for login
chore(deps): update dependencies to latest versions
perf(api): improve response time by caching

# Con breaking change:
feat(api)!: redesign REST endpoints

BREAKING CHANGE: All endpoints now require authentication header
```

### .gitignore Esencial

```bash
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
.venv/
*.egg-info/
dist/
build/

# Node
node_modules/
npm-debug.log
yarn-error.log
.env

# IDE
.vscode/
.idea/
*.swp
*.swo
.DS_Store

# Jupyter
.ipynb_checkpoints/
*.ipynb_checkpoints

# Logs
*.log
logs/

# Secrets
.env
.env.local
secrets/
*.key
*.pem

# Database
*.db
*.sqlite3

# OS
Thumbs.db
.DS_Store
```

### GitHub Actions (CI/CD) Básico

Crea `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [ develop, main ]
  pull_request:
    branches: [ develop, main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.10'
    
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install pytest flake8 black
    
    - name: Lint with flake8
      run: |
        flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
    
    - name: Check formatting with black
      run: |
        black --check .
    
    - name: Run tests
      run: |
        pytest tests/ -v --cov=src/
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
```

---

## 7. Flujos de Trabajo Comunes

### Scenario 1: Nueva Feature

```bash
# 1. Actualizar develop
git checkout develop
git pull origin develop

# 2. Crear branch de feature
git checkout -b feature/user-notifications

# 3. Desarrollar la feature (múltiples commits)
# ... trabajo ...
git add .
git commit -m "feat: add notification model"
# ... más trabajo ...
git commit -m "feat: implement notification service"
# ... más trabajo ...
git commit -m "test: add notification tests"

# 4. Mantener actualizado (rebase frecuentemente)
git fetch origin
git rebase origin/develop

# 5. Push a remoto
git push origin feature/user-notifications

# 6. Crear PR en GitHub (desde UI o CLI)
gh pr create --base develop --head feature/user-notifications \
  --title "Add user notification system" \
  --body "Implements real-time notifications for users"

# 7. Después de aprobación y merge, limpiar
git checkout develop
git pull origin develop
git branch -d feature/user-notifications
```

### Scenario 2: Hotfix Urgente

```bash
# 1. Crear hotfix desde main
git checkout main
git pull origin main
git checkout -b hotfix/critical-security-patch

# 2. Hacer el fix
# ... código ...
git add .
git commit -m "fix: patch XSS vulnerability in user input"

# 3. Push
git push origin hotfix/critical-security-patch

# 4. Crear PR a main (con aprobación rápida)
gh pr create --base main --head hotfix/critical-security-patch \
  --title "HOTFIX: Critical security patch"

# 5. Después de merge a main, también merge a develop
git checkout develop
git pull origin develop
git merge main
git push origin develop

# 6. Limpiar
git branch -d hotfix/critical-security-patch
```

### Scenario 3: Sincronizar Fork

```bash
# 1. Agregar upstream (solo una vez)
git remote add upstream https://github.com/original-owner/repo.git

# 2. Fetch upstream
git fetch upstream

# 3. Merge cambios del upstream
git checkout main
git merge upstream/main

# 4. Push a tu fork
git push origin main
```

### Scenario 4: Revisar PR localmente

```bash
# Método 1: GitHub CLI
gh pr checkout 123

# Método 2: Manual
git fetch origin pull/123/head:pr-123
git checkout pr-123

# Probar los cambios
# ... pruebas ...

# Dejar comentarios
gh pr review 123 --comment -b "Funciona bien en mi máquina"
```

---

## 📚 Recursos Adicionales

### Comandos Útiles de Git

```bash
# Ver historial gráfico
git log --graph --oneline --all

# Ver cambios antes de commit
git diff

# Ver cambios staged
git diff --staged

# Deshacer último commit (mantener cambios)
git reset --soft HEAD~1

# Deshacer último commit (descartar cambios)
git reset --hard HEAD~1

# Guardar cambios temporalmente
git stash
git stash pop

# Ver quién modificó cada línea
git blame archivo.py

# Buscar en el historial
git log -S "texto a buscar"

# Cherry-pick un commit específico
git cherry-pick <commit-hash>
```

### Alias Útiles

Agrega a `~/.gitconfig`:

```bash
[alias]
    st = status
    co = checkout
    br = branch
    ci = commit
    unstage = reset HEAD --
    last = log -1 HEAD
    visual = log --graph --oneline --all --decorate
    amend = commit --amend --no-edit
    pushf = push --force-with-lease
    recent = branch --sort=-committerdate
```

### Enlaces Importantes

- **Git Documentation:** https://git-scm.com/doc
- **GitHub Docs:** https://docs.github.com/
- **GitHub CLI:** https://cli.github.com/
- **Conventional Commits:** https://www.conventionalcommits.org/
- **Git Flow:** https://nvie.com/posts/a-successful-git-branching-model/
- **Semantic Versioning:** https://semver.org/

---

## 🎯 Checklist de Configuración para Nuevo Proyecto

```markdown
### Setup Inicial
- [ ] Crear repositorio en GitHub
- [ ] Clonar localmente
- [ ] Configurar .gitignore
- [ ] Crear README.md
- [ ] Configurar LICENSE

### Protección de Ramas
- [ ] Proteger rama main
  - [ ] Require PR before merging
  - [ ] Require 2 approvals
  - [ ] Require status checks
  - [ ] Restrict force push
- [ ] Proteger rama develop
  - [ ] Require PR before merging
  - [ ] Require 1 approval

### Documentación
- [ ] Crear CONTRIBUTING.md
- [ ] Crear CODEOWNERS
- [ ] Crear PR template
- [ ] Crear issue templates

### CI/CD
- [ ] Configurar GitHub Actions
- [ ] Configurar tests automáticos
- [ ] Configurar linting
- [ ] Configurar deployment

### Equipo
- [ ] Invitar colaboradores
- [ ] Asignar roles
- [ ] Documentar proceso de desarrollo
- [ ] Realizar kickoff meeting
```

---

## 💡 Consejos Finales

1. **Commitea frecuentemente**: Commits pequeños y frecuentes son mejor que uno grande
2. **Pull antes de push**: Siempre actualiza tu rama antes de subir cambios
3. **Escribe buenos mensajes de commit**: Tu yo del futuro te lo agradecerá
4. **Revisa tu propio código primero**: Auto-review antes de pedir review a otros
5. **Sé respetuoso en reviews**: Crítica constructiva, no destructiva
6. **Mantén las ramas actualizadas**: Rebase/merge frecuentemente desde develop
7. **No hagas commit de secretos**: Usa .env y .gitignore
8. **Documenta decisiones importantes**: En el código o en el PR
9. **Automatiza lo que puedas**: CI/CD, linting, tests
10. **Comunica con tu equipo**: Especialmente para cambios grandes

---

**Última actualización:** Noviembre 2024
**Versión:** 1.0
