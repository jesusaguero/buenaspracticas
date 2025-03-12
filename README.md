# Buenas Prácticas en AWS CloudFormation


## **CFN-001: Uso de Parámetros en lugar de valores hardcodeados**

### **Cómo debe ser:**
Usar parámetros para evitar valores hardcodeados que pueden cambiar entre entornos.

```yaml
Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
    Description: "Tipo de instancia para la EC2"
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: ami-123456
```

### **Cómo no debe ser:**
Hardcodear valores específicos sin la posibilidad de personalizarlos.

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-123456
```

---

## **CFN-002: Uso de Funciones Intrínsecas para Referencias**

### **Cómo debe ser:**
Usar funciones intrínsecas como `!Ref`, `!GetAtt`, y `!Sub` para hacer referencias a recursos de manera eficiente.

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref MyBucket
      InstanceType: t2.micro
```

### **Cómo no debe ser:**
Referencias directas que no utilizan funciones intrínsecas.

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: "ami-123456"
      InstanceType: t2.micro
```

---

## **CFN-003: Uso de Outputs para Compartir Información de Stack**

### **Cómo debe ser:**
Definir outputs para compartir información útil sobre los recursos creados en un stack.

```yaml
Outputs:
  InstanceId:
    Description: "ID de la instancia EC2"
    Value: !Ref MyInstance
```

### **Cómo no debe ser:**
No definir outputs, lo que dificulta la reutilización y la referencia entre stacks.

```yaml
# No outputs definidos
```

---

## **CFN-004: Evitar el uso de Claves de Acceso en `UserData`**

### **Cómo debe ser:**
No incluir claves de acceso directamente en el `UserData`. Usar roles de IAM para delegar permisos.

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-123456
      IamInstanceProfile: MyInstanceProfile
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          aws s3 cp s3://my-bucket/myfile.txt /home/ec2-user/
```

### **Cómo no debe ser:**
Incluir claves de acceso directamente en el `UserData`, lo cual es un riesgo de seguridad.

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-123456
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          aws s3 cp s3://my-bucket/myfile.txt /home/ec2-user/ --access-key MYACCESSKEY --secret-key MYSECRETKEY
```

---

## **CFN-005: Uso de Roles IAM con Principios de Menor Privilegio**

### **Cómo debe ser:**
Crear roles IAM con los permisos mínimos necesarios y asignarlos a los recursos.

```yaml
Resources:
  MyRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: "Allow"
            Principal:
              Service: "ec2.amazonaws.com"
            Action: "sts:AssumeRole"
      Policies:
        - PolicyName: "MyPolicy"
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
              - Effect: "Allow"
                Action:
                  - "s3:GetObject"
                Resource: "arn:aws:s3:::my-bucket/*"
```

### **Cómo no debe ser:**
Asumir permisos excesivos en los roles.

```yaml
Resources:
  MyRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: "Allow"
            Principal:
              Service: "ec2.amazonaws.com"
            Action: "sts:AssumeRole"
      Policies:
        - PolicyName: "MyPolicy"
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
              - Effect: "Allow"
                Action: "*"
                Resource: "*"
```

---

## **CFN-006: Uso de Mappings para Configuración Regional**

### **Cómo debe ser:**
Utilizar **Mappings** para manejar configuraciones específicas de la región.

```yaml
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-123456
    us-west-2:
      AMI: ami-654321
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", AMI]
```

### **Cómo no debe ser:**
No utilizar **Mappings** y definir configuraciones específicas dentro del recurso.

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-123456
```

---

## **CFN-007: Uso de Stacks Anidados para Modularización**

### **Cómo debe ser:**
Utilizar **Stacks Anidados** para dividir plantillas grandes en módulos más manejables.

```yaml
Resources:
  MyNestedStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: "https://s3.amazonaws.com/my-bucket/my-template.yaml"
```

### **Cómo no debe ser:**
Colocar todos los recursos en una sola plantilla sin modularización.

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-123456
```

---

## **CFN-008: Validación de Parámetros con Constraints**

### **Cómo debe ser:**
Usar restricciones de parámetros para garantizar que los valores sean válidos.

```yaml
Parameters:
  InstanceType:
    Type: String
    AllowedValues:
      - t2.micro
      - t2.small
    Description: "Tipo de instancia EC2"
```

### **Cómo no debe ser:**
No validar parámetros y permitir valores incorrectos.

```yaml
Parameters:
  InstanceType:
    Type: String
    Description: "Tipo de instancia EC2"
```

---

## **CFN-009: Descripción Detallada de Recursos**

### **Cómo debe ser:**
Incluir una descripción clara para cada recurso.

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-123456
    Metadata:
      Description: "Instancia EC2 utilizada para el entorno de desarrollo"
```

### **Cómo no debe ser:**
Omitir descripciones de recursos.

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-123456
```

---
