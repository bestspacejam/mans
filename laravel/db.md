# Типовые задачи при работе с базой данных

Лог выполняемых запросов
```php
DB::enableQueryLog();
DB::table('users')->where('name', 'John')->first();
DB::getQueryLog();
```
