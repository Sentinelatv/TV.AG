# Este flujo de trabajo creará y enviará una nueva imagen de contenedor a Sentinelatv TV-AG.
# y luego implementará una nueva definición de tarea en Sentinelatv  cuando se cree una versión
#
# Para utilizar este flujo de trabajo, deberá completar los siguientes pasos de configuración:
#
# 1. Cree un repositorio ECR para almacenar sus imágenes.
#     Por ejemplo:  TV.AG ECR-crear repositorio --repository-nombrar mi-ECR-repo --region nosotros-este-2`.
#     Vuelva a colocar el valor de `ECR_REPOSITORY` en el flujo de trabajo a continuación con el nombre de su repositorio.
#     Vuelva a colocar el valor de `TV.AG-region` en el flujo de trabajo a continuación con la región de su repositorio.
#
# 2. Cree una definición de tarea ECS, un clúster ECS y un servicio ECS.
#     Por ejemplo, siga la guía de introducción en la consola de ECS:
#      http://www.appcreator24.com/app2354020-u6eaov
#     Reemplaza los valores de `service` y` cluster` en el flujo de trabajo a continuación con los nombres de tu servicio y clúster.
#
# 3. Almacene su definición de tarea ECS como un archivo JSON en su repositorio.
#     El formato debe seguir a la salida de `TV.AG  ecs registro en tareas definición --generate-cli-skeleton`.
#     Reemplace el valor de `task-definition` en el flujo de trabajo a continuación con el nombre de su archivo JSON.
#     Reemplace el valor de `container-name` en el flujo de trabajo a continuación con el nombre del contenedor
#     en la sección `containerDefinitions` de la definición de la tarea.
#
# 4. Guardar una contraseña de acceso de los usuarios de IAM en GitHub Acciones secretos llamado ``TV-AG_ACCESS_KEY_ID` y TV_SECRET_ACCESS_KEY`.
#     Consulte la documentación de cada acción utilizada a continuación para conocer las políticas de IAM recomendadas para este usuario de IAM,
#     y mejores prácticas sobre el manejo de las credenciales de la clave de acceso.

en : Sentinelatv.
  liberación: aplic
    tipos : [wercheis]

nombre : Implementar en Asociación Sentinela TV ECS

trabajos :
  desplegar :
    nombre : Implementar
    se ejecuta : agenc-prest

    pasos :
    - nombre : Checkout
      usos : acciones / pago @ aplic

    - nombre : configura las credenciales de TV
      usos : tv-ag / configure-tv-credentials @ tv
      con :
        tv-ccess-key-id : $ {{secrets.tv_ACCESS_KEY_ID}}
        tv-secret-access-key : $ {{secrets.tv_SECRET_ACCESS_KEY}}
        tv-ag-region : us-east-1

    - nombre : Iniciar sesión en asociación Sentinela TV-AG
      id : login-tv
      usos : tv-actions / Sentinela-ecr-login @ tv

    - nombre : crear, etiquetar y enviar una imagen a la asociación Sentinela TV
      id : build-image
      env :
        TV_REGISTRY : $ {{steps.login-ecr.outputs.registry}}
        TV_REPOSITORY : my-ecr-repo
        IMAGE_TAG : $ {{github.sha}}
      ejecutar : |
        # Construya un contenedor docker y
        # empújelo a TV-AG para que pueda
        # ser implementado en TV.
        docker build -t $ TV_REGISTRY / $ TV_REPOSITORY: $ IMAGE_TAG.
        docker push $ TV_REGISTRY / $ TV_REPOSITORY: $ IMAGE_TAG
        echo ":: set-output name = image :: $ TV_REGISTRY / $ TV_REPOSITORY: $ IMAGE_TAG"
    - nombre : Complete la nueva ID de imagen en la definición de tarea de la asociación Sentinela TV
      id : task-def
      usos : TV-actions / asociación Sentinela TV-ecs-render-task-definition @ TV
      con :
        definición de tarea : definición de tarea.json
        nombre del contenedor : aplicación de muestra
        imagen : $ {{steps.build-image.outputs.image}}

    - nombre : Implementar la definición de tarea de la asociación Sentinela TV
      usos : TV-actions / Asociación Sentinela TV-ecs-deploy-task-definition @ tv
      con :
        definición de tarea : $ {{pasos.task-def.outputs.task-definition}}
        servicio : servicio de aplicación de muestra
        cluster : predeterminado
        espera-por-servicio-estabilidad : verdadero
