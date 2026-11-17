# Natas 11 → 12: XOR con key estática

[← Nivel 10](natas-10.md) · [Índice](../README.md) · [Nivel 12 →](natas-12.md)

| Campo | Detalle |
|-------|---------|
| **Wargame** | [OverTheWire: Natas](https://overthewire.org/wargames/natas/) |
| **Categoría** | Criptografía |
| **Debilidad** | [CWE-327](https://cwe.mitre.org/data/definitions/327.html): Use of a Broken or Risky Cryptographic Algorithm |
| **Herramientas** | Python, DevTools del navegador |
| **URL** | `http://natas11.natas.labs.overthewire.org` |
| **Credenciales** | `natas11` / `[redacted]` |

> [!NOTE]
> Las contraseñas y secretos están omitidos (`[redacted]`). El objetivo es documentar el método, no la respuesta.

## Resumen
La cookie de preferencias se protege con XOR y una key fija; con el plaintext por defecto conocido se recupera la key y se forja una cookie válida.

## Análisis y explotación
La aplicación guarda preferencias del usuario en una cookie `data`, "protegida" con XOR y luego codificada en base64. El código fuente revela el flujo completo:

```php
$defaultdata = array("showpassword"=>"no", "bgcolor"=>"#ffffff");

function xor_encrypt($in) {
    $key = '<censored>';
    $text = $in;
    $outText = '';
    for($i=0;$i<strlen($text);$i++) {
        $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }
    return $outText;
}

// Cookie = base64(xor(json(data), key))
function saveData($d) {
    setcookie("data", base64_encode(xor_encrypt(json_encode($d))));
}
```

La password se muestra solo si `$data["showpassword"] == "yes"`.

### Ataque: known-plaintext XOR key recovery

Como XOR es reversible y conocemos el plaintext por defecto, podemos recuperar la key:

```
key = plaintext XOR ciphertext
```

1. **Plaintext conocido** (JSON del array por defecto):
   ```
   {"showpassword":"no","bgcolor":"#ffffff"}
   ```

2. **Ciphertext**: `base64_decode(cookie_actual)`

3. **Script Python para recuperar la key y forjar la cookie:**
   ```python
   import base64, json

   cookie_b64 = "EGAgHwQ1IxYYMSQYGSZxTUksPFVHYDEQCC0/GBlgaVVIJDURDSQ1VRY="
   cipher = base64.b64decode(cookie_b64)

   known = json.dumps({"showpassword":"no","bgcolor":"#ffffff"},
                      separators=(',',':')).encode()

   # Recuperar key
   raw_key = bytes([cipher[i] ^ known[i] for i in range(len(known))])
   key = ""
   for n in range(1, len(raw_key)//2 + 1):
       pat = raw_key[:n].decode('latin-1')
       if (pat * (len(raw_key)//n + 1))[:len(raw_key)] == raw_key.decode('latin-1'):
           key = pat
           break
   print(f"Key encontrada: {key}")  # → kBSw

   # Forjar nueva cookie con showpassword=yes
   def xor_encrypt(data, key):
       return bytes([data[i] ^ ord(key[i % len(key)]) for i in range(len(data))])

   new_data = json.dumps({"showpassword":"yes","bgcolor":"#ffffff"},
                         separators=(',',':')).encode()
   new_cookie = base64.b64encode(xor_encrypt(new_data, key)).decode()
   print(f"Nueva cookie: {new_cookie}")
   ```

4. **Key recuperada:** `kBSw`
5. **Nueva cookie forjada:** `EGAgHwQ1IxYYMSQYGSZxTUk7NgRJbnEVDCE8GwQwcU1JYTURDSQ1EUk/`
6. Se reemplazó el valor de la cookie `data` en DevTools → Application → Cookies y se recargó la página.

## Resultado

- **Password obtenida (`natas12`):** `[redacted]`

## Lección aprendida y mitigación
XOR **no es cifrado seguro** cuando se usa con una key estática y corta. Al conocer el plaintext (aquí el valor por defecto del array), se recupera la key trivialmente con una sola operación. Para proteger cookies se debe usar HMAC (firma criptográfica) o cifrado autenticado (AES-GCM), nunca XOR con key fija.

## Referencias
- [CWE-327](https://cwe.mitre.org/data/definitions/327.html): Use of a Broken or Risky Cryptographic Algorithm
- [OverTheWire: Natas](https://overthewire.org/wargames/natas/)

---
[← Nivel 10](natas-10.md) · [Índice](../README.md) · [Nivel 12 →](natas-12.md)
