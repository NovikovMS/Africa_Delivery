# Сравнение облачных регионов

**Версия:** 0.1  
**Дата:** 27.08.2026  
**Автор:** Solution Architect  
**Связано:** 
- [ADR-003](adr-003-data-residency.md), 
- [цены облака](../Task2/cloud-prices.md), 
- C-02, C-05, NFR-14

Цены on-demand, месяц = 730 ч. Цены AWS, Azure и Layer3 - из [cloud-prices.md](../Task2/cloud-prices.md).

---

## 1. Сравнение

| Провайдер, регион | Сервисы для V1 | Задержка | Цена | SLA | Вывод |
|---|---|---|---|---|---|
| **AWS af-south-1, Кейптаун** | • Все нужные: Fargate, RDS PostgreSQL, SQS, S3, CloudFront, KMS<br><br>• 3 зоны доступности<br><br>• Local Zones в Лагосе и Найроби | • Лагос 60-240 мс, зависит от маршрута<br><br>• Найроби ~66 мс | • ВМ 2 vCPU / 8 ГБ: \$0,108/ч<br><br>• PostgreSQL HA: \$0,328/ч<br><br>• Блок 1 V1: \$967/мес | • Compute в 2+ зонах 99,99%<br><br>• RDS Multi-AZ 99,95%<br><br>• S3 99,9% | **Основной регион** |
| **GCP africa-south1, Йоханнесбург** | • Cloud Run, Cloud SQL, Pub/Sub, GCS<br><br>• 3 зоны<br><br>• Регион с 2024 года, сервисов меньше | • Лагос ~72 мс<br><br>• Найроби ~55 мс | • PostgreSQL HA: ~\$0,28/ч<br><br>• Цена ВМ не проверена | • Cloud SQL Enterprise HA 99,95%<br><br>• Enterprise Plus 99,99% | **Резервный регион (DR)** |
| Azure South Africa North, Йоханнесбург | • Container Apps, PostgreSQL Flexible<br><br>• Зоны доступности | Как у GCP: Йоханнесбург | • ВМ: \$0,127/ч<br><br>• PostgreSQL HA: \$0,470/ч<br><br>• Контейнеры: \$0,108/ч | PostgreSQL с резервом в другой зоне 99,99% | Отклонён: БД в 1,4 раза, контейнеры в 2 раза дороже AWS. Лучше только SLA БД |
| **Layer3, Лагос + Абуджа** | • ВМ, объектное хранилище<br><br>• 2 зоны: Лагос и Абуджа<br><br>• DBaaS заявлен, цен и SLA нет | • Лагос < 5 мс, заявлено<br><br>• Найроби ~185 мс | • 2 vCPU / 4 ГБ / 128 ГБ: ~\$30/мес<br><br>• Оплата в найре | • На сайте: ВМ 99,9%<br><br>• В договоре: прогноз 98% в год, компенсация по договорённости | **Хранилище ПДн Нигерии.** SLA 99,9% закрепить в договоре |
| AWS Local Zone, Лагос | • EC2 T3, C5, R5, M5, EBS gp2, ALB, ECS, EKS<br><br>• Нет RDS, S3, NAT<br><br>• Одна зона | • Лагос < 10 мс, оценка<br><br>• До Кейптауна по сети AWS | • t3.medium: \$0,0624/ч ≈ \$46/мес<br><br>• m5.large: \$0,127/ч | Отдельного SLA нет. Одна ВМ 99,5% | Не выбран для ПДн: снапшоты EBS и ключи KMS остаются в Кейптауне. **Путь на случай «вся платформа в Нигерии»** |
| Lineserve, Лагос | ВМ | Лагос, как у Layer3 | 2 vCPU / 4 ГБ: ~\$32/мес | 99,9% | **Копия бэкапов ПДн Нигерии** у второго провайдера. Реплика, если Layer3 не подпишет SLA |
| AWS Local Zone, Найроби, или Angani | • AWS: EC2, EBS<br><br>• Angani: ВМ | Найроби < 10 мс | Angani 4 vCPU / 4 ГБ: ~\$31/мес | Angani: не опубликован | Резерв, если юрист потребует копию данных в Кении |
| AWS eu-central-1, Франкфурт | Все | Кейптаун - Франкфурт 157 мс | Не считали | Как af-south-1 | Отклонён: за пределами Африки. Для ПДн Нигерии нарушает локализацию, шифрование это не меняет |

## 2. Задержка между городами

