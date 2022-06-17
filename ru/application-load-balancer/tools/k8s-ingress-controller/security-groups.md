# Настройка групп безопасности Ingress-контроллера {{ alb-name }}

{% include [security-groups-note](../../_includes_service/security-groups-note.md) %}

Для корректной работы [Ingress-контроллера](index.md) нужно настроить [группы безопасности](../../../vpc/concepts/security-groups.md) [кластера](../../../managed-kubernetes/concepts/index.md#kubernetes-cluster) и [групп узлов {{ managed-k8s-full-name }}](../../../managed-kubernetes/concepts/index.md#node-group) и балансировщика {{ alb-name }}. Для кластера, групп узлов и балансировщика можно использовать разные группы безопасности (рекомендуется) или одну и ту же группу.

В группах безопасности должны быть настроены:
* Все стандартные правила, описанные в соответствующих разделах документации:
  * для кластера и групп узлов — в разделе [{#T}](../../../managed-kubernetes/operations/connect/security-groups.md) документации {{ managed-k8s-name }};
  * для балансировщика — в разделе [{#T}](../../concepts/application-load-balancer.md#security-groups). Последнее правило, для исходящего трафика на виртуальные машины бэкендов, должно разрешать соединения в подсети [групп узлов](../../../managed-kubernetes/concepts/index.md#node-group) кластера или его группу безопасности.
* Правила для проверок состояния бэкендов, разрешающие:
  * балансировщику — отправлять трафик узлам кластера по протоколу TCP на порт 10501 (назначение трафика — подсе́ти или группы безопасности групп узлов кластера);
  * группам узлов — принимать этот трафик (источник трафика — подсе́ти балансировщика или его группа безопасности).

Группы безопасности кластера и групп узлов указываются в их настройках. Подробнее см. инструкции:
* по [созданию](../../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-create.md) и [изменению](../../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-update.md#update-cluster) кластера;
* по [созданию](../../../managed-kubernetes/operations/node-group/node-group-create.md) и [изменению](../../../managed-kubernetes/operations/node-group/node-group-update.md) группы узлов.

Идентификаторы групп безопасности балансировщика указываются в аннотации `ingress.alb.yc.io/security-groups` ресурса `Ingress`. Если балансировщик создается по нескольким `Ingress`, ему назначаются все указанные в этих `Ingress` группы безопасности.

## Пример настройки {#example}

Приведем пример для следующих условий:
* Требуется развернуть балансировщик с публичным IP-адресом, принимающий HTTPS-трафик, в трех подсетях, имеющих CIDR `10.128.0.0/24`, `10.129.0.0/24` и `10.130.0.0/24`, — далее они помечаются \[Б\].
* При создании кластера были указаны CIDR кластера `10.96.0.0/16` \[К\] и CIDR сервисов `10.112.0.0/16` \[С\].
* Группа узлов в кластере расположена в подсети́, имеющей CIDR `10.140.0.0/24` \[Узл\].
* Подключаться к узлам по SSH и управлять кластером через API, `kubectl` и другие утилиты можно только из CIDR `203.0.113.0/24` \[Упр\].

Тогда в группах безопасности нужно создать следующие правила:
* Группа безопасности кластера и группы узлов для служебного трафика:

  {% list tabs %}

  - Исходящий трафик

    Диапазон портов | Протокол | Тип назначения | Назначение | Описание
    --- | --- | --- | --- | ---
    Весь (`{{ port-any }}`) | Любой (`Any`) | CIDR | `0.0.0.0/0` | Для всего исходящего трафика

  - Входящий трафик

    Диапазон портов | Протокол | Тип источника | Источник | Описание
    --- | --- | --- | --- | ---
    Весь (`{{ port-any }}`) | TCP | CIDR | `198.18.235.0/24`<br>`198.18.248.0/24` | Для сетевого балансировщика нагрузки
    Весь (`{{ port-any }}`) | Любой (`Any`) | Группа безопасности | Текущая (`Self`) | Для трафика между [мастером](../../../managed-kubernetes/concepts/index.md#master) и узлами
    Весь (`{{ port-any }}`) | Любой (`Any`) | CIDR | `10.96.0.0/16`[^\[К\]^](#example)<br>`10.112.0.0/16`[^\[С\]^](#example) | Для трафика между [подами](../../../managed-kubernetes/concepts/index.md#pod) и [сервисами](../../../managed-kubernetes/concepts/index.md#service)
    Весь (`{{ port-any }}`) | ICMP | CIDR | `10.0.0.0/8`<br>`192.168.0.0/16`<br>`172.16.0.0/12` | Для проверки работоспособности узлов из подсетей внутри {{ yandex-cloud }}

  {% endlist %}

* Группа безопасности группы узлов для подключения к сервисам из интернета:

  {% list tabs %}

  - Входящий трафик

    Диапазон портов | Протокол | Тип источника | Источник | Описание
    --- | --- | --- | --- | ---
    `30000-32767` | TCP | CIDR | `0.0.0.0/0` | Для доступа к сервисам из интернета и подсетей {{ yandex-cloud }}

  {% endlist %}

* Группа безопасности группы узлов для подключения к узлам по SSH:

  {% list tabs %}

  - Входящий трафик

    Диапазон портов | Протокол | Тип источника | Источник | Описание
    --- | --- | --- | --- | ---
    `30000-32767` | TCP | CIDR | `203.0.113.0/24`[^\[Упр\]^](#example) | Для подключения к узлам по SSH

  {% endlist %}

* Группа безопасности кластера для доступа к API {{ k8s }}:

  {% list tabs %}

  - Исходящий трафик

    Диапазон портов | Протокол | Тип назначения | Назначение | Описание
    --- | --- | --- | --- | ---
    `443` | TCP | CIDR | `203.0.113.0/24`[^\[Упр\]^](#example) | Для доступа к API {{ k8s }}
    `6443` | TCP | CIDR | `203.0.113.0/24`[^\[Упр\]^](#example) | Для доступа к API {{ k8s }}

  {% endlist %}

* Группа безопасности группы узлов для проверок состояния бэкендов:

  {% list tabs %}

  - Входящий трафик

    Диапазон портов | Протокол | Тип источника | Источник | Описание
    --- | --- | --- | --- | ---
    `10501` | TCP | CIDR | `10.128.0.0/24`[^\[Б\]^](#example)<br>`10.129.0.0/24`[^\[Б\]^](#example)<br>`10.130.0.0/24`[^\[Б\]^](#example) | Для проверок состояния бэкендов

  {% endlist %}

* Группа безопасности балансировщика:

  {% list tabs %}

  - Исходящий трафик

    Диапазон портов | Протокол | Тип назначения | Назначение | Описание
    --- | --- | --- | --- | ---
    Весь (`{{ port-any }}`) | TCP | CIDR | `10.140.0.0/24`[^\[Узл\]^](#example) | Для отправки трафика, в том числе для проверок состояния, на узлы

  - Входящий трафик

    Диапазон портов | Протокол | Тип источника | Источник | Описание
    --- | --- | --- | --- | ---
    `80` | TCP | CIDR | `0.0.0.0/0` | Для получения входящего HTTP-трафика
    `443` | TCP | CIDR | `0.0.0.0/0` | Для получения входящего HTTPS-трафика
    `30080` | TCP | CIDR | `198.18.235.0/24`<br>`198.18.248.0/24` | Для проверок состояния узлов балансировщика

  {% endlist %}