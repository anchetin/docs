# Создать образ из другого пользовательского образа

Чтобы создать образ из другого пользовательского образа:

{% list tabs %}

- Консоль управления

  1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором нужно создать образ.
  1. Выберите сервис **{{ compute-name }}**.
  1. Перейдите на вкладку **Образы**.
  1. В строке с нужным снимком нажмите значок ![image](../../../_assets/horizontal-ellipsis.svg) и выберите в меню команду **Создать образ**.
  1. Введите имя образа.

     * Длина — от 2 до 63 символов.
     * Может содержать строчные буквы латинского алфавита, цифры и дефисы.
     * Первый символ — буква. Последний символ — не дефис.

  1. Если требуется, укажите произвольное текстовое описание образа.
  1. Чтобы создать [оптимизированный образ](../../concepts/image.md#images-optimized-for-deployment), включите опцию **Оптимизировать для развертывания**.
  1. Нажмите **Создать**.

- CLI

  {% include [cli-install](../../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

  1. Посмотрите описание команды CLI для создания образа:
  
      ```
      yc compute image create --help
      ```
  
  1. Получите список образов в каталоге по умолчанию:
  
      {% include [compute-image-list](../../../_includes/compute/image-list.md) %}
  
  1. Выберите идентификатор (`ID`) или имя (`NAME`) нужного снимка.
  1. Создайте образ в каталоге по умолчанию:
  
      ```
      yc compute image create \
          --name new-image \
          --source-image-name first-image \
          --description "new image via yc"
      ```
  
      Данная команда создаст образ с именем `new-image` и описанием `new image via yc` из образа `first-image`.

      Чтобы создать [оптимизированный образ](../../concepts/image.md#images-optimized-for-deployment), используйте флаг `--pooled`:

      ```
      yc compute image create \
          --name new-image \
          --source-image-name first-image \
          --description "new image via yc" \
          --pooled
      ```

- Terraform

  Если у вас еще нет Terraform, [установите его и настройте провайдер {{ yandex-cloud }}](../../../solutions/infrastructure-management/terraform-quickstart.md#install-terraform).

  1. Опишите в конфигурационном файле параметры ресурса `yandex_compute_image`.

     Пример структуры конфигурационного файла:

     ```
     resource "yandex_compute_image" "image-1" {
       name         = "<имя образа>"
       source_image = "<идентификатор образа-источника>"
     }
     ```

     Более подробную информацию о ресурсах, которые вы можете создать с помощью Terraform, см. в [документации провайдера](https://www.terraform.io/docs/providers/yandex/index.html).

  1. Проверьте корректность конфигурационных файлов.

     1. В командной строке перейдите в папку, где вы создали конфигурационный файл.
     1. Выполните проверку с помощью команды:

        ```bash
        terraform plan
        ```

       Если конфигурация описана верно, в терминале отобразится список создаваемых ресурсов и их параметров. Если в конфигурации есть ошибки, Terraform на них укажет.

  1. Разверните облачные ресурсы.

     1. Выполните команду:

        ```bash
        terraform apply
        ```

     1. Подтвердите создание ресурсов.

     После этого в указанном каталоге будут созданы все требуемые ресурсы. Проверить появление ресурсов и их настройки можно в [консоли управления]({{ link-console-main }}).

- API

  1. Получите список образов с помощью метода [ImageService/List](../../api-ref/grpc/image_service.md#List) gRPC API или метода [list](../../api-ref/Image/list.md) ресурса `Image` REST API.
  1. Создайте новый образ с помощью метода [ImageService/Create](../../api-ref/grpc/image_service.md#Create) gRPC API или метода [create](../../api-ref/Image/create.md) ресурса `Image` REST API. В запросе укажите идентификатор образа-источника.

{% endlist %}

После создания образ перейдет в статус `CREATING`. Дождитесь, когда образ перейдет в статус `READY`, прежде чем его использовать.