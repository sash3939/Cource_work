**Курсовой проект на профессии «DevOps-инженер с нуля»**

*Задание*
----
Ключевая задача — разработать отказоустойчивую инфраструктуру для сайта, включающую мониторинг, сбор логов и резервное копирование основных данных. Инфраструктура должна размещаться в Yandex Cloud.

**Решение**

*Ссылки в браузере на развернутые веб-сервисы:*
- Страничка с опубликованным на Nginx через внешний адрес и балансировщик от Yandex Cloud [WEB-сайт](http://158.160.137.9/)
- Веб интерфейс с логами nginx-сервера в [Kibana](http://158.160.96.134:5601/app/discover#/?_g=(filters:!(),refreshInterval:(pause:!t,value:0),time:(from:now-15m,to:now))&_a=(columns:!(),filters:!(),index:dba56bd0-3611-11ef-8806-290a3ef9f9f8,interval:auto,query:(language:kuery,query:''),sort:!(!('@timestamp',desc))))

- Веб инетерфейс Графаны с логами со всех серверов инфраструктуры: [Grafana](http://158.160.62.41:3000/d/rYdddlPWk/node-exporter-full?orgId=1&refresh=1m&var-datasource=default&var-job=node&var-node=192.168.30.31:9100&var-diskdevices=%5Ba-z%5D%2B%7Cnvme%5B0-9%5D%2Bn%5B0-9%5D%2B%7Cmmcblk%5B0-9%5D%2B)

*План выолнения:*  


- Настраиваем Terraform и пишем плейбуки для всей инфраструктуры.
- Пишем Ansible плейбуки и роли для централизованного управления конфигурации
- Заходим на веб ресурсы и проверяем что все правильно отработало, наш сайт доступен и порталы с логами и метрикиами активны.

  *Ход выполнения:*

  - Запускаем terraform apply и раскатываем все манифесты с описанием развертки.
  - Через функционал Terraform постредсвом манифеста output.tf выводим ip адреса созданных истансов и забираем их через скрипт output.sh в host.ini фаидл для работы Ansible.
  
   ![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/1.jpg)

   ![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/2.jpg)
  
- Далее запускаем плейбук настройки бастион-хоста как джмап-сервера для доступа внутрь нашей закрытйо из вне инфраструктуры и настройкой через него как через джамп всех хостов в нашей сети

  ![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/3.jpg)

  ![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/4.jpg)


  - Далее запускаем скрипт где у нас написан порядок автоматического выполнения наших Ansible плейбуков по очереди

   ![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/6.jpg)

   ![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/5.jpg)


  - Теперь видим как у наш пошел пошел плейбук с ролью по настройки зеркальных Nginx-серверов с готовым сайтом-страничкой.

  ![alt text](https://github.com/mezhibo/Course_work/blob/9cdd71da7fd6afd3946ff697548cf6878d2fc820/IMG/install_nginx.jpg)

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


  



    
