# Conventional Branch: convención para nombrar ramas de Git

> Basado en la especificación **Conventional Branch 1.1.0**: <https://conventionalbranch.org/>

## ¿Qué es?

**Conventional Branch** es una convención (un acuerdo) para **poner nombre a las ramas de Git** siempre de la misma forma.

Se inspira en [Conventional Commits](<Conventional Commits.md>):

| Convención | Qué ordena | Ejemplo |
|---|---|---|
| **Conventional Commits** | Los mensajes de *commit* | `feat: añadir página de login` |
| **Conventional Branch** | Los nombres de las ramas | `feature/add-login-page` |

> **Analogía:** si un repositorio fuese un edificio, las ramas serían las habitaciones. Poner un cartel claro en cada puerta («Cocina», «Baño») permite saber qué hay dentro sin tener que entrar.

## ¿Por qué usarla?

- **Se entiende de un vistazo** para qué sirve cada rama: `fix/header-bug` corrige un error; `feature/add-cart` añade algo nuevo.
- **Todo el equipo nombra igual**, así no hay ramas como `prueba`, `cambios-victor` o `nueva2`.
- **Facilita la automatización**: herramientas de integración continua (CI/CD) pueden actuar según el prefijo (por ejemplo, desplegar automáticamente las ramas `release/`).
- **Se enlaza con las tareas** del gestor de tickets si se incluye su número: `feature/issue-123-new-login`.

## Formato

Todas las ramas de trabajo siguen este formato:

```text
<tipo>/<descripción>
```

