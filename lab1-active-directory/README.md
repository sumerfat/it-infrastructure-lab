# Labor 1: Identitätsmanagement & Benutzer-Onboarding (Active Directory)

## 📋 Szenario-Übersicht
Ein neuer Mitarbeiter, **Max Mustermann**, beginnt seine Tätigkeit in der Abteilung "Finanzen" an einem regionalen Unternehmensstandort. Als Teil des IT-Support- / Systemadministrationsteams ist es meine Aufgabe, sein Benutzerkonto sicher bereitzustellen, ihn den korrekten Berechtigungsgruppen zuzuordnen und den Zugriff auf die benötigten Netzlaufwerke unter Einhaltung des "Principle of Least Privilege" (Minimale Rechtevergabe) zu gewährleisten.

---

## 🛠️ Schritt-für-Schritt Administrations-Workflow

### 1. Benutzeranlage & Bereitstellung (Provisioning)
* **Organizational Unit (OU):** Das Konto wird im Pfad `OU=Finanzen,OU=Personal,DC=firma,DC=local` erstellt. Dies sorgt für eine saubere Verzeichnisstruktur und stellt sicher, dass die spezifischen Gruppenrichtlinien (GPOs) der Finanzabteilung greifen.
* **Benutzerdetails:** 
  * **Vorname:** Max
  * **Nachname:** Mustermann
  * **Benutzeranmeldename (User Principal Name):** `m.mustermann@firma.local`
* **Passwort-Sicherheitsrichtlinie:** Ein temporäres, komplexes Passwort wird generiert. Die Option **"Benutzer muss Kennwort bei der nächsten Anmeldung ändern"** wird zwingend aktiviert, um die Datenschutz- und Sicherheitsrichtlinien zu erfüllen.

### 2. Rollenbasierte Zugriffskontrolle (RBAC) & Gruppenmitgliedschaften
Statt Berechtigungen direkt an Max zu vergeben, wird der Zugriff professionell über Sicherheitsgruppen gesteuert (AGDLP-Prinzip):
* **`G-Finanzen-Mitarbeiter` (Globale Gruppe):** Max wird dieser Gruppe hinzugefügt. Dies steuert die automatische Einbindung des Abteilungs-Netzlaufwerks (z. B. `F:\Finanzen-Share`) via Gruppenrichtlinien-Präferenzen (GPP) bei der Anmeldung.
* **`DL-Finanzen-ReadWrite` (Domänenlokale Gruppe):** Diese Gruppe kontrolliert die tatsächlichen NTFS-Ordnerberechtigungen auf dem Fileserver. Die globale Gruppe `G-Finanzen-Mitarbeiter` wird in diese domänenlokale Gruppe geschachtelt (Accounts -> Global -> Domain Local -> Permissions).

### 3. Verifizierung & Typische Support-Fehlersuche (Troubleshooting)
Um einen reibungslosen ersten Arbeitstag zu garantieren, werden folgende Support-Szenarien dokumentiert:
* **Kontosperrungen:** Gibt der Benutzer sein Passwort 3-mal falsch ein, greift die Kontosperrungsrichtlinie. In *Active Directory-Benutzer und -Computer (ADUC)* wird das Konto über die Eigenschaften des Benutzers im Reiter **"Konto"** mit dem Haken bei **"Konto entsperren"** wieder freigegeben.
* **Gruppenrichtlinien-Replikation:** Sollte das Netzlaufwerk beim Client nicht sofort erscheinen, erzwingt der Befehl `gpupdate /force` in der Windows-Eingabeaufforderung (CMD) die sofortige Aktualisierung der Richtlinien vom Domänencontroller.

---

## 💻 Technische Umgebung & Tools
* **Verzeichnisdienst:** Active Directory Domain Services (AD DS) / Microsoft Entra ID
* **Verwaltungstools:** Active Directory-Benutzer und -Computer (ADUC), Active Directory-Verwaltungscenter, Windows CMD / PowerShell.
* **Fokus-Fähigkeiten:** 1st/2nd Level Ticket-Lösung, Benutzer-Lifecycle, Berechtigungsmanagement.
