# h2-Lempiväri-violetti

x) Lyhyet vastaukset
- **Pyramid of Pain** kuvaa indikaattoreiden (IOC) eri tasoja, kuten alimmilla esim. hash, IP on vähäinen vaikutus hyökkääjään ja ne ovat helposti vaihdettavissa, kun taas ylhäällä olevat TTP:t ovat vaikeammin muutettavia ja niiden havaitseminen aiheuttaa hyökkääjälle eniten haasteita. https://detect-respond.blogspot.com/2013/03/the-pyramid-of-pain.html 

- **Diamont Model** mallintaa hyökkäyksen neljän elementin avulla: *Adversary*, eli hyökkäjä, *Capability*, eli kyvykkyys, *Infrastructure*, eli infrastruktuuri ja *Victim*, eli kohde, joka sitten tarkastelee näiden elementtien suhteita ja sisältöä hyökkäyksen ymmärtämiseksi. https://www.threatintel.academy/wp-content/uploads/2020/07/diamond-model.pdf

## a) Apache-asennus ja access.log -rivin analyysi

Apache asentamiseen kirjoitin terminaaliin rivi kerralaan:
- sudo apt-get update
- sudo apt-get install apache2
- sudo systemctl start apache2
- sudo systemctl enable apache2

![apache2_lataaminen](https://github.com/user-attachments/assets/a5bed565-2e1a-44ba-bfe5-7f2a08f46bfc)

Tarkistin vielä, että onko Apache2 asennettu, kirjoitamalla terminaaliin -> apache2 -v
Jonka jälkeen ilmeistyi yllä olevassa kuvassa oleva teksti. Tästä voidaan tulkita katsomalla lukeeko Active kohdassa -> active (running). Tämä tarkoittaa sitä, että asennus on suoriutunut. 

Tämän jälkeen testasin palvelinta selaimessa.

![localhost_avaaminen](https://github.com/user-attachments/assets/dedc64b0-b052-460a-b865-12603b1d1e1e)

- Ensimmäiseksi avasin Debian selaimen, eli firefox ja kirjoitin hakuun -> http://localhost/
- Ilmestyi sivusto, jossa luki -> "Apache2 Debian Default Page: It works!"
- Tämä tarkoittaa sitä, että Apache toimii ja kirjauttaa tapahtuman lokiin.

Tämän tarkistuksen jälkeen avataan apachen loki.
- Apache kirjaa kaikki HTTP-pyynnöt tiedostoon.
- Kirjoitin terminaaliin -> sudo tail -n 5 /var/log/apache2/access.log, jonka jälkeen sain sain tämän tuloksen:
- 
![lokirivi](https://github.com/user-attachments/assets/81d59c95-138c-4565-a4aa-56927e763864)

Alla listaan mitä kuvassa lokirivin kohdat atrkoittavat:
- **127.0.0.1** -> IP-osoite, josta pyyntö tuli, eli tässä localhost eli oma kone
- **(- -)**  -> Käyttäjätunnus ja todentamiskenttä (usein tyhjiä)
- **[30/0ct/2025:18:32:50 +0000]** -> Päivämäärä ja kellonaika, jolloin pyytö tehtiin (+0000 = aikavyöhyke)
- **"GET / HTTP/1.1"** -> HTTP-metodi (GET), polku (/ eli etusivu) ja käytetty portokolla (HTTP/1.1)
- **200** -> HTTP-vastauskoodi (200 = OK, onnistunut pyyntö)
- **3380** -> Palautetun sisällön koko tavuina
- **"-"** -> "Referer" eli mistä linkistä tultiin (tässä ei mitään, koska kirjoitettu suoraan selaimeen)
- **"Mozilla/5.0 (x11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0"** -> Käyttäjän selaimen User-Agent-tunniste

Yhteenveto tästä:
- Lokirivi kertoo, että oma kone (127.0.0.1) pyysi palvelimelta oletussivun klo 18:32:50, Apache palautti sen onnistuneesti (HHTP 200).

## b) Nmapped

Ensimmäiseksi kirjoitin terminaaliin komennon:
- sudo apt update
Tämän jälkeen kirjoitin
- sudo apt install nmap -y

Kun asennus oli valmis testasin sen komennolla:
- nmap --version

Tämän jälkeen suoritin komennon:
- sudo nmap -A localhost, jonka vastaukseksi sain:

![4b](https://github.com/user-attachments/assets/348fc36d-e34c-431b-b6b8-43b8d446f19f)

"sudo nmap -A localhost" komento tekee jaana palvelin- ja versiotunnistuksen paikallisesta koneesta (localhost = 127.0.0.1).

Selitys mitä kohdat tarkoittavat:
**Not shown: 998 closed ports** -> nmap skannasi 1000 porttia oletuksena, suurin osa suljettuja.
**PORT STATE SERVICE VERSION** -> näyttää avoimet portit, palvelun ja version (jos tunnistettu).
**|_http-title: Apache2...** -> NSE-skriptin tulos (http-title). Tämä on merkki siitä että -A on suorittanut HTTP-skriptejä.
**OS details** -> nmapin yritys päätellä käyttöjärjestelmää.

## c) Skriptit

Tarkistetaan skriptit edellisen tehtävän kuvasta. Skriptit tunnistaa tulosteessa riveistä, jotka alkavat | tai |_

![4b1](https://github.com/user-attachments/assets/5bdbf892-5cf5-448b-b715-7580291cca7b)

Tässä tapauksessa, kuvassa päällä olivat skriptit:
- ssl-cert
- ssl-date
- amtp-command
- http-server-header
- http-title
- http-robots.txt

  Miksi juuri nämä olivat päällä?
  - Kun käytetään -A, se sisältää --script=default, joka käynnistää oletusskriptin. Nämä oletusskriptit sitten valitsevat automaattisesti palveluun sopivat skriptit kuten esim. SMTP:lle smtp-commands, HTTP:lle http-title jne.

## d) Oikeesti TCP/IP

Käytin komentoa: sudo grep -i "nmaap" /var/log/apache2/access.log, jonka jälkee sain nämä tulokset:

![d](https://github.com/user-attachments/assets/621d262c-b2c8-4fb4-ba9c-d5d11ab4e61b)

Tässä tapauksessa -i komento etsii "nmap" lokkitiedostosta riipumatta siitä, että onko se kirjoitettu isolla vai pienellä kirjaimella.

Koska tämän tein ilman internet yhteyttä, ne kaikki tulivat omalta koneelta.

Usein lokeissa ei näy sana nmap, joten tämän voi tunnistaa esimerkiksi sillä tavalla, että IP-osoite tekee monta pyyntöä hyvin nopeasti.


## e) Wire sharking

Ensimmmäiseksi asensin wiresharkin komennolla "sudo apt install --reinstall wireshark-common wireshark ja sille annettiin tarvittavat käyttöoikeudett.

Kun asensin tämän, annettiin dumpcapille käyttöoikeudet komennolla:
*sudo setcap cap_net_raw.cap_net_admin=eip /user/bin/dumpcap*

Laitoin wireshark käyntiin.

Tämän jälkeen suoritin Nmap-skannauksen Apache-palvelimelle komennolla:
*sudo nmap -T4 -vv -A -p 80 --script http-title --skript-args "http.useragent=nmap-test" localhost*

Tämän jälkeen lopetin wiresharkin ja otin kaappauksen vain niistä kohdista jossa näkyi useita HTTP-paketteja.

![e](https://github.com/user-attachments/assets/2b870f4b-a7b0-4e7f-bc4a-79e4b3586b1f)

## f) Net grep

1) Asensin ngrep-komennon terminaalissa: *sudo apt update sudo apt install ngrep*
2) Tämän jälkeen tarkistin rajapinnan: *ip a*
3) Käynnistin sieppauksen komennolla: *sudo ngrep -d enp0s3 -W byline '(?!)nmap' -O nmap_hits.pcap |& tee nmap_hints.txt*
4) Lopetin sieppauksen: Ctrl + c

![f](https://github.com/user-attachments/assets/fb9c82cd-3ead-40fa-bcc4-b0e73060731a)

## g) Agentti

![g](https://github.com/user-attachments/assets/53d06fca-2df2-449d-a6aa-156a0f75fe76)

1) vEnsimmäiseksi ajoin ngrep tallennusta, joka kuunteli verkon liikennettä ja etsi User-Agent-tietoja.
2) Tämän jälkeen ajoin nmap 1, ilman muutoksia.
3) Ajoin nmap 2, jossa alkoi näyttämään User-Agentin tavalliselta selaimelta: *sudo nmap -p80 --script http-title --script-args 'http.useragent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/116' target.example.com*
4) Kun nämä molemmat ajot oli tehty, pysäytin sieppauksen painamalla Ctrl + c.
5) Lopuksi avasin tallennetun pcap-tiedoston Wiresharkissa ja tarkistin, että User-Agent näkyi pakettien tiedoissa -> *wireshark ua_test.pcap &*


## Lähteet
- https://httpd.apache.org/docs/2.4/logs.html
- https://nmap.org/nsedoc/scripts/http-useragent-tester.html
- https://github.com/jpr5/ngrep/blob/master/EXAMPLES.md 
- https://www.sumologic.com/blog/apache-access-log








 


