# h2 Voileipä

Aloitin raportin kirjoittamisen klo 11.50. Rautana tässä ja tulevissa tehtävissä toimii Windows 11 PC. Tarkemmat host specsit:

- OS: Microsoft Windows 11 Education
- System Type: x64-based PC
- Total Physical Memory: 16 292 MB
- Processor(s): Intel64 Family 6 Model 165 Stepping 2 GenuineIntel ~2592 Mhz

Virtuaalikone pyörii Oraclen Virtualboxissa, johon on asennettu Debian 13 (64-bit). Rakennan kurssin palvelinympäristöäni kyseisellä virtuaalikoneella.

## x) Tiivistelmä materiaalista 

- Ansible tarvitsee pääkäyttäjän oikeudet orjakoneelle 

- Sudon käyttäminen ilman salasanaa tarvitsee määrityksen sudoers-tiedostoon Visudolla: %sudoless ALL = (ALL) NOPASSWD: ALL 

- Tehdessä muutoksia konfiguraatioihin, muista aina testatata toimivuus. Mitä ei ole testattu, ei ole todistetusti tehty 

- T.Karvisen ykkössääntö IaC:ssa: manuaalinen ennen automaatiota. Paras käytäntö testata ja varmistaa tehtävien toimivuus, ennen automatisointia 

**Ansible doc tiivistelmät** 

- ansible-doc copy - käytetään tiedostojen kopioimiseen tai luomiseen kohdekoneille. Esimerkki:  
```yaml
- name: kopioi 
  ansible.builtin.copy:   
    Src: /home/miika/tiedosto.txt   
    dest: /home/antero/tiedosto2.txt 
    Owner: antero 
    Group: antero
```` 
  
- ansible-doc apt – paketinhallinta. Näyttää ohjeet, miten asennuksia, päivityksiä ja poistoja voi hallita. Esim.  poista apache2: 

```yaml
- name: poista Apache
  ansible.builtin.apt: 
    Name: apache2 
    State: absent 
````
- ansible-doc file - käytetään tiedostojen ominaisuuksien hallintaan. Voidaan mm. Muokata, linkittää, luoda ja poistaa. Esim. Tiedoston omistajan vaihto 

```yaml
- name: vaihda omistajaa 
  ansible.builtin.file: 
    Path: /home/miika/testitiedosto.txt 
    Owner: antero 
    Group: antero 
    Mode: '0644' 
````

- ansible-doc user - käyttäjätilien hallinta; luominen, muokkaaminen ja poisto. Esim. Salasanan vanhenemisen ilmoitus: 

```yaml
- name: Salasana vanhenee 30 pvä kuluttua 
  ansible.builtin.user: 
    Name: antero 
    Password_expire_warn: 30 
````

- ansible-doc authorized_key -  käytetään SSH-avainten hallintaan; lisää ja poista. Esim. Tiedostosta haettu julkinen avain: 
````yaml
- name: aseta avain käyttäjälle 
  ansible.posix.authorized_key: 
    User: antero 
    State: present 
    Key: "{{ lookup('file', '/home/antero/.ssh/id_rsa.pub') }}" 
````


## a) Sudoless. Ansiblelle tunnus, jolla voi käyttää sudoa ilman salasanaa ja SSH:n automatisointi

Loin Ansiblea varten uuden käyttäjän sekä uuden sudoless-ryhmän. Lisäsin luodun käyttäjän sudoless-ryhmään. 

````
$ sudo adduser miikaans 
$ sudo groupadd sudoless  
$ sudo adduser miikaans sudoless 
````
<img width="545" height="325" alt="1" src="https://github.com/user-attachments/assets/573ca90b-8edf-4cd6-abea-f1e3d1abd50e" />


Seuraavaksi loin säännön visudo-työkalulla, jota käytetään Linux-järjestelmän ylläpidossa. Tarkoituksena on sallia äsken luodun "sudoless"-ryhmän käyttää sudoa ilman salasanaa. Avasin visudon komennolla _$ sudo visudo /etc/sudoers.d/sudoless_, johon lisäsin salasanattoman kirjautumisen määrityksen _%sudoless ALL = (ALL) NOPASSWD: ALL_. 

<img width="801" height="170" alt="2" src="https://github.com/user-attachments/assets/91fef04f-f046-41ab-88e1-92103b2f7222" />

Nyt kun käyttäjälle oli määritetty salasanaton sudon käyttö, testasin sen kirjautumalla SSH:lla käyttäjälle ja ajamalla testin _$ sudo echo "Tervemoi! Pääsiäisenä syödään mämmiä!"_. Salasanaton sudon käyttäminen onnistuu nyt ongelmitta määritetyillä sudoless-ryhmän käyttäjillä. 

