# Buenas Prácticas en AWS CloudFormation

## (CFN-001) Uso de Plantillas Declarativas

CloudFormation utiliza plantillas declarativas para definir la infraestructura como código. Se recomienda estructurar las plantillas de la siguiente manera:

- **Template Version**: `2010-09-09`
- **Description**: Breve descripción del template.
- **Resources**: Definir los recursos necesarios.
- **Parameters**: Usar parámetros para hacer la plantilla reutilizable.
- **Outputs**: Exportar valores clave para otros stacks.
- **Mappings**: Definir valores fijos según entorno.
- **Rules**: Evaluar parámetros antes de crear el stack.
- **Conditions**: Usar condiciones para decidir si crear un recurso.
- **Metadata**: Incluir información adicional, configuraciones personalizadas.

**Ejemplo correcto:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: "Ejemplo de plantilla bien estructurada"
Parameters:
  InstanceType:
    Type: String
    Default: t3.micro
Resources:
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: ami-12345678
```

**Ejemplo incorrecto:** (Valores hardcodeados, sin descripción ni estructura clara)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: "t3.micro"
      ImageId: "ami-12345678"
```

---

## (CFN-002) Uso de Stack, StackSet y Nested Stack

- **Stack**: Implementación de una plantilla en una cuenta/región.
- **StackSet**: Permite desplegar una plantilla en múltiples regiones y cuentas.
- **Nested Stack**: Divide grandes stacks en sub-stacks modulares.

**Ejemplo correcto:** Uso de Nested Stacks para modularización
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: "https://s3.amazonaws.com/mybucket/network.yaml"
```

**Ejemplo incorrecto:** Definir todo en un solo archivo (poco mantenible)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: "10.0.0.0/16"
  MySubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: "10.0.1.0/24"
```

---

## (CFN-003) Validación de Plantillas

Antes de desplegar una plantilla:

- `Validate Template`: Verifica sintaxis JSON/YAML.
- `CFN Lint`: Valida sintaxis y lógica de los recursos.

**Ejemplo correcto:** Uso de `cfn-lint`
```sh
cfn-lint template.yaml
```

**Ejemplo incorrecto:** Desplegar sin validación previa
```sh
aws cloudformation deploy --template-file template.yaml --stack-name MiStack
```

---

## (CFN-004) Uso de Pseudo-Parámetros

CloudFormation proporciona pseudo-parámetros predefinidos como:

- `AWS::AccountId`
- `AWS::Region`
- `AWS::StackId`

**Ejemplo correcto:**
```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "mi-bucket-${AWS::AccountId}"
```

**Ejemplo incorrecto:** Uso de valores hardcodeados
```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: "mi-bucket-123456789012"
```

---

## (CFN-005) Implementación de Macros en CloudFormation

Las macros permiten modificar plantillas antes del despliegue.

**Ejemplo correcto:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: MyMacro
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
```

**Ejemplo incorrecto:** No usar macros cuando son necesarias
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      Tags:
        - Key: "Environment"
          Value: "Production"
```

---

## (CFN-006) Uso de AWS CloudFormation Metadata

**Ejemplo correcto:**
```yaml
Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: "Configuración de Red"
        Parameters:
          - VpcId
```

**Ejemplo incorrecto:** No usar metadata

---

## (CFN-007) Estrategias de Despliegue

- **Rolling Update**: Actualización gradual sin afectar disponibilidad.
- **Blue/Green Deployment**: Dos entornos separados para minimizar riesgos.

**Ejemplo correcto:** Rolling Update en Auto Scaling Group
```yaml
UpdatePolicy:
  AutoScalingRollingUpdate:
    MinInstancesInService: 1
    MaxBatchSize: 2
```

---

## (CFN-008) Uso de AWS Config y CloudWatch

- **AWS Config**: Audita cambios en la configuración de recursos.
- **CloudWatch**: Monitoreo en tiempo real del rendimiento y logs.

**Ejemplo correcto:**
```yaml
Resources:
  MyAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmDescription: "Alerta cuando CPU excede 80%"
      MetricName: CPUUtilization
      Namespace: AWS/EC2
      Threshold: 80
```

---

## (CFN-009) Aplicación de Buenas Prácticas de Seguridad

- Habilitar cifrado en buckets de S3, volúmenes EBS y bases de datos RDS.
- Aplicar etiquetas estándar a todos los recursos.
- Definir políticas de seguridad automáticas.

**Ejemplo correcto:**
```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
```
