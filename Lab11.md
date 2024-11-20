# Diseño de Infraestructura en la Nube para un eCommerce Integrado con Servicios Bancarios

## **Parte 1: Diseño de Infraestructura**
### **Componentes de la Arquitectura**
1. **Cómputo: AWS EC2**
   - **Uso**: Alojar la aplicación backend que maneja la lógica del eCommerce y la integración con servicios bancarios.
   - Configuración:
     - Crear un grupo de Auto Scaling para escalar instancias dinámicamente según la demanda.
     - Balanceador de carga (ALB) para distribuir solicitudes entre instancias.
     - Utilizar Amazon Machine Images (AMIs) para entornos homogéneos.

2. **Almacenamiento: AWS S3**
   - **Uso**: Almacenar activos estáticos como imágenes de productos y facturas.
   - Configuración:
     - Cifrado en reposo (AES-256).
     - Activar S3 Transfer Acceleration para cargas rápidas desde ubicaciones globales.

3. **Base de Datos: Amazon RDS**
   - **Uso**: Almacenar datos estructurados como información de usuarios, productos y transacciones.
   - Configuración:
     - Usar PostgreSQL con Multi-AZ para alta disponibilidad.
     - Backups automáticos y snapshots.
     - Habilitar Performance Insights para monitoreo.

4. **Red: AWS VPC**
   - **Uso**: Proporcionar un entorno seguro y segmentado.
   - Configuración:
     - Subred pública: Balanceadores de carga (ALB).
     - Subred privada: Instancias EC2 y base de datos.
     - NAT Gateway para acceso seguro de instancias privadas a internet.

---

## **Parte 2: Configuración de IAM**
### **Roles y Políticas Definidas**
1. **Desarrolladores**
   - **Rol**: `DeveloperRole`.
   - **Política**:
     - Acceso restringido a entornos de desarrollo y repositorios de código.
     - Sin acceso a producción.

2. **QA**
   - **Rol**: `QARole`.
   - **Política**:
     - Acceso a ambientes de pruebas.
     - Permisos de lectura en logs de CloudWatch.

3. **Producción**
   - **Rol**: `ProdRole`.
   - **Política**:
     - Acceso limitado a despliegues en producción a través de herramientas CI/CD.
     - Sin acceso directo a la infraestructura.

4. **Administradores**
   - **Rol**: `AdminRole`.
   - **Política**:
     - Acceso total a recursos con monitoreo obligatorio (MFA activado).

5. **Servidores de Aplicación**
   - **Rol**: `AppServerRole`.
   - **Política**:
     - Permisos para interactuar con RDS, S3 y Secrets Manager.
     - Acceso mínimo necesario a otros servicios.

6. **Integración Bancaria**
   - **Rol**: `BankIntegrationRole`.
   - **Política**:
     - Acceso a claves almacenadas en AWS Secrets Manager.
     - Restricciones estrictas para interactuar con APIs bancarias.

---

## **Parte 3: Gestión de Recursos**
### **Estrategias Implementadas**
1. **Despliegue de Código: AWS CodePipeline**
   - Uso de CodePipeline para gestionar el flujo de CI/CD:
     - Repositorio de código: AWS CodeCommit o GitHub.
     - Build: AWS CodeBuild para construir el proyecto y ejecutar pruebas unitarias.
     - Despliegue: AWS CodeDeploy para entregar a ambientes de QA, staging y producción.

2. **Ambientes Diferenciados**
   - **Desarrollo**:
     - Usar entornos aislados con configuraciones ligeras (instancias t3.micro).
     - Base de datos en Amazon Aurora Serverless.
   - **QA**:
     - Ambiente replicado de producción para pruebas funcionales y de integración.
   - **Producción**:
     - Configuración completa con Auto Scaling, RDS Multi-AZ y ALB.

3. **Monitoreo: Amazon CloudWatch**
   - Configuración:
     - Alarmas para métricas clave (uso de CPU, memoria, errores HTTP 5xx).
     - Logs centralizados de aplicaciones y bases de datos.
   - Integración con Amazon SNS para notificaciones en tiempo real.