- **tipo**: para qué sirve la rama (`feature`, `bugfix`, `hotfix`, `release` o `chore`).
- **/**: separa el tipo de la descripción.
- **descripción**: unas pocas palabras que explican el trabajo, separadas por guiones.

Ejemplo:

```text
feature/add-login-page
└──┬──┘ └──────┬─────┘
  tipo     descripción
```

### Ramas principales

Las ramas principales del proyecto **no llevan prefijo**:

| Rama | Uso habitual |
|---|---|
| `main` | Rama principal (la más usada hoy en día). |
| `master` | Nombre antiguo de la rama principal. |
| `develop` | Rama donde se van juntando los cambios antes de publicarlos (usada en algunos flujos de trabajo). |

## Tipos de rama

| Tipo | Alias corto | Para qué sirve | Ejemplo |
|---|---|---|---|
| `feature/` | `feat/` | Añadir una **nueva funcionalidad**. | `feature/add-login-page` |
| `bugfix/` | `fix/` | **Corregir un error** durante el desarrollo normal. | `bugfix/fix-header-bug` |
| `hotfix/` | — | **Corrección urgente** de algo que ya está en producción. | `hotfix/security-patch` |
| `release/` | — | **Preparar una nueva versión** para publicarla. | `release/v1.2.0` |
| `chore/` | — | **Tareas de mantenimiento** que no añaden funcionalidad (dependencias, configuración, documentación…). | `chore/update-dependencies` |

El alias corto significa exactamente lo mismo. Estas dos ramas son equivalentes:

```text
feature/add-login-page
feat/add-login-page
```

> **Consejo:** elige una de las dos formas (`feature/` o `feat/`) y úsala siempre en el mismo proyecto.

### ¿`bugfix` o `hotfix`?

- **`bugfix/`**: el error se ha encontrado mientras se desarrollaba. Se arregla con calma y entra en la próxima versión.
- **`hotfix/`**: el error ya está afectando a los usuarios en producción. Hay que arreglarlo **ya**.

## Reglas para escribir el nombre

1. **Solo minúsculas y números** (`a-z`, `0-9`).
   - ✅ `feature/add-login`
   - ❌ `Feature/Add-Login`
2. **Las palabras se separan con guiones** (`-`).
   - ✅ `fix/header-bug`
   - ❌ `fix/header bug` (espacio) · ❌ `fix/header_bug` (guion bajo)
3. **Los puntos (`.`) solo se usan para números de versión.**
   - ✅ `release/v1.2.0` · ✅ `chore/upgrade-node-v20.11`
4. **Nada de guiones o puntos repetidos, ni al principio o al final de la descripción.**
   - ❌ `feature/new--login` (dos guiones seguidos)
   - ❌ `feature/-new-login` (empieza por guion)
   - ❌ `feature/new-login-` (termina en guion)
   - ❌ `release/v1.-2.0` (punto y guion juntos)
5. **Sin caracteres especiales**: ni espacios, ni tildes, ni `ñ`, ni símbolos como `@`, `#`, `!`…
   - ✅ `feature/add-shopping-cart` · ❌ `feature/añadir-carrito`
6. **Incluye el número de ticket** si la tarea tiene uno, en minúsculas.
   - Ticket `PROJ-123` → ✅ `feature/proj-123-add-login`
7. **Descriptivo pero breve.** Que se entienda qué se hace, sin escribir una frase entera.
   - ✅ `fix/login-redirect`
   - ❌ `fix/fix-the-problem-where-the-user-is-not-redirected-after-login`

> 💡 **¿Por qué en inglés?** La especificación no obliga a usar un idioma, pero la regla de «solo `a-z`» impide usar tildes y `ñ`. Por eso es habitual escribir la descripción en inglés. Si prefieres castellano, escribe sin tildes: `feature/anadir-carrito`.

## Ejemplos correctos e incorrectos

### ✅ Nombres válidos

```text
main
develop
feature/add-login-page
feat/add-login-page
fix/header-bug
bugfix/empty-form-error
hotfix/security-patch
release/v1.2.0
chore/update-dependencies
feature/issue-123-new-login
```

### ❌ Nombres no válidos y cómo corregirlos

| Nombre incorrecto | Problema | Nombre corregido |
|---|---|---|
| `Feature/Add-Login` | Usa mayúsculas. | `feature/add-login` |
| `feature/new--login` | Dos guiones seguidos. | `feature/new-login` |
| `feature/-new-login` | Empieza por guion. | `feature/new-login` |
| `feature/new-login-` | Termina en guion. | `feature/new-login` |
| `release/v1.-2.0` | Punto y guion juntos. | `release/v1.2.0` |
| `fix/header bug` | Tiene un espacio. | `fix/header-bug` |
| `fix/header_bug` | Usa guion bajo. | `fix/header-bug` |
| `unknown/some-task` | `unknown` no es un tipo válido. | `chore/some-task` (o el tipo que corresponda) |
| `add-login` | Le falta el tipo. | `feature/add-login` |
| `feature/PROJ-123-login` | Ticket en mayúsculas. | `feature/proj-123-login` |

## Prefijos para agentes de IA (novedad de la versión 1.1.0)

Hoy en día algunas ramas las crean **asistentes de programación con IA**. La versión 1.1.0 añade prefijos para identificarlas:

| Prefijo | Quién crea la rama |
|---|---|
| `ai/` | Cualquier agente de IA (prefijo genérico). |
| `claude/` | Claude (Anthropic). |
| `codex/` | Codex (OpenAI). |
| `copilot/` | GitHub Copilot. |
| `cursor/` | Cursor. |

Ejemplos:

```text
ai/refactor-auth-flow
claude/security-patch
copilot/add-unit-tests
```

Las reglas del nombre son las mismas: minúsculas, guiones, sin espacios…

> Estos prefijos son **opcionales**. Si trabajas sin agentes de IA, puedes ignorar este apartado. La versión 1.1.0 es compatible con la 1.0.0: lo único que añade son estos prefijos.

## Relación con Conventional Commits

Las dos convenciones se complementan: **la rama dice en qué estás trabajando** y **cada *commit* dice qué paso concreto has dado**.

| Tipo de rama | *Commits* que suele contener |
|---|---|
| `feature/` | `feat`, `test`, `docs`, `refactor` |
| `bugfix/` / `fix/` | `fix`, `test` |
| `hotfix/` | `fix` |
| `release/` | `chore`, `docs` |
| `chore/` | `chore`, `build`, `ci`, `docs`, `style` |

Hay **menos tipos de rama que de *commit*** a propósito: una rama es **temporal** (se borra al fusionarla) y solo necesita indicar el propósito general. Por eso no existen ramas `docs/`, `style/` o `refactor/`.

Ejemplo de una rama con sus *commits*:

```text
Rama: feature/add-password-recovery

  feat: añadir formulario de recuperación de contraseña
  feat: enviar correo con enlace de recuperación
  test: añadir pruebas de recuperación de contraseña
  docs: documentar recuperación de contraseña
```

## Flujo básico con Git

```bash
# 1. Situarse en la rama principal y actualizarla
git switch main
git pull

# 2. Crear la rama de trabajo con un nombre convencional y cambiarse a ella
git switch -c feature/add-password-recovery

# 3. Trabajar y hacer commits (con Conventional Commits)
git add .
git commit -m "feat: añadir formulario de recuperación de contraseña"

# 4. Subir la rama a GitHub
git push -u origin feature/add-password-recovery

# 5. Abrir un Pull Request en GitHub y, cuando se fusione,
#    volver a main y borrar la rama local
git switch main
git pull
git branch -d feature/add-password-recovery
```

### Renombrar una rama mal nombrada

Si ya has creado una rama con un nombre incorrecto, puedes cambiarlo:

```bash
# Renombrar la rama en la que estás
git branch -m feature/add-password-recovery

# Renombrar otra rama
git branch -m Nueva_Rama feature/new-branch
```

> Si la rama ya estaba subida a GitHub, después de renombrarla hay que subirla con el nombre nuevo (`git push -u origin feature/new-branch`) y borrar la antigua del remoto (`git push origin --delete Nueva_Rama`).

## Tipos propios del equipo

La especificación permite que un equipo **añada sus propios tipos** (por ejemplo, `experiment/`), siempre que estén **documentados** y configurados en la herramienta que valide los nombres.

Si no se han documentado, cualquier tipo que no esté en la lista oficial no es válido: `unknown/some-task` ❌.

## Comprobación automática

La especificación publica una expresión regular para validar los nombres. Herramientas como **commit-check** (y su versión para GitHub Actions, **commit-check-action**) la usan para rechazar automáticamente las ramas mal nombradas.

## Resumen

| Regla | Ejemplo |
|---|---|
| Formato `<tipo>/<descripción>` | `feature/add-login-page` |
| Tipos: `feature` (`feat`), `bugfix` (`fix`), `hotfix`, `release`, `chore` | `hotfix/security-patch` |
| Ramas principales sin prefijo | `main`, `master`, `develop` |
| Solo minúsculas y números | `fix/header-bug` |
| Palabras separadas por guiones | `chore/update-dependencies` |
| Puntos solo para versiones | `release/v1.2.0` |
| Sin guiones o puntos repetidos, ni al principio o al final | `feature/new-login` |
| Número de ticket en minúsculas | `feature/proj-123-add-login` |
| Prefijos de IA opcionales | `claude/security-patch` |

## Regla final

Antes de crear una rama, responde a dos preguntas:

> 1. **¿Qué tipo de trabajo voy a hacer?** → el tipo (`feature`, `fix`…).
> 2. **¿Qué voy a hacer, en pocas palabras?** → la descripción.

```text
¿Qué tipo? Nueva funcionalidad   → feature
¿Qué hago? Añadir modo oscuro    → add-dark-mode

Rama: feature/add-dark-mode
```

## Referencias

- Especificación oficial: <https://conventionalbranch.org/>
- Convención de mensajes de *commit*: [Conventional Commits](<Conventional Commits.md>)
