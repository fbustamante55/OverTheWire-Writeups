# Natas 02 → 03: Directory listing activado

[← Nivel 1](natas-01.md) · [Índice](../README.md) · [Nivel 3 →](natas-03.md)

| Campo | Detalle |
|-------|---------|
| **Wargame** | [OverTheWire: Natas](https://overthewire.org/wargames/natas/) |
| **Categoría** | Exposición de información |
| **Debilidad** | [CWE-548](https://cwe.mitre.org/data/definitions/548.html): Exposure of Information Through Directory Listing |
| **Herramientas** | Navegador, `gobuster` |
| **URL** | `http://natas2.natas.labs.overthewire.org` |
| **Credenciales** | `natas2` / `[redacted]` |

> [!NOTE]
> Las contraseñas y secretos están omitidos (`[redacted]`). El objetivo es documentar el método, no la respuesta.

## Resumen
Una imagen incrustada delata el directorio `/files/`, que tiene el listado de directorio activado y expone un archivo con credenciales.

## Análisis y explotación
1. Se revisó el código fuente y no había ninguna contraseña a la vista, así que se probó con otras herramientas, empezando por `gobuster`.
2. El primer intento falló: `gobuster` se detuvo con un error indicando que el servidor devolvía `401` incluso para URLs inexistentes (comportamiento "wildcard"). Esto se debe a que la página está protegida con autenticación HTTP y la herramienta no estaba enviando credenciales.
3. Mientras tanto, el código fuente sí mostraba una imagen incrustada apuntando a `files/pixel.png`, lo que delató la existencia del directorio `/files/`.
4. Al navegar a `/files/` (con las credenciales del nivel), el servidor tenía el listado de directorio activado y mostraba un archivo `users.txt` que exponía las credenciales de múltiples usuarios, incluida la de `natas3`.

## Resultado

- **Password obtenida (`natas3`):** `[redacted]`

## Lección aprendida y mitigación
Desactivar el listado de directorios (en Apache, `Options -Indexes`) y no guardar archivos con credenciales dentro del directorio web. Una ruta que no está enlazada no es una ruta protegida.

## Referencias
- [CWE-548](https://cwe.mitre.org/data/definitions/548.html): Exposure of Information Through Directory Listing
- [OverTheWire: Natas](https://overthewire.org/wargames/natas/)

---
[← Nivel 1](natas-01.md) · [Índice](../README.md) · [Nivel 3 →](natas-03.md)
