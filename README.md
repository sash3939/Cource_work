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
  
   ![After terraform apply](https://github.com/sash3939/Devops_Cource_work/assets/156709540/5b3cc46f-a99e-4554-8c19-9f7bbed38d5c)

   ![output.sh](https://github.com/sash3939/Devops_Cource_work/assets/156709540/1a8966e0-cd47-436f-b0f2-1da86489f90b)

  
- Далее запускаем плейбук настройки бастион-хоста как джмап-сервера для доступа внутрь нашей закрытйо из вне инфраструктуры и настройкой через него как через джамп всех хостов в нашей сети

  ![bastion](https://github.com/sash3939/Devops_Cource_work/assets/156709540/67300204-cad6-4efe-b9cf-eea7acd627f4)


  - Далее запускаем скрипт где у нас написан порядок автоматического выполнения наших Ansible плейбуков по очереди

  ![run deploy sh playbooks](https://github.com/sash3939/Devops_Cource_work/assets/156709540/d9f760f1-1ca4-4d54-a744-333c7b08e9e6)


  ![process](https://github.com/sash3939/Devops_Cource_work/assets/156709540/4a7a33ae-2505-4839-acd5-e5c5de829511)



  - Теперь видим как у наш пошел пошел плейбук с ролью по настройки зеркальных Nginx-серверов с готовым сайтом-страничкой.

  ![web](https://github.com/sash3939/Devops_Cource_work/assets/156709540/a62ab641-fcab-4352-834c-b22ef22a6814)

  - После него по очереди по списку идет установка на хосты Node_exportera для отправки наших метрик в Prometheus
 
    ![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/install_node_exporter.jpg)

  - Далее идет деплой Графаны для получения и визуализации метрик с наших серверов, и установка дашборда

     ![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/install%20grafana.jpg)

  - Следующим шагом идет установка и настройка ЕКЛ-стека, начнем сервера для централизованного сбора и хранения логов ElsticSearch

 ![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/install%20elastic.jpg)

 - Идем дальше. Разворачиваем постпенно весь ЕЛК-стек. Депломи Кибану под визуализацию данных, которые бдуем получать через filebeat агента.

   ![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/install%20elastic.jpg)

- Теперь ставим и сам filebeat для отправки логов в Elastic и меням права на паки, чтобы не было проблем с доступ к файлам логов.

![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/Install%20filebeat.jpg)

- И в конце отрабатывает последний скрипт по деплою Prometheus сервера, что будет забирать данные с Node exporter и отправлять для визуализации в Kibana.

![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/install%20prometheus.jpg)

- После отработки всех плейбуков и террформ манифестов идем в консоль Yndex Cloud и проверяем что у нас все верно создалось.

![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/all_servers.jpg)

- Видим что все создалось как надо, сервера хранения логов и хранения метрик изолированы вообще от внешнего мира через NAT интерфейс.
- 2 nginx сервера в разных подсетях и в рахных зонах дсотупности для более высокой отказустойчивости кластера.

- Смотрим все и впорядке с фаерволом на стороне Yndex Cloud, а именно Security Group

![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/security_group.jpg)  


- Смотрим подсети
![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/network_sub_net.jpg)

- Роутер
![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/router.jpg)

- Балансировщик с внешним адресом и распределением нагрузки на веб-сервера Nginx
  ![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/alb.jpg)

- Ну и не забываем о самом главном-псмотреть создаются ли у нас реервные копии наших дисков.

![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/shedule%20shanshot.jpg)

- Ну и теперь самый волнующий момент, все ли работает?
- Переходим по ip адресу нашего балансировщиа и, ~О ЧУДО!!!~  видим что он переносит нас на страницу одного из Веб-серверов Nginx согласно правилу Round-Robin.

![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/web_nginx.jpg)

- Теперь глянем наши логи с nginx
- Супер! Логи доступа к Nginx тоже видим.

![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/web_kinaba.jpg)

- И теперь глянем сбор метрик нагрузки на наши сервера.
- Заходим, видим настройеный дашборд

![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/web_grafana.jpg)

- И проверим все ли серверы шлют данные в Prometheus:

![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/kibana_hosts.jpg)

*Ну вот собсвенно и все*
*В пару нажатий на клавиатуре создана отказоутойчивая инраструктура и системой мониторинга, логирования. балансировкой входящего траффика и автоматиеческого резервирования дисков*
~не считая сотни потраченных часов и нервов~


  



    
