# zsh_ansible

Playbooks Ansible qui installent mon environnement shell sur Debian/Ubuntu :

- zsh (défini comme shell de connexion), git, curl, jq
- oh-my-zsh et les plugins `zsh-autosuggestions`, `zsh-syntax-highlighting`, `fzf-tab` et `you-should-use` (`history-substring-search` est intégré à oh-my-zsh)
- fzf (binaire GitHub, car la version apt est trop ancienne pour `fzf --zsh`), mise et node (défini globalement via mise)
- starship
- `.zshrc` et `.zshenv` depuis le [gist zsh](https://gist.github.com/sedelpeuch/a595fc7352f803c089534b00cba9e2e7)
- `starship.toml` et `claude-statusline.sh` depuis le [gist starship](https://gist.github.com/sedelpeuch/1fc26438431e6a29ea35b102b40ab367)
- la clé `statusLine` de `~/.claude/settings.json` : elle est fusionnée dans le fichier existant, dont le reste n'est jamais touché, et une sauvegarde est faite

Les versions (fzf, node) et les URLs se règlent dans `group_vars/all.yml`.

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
