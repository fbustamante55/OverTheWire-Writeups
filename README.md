# OverTheWire - Writeups

Writeups de mi paso por los wargames de [OverTheWire](https://overthewire.org/wargames/): qué vulnerabilidad hay en cada nivel, cómo la explotué, con qué herramientas y cómo se corrige.

> ⚠️ **Spoilers:** las contraseñas de los niveles están omitidas (`[redacted]`). Aquí se explica el *cómo*, no la respuesta final.

## Índice

| Wargame | Enfoque | Progreso | Writeups |
|---------|---------|----------|----------|
| Natas | Seguridad web | Niveles 0 → 15 (en progreso) | [0-4](Natas/natas-00-04.md) · [5-9](Natas/natas-05-09.md) · [10-15](Natas/natas-10-15.md) |

## Natas: nivel por nivel

| Nivel | Vulnerabilidad | Técnica / herramienta |
|-------|----------------|-----------------------|
| 0 - 1 | Información expuesta en el código fuente | Inspección del HTML |
| 2 | Directory listing activado | `/files/users.txt`, gobuster |
| 3 | Ruta oculta revelada por `robots.txt` | gobuster con autenticación |
| 4 | Autorización basada en el header `Referer` | Burp Repeater, `curl -e` |
| 5 | Estado de sesión controlado por el cliente | Edición de cookie `loggedin` |
| 6 | Archivo incluido accesible desde la web | Lectura directa de `secret.inc` |
| 7 | Local File Inclusion (LFI) | `?page=/etc/natas_webpass/...` |
| 8 | Codificación reversible, no cifrado | `hex2bin`, `strrev`, `base64_decode` |
| 9 | Command Injection | Separador `;` |
| 10 | Command Injection con blocklist | Argumento extra en `grep` |
| 11 | XOR con key estática | Known-plaintext, cookie forjada |
| 12 | Upload sin validación de extensión | Webshell PHP, Burp Repeater |
| 13 | Upload validado solo por magic bytes | Archivo polyglot JPEG + PHP |
| 14 | SQL Injection (login bypass) | `" or 1=1 -- ` |
| 15 | Blind SQL Injection (booleana) | sqlmap `--technique=B --prefix --suffix` |

## Video

🎥 Video del proceso completo: _próximamente_

<!-- Cuando lo subas a YouTube, reemplaza la línea de arriba por: [Ver en YouTube](URL) -->

## Herramientas usadas
- Navegador (inspección de código fuente, DevTools)
- `gobuster` (enumeración de directorios)
- Burp Suite (Repeater)
- `curl`
- `sqlmap` (SQL Injection automatizada)
- Python (scripts de apoyo)

## Estructura del repositorio

```
OverTheWire-Writeups/
├── Natas/
│   ├── natas-00-04.md
│   ├── natas-05-09.md
│   └── natas-10-15.md
└── README.md
```

## Aviso
Este repositorio es solo con fines educativos. Los retos de OverTheWire son un entorno legal y controlado para practicar.
