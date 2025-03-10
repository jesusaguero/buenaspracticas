# 📌 Guía Empresarial de Buenas Prácticas

## 📁 Estructura Recomendada del Repositorio
```plaintext
📂 empresa-infraestructura
├── 📂 cloudformation
│   ├── templates/
│   ├── modules/
│   ├── README.md
│
├── 📂 terraform
│   ├── modules/
│   ├── environments/
│   ├── main.tf
│   ├── README.md
│
├── 📂 docker
│   ├── Dockerfile
│   ├── docker-compose.yml
│   ├── README.md
│
└── README.md  # Documento principal
```

---

## 🌟 AWS CloudFormation: Buenas Prácticas
### ✅ Estructura Recomendada
- **Usar YAML** en lugar de JSON para mayor legibilidad.
- **Dividir en módulos** reutilizables dentro de `templates/` y `modules/`.
- **Usar parámetros y mappings** en lugar de valores fijos.
- **Utilizar AWS SAM** si se implementan funciones Lambda.

### 🔒 Seguridad y Compliance
- Aplicar **Stack Policies** para evitar cambios accidentales.
- Definir **IAM Roles con privilegios mínimos**.
- Habilitar **AWS Config y CloudTrail** para auditoría.

### 🚀 Ejemplo de Estructura Base
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: "Plantilla base de CloudFormation"

Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "my-bucket-${AWS::AccountId}"
```

---

## 🌍 Terraform: Buenas Prácticas
### ✅ Organización del Código
- **Separar ambientes** (`staging`, `production`) en `environments/`.
- **Definir módulos** reutilizables dentro de `modules/`.
- Usar **terraform.tfvars** para variables específicas por ambiente.

### 🔒 Seguridad y Compliance
- **Bloquear versiones de providers** para evitar errores inesperados.
- Usar **State Backend en S3 con DynamoDB** para evitar corrupción del estado.
- Habilitar **AWS IAM Roles** en lugar de claves de acceso.

### 🚀 Ejemplo de Estructura Base
```hcl
provider "aws" {
  region = "us-east-2"
}

resource "aws_s3_bucket" "example" {
  bucket = "my-terraform-bucket"
  acl    = "private"
}
```

---

## 🐳 Docker: Buenas Prácticas
### ✅ Optimización del Dockerfile
- **Usar imágenes base ligeras** (`alpine` en lugar de `ubuntu`).
- **Evitar capas innecesarias** con menos `RUN`.
- **Usar multi-stage builds** para reducir el tamaño final de la imagen.

### 🔒 Seguridad y Compliance
- No ejecutar contenedores con **root user**.
- Usar **.dockerignore** para excluir archivos sensibles.
- Firmar imágenes y escanear vulnerabilidades antes del despliegue.

### 🚀 Ejemplo de Dockerfile Optimizado
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app .
CMD ["node", "server.js"]
```

---

## 📌 Referencias
- [Documentación Oficial de CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

🚀 **¡Usa esta guía como referencia para estandarizar y optimizar tus proyectos!**
