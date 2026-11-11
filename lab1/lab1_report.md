# Лабораторная работа №1. Обзор Google Cloud и исследование основных сервисов

 

---

## 1. Цель работы

Ознакомиться с основными возможностями облачной платформы Google Cloud, изучить работу с сервисами Identity and Access Management (IAM), Compute Engine и Cloud Storage, а также проверить влияние изменения ролей сервисного аккаунта на доступ к облачным ресурсам.

## 2. Ход работы

### 2.1. Подключение к Google Cloud

Был получен доступ к учебному проекту Google Cloud `cloud-platforms-as-the-basis`.

После авторизации выполнен переход в Google Cloud Console и проверена доступность основных сервисов платформы.

### 2.2. Создание сервисного аккаунта

В разделе IAM & Admin → Service Accounts был создан сервисный аккаунт со следующими параметрами:

- **Name:** `asurudzhian-sa-lab1`
- **Role:** `Storage Admin`
- **Status:** Enabled

Роль Storage Admin предоставляет разрешения для управления объектами и бакетами Google Cloud Storage.

После создания аккаунта его наличие и назначенные разрешения были проверены через IAM.

### 2.3. Создание виртуальной машины

Через сервис Compute Engine была создана виртуальная машина со следующими параметрами:

| Параметр | Значение |
|---|---|
| Name | asurudzhian-vm-lab1 |
| Region | us-central1 |
| Zone | us-central1-a |
| Machine type | e2-micro |
| Provisioning model | Spot |
| Operating system | Debian GNU/Linux 13 |
| Service account | asurudzhian-sa-lab1 |
| Access scopes | Allow full access to all Cloud APIs |

Использование режима Spot позволяет снизить стоимость вычислительных ресурсов, однако такая виртуальная машина может быть принудительно остановлена облачным провайдером.

После создания виртуальная машина перешла в состояние Running.

### 2.4. Подключение к VM и получение файлов

Подключение к виртуальной машине осуществлялось через SSH-in-browser.

Для просмотра содержимого бакета была выполнена команда:

```bash
gcloud storage ls gs://lab1-bucket-itmo/
```

В результате обнаружены три файла:

```text
gs://lab1-bucket-itmo/pic1.jpg
gs://lab1-bucket-itmo/pic2.jpg
gs://lab1-bucket-itmo/pic3.jpeg
```

Для сохранения файлов создана локальная директория:

```bash
mkdir -p ~/lab1-files
```

Копирование выполнено командой:

```bash
gcloud storage cp gs://lab1-bucket-itmo/* ~/lab1-files/
```

Результат выполнения:

```text
Completed files 3/3
```

Для проверки наличия файлов использована команда:

```bash
ls -lah ~/lab1-files/
```

Результат:

```text
pic1.jpg    228K
pic2.jpg    487K
pic3.jpeg   858K
```

Все три файла были успешно скопированы на виртуальную машину.

Дополнительно выполнена команда:

```bash
gcloud auth list
```

Установлено, что активным аккаунтом является:

```text
asurudzhian-sa-lab1@cloud-platforms-as-the-basis.iam.gserviceaccount.com
```

Таким образом, команды gcloud выполнялись с использованием созданного сервисного аккаунта.

### 2.5. Изменение роли сервисного аккаунта

В разделе IAM роль сервисного аккаунта была изменена:

- **Исходная роль:** Storage Admin
- **Новая роль:** Compute Viewer

Роль Compute Viewer предназначена для просмотра информации о вычислительных ресурсах Google Cloud и сама по себе не предоставляет разрешений на чтение объектов Cloud Storage.

После изменения роли была предпринята повторная попытка копирования файлов в новую директорию:

```bash
mkdir -p ~/lab1-after-role
gcloud storage cp gs://lab1-bucket-itmo/* ~/lab1-after-role/
ls -lah ~/lab1-after-role/
```

Результат:

```text
Completed files 3/3
```

Несмотря на изменение роли, файлы были успешно скопированы повторно.

### 2.6. Исследование причины сохранения доступа

Для дополнительной проверки была выполнена команда:

```bash
gcloud storage cp gs://lab1-bucket-itmo/pic1.jpg ~/lab1-role-check.jpg
```

Файл был успешно загружен.

