```bash
cd /data/ubt/ubtechinc-cloudmanager-web/webapps/cloudManager/WEB-INF/jsp

for i in `grep 'basePath.replace("http:", "https:")' -rl .`;do sed -i '/basePath.replace("http:", "https:")/ s/^/&\/\//g' $i;done  替换
说明：sed -i '/basePath.replace("http:", "https:")/ s/^/&\/\//g' $i中的s/^/&\/\//的^表示行首，&表示行首后面的其他内容，添加//替换


for i in `grep 'basePath.replace("http:", "https:")' -rl .`;do sed -i '/basePath.replace("http:", "https:")/ s/\/\///g' $i;done     还原
```

