# h3 Demoni

Aloitin raportin ja tehtävien tekemisen klo 18.30.

## x) Tiivistelmä

- Demonit, esim Apache2, konfiguroidaan package-file-service-mallilla 

// package - sudo apt-get install apache2 
// files - sudoedit /etc/apache2/... 
// service - sudo systemctl restart apache2 

- Tehdyt muutokset tulevat voimaan vasta demonin uudelleen käynnistyksen jälkeen _sudo systemctl restart ..._ 

- Ansiblen handler on tehtävä, joka suoritetaan vain, kun se saa toiselta tehtävältä ilmoituksen (notify) suorittaa. Handleria käytetää usein palvelun uudelleen käynnistykseen. Esim. Playbookin tehtävä ilmoittaa handlerille muutoksesta (notify) "muutoksia tehty" > handler käsittelee ilmoituksen ja tekee määritetyn tehtävän "käynnistyy uudelleen". 

- Playbookiin määritetyt useat saman handlerin kutsut käsitellään määritysjärjestyksessä, eli siinä kohtaa missä ne on kirjoitettu ja handlerin tehtävä suoritetaan vain kerran 

- Handlerilla voidaan tehdä kerralla useita muutoksia ja lopuksi käynnistää palvelu uudestaan vain kertaalleen, jolloin vältytään ylimääräisiltä keskeytyksiltä   

## a) Apassi. Apache2 asennus ja etusivu palvelimelle

Asensin virtuaalikoneelleni Apachen weppipalvelimen. Päivitin ensin pakettilistan ja asennukset tein sudona. Apachen enablointi tehty heti asennuksen perään. 
```bash
sudo apt-get update
sudo apt-get install apache2
sudo systemctl enable --now apache2

```
Tein **/etc/apache2/sites-available$** sijaintiin web-sivustolle konfitiedoston komennolla:

```
sudoedit ragewizard.example.conf
```
```
<VirtualHost *:80>
    
    ServerName ragewizard.example.com
    ServerAlias www.ragewizard.example.com
    DocumentRoot "/home/rage/publicsite/"

    <Directory "/home/rage/publicsite/">
	    require all granted
    </Directory>
</VirtualHost>
```
<img width="814" height="313" alt="1" src="https://github.com/user-attachments/assets/c21d25e9-1fac-4c22-91e3-09c9fe15704a" />

Enabloitu äsken tehty konfitiedosto
```
sudo a2ensite ragewizard.example.conf 
```
Tein käyttäjälle "rage" etusivua varten kansion:
```
mkdir /home/rage/publicsite
```
Tein publicsite-kansioon etusivutiedoston
```
sudo micro index.html
```
<img width="587" height="264" alt="3" src="https://github.com/user-attachments/assets/e16df8ad-5ef1-4413-b109-f04f783d57a4" />


Annoin suoritusoikeudet käyttäjän "rage" hakemistoon.
```
chmod ugo+x /home/rage
```

Potkaistu demoni käyntiin uusilla asetuksilla.
```
sudo systemctl reload apache2
```
Etusivu toimi hienosti local hostissa ja sitä pystyi muokkaamaan ilman sudoa.

<img width="745" height="496" alt="4" src="https://github.com/user-attachments/assets/614b6535-b41d-436f-a620-832fc4520c88" />



## b) Moottorix. Nginx asennus ja etusivu palvelimelle

Aluksi sammutin äsken pystyyn laittamani Apachen palvelimen:
```
sudo systemctl stop apache2
```

Disabloin vielä Apachen, ettei se ainakaan tuota porttien (80) kanssa ongelmaa Nginxillä.
```
sudo systemctl disable apache2
```
Asensin Ngixin sudona
```
sudo apt-get install -y nginx
```
Asennus onnistui muutamissa sekunneissa. Käynnistin demonin ja tarkistin sen tilan.
```
sudo systemctl start nginx
sudo systemctl status nginx
```
Nginx käynnistyi ongelmitta, mutta etusivuna pyöri vielä aiemmin määrittämäni etusivu, joka oli tässä tapauksessa väärä.

<img width="1185" height="651" alt="5" src="https://github.com/user-attachments/assets/5ce7837d-c64b-44e2-8868-bcc2ac996da0" />

Hain virhettä komennolla **sudo nginx -t**. joka näytti, että konfit ovat ok. 
<img width="741" height="97" alt="6" src="https://github.com/user-attachments/assets/906114fb-5507-4fee-81c6-5d0980840f89" />

