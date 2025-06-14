# Настройка PAM и доступов на виртуальной машине

Запуск виртуальной машины
Для развертывания виртуальной машины используется команда:

vagrant up

После выполнения команды все конфигурации применяются автоматически: создаются пользователи, назначаются группы, настраиваются ограничения на вход в систему.
Конфигурация пользователей и групп
- Созданы пользователи: otusadm и otus.
- Для каждого пользователя установлен пароль.
- Сформирована группа admin, в которую включены пользователи root, vagrant и otusadm.
- Пользователю otusadm предоставлены административные привилегии.

Настройка PAM для ограничения входа в выходные
- Создан скрипт /usr/local/bin/login.sh, который определяет текущий день недели и регулирует доступ к системе.
- В субботу и воскресенье вход разрешен только для членов группы admin. В остальные дни доступ открыт для всех пользователей.
- Скрипт интегрирован в PAM через конфигурацию в файле /etc/pam.d/sshd с использованием модуля pam_exec.

Настройка доступа к Docker
- На виртуальной машине установлен Docker.
- Пользователь otus добавлен в группу docker, что позволяет ему работать с Docker без необходимости использования sudo.
- Пользователю otus предоставлено право перезапуска службы Docker через systemctl без ввода пароля.

Проверка конфигурации
- Для тестирования ограничений входа дата на виртуальной машине установлена на 16 марта 2025 года (выходной день).
- Проверено подключение пользователей:
  - Пользователь otus не смог войти, так как не входит в группу admin.
  - Пользователь otusadm успешно подключился, так как является членом группы admin.

В рамках задания были выполнены следующие шаги:
- Создан файл Vagrantfile, который автоматически разворачивает виртуальную машину и применяет все настройки: создание пользователей, групп и ограничений на вход.
- Реализована система контроля доступа через PAM с использованием скрипта, ограничивающего вход в выходные дни для обычных пользователей.
- Настроены права для работы с Docker: пользователь otus получил доступ к управлению Docker и возможность перезапуска сервиса без ввода пароля.