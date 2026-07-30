# auditoria-seguridad-ssh-medusa
Auditoría automatizada de seguridad perimetral y validación de robustez de credenciales en servicios SSH mediante análisis en paralelo con Medusa.

# Reporte de Auditoría: Evaluación Ciberseguridad sobre Servicio SSH y Mitigación Perimetral

## 📌 1. Resumen Ejecutivo
Este documento técnico detalla los hallazgos, metodologías y resultados obtenidos durante la auditoría de seguridad perimetral realizada sobre el protocolo de acceso remoto SSH (Puerto 22) de un nodo de la infraestructura local. 

El objetivo fue identificar configuraciones criptográficas propensas a degradación (*downgrade*) y validar la eficiencia de los mecanismos de autenticación frente a vectores de ataque basados en fuerza bruta dirigida por hilos en paralelo.

| Componente | Detalle Técnico |
| :--- | :--- |
| **Identificación del Objetivo** | Nodo de Laboratorio Vulnerable (`192.168.1.50`) |
| **Protocolo Evaluado** | SSH (Secure Shell) - Puerto 22 |
| **Entorno Atacante** | Kali Linux Enterprise Architecture |
| **Fecha de la Auditoría** | 29 de Julio, 2026 |
| **Resultado de Explotación** | **Exitoso (Compromiso de Credenciales Administrativas)** |

---

## 🛠️ 2. Fase de Reconocimiento y Descubrimiento Criptográfico

### 2.1 Identificación de Restricciones Legacy
Durante la fase inicial de intrusión automatizada empleando la suite **THC-Hydra**, el subsistema criptográfico del sistema atacante interrumpió el *handshake* debido a políticas estrictas de seguridad de la plataforma moderna. El servidor objetivo obligaba al uso de algoritmos de intercambio de llaves obsoletos (`ssh-rsa` / `ssh-dss`).

text
[ERROR] Key Exchange Error - Remote host requires deprecated cryptographic primitives.

A continuación se presenta la evidencia del rechazo de la negociación por falta de compatibilidad de cifrado:
<img width="832" height="352" alt="image" src="https://github.com/user-attachments/assets/40b6b771-eb52-44cf-adff-d4c348880da9" />
3. Ejecución del Ataque y Pivote Técnico
3.1 Transición y Configuración en Medusa v2.3
Para resolver el bloqueo criptográfico sin degradar la seguridad global del entorno atacante, se pivotó hacia Medusa v2.3. Gracias a su modularidad dinámica, la herramienta logró negociar con los parámetros del demonio SSH objetivo de forma aislada.

Inicialmente se ejecutó un escaneo masivo sobre el alias administrativo estándar utilizando el diccionario de la comunidad rockyou.txt:
<img width="945" height="808" alt="image" src="https://github.com/user-attachments/assets/dfe39b23-cc0c-48de-81a1-5853c0d7b7dd" />
medusa -h 192.168.1.50 -u admin -P /usr/share/wordlists/rockyou.txt -M ssh
La consola procesó las solicitudes en paralelo dividiendo la carga por hilos

<img width="932" height="430" alt="image" src="https://github.com/user-attachments/assets/74c84962-195e-4731-ab93-045e5782d309" />
3.2 Optimización mediante Diccionario Dirigido
Para evitar la denegación de servicio (DoS) por saturación de memoria en el servidor auditado y reducir el tráfico de red, se generó un vector de ataque optimizado mediante un archivo plano (claves.txt).


Al correlacionar las firmas y vectores del entorno, se detectó que el alias real pertenecía al usuario de laboratorio msfadmin. El comando final fue estructurado de la siguiente forma:
medusa -h 192.168.1.50 -u msfadmin -P claves.txt -M ssh

<img width="948" height="372" alt="image" src="https://github.com/user-attachments/assets/4f15c67f-c7e3-47e9-b285-3cb843cad6a5" />

4. Resultados de Explotación (Compromiso de Sockets)
El ataque de diccionario dirigido redujo el ruido del canal en un 99.99%. En la permutación número 7, el motor de Medusa validó con éxito el par de credenciales, deteniendo la ejecución de subprocesos y exponiendo el acceso en texto plano:

Usuario Comprometido: msfadmin

Contraseña Recuperada: msfadmin

Vector de Entrada: Servicio SSH (Puerto 22)
<img width="852" height="70" alt="image" src="https://github.com/user-attachments/assets/109dfbf1-7cb2-4bc2-9f7f-6f8a3c0306f6" />
5. Plan de Remediación e Ingeniería Inversa (Hardening)
Para elevar la postura de seguridad de este activo a un estándar corporativo, se deben desplegar inmediatamente los siguientes controles de ingeniería:

Endurecimiento del Demonio SSH (/etc/ssh/sshd_config):

Deshabilitar el soporte para cifrados obsoletos e implementar el uso mandatorio de curvas elípticas seguras (HostKey /etc/ssh/ssh_host_ed25519_key).

Deshabilitar por completo la autenticación directa para el usuario root o alias administrativos inseguros (PermitRootLogin no).

Migración a Criptografía Asimétrica (Llaves Públicas):

Eliminar el acceso SSH basado en contraseñas tradicionales tradicionales y migrar al uso exclusivo de llaves Ed25519 de alta entropía.

Control Adaptativo de Tráfico Perimetral:

Desplegar un servicio de análisis reactivo de registros como Fail2ban configurado para banear temporalmente direcciones IP locales o remotas que excedan de 3 intentos fallidos en una ventana de 5 minutos.

