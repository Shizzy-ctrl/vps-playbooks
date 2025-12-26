# Ansible: Bezpieczna konfiguracja serwera VPS (WireGuard, Docker, Kasm, iptables)

## Kolejność uruchamiania playbooków

1. **initial-playbook.yml**  
   _Aktualizuje system, instaluje unattended-upgrades oraz Docker._  
   **Uruchom jako pierwszy!**

2. **wireguard-playbook.yml**  
   _Instaluje i konfiguruje WireGuard (serwer + klienci)._  
   **Zarządzanie klientami odbywa się przez edycję listy `wireguard_clients` w tym playbooku. Możesz dodawać/usuwać klientów, a playbook automatycznie wygeneruje/wyczyści klucze i konfiguracje._

3. **playbook-security.yml**  
   _Instaluje fail2ban, kopiuje własny jail.local, wyłącza logowanie SSH na hasło._

4. **iptables-playbook.yml**  
   _Konfiguruje iptables: blokuje ruch na publicznym interfejsie, zostawia tylko SSH, WireGuard, ICMP, ustawia NAT dla WireGuard, konfiguruje Docker do współpracy z iptables._

5. **sshblock-playbook.yml**  
   _Blokuje SSH na publicznym interfejsie, zostawia dostęp tylko przez WireGuard._
   **UWAGA: Po tym kroku SSH będzie dostępne wyłącznie przez VPN!**

6. **kasm-playbook.yml** (opcjonalnie)  
   _Instaluje Kasm Workspaces i binduje porty tylko do adresu WireGuard._

---

## Dlaczego taka kolejność?

- **initial-playbook.yml** przygotowuje system i instaluje Docker, wymagany przez kolejne playbooki.
- **wireguard-playbook.yml** musi być uruchomiony zanim zablokujesz SSH na publicznym interfejsie, by nie stracić dostępu do serwera.
- **playbook-security.yml** i **iptables-playbook.yml** wzmacniają bezpieczeństwo, ale nie blokują jeszcze całkowicie SSH.
- **sshblock-playbook.yml** to ostatni krok bezpieczeństwa – po nim dostęp administracyjny SSH jest możliwy tylko przez VPN (WireGuard).
- **kasm-playbook.yml** jest opcjonalny, instaluje środowisko Kasm dostępne wyłącznie przez VPN.

---

## Zarządzanie klientami WireGuard

- Edytuj listę `wireguard_clients` w pliku `wireguard-playbook.yml`.
- Dodanie nowego klienta (np. laptop, telefon) i ponowne uruchomienie playbooka wygeneruje klucze i konfigurację.
- Usunięcie klienta z listy i ponowne uruchomienie playbooka usunie jego klucze i konfigurację.
- Konfiguracje klientów są generowane automatycznie na serwerze.

---

## Pliki i katalogi

- `files/jail.local` – własna konfiguracja fail2ban
- `templates/wg0-server.conf.j2` – szablon konfiguracji serwera WireGuard
- `templates/client.conf.j2` – szablon konfiguracji klienta WireGuard
- `hosts` – plik inwentarza Ansible

---

## Szybki start

```sh
ansible-playbook initial-playbook.yml -i hosts
ansible-playbook wireguard-playbook.yml -i hosts
ansible-playbook playbook-security.yml -i hosts
ansible-playbook iptables-playbook.yml -i hosts
ansible-playbook sshblock-playbook.yml -i hosts
# (opcjonalnie)
ansible-playbook kasm-playbook.yml -i hosts
```

---

**PAMIĘTAJ:**
- Po uruchomieniu sshblock-playbook.yml dostęp SSH tylko przez WireGuard!
- Przetestuj połączenie VPN przed blokadą SSH na publicznym interfejsie!
