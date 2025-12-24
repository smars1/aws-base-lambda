# Instructivo de Uso del Repositorio
## TechModa Serverless Capstone (AWS Serverless)

Este repositorio ya esta completamente preparado para desplegar una API Serverless en AWS usando
Lambda, API Gateway y DynamoDB.

Tu objetivo como alumno NO es crear la infraestructura, sino implementar la logica del CRUD
reemplazando las funciones Lambda existentes por las tuyas, generadas a partir de prompts.

---

## Flujo General

1. Abrir el repositorio en GitHub Codespaces
2. Configurar credenciales de AWS (aws configure)
3. Desplegar la infraestructura
4. Reemplazar las funciones CRUD
5. Probar la API
6. Eliminar los recursos

---

## Paso 1: Abrir el Repositorio en Codespaces

Usa GitHub Codespaces. No necesitas clonar el repositorio.
El entorno ya incluye AWS CLI, SAM CLI y Node.js.

---

## Paso 2: Configurar Credenciales de AWS

Ejecuta:

aws configure

Verifica:

aws sts get-caller-identity

---

## Paso 3: Estructura Importante

functions/
  list-items
  create-item
  get-item
  update-item
  delete-item

docs/
  specs
  prompts

---

## Paso 4: Crear el CRUD con Prompts

Usa el archivo docs/prompts/02_LAMBDA_IMPLEMENTATION.md.
Genera el codigo con IA y reemplaza completamente cada index.js.

---

## Paso 5: Build

./scripts/build.sh
o
sam build

---

## Paso 6: Deploy

./scripts/deploy.sh
o
sam deploy --guided

---

## Paso 7: Probar

curl $API_URL/products

---

## Paso 8: Logs

./scripts/logs.sh

---

## Paso 9: Limpiar Recursos

./scripts/delete-all.sh

---

Buena suerte con tu capstone.
