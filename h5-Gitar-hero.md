# H5 Gitar Hero

Tehtävänanto [terokarvinen.com](https://terokarvinen.com/palvelinten-hallinta/#h5-gitar-hero)
Aloitin raportin tekemisen klo 13.25. 

## x) Tiivistelmä

- Git on versionhallintajärjestelmä, joka käsittelee dataa ikäänkuin sarjana snapshotteja (tilannekuvia). Commitin (eli tallennuksen) yhteydessä Git ottaa snapshotin sen hetkisestä projektin tiedoista/tilasta ja tallentaa viittauksen kyseiseen snapshottiin. Jos projektin tiedostoja ei muuteta, Git ei tallenna tiedostoa uudelleen, vaan viittaa aikaisempaan versioon, jonka se on tallentanut.
- Muut järjestelmät, kuten CSV, Subversion ja Perforce tallentavat dataa delta-pohjaisella versionhallinnalla, jossa vain muuttuneet tiedostot tallennetaan - ei koko projektia. Gitissä tallennetaan koko projektin snapshot. 
- Git toimii pitkälti paikallisesti ja projektin historia luetaan suoraan paikalliseta tietokannasta. Etuna on esim. ettei verkkoyhteyttä tarvita versiohistoriaa selatessa ja commitit onnistuvat offlinessa.
- Gitissä on sisäänrakennettu toiminnallisuus, joka varmistaa, että kaikki tiedostot tarkistetaan SHA-1-hashillä (tarkistussummalla) ennen tallentamista. Tällä varmistetaan datan eheys. 
- Gitin kolme päätilaa ovat: `modified` - tiedostoja on mutettu, mutta ei vielä viety tietokantaan, `staged` - muokatut tiedostot merkattu seuraavaan commitiin, ja `committed` - tiedostot tallennettu paikalliseen Git-tietokantaan. Tämä flow muodostaa Gitin perustyönkulun.

### Gitin peruskäyttö

| Toiminto      | Kuvaus
|---------------|-------------------------------------------|
| git add --all | valmistelee muutokset seuraavaan commitiin |
| git commit    | tallentaa staging -vaiheen muutokset paikalliseen Git-historiaan |
| git pull      | hakee uusimmat muutokset repositoriosta ja integroi ne nykyiseen haaraan (branch) |
| git push      | puskee paikalliset muutokset palvelimelle |


## a) Online. Uusi varasto GitHubiin 
Varaston nimessä ja lyhyessä kuvauksessa tulee olla sana "sunshine". (Muista tehdä varastoon tiedostoja luomisvaiheessa, esim README.md ja GNU General Public License 3) Edistyneemmille voi olla hauskaa etsiä ja kokeilla jokin muu palvelu kuin Github.

Menin työasemani selaimella oman Github-sivustoni repositorioihin `https://github.com/repos`. Gitin repositorio sisältää projektien tiedostot ja toimii varastona niille. Loin uuden julkisen repositorion, jonka nimesin, määritin kuvauksen, lisäsin README-tiedoston sekä lisenssiksi GNU General Public License v3.0. Uuden repositorion luominen tyhjästä Gitiin kestää alle minuutin, riippuen paljonko asetuksia/tekstiä haluaa säätää.

<img width="1274" height="606" alt="1" src="https://github.com/user-attachments/assets/28320620-230c-4229-8441-134db6778fe8" />




## b) Dolly. Varaston kloonaaminen, muutosten tekeminen ja palvelimelle puskeminen

Tässä tehtävässä lähdin Kloonaamaan edellisessä kohdassa tehtyä uutta varastoa itselleni. Loin virtuaalikoneelleni kansion koodeja varten `mkdir code`. Tänne kansioon haen projektin Gitin repositoriosta. Näin kansiorakenne pysyy siistinä ja helposti hallittavana.

Tietoturvamielessä on hyvä käyttää SSH:ta, ja tässäkin tapauksessa kloonasin repon SSH:lla julkista avainta käyttäen. Olin jo aiemmin luonut koneelleni julkisen avaimen ja se tapahtuu komennolla `ssh-keygen`. Tulostin julkisen avaimeni `cat /home/rage2/.ssh/id_ed25519.pub` HUOM! Vain .pub eli julkisen avaimen (public) saa näyttää julkisesti ja tallentaa Gitiin. Julkinen avain toimii koneeni salaisen avaimen vastinparina. Tallensin julkisen avaimen Gitiin: settings > SSH and GPG keys > New SSH key > Add SSH key. Tämän olin tehnyt jo muutamaa päivää aiemmin. Nyt ei tarvitse kirjoittaa salasanaa erikseen joka kerta komentoja ajaessa; esim. push, pull ja clone.

<img width="1272" height="345" alt="3" src="https://github.com/user-attachments/assets/c5e919ad-abc1-49b0-8b81-0a64d551ddbd" />

Kloonasin onnistuneesti aiemmin luodun sunshine-repon itselleni code-hakemiston sisällä `git clone git@github.com:miika-site/sunshine`.

<img width="715" height="197" alt="4" src="https://github.com/user-attachments/assets/d9c28f42-9d84-43a1-a952-5e8e83a0c604" />

Menin repositorion hakemistoon `cd sunshine`ja vedin varaston koneelleni varmistaen, että muutokset ovat ajan tasalla `git pull`.

<img width="425" height="67" alt="6" src="https://github.com/user-attachments/assets/d634dc92-e4fa-4353-8d75-655072907e90" />


Testasin toimivuuden tekemällä muutoksia README-tiedostoon. Kirjoitin lisää tekstiä tiedostoon ja puskin muutokset palvelimelle. Commit-vaiheessa lisäsin selkeän kommentin, mitä muutoksia tehty.
```

git add --all # valmistelee muutokset seuraavaan commitiin
git commit # tallentaa staging -vaiheen muutokset paikalliseen Git-historiaan
git push # puskee paikalliset muutokset palvelimelle

```

<img width="785" height="312" alt="7" src="https://github.com/user-attachments/assets/82372a5c-5f24-41b2-bba1-a2672ca4e7e2" />

Tarkistettu logeista, mitä tuli tehtyä ``git log --patch``. Näin voidaan myös tarkistaa yhteisissä projekteissa, mitä muutoksia muut ovat repoon tehneet. Kuvassa muutokset README-tiedostoon, kuten halusinkin.

<img width="928" height="578" alt="8" src="https://github.com/user-attachments/assets/fba21981-60f0-46f1-be9b-f4cb5a178f1f" />

Repositorio päivittyi Gitin web-liittymään, kuten kuuluu.

<img width="1235" height="628" alt="9" src="https://github.com/user-attachments/assets/6fc365d2-f998-475a-b220-ae7cc0e3ceb4" />





## c) Doh! Tyhmä muutos Gittiin ja huonon muutoksen tuhoaminen ‘git reset --hard’

Lisäsin tekstiä repositorion README-tiedostoon. Commitia en tehnyt. 

<img width="737" height="164" alt="10" src="https://github.com/user-attachments/assets/d0cff5df-aaa6-4268-9c7e-6dd43110eab0" />

Tätä muutosta ei haluttu puskea palvelimelle, joten peruttu se komennolla `git reset --hard`. Viimeisintä muutosta ei nyt siis puskettu, vaan projekti palasi viimeisimmän puskun toimivaan tilaan. Tätä voidaan hyödyntää, jos projektissa rikkoontuu jotain. Huomionarvoista, että `git reset --hard` toimintoa ei voi peruuttaa.

<img width="551" height="162" alt="11" src="https://github.com/user-attachments/assets/8bfaf33c-0591-4a40-85f0-d4fb5db716d5" />

Aikaa raporttiin mennyt n. 1,5h - tauko klo 14.55.


## d) Tukki. Varaston lokien tarkastelu ja käyttäjätietojen tarkistus

Jatkoin raporttia klo 15.50.

`Git config`:lla hallitaan Git-asetuksia koneelta. 

Lokeja voi tutkia komennoilla `git log` (perusloki mm. tekijä, pvm ja commit-viesti) ja `git log --patch` (rivikohtaiset muutokset). Muokkasin varaston README-tiedostoa; poistin rivin ja lisäsin uuden rivin. Ajoin muutokset 
`git add --all` / lisätään muutokset
`git commit`/ tallennus paikalliseen > kommentilla, mitä muutoksia tein.
`git push`/ pusku palvelimelle.

Ajoin `git log --patch` komennon. Alla kuvassa rivikohtaiset muutokset, eli muokkaukset README-tiedstoon: "-" rivi poistettu, "+" rivi lisätty.

<img width="900" height="472" alt="12" src="https://github.com/user-attachments/assets/946d464b-63a3-4a95-86a5-35f96b55245c" />

Olin aiemmin määrittänyt varaston käyttäjäksi sähköpostiosoitteeni ja käyttäjänimen. Ne tehdään `git config` asetustiedostoon.
````
git config --global user.email "johndoe@example.com"
git config --global user.name "John Doe"
````
Käyttäjäasetukset ovat tässä varastossa haluamallani tavalla.

<img width="659" height="206" alt="13" src="https://github.com/user-attachments/assets/81dc06bb-38ed-4ac5-8cd7-a554df173cbe" />



## e) Gitanbile. Laita Ansible-kansio versionhallintaan. Tee jokin muutos, aja ansiblella, tallenna versio (commit).

Edellisen raportin h4 tehtävässä olin luonut Ansible-kansion ja playbookin, jolla automatisoidaan MariaDB:n asennus. 

<img width="487" height="101" alt="14" src="https://github.com/user-attachments/assets/0e1a780d-9168-465b-8def-88578e3193e1" />

Laitoin kyseisen kansion Gittiin versionhallintaan tekemällä ensin kansion sisällä siitä uuden Git-projektin komennolla `git init`.

<img width="819" height="239" alt="15" src="https://github.com/user-attachments/assets/e7c39bcb-c18f-4728-a19c-ea89b682ef66" />

Ajoin `git add --all`jolla valmistelin nykyiset tiedostot committia varten. Ajettu `git commit`jolla tallensin varaston ensimmäisen paikallisen version.

<img width="717" height="665" alt="16" src="https://github.com/user-attachments/assets/329a9097-d92f-43b8-afd8-f36128ef7a3e" />

Ansible playbookini asetukset kuvassa alla ensimmäisessä versiossa:

<img width="664" height="183" alt="17" src="https://github.com/user-attachments/assets/435ef1a2-5afb-42e5-91de-1b616e083dc5" />

Lisäsin tehtäviin `cowsay` ohjelman asennuksen. 

<img width="662" height="268" alt="18" src="https://github.com/user-attachments/assets/0e2ef532-d45e-40de-a7ef-53452835ab29" />

Ajoin playbookin, jotta muutokset tulevat voimaan. Tässä tapauksessa cowsay-ohjelma asentui, ja kaksi muutosta tapahtui (2=changed), toinen oli tosin pakettivaraston päivitys, toinen cowsay asennus.

<img width="1202" height="415" alt="19" src="https://github.com/user-attachments/assets/fe5c92f8-72d7-47d4-ba2f-0dce9f56bb5a" />

Tallennettu uusin versio komennolla `git add --all && git commit`. Tarkistin vielä, että muutokset näkyvät ja ovat halutulla tavalla `git log --patch`. Kuvassa commit-kommentti, mitä on tehty (eli lisätty Cowsay asennus) sekä vihreät rivit "+", jossa otettu uudet asetukset käyttöön ja tallennettu versio.

<img width="804" height="488" alt="20" src="https://github.com/user-attachments/assets/84a50db8-df31-4680-8870-ec09f33ff986" />



## f) Pari projektiin Moodlen keskustelusta

Minulla on jo pari valittuna projektia varten. Tästä projektista kuullaan seuraavissa raporteissa. 

Tehtävät tehty ja raportti valmistui klo 17.40. 

## Lähteet

- Chacon and Straub 2014: Pro Git, 2ed: 1.3 Getting Started - What is Git?. Luettavissa: https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F
- Git Documentation. Tietoja perustoiminnoista. Luettavissa: https://git-scm.com/docs
- Karvinen Tero. Tehtävät. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/#h5-gitar-hero

## Tekijätiedot 

- Miika Vänttinen, Haaga-Helia Ammattikorkeakoulu, tietojenkäsittely 
- Tätä dokumenttia saa kopioida ja muokata GNU General Public License (versio 2 tai uudempi) mukaisesti. http://www.gnu.org/licenses/gpl.html 
- Pohjana Tero Karvinen 2026: Palvelinten hallinnan kurssi, http://terokarvinen.com 
