# Convención de commits para GitHub
Una buena convención de commits hace que el historial del proyecto sea más claro, fácil de revisar y sencillo de mantener.

## Formato recomendado
Usa:
`tipo: descripción breve`

Ejemplo:
`feat: añadir botón de login`

La descripción debe ser breve y explicar **qué cambio se hizo**.

## Tipos de commit habituales
- `feat`: nueva funcionalidad.
- `fix`: corrección de un error.
- `docs`: cambios en documentación.
- `style`: cambios de formato o estilo que no afectan al código.
- `refactor`: reorganización del código sin cambiar su comportamiento.
- `test`: añadir o modificar pruebas.
- `chore`: tareas de mantenimiento o configuración.

## Ejemplos sencillos
```bash
git commit -m "feat: añadir registro de usuarios"
git commit -m "fix: corregir validación del email"
git commit -m "docs: actualizar README"
git commit -m "test: añadir pruebas para login"
git commit -m "refactor: simplificar función de búsqueda"
git commit -m "style: formatear archivos JavaScript"
git commit -m "chore: actualizar dependencias"

git commit -m "build(logging): add log4j dependencies and configuration"
git commit -m "chore: initialise repository for level1"
git commit -m "docs: update README.md"
git commit -m "feat: add exception, model, service and ui classes"
git commit -m "feat: add exception, utility and main classes"
git commit -m "feat: add Menu, EditorManage and NewsManage classes" -m "Refactor price and score calculation out of News model into NewsManage utility class. Wire everything together in Main."
git commit -m "feat: add new project files and code"
git commit -m "feat: added more clases, utils and 'ui'. Refactored some code"
git commit -m "fix: fix bugs and improve code"
git commit -m "fix: improve code"
git commit -m "refactor: improve code structure"
git commit -m "style: remove comment and improve code formatting"
```

## Buenas prácticas
### 1. Un commit debe representar un cambio concreto
Cada commit debería centrarse en una tarea o cambio específico.

**Mejor:**
    feat: añadir búsqueda de productos
    fix: corregir error al eliminar productos

**Evita:**
    añadir búsqueda, corregir login y actualizar documentación

### 2. Usa el imperativo
Es recomendable escribir el mensaje como una acción.

**Preferible:**
    feat: añadir sistema de notificaciones

**En lugar de:**
    feat: añadí sistema de notificaciones

### 3. Sé específico
Evita mensajes demasiado genéricos.

**Poco útil:**
    fix: arreglar cosas

**Mejor:**
    fix: evitar error al enviar formulario vacío

### 4. Mantén los commits pequeños
Un commit pequeño es más fácil de revisar, entender y revertir.

Por ejemplo:
    feat: añadir endpoint de usuarios
    test: añadir pruebas del endpoint de usuarios
    docs: documentar endpoint de usuarios

Es preferible a un único commit enorme que contenga todos esos cambios.

### 5. Escribe mensajes claros
El mensaje debería permitir entender el cambio sin tener que revisar todo el código.

**Poco claro:**
    fix: cambios

**Más claro:**
    fix: corregir redirección después del login

### 6. Evita mezclar cambios que no están relacionados
Evita hacer un único commit con cambios completamente diferentes.

**Evita:**
    feat: añadir login y cambiar colores y actualizar README

**Mejor:**
    feat: añadir sistema de login
    style: actualizar colores de la interfaz
    docs: actualizar README

### 7. Revisa los cambios antes de hacer commit
Antes de crear un commit, revisa qué archivos han cambiado:
    git status

Para revisar exactamente qué modificaciones has realizado:

    git diff

Después puedes añadir los archivos:

    git add .

Y crear el commit:

    git commit -m "feat: añadir sistema de login"

### 8. Evita mensajes demasiado genéricos

Evita mensajes como:

    update
    changes
    final
    final version
    arreglos
    cosas nuevas

Utiliza mensajes que expliquen realmente el cambio:

    fix: corregir validación del formulario
    feat: añadir recuperación de contraseña
    docs: actualizar instrucciones de instalación

## Commits con más información

Cuando un cambio necesita una explicación adicional, puedes añadir un cuerpo al commit:

    fix: corregir cálculo del total

    El total no incluía correctamente los descuentos
    cuando había varios productos en el carrito.

Puedes crear un commit de este tipo ejecutando:

    git commit

Esto abrirá el editor para escribir el mensaje completo.

## Regla rápida

Antes de hacer un commit, intenta responder:

> ¿Qué he cambiado?

Por ejemplo:

    feat: añadir modo oscuro
    fix: corregir cálculo del IVA
    docs: explicar instalación del proyecto
    test: cubrir caso de usuario sin permisos

## Resumen

| Tipo | Uso |
|---|---|
| `feat` | Nueva funcionalidad |
| `fix` | Corrección de errores |
| `docs` | Documentación |
| `style` | Formato o estilo |
| `refactor` | Reestructuración del código |
| `test` | Pruebas |
| `chore` | Mantenimiento |

## Flujo básico

    # 1. Revisar el estado
    git status

    # 2. Revisar los cambios
    git diff

    # 3. Añadir los archivos
    git add .

    # 4. Crear el commit
    git commit -m "feat: añadir recuperación de contraseña"

    # 5. Subir los cambios a GitHub
    git push

## Regla final

Una convención sencilla para empezar es:

    tipo: descripción breve

Ejemplo:

    git commit -m "feat: añadir recuperación de contraseña"

Un buen commit debe ser **claro, específico, pequeño y relacionado con un único cambio**.

