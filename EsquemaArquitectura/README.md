

### 1. API Gateway - Punto de Entrada Único
- Actúa como el único punto de entrada para todas las solicitudes de clientes.
- Valida cada solicitud utilizando JWT para autenticar usuarios.
- Enruta las solicitudes a los microservicios específicos según la configuración.

### 2. Microservicios
Cada microservicio es independiente y se despliega en su propio entorno usando EC2, con su propia base de datos en Amazon RDS.

- **Catálogo de Productos**:
    - Proporciona una API RESTful para el acceso a datos de productos.
    - Almacena datos en una base de datos RDS.
    - Utiliza SQS para notificar otros servicios sobre cambios en inventario.
    - Utiliza tambien Redis para cacheo de datos.
  
- **Gestión de Usuarios**:
    - Maneja autenticación, perfiles y emite JWT tras la autenticación del usuario.
    - Almacena la información en RDS.
    - Otros servicios pueden validar autenticación mediante JWT y obtener perfiles de usuario.

- **Procesamiento de Órdenes**:
    - Gestiona la lógica de órdenes, historial de compras y pagos.
    - Interactúa con **Gestión de Usuarios** para autenticación y **Catálogo de Productos** para inventario.
   
- **Atención al Cliente**:
    - Proporciona soporte basado en el historial de órdenes y perfil del usuario.
    - Consulta **Gestión de Usuarios** y **Procesamiento de Órdenes** para obtener datos en tiempo real.

### 3. Comunicación Asíncrona mediante SQS
Para facilitar la comunicación y la sincronización de eventos entre microservicios:
- SQS se utilizan para enviar mensajes asíncronos entre microservicios, lo que permite una arquitectura desacoplada.

## ¿Cómo interactúan los servicios?


## Ejemplo Completo de Flujo de Solicitud

1. El cliente inicia sesión en el sistema y recibe un JWT.
2. El cliente realiza una solicitud de compra. El **API Gateway** verifica el JWT y redirige la solicitud al microservicio de **Procesamiento de Órdenes**.
3. **Procesamiento de Órdenes** consulta el **Catálogo de Productos** para verificar inventario y con **Gestión de Usuarios** para autenticación adicional si es necesario.
4. Una vez procesada la orden, **Procesamiento de Órdenes** envía un evento a **SNS/SQS** notificando la actualización de la orden.
