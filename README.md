# SASE Reference Architecture Documentation

## Building locally

To work locally with this project, you'll have to follow the steps below:

1. Fork, clone or download this project.
2. [Install][] MkDocs.
3. Preview your project: `mkdocs serve` or `python3 -m mkdocs serve`, your site can be accessed under `localhost:8000.`
4. Add content and verify it.

## Syncing with this repo ( if needed)
```
 git remote add upstream git@gitlab.eng.vmware.com:sa-field-group/sase-ref-arch-mkdocs.git
```

# Fetch all the branches of that remote into remote-tracking branches

```
git fetch upstream
```

Make sure that you're on your master branch:
```
git checkout main
```
Rewrite your master branch so that any commits of yours that
```
git rebase upstream/main
git push  
```



[install]: http://www.mkdocs.org/#installation
