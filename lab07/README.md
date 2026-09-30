# Vaja 7: Procesi in uporabniki

Vaša naloga v tej vaji bo raziskati in analizirati obdelavo log datotek na sistemu Ubuntu, pregledati procese ter upravljati uporabnike in skupine. S pomočjo obstoječih log datotek boste razvili skripte, ki izločijo relevantne informacije o delovanju sistema, napakah in uporabniških aktivnostih. Na koncu boste pripravili poročilo o analizi logov in o dogodkih, ki jih zaznate v realnem času.

Log datoteke se nahajajo v mapi `/var/log/` na sistemu Ubuntu. Analizirajte naslednje log datoteke:

- **syslog** — vse splošne sistemske informacije.
- **auth.log** — informacije o prijavah, odjavah in napakah pri prijavi.
- **dmesg** — nizkonivojski logi sistemskih dogodkov (zaznavanje naprav, napake strojne opreme itd.).
- **dpkg.log** — beleži dogodke, povezane z upravljanjem paketov (npr. nameščanje ali posodabljanje paketov z uporabo APT).

Za branje večine datotek v `/var/log/` potrebujete `sudo`.

---

## Naloga 1: Analiza dnevniških datotek

Ustvarite skript `analiza_logov.sh`, ki bo opravil naslednje naloge:

- Preštel vse napake (`error` ali `fail`) v zadnjih 100 vrsticah datoteke `syslog` in `auth.log` ter izpisal te napake.
- Preštel vse neuspele prijave (failed login attempts) v datoteki `auth.log`.
- Izpisal zadnjih 50 sistemskih sporočil iz `dmesg`, ki vključujejo naprave, ki jih sistem zazna ali odstrani.

Primer skripte:

```bash
#!/bin/bash

echo "=== Napake v zadnjih 100 vrsticah syslog ==="
sudo tail -n 100 /var/log/syslog | grep -Ei 'error|fail'
echo "Število: $(sudo tail -n 100 /var/log/syslog | grep -Eic 'error|fail')"

echo "=== Napake v zadnjih 100 vrsticah auth.log ==="
sudo tail -n 100 /var/log/auth.log | grep -Ei 'error|fail'
echo "Število: $(sudo tail -n 100 /var/log/auth.log | grep -Eic 'error|fail')"

echo "=== Neuspele prijave v auth.log ==="
sudo grep -Ei 'failed password|authentication failure' /var/log/auth.log
echo "Število: $(sudo grep -Eic 'failed password|authentication failure' /var/log/auth.log)"

echo "=== Zadnjih 50 sporočil dmesg o napravah ==="
sudo dmesg | grep -Ei 'usb|device|attached|removed|detect' | tail -n 50
```

Zaženite skripto:

```bash
chmod +x analiza_logov.sh
./analiza_logov.sh
```

Ustvarite še skript `dpkg_analiza.sh`, ki analizira datoteko `/var/log/dpkg.log` in izpiše informacije o vseh paketih, ki so bili nameščeni ali posodobljeni v zadnjih 7 dneh.

Primer skripte:

```bash
#!/bin/bash

since=$(date -d '7 days ago' '+%Y-%m-%d')

echo "Paketi, nameščeni ali posodobljeni od $since:"
sudo awk -v since="$since" '
  $1 >= since && ($3 == "install" || $3 == "upgrade") { print }
' /var/log/dpkg.log
```

```bash
chmod +x dpkg_analiza.sh
./dpkg_analiza.sh
```

---

## Naloga 2: Poročilo o analizi

Recimo, da ste sistemski administrator v podjetju, ki se ukvarja z IT storitvami, in skrbite za strežnik naročnika. Vodja tehnološkega razvoja ima prihodnji teden sestanek, kjer bo predstavil poročilo varnostnega testiranja strežnika. Del tega poročila bo tudi poročilo o varnostnih tveganjih.

Po izvedbi prejšnje naloge pripravite poročilo, v katerem vključite naslednje:

- Povzetek o številu napak in neuspelih prijav, ki ste jih zaznali.
- Analizo zadnjih 50 sistemskih sporočil iz `dmesg` in ugotovitev, ali so bile kakšne naprave nepravilno zaznane ali odstranjene.
- Učinkovitost vašega sistema (npr. ali ste zaznali kakšno napako ali ranljivost).

---

## Naloga 3: Delo s procesi

1. Prikažite vse trenutno aktivne procese v terminalu:

```bash
ps aux
```

2. Prikažite vse procese, ki jih poganja vaš uporabnik:

```bash
ps -u "$USER"
```

3. S pomočjo ukaza `top` preglejte procese v realnem času. Kateri porablja največ procesorja in pomnilnika?

```bash
top
```

Iz `top` izstopite s tipko `q`.

4. Ponovno zaženite `top` in zabeležite PID poljubnega procesa, nato proces ustavite v konzoli:

```bash
kill PID
```

Če se proces ne ustavi, uporabite:

```bash
kill -9 PID
```

5. Z ukazom `ps` poiščite procese, ki vsebujejo določeno ime (na primer `bash`):

```bash
ps aux | grep bash
```

---

## Naloga 4: Dodajanje novega uporabnika

1. Dodajte novega uporabnika z imenom `student1`.
2. Med ustvarjanjem vnesite osnovne podatke, kot so geslo, polno ime in ostale informacije.
3. Preverite, ali je bil uporabnik `student1` uspešno dodan.

```bash
sudo adduser student1
id student1
```

---

## Naloga 5: Dodajanje uporabnika v skupino

1. Ustvarite novo skupino `studenti`.
2. Dodajte uporabnika `student1` v skupino `studenti`.
3. Preverite, ali je bil uporabnik `student1` uspešno dodan v skupino.

```bash
sudo groupadd studenti
sudo usermod -aG studenti student1
groups student1
```

---

## Naloga 6: Sprememba gesla uporabnika

1. Spremenite geslo za uporabnika `student1`.
2. Vpišite novo geslo in preverite, ali je bila sprememba uspešna.

```bash
sudo passwd student1
sudo passwd -S student1
```

Stanje `P` pomeni, da ima račun uporabno geslo. Prijavo preverite z:

```bash
su - student1
```

---

## Naloga 7: Uporabniške pravice

1. Dovolite uporabniku `student1`, da lahko uporablja `sudo` ukaze.
2. Preverite, ali ima uporabnik `student1` zdaj pravice za `sudo`.

```bash
sudo usermod -aG sudo student1
groups student1
sudo -l -U student1
```

---

## Naloga 8: Zaklepanje in odklepanje uporabniškega računa

1. Zaklenite uporabniški račun za `student1`.
2. Preverite, ali je račun zaklenjen, tako da se poskusite prijaviti kot `student1`. Prijava bi morala biti zavrnjena.
3. Odklenite račun za `student1`.

```bash
sudo passwd -l student1
sudo passwd -S student1
su - student1
sudo passwd -u student1
```

Stanje `L` pomeni, da je geslo zaklenjeno.

---

## Naloga 9: Brisanje uporabnika

1. Ustvarite novega uporabnika z imenom `testni_uporabnik`.
2. Preverite, ali je bil uporabnik uspešno dodan.
3. Izbrišite uporabnika `testni_uporabnik`, vendar obdržite domačo mapo.
4. Nato odstranite še domačo mapo.

```bash
sudo adduser testni_uporabnik
id testni_uporabnik
sudo deluser testni_uporabnik
ls /home/testni_uporabnik
sudo rm -rf /home/testni_uporabnik
```

Ukaz `deluser` na Ubuntu račun odstrani in domačo mapo pusti. Popoln izbris računa skupaj z domačo mapo v enem koraku je:

```bash
sudo deluser --remove-home testni_uporabnik
```

---

## Oddaja

Oddajte:

- skripti `analiza_logov.sh` in `dpkg_analiza.sh`,
- poročilo iz naloge 2,
- kratke izpise ukazov iz naloge 3 (procesi, PID, ki ste ga ustavili, in iskanje po imenu),
- izpise ukazov iz nalog 4–9 (uporabnik, skupina, geslo, sudo, zaklep in brisanje).