<img width="954" height="270" alt="3" src="https://github.com/user-attachments/assets/f00ab8f5-a114-4ac0-996b-6a14599aca4d" />


Automatisoin SSH-kirjautumisen julkisella avaimella kopioimalla tunnistetiedot käyttäjälle "miikaans".  

<img width="1192" height="250" alt="4" src="https://github.com/user-attachments/assets/e89697c7-6ad4-4605-bfe1-6de0fdd46a3a" />

SSH-kirjautuminen toimii nyt ilman salasanaa ja käyttäen julkista avainta. 

<img width="943" height="232" alt="5" src="https://github.com/user-attachments/assets/0f5085ae-9e5e-4807-8f2d-e0441b117154" />


## b) Antero. Salasanaton, automaattisesti SSH:lla kirjautuva tunnus Ansiblella

Loin uuden roolin tehtävää varten aiemmin luotuun ansible-hakemistoon. 

<img width="682" height="362" alt="6" src="https://github.com/user-attachments/assets/3f762fd0-a7f4-4227-a688-b2f7ae83cc6e" />

Tulostin julkisen avaimeni uuden tehtävän main.yml koodiin komennolla _cat .ssh/id_ed25519.pub >> ansible/roles/newuser/tasks/main.yml_. 

<img width="797" height="67" alt="7" src="https://github.com/user-attachments/assets/da37a11d-9ad7-407c-89c9-20426e000576" />

Tein main.yml tiedostoon koodia Karvisen materiaalin ohjeistamana (Karvinen 2026). Koodiin määritin automaation: palvelimelle kirjautuminen käyttäjänä "antero" käyttäen SSH-avainta ja ilman sudo-salasanan kirjoittamisen tarvetta.  

<img width="1053" height="406" alt="8" src="https://github.com/user-attachments/assets/3e332017-96b7-48dc-af8d-90676f7fe1ff" />

Lisäsin äsken tehdyn roolin "newuser" site.yml tiedostoon, johon myös määritin _become: true_, jolla kerrotaan Ansiblelle, että tiedoston tehtävät tulee suorittaa pääkäyttäjäoikeuksilla. Playbookia ajaessa komennolla _ansible-playbook site.yml_ sain vielä virheen puuttuvasta sudo-salasanasta. 

<img width="1271" height="349" alt="10" src="https://github.com/user-attachments/assets/284a55dd-3913-4183-bb45-10412ef2cdaa" />

Ajoin playbookin komennolla _ansible-playbook site.yml -K_, jolla testasin manuaalisesti, että homma toimii. Playbook kysyi nyt sudo-salasanan ja toimitti tehtävänsä ongelmitta.  

<img width="1263" height="595" alt="11" src="https://github.com/user-attachments/assets/8b7d87ec-93a7-4aeb-bfc9-bdf7f700714a" />

Playbookin ajon jälkeen testasin, että rooli varmasti teki tehtävän oikein: kirjauduin playbookilla luodulla käyttäjällä "antero" SSH-yhteyteen ja kirjautuminen onnistui ilman sudo-salasanaa, kuten pitikin. 

<img width="975" height="307" alt="12" src="https://github.com/user-attachments/assets/b9a241a6-20de-48ce-a656-c5928177e700" />

Playbookin käyttöoikeudet:
<img width="761" height="209" alt="13" src="https://github.com/user-attachments/assets/e53ba367-0f8a-4afa-87db-f15d4b440920" />


## c) Package. Pakettien asennus Ansiblella

Loin aluksi ansiblen kansioon uuden roolin _packages_ ja alikansion _tasks_ sekä main.yml-tiedoston. 

<img width="604" height="251" alt="14" src="https://github.com/user-attachments/assets/ef7a3460-a971-44bd-b8fe-ee1797d0106d" />

Lisäsin package-roolin playbookiin site.yml, jotta tehtävä toimii. Ajoin playbookin ja Git asentui nätisti, mutta ntpdate-pakettia ei löytynyt, joten siitä herjasi virhettä. Playbook sinänsä toimi oikein.  

<img width="1259" height="723" alt="16" src="https://github.com/user-attachments/assets/ceca3112-46de-41ed-ba55-0beaebc47c4f" />

Vaihdoin ntpdate ohjelman asennuksen curl-työkalun asennukseen ja lisäsin sen packages-roolin main.yml tiedostoon. Ajoin playbookin uudestaan ja _curl_  työkalun asennus toimi playbookista ongelmitta, tosin se näytti olevan jo asennettuna koneelleni, josta johtuen mikään ei muuttunut tehtävänajossa. 

<img width="1263" height="696" alt="17" src="https://github.com/user-attachments/assets/751e56c5-8bc5-407f-9906-9bb2134f9045" />

Pidin luovan tauon tässä kohtaa klo 14.30. 

## d) File. Useamman rivin tiedosto orjalle Ansiblella

