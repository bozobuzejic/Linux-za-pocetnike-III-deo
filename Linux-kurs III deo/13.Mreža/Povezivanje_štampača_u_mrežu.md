POVEZIVANJE ŠTAMPAČA

*Instaliranje štampača (usb i štampača u mreži)

         A. Instaliranja printer drajvera (Brother HL-2030): 
         
         wget https://download.brother.com/welcome/dlf006893/linux-brprinter-installer-2.2.1-1.gz        .....downloadujemo drajver install tool za Brother štampače
         gunzip linux-brprinter-installer-2.2.1-1.gz                                                     .....raspakujemo paket
         bash linux-brprinter-installer-2.2.1-1                                                          .....pokrenemo interaktivnu instalaciju 
         model: HL-2030 -> yes -> y -> y -> 
         service cups restart                                                                            .....restartujemo cups
         lpoptions -d HL2030                                                                             .....podesimo da Brother HL2030 bude default štampač
         reboot                                                                                          .....restartujemo mašinu
                                                
         B. Instaliranje cups servisa na SHARE mašini, za konfiguraciju štampača:
     
         sudo apt install cups                                                     .....instaliramo cups - program za podešavanje štampača
         systemctl enable --now cups.service                                       .....omogućimo i pokrenemo progran
         
         
    Postavljanje USB štampača
         
         1. Postavljanje preko aplikacije (vezane za cups):
         Aplikacija za printer: system-config-printer -> Server -> connect -> odabrati iz padajućeg menija: localhost -> Add -> Devices: odabereno marku printera -> 
         -> Brother -> Model: Brother HL2030 for cups -> OK
         
         2. Postavljanje preko cups:
         Otvorimo web browser -> upišemo: http://localhost:631 -> Administration -> Add printers ->  USB PRINTER -> Continue ->  Connection: štampač se pojavljuje na 
         usb priključku -> Continue ->
            Name: NEKOIMEŠTAMPAČA (Brother_HL2030)
            Description: ModelŠtampača (Brother_HL2030)
            Location: share 
         Continue -> 
            Make: odaberemo marku (proizvođača) štampača (Brother_HL2030)
         Continue->
            Model: odaberemo model izabrane marke štampača (za Brother_HL2030)
            Add Printer -> 
            Media size: A4
            Resolution: odaberemo rezoluciju
            Set Default Options                                                         .....instaliran i podešen štampač 
            
            
    Postavljanje mrežnog štampača  (u datom primeru, Brother štampač je priključen na windows mašinu) 
    
         1. Podesiti windows:    
                            cmd -> ipconfig -> enter -> zapamtimo IP adresu windows mašine;
                            Control Panel -> Programs and Features -> Turn windows features on or off -> + Print and Document Service ->
                            -> čekirati: LPD Print Service -> OK
                            
                            Settings -> Devices -> Printers & scanners -> Brother HL-2030 -> Manage -> Printer properties -> Sharing ->
                            -> čekirati: Share this printer; Share name: Brother_HL-2030_series; čekirati: Render print jobs... -> Aplly -> OK
                            
                            Control Panel -> Network and Internet -> Network and Sharing Center -> Change advanced sharing settings -> 
                            -> čekirati; Turn on network discovery; čekirati: Turn on file and printer sharing -> Save changes
                            
                            Control Panel -> Windows Firewall -> Exceptions -> čekirati: File and Printer Sharing -> OK
         
         2. Instalirati printer drajver i cups servis (A. + B.) na linux mašini
         
         3. Podesiti zaštitu:
         
            iptables -vnL --line-number                                                 .....izvršimo proveru upisanih pravila firewall-a
            iptables -I INPUT 10 -m state --state NEW -p tcp --dport 631 -j ACCEPT      .....dodamo pravila da gw mašina sluša na portu 631           
            iptables-save > /etc/sysconfig/iptables                                     .....snimimo promene
            systemctl reload iptables.service                                           .....restartujemo iptables
          
         4. Podesiti konfiguracioni fajl za cups: 
            vim /etc/cups/cupsd.conf                                                    .....otvaramo konfiguracioni fajl za editovanje
            (vim /etc/cups/cupsd.conf                                                        alternativno, na nekim sistemima)
                 DefaultEncryption Never                                                      .....upisujemo u prvi red (a iznad MaxLogSize 0)
                 MaxLogSize 0
                 ...
                 # Only listen for connections from the local machine.
                 Listen localhost:631
                 Listen /var/run/cups/cups.sock
                 ...
                 # Restrict access to the server...
                 <Location />
                 Order allow,deny
                 allow localhost                                                              .....ubacujemo red kojim dozvoljavamo pristup serveru sa localhosta 
                 </Location>
                 ...
                 # Restrict access to the admin pages...
                 <Location /admin>
                 Order allow,deny
                 allow localhost                                                              .....dozvoljavamo pristup administrativnoj stranici sa localhosta i 
                 </Location>                                                                       
                 ...
                                                  esc -> shift: -> wq                         .....snimimo izmene u fajlu
                                             
         5. Podesiti štampač preko cups servisa 
        
         Web browser -> upišemo: http://localhost:631 -> Administration -> Add printers ->  Other Network Printers: LPD/LPR Host or Printer ->                                                   
         -> Continue ->  Connection: lpd://192.168.0.115/Brother_HL-2030_series (naziv printera kao u Share name polju printera na windows hostu) ->
         -> Continue ->
            Name: NEKOIMEŠTAMPAČA (Brother_HL2030)
            Description: ModelŠtampača (Brother sa win)
            Location: share 
         Continue -> 
            Make: odaberemo marku (proizvođača) štampača (Brother_HL2030)
         Continue->
            Model: odaberemo model izabrane marke štampača (za Brother_HL2030)
            Add Printer -> 
            Media size: A4
            Resolution: odaberemo rezoluciju
            Set Default Options                                                         .....instaliran i podešen štampač 
            
            
         5.a. Alternativno - Podešavanje štampača na ostalim klijentima u mreži (korisniku štampača sa druge mašine)
          
         Application Launcher -> Printers -> Add -> Network Printer -> Find Network Printer -> Host: 192.168.xxx.xxx -> Find -> 
         -> Devices: select printer (Brother_HL2030)-> Forward -> Choose Driver: select driver -> User: nekiuser, Password: password -> Authenticate ->  
         -> Select the printer's manufacturer -> Forward -> Select the printer's model -> Forward -> Apply -> Print Test Page      
         
                                                                                                                                                                                   
         6. Ostalo:
       
         cupsaccept NAZIVŠTAMPAČA                                                      .....prihvatimo štampač
         cupsenable NAZIVŠTAMPAČA                                                      .....omogućimo štampač
         lpoptions -d NAZIVŠTAMPAČA                                                    .....odredimo default štampač
         lpstat -p -d                                                                  .....prikaz štampača u sistemu
         lpq                                                                           .....prikaz aktuelnog stanja i redosled za štampanje
         
         lpstat -t                                                                     .....prikazuje status spooler-a i red čekanja zahteva (ako ih ima)
         
         lpr /path/to_folder/nekifajl                                                  .....štampamo nekifajl
         lpr -P EPSON-AL-M2000 /home/nekiuser/Downloads/neki.txt                       .....zadajemo komandu štampaču EPSON-AL-M2000 da štampa neki.txt
         lprm 3                                                                        .....uklanjamo fajl za štampanje pod oznakom job 3                                                                                                      
               
       
 

