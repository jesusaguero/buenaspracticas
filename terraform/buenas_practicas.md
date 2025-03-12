### Terraform Buenas Prácticas

#### 1. **Configuración del proveedor de AWS**
**Correcto:**
```hcl
provider "aws" {
  region = var.aws_region
}
```

**Incorrecto:**
```hcl
# No usar variables para la región hace que el código sea menos flexible y reutilizable.
provider "aws" {
  region = "us-east-2"
}
```
**Explicación**: Hardcodear valores como la región hace que el código sea menos flexible. Usar variables permite cambiar la región fácilmente sin modificar el código.

---

#### 2. **Definición de variables**
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
# No usar descripciones, tipo o valores predeterminados.
variable "aws_region" {}
```
**Explicación**: No incluir descripciones o tipos específicos puede hacer que el código sea difícil de entender y usar. Las buenas prácticas indican ser explícitos con las variables.

---

#### 3. **Crear una VPC**
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
  cidr_block = "10.0.0.0/16" # Hardcodear el CIDR sin utilizar variables limita la flexibilidad.
  enable_dns_support = true
  enable_dns_hostnames = true
}
```
**Explicación**: Hardcodear el CIDR del bloque de red limita la flexibilidad del template, ya que si se desea cambiar, se debe modificar directamente el código.

---

#### 4. **Crear una subred pública**
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
  # Olvidar la zona de disponibilidad puede causar problemas si se desea un despliegue en varias zonas.
}
```
**Explicación**: No especificar la `availability_zone` puede generar despliegues indeseados o complicados en caso de tener zonas de disponibilidad específicas.

---

#### 5. **Crear una instancia EC2**
**Correcto:**
```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"  # Cambia esto por la AMI de tu preferencia
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
  instance_type = "t2.micro"  # Hardcodear el tipo de instancia sin usar variables limita la reutilización.
  subnet_id     = aws_subnet.public.id
  tags = {
    Name = "ExampleInstance"
  }
  # No agregar seguridad ni configuración básica de User Data puede ser un riesgo de seguridad.
}
```
**Explicación**: Hardcodear el tipo de instancia limita la flexibilidad del código. Además, no configurar adecuadamente la seguridad ni el `user_data` deja el sistema vulnerable.

---

#### 6. **Outputs**
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
  # No se proporciona una descripción, lo que dificulta la comprensión del output.
}
```
**Explicación**: Es importante proporcionar descripciones para los outputs para hacer que el código sea más entendible, especialmente en proyectos colaborativos.

---

#### 7. **Manejo de estado remoto**
**Correcto:**
```hcl
terraform {
  backend "s3" {
    bucket = "mi-bucket-de-terraform"
    key    = "terraform/statefile"
    region = "us-east-2"
    dynamodb_table = "terraform-lock"
    encrypt = true
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
**Explicación**: Usar el backend local puede funcionar para proyectos pequeños, pero para equipos o proyectos grandes, es recomendable utilizar un backend remoto (como S3 con DynamoDB) para evitar conflictos de estado y garantizar la integridad del archivo de estado.

---
