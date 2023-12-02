---
title: Alten Laptop als mobile Hack The Box - Box
date: 02-12-2023
feed: show
---
Nach ausprobieren mehrerer Plattformen, habe ich mich für Hack the Box (HTB) und deren Academy entschieden. 
Die Lektionen sind in Textform (und ich kann schneller lesen, als die meisten reden), ich muss nicht immer Pause drücken oder vor- und zurück spulen usw. taugt mir einfach am besten. 

Nun bin ich aber häufiger auch mal unterwegs und habe ab- und an auch mal ne Stunde Leerlauf. 
Da wäre es doch genial diese Zeit auch nutzen zu können. 
Denn zu lernen gibt es unglaublich viel und die Zeit, die man dafür hat ist sehr begrenzt. 

Ich habe hier noch einen älteren Laptop und zwar einen *ThinkPad X230  mit einem Intel-i5, 16GB RAM und 2x256GB SSDs*

Also wollte ich diesen zu meiner mobilen "Hack the Box Box" machen. 
## Meine Vorstellung
Ein möglichst stabiles und ressourcenschonendes Basissystem auf dem ich eine virtuelle Maschine (VM) für HTB laufen lassen kann. 
Als erstes schmiss ich den Gedanken an Windows aus dem Fenster. Ich brauch hier keine dauernden Updates und erfahrungsgemäß, werden Windows Maschinen, vor allem auf älterer Hardware recht schnell, recht langsam. 
Als VM kamen *Kali*, *Parrot* und *Athena* in Frage. Die ersten beiden auf Debian basierend, die letzte auf Arch. 
Linux ist jetzt kein Fremdwort für mich, aber so wirklich tief steck ich da in der Materie auch nicht drin. Bisherige Erfahrungen basieren vor allem auf Debian und Debian basierten Distros. 
Aber warum auch nicht mal Arch probieren? In einer VM darf ja auch ruhig mal was schiefgehen. 

Erstmal zurück zur Basis oder Host-Maschine (die die VM laufen lassen soll). 

