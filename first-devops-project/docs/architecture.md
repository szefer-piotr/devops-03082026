# System report — systemd timer

Para jednostek systemd uruchamia skrypt raportu stanu maszyny codziennie o **06:00** i **18:00**.

```
timers.target
    → system-report.timer          (kiedy: 06:00 i 18:00)
        → system-report.service    (jak: oneshot)
            → /usr/local/bin/system-report.sh
                → stdout/stderr → journald
```

- `system-report.timer` — scheduler (odpowiednik crona). Nie wykonuje skryptu; budzi service.
- `system-report.service` — `Type=oneshot`: odpal skrypt, poczekaj aż skończy, zakończ. Nie ma sekcji `[Install]`, bo nie startuje przy bootcie — tylko z timera.
- `system-report.sh` — raport na stdout: hostname, data, kernel, uptime, `df`, `free`, top procesy po CPU. Nic nie zmienia na systemie.
- `Persistent=true` — jeśli maszyna spała przez 06:00/18:00, systemd dociągnie pominięty przebieg po starcie.

Pliki w repozytorium są szablonem. Żeby timer naprawdę tykał, trzeba je skopiować we właściwe katalogi systemd (poniżej). Ten projekt **nie instaluje** nic na hoście.

## Instalacja (systemowa)

Z katalogu `first-devops-project/`:

```bash
sudo cp scripts/system-report.sh /usr/local/bin/system-report.sh
sudo chmod 755 /usr/local/bin/system-report.sh

sudo cp systemd/system-report.service /etc/systemd/system/
sudo cp systemd/system-report.timer   /etc/systemd/system/

sudo systemctl daemon-reload
sudo systemctl enable --now system-report.timer
```

Włącza się **tylko timer**. Service odpali się sam o zaplanowanej godzinie.

## Weryfikacja

```bash
systemctl list-timers system-report.timer
systemctl status system-report.timer

# ręczny test service (nie czeka na 06:00/18:00)
sudo systemctl start system-report.service
journalctl -u system-report.service -e
```

Skrypt można też odpalić bez systemd:

```bash
./scripts/system-report.sh
```

## Cofnięcie

```bash
sudo systemctl disable --now system-report.timer
sudo rm /etc/systemd/system/system-report.service
sudo rm /etc/systemd/system/system-report.timer
sudo rm /usr/local/bin/system-report.sh
sudo systemctl daemon-reload
```
