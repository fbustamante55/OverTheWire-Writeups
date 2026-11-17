# Natas 05 → 06: Estado de sesión controlado por el cliente

[← Nivel 4](natas-04.md) · [Índice](../README.md) · [Nivel 6 →](natas-06.md)

| Campo | Detalle |
|-------|---------|
| **Wargame** | [OverTheWire: Natas](https://overthewire.org/wargames/natas/) |
| **Categoría** | Control de acceso |
| **Debilidad** | [CWE-565](https://cwe.mitre.org/data/definitions/565.html): Reliance on Cookies without Validation and Integrity Checking |
| **Herramientas** | DevTools del navegador, `curl` |
| **URL** | `http://natas5.natas.labs.overthewire.org` |
| **Credenciales** | `natas5` / `[redacted]` |

> [!NOTE]
> Las contraseñas y secretos están omitidos (`[redacted]`). El objetivo es documentar el método, no la respuesta.

## Resumen
El estado de autenticación se guarda en una cookie `loggedin` que el cliente puede editar libremente.

## Análisis y explotación
1. La aplicación bloqueaba el contenido con el mensaje *"You are not logged in"*.
2. Al inspeccionar las cookies del navegador (DevTools → Application/Storage) apareció una cookie `loggedin` con valor `0`.
3. Se cambió el valor a `1` y se recargó la página: el servidor dio la sesión por válida y mostró la contraseña.

Equivalente con `curl`:
```bash
curl -u natas5:'[redacted]' --cookie "loggedin=1" http://natas5.natas.labs.overthewire.org/
```

## Resultado

- **Password obtenida (`natas6`):** `[redacted]`

## Lección aprendida y mitigación
El estado de autenticación no puede depender de un valor que el cliente puede editar. Hay que usar sesiones gestionadas por el servidor.

## Referencias
- [CWE-565](https://cwe.mitre.org/data/definitions/565.html): Reliance on Cookies without Validation and Integrity Checking
- [OverTheWire: Natas](https://overthewire.org/wargames/natas/)

---
[← Nivel 4](natas-04.md) · [Índice](../README.md) · [Nivel 6 →](natas-06.md)
