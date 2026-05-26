Error: Restarting /home/sites/accounts.comodo.com

Solution:

  SSL validation issue

1. check 
  [root@cam3-web-virt-sjc3#stretch@production5:~]# curl -s -o /dev/null --max-time 10 https://accounts.comodo.com/login   --resolve accounts.comodo.com:443:144.126.219.119

2.  [root@cam3-web-virt-sjc3#stretch@production5:~]# echo $?
60

3. [root@cam3-web-virt-sjc3#stretch@production5:~]# curl -vk https://accounts.comodo.com/login --resolve accounts.comodo.com:443:144.126.219.119

4. [root@cam3-web-virt-sjc3#stretch@production5:~]# openssl x509 -in /etc/nginx/ssl/accounts.comodo.com/accounts_comodo_com.crt -noout -dates

5. systemctl reload nginx

