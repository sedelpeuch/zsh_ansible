# zsh_ansible

Playbooks Ansible qui installent mon environnement shell sur Debian/Ubuntu :

- zsh (défini comme shell de connexion), git, curl, jq
- oh-my-zsh et les plugins `zsh-autosuggestions`, `zsh-syntax-highlighting`, `fzf-tab` et `you-should-use` (`history-substring-search` est intégré à oh-my-zsh)
- fzf (binaire GitHub, car la version apt est trop ancienne pour `fzf --zsh`), mise et node (défini globalement via mise)
- starship
- `.zshrc` et `.zshenv` depuis le [gist zsh](https://gist.github.com/sedelpeuch/a595fc7352f803c089534b00cba9e2e7)
- `starship.toml` et `claude-statusline.sh` depuis le [gist starship](https://gist.github.com/sedelpeuch/1fc26438431e6a29ea35b102b40ab367)
- la clé `statusLine` de `~/.claude/settings.json` : elle est fusionnée dans le fichier existant, dont le reste n'est jamais touché, et une sauvegarde est faite
- la personnalisation GNOME (`configure_gnome.yml`) : extensions, leurs réglages dconf, Terminator sans barre de titre, marge intérieure des terminaux VTE. Ignorée hors session GNOME (conteneur, SSH)

Les versions (fzf, node), les URLs, les extensions GNOME et leurs réglages se règlent dans `group_vars/all.yml`.

## GNOME

Testé sur Ubuntu 26.04, GNOME Shell 50.1 sous Wayland. À lancer depuis un terminal de la session graphique (le playbook passe par le D-Bus de session).

| Extension | Réglages |
|---|---|
| Blur my Shell | Flou derrière Terminator uniquement (opacité 255). Flou des popups coupé : sur GNOME 50, il dessine des coins carrés noirs |
| Rounded Window Corners Reborn | Par défaut |
| Dynamic Music Pill | Par défaut (visible quand un lecteur joue) |
| Top Panel Notification Icons Revived | Par défaut |
| Notification Configurator | Bannière noire à 80 %, polices réduites, centrée en haut |
| Hide Top Bar | Intellihide + révélation à la souris |
| Quick Settings Audio Panel | Par défaut |
| Bluetooth Quick Connect | Par défaut |
| Rounded Corners | Coins de l'écran arrondis |
| Panel corners | Coins arrondis sous la barre du haut |

Essayées puis retirées : Home Assistant Extension, Notification Banner Re-Reloaded, Coverflow Alt-Tab, Space Bar, Gnome 4x/5x UI Improvements, Dash2Dock Animated.

Après le premier passage : se déconnecter puis se reconnecter (correctif `rgba()`), et fermer toutes les fenêtres Terminator pour qu'il relise sa config. Non géré par le playbook : le profil Terminator (`background_type = transparent`, `background_darkness = 0.8`, police `Monaspace Xenon NF 11`, à installer à part).

Pièges :

- **Installation** : une boîte de dialogue « Installer » s'ouvre par extension manquante (D-Bus `InstallRemoteExtension`, sans relog). L'erreur « NoReply » est sans conséquence. `gnome-extensions install` avec un zip demanderait un relog.
- **Correctif Notification Configurator** : l'extension écrit les couleurs semi-transparentes en `#rrggbbaa`, que le CSS de GNOME Shell ne lit pas, et la bannière devient illisible. Le playbook réécrit `utils/themes.js` en `rgba()`. Chaque mise à jour de l'extension écrase le correctif : relancer le playbook.
- **Bannières décalées à droite** : se déconnecter et se reconnecter d'abord. Si le décalage persiste, compenser : `ansible-playbook main.yml -e gnome_notif_right_margin=254`.
- **Autre version de GNOME** : vérifier sur extensions.gnome.org que chaque extension existe pour cette version.

## Utilisation

```sh
ansible-playbook main.yml --ask-become-pass
```

L'inventaire par défaut (`inventory`) cible `localhost`. Pour une autre machine, crée un `inventory.<nom>` (ignoré par git) et passe-le avec `-i`.

## Tester dans un conteneur jetable

```sh
docker run --rm -it -v "$PWD":/src:ro ubuntu:24.04 bash -c '
  apt-get update && apt-get install -y ansible-core sudo &&
  useradd -m -s /bin/bash tester && echo "tester ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/tester &&
  cp -r /src /home/tester/zsh_ansible && chown -R tester: /home/tester/zsh_ansible &&
  cd /home/tester/zsh_ansible &&
  sudo -u tester ansible-playbook main.yml &&
  sudo -u tester ansible-playbook main.yml   # 2e run : doit afficher changed=0
'
```