Jatkoin raporttia klo 15.15. Tein tälle tehtävälle uuden roolin ansible-hakemistoon _mkdir –p roles/file/tasks_. Avasin tekstieditorin, jolla loin main.yml tiedoston. Koodin sisältöön otin vinkkiä Karvisen tehtävästä, jossa käsiteltiin salasanatonta kirjautumista Ansiblella (Karvinen 2026). Määritin koodiin tehtävän ja oikeudet, alla kuvassa. 

<img width="628" height="293" alt="18" src="https://github.com/user-attachments/assets/c1d8697f-3c77-4b73-b2da-6189f19fe637" />

Kävin lisäämässä roolin _file_ site.yml määrityksiin. Ajoin playbookin, ja törmäsin virheeseen, ettei kansio kelpaa destinationiksi.

<img width="1265" height="355" alt="19" src="https://github.com/user-attachments/assets/4cd261eb-6897-4b7c-a578-02d7ec9f754a" />

Avasin file-roolin main.yml-tiedoston, johon muokkasin destinationin txt-päätteiseksi "hello.txt". 

<img width="558" height="80" alt="20" src="https://github.com/user-attachments/assets/8fe6fce7-b47c-4a1f-9ed5-3732c4577f52" />

Ajoin playbookin ja sain jälleen virheen, tällä kertaa groupista. Sudoers-ryhmää ei siis pystytty tässä käsittelemään, en ole ihan varma kirjoitinko tuon peräti väärin.  

<img width="1249" height="403" alt="21" src="https://github.com/user-attachments/assets/d9b04f00-0871-4110-a2a3-cf15949f5bcc" />

Vaihdoin kuitenkin main.yml konfeihin groupiksi: antero > tallensin ja ajoin playbookin uudestaan. Testasin vielä, että tehtävä meni varmasti maaliin halutulla tavalla ja kirjauduin SSH:lla antero-käyttäjälle. Hello.txt-tiedosto löyty halutusta sijainnista ja nyt playbook toimi hyvin. 

<img width="947" height="382" alt="22" src="https://github.com/user-attachments/assets/84f898be-44cb-40df-927e-956bb7fb254a" />

Käyttöoikeuksien tarkistus komennolla _stat_.  

- 6 omistaja - rw – saa lukea ja kirjoittaa 
- 6 ryhmä - rw - saa lukea ja kirjoittaa 
- 0 muut – "---" ei oikeuksia 

<img width="762" height="184" alt="23" src="https://github.com/user-attachments/assets/8e5da873-7dcb-460e-85e5-348c08445f5b" />

## e) Jotain muuta. Esimerkki Ansiblen käskystä

Hain lisätietoa ansible docista koskien palomuuria komennolla _ansible-doc –list | grep ufw_. Sain vastaukseksi community.general.ufw ja hain tarkemmat tiedot komennolla _ansible-doc community.general.ufw_. Tämän ohjeistuksen avulla pystytään säätämään palomuuria.  

<img width="888" height="528" alt="24" src="https://github.com/user-attachments/assets/87f4d7a1-fde8-4bc5-b1f8-08a828fc5f8b" />

Esimerkiksi OpenSSH-yhteyden salliminen palomuurista komennolla: 

```yaml
- community.general.ufw: 
    Rule: allow 
    Name: OpenSSH 
````

Ja säännön poistaminen komennolla: 
```yaml
- name: Delete OpenSSH rule 
  Community.general.ufw: 
    Rule: allow 
    Name: OpenSSH 
    Delete: true 
````
Lopetin raportin kirjoittamisen ja testit klo 18.05. Pitkä tovi tähän vierähti ja rauhassa tutkiskelin, ja hyvää oppia Ansiblen käytöstä tarttui matkaan.

## Lähteet 

- Ansible community documentation. Pakettien asennus Ansiblella. Luettavissa: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/package_module.html 
- Karvinen Tero, 2026. Tehtävät. https://terokarvinen.com/palvelinten-hallinta/#laksyt
- Karvinen Tero, 2026. Salasanaton Ansible. Luettavissa: https://terokarvinen.com/passwordless-sudo-with-ansible/ 
- Stackoverflow. Ansible.builtin.copy komennosta. Luettavissa: https://stackoverflow.com/questions/79372409/how-to-use-ansible-builtin-copy-with-content-and-ensure-resulting-file-ends-with 

## Tekijätiedot 

- Miika Vänttinen, Haaga-Helia Ammattikorkeakoulu, tietojenkäsittely 
- Tätä dokumenttia saa kopioida ja muokata GNU General Public License (versio 2 tai uudempi) mukaisesti. http://www.gnu.org/licenses/gpl.html 
- Pohjana Tero Karvinen 2026: Palvelinten hallinnan kurssi, http://terokarvinen.com 

 