4. **Alertas de Monitoreo de Costos**
   - **AWS Budgets**:
     - Configuración de presupuestos para cada componente crítico de la infraestructura (EC2, RDS, S3).
     - Crear alertas para notificar cuando los costos alcanzan un porcentaje del presupuesto asignado (por ejemplo, 80%).
   - **AWS Cost Anomaly Detection**:
     - Identificar gastos atípicos en tiempo real.
     - Configurar reglas para servicios individuales o combinaciones de servicios.
   - **AWS Trusted Advisor**:
     - Monitorear recursos infrautilizados o configurados incorrectamente para reducir costos.

---

## **Parte 4: Seguridad**
1. **Cifrado**
   - **S3**: Cifrado en reposo con AES-256 y en tránsito con SSL.
   - **RDS**: Habilitar TLS/SSL para conexiones seguras.
   - **Secrets Manager**: Gestionar claves y credenciales sensibles.

2. **Control de Acceso**
   - Uso del principio de mínimo privilegio en IAM.
   - Implementación de políticas para acceso segmentado por ambiente.

3. **DDoS Protection**
   - Usar AWS Shield para proteger contra ataques DDoS.
   - Configurar WAF (Web Application Firewall) para filtrar tráfico malicioso.

4. **Auditoría**
   - Activar AWS CloudTrail para registrar todas las actividades en la cuenta.

---

## **Parte 5: Implementación Teórica**
### **Flujo de Datos**
1. **Frontend**:
   - Los usuarios acceden a la aplicación mediante el balanceador de carga ALB.
2. **Backend**:
   - El ALB redirige las solicitudes a las instancias EC2 que procesan la lógica del negocio.
   - Las EC2 acceden a la base de datos RDS para obtener datos del usuario y productos.
3. **Pagos**:
   - Las instancias EC2 interactúan con las APIs bancarias utilizando claves almacenadas en AWS Secrets Manager.
4. **Logs y Monitoreo**:
   - CloudWatch almacena métricas y logs del sistema para análisis.
   - Alertas de costos notifican si se detectan anomalías en el gasto.

```
1. Desarrolladores y QA:
   ┌──────────────┐
   │ Developers   │
   └──────┬───────┘
          │
          │Acceso a repositorios de código y CI/CD
          ▼
   ┌──────────────┐
   │ AWS CodeCommit│
   └──────┬───────┘
          │
          ▼
   ┌──────────────┐         ┌──────────────┐
   │ CodePipeline  │<------>│ CodeBuild     │
   └──────┬───────┘         └──────┬───────┘
          │                         │
          ▼                         ▼
   ┌──────────────┐         ┌──────────────┐
   │ QA Environment│         │ Dev Environment│
   └──────────────┘         └──────────────┘

2. Cliente (Frontend):
   ┌──────────────┐
   │   Cliente    │
   └──────┬───────┘
          │
          ▼
   ┌──────────────┐
   │ Application  │
   │ Load Balancer│
   └──────┬───────┘
          │
          ▼

3. Backend:
   ┌──────────────┐         ┌──────────────┐
   │  EC2 Instance│<------->│ Secrets Manager│
   └──────┬───────┘         └──────────────┘
          │
          ▼

4. Base de Datos:
   ┌──────────────┐
   │     RDS      │
   └──────────────┘

5. Almacenamiento de Activos:
   ┌──────────────┐
   │      S3      │
   └──────────────┘

6. Monitoreo y Seguridad:
   ┌──────────────┐         ┌──────────────┐
   │ CloudWatch   │<------->│ CloudTrail    │
   └──────────────┘         └──────────────┘

7. Alertas de Costo:
   ┌──────────────┐
   │ AWS Budgets  │
   └──────────────┘
```
---

## **Parte 6: Evaluación**
1. **Resiliencia**:
   - Multi-AZ en RDS y Auto Scaling garantizan alta disponibilidad.
2. **Seguridad**:
   - IAM segmentado y cifrado protegen datos sensibles.
3. **Escalabilidad**:
   - Auto Scaling y ALB manejan variaciones de tráfico.
4. **Control de Costos**:
   - Alertas configuradas en AWS Budgets y Cost Anomaly Detection brindan un control preventivo de gastos.
