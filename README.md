## À faire en premier: récupérer le site de GitHub
(Testé avec VSCode)

1. Créer une paire de clés pour authentification GitHub
```bash
ssh-keygen -t ed25519 -C "nom@email.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```
2. Mettre la publique (~/.ssh/id_ed25519.pub) sur GitHub (section Profile - Settings - SSH and GPG keys)

3. Tester la connexion
```bash
ssh -T git@github.com
```

4. Créer un site vide local et sync sur repo GitHub 
```
hugo new site 420-410
cd 420-410
git init
git add .
git submodule add https://github.com/McShelby/hugo-theme-relearn.git themes/hugo-theme-relearn
git remote add origin git@github.com:cegepmv/420-410.git
git branch -M main
git fetch --all
git reset --hard origin/main
```

5. Tester le site local
```
hugo server
```

## Commandes habituelles _hugo_

### Nouveau chapitre:
```
hugo new --kind chapter NOMCHAP/_index.md
```
Dans le fichier _index.md la valeur *weight* détermine la position dans le menu à gauche.

### Nouvelle page dans le chapitre:
```
hugo new NOMCHAP/PAGE.md
```
Modifier ensuite `/home/USER/MONSITE/content/NOMCHAP/PAGE.md`. Ne pas oublier de mettre `draft=false` si on veut voir les pages!


