```
# Store the cache for rpki-client running against apnic
tar -cf ~/20220519-rpki-client-apnic-cache.tar -C `pwd` apnic/
# Try to create diff with rsync, no files changed in backup, lots of changes
rsync -v -c -r -b --backup-dir backup rsync://rpki-repository.nic.ad.jp/ap/ rpki-repository.nic.ad.jp-ap/ 2>&1 >> rsync.log
# Unpack again, use git to track changes
tar -xf ../20220519-rpki-client-apnic-cache.tar
mv apnic/cache/rpki-repository.nic.ad.jp/ap/ rpki-repository.nic.ad.jp-ap
cd rpki-repository.nic.ad.jp-ap/
git init .
git commit -a -m "Initial state"
cd ..
rsync -v -c -r -b --backup-dir backup rsync://rpki-repository.nic.ad.jp/ap/ rpki-repository.nic.ad.jp-ap/ 2>&1 >> rsync.log
cd rpki-repository.nic.ad.jp-ap/
git add .


```
