# Ubuntu Hardening mit Ansible

[![Ansible](https://github.com/eazzy1989/ubuntu-hardening-ansible/actions/workflows/ansible-lint.yml/badge.svg)](https://github.com/eazzy1989/ubuntu-hardening-ansible/actions/workflows/ansible-lint.yml)

Ein Ansible-Playbook, das einen frischen Ubuntu-Server mit einem Befehl auf einen sicheren Grundzustand bringt – genau die Maßnahmen, die ich auf meinem [Homelab](https://github.com/eazzy1989/homelab)-Server zuerst von Hand umgesetzt habe, jetzt reproduzierbar als Code.

## Was das Playbook macht

| Bereich | Maßnahme |
|---|---|
| **Firewall (ufw)** | eingehend alles verweigern, ausgehend erlauben; SSH nur aus festgelegten Admin-Netzen; weitere Ports per Variable |
| **SSH** | nur Anmeldung per Schlüssel, Passwort- und root-Login aus, `MaxAuthTries 3`, kein X11-Forwarding |
| **fail2ban** | sperrt IPs nach wiederholten SSH-Fehlversuchen |
| **Updates** | automatische Sicherheitsupdates mit `unattended-upgrades`, optional mit nächtlichem Neustart |

## Sicherheitsnetze gegen Aussperren

- **Kein Passwort-Aus ohne Schlüssel:** Das Playbook bricht ab, wenn für den Admin-Benutzer keine `authorized_keys` existiert.
- **SSH zuerst freigeben:** Die SSH-Regel wird angelegt, **bevor** die Firewall eingeschaltet wird.
- **Erst prüfen, dann neu laden:** Nach einer Änderung prüft `sshd -t` die Konfiguration. Nur wenn sie gültig ist, wird SSH neu geladen – bestehende Sitzungen bleiben offen.
- **Eigene Konfigurationsdatei** (`sshd_config.d/00-hardening.conf`): Die Originaldatei bleibt unverändert; durch das Präfix `00-` haben die Einstellungen Vorrang vor z. B. `50-cloud-init.conf`.

## Verwendung

```bash
# Ansible und benötigte Collection installieren
pip install ansible-core
ansible-galaxy collection install -r requirements.yml

# Inventar anlegen
cp inventory.example.ini inventory.ini
nano inventory.ini

# Erst trocken laufen lassen: zeigt, was sich ändern würde
ansible-playbook site.yml --check --diff --ask-become-pass

# Anwenden
ansible-playbook site.yml --ask-become-pass
```

Der Admin-Benutzer (`ansible_user`) muss sich bereits per SSH-Schlüssel anmelden können (`ssh-copy-id`).

## Konfiguration

Alle Einstellungen stehen in [`roles/hardening/defaults/main.yml`](roles/hardening/defaults/main.yml) und lassen sich im Inventar oder per `-e` überschreiben:

| Variable | Standard | Bedeutung |
|---|---|---|
| `hardening_ssh_allowed_networks` | `192.168.1.0/24` | Netze, aus denen SSH erlaubt ist |
| `hardening_ufw_rules` | 80, 443 | weitere Ports (`port`, `proto`, optional `source`, `comment`) |
| `hardening_ssh_password_authentication` | `false` | Passwort-Login per SSH |
| `hardening_ssh_permit_root_login` | `no` | root-Login per SSH |
| `hardening_fail2ban_maxretry` | `5` | Fehlversuche bis zur Sperre |
| `hardening_unattended_reboot` | `false` | automatischer Neustart nach Kernel-Updates |

## Automatische Prüfung

Bei jedem Push prüft GitHub Actions die Syntax und führt **ansible-lint** aus.

## Tech-Stack

Ansible · ufw · OpenSSH · fail2ban · unattended-upgrades · GitHub Actions

## Lizenz

[MIT](LICENSE)
