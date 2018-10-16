# Aplicación en .Net Core 2 sobre Linux (Ubuntu Server)

En el siguiente taller, veremos como crear de manera simple una máquina virtual Linux en Azure y subir en ella una aplicación web 
creada con .Net Core 2.0

## Creando la máquina
Lo primero que debemos hacer, es crear una cuenta en Azure, ya sea premium o de evaluación gratuita. Una vez terminado el 
paso anterior, procedemos a crear nuestra máquina virtual **Ubuntu Server 18**  
[!ubuntu1()]
[!ubuntu1()]

## Configurando
Una vez creada nuestra máquina virtual, ejecutaremos los siguientes comandos con los que instalaremos el SDK de .Net Core en Linux.
El artículo en donde detallan cada uno de los comandos se encuentra en: 

**Instalamos el SDK**  
```
  wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.asc.gpg
  sudo mv microsoft.asc.gpg /etc/apt/trusted.gpg.d/

  wget -q https://packages.microsoft.com/config/ubuntu/18.04/prod.list
  sudo mv prod.list /etc/apt/sources.list.d/microsoft-prod.list 
  sudo chown root:root /etc/apt/trusted.gpg.d/microsoft.asc.gpg 
  sudo chown root:root /etc/apt/sources.list.d/microsoft-prod.list

  sudo apt-get install apt-transport-https 
  sudo apt-get update 
  sudo apt-get install dotnet-sdk-2.1 
```
**Instalamos y configuramos Apache**  
Con los siguientes comandos, instalamos apache en ubuntu, y activamos los módulos a activar requeridos para convertirlo 
en Proxy inverso
```
  sudo apt-get install apache2
  sudo a2enmod
  proxy proxy_ajp proxy_http rewrite deflate headers proxy_balancer proxy_connect proxy_html
  service apache2 restart
```
**Configuramos el entorno para nuestra aplicación Aspnet Core**  
Luego de activar los módulos de apache y de haberlo instalado, crearemos el host virtual que alojará el proxy 
inverso descrito anteriormente
```
  sudo nano /etc/apache2/conf-enabled/aspnetcorelinux.conf
```
dentro del archivo, pegamos lo siguiente:
```
  <VirtualHost *:80>
	  ProxyPreserveHost On
	  ProxyPass / http://127.0.0.1:5000/
	  ProxyPassReverse / http://127.0.0.1:5000/
	  ErrorLog /var/log/apache2/aspnetcorelinux-error.log
	  CustomLog /var/log/apache2/aspnetcorelinux-access.log common
  </VirtualHost>
```
guardamos los cambios del archivo y ejecutamos lo siguiente para verificar que haya quedado correctamente
```
  sudo service apache2 restart
  sudo journalctl -xe
```

## Creando la aplicación web
Creamos una aplicación web AspNet Core 2.0 desde VisualStudio  
[imagen]

## Publicando
Luego de crearla, probamos la aplicación y publicamos localmente. Los archivos generados en la publicación, los comprimimos 
en un .ZIP para subirlos al servidor Ubuntu  
[imagen]

## Desplegando
Teniendo ya el archivo local de la publicación, lo subimos desde la terminal  
[imagen]

Luego de subirlo, movemos el archivo desde su origen (en éste caso /home/mymaquina) hasta el lugar donde tendremos el sitio 
de .Net Core que en este caso sería (/var/aspnetcorelinux). Ya teniendo el archivo, lo descomprimimos y verificamos que en la carpeta 
/var/aspnetcorelinux/ se encuentren los archivos de la aplicación.  

## Verificando
Al tener la aplicación en el servidor, nos queda un último paso
```
  sudo nano /etc/systemd/system/kestrel-aspnetcorelinux.service
```
En el nuevo archivo, pegamos el siguiente bloque de código
```
  [Unit]
  Description=Example ASP .NET Web Application running on Ubuntu 16.04
  [Service]
  WorkingDirectory=/var/aspnetcorelinux
  ExecStart=/usr/bin/dotnet /var/aspnetcorelinux/aspnetcorelinux.dll
  Restart=always
  RestartSec=10
  SyslogIdentifier=dotnet-demo
  User=www-data
  Environment=ASPNETCORE_ENVIRONMENT=Production
  [Install]
  WantedBy=multi-user.target
```
Lanzamos nuestra aplicación con los siguientes comandos y listo!!
```
  sudo systemctl enable kestrel-aspnetcoredemo.service
  sudo systemctl start kestrel-aspnetcoredemo.service
```
## Probando
Probemos ahora nuestro sitio en .Net Core desde una máquina virtual Ubuntu en Azure  
[imagen]

## Conclusión
Con éste taller logramos experimentar que la nueva tecnología de Microsoft es realmente multiplataforma y sencilla de implementar en 
sistemas operativos diferentes a Windows y en servidores de aplicaciones diferentes a IIS (el temido IIS)
