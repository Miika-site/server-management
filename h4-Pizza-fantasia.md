# H4 Pizza Fantasia

Aloitin raportin kirjoittamisen klo 17.00.

## x) Tiivistelmä

- DLS tarkoittaa domain spesific languagea eli ns. Kuvauskieli, joka on tarkoitettu tiettyyn tarkoitukseen, kuten Ansible Yaml (konfiguraatiohallinta)
- Salt ja Puppet ovat suosittuja konfiguraationhallintatyökaluja, jotka käyttävät omia DSL:iä
- Saltissa on 510 funktiota, joilla voidaan hallita koneita ja Puppetissa funktioita on 113
- Vaikka hallintatyökalut sisältävät hyvin laajan skaalan funktioita, niistä käytetään eniten tiettyjä rakenteita, joita ovat case, file, package, service ja exec
- Yleisimmät funktiot konfigurointityökaluissa on package-file-service sekä package-file rakenteella

## a) Räpylä. Valitsemani demonin asennus ja toiminnan testaus

Valitsin tämän tehtävän demoniksi Debian 13 pakettikirjastosta löytyvän MariaDB:n, joka on alkuperäisten MySQL kehittäjien luoma avoimen lähdekoodin relaatiotietokanta (MariaDB 2026). Päivitin aluksi pakettilistan komennolla **sudo apt-get update**, jonka jälkeen MarianDB serverin ja client-pakettien asennus. 
````
sudo apt-get install mariadb-server mariadb-client galera-4
```` 
Asennuksessa kesti pari minuuttia. Konfiguroin tarvittavat asetukset MariaDB:lle
````
sudo mariadb-secure-installation
````
Konfiguroinnissa määritellään root-käyttö, anonyymin käyttäjän poisto ja rootin etäkirjautuminen. Määrityksissä ilmoituksen toteamana root-tili oli jo suojattu salasanalla, joten lisämäärityksiä ei tuohon tarvittu. Jätin anonymous userin voimaan, koska kyseessä on local hostissa toimiva testiympäristö, eli nyt MariaDB:hen voi logata anonyymilläkin käyttäjällä. Oikeassa tuontantoympäristössä anonyymin käyttäjän login tulee blokata. Rootin etäkirjautumisen estin. Jätin oletus-databasen **test** käyttöön (tuotannossa tämä test-db pitäisi poistaa). Latasin databasen taulut ja määritykset onnistuivat.

<img width="787" height="847" alt="image" src="https://github.com/user-attachments/assets/1405a6fa-a749-411f-b74c-14014555dc6e" />

<img width="734" height="326" alt="image" src="https://github.com/user-attachments/assets/b58a17f5-8725-4664-87ce-19097250b00e" />

Testasin MariaDB:n tilan ja nätisti tämä demoni potkaistiin käyntiin.

<img width="885" height="294" alt="3" src="https://github.com/user-attachments/assets/2a8e95c0-0398-41da-87ec-8b569ad824e2" />

Varmistin MariaDB:n asennuksen onnistumisen dokumentaation mukaisesti (MariaDB 2026), mutta koneeni käyttäjillä enkä rootilla päässyt yhdistämään databaseen, vaan herjasi oikeuksien puuttumisesta.

<img width="872" height="241" alt="4" src="https://github.com/user-attachments/assets/5623874a-8264-4fa9-b1fd-a42f9cfc39d1" />

Kokeilin käynnistää MariaDB:n konsoliin **sudo mariadb** ja sinne yhdistyi ongelmitta eli ohjelma on käynnissä ja toimii.

<img width="1227" height="314" alt="5" src="https://github.com/user-attachments/assets/df30b78f-bd40-4608-8139-4cee66146213" />



## b) Automaatti. Demonin asennuksen automatisointi Ansiblella.

Tein aluksi tarvittavat tiedostot: 
- hosts.ini // inventaariotiedosto, joka määrittelee orjat (kohdekoneet). Tässä **localhost**.
- ansible.cfg // määrittää inventaarion sijainnin esim. inventory = hosts.ini
- mariainstall.yml // määrittää kohderyhmät ja roolit sekä toimii playbookin ajotiedostona

<img width="503" height="317" alt="6" src="https://github.com/user-attachments/assets/a0faa8c2-46f4-40aa-8b98-0d3692f33fac" />


Aloitin automatisoinnin luomalla uuden roolin ja taskin Ansiblen hakemistoon **mkdir -p roles/automaria/tasks**. Taskiin, eli siihen mitä playbookin tulee suorittaa, määritin main.yml -tiedostoon seuraavan koodin, jolla automatisoidaan MariaDB:n asennus.

