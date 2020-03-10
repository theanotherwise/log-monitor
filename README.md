# Log parser via crontab

```bash
#!/bin/bash

SCRIPT_NAME=$0
SCRIPT_PATH=`readlink -f $0`
SCRIPT_DIR=`dirname $SCRIPT_PATH`

[ ! -d "$SCRIPT_DIR" ] && exit

VARLOG_DIR=/var/log

[ ! -d "$VARLOG_DIR" ] && exit

FILTERED=$SCRIPT_DIR/.$SCRIPT_NAME.filtered
COMPARED=$SCRIPT_DIR/.$SCRIPT_NAME.compared

touch $FILTERED $COMPARED

VARLOG_KW_KERN="error|critical|failed|problem"
VARLOG_KW_BOOT="error|critical|failed|problem"
VARLOG_KW_AUTH="password check failed|authentication failure"
VARLOG_KW_DPKG="upgrade|install|purge|remove"

tail -25000 $VARLOG_DIR/kern.log 2>/dev/null | grep -iE "$VARLOG_KW_KERN" >> $FILTERED
tail -25000 $VARLOG_DIR/boot.log 2>/dev/null | grep -iE "$VARLOG_KW_BOOT" >> $FILTERED
tail -25000 $VARLOG_DIR/auth.log 2>/dev/null | grep -iE "$VARLOG_KW_AUTH" >> $FILTERED
tail -25000 $VARLOG_DIR/dpkg.log 2>/dev/null | grep -iE "$VARLOG_KW_DPKG" >> $FILTERED

RES=`diff $FILTERED $COMPARED`
cat $FILTERED > $COMPARED

rm -f $FILTERED

[ -n "$RES" ] && echo -e "$RES"
```