Nginx luki konfitiedostoa /etc alta ja /var hakemistosta tässä vaihessa, joten loin sille uuden konfitiedoston Apachen tyyliin polkuun **/etc/nginx/sites-available**. 

Muokkasin Nginxin default-konfiin root-hakemistoksi käyttäjän kotihakemiston, josta sivuston muut asetukset tulevat ja sitä pystytään sieltä käsin muokkaamaan.
<img width="733" height="203" alt="7" src="https://github.com/user-attachments/assets/42f27cad-386f-4f87-9395-af2193d579e7" />

Ajoin vielä testiksi komennon **sudo nginx -t** tsekatakseni palvelun tilan, ja kunnossa oli. 
<img width="686" height="67" alt="8" src="https://github.com/user-attachments/assets/1e340f9f-8e2b-42e3-a33d-05a7ffab8afa" />

Potkaisin Nginxia vielä kertaalleen
```
sudo systemctl restart nginx
```
Testasin muokata aiemmin tehtyä index.html-tiedostoa käyttäjänä polussa **/home/rage/publicsite** ja etusivu toimi webissä odotetulla tavalla.

<img width="562" height="354" alt="9" src="https://github.com/user-attachments/assets/a433296d-fd3b-4a64-8b95-c50e1e3cec09" />

Tarkistin etusivun index-hakemiston käyttöoikeudet komennolla **stat publicsite**; tiedoston omistaa käyttäjä (rage) ja othersilla (Nginx) on suoritusoikeudet. 
<img width="741" height="198" alt="10" src="https://github.com/user-attachments/assets/ee24348e-5185-455a-8353-554e2020bde8" />

## c) Automoottorix. Nginxin asennuksen automatiointi Ansiblella

Sitten itse automatisointiin. Loin aluksi uuden kansion **mkdir ansibleauto** käyttäjän kotihakemistoon, johon aloin rakentaa uutta pelikirjaa. 

Kopioin aiemman ansible.cfg -tiedoston uuteen hakemistoon komennolla
```
cp ansible.cfg /home/rage/ansibleauto/
```
Kopioin samaan tapaan myös hosts.ini -tiedoston uuteen pelikirjan hakemistoon.

<img width="482" height="131" alt="12" src="https://github.com/user-attachments/assets/3414f3a4-febd-482e-9540-533db50cfc54" />

Tein rooli-kansion ja uudelle tehtävälle **nginxauto** task-kansion.
```
mkdir -p roles/nginxauto/tasks/
```
<img width="451" height="202" alt="11" src="https://github.com/user-attachments/assets/78814049-9659-4983-a341-d11e75cefb2f" />

Tein task-kansioon main.yml -tiedoston, johon määritin Karvisen materiaalista vinkkiä ottamalla asetukset eli itse Ansiblen koodin. (Karvinen.T 2026).
```
- apt:
    name: nginx
    state: present

- copy:
    dest: "/etc/nginx/sites-available/ragewizard.example.conf"
    src: "ragewizard.example.conf"
    owner: "root"
    group: "root"
    mode: "0644"
  notify: restart nginx

- file:
    src: /etc/nginx/sites-available/ragewizard.example.conf
    dest: /etc/nginx/sites-enabled/ragewizard.example.conf
    owner: root
    group: root
    state: link
  notify: restart nginx
```
<img width="708" height="384" alt="19" src="https://github.com/user-attachments/assets/c866c056-af3b-4db2-af58-c1b7869dc413" />


Handlereille tein oman kansion **mkdir handlers**, johon määritin handlerin tehtävän käynnistää nginx uudestaan.

<img width="674" height="118" alt="14" src="https://github.com/user-attachments/assets/6e35b9a9-77f8-40f4-bfc3-5ed2518682bc" />

Loin tehtävälle kopioitavia tietoja varten **files** kansion ja sinne määritin conf-tiedoston, 

<img width="848" height="261" alt="15" src="https://github.com/user-attachments/assets/8296b833-55f5-4676-8079-25a8f540c677" />

Lopuksi tein site.yml tiedoston playbookille, johon määritin tämän tehtävän roolin **nginxauto**

<img width="533" height="103" alt="16" src="https://github.com/user-attachments/assets/553da14d-25e0-4c06-9712-5fb8662e1ab1" />

Ajoin playbookin 
```
ansible-playbook site.yml
```
ja sain virhettä destinationista "not writable". 

<img width="1182" height="318" alt="17" src="https://github.com/user-attachments/assets/b838e061-e7ad-4fcf-81b6-dd8ead50782d" />

