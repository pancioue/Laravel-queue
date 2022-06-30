### driver:database
run 程序時，跑到queue job時，若不是設定`sync`不會馬上執行，
會先在database寫入要執行的job(queue)，記錄相關資訊，包含要傳的參數。

實際要執行的job必須要啟動`queue worker`
```sh
php artisan queue:work
```
下此指令會先執行Jobs，才會執行監聽queue任務(所以若修改job的code，需要重新下指令，重新加載到記憶體)  
* 網上有一說 `php artisan queue:listen`修改code後並不用重啟


### driver:redis
可利用redis-cli
```sh
keys *
```
redis jobs key=> `[project-name]_database_queues:default`

打印:
```sh
LRANGE [project-name]_database_queues:default 0 -1
```

查看後會發現跟 database 的紀錄是一樣的

參考:  
https://tech.ray247k.com/blog/202203-laravel-queue/


## php artisan queue:work
其原理是在後端run一段無窮迴圈的code，類似於下面這樣
```php
while (true) {
  echo 'hello queue';
  sleep(2);
}
```
實際的程式碼會去資料庫確認現在是否有待執行的queue job  
若沒有則先sleep一段時間，如此反覆執行

### pcntl_alarm
queue底層程式碼有用到`pcntl_alarm`
```php
// 註冊收到信號要執行的事
pcntl_signal(SIGALRM, function () {
  echo 'Hello Pcntl Alarm !' . PHP_EOL;
}, false);

while (true) {
  pcntl_alarm(2); // 過兩秒後發送信號
  pcntl_signal_dispatch(); // 捕捉信號
  sleep(2);
}
```

參考:  
https://blog.csdn.net/shj_php/article/details/89236401
