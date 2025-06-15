# k8servers_template

## Descripción

**k8servers_template** es el proyecto base para la construcción de imágenes Docker personalizadas de Nginx utilizadas en la plataforma [k8servers](https://k8servers.es).

Este repositorio contiene el `Dockerfile` y la configuración por defecto de Nginx (`default.conf`) que sirven como plantilla para desplegar los sitios web de los clientes en contenedores aislados y seguros.

---

## Estructura del repositorio

- `Dockerfile` — Archivo de construcción de la imagen Docker basada en Nginx.
- `nginx/`
  - `default.conf` — Configuración por defecto de Nginx para servir aplicaciones PHP y archivos estáticos.
- `README.md` — Este archivo.

---

## Tecnologías utilizadas

- **Nginx**: Servidor web y proxy inverso.
- **Docker**: Para la construcción y despliegue de la imagen.
- **(Opcional) PHP-FPM**: La configuración está preparada para conectarse a un backend PHP-FPM externo.
- Git Actions Automatico para poder tener la imagen ECR de forma automatica en nuestro entorno de Amazon.

---

## Uso

### Construcción de la imagen Docker (MANUAL)

1. Clona este repositorio:
    ```bash
    git clone https://github.com/tu_usuario/k8servers_template.git
    cd k8servers_template
    ```

2. Construye la imagen Docker:
    ```bash
    docker build -t k8servers-nginx-template .
    ```

3. (Opcional) Subir la imagen al ECR o usarla en el cluster Kubernetes.

### Personalización

- Puedes modificar el archivo `nginx/default.conf` para adaptar la configuración a las necesidades de cada cliente o proyecto.
- El `Dockerfile` está preparado para copiar archivos de configuración adicionales si es necesario.

---

## Autor

Daniel Ramírez Linares (dramlin1010)  
TFG - k8servers.es
