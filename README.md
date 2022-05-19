# Debugging an issue where some files are rejected
This appears to be happening on _some_ RP instances only.

  * rpki-client output is in `stdout.txt`
  * Cache directory is in `20220519-rpki-client-apnic-cache.tar.xz`


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
# Add version information
docker exec -ti rpki-client-web-tals_rpki_client_web_apnic_1 bash
bash-5.1$ yum info rpki-client
Fedora 35 - x86_64                                                                             26 MB/s |  79 MB     00:02    
Fedora 35 openh264 (From Cisco) - x86_64                                                      3.8 kB/s | 2.5 kB     00:00    
Fedora Modular 35 - x86_64                                                                    4.6 MB/s | 3.3 MB     00:00    
Fedora 35 - x86_64 - Updates                                                                   15 MB/s |  30 MB     00:01    
Fedora Modular 35 - x86_64 - Updates                                                          5.6 MB/s | 3.1 MB     00:00    
Installed Packages
Name         : rpki-client
Version      : 7.8
Release      : 1.fc35
Architecture : x86_64
Size         : 199 k
Source       : rpki-client-7.8-1.fc35.src.rpm
Repository   : @System
From repo    : updates-testing
Summary      : RPKI validator to support BGP Origin Validation
URL          : https://www.rpki-client.org/
License      : ISC and BSD and Public Domain
Description  : The OpenBSD rpki-client is a free, easy-to-use implementation of the
             : Resource Public Key Infrastructure (RPKI) for Relying Parties (RP) to
             : facilitate validation of the Route Origin of a BGP announcement. The
             : program queries the RPKI repository system, downloads and validates
             : Route Origin Authorisations (ROAs) and finally outputs Validated ROA
             : Payloads (VRPs) in the configuration format of OpenBGPD, BIRD, and
             : also as CSV or JSON objects for consumption by other routing stacks.

```
