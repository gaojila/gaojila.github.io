# 记一次批量导出数据


# 场景

开发人员给定数据路径，使用 exel 显示，将 exel 转化为 csv。

# 方法

- 批量导出脚本

```bash
#!/bin/bash

while read line; do
    ROOM=$(echo $line | awk -F',' '{print $3"/"$1"/"$1"_"$4}')
    TARGET=$(echo $line | awk -F',' '{print $9}')
    LINK_NAME=$(echo $line | awk -F',' '{print $8}')

    if [[ -e $TARGET ]]; then
        /bin/mkdir -p "$ROOM"
        /bin/ln -bv -sf "$TARGET" "$ROOM/$LINK_NAME"
    fi
done <<< "$(tail -n +2 foo02.csv)"
```

- 批量打包

```bash
ls -1 | xargs -I{} zip -r "{}.zip" {}
find 2016/ -name "*.zip" -exec mv {} /var/tmp/0415/gsk/2016/ \;
```

