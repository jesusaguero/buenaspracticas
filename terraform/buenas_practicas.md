### Terraform Buenas Prácticas  

#### TF-001: **Configuración del proveedor de AWS**  
**Correcto:**  
```hcl
provider "aws" {
  region = var.aws_region
}
```  
**Incorrecto:**  
```hcl
provider "aws" {
  region = "us-east-2"
}
```  
**Explicación**: Hardcodear valores como la región hace que el código sea menos flexible. Usar variables permite cambiar la región fácilmente sin modificar el código.  

---

#### TF-002: **Definición de variables**  
**Correcto:**  
```hcl
variable "aws_region" {
  description = "La región de AWS donde se desplegarán los recursos"
  type        = string
  default     = "us-east-2"
}
```  
**Incorrecto:**  
```hcl
variable "aws_region" {}
```  
**Explicación**: No incluir descripciones o tipos específicos puede hacer que el código sea difícil de entender y usar. Las buenas prácticas indican ser explícitos con las variables.  

---

#### TF-003: **Crear una VPC**  
**Correcto:**  
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr_block
  enable_dns_support = true
  enable_dns_hostnames = true
  tags = {
    Name = "MainVPC"
  }
}
```  
**Incorrecto:**  
```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  enable_dns_support = true
  enable_dns_hostnames = true
}
```  
**Explicación**: Hardcodear el CIDR del bloque de red limita la flexibilidad del template.  

---

#### TF-004: **Crear una subred pública**  
**Correcto:**  
```hcl
resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
  availability_zone = "us-east-2a"
  map_public_ip_on_launch = true
  tags = {
    Name = "PublicSubnet"
  }
}
```  
**Incorrecto:**  
```hcl
resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
  map_public_ip_on_launch = true
}
```  
**Explicación**: No especificar la `availability_zone` puede generar despliegues indeseados en ciertas zonas.  

---

#### TF-005: **Crear una instancia EC2**  
**Correcto:**  
```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public.id
  tags = {
    Name = "ExampleInstance"
  }
  security_groups = ["default"]
  user_data = <<-EOF
              #!/bin/bash
              echo "Hello World" > /var/www/html/index.html
              EOF
}
```  
**Incorrecto:**  
```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"  
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public.id
  tags = {
    Name = "ExampleInstance"
  }
}
```  
**Explicación**: Hardcodear el tipo de instancia limita la flexibilidad del código.  

---

#### TF-006: **Outputs**  
**Correcto:**  
```hcl
output "instance_public_ip" {
  description = "La IP pública de la instancia EC2"
  value       = aws_instance.example.public_ip
}
```  
**Incorrecto:**  
```hcl
output "instance_public_ip" {
  value = aws_instance.example.public_ip
}
```  
**Explicación**: Es importante proporcionar descripciones en los outputs para mejorar la comprensión del código.  

---

#### TF-007: **Manejo de estado remoto**  
**Correcto:**  
```hcl
terraform {
  backend "s3" {
    bucket         = "mi-bucket-de-terraform"
    key            = "terraform/statefile"
    region         = "us-east-2"
    dynamodb_table = "terraform-lock"
    encrypt        = true
  }
}
```  
**Incorrecto:**  
```hcl
terraform {
  backend "local" {
    path = "terraform.tfstate"
  }
}
```  
**Explicación**: Usar un backend remoto como S3 con DynamoDB evita conflictos de estado y mejora la integridad del archivo de estado.  

---

#### TF-008: **Definir la versión de Terraform**

✅ **Correcto (en versions.tf):**
📌 **Ubicación recomendada:** `versions.tf`
```hcl
terraform {
  required_version = ">= 1.0"
}
```

❌ **Incorrecto:**
```hcl
terraform {
}
```

💡 **Explicación:**  
Es importante definir la versión de Terraform en el archivo `versions.tf` para evitar problemas de compatibilidad entre entornos y garantizar la consistencia en los despliegues.  
Mantener esta configuración en un archivo separado ayuda a organizar mejor el código y facilitar actualizaciones de versión.


#### TF-009: **Los módulos deben terminar con "-applying"**  
**Correcto:**  
```hcl
module "network-applying" {
  source = "./modules/network"
}
```
**Incorrecto:**  
```hcl
module "network" {
  source = "./modules/network"
}
```
**Explicación**: Usar un sufijo estándar como `-applying` en los módulos facilita la identificación y el mantenimiento del código.
