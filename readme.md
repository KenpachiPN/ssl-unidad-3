# SSL/TLS Autofirmado en XAMPP — Unidad 3

Implementación de un certificado SSL/TLS autofirmado sobre Apache (XAMPP) para servir una aplicación web local por HTTPS. Taller AA2 — Cifrado y Autenticación.

## Herramientas

- XAMPP (Apache 2.4 + mod_ssl)
- OpenSSL 3.5.5
- Git Bash (MINGW64)

## Pasos

1. **Instalar XAMPP** y verificar que Apache funcione en `http://localhost`.

2. **Habilitar SSL** en `xampp/apache/conf/httpd.conf`, descomentando:
   ```
   LoadModule ssl_module modules/mod_ssl.so
   Include conf/extra/httpd-ssl.conf
   ```

3. **Generar el certificado autofirmado** (RSA 2048) desde `xampp/apache/bin`:
   ```bash
   openssl req -x509 -newkey rsa:2048 -keyout server.key -out server.crt -days 365 -nodes -subj "//CN=localhost"
   ```
   > En Git Bash, `/CN=localhost` se debe escribir `//CN=localhost` para evitar que MSYS2 lo confunda con una ruta de Windows.

4. **Configurar el Virtual Host** en `httpd-ssl.conf`:
   - `SSLCertificateFile` → ruta a `server.crt`
   - `SSLCertificateKeyFile` → ruta a `server.key`
   - `DocumentRoot` → carpeta del proyecto
   - `ServerName localhost:443`

5. **Reiniciar Apache** y probar `https://localhost`. El navegador mostrará advertencia de conexión no segura (normal en un certificado autofirmado); al aceptarla, el sitio carga por HTTPS.

## Problemas comunes

| Problema | Solución |
|---|---|
| `OPENSSL_CONF` apunta a un `.cnf` de otra app (ej. PostgreSQL) | `export OPENSSL_CONF="C:/xampp/apache/conf/openssl.cnf"` |
| `set` no funciona en Git Bash | Usar `export` en vez de `set` |
| Apache sigue usando el certificado viejo de XAMPP (2013) | Sobrescribir `ssl.crt/server.crt` y `ssl.key/server.key` con los generados |
| `Certificate and private key ... do not match` | Regenerar `.crt` y `.key` juntos en una sola ejecución |
| Advertencia `AH01909` (ID no coincide) | Ajustar `ServerName` a `localhost:443` |

## Nota de seguridad

`server.key` (clave privada) **no se incluye** en este repositorio — está excluido vía `.gitignore`. Nunca debe compartirse públicamente.

## Certificado

Autofirmado, válido 365 días, `CN=localhost`. Uso exclusivo para desarrollo/pruebas locales — en producción se usaría una CA pública (ej. Let's Encrypt).