Ich habe mich hier für [Crunchbang++](https://www.crunchbangplusplus.org/) entschieden. 
Minimalistisch, dennoch modern (mit Debian 12), einfach perfekt für mich. 

Installiert, ein wenig angepasst und gefreut. 

Jetzt noch die VM. Um diese laufen zu lassen braucht man ja ein Virtualisierungsprogramm. 
Auf meinem Windows-Desktop nutze ich dafür [VirtualBox](https://www.virtualbox.org/). Für den Laptop ist es aber zu viel "overhead" die VMs laufen ruckelig, die Eingaben sind etwas verzögert... man kann schon damit arbeiten, aber Spaß macht es nicht. 

Bei meiner Suche nach Alternativen bin ich dann auf [quickemu](https://github.com/quickemu-project/quickemu) (ein Wrapper für QEMU) gestoßen. 
Das sah schon vielversprechender aus. 

So sieht meine aktuelle Lösung dafür aus: 

## Host Setup
[Crunchbang++](https://www.crunchbangplusplus.org/) installiert. 
Zusätzlich noch folgendes: 
- [Obsidian](https://obsidian.md/) (als AppImage) für meine Notizen
- [NextCloud](https://nextcloud.com/de/) (als AppImage) um meine Notizen synchronisieren zu können
- [Thorium](https://thorium.rocks/) (als AppImage) als zusätzlichen Browser zu Firefox
- [Neovim](https://neovim.io/) (als AppImage) und [nvim Kickstart](https://github.com/nvim-lua/kickstart.nvim) als Startkonfiguration 
- [flameshot](https://flameshot.org/) (aus den apt repositories) bester Screenshotter. Screenshot - Bearbeiten (markieren etc.) - in die Zwischenablage und daraus direkt in Obsidian einfügen. 

Die Installation von [quickemu](https://github.com/quickemu-project/quickemu)  erschien etwas komplexer, viele Abhängigkeiten und da ich schon mal in den Genuss von Abhägigkeitskonflikten unter Linux gekommen bin, wollte ich das so gut es geht vermeiden (so erklärt sich auch die Verwendung von den AppImages oben). 

Ich habe mich also für [Nix](https://nixos.org/) als zusätzlichen Paketmanager entschieden. 

Über Nix habe ich dann noch folgendes installiert: 
- [quickemu](https://github.com/quickemu-project/quickemu) (selbsterklärend, da das ja das Hauptziel war) auf [quickgui](https://github.com/quickemu-project/quickgui) habe ich verzichtet. 
- später hat sich noch herausgestellt, dass [nixGL](https://github.com/nix-community/nixGL) notwendig ist. 
- zu guter Letzt noch [bitwarden](https://bitwarden.com/) damit ich mich auch überall anmelden kann. 

Das ist jetzt die Basis.

## VM installieren
Ich habe nicht den Weg über `quickget` gewählt, sondern habe die ISOs der Systeme, die ich ausprobieren wollte, von den entsprechenden Seiten gezogen. 
- [Kali](https://www.kali.org/get-kali/#kali-platforms)
- [Parrot](https://parrotlinux.org/download/)
- [Athena](https://athenaos.org/)

Um jetzt die selber bereitgestellte ISO mit `quickemu` starten zu können, bedarf es einer Konfigurationsdatei. 
`vm-os.conf` wobei vm-os der Name der VM sein soll. Ob man sie jetzt kali oder herbert nennt wird wahrscheinlich egal sein, aber man will ja nicht durcheinander kommen. 

Das Minimum, was in der `vm-os.conf` stehen sollte ist: 
```bash
iso="pfad zur iso"
guest_os="linux"
disk_img="verzeichnis-für-vm/disk.qcow2"
```
So kann man die ganzen anderen Details `quickemu` überlassen. 
Das Verzeichnis, das man der VM zur Verfügung stellt, sollte nicht die `vm-iso.conf` enthalten. Ich habe sie im übergeordneten Verzeichnis abgelegt. 

Optional kann man aber auch noch die Anzahl der VM zur Verfügung gestellten Kerne und des Arbeitsspeichers angeben. In meinem Fall: 
```bash
cpu_cores="2"
ram="8G"
```
Und es empfiehlt sich auch noch die Speichergröße, die man der VM zur Verfügung stellen will anzugeben. Da `quickemu` damit wohl recht sparsam umzugehen scheint, kann es knapp werden, wenn man in der VM noch zusätzliche Programme installieren und oder Dateimengen abspeichern will. Also z.B. 
```bash
disk_size="60G"
```

Alle möglichen Parameter für die `vm-os.conf`:
```bash
# Lowercase variables are used in the VM config file only
boot="efi"
bridge=""
cpu_cores=""
disk_img=""
disk_size=""
fixed_iso=""
floppy=""
guest_os="linux"
img=""
iso=""
macos_release=""
port_forwards=()
preallocation="off"
ram=""
secureboot="off"
tpm="off"
usb_devices=()
```

Hat man die `vm-os.conf` erstellt kann man auch schon die erste VM starten mit: 
`quickemu --vm vm-os.conf`

Leider hatte ich so nur eine schwarze Box ohne Inhalt. 

Die Lösung findet sich mit [nixGL](https://github.com/nix-community/nixGL). 
Installiert: 
```bash
$ nix-channel --add https://github.com/guibou/nixGL/archive/main.tar.gz nixgl && nix-channel --update
$ nix-env -iA nixgl.auto.nixGLDefault   # or replace `nixGLDefault` with your desired wrapper
```

Und der Befehl zum Start der VM lautet nun: 
`nixGL quickemu --vm vm-os.conf` 

Jetzt gibt's auch ein Bild und man kann das entsprechende Betriebssystem installieren. 
Folgender freundlicher Hinweis ist aber noch wichtig: 
*When in the VM, use the keys `CTRL + ALT + F` to switch to/from full-screen.
Depending on your system, if the guest is in a window or second monitor and your cursor gets "locked" inside, use the keys `CTRL + ALT + G`.*

Jetzt hat bei mir aber die Bildschirmauflösung nicht ganz gepasst.
Und es fehlten noch ein paar nette Eigenschaften wie: 
- Copy/Paste zwischen Host und VM 
- Filesharing zwischen Host und VM
- Und es wäre super auf USB vom Host aus der VM zugreifen zu können. 

Das lässt sich wohl mit dem *SPICE*-Protokoll erreichen. Dafür brauchen wir auf dem Host erstmal das `spice-client-gtk` Paket. Das ist gottseidank nur ein `sudo apt install spice-client-gtk` entfernt. 

Der Befehl zum Starten lautet jetzt: 
`nixGL quickemu --vm vm-os.conf --display spice`

In der VM selber müssen wir aber auch noch etwas Arbeit erbringen, damit das Ganze auch wirklich so funktioniert.
Wir müssen den SPICE Agent `spice-vdagent` installieren, damit wir Copy/Paste und Zugriff auf USB erhalten. Und für das Filesharing brauchen wir noch den `spice-webdavd` Agenten. 

Das ist schnell erledigt: `sudo apt install spice-vdagent spice-webdavd`

Nach dem Start mit dem schon bekannten: 
`nixGL quickemu --vm vm-os.conf --display spice`
Können wir mit `SHIFT + F11` in den Vollbildmodus wechseln und mit `SHIFT + F12` da auch wieder rauskommen. 

Wir können jetzt auch die Auflösung an den Bildschirm des Host-Systems anpassen über die Schaltfläche *Resize to*.

Falls man diese nicht kennt, helfen: 
`$ xdpyinfo | grep 'dimensions:' ` 
oder
`$ xdpyinfo | awk '/dimensions/ {print $2}'`

Und damit man das nicht bei jedem Start neu eingeben muss, habe ich in den Display-Einstellungen der VM die Auflösung die meinem Bildschirm am nächsten kommt eingestellt. 

Zu guter Letzt, hier noch die `quickemu` Befehle im Überblick: 
```bash
Usage
  quickemu --vm ubuntu.conf

You can also pass optional parameters
  --access                          : Enable remote spice access support. 'local' (default), 'remote', 'clientipaddress'
  --braille                         : Enable braille support. Requires SDL.
  --delete-disk                     : Delete the disk image and EFI variables
  --delete-vm                       : Delete the entire VM and it's configuration
  --display                         : Select display backend. 'sdl' (default), 'gtk', 'none', 'spice' or 'spice-app'
  --fullscreen                      : Starts VM in full screen mode (Ctl+Alt+f to exit)
  --ignore-msrs-always              : Configure KVM to always ignore unhandled machine-specific registers
  --screen <screen>                 : Use specified screen to determine the window size.
  --screenpct <percent>             : Percent of fullscreen for VM if --fullscreen is not specified.
  --shortcut                        : Create a desktop shortcut
  --snapshot apply <tag>            : Apply/restore a snapshot.
  --snapshot create <tag>           : Create a snapshot.
  --snapshot delete <tag>           : Delete a snapshot.
  --snapshot info                   : Show disk/snapshot info.
  --status-quo                      : Do not commit any changes to disk/snapshot.
  --viewer <viewer>                 : Choose an alternative viewer. @Options: 'spicy' (default), 'remote-viewer', 'none'
  --ssh-port <port>                 : Set ssh-port manually
  --spice-port <port>               : Set spice-port manually
  --public-dir <path>               : Expose share directory. @Options: '' (default: xdg-user-dir PUBLICSHARE), '<directory>', 'none'
  --monitor <type>                  : Set monitor connection type. @Options: 'socket' (default), 'telnet', 'none'
  --monitor-telnet-host <ip/host>   : Set telnet host for monitor. (default: 'localhost')
  --monitor-telnet-port <port>      : Set telnet port for monitor. (default: '4440')
  --monitor-cmd <cmd>               : Send command to monitor if available. (Example: system_powerdown)
  --serial <type>                   : Set serial connection type. @Options: 'socket' (default), 'telnet', 'none'
  --serial-telnet-host <ip/host>    : Set telnet host for serial. (default: 'localhost')
  --serial-telnet-port <port>       : Set telnet port for serial. (default: '6660')
  --keyboard <type>                 : Set keyboard. @Options: 'usb' (default), 'ps2', 'virtio'
  --keyboard_layout <layout>        : Set keyboard layout.
  --mouse <type>                    : Set mouse. @Options: 'tablet' (default), 'ps2', 'usb', 'virtio'
  --usb-controller <type>           : Set usb-controller. @Options: 'ehci' (default), 'xhci', 'none'
  --sound-card <type>               : Set sound card. @Options: 'intel-hda' (default), 'ac97', 'es1370', 'sb16', 'none'
  --extra_args <arguments>          : Pass additional arguments to qemu
  --version                         : Print version
```

Um jetzt nicht ständig erst in das richtige Verzeichnis wechseln um dort dann diesen ewigen Befehl `nixGL quickemu --vm vm-os.conf --display spice` eingeben zu müssen, empfehle ich zum Schluss noch die `.bash_aliases` entsprechen anzupassen. 
```bash
alias vm-os='( cd Verzeichnis_in_der_sich_die_vm-os.conf_befindet/ && nixGL quickemu --vm vm-os.conf --display spice )'`
```

Und fertig. 

## Nachtrag
Ich habe mich nun letztlich für Kali entschieden. 
In der VM selber noch folgendes installiert: 

- Obsidian (gleicher Grund wie oben) 
- flameshot (gleicher Grund wie oben)

Noch ein paar Tastenkürzel angepasst, die für mich Sinn machen. 
Und zu guter Letzt noch auf beiden (also Host und VM) die elende Caps-Lock mit Esc belegt. 
Da lässt es sich in vim deutlich leichter Leben. 
Ich hab's so gelöst: in  `.bashrc`
```bash
/usr/bin/setxkbmap -option "caps:escape"
```

Außerdem habe ich zwei Kali-VMs angelegt. Eine "original" in der für's erste alles installiert, aktualisiert und eingestellt ist und eine "clone"-Version. 
Die "clone"-Version ist die Arbeitsversion, sollte ich das irgendwie mal komplett zerschießen, brauche ich nur die "original" wieder zu kopieren. 
Und ja, das Klonen scheint so einfach zu sein, einfach: 

`cp -r kali-orignal kali-clone`

Und den Startbefehl entsprechend anpassen. 

Das Ganze hat so viel länger gebraucht als ich zugeben mag, aber jetzt ist es geschafft und meine kleine HTB-Lernpause ist vorbei. 

Die Zukunft wird zeigen, ob und wie gut dieses System sich bewähren wird. 



LG

rolirk
