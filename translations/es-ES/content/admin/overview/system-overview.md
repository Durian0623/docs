---
title: Descripción del sistema
intro: 'El {% data variables.product.prodname_ghe_server %} es la copia privada de tu organización de {% data variables.product.prodname_dotcom %} contenida dentro de un aparato virtual, alojada localmente o en la nube, que configuras y controlas.'
redirect_from:
  - /enterprise/admin/installation/system-overview
  - /enterprise/admin/overview/system-overview
versions:
  ghes: '*'
type: overview
topics:
  - Enterprise
  - Fundamentals
  - Infrastructure
  - Security
  - Storage
---

## Arquitectura de almacenamiento

El {% data variables.product.prodname_ghe_server %} requiere dos volúmenes de almacenamiento, uno instalado en la ruta del *sistema de archivos raíz* (`/`) y otro en la ruta del *sistema de archivos del usuario* (`/data/user`). Esta arquitectura simplifica los procedimientos de actualización, reversión y recuperación al separar el entorno del software que se ejecuta de los datos de aplicación persistentes.

El sistema de archivos raíz está incluido en la imagen de máquina distribuida. Contiene el sistema operativo base y el entorno de aplicación {% data variables.product.prodname_ghe_server %}. El sistema de archivos raíz debería tratarse como efímero. Cualquier dato en el sistema de archivos raíz será reemplazado cuando se actualice con futuros lanzamientos del {% data variables.product.prodname_ghe_server %}.

El volumen de almacenamiento raíz se divide en dos particiones del mismo tamaño. Una de las particiones se montará como el sistema de archivos raíz (`/`). La otra partición solo se montará durante mejoras y reversiones de mejoras como `/mnt/upgrade`, para hacer que dichas reversiones se lleven a cabo más fácilmente en caso de que sea necesario. Por ejemplo, si se asigna un volumen raíz de 200GB, 100GB se asignarán al sistema de archivos raíz y otros 100GB se reservarán para las mejoras y reversiones.

El sistema de archivos raíz contiene:
  - Los certificados de autoridad de certificación personalizados (CA) (en */usr/local/share/ca-certificates*)
  - Las configuraciones de red personalizadas
  - Las configuraciones de firewall personalizadas
  - El estado de replicación

El sistema de archivos del usuario contiene la configuración y los datos del usuario, tales como:
  - Repositorios Git
  - Bases de datos
  - Índices de búsqueda
  - Contenido publicado en los sitios {% data variables.product.prodname_pages %}
  - Archivos grandes de {% data variables.large_files.product_name_long %}
  - Entornos de enlaces de pre-recepción

## Opciones de implementación

Puedes implementar {% data variables.product.prodname_ghe_server %} como un aparato virtual único, o en una configuración de alta disponibilidad. Para obtener más información, consulta "[Configurar {% data variables.product.prodname_ghe_server %} para alta disponibilidad](/admin/enterprise-management/configuring-high-availability)."

Algunas organizaciones con decenas de miles de programadores podrían también beneficiarse de una Agrupación {% data variables.product.prodname_ghe_server %}. Para obtener más información, consulta "[Acerca de las agrupaciones](/admin/enterprise-management/configuring-clustering/about-clustering)."

## Retención de datos y redundancia de centro de datos

{% danger %}

Antes de usar {% data variables.product.prodname_ghe_server %} en un entorno de producción, recomendamos firmemente que configures copias de seguridad y un plan de recuperación ante desastres. Para obtener más información, consulta "[Configurar copias de seguridad en tu aparato](/admin/configuration/configuring-your-enterprise/configuring-backups-on-your-appliance)"

{% enddanger %}

