# Laboratorio: Función serverless de registro

Práctica de la asignatura Introducción al Cómputo en la Nube (TSU Desarrollo de Software en Código Abierto, ESIT). Consiste en una función Serverless en Python, desplegada en OpenFaaS sobre Kubernetes (Docker Desktop), que presenta un formulario web y genera un archivo JSON descargable.

## Descripción

La función `registro-serverless` muestra un formulario con nombre, apellido y correo del aula virtual. Al enviarlo, genera un archivo JSON con los datos registrados, un token UUID único, la fecha y hora de ejecución, información del entorno de ejecución y una firma SHA-256 para comprobar la integridad del registro.

## Estructura del proyecto

```
laboratorio-serverless/
├── registro-serverless/     # Código de la función (handler.py, requirements.txt, tox.ini)
├── template/                # Plantillas de OpenFaaS
├── stack.yaml               # Configuración de la función
├── registro_<token>.json    # Evidencia de la ejecución
└── .gitignore
```

## Requisitos

- Windows 10/11 con Docker Desktop (motor WSL 2)
- Kubernetes habilitado en Docker Desktop (Kubeadm)
- `kubectl`, `helm` y `faas-cli`
- OpenFaaS instalado (namespace `openfaas`) con el espacio de funciones `openfaas-fn`

## Despliegue

1. Descargar la plantilla de Python:

```powershell
faas-cli template store pull python3-http
```

2. Permitir que OpenFaaS use imágenes locales (se ejecuta una sola vez):

```powershell
helm upgrade openfaas openfaas/openfaas `
  --namespace openfaas `
  --reuse-values `
  --set functions.imagePullPolicy=Never
```

3. Preparar la carpeta de construcción:

```powershell
New-Item -ItemType Directory -Force -Path "build\registro-serverless\function" | Out-Null
Copy-Item "template\python3-http\Dockerfile" "build\registro-serverless\"
Copy-Item "template\python3-http\index.py" "build\registro-serverless\"
Copy-Item "template\python3-http\requirements.txt" "build\registro-serverless\"
Copy-Item "registro-serverless\*" "build\registro-serverless\function\" -Recurse -Force
```

4. Construir la imagen:

```powershell
docker build -t registro-serverless:v1 ./build/registro-serverless
```

## Ejecución

1. Ejecutar el contenedor en el puerto 8080:

```powershell
docker run --rm -p 8080:8080 registro-serverless:v1
```

Si el puerto 8080 está ocupado por el Gateway de OpenFaaS, usa `-p 8081:8080` y abre el puerto 8081 en el navegador.

2. Con el contenedor en ejecución, abrir `http://localhost:8080` en el navegador.
3. Completar el formulario con nombre, apellido y correo del aula virtual.
4. Presionar **Generar archivo JSON**. El navegador descargará `registro_<token>.json`.

## Verificación

Con el clúster activo, el espacio de funciones puede aparecer vacío si no hay funciones desplegadas:

```powershell
kubectl get pods -n openfaas-fn
```

## Autor

Héctor Mauricio Fuentes Araujo  · Introducción al Cómputo en la Nube