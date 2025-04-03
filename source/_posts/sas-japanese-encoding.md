---
title: SAS Japanese Encoding (SJIS/EUC)
date: 2020-05-15 18:48:30
tags:
---

Migrate SAS9.3 on AIX to SAS9.4 on Linux.

The output/log files are in SJIS(Shift-JIS) encoding on AIX and in EUC-JP coding on Linux.

To determine the encoding of the files, open them by MS Word and select encoding, only correct encoding can display Japanese characters correctly.

The issue is caused by different default JP encoding on SAS9.3 and SAS9.4.

On AIX SAS9.3, there are 2 Japanese NLS.
!SASROOT/nls/**ja**/sasv9.cfg
!SASROOT/nls/**ja.euc**/sasv9.cfg

The !SASROOT/nls/ja/sasv9.cfg sets "-ENCODING shift-jis"
The !SASROOT/nls/ja.euc/sasv9.cfg sets "-ENCODING euc-jp"

On Linux SAS9.4, there are 2 Japanese NLS too.
!SASROOT/nls/**ja**/sasv9.cfg
!SASROOT/nls/**ja.sjis**/sasv9.cfg

The !SASROOT/nls/ja/sasv9.cfg sets "-ENCODING euc-jp"
The !SASROOT/nls/ja.sjis/sasv9.cfg sets "-ENCODING shift-jis"

That means that the default JP encoding is SJIS on SAS9.3/AIX and the defualt JP encoding is EUC-JP on SAS9.4/Linux

On SAS9.3/AIX, the **sas** command is a link to **bin/sas_ja**, we can link **sas** command to **bin/sas_ja.sjis** instead if we want the SAS9.4/Linux use Shift-JIS too.
