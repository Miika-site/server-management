# h1 Hei Ansiblen maailma

## x) Tiivistelmä

- SSH:tä käytetään turvallisien yhteyksien luomiseen palvelimille 

- Julkisen avaimen autentikointia käyttämällä poistetaan tarve syöttää salasanaa palvelinten halllintaa tehdessä, kuten ajaessa ansible-playbookia 

- Ansiblella voidaan kirjoittaa infrastruktuuri koodina (IaC) ja hallita kätevästi useita koneita. 

- Ansible on hallintatyökalu, jolla voidaan määrittää ja kuvata infra, sekä tehdä siihen muutoksia tarvittaessa työkalun avulla. 

- Ansible toimii SSH:n kautta 

- Ansiblea käyttävälle tilille on hyvä määrittää salasanaton SSH autentikointi hallinnan helpottamiseksi, jolloin jatkuvaa salasanan syöttämistä ei tarvita työkalua käyttäessä

## SSH-demonin asentaminen ja testaus

Aloitin raportin kirjoittamisen klo 17.10.  

 

Kirjauduin Debian 13 -pohjaiselle virtuaalikoneelleni ja ajoin aluksi komennon _sudo apt-get update_. Asensin SSH-demonin onnistuneesti komennolla _sudo apt-get install ssh_.  

 
<img width="646" height="165" alt="2" src="https://github.com/user-attachments/assets/dde05d8a-bcab-4cf3-8417-448557b736c0" />

 
Loin uuden SSH-avaimen komennolla _ssh-keygen_. Kopioin avaimen local hostille komennolla _ssh-copy-id localhost_. Enabloitu SSH komennolla _sudo systemctl enable –now ssh_. 

Testasin SSH-yhteyttä, ja sain virheen avaimesta, enkä päässyt yhteyteen. Olin viime luennolla luonut jo avaimen sekä muokkaillut koneen asetuksia ja siksi sain nyt virhettä tunnistetiedoista.  

<img width="1179" height="505" alt="4" src="https://github.com/user-attachments/assets/7a74da6e-81ba-4566-ae79-83fefbc94a74" />

Poistin vanhan tunnistetiedon koneelta komennolla _ssh-keygen –f /home/rage/.ssh/known_hosts –R localhost_. Tämän jälkeen kopioin tuoreen avaimen localhostille komennolla _ssh-copy-id localhost_. Avasin SSH-yhteyden onnistuneesti locahostiin. 

<img width="972" height="324" alt="5" src="https://github.com/user-attachments/assets/c3d4eb8b-88f2-4cd1-a795-d15531bd83b4" />


## b) Public key - SSH-kirjautumisen automatisointi julkisella avaimella

Tämä tehtävä tulikin jo tehtyä tehtävässä a), koska jouduin päivittämään koneen tunnistetiedon. Kirjautuminen SSH:lla localhostiin käyttäen julkista avainta on nyt automatisoitu. 

<img width="952" height="320" alt="7" src="https://github.com/user-attachments/assets/a3f567c0-4deb-4b3a-908b-655a9c099458" />


## c) Hei Ansible - Ansible-työkalun asennus, määritykset ja sen testaus SSH:lla 

Nyt kun olin jo testannut, että SSH-yhteys toimii, asensin hallintaohjelma Ansiblen ja muutaman muun tarpeellisen ohjelman komennolla _sudo apt-get install ansible micro bash-completion tree_. Kuvassa ei ole uusia asennuksia, koska olin nämä ohjelmat jo kerennyt asentaa luennolla. Asennuksissa kestää max. Muutama minuutti. 

<img width="757" height="233" alt="8" src="https://github.com/user-attachments/assets/d91fbb63-9bbb-4470-9f28-7986d68ac6cf" />

Tein Ansiblea varten oman kansion komennolla _mkdir ansible_ ja lisäsin kansioon hosts.ini -tiedoston sekä tiedostoon sisällön "localhost". 

<img width="436" height="159" alt="9" src="https://github.com/user-attachments/assets/0f67d639-a8b5-444b-ace7-39fd17e80539" />

Tein lisää sisältöä hosts-tiedostoon, joka näyttää koneen uptime-tietoa komennolla _ansible all -a 'uptime' -i hosts.ini_. Testasin SSH:lla localhostille toimiiko Ansible, ja toimihan se –uptime näkyi.  

<img width="1271" height="216" alt="10" src="https://github.com/user-attachments/assets/7f9e84e3-b726-4f53-ad73-cc8f2ea62e84" />

Muokkasin hosts-tiedostoa, ettei terminaali herjaa jatkuvasti Python-tulkista. 

<img width="684" height="227" alt="11" src="https://github.com/user-attachments/assets/2ca51db9-a15d-46bb-b22d-87865f651e50" />

Määritin ansible.cgf tiedoston lukemaan hosts.iniä, joten hosts.ini-komentoa ei tarvitse enää lisätä komentaessa Ansiblea. 

<img width="680" height="215" alt="12" src="https://github.com/user-attachments/assets/edf09446-f50b-40d6-8aa1-c12bd2c29789" />

Seuraavaksi loin site.yml -tiedosto, jolla määritin, mitkä koneet saavat mitkäkin konfiguraatiot. Kaikille koneille (hosts) konfasin tässä tapauksessa rooliksi (role) "hello".

<img width="1273" height="352" alt="13" src="https://github.com/user-attachments/assets/9dd776d6-d8ae-4d3b-baf7-ab62027b2d45" />

Ajoin Ansiblen playbook –komennon, joka ei vielä tehnyt mitään, koska rooleja ei ole määritetty. Tein ne seuraavaksi luomalla ensin rooleille ja suoritettaville tehtäville kansion välivaiheineen (-p) _mkdir -p roles/hello/tasks/_. Tasks-kansioon tein main.yml -tiedoston, jonne määritin sisällön "Hei Ansible". Ajoin tehtävän komennolla _ansible-playbook site.yml_. 

<img width="1278" height="441" alt="14" src="https://github.com/user-attachments/assets/ee021552-b9f0-4e05-b5a6-887e064967ec" />

Testasin toimintaa myös SSH:lla komennolla _ssh localhost 'cat /tmp/hello-ansible'_ ja sain halutun sisällön, eli hyvin toimi. 

<img width="672" height="110" alt="15" src="https://github.com/user-attachments/assets/4709c276-3a57-4d30-aa1c-77e8e1b9405d" />

Tein vielä konfitiedostoon määrityksen, joka näyttää mitä ansible tekee toimiessaan. Testasin ansiblea, ja kuten kuvassa näkyy, komento ei tehnyt nyt mitään muutoksia (changed=0), koska mikään ei ollut muuttunut viime ajon jälkeen. 

<img width="1255" height="456" alt="16" src="https://github.com/user-attachments/assets/8c5cf6e4-767b-4866-b7c9-8d2aee2f6b33" />

Raportin kirjoittamisen lopetin klo 19.30 ja sain Ansiblen toimimaan, myös SSH:lla.

## Lähteet 

 - Tero Karvinen 2026. Ansible ja sen konfaukset. https://terokarvinen.com/palvelinten-hallinta/#laksyt   

## Tekijätiedot 

- Miika Vänttinen, Haaga-Helia Ammattikorkeakoulu, tietojenkäsittely 

- Tätä dokumenttia saa kopioida ja muokata GNU General Public License (versio 2 tai uudempi) mukaisesti. http://www.gnu.org/licenses/gpl.html 

- Pohjana Tero Karvinen 2026: Palvelinten hallinnan kurssi, http://terokarvinen.com 


 


