問題：在Vagrant裡面建立Laravel專案，進行`composer update`時因為內存不足而崩潰
解決方法：在使用Vagrant時，在`Vagrantfile`裡面取消以下段落的註釋：
```
config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end
```

問題：因Apache2關閉了htaccess override的設定而導致Laravel public裡面的`htaccess`檔案無法被正常利用，導致404的情形出現
解決方法：
1. 在Vagrant ssh裡面打開`/etc/apache2/apache2.conf`：
```
$ sudo nano /etc/apache2/apache2.conf
```
2. 把以下段落：
```
<directory var="" www="">
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</directory>
```
修改成：
```
<directory var="" www="">
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</directory>
```

情形：想要把Laravel網站URL的public去掉
解決方法：
1. 在Vagrant ssh裡面打開`/etc/apache2/apache2.conf`：
```
$ sudo nano /etc/apache2/sites-available/000-default.conf
```
2. 把以下段落：
```
DocumentRoot "/var/www/html"
```
修改成：
```
DocumentRoot "/var/www/html/your_project_folder/public"
```
3. 重新啟動Apache2
```
$ sudo service apache2 restart
```

此外之後再遇到什麼，會補上。
