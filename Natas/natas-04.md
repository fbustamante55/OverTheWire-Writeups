# Natas 04 → 05: Autorización basada en el header `Referer`

[← Nivel 3](natas-03.md) · [Índice](../README.md) · [Nivel 5 →](natas-05.md)

| Campo | Detalle |
|-------|---------|
| **Wargame** | [OverTheWire: Natas](https://overthewire.org/wargames/natas/) |
| **Categoría** | Control de acceso |
| **Debilidad** | [CWE-293](https://cwe.mitre.org/data/definitions/293.html): Using Referer Field for Authentication |
| **Herramientas** | Burp Suite (Repeater), `curl` |
| **URL** | `http://natas4.natas.labs.overthewire.org` |
| **Credenciales** | `natas4` / `[redacted]` |

> [!NOTE]
> Las contraseñas y secretos están omitidos (`[redacted]`). El objetivo es documentar el método, no la respuesta.

## Resumen
La aplicación decide si da acceso según el header `Referer`, un valor que el cliente controla y puede falsificar.

## Análisis y explotación
1. La aplicación web mostraba el mensaje:
   `Access disallowed. You are visiting from "" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/"`
2. Por costumbre se probó primero `gobuster` (con credenciales de `natas4`), pero solo apareció `index.php` (200) y rutas `403`: no había directorios ocultos. La pista estaba en el propio mensaje, no en el servidor de archivos.
3. Al pulsar "Refresh page" el enlace llevaba a `index.php`, y la página insistía en saber *desde dónde* se llegaba. Eso apunta a la cabecera HTTP `Referer`, con la que el navegador indica la página de origen.
4. Utilizando Burp Suite (Repeater), se interceptó la petición `GET /index.php` y se modificó manualmente el encabezado `Referer` para que apuntara a `http://natas5.natas.labs.overthewire.org/`.
5. Al enviar la petición modificada (`Send`), el servidor respondió `200 OK` con el mensaje "Access granted" y la contraseña del siguiente nivel.

Equivalente desde la terminal con `curl`:
```bash
curl -u natas4:'[redacted]' -e "http://natas5.natas.labs.overthewire.org/" http://natas4.natas.labs.overthewire.org/
```

## Resultado

- **Password obtenida (`natas5`):** `[redacted]`

## Lección aprendida y mitigación
El header `Referer` lo controla el cliente, por lo que **nunca debe usarse como mecanismo de autorización**: cualquiera puede falsificarlo.

## Referencias
- [CWE-293](https://cwe.mitre.org/data/definitions/293.html): Using Referer Field for Authentication
- [OverTheWire: Natas](https://overthewire.org/wargames/natas/)

---
[← Nivel 3](natas-03.md) · [Índice](../README.md) · [Nivel 5 →](natas-05.md)