<img width="633" height="118" alt="8" src="https://github.com/user-attachments/assets/a7c13326-1b30-4867-9a3e-e4c85bf93ee5" />


Ajoin playbookin ja mitään virheitä ei tullut. Ok=2 eli kaksi tehtävää suoritettu onnistuneesti. Mikään ei myös muuttunut (changed 0), koska minulla oli jo MariaDB asennettuna.

<img width="1366" height="167" alt="7" src="https://github.com/user-attachments/assets/e2e14d03-ea0e-4083-b0d2-38517b14634b" />





## c) Asetus. Asetustiedoston muuttaminen ja osoitus, että asetukset tulivat käyttöön

Muutin äsken luodun automatisoinnin task-asetustiedostoa avaamalla tiedoston tekstieditorilla **micro roles/automaria/tasks/main.yml**. Lisäsin pakettilistan päivityskomennon koodiin.

````
- name: Update apt cache
  apt:
    update_cache: yes
````

<img width="650" height="182" alt="9" src="https://github.com/user-attachments/assets/d010bc95-8944-4dfa-b708-701d5ba5b5f8" />

Ajoin playbookin uudestaan ja sain yhden muutoksen aikaan (changed=1) eli asetus tuli käyttöön, ja toimi odotetulla tavalla.

<img width="1370" height="359" alt="10" src="https://github.com/user-attachments/assets/13191ca3-a463-4bd7-8de2-5a14368061b9" />



## d) Paikka remonttiin. Asetusten rikkominen ja Ansible-playbookin toimivuuden testaus 

Ansiblen playbookini toimivuutta testatakseni, poistin aiemmin asennetun MariaDB:n kokonaan komennolla **sudo apt-get purge mariadb-server**. Testasin vielä, että palvelu on varmasti poistunut koneeltani ja olihan se.

<img width="544" height="49" alt="11" src="https://github.com/user-attachments/assets/71522ad5-83d4-494f-87bc-fa8259f5b5a8" />

Ajoin playbookin heti poiston perään uudestaan. Sain changed=1 ja tässä kohtaa muuttui nimenomaan MariaDB:n asennus-taskin osalta eli hyvin toimi ilman virheitä.

<img width="1198" height="327" alt="12" src="https://github.com/user-attachments/assets/7928ff5c-5f51-4ca1-8261-d1ef83bb675b" />


## e) Idempotentti. Idempotentin osoittaminen

Idempotenssi tarkoittaa, että samat tehtävät (playbook) voidaan suorittaa monta kertaa ja lopputulos muuttuu vain, jos jotain uutta tapahtuu. 

Testasin, että tässä tehtävässä idempotenssi toimii, ajamalla playbookin kaksi kertaa peräkkäin. Ensimmäisellä ajokerralla pakettilista päivittyi (changed=1) ja heti perään toiseen kertaan ajaessa muutoksia ei tapahtunut (changed=0), koska tila oli jo halutunlainen ensimmäisen onnistuneen ajon jälkeen. Tästä juuri idempotenssissa on kyse.

<img width="1276" height="675" alt="13" src="https://github.com/user-attachments/assets/fa33fb67-dd33-436e-ade0-a7e039052165" />

Raportin kirjoittamisen lopetin klo 20.20.

## Lähteet


- Ansible community documentation 2026. Luettavissa: https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_intro.html#playbook-syntax
- Ansible community documentation 2026. Luettavissa: https://docs.ansible.com/projects/ansible/latest/playbook_guide/index.html
- Athreya aka Maneshwar, Dev Community 2025. Playbookin konfiguraatiot. Luettavissa: https://dev.to/lovestaco/getting-started-with-ansible-automate-setups-like-a-pro-5beh
- Karvinen Tero. Tehtävät. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/#h4-pizza-fantasia
- MariaDB 2026. MariaDB asennus. Luettavissa: https://mariadb.com/docs/server/mariadb-quickstart-guides/installing-mariadb-server-guide
  
## Tekijätiedot 

- Miika Vänttinen, Haaga-Helia Ammattikorkeakoulu, tietojenkäsittely 
- Tätä dokumenttia saa kopioida ja muokata GNU General Public License (versio 2 tai uudempi) mukaisesti. http://www.gnu.org/licenses/gpl.html 
- Pohjana Tero Karvinen 2026: Palvelinten hallinnan kurssi, http://terokarvinen.com 
