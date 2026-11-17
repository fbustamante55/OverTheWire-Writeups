# Natas Level 5 → Level 6

> ⚠️ Spoilers: las contraseñas están omitidas (`[redacted]`). Lo importante es el método.

- **URL:** `http://natas5.natas.labs.overthewire.org`
- **Credenciales:** `natas5` / `[redacted]`

## Análisis y Explotación
1. La aplicación bloqueaba el contenido con el mensaje *"You are not logged in"*.
2. Al inspeccionar las cookies del navegador (DevTools → Application/Storage) apareció una cookie `loggedin` con valor `0`.
3. Se cambió el valor a `1` y se recargó la página: el servidor dio la sesión por válida y mostró la contraseña.

Equivalente con `curl`:
```bash
curl -u natas5:'[redacted]' --cookie "loggedin=1" http://natas5.natas.labs.overthewire.org/
```

- **Password obtenida (`natas6`):** `[redacted]`

## Lección aprendida
El estado de autenticación no puede depender de un valor que el cliente puede editar. Hay que usar sesiones gestionadas por el servidor.
