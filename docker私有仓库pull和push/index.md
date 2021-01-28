# docker私有仓库pull和push


# login

```bash
docker login <host>
```

# pull

```bash
docker pull <host>/<porject>/<repo>:<tag>
```

# push

- 重新打 tag

```bash
docker tag <img_name>:<tage> <host>/<project>/<repo>:<tag>
```

- push

```bash
docker push <host>/<porject>/<repo>:<tag>
```