Tutkin aiempaa Karvisen materiaalia Ansiblen asennuksesta, ja site.yml tarvitsi vielä root-oikeudet tehtävänajoon, eli lisäsin site.yml -tiedostoon **become: true**, jolla määritetään, että tehtävä ajetaan pääkäyttäjäoikeuksin.

<img width="436" height="128" alt="18" src="https://github.com/user-attachments/assets/3eb89c93-366f-4661-b5cc-64ee90555e72" />

Ajoin playbookin uudestaan sudona
```
ansible-playbook site.yml --ask-become-pass
```
ja sain uuden virheen. 

<img width="1551" height="661" alt="20" src="https://github.com/user-attachments/assets/da825cad-6ce3-45c5-a00a-a14cf8fb10eb" />

Selvittelin Nginx tilaa komennolla **sudo nginx -t** ja nyt itse Nginx antoi herjaa.

<img width="1410" height="126" alt="21" src="https://github.com/user-attachments/assets/040a0a6c-8ecf-4b3f-a610-2fb736d17834" />

Nyt ei myöskään etusivuni toiminut testatessa, vaan antoi virheen "Unable to connect".

Kokeilin **curl localhost**, ja sain valitusta portista. 

<img width="918" height="65" alt="22" src="https://github.com/user-attachments/assets/0dab7992-dd7e-479c-8bef-f53633e3006d" />

Huomasin, että minulla oli nyt kaksi Nginxin konfitiedostoa **sites-enablessa**. Virhe 1. loin erheessä tälle uudelle Nginx-tehtävälle Apachen konfitiedoston. Korjasin sen kopioimalla Nginxin default (joka siis toimi aiemmin) konfit > **ragewizard.example.conf** -tiedostoon - koko polku /etc/nginx/sites-enabled/ragewizard.example.conf. 

Ajoin playbookin uudestaan nyt oikeilla konfeilla ja yksi muutos tuli onnistuneesti, mutta tehtävä päätyi virheeseen.

<img width="1545" height="582" alt="23" src="https://github.com/user-attachments/assets/0adefc32-2b6e-4240-98d9-808a32bb8da3" />

Ajoin **sudo nginx -t** ja sain saman virheen mitä aiemminkin.
<img width="1401" height="73" alt="24" src="https://github.com/user-attachments/assets/dbf9db86-7e67-4404-8487-246df4c56dae" />

Tarkistin **/etc/nginx/sites-enabled/ragewizard.example.conf** tiedoston, joka oli muuttunut samaksi vääränlaiseksi Apachen konfiksi, mikä aiemmin oli ongelmana. Playbook siis ajanut konfien yli. Avasin konfitiedoston playbookista **/ansibleauto/roles/nginxauto/files/ragewizard.example.conf**  ja korjasin Nginx -konfit suoraan tänne tiedostoon.

<img width="789" height="376" alt="25" src="https://github.com/user-attachments/assets/0f553bb9-19b8-4d84-b135-c0d200fcf3d7" />

Testasin potkaista Nginxiä, mutta päätyi virheeseen. Tarkistin Nginxin tilan **sudo systemctl restart nginx.service**

<img width="1558" height="410" alt="26" src="https://github.com/user-attachments/assets/d9a11c2c-1e69-4ace-ace7-8812495ca592" />

Nginxin Default-konfit tekivät tässä kohtaa temput > poistin **default** konfin sijainnista **etc/nginx/sites-enabled**. 

Ajoin playbookin 
```
ansible-playbook site.yml --ask-become-pass

```

Ja nyt Nginx käynnistyi palybookilla ja etusivu oli käytettävissä - 2 muutosta tapahtui (kuvassa). Homma toimii siis! 

<img width="1552" height="489" alt="27" src="https://github.com/user-attachments/assets/bbd5f241-a713-4556-815c-f3f2fd604aa0" />

<img width="618" height="374" alt="28" src="https://github.com/user-attachments/assets/ccaf675b-2db3-45cc-bfb0-fed5d2b585c6" />

Raportin kirjoittamisen lopetin 22.10.

## Lähteet
- Karvinen Tero. Ansiblen asennus ja ongelmanselvitys. Luettavissa: https://terokarvinen.com/hello-ansible/
- Karvinen Tero. Tehtävät. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/#h3-demoni

## Tekijätiedot 

- Miika Vänttinen, Haaga-Helia Ammattikorkeakoulu, tietojenkäsittely 
- Tätä dokumenttia saa kopioida ja muokata GNU General Public License (versio 2 tai uudempi) mukaisesti. http://www.gnu.org/licenses/gpl.html 
- Pohjana Tero Karvinen 2026: Palvelinten hallinnan kurssi, http://terokarvinen.com 

