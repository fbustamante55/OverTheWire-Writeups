# Natas Level 6 → Level 7

> ⚠️ Spoilers: las contraseñas están omitidas (`[redacted]`). Lo importante es el método.

- **URL:** `http://natas6.natas.labs.overthewire.org`
- **Credenciales:** `natas6` / `[redacted]`

## Análisis y Explotación
1. La página pedía un "secret" y ofrecía un enlace a `index-source.html` con el código PHP.
2. El código mostraba que el secreto se cargaba con `include "includes/secret.inc";`.
3. Al abrir directamente `http://natas6.natas.labs.overthewire.org/includes/secret.inc` el servidor devolvía el archivo sin interpretarlo, con la variable `$secret` en texto plano.
4. Se envió ese secreto en el formulario y la aplicación entregó la contraseña.

- **Secret:** `[redacted]`
- **Password obtenida (`natas7`):** `[redacted]`

## Lección aprendida
Los archivos incluidos no deben estar dentro del directorio web accesible, o deben ser ejecutables (`.php`) para que no se sirvan como texto.