| Откуда | Кейптаун (AWS) | Йоханнесбург (GCP, Azure) | Лагос (Layer3) |
|---|---|---|---|
| Лагос | 60-240 мс | ~72 мс | < 5 мс |
| Найроби | ~66 мс | ~55 мс | ~185 мс |
| Йоханнесбург | ~30 мс | < 5 мс | ~72 мс |

- Маршрут Лагос - Кейптаун нестабилен: прямой кабель WACS даёт около 60 мс, путь через Европу около 240 мс.
- CloudFront имеет точки присутствия в Лагосе и Найроби. Запрос входит в сеть AWS у пользователя и идёт до Кейптауна по ней.
- Цель «первый товар за 2 с» (А2) выдерживается при любом из значений: каталог отдаёт CDN ([ADR-002](../Task3/adr-002-caching-strategy.md)).
- Между ядром и хранилищем ПДн 60-240 мс на запрос. Поэтому ядро обращается к хранилищу только в фоне: передача адреса курьеру, отправка SMS. На пути оформления заказа обращений к хранилищу нет.

## 3. Вывод

| Роль | Выбор | Почему |
|---|---|---|
| Ядро | AWS af-south-1, Кейптаун | • Все managed-сервисы V1 есть, цены проверены<br><br>• Данные ЮАР остаются в ЮАР<br><br>• Local Zones в Лагосе и Найроби дают путь к локализации без смены провайдера |
| Резерв ядра | GCP africa-south1, Йоханнесбург | • У AWS в Африке один регион. Ближайший другой регион AWS - Испания, 131 мс, за пределами Африки<br><br>• Отказ AWS целиком не останавливает платформу<br><br>• ~\$80-90/мес, в TCO заложено \$80 |
| ПДн Нигерии | Layer3 Лагос, реплика в Абудже | • Нигерийский провайдер в двух городах, данные и бэкапы в стране<br><br>• Укладывается в \$75/мес из TCO |
| Копия бэкапов ПДн Нигерии | Lineserve, Лагос | • Потеря Layer3 целиком не означает потерю ПДн<br><br>• ~\$32/мес сверх TCO |

Блок 1 TCO V1: ~\$1 020/мес вместо \$967. Добавлены копия бэкапов на Lineserve, сервис ПДн Кении ~\$10 и разница по резерву до \$10. Выбор V1 остаётся в силе.

## Допущения

- Задержки - медианы ping между городами, а не до конкретного дата-центра. До 15.10 замеряем из сетей MTN и Airtel в Лагосе с ВМ синтетической проверки ([observability.md](../Task3/observability.md)).
- Резерв в GCP: Cloud SQL 1 vCPU без HA ~\$60, диск 100 ГБ ~\$20, Cloud Run в простое \$0, трафик репликации из AWS ~\$8. Цены Йоханнесбурга взяты как в us-central1 + 20%.

## Источники

- [AWS Local Zones: сервисы по зонам](https://aws.amazon.com/about-aws/global-infrastructure/localzones/features/), [список зон](https://docs.aws.amazon.com/local-zones/latest/ug/available-local-zones.html)
- [Цены EC2 в af-south-1-los-1](https://aws-pricing.com/af-south-1-los-1.html)
- [Amazon Compute SLA](https://aws.amazon.com/compute/sla/), [Cloud SQL SLA](https://cloud.google.com/sql/sla), [Azure PostgreSQL: надёжность и SLA](https://learn.microsoft.com/en-us/azure/reliability/reliability-database-postgresql)
- [Layer3: ВМ и SLA](https://www.layer3.cloud/virtual-servers), [Layer3: условия договора](https://www.layer3.cloud/terms-and-conditions)
- [WonderNetwork: Лагос - Йоханнесбург](https://wondernetwork.com/pings/Lagos/Johannesburg), [Лагос - Кейптаун](https://wondernetwork.com/pings/Lagos/Cape%20Town), [Найроби - Кейптаун](https://wondernetwork.com/pings/Nairobi/Cape%20Town), [Найроби - Йоханнесбург](https://wondernetwork.com/pings/Nairobi/Johannesburg), [Лагос - Найроби](https://wondernetwork.com/pings/Lagos/Nairobi)
- [ITWeb: Лагос - Кейптаун около 60 мс](https://www.itweb.co.za/article/taking-content-to-the-edge-slashing-latency-and-widening-coverage-in-africa/rW1xLv5n4rZ7Rk6m)
- [cloudping.co: задержки между регионами AWS](https://www.cloudping.co/)
