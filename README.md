Buenas Prácticas en AWS CloudFormation

(CFN-001) Uso de Plantillas Declarativas

CloudFormation utiliza plantillas declarativas para definir la infraestructura como código. Se recomienda estructurar las plantillas de la siguiente manera:

Template Version: 2010-09-09

Description: Breve descripción del template.

Resources: Definir los recursos necesarios.

Parameters: Usar parámetros para hacer la plantilla reutilizable.

Outputs: Exportar valores clave para otros stacks.

Mappings: Definir valores fijos según entorno.

Rules: Evaluar parámetros antes de crear el stack.

Conditions: Usar condiciones para decidir si crear un recurso.

Metadata: Incluir información adicional, configuraciones personalizadas.

(CFN-002) Uso de Stack, StackSet y Nested Stack

Stack: Implementación de una plantilla en una cuenta/región.

StackSet: Permite desplegar una plantilla en múltiples regiones y cuentas.

Nested Stack: Divide grandes stacks en sub-stacks modulares.

(CFN-003) Validación de Plantillas

Antes de desplegar una plantilla:

Validate Template: Verifica sintaxis JSON/YAML.

CFN Lint: Valida sintaxis y lógica de los recursos.

(CFN-004) Uso de Pseudo-Parámetros

CloudFormation proporciona pseudo-parámetros predefinidos como:

AWS::AccountId

AWS::Region

AWS::StackId

Se recomienda su uso para evitar valores hardcodeados.

(CFN-005) Implementación de Macros en CloudFormation

Las macros permiten modificar plantillas antes del despliegue.

Se definen en la sección Transform.

Pueden aplicarse a:

Etiquetado automático de recursos.

Prefijos en nombres de recursos.

Cifrado forzado en buckets de S3 y bases de datos.

Creación dinámica de múltiples recursos.

Transformación YAML a JSON.

(CFN-006) Uso de AWS CloudFormation Metadata

La metadata permite almacenar información adicional y configurar instancias EC2 después de su creación.

AWS::CloudFormation::Init: Configurar instancias EC2.

AWS::CloudFormation::Interface: Personalizar experiencia en la consola.

AWS::CloudFormation::Designer: Almacenar información visual de los recursos.

(CFN-007) Estrategias de Despliegue

Rolling Update: Actualización gradual sin afectar disponibilidad.

Blue/Green Deployment: Dos entornos separados para minimizar riesgos.

(CFN-008) Uso de AWS Config y CloudWatch

AWS Config: Audita cambios en la configuración de recursos.

CloudWatch: Monitoreo en tiempo real del rendimiento y logs.

(CFN-009) Aplicación de Buenas Prácticas de Seguridad

Habilitar cifrado en buckets de S3, volúmenes EBS y bases de datos RDS.

Aplicar etiquetas estándar a todos los recursos.

Definir políticas de seguridad automáticas.

(CFN-010) Uso de AWS SAM para Aplicaciones Serverless

AWS::Serverless-2016-10-31 simplifica la definición de funciones Lambda y API Gateway.

Se recomienda usarlo para reducir el código necesario.

(CFN-011) Uso de StackSets en Escenarios Empresariales

Multi-Región: Empresas que requieren la misma infraestructura en varias regiones.

Apps a Gran Escala: Despliegue de infraestructura estándar en cuentas de clientes.

