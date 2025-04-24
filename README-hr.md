[English](README.md) ∙ Hrvatski

# dashcam-pi

Projekt **dashcam-pi** je kompaktna auto dash kamera za snimanje vožnje, a može se koristiti i kao mrežna kamera za videonazor, potencijalno kao akcijska kamera ili baby monitor. Za izradu sam koristio Raspberry Pi Zero 2 W, ali može poslužiti bilo koja verzija Raspberry Pi-a koja podržava modul s kamerom. Za ovako koncipiran projekt najveća snaga je u softwareu pa sam razvio Python skriptu kojom se upravlja dash kamerom, od live previewa, snimanja videa, slikanja do preglednika tako snimljenog materijala na SD kartici.

U nastavku je tutorial kako za ovaj projekt posložiti Raspberry Pi, instalirati i ako se želi prilagoditi Python skripta, kako upravljati kamerom i dijeliti snimljeno preko mreže.


## Komponente

Osnova projekta je Raspberry Pi Zero 2 W i Raspberry Pi Camera Module 3 Wide, može poslužiti i bilo koja druga verzija Raspberry Pi-a koja podržava neki od dostupnih modula kamere i Wi-Fi. Što je novija verzija Raspberryja bit će bolje, fluidnije performanse. Korištenje Zero 2 W donosi kompaktnost rješenja. Također i novija kamera omogućuje bolju sliku i veću rezoluciju. Odabir Camera Module 3 Wide mi je bio bitan pošto Wide verzija ima širi kut snimanja što je dosta bitno za dash kameru. Wi-Fi je bitan zbog spajanja na PC, konfiguriranje, upload Python koda, upravljanje kamerom preko mreže i download fileova snimki s SD kartice. Poželjno je također imati SD karticu što većeg kapaciteta.

Za napajanje projekta koristim USB u autu bilo ugrađeni ili preko 12V punjača, a može poslužiti i powerbank.

| **Korištene komponente** |
|---|
| 1x Raspberry Pi Zero 2 W |
| 1x Raspberry Pi Camera Module 3 Wide |
| 1x Raspberry Pi Zero Camera Adapter |
| 1x powerbank |
| 1x 12V USB autopunjač |
| 1x što duži USB kabel za spajanje powerbanka / autopunjača na Raspberry Pi |
| 4x 2.5 mm i 4x 2mm vijci, podloške i matice za sastavljanje u 3D isprintanom kućištu |

Prilikom spajanja Raspberry Pi-a s modulom kamere najrizičniji dio je ispravno okrenuti kabel pošto se mora paziti na poziciju pinova kabela i konektora na Raspberryju. Ostalo je relativno jednostavno, za napajanje najbolje spojiti USB kabel na micro USB konektor koji je označen s PWR.

Kućište za ovu dash kameru je vlastiti dizajn i 3D isprintano. Ima veće otvore za pristup SD kartici i USB konektorima za svaki slučaj, a vijci i matice se samo koriste za fiksiranje pločice i kamere u kućištu.

Komponente tijekom sastavljanja i konačno sastavljen projekt:

![alt](/images/raspberry_camera.png)

![alt](/images/raspberry_camera_case.png)

![alt](/images/dashcam_assembled.png)


