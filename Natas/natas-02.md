# Natas Level 2 → Level 3

> ⚠️ Spoilers: las contraseñas están omitidas (`[redacted]`). Lo importante es el método.

- **URL:** `http://natas2.natas.labs.overthewire.org`
- **Credenciales:** `natas2` / `[redacted]`

## Análisis y Explotación
1. Se revisó el código fuente y no había ninguna contraseña a la vista, así que se probó con otras herramientas, empezando por `gobuster`.
2. El primer intento falló: `gobuster` se detuvo con un error indicando que el servidor devolvía `401` incluso para URLs inexistentes (comportamiento "wildcard"). Esto se debe a que la página está protegida con autenticación HTTP y la herramienta no estaba enviando credenciales.
3. Mientras tanto, el código fuente sí mostraba una imagen incrustada apuntando a `files/pixel.png`, lo que delató la existencia del directorio `/files/`.
4. Al navegar a `/files/` (con las credenciales del nivel), el servidor tenía el listado de directorio activado y mostraba un archivo `users.txt` que exponía las credenciales de múltiples usuarios, incluida la de `natas3`.

- **Password obtenida (`natas3`):** `[redacted]`
