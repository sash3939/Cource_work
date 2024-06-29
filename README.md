**Курсовой проект на профессии «DevOps-инженер с нуля»**

*Задание*
----
Ключевая задача — разработать отказоустойчивую инфраструктуру для сайта, включающую мониторинг, сбор логов и резервное копирование основных данных. Инфраструктура должна размещаться в Yandex Cloud.

**Решение**

*Ссылки в браузере на развернутые веб-сервисы:*
- Страничка с опубликованным на Nginx через внешний адрес и балансировщик от Yandex Cloud [WEB-сайт](http://158.160.137.9/)
- Веб интерфейс с логами nginx-сервера в [Kibana](http://158.160.96.134:5601/app/discover#/?_g=(filters:!(),refreshInterval:(pause:!t,value:0),time:(from:now-15m,to:now))&_a=(columns:!(),filters:!(),index:dba56bd0-3611-11ef-8806-290a3ef9f9f8,interval:auto,query:(language:kuery,query:''),sort:!(!('@timestamp',desc))))

- Веб интерфейс Графаны с логами со всех серверов инфраструктуры: [Grafana](http://158.160.47.100:3000/explore?orgId=1&left=%7B%22datasource%22:%22PBFA97CFB590B2093%22,%22queries%22:%5B%7B%22refId%22:%22A%22,%22datasource%22:%7B%22type%22:%22prometheus%22,%22uid%22:%22PBFA97CFB590B2093%22%7D,%22editorMode%22:%22builder%22,%22expr%22:%22node_exporter_build_info%7Bgoversion%3D%5C%22go1.18.1%5C%22%7D%22,%22legendFormat%22:%22__auto%22,%22range%22:true,%22instant%22:true%7D%5D,%22range%22:%7B%22from%22:%22now-5m%22,%22to%22:%22now%22%7D%7D)

*План выолнения:*  


- Настраиваем Terraform и пишем плейбуки для всей инфраструктуры.
- Пишем Ansible плейбуки и роли для централизованного управления конфигурации
- Заходим на веб ресурсы и проверяем что все правильно отработало, наш сайт доступен и порталы с логами и метрикиами активны.

  *Ход выполнения:*

  - Запускаем terraform apply и раскатываем все манифесты с описанием развертки.
  - Через функционал Terraform постредсвом манифеста output.tf выводим ip адреса созданных истансов и забираем их через скрипт output.sh в host.ini файл для работы Ansible.
  
   ![After terraform apply](https://github.com/sash3939/Cource_work/assets/156709540/0545bbf8-a749-4ac2-985d-74abee5fa630)

   ![output.sh](https://github.com/sash3939/Cource_work/assets/156709540/9a16ffcd-eda3-4242-aaa1-53d86acef97b)


  
- Далее запускаем плейбук настройки бастион-хоста как джмап-сервера для доступа внутрь нашей закрытйо из вне инфраструктуры и настройкой через него как через джамп всех хостов в нашей сети

  ![bastion](https://github.com/sash3939/Cource_work/assets/156709540/70684e53-56c5-4e9e-bb8a-1a892a10f87b)



 - Далее запускаем скрипт где у нас написан порядок автоматического выполнения наших Ansible плейбуков по очереди

  ![run playbooks](https://github.com/sash3939/Cource_work/assets/156709540/b76ff9bf-aea2-48c1-9a32-66ec08815a81)

  ![process](https://github.com/sash3939/Cource_work/assets/156709540/75331a68-371b-45db-b97a-8821a6dc4c96)

-----

  - Теперь видим как у наш пошел пошел плейбук с ролью по настройки зеркальных Nginx-серверов с готовым сайтом-страничкой.

  ![web](https://github.com/sash3939/Cource_work/assets/156709540/57fb1ec8-a5ff-458f-a9a4-db36908a2b5a)


  - После него по очереди по списку идет установка на хосты Node_exportera для отправки наших метрик в Prometheus

   ![node_exporter](https://github.com/sash3939/Cource_work/assets/156709540/c120ce9e-e0b8-498c-b45e-8bef854270bd)

  - Prometheus после повторной поптыки поставился отдельно

   ![prometheus node_exporter](https://github.com/sash3939/Cource_work/assets/156709540/09c35d25-4883-4419-922d-2bcee7ff3fa5)

   ![web grafana](https://github.com/sash3939/Cource_work/assets/156709540/0383b749-693d-4aad-b3a9-0e0f6caf3fb0)


  - Далее идет деплой Графаны для получения и визуализации метрик с наших серверов, и установка дашборда

   ![grafana deploy](https://github.com/sash3939/Cource_work/assets/156709540/6c949455-5ae0-4ad4-bdb3-d1e29e36a7f6)


  - Далее ставим ELK-stack, начнем сервера для централизованного сбора и хранения логов ElsticSearch

   ![ELK deploy](https://github.com/sash3939/Cource_work/assets/156709540/a925a8f4-3dea-43fd-a55e-f685fd7a7de4)


 - Идем дальше. Разворачиваем постепенно весь ELK-stack. Депломи Kibana под визуализацию данных, которые бдуем получать через filebeat агента.

![Kibana deploy](https://github.com/sash3939/Cource_work/assets/156709540/504dd92c-3e43-47a0-98a8-380c6f2a15e4)

  
- Теперь ставим и сам filebeat для отправки логов в Elastic и меням права на паки, чтобы не было проблем с доступ к файлам логов.

![filebeat deploy](https://github.com/sash3939/Cource_work/assets/156709540/fd89f50b-5da7-48e0-b210-3a879ba72e9f)
![done filebeat](https://github.com/sash3939/Cource_work/assets/156709540/124bbd0b-7b93-48b0-b3c4-867123171af9)
![filebeat deploy ok](https://github.com/sash3939/Cource_work/assets/156709540/4250a0d0-b516-4fd8-818e-72affd1830b3)


- И в конце отрабатывает последний скрипт по деплою Prometheus сервера, что будет забирать данные с Node exporter и отправлять для визуализации в Kibana.

![prometheus deploy](https://github.com/sash3939/Cource_work/assets/156709540/13be12bd-5b72-4c50-a3e1-5377ca8f1bb7)

----

- После отработки всех плейбуков и терраформ манифестов идем в консоль Yandex Cloud и проверяем, что у нас все верно создалось.

![YC](https://github.com/sash3939/Cource_work/assets/156709540/fe2a22bf-9148-4d4d-8811-2ce4dad70055)

- Видим что все создалось как надо, сервера хранения логов и хранения метрик изолированы от внешнего мира через NAT интерфейс.

- 2 nginx сервера в разных подсетях и в рахных зонах дсотупности для более высокой отказустойчивости кластера.

-------

- Смотрим все и впорядке с фаерволом на стороне Yandex Cloud, а именно Security Group

![Sec groups](https://github.com/sash3939/Cource_work/assets/156709540/8673fb5d-f298-420d-a44b-3785a5316310)

  
- Смотрим подсети

![Subnets](https://github.com/sash3939/Cource_work/assets/156709540/685ca5e1-e8ac-419f-8cc0-cc03212fa43d)


- Роутер
 ![Router](https://github.com/sash3939/Cource_work/assets/156709540/58207dc9-a96f-4138-a1e6-5439add3141e)


- Балансировщик с внешним адресом и распределением нагрузки на веб-сервера Nginx

 ![Balancer](https://github.com/sash3939/Cource_work/assets/156709540/b38826e6-ab8f-4916-a115-cbd407066cef)


- И резервные копии наших дисков.

![Snapshots](https://github.com/sash3939/Cource_work/assets/156709540/70399f6e-67c2-47c7-bf89-ac752a6b1bdc)

----

- Переходим по ip адресу нашего балансировщиа и видим что он переносит нас на страницу одного из Веб-серверов Nginx согласно правилу Round-Robin.

![web page cource work](https://github.com/sash3939/Devops_Cource_work/assets/156709540/615c918e-b631-48d7-8765-c4eadb32c54d)

- Теперь глянем наши логи с nginx
- Супер! Логи доступа к Nginx тоже видим.

![logs nginx](https://github.com/sash3939/Devops_Cource_work/assets/156709540/7ac9ca8b-4fbc-4d1e-9b70-ead76186a907)


- И теперь глянем сбор метрик нагрузки на наши сервера.
- Заходим, видим настроенный дашборд

![dashboard](https://github.com/sash3939/Devops_Cource_work/assets/156709540/fc6ede06-dcb4-4f6e-80ac-cc44776cb873)

- И проверим все ли серверы шлют данные в Prometheus:

![sended](https://github.com/sash3939/Devops_Cource_work/assets/156709540/ed9586d0-66ce-47ba-89de-5b7fa1e46854)


*В пару нажатий на клавиатуре создана отказоутойчивая инраструктура и системой мониторинга, логирования. балансировкой входящего траффика и автоматиеческого резервирования дисков*


  



    
