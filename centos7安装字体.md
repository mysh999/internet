1、查看当前字体信息

```bash
# fc-list 
/usr/share/fonts/dejavu/DejaVuSansCondensed-Oblique.ttf: DejaVu Sans,DejaVu Sans Condensed:style=Condensed Oblique,Oblique
/usr/share/fonts/dejavu/DejaVuSansCondensed-Bold.ttf: DejaVu Sans,DejaVu Sans Condensed:style=Condensed Bold,Bold
/usr/share/fonts/dejavu/DejaVuSans.ttf: DejaVu Sans:style=Book
/usr/share/fonts/dejavu/DejaVuSans-Bold.ttf: DejaVu Sans:style=Bold
/usr/share/fonts/dejavu/DejaVuSansCondensed.ttf: DejaVu Sans,DejaVu Sans Condensed:style=Condensed,Book
/usr/share/fonts/dejavu/DejaVuSans-ExtraLight.ttf: DejaVu Sans,DejaVu Sans Light:style=ExtraLight
/usr/share/fonts/dejavu/DejaVuSansCondensed-BoldOblique.ttf: DejaVu Sans,DejaVu Sans Condensed:style=Condensed Bold Oblique,Bold Oblique
/usr/share/fonts/dejavu/DejaVuSans-Oblique.ttf: DejaVu Sans:style=Oblique
/usr/share/fonts/dejavu/DejaVuSans-BoldOblique.ttf: DejaVu Sans:style=Bold Oblique
```



2、上传新字体文件到/usr/share/fonts下 

```bash
# pwd
/usr/share/fonts

# ll
total 8
drwxr-xr-x 2 root root 4096 Jul  9  2019 dejavu
drwxr-xr-x 2 root root 4096 Aug 17 16:12 siyuan
```



3、刷新字体缓存

```bash
# fc-cache -f -v
/usr/share/fonts: caching, new cache contents: 0 fonts, 2 dirs
/usr/share/fonts/dejavu: caching, new cache contents: 9 fonts, 0 dirs
/usr/share/fonts/siyuan: caching, new cache contents: 24 fonts, 0 dirs
/usr/share/X11/fonts/Type1: skipping, no such directory
/usr/share/X11/fonts/TTF: skipping, no such directory
/usr/local/share/fonts: skipping, no such directory
/root/.local/share/fonts: skipping, no such directory
/root/.fonts: skipping, no such directory
/usr/share/fonts/dejavu: skipping, looped directory detected
/usr/share/fonts/siyuan: skipping, looped directory detected
/usr/lib/fontconfig/cache: cleaning cache directory
/root/.cache/fontconfig: not cleaning non-existent cache directory
/root/.fontconfig: not cleaning non-existent cache directory
/usr/bin/fc-cache-64: succeeded
```





4、查看新字体信息

```bash
# fc-list
/usr/share/fonts/siyuan/msyh.ttc: Microsoft YaHei UI:style=Normal
/usr/share/fonts/siyuan/SourceHanSansCN-Normal.otf: Source Han Sans CN Normal,思源黑体 CN,Source Han Sans CN:style=Regular,Normal
/usr/share/fonts/dejavu/DejaVuSansCondensed-Oblique.ttf: DejaVu Sans,DejaVu Sans Condensed:style=Condensed Oblique,Oblique
/usr/share/fonts/dejavu/DejaVuSansCondensed-Bold.ttf: DejaVu Sans,DejaVu Sans Condensed:style=Condensed Bold,Bold
/usr/share/fonts/siyuan/msyhbd.ttc: Microsoft YaHei UI:style=Έντονα
/usr/share/fonts/siyuan/impact.ttf: Impact:style=Regular,Standard
/usr/share/fonts/siyuan/SourceHanSansCN-Heavy.otf: Source Han Sans CN Heavy,思源黑体 CN,Source Han Sans CN:style=Bold,Heavy
/usr/share/fonts/siyuan/calibrib.ttf: Calibri:style=Bold
/usr/share/fonts/siyuan/calibrii.ttf: Calibri:style=Italic
/usr/share/fonts/dejavu/DejaVuSans.ttf: DejaVu Sans:style=Book
/usr/share/fonts/siyuan/SourceHanSansCN-Bold.otf: Source Han Sans CN,Source Han Sans CN Bold:style=Bold
/usr/share/fonts/siyuan/SourceHanSansCN-ExtraLight.otf: Source Han Sans CN ExtraLight,思源黑体 CN,Source Han Sans CN:style=Regular,ExtraLight
/usr/share/fonts/siyuan/SourceHanSansCN-Regular.otf: Source Han Sans CN,Source Han Sans CN Regular:style=Regular
/usr/share/fonts/siyuan/SourceHanSansCN-Medium.otf: Source Han Sans CN Medium,思源黑体 CN,Source Han Sans CN:style=Regular,Medium
/usr/share/fonts/siyuan/calibri.ttf: Calibri:style=Regular
/usr/share/fonts/siyuan/verdanaz.ttf: Verdana:style=Bold Italic,Negreta cursiva
/usr/share/fonts/siyuan/verdana.ttf: Verdana:style=Regular
/usr/share/fonts/siyuan/SourceHanSansCN-Light.otf: Source Han Sans CN Light,思源黑体 CN,Source Han Sans CN:style=Regular,Light
/usr/share/fonts/dejavu/DejaVuSans-Bold.ttf: DejaVu Sans:style=Bold
/usr/share/fonts/siyuan/calibriz.ttf: Calibri:style=Bold Italic
/usr/share/fonts/siyuan/msyhl.ttc: Microsoft YaHei UI,Microsoft YaHei UI Light:style=Light,Regular
/usr/share/fonts/siyuan/calibril.ttf: Calibri,Calibri Light:style=Light,Regular
/usr/share/fonts/siyuan/msyh.ttc: Microsoft YaHei:style=Normal
/usr/share/fonts/siyuan/msyhl.ttc: Microsoft YaHei,Microsoft YaHei Light:style=Light,Regular
/usr/share/fonts/siyuan/calibrili.ttf: Calibri,Calibri Light:style=Light Italic,Italic
/usr/share/fonts/dejavu/DejaVuSansCondensed.ttf: DejaVu Sans,DejaVu Sans Condensed:style=Condensed,Book
/usr/share/fonts/dejavu/DejaVuSans-ExtraLight.ttf: DejaVu Sans,DejaVu Sans Light:style=ExtraLight
/usr/share/fonts/dejavu/DejaVuSansCondensed-BoldOblique.ttf: DejaVu Sans,DejaVu Sans Condensed:style=Condensed Bold Oblique,Bold Oblique
/usr/share/fonts/dejavu/DejaVuSans-Oblique.ttf: DejaVu Sans:style=Oblique
/usr/share/fonts/siyuan/verdanab.ttf: Verdana:style=Bold,Negreta
/usr/share/fonts/dejavu/DejaVuSans-BoldOblique.ttf: DejaVu Sans:style=Bold Oblique
/usr/share/fonts/siyuan/msyhbd.ttc: Microsoft YaHei:style=Έντονα
/usr/share/fonts/siyuan/verdanai.ttf: Verdana:style=Italic,Cursiva
```

新字体生效