# Сервисы по capability

**Версия:** 0.1  
**Дата:** 31.08.2026  
**Автор:** Solution Architect  
**Связано:** 
- [Capability Map](capability-map.md), 
- [команда](team-structure.md), 
- [C4 Container](../Task4/c4-container.puml)

Сервис - модуль монолита ядра (ADR-001) или отдельный контейнер из C4 Container.  
Владелец - роль из [оргструктуры](team-structure.md).

| Capability | Сервис | Владелец | Внешняя зависимость |
|---|---|---|---|
| 1.1 Карточки и категории | Ядро: `catalog` | Backend 2 | - |
| 1.2 Поиск | Ядро: `catalog`, PostgreSQL FTS | Backend 2 | - |
| 1.3 Фото и медиа | S3, CloudFront | DevOps | AWS |
| 1.4 Модерация | Бэк-офис, ядро: `catalog` | Веб | - |
| 2.1 Заказ | Ядро: `orders` | Техлид | - |
| 2.2 Резервирование остатков | Ядро: `orders` | Техлид | - |
| 2.3 Промо и флеш-распродажи | Ядро: `catalog` | Backend 2 | - |
| 2.4 Заказ без интернета | Ядро: `ussd` | Backend 3 | USSD-шлюзы Кении и Нигерии |
| 3.1 Приём оплаты | Ядро: `payments` | Техлид | • Safaricom Daraja<br><br>• Flutterwave<br><br>• Paystack |
| 3.2 Маршрутизация | Ядро: `payments`, адаптеры | Техлид | Те же |
| 3.3 Выплаты продавцам | Ядро: `payouts` | Техлид | • M-Pesa B2C<br><br>• Flutterwave, Paystack |
| 3.4 Сверка | Ядро: `payments`, ежедневная задача | Техлид | Выписки провайдеров |
| 4.1 Отправления | Ядро: `delivery`. Сервис ПДн NG для адреса | Backend 3 | • Sendbox<br><br>• Sendy |
| 4.2 Отслеживание | Ядро: `delivery`, вебхуки | Backend 3 | Те же |
| 4.3 ПВЗ и наличные | Ядро: `delivery`, 6 мес | Backend 3 | Сеть ПВЗ партнёра |
| 4.4 Возврат товара | Бэк-офис, ядро: `orders` | Веб | Те же |
| 5.1 Онбординг и KYC | Бэк-офис, ядро: `sellers` | Backend 2 | KYC провайдера при открытии субсчёта |
| 5.2 Товары и остатки | Кабинет продавца, ядро: `sellers` | Веб | WhatsApp BSP |
| 5.3 Аналитика продавца | Кабинет, реплика БД, 12 мес | Backend 2 | - |
| 6.1 Аккаунт | Сервис ПДн NG, сервис ПДн KE и ZA | Backend 3 | Layer3, Lineserve |
| 6.2 Уведомления | Ядро: `notifications`. Сервис ПДн NG для номеров Нигерии | Backend 3 | • SMS-агрегаторы<br><br>• WhatsApp BSP<br><br>• Firebase |
| 6.3 Маркетинг с согласием | Сервисы ПДн: журнал согласий | Backend 3 | Те же |
| 6.4 Поддержка и споры | Бэк-офис | Веб | - |
| 7.1 ПДн по юрисдикциям | Сервисы ПДн, OpenBao | Backend 3 | Layer3, Lineserve |
| 7.2 Антифрод | Ядро: `orders`, лимиты | Техлид | Инструменты провайдеров |
| 7.3 Аналитика и дашборд | Реплика БД, Grafana | DevOps | Grafana Cloud |
| 8.1 Клиентские каналы | Android, мобильный веб | Android, Веб | Google Play |
| 8.2 Хостинг, БД, очередь | Fargate, RDS, SQS, Cloud Run, Cloud SQL | DevOps | AWS, GCP |
| 8.3 Наблюдаемость | OpenTelemetry, Grafana | DevOps | Grafana Cloud |
| 8.4 CI/CD | GitHub Actions | DevOps | GitHub |

## Нагрузка на владельцев

- Техлид: деньги целиком. Ошибка здесь стоит денег, поэтому один владелец.
- Backend 3: больше всего внешних зависимостей. Адаптеры с единым контрактом (ADR-001), каждый партнёр за Circuit Breaker.
- Модули `ussd` и `payouts` новые относительно C4 Container: выделены внутри ядра, новых контейнеров нет.
