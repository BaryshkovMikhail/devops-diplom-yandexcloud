# Дипломный практикум в Yandex.Cloud - Барышков Михаил
  * [Цели:](#цели)
  * [Этапы выполнения:](#этапы-выполнения)
     * [Создание облачной инфраструктуры](#создание-облачной-инфраструктуры)
     * [Создание Kubernetes кластера](#создание-kubernetes-кластера)
     * [Создание тестового приложения](#создание-тестового-приложения)
     * [Подготовка cистемы мониторинга и деплой приложения](#подготовка-cистемы-мониторинга-и-деплой-приложения)
     * [Установка и настройка CI/CD](#установка-и-настройка-cicd)
  * [Что необходимо для сдачи задания?](#что-необходимо-для-сдачи-задания)
  * [Как правильно задавать вопросы дипломному руководителю?](#как-правильно-задавать-вопросы-дипломному-руководителю)

**Перед началом работы над дипломным заданием изучите [Инструкция по экономии облачных ресурсов](https://github.com/netology-code/devops-materials/blob/master/cloudwork.MD).**

---
## Цели:

1. Подготовить облачную инфраструктуру на базе облачного провайдера Яндекс.Облако.
2. Запустить и сконфигурировать Kubernetes кластер.
3. Установить и настроить систему мониторинга.
4. Настроить и автоматизировать сборку тестового приложения с использованием Docker-контейнеров.
5. Настроить CI для автоматической сборки и тестирования.
6. Настроить CD для автоматического развёртывания приложения.


### 🗂️ Структура проекта

```text

yandex-cloud-diploma/
│
├── README.md                          # Общая документация проекта
├── .gitignore                         # Исключаем секреты и локальные файлы
│
├── 📁 bootstrap/                      # 🔄 Начальная настройка (запускается первым)
│   │                                  # Создаёт: сервисный аккаунт + S3 bucket для backend
│   ├── main.tf                        # Ресурсы: service account, bucket, права
│   ├── variables.tf                   # Переменные (токены, ID облака)
│   ├── outputs.tf                     # Вывод: access_key, secret_key, bucket_name
│   ├── providers.tf                   # Провайдер Yandex (без backend)
│   ├── terraform.tfvars.example       # Шаблон для переменных (копируется пользователем)
│   └── .gitignore                     # Исключаем terraform.tfvars
│
├── 📁 infrastructure/                 # 🏗️ Основная инфраструктура (запускается вторым)
│   │                                  # Создаёт: VPC, подсети, security groups
│   ├── main.tf                        # Ресурсы: network, subnets, security groups
│   ├── variables.tf                   # Переменные конфигурации
│   ├── outputs.tf                     # Вывод: IDs созданных ресурсов
│   ├── providers.tf                   # Провайдер + backend (S3 из bootstrap)
│   ├── terraform.tfvars.example       # Шаблон переменных
│   └── .gitignore                     # Исключаем секреты
│
├── 📁 docs/                           # 📄 Материалы для дипломной работы
│   ├── screenshots/                   # Папка для скриншотов
│   │   ├── 01_bootstrap_apply.png
│   │   ├── 02_vpc_console.png
│   │   ├── 03_terraform_state.png
│   │   └── 04_service_account_roles.png
│   └── explanations.md                # Текстовые пояснения для вставки в диплом
│
└── 📁 scripts/                        # 🔧 Вспомогательные скрипты (опционально)
    ├── init-backend.sh                # Автоматизация инициализации backend
    └── validate-config.sh             # Проверка конфигурации перед apply

```

---
## Этапы выполнения:


### Создание облачной инфраструктуры

Для начала необходимо подготовить облачную инфраструктуру в ЯО при помощи [Terraform](https://www.terraform.io/).

Особенности выполнения:

- Бюджет купона ограничен, что следует иметь в виду при проектировании инфраструктуры и использовании ресурсов;
Для облачного k8s используйте региональный мастер(неотказоустойчивый). Для self-hosted k8s минимизируйте ресурсы ВМ и долю ЦПУ. В обоих вариантах используйте прерываемые ВМ для worker nodes.

Предварительная подготовка к установке и запуску Kubernetes кластера.

1. Создайте сервисный аккаунт, который будет в дальнейшем использоваться Terraform для работы с инфраструктурой с необходимыми и достаточными правами. Не стоит использовать права суперпользователя
2. Подготовьте [backend](https://developer.hashicorp.com/terraform/language/backend) для Terraform:  
   а. Рекомендуемый вариант: S3 bucket в созданном ЯО аккаунте(создание бакета через TF)
   б. Альтернативный вариант:  [Terraform Cloud](https://app.terraform.io/)
3. Создайте конфигурацию Terrafrom, используя созданный бакет ранее как бекенд для хранения стейт файла. Конфигурации Terraform для создания сервисного аккаунта и бакета и основной инфраструктуры следует сохранить в разных папках.
4. Создайте VPC с подсетями в разных зонах доступности.
5. Убедитесь, что теперь вы можете выполнить команды `terraform destroy` и `terraform apply` без дополнительных ручных действий.
6. В случае использования [Terraform Cloud](https://app.terraform.io/) в качестве [backend](https://developer.hashicorp.com/terraform/language/backend) убедитесь, что применение изменений успешно проходит, используя web-интерфейс Terraform cloud.

Ожидаемые результаты:

1. Terraform сконфигурирован и создание инфраструктуры посредством Terraform возможно без дополнительных ручных действий, стейт основной конфигурации сохраняется в бакете или Terraform Cloud
2. Полученная конфигурация инфраструктуры является предварительной, поэтому в ходе дальнейшего выполнения задания возможны изменения.

## Решение :

### Раздел: Проектирование облачной инфраструктуры
Для автоматизации развёртывания инфраструктуры в облаке Yandex.Cloud использовался инструмент Infrastructure as Code — Terraform. Архитектура конфигураций разделена на два логических этапа:

1. **Bootstrap-этап**: создание сервисного аккаунта с минимально необходимыми правами (роль `editor`) и bucket в Object Storage для хранения state-файла. Такой подход соответствует принципу наименьших привилегий: сервисный аккаунт не имеет прав суперпользователя, что снижает риски при компрометации учетных данных.
2. **Infrastructure-этап**: развёртывание сетевой инфраструктуры — VPC с подсетями в трёх зонах доступности (`ru-central1-a`, `ru-central1-b`, `ru-central1-d`) для обеспечения отказоустойчивости. Все подсети настроены с включённым NAT для обеспечения исходящего доступа к интернету, необходимого для загрузки контейнерных образов и обновлений.

Для хранения состояния Terraform использовался remote backend на базе S3-совместимого хранилища Yandex Object Storage с включённым шифрованием AES256. Это позволяет обеспечить целостность state-файла и возможность командной работы с инфраструктурой.
Все чувствительные данные (токены, ключи доступа) передаются через переменные окружения или защищённые файлы `terraform.tfvars`, исключённые из системы контроля версий через `.gitignore`. Это соответствует лучшим практикам безопасности при работе с облачными провайдерами.
Особое внимание уделено оптимизации затрат: в конфигурацию заложена возможность использования прерываемых виртуальных машин (preemptible instances) для worker-узлов будущего Kubernetes-кластера, что позволяет сократить расходы на вычислительные ресурсы до 80% без потери функциональности в тестовом окружении.

### Раздел: Реализация безопасного хранения состояния Terraform
Для обеспечения целостности и конфиденциальности state-файла инфраструктуры был использован remote backend на базе Yandex Object Storage с обязательным шифрованием. Конфигурация основана на проверенных паттернах из предыдущих этапов работы:

1. **KMS-шифрование:** Создан симметричный ключ `yandex_kms_symmetric_key` с алгоритмом AES-128 и периодом ротации 1 год. Ключ используется в блоке `server_side_encryption_configuration` ресурса бакета с параметром `sse_algorithm = "aws:kms"`, что является единственным поддерживаемым значением в провайдере Yandex Cloud.
2. **Организация ресурсов:** Всем ресурсам явно указан `folder_id`, что обеспечивает корректный учёт затрат в рамках папки проекта и соответствие требованиям организационной структуры облака.
3. **Минимальные привилегии:** Сервисный аккаунт `terraform-sa` имеет только роль `editor` на уровне папки, без прав администратора каталога, что снижает поверхность атаки при компрометации учетных данных.
4. **Управление жизненным циклом:** На бакете настроена политика автоматического удаления старых версий объектов через 30 дней, что предотвращает неконтролируемый рост затрат на хранение.

Предупреждение о депрекации атрибута `acl` принято осознанно: в учебном проекте приоритетом является работоспособность конфигурации, а миграция на `yandex_storage_bucket_grant` может быть выполнена на этапе промышленной эксплуатации.

---
### 📦 ЭТАП 1.1: Bootstrap — Сервисный аккаунт и S3 Backend

### [bootstrap/providers.tf](bootstrap/providers.tf)
### [bootstrap/variables.tf](bootstrap/variables.tf)
### [bootstrap/main.tf](bootstrap/main.tf)
### [bootstrap/outputs.tf](bootstrap/outputs.tf)

### 🔄 Выполнение этапа Bootstrap

### 📋 Пошаговая инструкция:

```bash
# 1. Перейдите в директорию bootstrap
cd bootstrap

# 2. Инициализация Terraform (state будет локальным)
terraform init

# 3. Проверка конфигурации
terraform validate
terraform fmt -check
```

![img1](bootstrap/img/img1.png)

``` bash
# 4. Просмотр плана изменений
terraform plan -out=tfplan
```

![img2](bootstrap/img/img2.png)

```bash
# 5. Применение конфигурации
terraform apply tfplan

# 6. Сохраните выходные значения для следующего этапа
terraform output -json > ../infrastructure/bootstrap-outputs.json
```

![img3](bootstrap/img/img3.png)

### Консоль KMS → Созданный ключ шифрования

![img4-1](bootstrap/img/img4-1.png)

### Консоль KMS → Созданный сервисный аккаунт

![img5](bootstrap/img/img5.png)

### Консоль KMS → Созданный бакет

![img4](bootstrap/img/img4.png)
![img6](bootstrap/img/img6.png)
![img7](bootstrap/img/img7.png)


### 🏗️ ЭТАП 1.2: Infrastructure — VPC и сетевая инфраструктура

Теперь создаём основную инфраструктуру, используя bucket из bootstrap как backend.

### 🗂️ Структура папки infrastructure/

```text
infrastructure/
├── main.tf           # VPC, подсети, security groups
├── variables.tf      # Переменные конфигурации
├── outputs.tf        # Выходные значения
├── providers.tf      # Провайдер + backend конфигурация
├── terraform.tfvars  # Шаблон переменных
└── .gitignore        # Исключаем секреты
```

### [infrastructure/providers.tf](infrastructure/providers.tf)
### [infrastructure/variables.tf](infrastructure/variables.tf)
### [infrastructure/main.tf](infrastructure/main.tf)
### [infrastructure/outputs.tf](infrastructure/outputs.tf)

### Раздел: Результаты развёртывания сетевой инфраструктуры
После успешной инициализации backend и валидации конфигурации была выполнена команда terraform apply, в результате которой в облаке Яндекс.Облако созданы следующие ресурсы:

1. VPC Network diploma-dev-network с CIDR-блоком 10.0.0.0/16 — изолированная сетевая среда проекта.
2. Три подсети в зонах доступности ru-central1-a, ru-central1-b, ru-central1-d с включённым NAT для обеспечения исходящего доступа к интернету. Такое распределение обеспечивает отказоустойчивость будущего Kubernetes-кластера: при недоступности одной зоны мастер-ноды и рабочие узлы продолжат работу в остальных зонах.
3. Группа безопасности diploma-dev-k8s-nodes-sg с правилами межсетевого экрана, разрешающими:
   -  Порт 22 (SSH) — для административного доступа к узлам;
   - Порты 80/443 (HTTP/HTTPS) — для входящего трафика приложений;
   - Диапазон 30000-32767 (NodePort) — для сервисов Kubernetes типа NodePort;
   - Внутренний трафик в пределах CIDR сети — для коммуникации компонентов кластера.

Состояние инфраструктуры (terraform state) автоматически сохраняется в защищённый bucket Object Storage с шифрованием через KMS, что обеспечивает целостность данных и возможность командной работы. Все чувствительные параметры передаются через CLI-аргументы и не хранятся в репозитории, что соответствует лучшим практикам безопасности.

Вместо устаревшего параметра nat = true использована современная архитектура маршрутизации через yandex_vpc_gateway с типом shared_egress_gateway. Это соответствует рекомендациям Яндекс.Облако для организации исходящего доступа из приватных подсетей.


### 🔄 Пошаговая инструкция запуска

```bash
# 1. Перейдите в директорию infrastructure
cd ~/git/homework/devops-diplom-yandexcloud/infrastructure


# 2. Инициализация с настройкой backend
# Извлеките значения из bootstrap-outputs.json (только для локального использования!)
export TF_BACKEND_ACCESS_KEY=$(jq -r '.backend_access_key.value' bootstrap-outputs.json)
export TF_BACKEND_SECRET_KEY=$(jq -r '.backend_secret_key.value' bootstrap-outputs.json)
export TF_BUCKET_NAME=$(jq -r '.bucket_name.value' bootstrap-outputs.json)

# Проверьте, что переменные установлены:
echo "Bucket: $TF_BUCKET_NAME"
echo "Access Key: ${TF_BACKEND_ACCESS_KEY:0:10}..."  # Покажет только первые 10 символов

terraform init \
 -backend-config="bucket=${TF_BUCKET_NAME}" \
 -backend-config="access_key=${TF_BACKEND_ACCESS_KEY}" \
 -backend-config="secret_key=${TF_BACKEND_SECRET_KEY}"
```

![img0](infrastructure/img/img0.png)

```bash
# 5. Проверка конфигурации


terraform validate
```

![img1](infrastructure/img/img1.png)

```bash
terraform plan -out=tfplan
```

![img2](infrastructure/img/img2.png)

```bash
# 6. Применение инфраструктуры

terraform apply tfplan
```

![img3](infrastructure/img/img3.png)

### Консоль → VPC → Сети → diploma-dev-network


![img4](infrastructure/img/img4.png)

### Консоль → VPC → Группы безопасности → diploma-dev-k8s-nodes-sg

![img5](infrastructure/img/img5.png)
![img6](infrastructure/img/img6.png)


### Консоль → Object Storage → diploma-tfstate-bucket-unique → terraform/infrastructure.tfstate

![img7](infrastructure/img/img7.png)

---
### Создание Kubernetes кластера

На этом этапе необходимо создать [Kubernetes](https://kubernetes.io/ru/docs/concepts/overview/what-is-kubernetes/) кластер на базе предварительно созданной инфраструктуры.   Требуется обеспечить доступ к ресурсам из Интернета.

Это можно сделать двумя способами:

1. Рекомендуемый вариант: самостоятельная установка Kubernetes кластера.  
   а. При помощи Terraform подготовить как минимум 3 виртуальных машины Compute Cloud для создания Kubernetes-кластера. Тип виртуальной машины следует выбрать самостоятельно с учётом требовании к производительности и стоимости. Если в дальнейшем поймете, что необходимо сменить тип инстанса, используйте Terraform для внесения изменений.  
   б. Подготовить [ansible](https://www.ansible.com/) конфигурации, можно воспользоваться, например [Kubespray](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/)  
   в. Задеплоить Kubernetes на подготовленные ранее инстансы, в случае нехватки каких-либо ресурсов вы всегда можете создать их при помощи Terraform.
2. Альтернативный вариант: воспользуйтесь сервисом [Yandex Managed Service for Kubernetes](https://cloud.yandex.ru/services/managed-kubernetes)  
  а. С помощью terraform resource для [kubernetes](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/kubernetes_cluster) создать **региональный** мастер kubernetes с размещением нод в разных 3 подсетях      
  б. С помощью terraform resource для [kubernetes node group](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/kubernetes_node_group)
  
Ожидаемый результат:

1. Работоспособный Kubernetes кластер.
2. В файле `~/.kube/config` находятся данные для доступа к кластеру.
3. Команда `kubectl get pods --all-namespaces` отрабатывает без ошибок.

---
### Создание тестового приложения

Для перехода к следующему этапу необходимо подготовить тестовое приложение, эмулирующее основное приложение разрабатываемое вашей компанией.

Способ подготовки:

1. Рекомендуемый вариант:  
   а. Создайте отдельный git репозиторий с простым nginx конфигом, который будет отдавать статические данные.  
   б. Подготовьте Dockerfile для создания образа приложения.  
2. Альтернативный вариант:  
   а. Используйте любой другой код, главное, чтобы был самостоятельно создан Dockerfile.

Ожидаемый результат:

1. Git репозиторий с тестовым приложением и Dockerfile.
2. Регистри с собранным docker image. В качестве регистри может быть DockerHub или [Yandex Container Registry](https://cloud.yandex.ru/services/container-registry), созданный также с помощью terraform.

---
### Подготовка cистемы мониторинга и деплой приложения

Уже должны быть готовы конфигурации для автоматического создания облачной инфраструктуры и поднятия Kubernetes кластера.  
Теперь необходимо подготовить конфигурационные файлы для настройки нашего Kubernetes кластера.

Цель:
1. Задеплоить в кластер [prometheus](https://prometheus.io/), [grafana](https://grafana.com/), [alertmanager](https://github.com/prometheus/alertmanager), [экспортер](https://github.com/prometheus/node_exporter) основных метрик Kubernetes.
2. Задеплоить тестовое приложение, например, [nginx](https://www.nginx.com/) сервер отдающий статическую страницу.

Способ выполнения:
1. Воспользоваться пакетом [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus), который уже включает в себя [Kubernetes оператор](https://operatorhub.io/) для [grafana](https://grafana.com/), [prometheus](https://prometheus.io/), [alertmanager](https://github.com/prometheus/alertmanager) и [node_exporter](https://github.com/prometheus/node_exporter). Альтернативный вариант - использовать набор helm чартов от [bitnami](https://github.com/bitnami/charts/tree/main/bitnami).

### Деплой инфраструктуры в terraform pipeline

1. Если на первом этапе вы не воспользовались [Terraform Cloud](https://app.terraform.io/), то задеплойте и настройте в кластере [atlantis](https://www.runatlantis.io/) для отслеживания изменений инфраструктуры. Альтернативный вариант 3 задания: вместо Terraform Cloud или atlantis настройте на автоматический запуск и применение конфигурации terraform из вашего git-репозитория в выбранной вами CI-CD системе при любом комите в main ветку. Предоставьте скриншоты работы пайплайна из CI/CD системы.

Ожидаемый результат:
1. Git репозиторий с конфигурационными файлами для настройки Kubernetes.
2. Http доступ на 80 порту к web интерфейсу grafana.
3. Дашборды в grafana отображающие состояние Kubernetes кластера.
4. Http доступ на 80 порту к тестовому приложению.
5. Atlantis или terraform cloud или ci/cd-terraform
---
### Установка и настройка CI/CD

Осталось настроить ci/cd систему для автоматической сборки docker image и деплоя приложения при изменении кода.

Цель:

1. Автоматическая сборка docker образа при коммите в репозиторий с тестовым приложением.
2. Автоматический деплой нового docker образа.

Можно использовать [teamcity](https://www.jetbrains.com/ru-ru/teamcity/), [jenkins](https://www.jenkins.io/), [GitLab CI](https://about.gitlab.com/stages-devops-lifecycle/continuous-integration/) или GitHub Actions.

Ожидаемый результат:

1. Интерфейс ci/cd сервиса доступен по http.
2. При любом коммите в репозиторие с тестовым приложением происходит сборка и отправка в регистр Docker образа.
3. При создании тега (например, v1.0.0) происходит сборка и отправка с соответствующим label в регистри, а также деплой соответствующего Docker образа в кластер Kubernetes.

---
## Что необходимо для сдачи задания?

1. Репозиторий с конфигурационными файлами Terraform и готовность продемонстрировать создание всех ресурсов с нуля.
2. Пример pull request с комментариями созданными atlantis'ом или снимки экрана из Terraform Cloud или вашего CI-CD-terraform pipeline.
3. Репозиторий с конфигурацией ansible, если был выбран способ создания Kubernetes кластера при помощи ansible.
4. Репозиторий с Dockerfile тестового приложения и ссылка на собранный docker image.
5. Репозиторий с конфигурацией Kubernetes кластера.
6. Ссылка на тестовое приложение и веб интерфейс Grafana с данными доступа.
7. Все репозитории рекомендуется хранить на одном ресурсе (github, gitlab)

