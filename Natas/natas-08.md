# Natas Level 8 → Level 9

> ⚠️ Spoilers: las contraseñas están omitidas (`[redacted]`). Lo importante es el método.

- **URL:** `http://natas8.natas.labs.overthewire.org`
- **Credenciales:** `natas8` / `[redacted]`

## Análisis y Explotación
1. El código fuente de la página mostraba cómo se comparaba el secreto: el valor se ofuscaba con
   ```php
   bin2hex(strrev(base64_encode($secret)))
   ```
   y se comparaba contra un valor ya codificado (`[redacted]`).
2. Basta con aplicar las operaciones **en orden inverso**: `hex2bin` → `strrev` → `base64_decode`. Con PHP en la terminal:
   ```bash
   php -r 'echo base64_decode(strrev(hex2bin("[valor codificado]")));'
   ```
3. El resultado fue el secreto en texto plano (`[redacted]`). Al enviarlo en el formulario, la aplicación mostró la contraseña.

- **Password obtenida (`natas9`):** `[redacted]`

## Lección aprendida
Base64, hexadecimal o invertir una cadena son **codificaciones, no cifrado**: no dan confidencialidad. Para verificar secretos se deben usar hashes con sal (por ejemplo Argon2id o bcrypt).
