# Ubuntu Server Headless Homelab en Laptop (Wi-Fi + IP Estática)

Documentación técnica y registro de resolución de problemas para la configuración de un servidor casero (*homelab*) sobre hardware de laptop con **Ubuntu Server 24.04 LTS**, operando de manera headless (sin monitor ni periféricos continuos) mediante conexión inalámbrica estática.

---

## 1. Problemas Encontrados y Solución Aplicada

### 1.1 Mensajes del Kernel superpuestos en consola TTY
* **Problema:** Durante la edición de archivos locales con `nano`, aparecían mensajes del kernel (`rtlwifi: AP off, try to reconnect now`) imprimiéndose sobre la pantalla y deformando visualmente el texto del archivo.
* **Causa:** Los registros de advertencia del kernel enviados a la consola estándar sobreescriben el buffer visual del editor en sesiones locales directas.
* **Solución:** Uso de la combinación de teclas `Ctrl + L` dentro de `nano` para forzar un refresco completo del buffer de la pantalla sin modificar el archivo, seguido de la eliminación de caracteres residuales.

### 1.2 Errores de indentación estricta en Netplan (YAML)
* **Problema:** Fallos repetidos al aplicar la configuración (`sudo netplan apply`) con errores del tipo `Invalid YAML: inconsistent indentation` en los bloques `access-points:` y `password:`. La interfaz mantenía la IP dinámica y no tomaba los parámetros estáticos.
* **Causa:** El estándar YAML rechaza tabulaciones y exige un esquema par estricto de espacios por nivel jerárquico. Las directivas de SSID y credenciales deben estar anidadas explícitamente dentro de `access-points:`.
* **Solución:** Corrección manual de la estructura respetando la sangría exacta:
  * `wifis:` (2 espacios)
  * `wlo1:` (4 espacios)
  * `access-points:` (6 espacios)
  * `"SSID":` (8 espacios)
  * `password:` (10 espacios)

### 1.3 Error de direccionamiento en la Puerta de Enlace (Gateway)
* **Problema:** La configuración asignaba inicialmente `via: x.x.x.x` como ruta por defecto, dejando al servidor sin comunicación externa hacia internet.
* **Causa:** `x.x.x.x` define la dirección base de la subred en una máscara `/24`, no una interfaz de enrutamiento válida.
* **Solución:** Corrección del parámetro apuntando a la dirección IP real del router local: `x.x.x.x`.

### 1.4 Advertencia de seguridad por permisos abiertos en Netplan
* **Problema:** Advertencia en consola: `Permissions for /etc/netplan/*.yaml are too open. Netplan configuration should not be accessible by others`.
* **Causa:** Al contener claves de acceso Wi-Fi en texto plano, las políticas de seguridad de Netplan exigen que únicamente el superusuario `root` tenga permisos de lectura y escritura.
* **Solución:** Restricción estricta de permisos sobre el archivo:
  ```bash
  sudo chmod 600 /etc/netplan/*.yaml
