# Opdracht automatiseren voor SIMLAB2
 
Voor deze opdracht zal ik de service tftp installeren en configureren via een playbook.
Verder zijn er 2 scripts aan toegevoegd. Script log.sh om de logs uit syslog te halen. Script tar.sh om iedere maand een tarball te maken van de inhoud van de map /tftpboot en /tftplogs
 
## Installeer tftpd-hpa
Na de replay van de ping wordt tftpd-hpa geinstalleerd.
De configguratie zal volledig verlopen via de playbook.
 
## Installeren van een package.
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/package_module.html

## Maak de directory /tftpboot aan.
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html
Plaats deze in de root. Pas owner en group aan naar tftp:tftp.  Als de directory /tftpboot niet in de root geplaats wordt kan er via archive geen running-config doorgestuurd worden vanuit de router.

## Neem een backup van de config tftpd-hpa.
Voor dat je een config file bewerkt neem je best een backup van deze file.
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html

## PAs de config file van tftpd-hpa aan.
Geef de locatie op waar tftp de files bewaard. In dit geval is dit /tftpboot.
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile_module.html

## Stop tftpd-hpa.service.
tftpd-hpa maakt enkel een log aan in de syslog over wanneer de service is gestart of gestopt.  
Ik wil een beter zicht krijgen over welke files er getransporteerd worden door deze service en van en naar waar deze verstuurt worden.
Hievoor gaan we een debug inschakelen.  De service zal eerst gestopt worden, we passen de config file aan zodat er logs gemaakt worden en starten nadien de service handmatig op met -verbosity 5.  Dit geeft zowat de meeste output van logs.
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html

## Pas config tftpd-hpa aan.
Pas TFTP_OPTIONS aan naar "--secure --create --verbose".
Hierdoor zullen er logs gemaakt worden in /var/log/syslog.

## Start service tftpd-hpa handmatig op.
We starten deze keer niet via systemctl de service op maar gaan dit handmatig opstarten.
Hierdoor kan ik verbosity 5 meegeven wat ervoor zorgt dat het maximum aan logging wordt gegenereerd.
Dit zal via het volde cmd gebeuren "sudo service tftpd-hpa start -verbosity 5 ".

## Maak de directory tftplogs aan in /home/rudi.
Deze directory zal gebruikt worden om de tftp logs weg te schrijven.

## logfile.
Vervolgens maak ik de file log.sh aan via touch en geef ik deze file rechten 774 rechten zodat deze uitvoerbaar wordt.
Deze file is nodig omdat er dagelijks een script zal runnen via crontab om de tftp logs uit de syslog weg te schijven. Dit script wordt in 
/usr/local/sbin geplaatst.

Uitleg wat het script doet:

Er wordt 2 keer een grep uitgevoerd.  Blijkbaar wordt er regelmatig gewisseld tussen syslog en syslog.1 om de logs weg te schrijven.  Telkens er een .gz wordt aangemaakt omdat de syslog te groot is veranderd de locatie waar de logs worden weggeschreven.  Omdit op te lossen voer ik dus een grep in syslog en syslog.1 uit.

•	grep -E "$CURRENT_DATE.*in\.tftpd.*WRQ" /var/log/syslog: zoekt naar lijnen in het syslog-bestand die de string "in.tftpd" bevatten.
Zelfde voor *RRQ*.  Dit zowel in syslog en syslog.1

•	cut -d" " -f1-4,6-11: knipt vervolgens de regels op basis van het scheidingsteken "spatie" (-d" ") en bewaart alleen de eerste vier velden en velden 6 tot en met 11. De velden bevatten informatie over de datum en tijd, de naam van de server, het IP-adres van de client en de bestandsnaam die door de client is opgevraagd.

•	>> $LOG_FILE zal alles wegschrijven naar /home/rudi/tftplog.

•  grep "in\.tftp" /var/log/syslog /var/log/syslog >> "$FULL_LOG_FILE"  zal een volledig log maken van alles wat tftp betreft.  Dit zowel in syslog als syslog.1

## Maak een nieuw script aan.
Ik maak een nieuw script aan via touch en geef deze file rechten 774 rechten zodat deze uitvoerbaar wordt.  
Dit is het tar.sh script.
Dit script zal de tftplog en tftpboot files comprimeren en verplaatsen naar /home/rudi/.
dit script is ook in /usr/local/sbin geplaatst.

Uitleg over het script:

•	Het script definieert variabelen voor de logmap, de huidige datum en de namen van de logbestanden en archieven.

•	Er worden verschillende berichten gedefinieerd die worden gebruikt om de status van de acties te rapporteren.

•	Het script maakt een tar-archief van de tftplog-directory met behulp van de tar-opdracht en slaat het op in een bestand met de huidige datum in de bestandsnaam. Als de actie succesvol is, wordt een "SUCCESS" -bericht naar het logbestand geschreven, anders wordt een "ERROR" -bericht geschreven en wordt het script beëindigd met een exit-code 1.

•	Het script controleert de integriteit van het gemaakte tftplog-archief met behulp van de tar-opdracht en controleert of het bestand bestaat en leesbaar is. Als de controle mislukt, wordt een "ERROR" -bericht naar het logbestand geschreven en wordt het script beëindigd met een exit-code 1.

•	Het script maakt een tar-archief van de tftpboot-directory met behulp van de tar-opdracht en slaat het op in een bestand met de huidige datum in de bestandsnaam. Als de actie succesvol is, wordt een "SUCCESS" -bericht naar het logbestand geschreven, anders wordt een "ERROR" -bericht geschreven en wordt het script beëindigd met een exit-code 1.

•	Het script controleert de integriteit van het gemaakte tftpboot-archief met behulp van de tar-opdracht en controleert of het bestand bestaat en leesbaar is. Als de controle mislukt, wordt een "ERROR" -bericht naar het logbestand geschreven en wordt het script beëindigd met een exit-code 1.

•	Het script verwijdert alle bestanden uit de tftplog- en tftpboot-directories met behulp van de rm-opdracht. Als de actie succesvol is, wordt een "SUCCESS" -bericht naar het logbestand geschreven, anders wordt een "ERROR" -bericht geschreven en wordt het script beëindigd met een exit-code 1.

## Maak een crontab aan.
Er worden 2 cron jobs ingepland.  Het log.sh script zal dagelijks om 23u30 lopen en het tar.sh script zal telkes op de 15e van de maand om 23u55 lopen.





