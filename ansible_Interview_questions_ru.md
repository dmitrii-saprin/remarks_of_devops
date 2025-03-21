## Вопросы по Ansible для Senior DevOps Engineer

### Тема: Основы и управление задачами

1. **Что такое handler в Ansible и чем он отличается от обычного task?**  
   Приведите пример использования handler.

2. **Как Ansible определяет, нужно ли запускать handler?**  
   Опишите механизм уведомлений (`notify`) и порядок выполнения handler-ов.

3. **Что делать, если на удалённом хосте отсутствует Python?**  
   Опишите несколько подходов, как установить Python на удалённый хост с помощью Ansible.

4. **Перечислите уровни переменных в Ansible по приоритету (от наивысшего к низшему).**  
   Какой уровень переменных будет использован при конфликте и почему?

### Тема: Инвентарь и модули

5. **Как использовать динамический инвентарь (dynamic inventory) в Ansible?**  
   В каких случаях он предпочтительнее статического инвентаря?

6. **Что такое модули `raw` и `command` в Ansible и чем они отличаются?**  
   Когда оправдано использование каждого из них?

7. **Как реализуется параллельное выполнение задач в Ansible?**  
   Что такое параметр `forks`, и как он влияет на производительность?

### Тема: Тестирование и организация кода

8. **Как тестировать и проверять корректность плейбуков перед применением на production-средах?**  
   Какие инструменты вы использовали для тестирования и проверки плейбуков?

9. **Как использовать роли (roles) в Ansible и каковы best practices по их организации?**  
   В чём преимущество использования ролей?

### Тема: Сравнение и выбор инструментов

10. **В каких случаях вы предпочтёте Ansible другим инструментам управления конфигурацией (например, Chef, Puppet, SaltStack)?**  
    Перечислите основные преимущества и недостатки Ansible на ваш взгляд.

### Тема: Безопасность

11. **Как защитить чувствительные данные в Ansible без использования Ansible Vault?**  
    Приведите пример интеграции Ansible с внешними хранилищами секретов (например, HashiCorp Vault).

12. **Как настроить безопасное выполнение Ansible через SSH для сотен хостов?**

### Тема: Масштабирование и оптимизация

13. **Как вы оптимизируете плейбук для работы с тысячами хостов?**  
    Какие параметры и подходы вы будете использовать?

14. **Что такое strategy в Ansible (например, `linear`, `free`)?**  
    Как выбор стратегии влияет на выполнение задач?

15. **Как работает идемпотентность в Ansible?**  
    Как обеспечить идемпотентность при написании кастомных модулей?

### Тема: Интеграция и переменные

16. **Как интегрировать Ansible с CI/CD-пайплайном (например, GitLab CI)?**  
    Приведите пример рабочего workflow.

17. **Как использовать Ansible для управления Kubernetes-кластером?**  
    Какие модули вы использовали для этого?

18. **Как уровни вложенности переменных в Ansible определяют область видимости и приоритет?**  
    Приведите пример конфликта и его разрешения.

### Тема: Ситуационные вопросы

19. **Вам необходимо автоматизировать rolling update приложения на 200 серверов с минимальным downtime.**  
    Как это реализовать с помощью Ansible?

20. **Плейбук выполняется слишком медленно на 500 хостах.**  
    Как вы будете искать bottleneck и оптимизировать процесс выполнения?