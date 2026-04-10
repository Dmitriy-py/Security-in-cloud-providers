# Домашнее задание к занятию «Безопасность в облачных провайдерах»

## ` Дмитрий Климов `

---

## Задание 1. Yandex Cloud

1. С помощью ключа в KMS необходимо зашифровать содержимое бакета:
   * создать ключ в KMS;
   * с помощью ключа зашифровать содержимое бакета, созданного ранее.
2. (Выполняется не в Terraform)* Создать статический сайт в Object Storage c собственным публичным адресом и сделать          доступным по HTTPS:
   * создать сертификат;
   * создать статическую страницу в Object Storage и применить сертификат HTTPS;
   * в качестве результата предоставить скриншот на страницу с сертификатом в заголовке (замочек).

---

## Ответ:

---

## Конфигурация Terraform

### В ходе выполнения были созданы следующие файлы конфигурации (основные фрагменты):

### 1. Создание ключа KMS и бакета с шифрованием (s3.tf + kms.tf):

```Hcl
resource "yandex_kms_symmetric_key" "kms-key" {
  name              = "bucket-kms-key"
  folder_id         = var.yc_folder_id
  description       = "Ключ для шифрования содержимого бакета"
  default_algorithm = "AES_256"
  rotation_period   = "8760h"
}

resource "yandex_storage_bucket" "img_bucket" {
  access_key = yandex_iam_service_account_static_access_key.sa_key.access_key
  secret_key = yandex_iam_service_account_static_access_key.sa_key.secret_key
  bucket     = "nnetology-bucket-new-unique-name-12345"
  acl        = "public-read"

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        kms_master_key_id = yandex_kms_symmetric_key.kms-key.id
        sse_algorithm     = "aws:kms"
      }
    }
  }

  website {
    index_document = "index.html"
    error_document = "error.html"
  }
}

# Загрузка объекта (картинки) в бакет
resource "yandex_storage_object" "picture" {
  access_key = yandex_iam_service_account_static_access_key.sa_key.access_key
  secret_key = yandex_iam_service_account_static_access_key.sa_key.secret_key
  bucket     = yandex_storage_bucket.img_bucket.id
  key        = "image.png"
  source     = "image.png"
  acl        = "public-read"
}
```

---

### 2. Настройка прав доступа (IAM):

```Hcl
resource "yandex_iam_service_account" "sa" {
  name = "ig-sa"
}

resource "yandex_resourcemanager_folder_iam_member" "sa_editor" {
  folder_id = var.yc_folder_id
  role      = "editor"
  member    = "serviceAccount:${yandex_iam_service_account.sa.id}"
}

resource "yandex_resourcemanager_folder_iam_member" "sa_storage" {
  folder_id = var.yc_folder_id
  role      = "storage.admin"
  member    = "serviceAccount:${yandex_iam_service_account.sa.id}"
}

resource "yandex_iam_service_account_static_access_key" "sa_key" {
  service_account_id = yandex_iam_service_account.sa.id
}
```

### 3. Настройка статического сайта и HTTPS

Для реализации доступа по HTTPS был использован механизм статического хостинга в Yandex Object Storage.

### Этапы выполнения:

1. Создан файл index.html с приветственным текстом и ссылкой на изображение.
2. Файл загружен в бакет, для него настроены публичные права на чтение (ACL Public Read).
3. В настройках бакета включен режим «Хостинг».
4. Доступ по HTTPS обеспечен использованием стандартного домена Yandex Cloud (.website.yandexcloud.net), который           автоматически защищен SSL-сертификатом провайдера.

### Результаты

### 1. Шифрование KMS

<img width="1920" height="1080" alt="Снимок экрана (3298)" src="https://github.com/user-attachments/assets/6122a0ea-e197-4559-9e04-4fbdc8649abe" />

### 2. Работающий сайт с HTTPS

<img width="1920" height="1080" alt="Снимок экрана (3294)" src="https://github.com/user-attachments/assets/8a0c8dbc-eaaa-4af7-a73a-ca8142676b2f" />

<img width="1920" height="1080" alt="Снимок экрана (3295)" src="https://github.com/user-attachments/assets/8e318719-9a61-4888-9384-95fa9284c72e" />

<img width="1920" height="1080" alt="Снимок экрана (3297)" src="https://github.com/user-attachments/assets/c92b9dff-46c5-4a68-b71b-836e61bbf92f" />
















