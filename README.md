# noip2
configuracion de NO-IP en ubuntu

Ha sido facil la configuracion de NO-IP en Ubuntu. Sino que de un tiempo para aca, he tenido problemas
con la actualizacion automatica de la direccion IP dinamica por parte del agente (DUC).

Por eso dejo un log de lo que he hecho hasta ahora, que creo que empezó a funcionar bien con el ultimo
cambio que le hice al archivo de configuracion.

El problema empezó cuando hice la actualizacion de ubuntu 18.04 a 20.04. El software que tenia instalado 
en 18.04 debio ser reinstalado en 20.04. NoIP sufrio desinstalaciones, actualizaciones y reinstalaciones, 
pero nunca quedo funcionando bien. Cuando revisaba el status, me aparecian mensajes como 
"Can't gethostbyname for dynupdate.no-ip.com"
y no sabía por que. Debia estar reiniciando el servicio manualmente para que actualizara los hosts (tenia
funcionando a tres simultaneamente).

Por ultimo, despues de leer muchos articulos, deje el archivo de configuracion 
(/etc/systemd/system/noip2.service) asi:

[Unit]
Description=No-ip.com dynamic IP address updater
After=network.target systemd-resolved.service
After=syslog.target

[Install]
WantedBy=multi-user.target
Alias=noip.service

[Service]
Type=forking
TimeoutStartSec=30
# Start main service
ExecStart=/usr/local/bin/noip2 -d -c /usr/local/etc/no-ip2.conf
Restart=on-failure
RestartSec=30

Tambien, creé el archivo de log y le cambie permisos:

> /var/log/noip2/messages
chmod 0666 /var/log/noip2/messages

Y al revisar el status de vez en cuando, obtenia algo como esto:

~# service noip2 status
● noip2.service - No-ip.com dynamic IP address updater
     Loaded: loaded (/etc/systemd/system/noip2.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2021-08-24 08:22:09 -05; 32min ago
    Process: 305927 ExecStart=/usr/local/bin/noip2 -d -c /usr/local/etc/no-ip2.conf (code=exited, status=0/SUCCESS)
   Main PID: 305928 (noip2)
      Tasks: 1 (limit: 18851)
     Memory: 1.3M
     CGroup: /system.slice/noip2.service
             └─305928 /usr/local/bin/noip2 -d -c /usr/local/etc/no-ip2.conf

Aug 24 08:52:10 softfarrsrv noip2[305928]: < HTTP/1.1 200 OK
Aug 24 08:52:10 softfarrsrv noip2[305928]: < Server: nginx
Aug 24 08:52:10 softfarrsrv noip2[305928]: < Date: Tue, 24 Aug 2021 13:52:10 GMT
Aug 24 08:52:10 softfarrsrv noip2[305928]: < Content-Type: text/plain
Aug 24 08:52:10 softfarrsrv noip2[305928]: < Content-Length: 12
Aug 24 08:52:10 softfarrsrv noip2[305928]: < Connection: close
Aug 24 08:52:10 softfarrsrv noip2[305928]: <
Aug 24 08:52:10 softfarrsrv noip2[305928]: < 1x6.x4.xx.94
Aug 24 08:52:10 softfarrsrv noip2[305928]: ! Our NAT IP address is 1x6.x4.xx.94
Aug 24 08:52:10 softfarrsrv noip2[305928]: ! Last_IP_Addr = 1x6.x4.xx.94, IP = 1x6.x4.xx.94

