## Создание нового пользователя

- Configure – используется для входа в режим полной конфигурации.
- Hostname – используется для изменения имени хоста.
- Commit – используется для применения изменений конфигурации устройства.
- Confirm – используется для сохранения изменений конфигурации устройства.

```
Router# configure
Router(config)# username user
Router(config-user)# password user 
Router(config-user)# privilege 1
Router(config-user)# exit
Router(config)# exit
Router# commit
Router# confirm
```

## Сброс устройства к заводским настройкам
```
Router# copy system:default-config system:candidate-config
```

## 
