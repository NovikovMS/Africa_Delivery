### Артефакт 5. Письмо от CTO

**На русском языке**
<details>
От: Rajesh Menon r.menon@africadelivery.io
Кому: Solution Architect 
Копия: К. Орлов; М. Лефевр
Дата: 11 августа 2026, 02:14
Тема: Re: архитектура — мысли после KubeCon (длинно, извини)
Вложения: CN чек-лист
Добро пожаловать. Пишу из аэропорта, только что с KubeCon + CloudNativeCon, голова кипит. Хочу, чтобы мы синхронизировались до того, как ты начнёшь рисовать квадратики.
Целевой масштаб. Планируем 
10 миллионов пользователей с первого дня. Не когда-нибудь, а с первого дня. Рекламный охват огромный, и я не собираюсь быть тем CTO, у которого платформа легла в ночь запуска. Проектируем под 10M, потом отмасштабируемся вниз, если ошиблись. Компании умирают под трафиком потому что откладывали на потом масштабирование.
Cloud native по-настоящему. Хочу сертификацию CNCF и хочу зафиксировать это письменно (посмотри CN чек лист утащил с конференции). Конкретно:
Kubernetes, мульти-кластер, минимум три кластера (два региона плюс один on-prem под нигерийские данные)
Service mesh — Istio. mTLS повсюду, управление трафиком, канареечные релизы
Kafka как хребет системы. Всё событийно, никаких синхронных вызовов между сервисами
40–50 сервисов в целевом состоянии. Набросал карту сервисов в самолёте, скину
GitOps на ArgoCD, всё декларативно, никаких ручных изменений никогда
OpenTelemetry, Prometheus, Grafana, Jaeger, Loki — весь стек, с первого дня
Мульти-облако с самого начала. Никакого vendor lock-in. Ничему, что мы запускаем, не должно быть важно, где оно работает — AWS, GCP, Azure или наше железо. Совет директоров прямо сказал, что вариант on-prem должен оставаться на столе.
Именно поэтому я не хочу managed-сервисы. Managed Postgres, managed Kafka, managed что угодно — это тот же lock-in, только с ежемесячным счётом. Поднимаем своё в Kubernetes. Операторы сейчас зрелые, я видел три доклада ровно про это.
База данных. Oracle для транзакционного ядра. Я знаю, что об этом думают в комнате, но я шесть лет эксплуатировал Oracle под нагрузкой на логистической платформе и точно знаю, как он себя ведёт. Postgres для периферийных сервисов, если настаиваешь. Go для всех сервисов — здесь я не торгуюсь: полиглотный стек при такой численности команды заканчивается тем, что в три часа ночи чинить некому.
Строим, а не покупаем. Мне всё время говорят, что надо интегрировать Sendbox и Sendy. Я уже строил логистический движок — маршрутизация, зоны, назначение курьеров, подтверждение доставки. Шесть месяцев, четыре инженера. Мы владели бы самой защищаемой частью бизнеса, а не арендовали её. Тот же аргумент по платёжной оркестрации: агрегаторы вечно берут процент с каждой транзакции, а мы можем говорить с M-Pesa и Flutterwave напрямую.
Доступность. Минимум 99,99%, active-active между регионами, автоматическое переключение, никаких единых точек отказа. Мультирегион с запуска. RPO — ноль.
По срокам — да, я знаю, что восемь недель. Я делал агрессивные запуски. Если наймём трёх-четырёх сильных Go-разработчиков в ближайшие две недели, мы в порядке; и, честно говоря, инструментарий за последние два года ушёл так далеко, что бо́льшая часть этого — выходные за написанием YAML.
Ещё одно. Инвестор спрашивал меня про блокчейн для лояльности. Я сказал, что мы это оценим. Пожалуйста, обеспечь, чтобы это где-нибудь фигурировало в документе о видении архитектуры, чтобы потом не прилетело обратно.
Пришли мне черновик ADR до встречи в Найроби. Хочу, чтобы мы зашли туда согласованными.
Раджеш 
Отправлено с телефона
</details>

---

**На английском языке**
<details>
From: Rajesh Menon r.menon@africadelivery.io
To: Solution Architect
Cc: K. Orlov; M. Lefèvre
Date: 11 August 2026, 02:14
Subject: Re: architecture — thoughts after KubeCon (long, sorry)
Attachment: CN check list
Welcome aboard. I'm writing this from the airport, just came out of KubeCon + CloudNativeCon and my head is full. I want us aligned before you start drawing boxes.
The target. We plan for 10 million users from day one. Not "eventually" — day one. The advertising reach is enormous and I refuse to be the CTO whose platform fell over on launch night. Design for 10M concurrent-capable, we scale down if we're wrong. Scaling up later is what kills companies.
Cloud native, properly. I want us CNCF-certified and I want it on the record (see attached CN check list i grab from the conference). Concretely:
Kubernetes, multi-cluster, at least three clusters (two regions plus one on-prem for the Nigerian data)
Service mesh — Istio. mTLS everywhere, traffic shifting, canary releases
Kafka as the backbone. Everything event-driven, no synchronous calls between services
40–50 services at target state. I sketched a service map on the plane, will share
GitOps with ArgoCD, everything declarative, no manual changes ever
OpenTelemetry, Prometheus, Grafana, Jaeger, Loki — full stack, from day one
Multi-cloud from the start. No vendor lock-in. Nothing we run should care whether it sits on AWS, GCP, Azure or our own metal. The board has been explicit that on-prem must remain an option.
Which is why I don't want managed services. Managed Postgres, managed Kafka, managed anything — that is lock-in with a monthly invoice attached. We run our own on Kubernetes. Operators are mature now, I saw three talks on exactly this.Database. Oracle for the transactional core. I know what the room thinks, but I have run Oracle at scale on a logistics platform for six years and I know exactly what it does under load. Postgres for the peripheral services if you insist. 
Golang for all services — I'm not negotiating on language, a polyglot stack with this team size is how you end up with nobody able to fix anything at 3am.
Build, don't buy. I keep hearing "integrate Sendbox and Sendy". I've built a logistics engine before — routing, zones, courier assignment, proof of delivery. Six months, four engineers. We'd own the most defensible part of the business instead of renting it. Same argument for payment orchestration: the aggregators take a cut of every transaction forever, and we can talk to M-Pesa and Flutterwave directly.
Availability. 99.99% minimum, active-active across regions, automatic failover, no single point of failure anywhere. Multi-region from launch. RPO zero.
On timing — yes, I know it's eight weeks. I've done aggressive launches. If we hire three or four strong Go engineers in the next two weeks we're fine, and honestly the tooling has moved so far in the last two years that a lot of this is a weekend of YAML.
One more thing. The investor asked me about blockchain for loyalty. I told him we'd evaluate it. Please make sure it appears somewhere in the vision document so it doesn't come back at us later.
Send me your ADR draft before the Nairobi meeting. I'd like us to walk in agreeing.
Rajesh 
Sent from my phone
</details>