Plastično kućište koje sam dizajnirao može se preuzeti i isprintati na 3D printeru, odgovara dimenzijama mojih korištenih komponenti. Izradio sam ga koristeći [Sketchup](https://app.sketchup.com/app) pa u [**/resources**](/resources/) možete pronaći originalni **skp** file i **stl** fileove za ispis na 3D printeru.

**.stl** fileovi:<br>
[![Download Icon]](/resources/DashCam3DcaseFront.stl)
[![Download Icon]](/resources/DashCam3DcaseBack.stl)

Nakon što su komponente spojene slijedi konfiguriranje Raspberry Pi-a.


## Raspberry Pi konfiguriranje i potrebne instalacije

U ovom dijelu preduvjet je već instaliran Raspberry Pi OS, npr. instaliran je preko Raspberry Pi Imagera pa taj dio nije obuhvaćeno tutorialom. Također je već prilikom instalacije kreiran **pi** user s passwordom te se taj user koristi u nastavku. Raspberry inače ne koristim kad je spojen na monitor nego se spajam i konfiguriram u headless modu. To može samo malo komplicirati rad u početku, ali kasnije se brže boota kad se ugasi GUI. Bitno mu je složiti na koji se Wi-Fi spaja i koja će mu biti fiksna IP adresa.

Kako bi se moglo na njega spajati preko kućnog Wi-Fija prvo je potrebno na njegovoj SD kartici u rootu kreirati prazan file naziva **ssh** bez ekstenzije:

![alt](/images/empty_ssh_file.png)

Također na SD kartici u rootu kreirati file točnog naziva **wpa_supplicant.conf** u kojem će se popuniti podaci za spajanje na Wi-Fi Access Pointove. Ako će se spajati samo na kućni Wi-Fi, dovoljan je jedan entry, ali najbolje je unijeti kućni AP SSID kao manjeg prioriteta spajanja i mobilni AP SSID višeg prioriteta na koji će se spajati Raspberry kada se koristi u živom radu u autu. Sadržaj samog file treba biti:
```sh
country=YOUR_COUNTRY_CODE
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
    ssid="YOUR_HOME_WIFI_SSID"
    psk="YOUR_HOME_WIFI_PASSWORD"
    key_mgmt=WPA-PSK
    priority=0 #lower priority
}
network={
    ssid="YOUR_MOBILE_WIFI_SSID"
    psk="YOUR_MOBILE_WIFI_PASSWORD"
    key_mgmt=WPA-PSK
    priority=1 #higher priority
}
```

Pod **YOUR_COUNTRY_CODE** staviti oznaku države (moja je HR), pod **YOUR_HOME_WIFI_SSID** staviti SSID kućnog Wi-Fija i u **YOUR_HOME_WIFI_PASSWORD** staviti password mreže. Također za mobilni Wi-Fi koji se pokrene za dijeljenje na mobitelu unijeti SSID **YOUR_MOBILE_WIFI_SSID** i password **YOUR_MOBILE_WIFI_PASSWORD**. Ovdje je očito kako sigurnost nije na razini, ali barem će se Raspberry prilikom bootanja spojiti na mrežu. Namjera je ako je pokrenut mobilni AP spojiti se na njega, u suprotnom na kućni, ali za to će trebati još odraditi nekoliko koraka. Sljedeće je nakon što se Raspberry boota pronaći njegov IP koji je dobio na mreži što se može pronaći najčešće spajanjem i provjerom na kućnom routeru. Bitno za testiranje u kućnim uvjetima ili npr. za video stream kao nadzorna kamera ako neće biti u autu.

Spojiti se preko njegove IP adrese koristeći **Putty**, jednostavno unese se SSH konekcija s IP adresom Raspberry Pi-ja pa kad zatraži unese se username i password. Nakon spajanja dobije se odmah shell i nakon toga treba pokrenuti konfiguraciju Raspberry Pi OS-a:
```sh
sudo raspi-config
```

Dobije se ekran Configuration Tool i preko njega obavezno postaviti autologin s userom **pi** (postavio defaultnog usera) radi kasnijeg automatskog pokretanja Python skripte projekta:

![alt](/images/raspi_config1.png)

Također obavezno treba omogućiti SSH server:

![alt](/images/raspi_config2.png)

Od ostalih opcija, najbitnije je na konfig ekranu ugasiti Desktop GUI i login, ostaviti text konzolu i login kako bi se ubrzao boot.

![alt](/images/raspi_config3.png)

Nakon izlaska iz raspi-config potrebno je postaviti statičku IP adresu kako ne bi morali svaki put tražiti koji je IP kad se Raspberry spoji na router. Za to se editira file **/etc/dhcpcd.conf**, od samih editora kroz shell koristim najviše **pico**:
```sh
sudo pico /etc/dhcpcd.conf
```

U fileu je potrebno doći do ovog dijela u kojem se konfigurira statička IP adresa, inicijalno su sve linije u fileu zakomentirane s **#**. Linije kao u primjeru otkomentirati i izmijeniti prema adresi routera i željenoj fiksnoj IP adresi:
```sh
# Example static IP configuration:
interface wlan0
static ip_address=<YOUR_RASPBERRY_IP>/24
#static ip6_address=fd51:42f8:caae:d92e::ff/64
static routers=<YOUR_ROUTER_IP>
static domain_name_servers=<YOUR_ROUTER_IP> 8.8.8.8
```
Pod **<YOUR_RASPBERRY_IP>** staviti željenu fiksnu adresu preko koje će spajati na Raspberry, može biti i ona pod kojom se prvi put automatski Raspberry spojio na router, a pod **<YOUR_ROUTER_IP>** staviti IP adresu routera, npr. neki automatski imaju 192.168.0.1 i ta adresa se ne mijenja. Promjene spremiti i izaći iz editora.

Nastavak potrebnih mrežnih postavki za spajanje na Wi-Fi je kroz tool **nmtui** kad se pokrene kroz shell:
```sh
sudo nmtui
```

Konfigurirati konekciju za kućni Wi-Fi i dodati za mobilni Access Point, za obje staviti fiksnu IP addresu koja će biti za Raspberry jer će se preko nje pristupati i interfaceu za upravljanje dash kamerom:

![alt](/images/nmtui_config1.png)

![alt](/images/nmtui_config2.png)

![alt](/images/nmtui_config3.png)

![alt](/images/nmtui_config4.png)

Na mobilnom Access Pointu pod IPv4 CONFIGURATION može se postaviti i automatska dodjela adrese ako se ne uspijevate spojiti, ali u tom slučaju se nakon spajanja Raspberryja na mobitel treba ručno pronaći IP adresa (na Androidu koristim Network Analyzer).

Nakon promjena restartati Raspberry:
```sh
sudo shutdown -r now
```

Za pregled konekcija, dodatnu konfiguraciju i ručno spajanje koristit ću tool **nmcli** kroz shell, pregled Wi-Fi konekcija koje su postavljene u **nmtui** dobije se kroz:
```sh
sudo nmcli connection
```

U rezultatu pronaći UUID koji je dodan za mobilni AP hotspot pošto će se za njega podići prioritet, u idućoj naredbi **HOTSPOT_UUID** postaviti UUID hotspota:
```sh
sudo nmcli connection modify HOTSPOT_UUID connection.autoconnect yes connection.autoconnect-priority 100
```

Ako se želi probati automatsko spajanje na taj AP prilikom bootanja Raspberryja, treba ga restartati.
Lista dostupnih Wi-Fi mreža i njihovih SSID-ova može se dohvatiti koristeći:
```sh
sudo nmcli dev wifi list
```

Ako se pojavio neki novi AP mreže ili se postojeći ne vidi jer ga Raspberry nije pronašao prilikom bootanja, potrebno je reskenirati dostupne:
```sh
sudo nmcli dev wifi rescan
```

Za naknadno spajanje na neki AP, odnosno promjenu aktivnog AP-a što je dobro za korištenja kad se želi spojiti na mobilni hotspot, potrebno je inicirati spajanje, umjesto **YOUR_MOBILE_WIFI_SSID** postaviti AP hotspota:
```sh
sudo nmcli dev wifi connect YOUR_MOBILE_WIFI_SSID
```

Obje prethodne naredbe dodat ću kasnije u proceduru pokretanja Raspberryja, prije pokretanja Python skripte.
> :warning: Kada će interface dash kamere biti aktivan, prebacivanje između mreža ili paljenje / gašenje mobilnog AP hotspota može biti izazovno za rad pošto se interfaceu pristupa putem IP adrese kroz browser. Ako se ne može nakon paljenja / gašenja hotspota pristupiti interfaceu, znači kako se Raspberry nije uspio spojiti pa je potreban restart pošto se drugačije ne može pristupiti interfaceu.

Prije instalacije ostalih paketa i libraryja koji će se koristiti najbolje odraditi update postojećih:
```sh
sudo apt update
sudo apt upgrade
```

Python je već instaliran na OS-u (s ```sh python --version``` provjeriti verziju, najbolje imati što noviju) pa slijedi instalacija libraryja koji će se koristiti u source kodu. Također ću koristiti **pip** koji ako ne postoji također treba instalirati.
```sh
sudo apt install python3-pip
```

Pošto će Python skripta podići web server, treba i instalacija **Flask** ako ne postoji (provjeriti postojanje s ```sh pip list | grep Flask```):
```sh
sudo apt install python3-flask
```

U sourceu se koristi **Picamera2** library, također ga treba instalirati:
```sh
sudo apt install python3-picamera2
```

Kamera će video snimati u **mp4** formatu pa je potrebna instalacija **ffmpeg** libraryja:
```sh
sudo apt install ffmpeg
```

Za network share koristi se **Samba** pa je potrebna instalacija ako već ne postoji:
```sh
sudo apt-get install samba
```

Za upload i download fileova (Python koda, korištenih resursa, snimljenog videa i slika) na Raspberryjevoj SD kartici dok je na mreži najbolje i najpraktičnije je koristiti FTP protokol s nekim besplatnim klijentom kao što je npr. FileZilla. Zato kroz shell treba instalirati i FTP server:
```sh
sudo apt install vsftpd
```

Nakon instalacije slijedi konfiguracija FTP servera editiranjem filea **/etc/vsftpd.conf**:
```sh
sudo pico /etc/vsftpd.conf
```

U fileu pronaći ove zakomentirane linije i otkomentirati ih brisanjem **#**:
```sh
#write_enable=YES
#local_umask=022
#anon_upload_enable=YES
```

Na kraj filea dodati sljedeću liniju:
```sh
user_sub_token=$USER
```

Restartati FTP deamon:
```sh
sudo service vsftpd restart
```


## Instalacija dashcam-pi projekta

Sljedeći korak je kreiranje direktorija za potrebe **dashcam-pi** projekta. Pozicionirati se u **/home/pi**, odnosno home direktorij logiranog usera (koristim default **pi**) i u njemu kreirati direktorij **dashcam-pi**. Nakon toga u direktoriju **dashcam-pi** kreirati njegove poddirektorije **src** i **DCIM**. U **DCIM** kreirati poddirektorije **PHOTO**, **THUMB** i **VIDEO**, ako se sada ne kreiraju, kreirat će se prilikom prvog pokretanja Python skripte.
U direktorij **src** treba kopirati sve fileove koji se nalaze u direktoriju [**/src**](/src/) ovog tutoriala.
Nakon kreiranja i kopiranja sourceva struktura direktorija trebala bi izgledati ovako:

/home/pi/dashcam-pi<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── DCIM<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── PHOTO<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── THUMB<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└── VIDEO<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└── src<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── dashcam-pi.py<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── favicon.png<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── gallery.html<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── index.html<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└── styles.css<br>

Detaljnije o fileovima u **src** direktoriju nalazi se u nastavku gdje analiziram Python skriptu.

Nakon ovako kreirane strukture direktorija bitno je snimljeni materijal koji se nalazi u **DCIM** direktoriju projekta dodati u **Samba** share, editirati konfiguraciju Sambe:
```sh
sudo pico /etc/samba/smb.conf
```

Na kraj filea dodati parametre sharea:
```sh
[dashcam]
comment = dashcam-pi shared folder
browseable = yes
writable = yes
path = /home/pi/dashcam-pi/DCIM
guest ok = yes
only guest = yes
read only = no
create mask = 0777
directory mask = 0777
public = yes
force user = pi

[nobody]
browseable = no
```

Kao što se vidi, bitan je path do direktorija **DCIM** pošto samo njega želim shareati. Prilikom Samba konfiguracije bitno je postaviti forsiranje usera s kojim se spaja na share pošto se preko tog usera kreiraju direktoriji sharea, u fileu vidljivo pod ```sh force user = pi```. Također, ako se spajanjem na Raspberry Pi na mrežnom shareu osim direktorija **dashcam** pojavi i nepoznat direktorij naziva **nobody**, tada je u **/etc/samba/smb.conf** potrebno skrivanje tog direktorija, samo dodati na kraj filea kao u mom primjeru:
```sh
[nobody]
browseable = no
```

U samom fileu **/etc/samba/smb.conf** treba provjeriti i workgroup te postaviti u kojem se nalazi Raspberry Pi, defaultni workgroup u fileu mi je ok pošto je u njemu i računalo preko kojeg pristupam Raspberry shareu:
```sh
workgroup = WORKGROUP
```

Nakon svih promjena po konfiguraciji restartati Samba servis:
```sh
sudo systemctl restart smbd.service
```

Nakon restarta trebao bi biti moguć pristup na network share preko IP adrese **<YOUR_RASPBERRY_IP>** koja je postavljena prilikom konfiguriranja Raspberry Pi-ja, npr. ovo je moj slučaj:

![alt](/images/samba_share.png)


## Analiza Python skripte

U direktoriju [**/src**](/src/) nalazi se samo jedna Python skripta **dashcam-pi.py**, source koji se pokreće za ovaj projekt. Skripta sadrži web server i interface za upravljanje Raspberry Pi dash kamerom. Interface je zapravo sadržaj **index.html** stranice koju učitava Python skripta, a sadrži gumbe za upravljanje Raspberryjevom kamerom, live preview / streaming kamere i galeriju snimljenog materijala. Galerija je linkana na **gallery.html**, stilovi obje HTML stranice interfacea nalaze se u **styles.css** i dodatno ikona stranice u browseru je **favicon.png**.

Kompletan Python source code **dashcam-pi.py** izgleda ovako:
```python
"""
  DashCam Pi - Raspberry Pi car dash camera web app project

  Author: Darko Golner
    https://www.darko-golner.com
    https://github.com/dgolner
"""

import io
from picamera2 import Picamera2
from picamera2.encoders import H264Encoder
from picamera2.encoders import MJPEGEncoder
from picamera2.encoders import Quality
from picamera2.outputs import FileOutput
from picamera2.outputs import FfmpegOutput
import socketserver
import threading
from threading import Condition
from http import server
import os
import datetime
import time
from PIL import Image

# global variables
camera_enabled = True
streaming = False
recording = False
global recording_thread
folder = '../DCIM/'

# make new directories if they don't exist
cmd = 'mkdir -p ' + folder + 'PHOTO ' + folder + 'VIDEO ' + folder + 'THUMB'
fp = os.popen(cmd)
fp.close()

# get my IP address
cmd = 'ip addr show wlan0 | grep global | grep inet | awk \'{print $2}\''
fp = os.popen(cmd)
IPAddr = fp.read() 
fp.close()
IPAddr = IPAddr[:IPAddr.index('/')]

# get index.html template content
page = ''
with open('index.html', 'r') as f:
  page = f.read()

# web video streaming content
class StreamingOutput(io.BufferedIOBase):
  def __init__(self):
    self.frame = None
    self.condition = Condition()

  def write(self, buf):
    if buf.startswith(b'\xff\xd8'):
      with self.condition:
        self.frame = buf
        self.condition.notify_all()

# HTTP request handler
class StreamingHandler(server.BaseHTTPRequestHandler):
  def do_GET(self):
    global page
    global camera_enabled
    global streaming
    global recording
    global recording_thread
    global folder
    # base path redirects to index.html
    if self.path == '/':
      self.send_response(301)
      self.send_header('Location', '/index.html')
      self.end_headers()     
    elif self.path == '/index.html':
      # substitute custom html tags with paths and text in user interface template (index.html content)
      content = page
      if camera_enabled:
        content = content.replace('\'<!--##VISIBILITY##-->\'', 'visible') \
                  .replace('\'<!--##ERROR##-->\'', 'hidden')
      else:
        content = content.replace('\'<!--##VISIBILITY##-->\'', 'hidden') \
                  .replace('\'<!--##ERROR##-->\'', 'visible')
      if streaming:
        content = content.replace('<!--##HREF_PREVIEW##-->', '/stop_streaming') \
                  .replace('<!--##BUTTON_PREVIEW##-->', 'STOP PREVIEW') \
                  .replace('<!--##IMG_STREAM##-->', '<img src="stream.mjpg" alt="camera preview">')
      else:
        content = content.replace('<!--##HREF_PREVIEW##-->', '/start_streaming') \
                  .replace('<!--##BUTTON_PREVIEW##-->', 'START PREVIEW')
      if recording:
        content = content.replace('<!--##HREF_RECORD##-->', '/stop_recording') \
                  .replace('<!--##BUTTON_RECORD##-->', 'STOP VIDEO')
      else:
        content = content.replace('<!--##HREF_RECORD##-->', '/start_recording') \
                  .replace('<!--##BUTTON_RECORD##-->', 'START VIDEO')
      content = content.encode('utf-8')
      self.send_response(200)
      self.send_header('Content-Type', 'text/html')
      self.send_header('Content-Length', len(content))
      self.end_headers()
      self.wfile.write(content)
    elif self.path == ('/start_streaming'):
      if not streaming:
        # start video stream (live preview) using streaming encoder and lower resolution camera image stream
        camera.start_recording(encoder = streaming_encoder, output = FileOutput(output), name = 'lores')
        time.sleep(0.2)
        streaming = True
      self.send_response(302)
      self.send_header('Location', '/index.html')
      self.end_headers()    
    elif self.path == '/start_recording':      
      if not recording:
        # start video recording in separate thread
        recording = True
        recording_thread = threading.Thread(target = record_video)
        recording_thread.start()
      self.send_response(302)
      self.send_header('Location', '/index.html')
      self.end_headers()
    elif self.path == ('/stop_streaming'):
      if streaming:
        # stop video stream (live preview)
        camera.stop_encoder(streaming_encoder)
        streaming = False
      self.send_response(302)
      self.send_header('Location', '/index.html')
      self.end_headers()
    elif self.path == '/stop_recording':
      if recording:
        # stop video recording
        recording = False
        recording_thread.join()
      self.send_response(302)
      self.send_header('Location', '/index.html')
      self.end_headers()
    elif self.path == '/take_photo':
      # define picture filename from timestamp
      x = datetime.datetime.now()
      filename = folder + 'PHOTO/dcp_' + x.strftime('%Y%m%d%H%M%S') + '.jpg'
      # save picture from main camera image stream and make its thumbnail copy
      save_with_thumbnail(filename)
      self.send_response(302)
      self.send_header('Location', '/index.html')
      self.end_headers()
    elif self.path == '/restart':
      # restart Raspberry Pi
      stop_video()
      self.send_response(302)
      self.send_header('Location', '/index.html')
      self.end_headers()
      cmd = 'sudo shutdown -r now'
      fp = os.popen(cmd)
      fp.close()      
    elif self.path == '/turnoff':
      # shut down Raspberry Pi
      stop_video()
      self.send_response(302)
      self.send_header('Location', '/index.html')
      self.end_headers()
      cmd = 'sudo shutdown now'
      fp = os.popen(cmd)
      fp.close()
    elif self.path == '/stream.mjpg' and streaming:
      # get and return video stream content
      self.send_response(200)
      self.send_header('Age', 0)
      self.send_header('Cache-Control', 'no-cache, private')
      self.send_header('Pragma', 'no-cache')
      self.send_header('Content-Type', 'multipart/x-mixed-replace; boundary=FRAME')
      self.end_headers()
      # read and return video stream output buffer
      try:
        while True:
          with output.condition:
            output.condition.wait()
            frame = output.frame
          self.wfile.write(b'--FRAME\r\n')
          self.send_header('Content-Type', 'image/jpeg')
          self.send_header('Content-Length', len(frame))
          self.end_headers()
          self.wfile.write(frame)
          self.wfile.write(b'\r\n')
      except Exception as e:
        pass
    elif self.path == '/favicon.png':
      # get and return favicon.png
      self.send_response(200)
      self.send_header('Content-Type', 'image/png')
      self.end_headers()
      with open(self.path.replace('/', '', 1), 'rb') as f:
        self.wfile.write(f.read())
    elif self.path == '/styles.css':
      # get and return styles.css
      self.send_response(200)
      self.send_header('Content-Type', 'text/css')
      self.end_headers()
      with open(self.path.replace('/', '', 1), 'rb') as f:
        self.wfile.write(f.read())
    elif self.path == '/gallery.html':
      # get gallery.html content and substitute custom html tag with list of paths to videos and pictures
      with open('gallery.html', 'r') as f:
        content = f.read()
      content = content.replace('<!--##IMG_GALLERY##-->', get_files())
      content = content.encode('utf-8')
      self.send_response(200)
      self.send_header('Content-Type', 'text/html')
      self.send_header('Content-Length', len(content))
      self.end_headers()
      self.wfile.write(content)
    elif folder.replace('..', '') in self.path and '.jpg' in self.path:
      # get and return .jpg file from corrected path
      self.path = self.path.replace('/', '../', 1)
      self.send_response(200)
      self.send_header('Content-Type', 'image/jpeg')
      self.end_headers()
      try:
        with open(self.path, 'rb') as f:
          self.wfile.write(f.read())
      except Exception as e:
        pass
    elif folder.replace('..', '') in self.path and '.mp4' in self.path:
      # get and return .mp4 file from corrected path
      self.path = self.path.replace('/', '../', 1)
      self.send_response(200)
      self.send_header('Content-Type', 'video/mp4')
      self.end_headers()
      try:
        with open(self.path, 'rb') as f:
          self.wfile.write(f.read())
      except Exception as e:
        pass
    else:
      # for unknown path, redirect to index.html, it's better than return code 404
      self.send_response(302)
      self.send_header('Location', '/index.html')
      self.end_headers()

# streaming server class
class StreamingServer(socketserver.ThreadingMixIn, server.HTTPServer):
  allow_reuse_address = True
  daemon_threads = True

# capture picture from main camera image stream and make its thumbnail copy
def save_with_thumbnail(origin):
  thumb = origin.replace('PHOTO', 'THUMB')
  camera.capture_file(origin)
  img = Image.open(origin)
  img.thumbnail((160, 90), Image.NEAREST)
  img.save(thumb)

# capture video using recording encoder and main high resolution camera stream
def record_video():
  global recording
  global folder
  x = None
  d = 0
  # recording flag that stops this loop is set from the main thread
  while recording:
    y = datetime.datetime.now()
    if x is not None:
      d = (y - x).total_seconds()
    if x is None or d > 60:
      # every video file contains 60 seconds of recording
      x = y
      filename = folder + 'VIDEO/dcp_' + x.strftime('%Y%m%d%H%M%S') + '.mp4'
      try:
        camera.stop_encoder(recording_encoder)
      except:
        pass
      camera.start_recording(encoder = recording_encoder, output = FfmpegOutput(output_filename = filename, audio = False), quality = quality, name = 'main')
      # every video file gets its thumbnail picture for gallery
      save_with_thumbnail(filename.replace('VIDEO/dcp_', 'THUMB/dcpvid_').replace('.mp4', '.jpg'))
    time.sleep(2)
  camera.stop_encoder(recording_encoder)

# stop video recording
def stop_video():
  global camera_enabled
  if camera_enabled:
    camera.stop_recording()
    camera.stop()

# get list of videos and pictures from DCIM subfolders and return as gallery content
def get_files():
  files = ''
  # list all dcp_*.jpg and dcp_*mp4 files in DCIM folder and its subfolders excluding THUMB subfolder, extract timestamp, path name, then sort from newest to oldest file, return path with filename
  cmd = 'find .. -type f \( -iname "dcp_*.jpg" -o -iname "dcp_*.mp4" -path "*/DCIM/*" \) \( -not -path "*/THUMB/*" \) -printf "%T@\t%p\n" | sort -n -r | cut -f2-'
  fp = os.popen(cmd)
  for line in fp:
    if '.mp4' in line:
      files = files + '<video wihth="160" height="90" controls preload="none" poster="%s"><source src="%s" type="video/mp4"></video>\n' %  \
        (line.replace('VIDEO', 'THUMB').replace('dcp_', 'dcpvid_').replace('.mp4', '.jpg'), line)
    if '.jpg' in line:
      files = files + '<img wihth="160" height="90" src="%s" alt="photo">\n' % line.replace('PHOTO', 'THUMB')
  fp.close()
  return files

# Raspberry Pi camera module settings for main high resolution stream and lower resolution stream
# main stream is for video and pictures, lores is for web streaming
# change resolutions and quality for your purposes
try:
  camera = Picamera2()
  output = StreamingOutput()
  main_stream = {'size': (1920, 1080)}
  lores_stream = {'size': (640, 360), 'format': 'YUV420'}
  config = camera.create_video_configuration(main_stream, lores_stream, encode = 'lores')
  camera.configure(config)
  camera.start()
  streaming_encoder = MJPEGEncoder()
  streaming_encoder.quality = 20
  recording_encoder = H264Encoder(10000000)
  quality = Quality.VERY_HIGH

  # uncomment these lines if you want to start video recording right away
  #recording = True
  #recording_thread = threading.Thread(target = record_video)
  #recording_thread.start()
except Exception as e:
  camera_enabled = False
  print(e)

# start streaming server
try:
  address = ('', 8000)
  print('IP address http://%s:%s' % (IPAddr, address[1]))
  server = StreamingServer(address, StreamingHandler)
  server.serve_forever()
finally:
  stop_video()
```

[![Download Icon]][def-dashcam-pi.py]

Nakon proučavanja source koda treba ga pokrenuti kroz shell (obavezno biti pozicioniran u direktoriju **/src**):
```sh
python dashcam-pi.py
```

Pokretanjem skripte prvo se kreira struktura DCIM direktorija za snimljeni materijal ako direktoriji ne postoje, dohvaća se IP adresa Raspberry Pi-ja i učitava index.html template interfacea za upravljanje dash kamerom (GUI).

Najbitnija stavka pokretanja je zapravo na kraju skripte, inicijaliziranje kamere i definiranje postavki po glavnom / main streamu te sporednom / lower resolution (lores) streamu. Main stream se koristi za snimanje videa i snimanje fotografija dok je lores stream za live preview, odnosno streaming videa u browser. Rezolucija main streama je 1920x1080 px i slobodno se može mijenjati zajedno s kvalitetom kodiranja, veća zauzima više prostora na SD kartici, ali i više opterećuje Raspberry Pi. Lores stream je mnogo manje rezolucije također radi performansi, ne samo opterećenje Raspberryja nego i bandwidtha mreže.

Ako se prilikom konfiguracije dogodila greška ili je krivo spojen kabel, pojavit će se npr. greška:
```sh
[0:06:00.891324439] [918]  INFO Camera camera_manager.cpp:325 libcamera v0.3.2+27-7330f29b
list index out of range
```

Prekidom programa (Ctrl+C) vidjet će se cijeli stack greške, npr.:
```sh
^CTraceback (most recent call last):
  File "/home/pi/dashcam-pi/src/dashcam-pi.py", line 328, in <module>
    server.serve_forever()
  File "/usr/lib/python3.11/socketserver.py", line 233, in serve_forever
    ready = selector.select(poll_interval)
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/lib/python3.11/selectors.py", line 415, in select
    fd_event_list = self._selector.poll(timeout)
                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Kada je sve ispravno prošlo na pokretanju i konfiguraciji pojavit će se npr. ovakav output:
```sh
[0:00:39.474500400] [734]  INFO Camera camera_manager.cpp:325 libcamera v0.3.2+27-7330f29b
[0:00:39.646159775] [748]  WARN RPiSdn sdn.cpp:40 Using legacy SDN tuning - please consider moving SDN inside rpi.denoise
[0:00:39.651449827] [748]  INFO RPI vc4.cpp:447 Registered camera /base/soc/i2c0mux/i2c@1/imx708@1a to Unicam device /dev/media3 and ISP device /dev/media0
[0:00:39.651673473] [748]  INFO RPI pipeline_base.cpp:1126 Using configuration file '/usr/share/libcamera/pipeline/rpi/vc4/rpi_apps.yaml'
[0:00:39.671712952] [734]  INFO Camera camera.cpp:1197 configuring streams: (0) 1920x1080-XBGR8888 (1) 640x360-YUV420 (2) 2304x1296-SBGGR10_CSI2P
[0:00:39.672989931] [748]  INFO RPI vc4.cpp:622 Sensor: /base/soc/i2c0mux/i2c@1/imx708@1a - Selected sensor format: 2304x1296-SBGGR10_1X10 - Selected unicam format: 2304x1296-pBAA
```

Odmah se pokreće i web server na IP adresi **<YOUR_RASPBERRY_IP>** i portu **8000** pa se može iz browsera pristupiti interfaceu preko URL-a **http://<YOUR_RASPBERRY_IP>:8000** (vidi se u outputu izvršavanja skripte).

![alt](/images/browser_start.png)

Web server odmah poslužuje **index.html** kao template GUI-ja u kojem se tagovi unutar HTML-a zamjenjuju pathovima do pojedinih funkcionalnosti ili resursa. Svaki gumb je path do neke od funkcionalnosti, vidljivo u metodi **do_GET** klase **StreamingHandler**. Ista metoda dohvaća i pojedini resurs (vidljivo po parsiranju **self.path** i nazivima točnih fileova ili ekstenzija npr. html, css, png, mp4, jpg), tada se kao odgovor vraća sadržaj resursa. Malo specifičan resurs je **stream.mjpg** pošto se za njega dohvaćaju frameovi streaming pipelinea.

**/start_streaming** ako već nije pokrenut live preview pokreće snimanje na lores streamu kamere koji koristi nižu rezoluciju. Output je **StreamingOutput** čiji se frameovi prikazuju kao resurs **stream.mjpg**. Odmah bi se takav video trebao pojaviti na stranici u browseru. Ako se ugasi browser, streaming i dalje radi. Može se također posebno otvoriti u nekom od playera, npr. VLC-u kao network stream što je zanimljivo za korištenje projekta u druge svrhe. Način otvaranja u VLC-u:

![alt](/images/vlc_sream.png)

**/stop_streaming** ako je pokrenut live preview zaustavlja lores stream kamere, a **stream.mjpg** više nije vidljiv.

**/start_recording** pokreće snimanje videa iz glavnog / main streama kamere u file. Pošto je snimanje sam po sebi kontinuiran proces, ne smije se zadržavati web server pa se za snimanje u threadu poziva metoda **record_video**. Metoda **record_video** kreira video file u koji se sprema main stream kamere i izvršava se u petlji koju će prekinut flag za prestanak snimanja. Dok traje snimanje videa, nakon svakih 60 sekundi kreira se novi video file, želja je rascjepkati snimku, a ne imati jedan veliki file s cijelim video sessionom. Može se vidjeti kako se prilikom svakog novog videa snima i kreira posebna thumbnail jpg fotografija koja će se iskoristitiu galeriji snimljenog materijala.

**/stop_recording** postavlja flag prestanka snimanja videa i zaustavlja metodu **record_video** koja se izvršavala u threadu.

**/take_photo** pokrenut će snimanje fotografije iz glavnog streama i također spremiti thumbnail verziju fotografije radi prikaza u galeriji.

**/restart** zaustavlja sva snimanja i restarta Raspberry Pi.

**/turnoff** zaustavlja sva snimanja i sistemski gasi Raspberry Pi. Nakon toga ponovno se ručno mora pokrenuti Raspberry ako se želi koristiti.

**/gallery.html** poziva template **gallery.html** u kojem se tag sadržaja galerije puni listom snimljenog materijala iz **DCIM** direktorija. Za video i fotografije prikazuju se thumbnailovi spremljeni prilikom njihovog snimanja umjesto punih materijala. Za video je dalje moguće pokretanje, ali ovisno o network bandwidthu to može biti dosta zahtjevno.

Kako se inicijalno na pokretanju skripte snimanje ne pokreće automatski, slobodno možete otkomentirati linije kojima će se to automatski pokrenuti. Uz male izmjene može se automatski pokrenuti live preview, korisno ako će se dash kamera koristiti kao npr. video nadzor.

Uz nešto igranja s parametrima snimanja (ako se želi), dobro je dok je projekt ovako u analizi i testiranju lokalno na stolu pratiti i opterećenje Raspberry Pi-ja. 

Moj primjer opterećenja Raspberryja dok je pokrenut **dashcam-pi.py** izgleda ovako kad se koristi naredba **htop**:
```sh
htop
```

Pokrenut live preview i snimanje videa:<br>
![alt](/images/htop1.png)

Pokrenut samo live preview:<br>
![alt](/images/htop2.png)

Pokrenuto samo snimanje videa:<br>
![alt](/images/htop3.png)

Pokrenut samo interface bez snimanja:<br>
![alt](/images/htop4.png)

Ako sve i dalje izgleda ok, može nastavak slaganja projekta. :smiley:


## Automatsko pokretanje Python skripte i ubrzanje bootanja

Ručno pokretanje ima smisla samo tijekom razvoja i testa, sada je na redu dodavanje pokretanja Python skripte u boot proceduru Raspberryja. Za automatsko pokretanje prilikom bootanja potrebno je izmijeniti file **/etc/rc.local**:
```sh
sudo pico /etc/rc.local
```

Potrebno se pozicionirati prije linije **exit 0** i dodati sljedeće linije:
```sh
cd /home/pi/dashcam-pi/src
sudo nmcli dev wifi rescan
sudo nmcli dev wifi connect YOUR_MOBILE_WIFI_SSID &
sudo -u pi python dashcam-pi.py > log.txt 2>&1 &
```

Ovdje je dodatno vidljivo kako se nakon pozicioniranja u direktorij gdje se nalazi Python skripta prvo reskenira Wi-Fi i spaja na mobilni AP hotspot kao što sam naveo u prethodnim koracima tutoriala, također umjesto **YOUR_MOBILE_WIFI_SSID** postaviti naziv AP hotspota. Ukoliko se ne spoji na željeni, ostat će spojen na postojećem. Na kraju naredbe je **&** kako bi se pokrenula u pozadini i greške ne bi uzrokovale prekid daljnjeg izvršavanja skripte.

U mom slučaju file izgleda ovako:

![alt](/images/putty_rc_local.png)

Nakon spajanja se pod userom **pi** (ovo je obavezno) pokreće Python skripta **dashcam-pi.py**, output pokretanja skripte prosljeđuje se u log file **log.txt** kako bi se u njemu mogle vidjeti greške pokretanja, a na kraju cijelog reda obavezan je znak **&** kojim se tako pokrenut proces izvršava u backgroundu. Ako se ne želi logirati output u log file, tada ne treba u liniji ```> log.txt 2>&1```.

Nakon spremanja promjene filea **/etc/rc.local** potrebno je restartati Raspberry Pi kroz shell:
```sh
sudo shutdown -r now
```

Time će se ako je sve prošlo ispravno automatski pokrenuti Python skripta i može se pristupiti interfaceu dash kamere koji je podignut na IP adresi **<YOUR_RASPBERRY_IP>**. Ako je došlo do greške u skripti, ona je vidljiva u istom folderu u fileu **log.txt**.

Bitno je napomenuti ako se nakon uspješnog pokretanja ponovno spajamo Puttyjem na Raspberry Pi te mijenjamo skriptu potrebno je prije ručnog pokretanja nove verzije kill-ati postojeći proces. Provjera pokrenute skripte kroz shell:
```sh
ps -fu pi | grep dashcam-pi
```

Prema rezultatu pronaći ```<pid>``` aktivnog procesa te kroz shell pozvati ```kill <pid>``` tog procesa.

Također je želja što prije pokrenuti **dashcam-pi** kako bi se moglo pristupiti dash kameri pa treba dodatno ubrzati bootanje Raspberryja. Prvo je prilikom setupa kroz raspi-config gašen GUI i ostavljena text konzola, sada gasim nivo vidljivih log poruka na bootanju promjenom filea **/boot/firmware/cmdline.txt** (ako je prazan onda treba promijeniti **/boot/cmdline.txt** koji se koristi u starijim verzijama OS-a):
```sh
sudo pico /boot/firmware/cmdline.txt
```

File ima samo jednu liniju pa na kraju te linije mora se dodati space nakon kojeg ide ```quiet splash loglevel=0``` te na kraju linije još jedan space.

Dodatno ubrzanje može se dobiti promjenom filea **/boot/firmware/config.txt** (ako je prazan onda **/boot/config.txt** koji se koristi u starijim verzijama OS-a):
```sh
sudo pico /boot/firmware/config.txt
```

Jedinu izmjenu napravio sam na sljedećoj liniji kako bi ugasio audio koji ne koristim:
```sh
dtparam=audio=off
```

Nakon promjene fileova ponovno restartati Raspberry Pi kroz shell.

Mjerenje trajanja bootanja Raspberry Pi-ja osim ručno može se provjeriti i koristeći shell naredbu:
```sh
systemd-analyze
```

Rezultat može biti sličan mom primjeru gdje se vidi trajanje pojedinih segmenata:
```sh
Startup finished in 4.681s (kernel) + 13.620s (userspace) = 18.301s
multi-user.target reached after 13.551s in userspace.
```

Što je manje aktivnih servisa u startu prije će se pokrenuti Python skripta interfacea dash kamere. Mjerenjem od trenutka paljenja Raspberryja pa do prikazivanja index.html u browseru za moj primjer trebalo je oko 35 sekundi.


## Projekt u živom radu i usporedba s komercijalnim rješenjem

Do sada se moglo primijetiti kako je priprema najvažniji dio projekta. Python skripta upravlja radom dash kamere koja je sklopljena u vlastito kućište, a još kada se postavi na nosač za vjetrobransko staklo, može izgledati ovako:

![alt](/images/dashcam_incar1.png)

![alt](/images/dashcam_incar2.png)

Spajanjem dash kamere na auto USB čim počne raditi spaja se na AP hotspot mobitela, dalje se treba spojiti putem mobitela na dodijeljenu IP adresu i u browseru će se pojaviti interface za upravljanje:

![alt](/images/browser_stream.png)
![alt](/images/browser_gallery.png)

Dalje se može koristiti live preview koji je koristan za kadriranja snimke. I najbitnije, kad se želi može se započeti snimati video vožnje. :smiley:

Primjer mog testnog snimljenog materijala: video, fotografija i njihovi thumbnailovi mogu se vidjeti u direktoriju [**/DCIM**](/DCIM/) projekta.

Kako na tržištu postoji puno komercijalnih rješenja od kojih su neka jeftinija, a neka mnogo skuplja, probao sam usporediti moju Raspberry Pi dash kameru s kamerom Garmin Dash Cam Mini 2. Uživo ovako izgledaju zajedno:

![alt](/images/dashcam_garmin.png)

Video s usporednim snimkama koje sam napravio koristeći obje kamere objavio sam i na svom YouTube kanalu:

[![dashcam-pi Raspberry Pi vs Garmin Dash Cam Mini 2](https://img.youtube.com/vi/xP-zit23gdM/0.jpg)](https://youtu.be/xP-zit23gdM)

Još kada se neke tehničke karakteristike stave u tablicu:

|| **dashcam-pi na korištenim komponentama** | **Garmin Dash Cam Mini 2** |
|---|---|---|
| senzor kamere | 12 MP, Sony IMX708 | 2 MP, Garmin Clarity HDR optics |
| korištena rezolucija | 1080p | 1080p |
| kut snimanja kamere | 120&deg; | 140&deg; |
| fps | 30 | 30 |
| aplikacija | dashcam-pi | Garmin Drive |
| dimenzije kućišta | ŠxVxD: 3.60 cm x 7.70 cm x 2.50 cm | ŠxVxD: 3.13 cm x 5.33 cm x 2.91 cm |
| vrijeme bootanja | oko 35 s | 9 - 18 s |
| potrošnja na sat | 400 mAh | 340 mAh |

Osobni dojam mi je dvojak. Jako sam zadovoljan kako sam izveo projekt, pogotovo softverski jer aplikacijom koju sam razvio lako je upravljati kamerom, lakše nego Garminovom aplikacijom, međutim hardverski Raspberry Pi Camera Module 3 Wide kaska za Garminom. Usporedba snimki pokazuje razliku u kvaliteti optike i senzora između Raspberry Pi Camera Module 3 Wide i Garminovog senzora u dash kameri. Ako se uzme u obzir cijena Raspberry Pi Camera modula, to je još prihvatljivo i ovo je sigurno korisno za osobne, kućne potrebe. Kao varijacija projekta može se probati s Raspberry Pi High Quality Camera, nisam je do sada isprobao, ali to bi ostavio za neku iduću samogradnju.


## Reference

- [Moj Github Raspberry Pi projekt raspi-gps](https://github.com/dgolner/raspi-gps)
- [Headless Raspberry Pi Setup](https://learn.sparkfun.com/tutorials/headless-raspberry-pi-setup/wifi-with-dhcp)
- [Raspberry Pi Configuration](https://www.raspberrypi.com/documentation/computers/configuration.html)
- [Raspberry Pi Camera Modules](https://www.raspberrypi.com/documentation/accessories/camera.html)
- [The Picamera2 Library](https://datasheets.raspberrypi.com/camera/picamera2-manual.pdf)
- [Raspberry Pi Picamera2 examples](https://github.com/raspberrypi/picamera2/tree/main/examples)
- [Samba: Set up a Raspberry Pi as a File Server for your local network](https://magpi.raspberrypi.com/articles/samba-file-server)
- [Make a Pi Zero W Smart USB flash drive](https://magpi.raspberrypi.com/articles/pi-zero-w-smart-usb-flash-drive)
- [Host a Wi-Fi hotspot with a Raspberry Pi](https://www.raspberrypi.com/tutorials/host-a-hotel-wifi-hotspot/)

<!----------------------------------------------------------------------------------------------------------------->

[def-dashcam-pi.py]: /src/dashcam-pi.py

[Download Icon]: https://img.shields.io/badge/Download-37a779?style=for-the-badge&logoColor=white&logo=DocuSign
