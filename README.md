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
# wget the snapshot
# unpack it with reconstruct.py from https://github.com/ties/rpki-rrdp-tools-py
python reconstruct.py ~/tmp/20220519-rpki-client-apnic-cache/rpki-repository.nic.ad.jp-ap/snapshot-69.xml ~/tmp/20220519-rpki-client-apnic-cache/rpki-repository.nic.ad.jp-ap/reconstructed
INFO:__main__:Wrote 1762 files to /home/ties/tmp/20220519-rpki-client-apnic-cache/rpki-repository.nic.ad.jp-ap/reconstructed
# Clean up current files
rm -rf A91A73810000
# Move this into the folder, and use git to diff the state
mv reconstructed/ap/A91A73810000/ .
# Remove spare folder
rm -rf reconstructed
xz snapshot-69.xml
git add .
git status # looks empty
git commit -a -m "Changes look empty"
```