После этого проведена проверка доступности объекта без авторизации:

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://storage.googleapis.com/lab1-bucket-itmo/pic1.jpg
```

Результат:

```text
200
```

HTTP-статус `200 OK` подтверждает, что объект `pic1.jpg` доступен по публичному URL без авторизации.

Следовательно, успешное повторное скачивание данного файла не свидетельствует о наличии у роли Compute Viewer разрешений на чтение Cloud Storage.

Для остальных объектов успешное копирование было зафиксировано, однако их публичная доступность отдельно не проверялась.

В результате эксперимента установлено, что фактический доступ к облачному объекту зависит не только от роли сервисного аккаунта, но и от других действующих разрешений и настроек доступа самого ресурса.

### 2.7. Удаление созданных ресурсов

После завершения эксперимента выполнена очистка учебного проекта от созданных ресурсов.

Удалены:

1. Виртуальная машина `asurudzhian-vm-lab1` вместе с загрузочным диском.
2. Сервисный аккаунт `asurudzhian-sa-lab1`.

Удаление выполнено через Google Cloud Console.

## 3. Вывод

В ходе лабораторной работы были изучены основные сервисы Google Cloud: IAM, Compute Engine и Cloud Storage.

Были получены практические навыки создания сервисных аккаунтов, назначения ролей, настройки виртуальных машин, подключения по SSH и использования утилиты gcloud для работы с облачным хранилищем.

Успешно выполнено копирование трёх файлов из Cloud Storage на виртуальную машину и проверено их наличие с помощью команды `ls -lah`.

Эксперимент с изменением роли Storage Admin на Compute Viewer показал, что успешное скачивание файлов сохранилось. Дополнительная проверка подтвердила публичную доступность одного из объектов, что продемонстрировало необходимость учитывать не только IAM-роли, но и настройки доступа непосредственно к ресурсам.

После выполнения работы созданные ресурсы были удалены.

**Цель лабораторной работы достигнута.**

---

## 4. Скриншоты выполнения лабораторной работы

1. [Создание сервисного аккаунта — аккаунт asurudzhian-sa-lab1 со статусом Enabled]
2. <img width="850" height="51" alt="01_service_account_created" src="https://github.com/user-attachments/assets/d3f6bd98-c927-4dc4-aa5f-6e77aec2c907" />


3. [Назначение роли Storage Admin сервисному аккаунту]
4. <img width="848" height="63" alt="02_storage_admin_role" src="https://github.com/user-attachments/assets/03e48ae0-7074-4633-ace2-990678ecad66" />


5. [Настройка виртуальной машины и сервисного аккаунта]
6. <img width="1470" height="956" alt="03_vm_configuration" src="https://github.com/user-attachments/assets/c333db0e-35f4-427c-bf50-d8fe07aedbf3" />


7. [Конфигурация виртуальной машины: e2-micro и Spot]
8. <img width="1470" height="956" alt="04_vm_configuration" src="https://github.com/user-attachments/assets/2ed78bb4-6e4d-48a6-a14c-f77a0794e537" />


9. [Успешное создание виртуальной машины со статусом Running]<img width="1470" height="956" alt="05_vm_running" src="https://github.com/user-attachments/assets/b15d73bb-06aa-456c-ac3b-486cd1b6e8a5" />


10. [Просмотр трёх файлов в бакете lab1-bucket-itmo через gcloud]
11. <img width="1470" height="956" alt="06_bucket_contents" src="https://github.com/user-attachments/assets/545cf779-1d6a-4e2c-bffb-7d469649c58e" />


12. [Копирование файлов на VM, проверка через ls -lah и gcloud auth list]<img width="1470" height="956" alt="07_files_copied" src="https://github.com/user-attachments/assets/42716307-4aad-40e9-bc87-8aa0e662b4a7" />


13. [Изменение роли сервисного аккаунта на Compute Viewer]<img width="1470" height="956" alt="08_compute_viewer_role" src="https://github.com/user-attachments/assets/d7f672d4-ee05-402b-8d50-2c2c5ef28111" />


14. [Повторное копирование файлов после изменения роли — операция завершилась успешно]
15. <img width="1470" height="956" alt="09_access_denied" src="https://github.com/user-attachments/assets/36e29dfb-0c95-41d2-97cf-837cfd4ba579" />

16. [Дополнительная проверка доступа к объекту pic1.jpg]<img width="1468" height="84" alt="10_role_access_check" src="https://github.com/user-attachments/assets/18c82ab6-5035-4ef6-ba68-f52eda95787c" />

17. [Проверка публичного доступа через curl — HTTP 200 OK]<img width="1196" height="64" alt="11_public_access png" src="https://github.com/user-attachments/assets/8b5dafc5-5565-4e5e-8e3b-05258cd1125d" />
18. [Удаление созданной виртуальной машины](<img width="1470" height="956" alt="12_vm_deleted" src="https://github.com/user-attachments/assets/75225159-e8c3-4238-ad19-7ee2a72e7182" />

19. [Удаление созданного сервисного аккаунта]<img width="1470" height="956" alt="13_service_account_deleted" src="https://github.com/user-attachments/assets/26b730b4-cd9f-42e0-a93a-cd997f5c5547" />
