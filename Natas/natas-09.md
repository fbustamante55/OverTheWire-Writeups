# Natas Level 9 → Level 10

> ⚠️ Spoilers: las contraseñas están omitidas (`[redacted]`). Lo importante es el método.

- **URL:** `http://natas9.natas.labs.overthewire.org`
- **Credenciales:** `natas9` / `[redacted]`

## Análisis y Explotación
1. La página era un buscador de palabras. Su código fuente mostraba que el parámetro `needle` se insertaba directamente en un comando de shell:
   ```php
   passthru("grep -i $key dictionary.txt");
   ```
2. Sin ninguna sanitización, se puede inyectar un separador de comandos (`;`) para ejecutar un segundo comando. Payload en el campo de búsqueda:
   ```
   ;cat /etc/natas_webpass/natas10
   ```
3. El servidor ejecutó `grep -i ` (sin argumento) y luego el `cat`, mostrando la contraseña en la respuesta.

- **Password obtenida (`natas10`):** `[redacted]`

## Lección aprendida
Nunca pasar entrada del usuario a funciones de ejecución de comandos (`passthru`, `exec`, `system`). Si es inevitable, validar con lista blanca y escapar con `escapeshellarg()`.