{% data variables.product.prodname_ghe_server %} incluye soporte para copias de seguridad en línea e incrementales con [{% data variables.product.prodname_enterprise_backup_utilities %}](https://github.com/github/backup-utils). Puedes tomar instantáneas incrementales sobre un enlace de red seguro (el puerto administrativo SSH) sobre grandes distancias para el almacenamiento externo o geográficamente disperso. Puedes restaurar instantáneas a través de la red en un nuevo aparato virtual recientemente aprovisionado al momento de la recuperación en el caso de un desastre en el centro de datos principal.

Además se admiten las copias de seguridad de red, las instantáneas de disco AWS (EBS) y VMware de los volúmenes de almacenamiento del usuario mientras que el aparato está fuera de línea o en modo mantenimiento. Las instantáneas de volumen regulares pueden usarse como una alternativa de bajo costo y baja complejidad para las copias de seguridad de red con {% data variables.product.prodname_enterprise_backup_utilities %} si tus requisitos de nivel de servicio permiten un mantenimiento fuera de línea regular.

Para obtener más información, consulta "[Configurar copias de seguridad en tu aparato](/admin/configuration/configuring-your-enterprise/configuring-backups-on-your-appliance)"

## Seguridad

El {% data variables.product.prodname_ghe_server %} es un aparato virtual que se ejecuta en tu infraestructura y está gobernado por tus controles de seguridad de información existentes, como cortafuegos, IAM, monitoreo y VPN. Usar el {% data variables.product.prodname_ghe_server %} puede ayudarte a evitar problemas de cumplimiento regulatorio que surgen de las soluciones basadas en la nube.

El {% data variables.product.prodname_ghe_server %} también incluye características de seguridad adicionales.

- [Sistema operativo, software y parches](#operating-system-software-and-patches)
- [Seguridad de la red](#network-security)
- [Seguridad de la aplicación](#application-security)
- [Servicios externos y acceso de soporte](#external-services-and-support-access)
- [Comunicación encriptada](#encrypted-communication)
- [Usuarios y permisos de acceso](#users-and-access-permissions)
- [Autenticación](#authentication)
- [Auditoría y registro de acceso](#audit-and-access-logging)

### Sistema operativo, software y parches

El {% data variables.product.prodname_ghe_server %} ejecuta un sistema operativo Linux personalizado con las aplicaciones y los servicios necesarios únicamente. El {% data variables.product.prodname_dotcom %} gestiona el parche del sistema operativo central del aparato como parte de su ciclo estándar de lanzamiento de productos. Los parches abordan problemas de funcionalidad, de estabilidad y de seguridad no críticos para las aplicaciones de {% data variables.product.prodname_dotcom %}. {% data variables.product.prodname_dotcom %} también proporciona parches de seguridad críticos según se necesita fuera del ciclo de lanzamiento regular.

{% data variables.product.prodname_ghe_server %} se proporciona como un aplicativo y muchos de los paquetes de los sistemas operativos se modifican en comparación con la distribución común de Debian. No ofrecemos compatibilidad con la modificación del sistema operativo subyacente por esta razón (incluyendo las mejoras de los sistemas operativos), lo cual se alinea con la [licencia de {% data variables.product.prodname_ghe_server %} y el acuerdo de soporte](https://enterprise.github.com/license), bajo las exclusiones de la sección 11.3.

Actualmente, la base del aplicativo de {% data variables.product.prodname_ghe_server %} es Debian 9 (Stretch) y recibe soporte bajo el programa de Soporte a Largo Plazo de Debian.  Existen planes para migrarse a un sistema operativo base nuevo antes del final del periodo de Debian LTS para Stretch.

Las actualizaciones de parche regulares se lanzan en la página de [lanzamientos](https://enterprise.github.com/releases) de {% data variables.product.prodname_ghe_server %} y la página de [notas de lanzamiento](/admin/release-notes) proporciona más información sobre esto. Estos parches a menudo contienen un proveedor de nivel superior y parches de seguridad de proyecto después de que se prueban y que nuestro equipo de ingeniería aprueba su calidad. Puede haber una pequeña demora en tiempo desde cuando la actualización de nivel superior se lanza hasta cuando se prueba y se empaqueta en un lanzamiento de parche futuro de {% data variables.product.prodname_ghe_server %}.

### Seguridad de la red

El cortafuegos interno del {% data variables.product.prodname_ghe_server %} restringe el acceso de la red a los servicios del aparato. Están disponibles en la red únicamente los servicios necesarios para que el aparato funcione. Para obtener más información, consulta "[Puertos de red](/admin/configuration/configuring-network-settings/network-ports)."

### Seguridad de la aplicación

El equipo de seguridad de la aplicación de {% data variables.product.prodname_dotcom %} se centra en la evaluación de vulnerabilidad, la prueba de penetración y la revisión del código para los productos de {% data variables.product.prodname_dotcom %} , incluido el {% data variables.product.prodname_ghe_server %}. {% data variables.product.prodname_dotcom %} también contrata firmas de seguridad externas para proporcionar evaluaciones de seguridad puntuales de los productos de {% data variables.product.prodname_dotcom %}.

### Servicios externos y acceso de soporte

El {% data variables.product.prodname_ghe_server %} puede funcionar sin ningún acceso de salida de tu red a servicios externos. De forma opcional, puedes habilitar la integración con servicios externos para la entrega de correo electrónico, el monitoreo externo y el reenvío de bitácoras. Para obtener más información, consulta las secciones "[Configurar las notificaciones por correo electrónico](/admin/configuration/configuring-your-enterprise/configuring-email-for-notifications)", "[Configurar el monitoreo externo](/admin/enterprise-management/monitoring-your-appliance/setting-up-external-monitoring)", y "[Reenvío de bitácoras](/admin/monitoring-activity-in-your-enterprise/exploring-user-activity/log-forwarding)".

Puedes recopilar y enviar manualmente datos de resolución de problemas a {% data variables.contact.github_support %}. Para obtener más información, consulta "[Proporcionar datos a {% data variables.contact.github_support %}](/support/contacting-github-support/providing-data-to-github-support)".

### Comunicación encriptada

{% data variables.product.prodname_dotcom %} diseña {% data variables.product.prodname_ghe_server %} para ejecutar detrás de tu cortafuegos corporativo. Para asegurar la comunicación a través del cable, te alentamos a habilitar la seguridad de la capa de transporte (TLS). El {% data variables.product.prodname_ghe_server %} admite certificados TLS comerciales de 2048 bits y superiores para el tráfico HTTPS. Para obtener más información, consulta la sección "[Configurar el TLS](/admin/configuration/configuring-network-settings/configuring-tls)".

Por defecto, el aparato también ofrece acceso a Secure Shell (SSH) para el acceso al repositorio utilizando Git y con fines administrativos. Para obtener más información, consulta "[Acerca de SSH](/authentication/connecting-to-github-with-ssh/about-ssh)" y "[Acceder al shell administrativo (SSH)](/admin/configuration/configuring-your-enterprise/accessing-the-administrative-shell-ssh)."

{% ifversion ghes > 3.3 %}

Si configuras la autenticación de SAML para {% data variables.product.product_location %}, puedes habilitar las aserciones cifradas entre la instancia y tu IdP de SAML. Para obtener más información, consulta la sección "[Utilizar SAML](/admin/identity-and-access-management/authenticating-users-for-your-github-enterprise-server-instance/using-saml#enabling-encrypted-assertions)".

{% endif %}

### Usuarios y permisos de acceso

El {% data variables.product.prodname_ghe_server %} proporciona tres tipos de cuentas.

- La cuenta de usuario de Linux del `administrador` ha controlado el acceso al sistema operativo subyacente, incluido el sistema de archivos directo y el acceso a la base de datos. Un pequeño conjunto de administradores de confianza debería tener acceso a esta cuenta, a la que pueden acceder por medio de SSH. Para obtener más información, consulta "[Acceder al shell administrativo (SSH)](/admin/configuration/configuring-your-enterprise/accessing-the-administrative-shell-ssh)".
- Las cuentas de usuario en la aplicación web del aparato tienen acceso completo a sus propios datos y a cualquier dato que otros usuarios u organizaciones concedan de manera explícita.
- Los administradores del sitio en la aplicación web del aparato son cuentas de usuario que pueden administrar los ajustes de aplicaciones web y de aparatos de alto nivel, la configuración de cuenta de usuario y de organización y los datos del repositorio.

Para más información sobre los permisos de usuario del {% data variables.product.prodname_ghe_server %}, consulta "[Permisos de acceso en GitHub](/get-started/learning-about-github/access-permissions-on-github)."

### Autenticación

El {% data variables.product.prodname_ghe_server %} proporciona cuatro métodos de autenticación.

- La autenticación de claves públicas SSH proporciona acceso del repositorio usando Git y el shell administrativo. Para obtener más información, consulta "[Acerca de SSH](/authentication/connecting-to-github-with-ssh/about-ssh)" y "[Acceder al shell administrativo (SSH)](/admin/configuration/configuring-your-enterprise/accessing-the-administrative-shell-ssh)."
- El nombre de usuario y la autenticación de contraseña con cookies HTTP proporciona acceso a la aplicación web y la gestión de sesiones, con autenticación opcional de dos factores (2FA). Para obtener más información, consulta ña sección "[Utilizar la autenticación integrada](/admin/identity-and-access-management/authenticating-users-for-your-github-enterprise-server-instance/using-built-in-authentication)".
- La autenticación externa LDAP, SAML o CAS mediante un servicio LDAP, SAML Identity Provider (IdP) u otro servicio compatible proporciona acceso a la aplicación web. Para más información, consulta "[Autenticar usuarios para tu instancia de servidor de GitHub Enterprise](/admin/identity-and-access-management/authenticating-users-for-your-github-enterprise-server-instance)."
- OAuth y los token de acceso personal proporcionan acceso a los datos del repositorio de Git y a API para clientes externos y servicios. Para obtener más información, consulta la sección "[Crear un token de acceso personal](/github/authenticating-to-github/creating-a-personal-access-token)".

### Auditoría y registro de acceso

El {% data variables.product.prodname_ghe_server %} almacena tanto registros tradicionales de sistema operativo como de aplicación. La aplicación también escribe registros de auditoría y de seguridad detallados, que el {% data variables.product.prodname_ghe_server %} almacena de forma permanente. Puedes reenviar ambos tipos de bitácoras en tiempo real a varios destinos a través del protocolo `syslog-ng`. Para obtener más información, consulta la sección "[Reenvío de bitácoras](/admin/monitoring-activity-in-your-enterprise/exploring-user-activity/log-forwarding)".

Los registros de acceso y de auditoría incluyen información como la siguiente.

#### Registros de acceso

- Registros completos de servidor web tanto para el navegador como para el acceso a la API
- Registros completos para acceder a los datos del repositorio por medio de protocolos Git, HTTPS y SSH
- Registros de acceso administrativo por medio de HTTPS y SSH

#### Registros de auditoría

- Inicios de sesión del usuario, restablecimientos de contraseña, solicitudes 2FA, cambios en la configuración del correo electrónico y cambios en aplicaciones autorizadas y API
- Acciones de administrador del sitio, como desbloquear cuentas de usuario y repositorios
- Eventos push de repositorio, permisos de acceso, transferencias y renombres
- Cambios de membresía de la organización, incluida la creación y la destrucción de equipo

## Dependencias de código abierto para {% data variables.product.prodname_ghe_server %}

Puedes consultar una lista completa de dependencias en la versión de tu aparato de {% data variables.product.prodname_ghe_server %}, y la licencia de cada proyecto, en `http(s)://HOSTNAME/site/credits`.

Están disponibles en tu aparato los tarballes con una lista completa de dependencias y metadatos asociados:
- Para conocer las dependencias comunes a todas las plataformas, ingresa en `/usr/local/share/enterprise/dependencies-<GHE version>-base.tar.gz`.
- Para conocer las dependencias específicas de una plataforma, ingresa en `/usr/local/share/enterprise/dependencies-<GHE version>-<platform>.tar.gz`.

También están disponibles los tarballes, con una lista completa de las dependencias y los metadatos, en `https://enterprise.github.com/releases/<version>/download.html`.

## Leer más

- "[Configurar una prueba de {% data variables.product.prodname_ghe_server %}](/get-started/signing-up-for-github/setting-up-a-trial-of-github-enterprise-server)"
- "[Configurar una instancia de {% data variables.product.prodname_ghe_server %}](/admin/installation/setting-up-a-github-enterprise-server-instance)"
- [ {% data variables.product.prodname_roadmap %} ]({% data variables.product.prodname_roadmap_link %}) en el repositorio `github/roadmap`
