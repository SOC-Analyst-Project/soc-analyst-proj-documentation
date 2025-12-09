(Be prepared to do this again for every new device)
1. Accept my invite to the org
2. Install Git plugin in obsidian
3. Create an empty folder for the project (e.g., Github-SOC-Analyst-Project) in obsidian
4. Ctrl+P in obsidian, Git Clone this URL https://github.com/SOC-Analyst-Project/soc-analyst-proj-documentation.git
5. Choose an empty folder on obsidian
6. restart obsidian
7. Ctrl+P Git commit and sync, Authenticate using git on windows. if anything goes wrong go to this doc (https://publish.obsidian.md/git-doc/Authentication)
8. Done. Check change on github repo


# Required: Obsidian setup for attachments location
![525](resources/Pasted%20image%2020251119205016.png)
# Recommended: 
Pull every 5 minute, commit & sync 5 minute after edit
![Pasted image 20251122202055](resources/Pasted%20image%2020251122202055.png)

## Troubleshoot: Merge error "Obsidian Git Sync Setup"
(Find obsidian vault in explorer and open it with vscode)
.gitignore
```
"Obsidian Git Sync Setup.md"
```

Alternatively, use this directly
```
echo "\"Obsidian Git Sync Setup.md\"" >> .gitignore
```

```
git commit -m "Update .gitignore and untrack files"
git check-ignore -v Obsidian\ Git\ Sync\ Setup.md
git push
```
