To be reviewed once 1 join.
CIFS
vol show -vserver <svm> -volume <cifs_vol>
vserver show -vserver <cifs_vserver>
cifs share show <cifs_share> -instance
net int show vserver <cifs_vserver>
Check in GUI also vserver configs
make make a note and prepare how you're going to create share from scratch

NFS
vol show -vserver <nfs_vserver>
vserver show -vserver <nfs_vserver>
check export policy permission of an volume/qtree and check for root vol permission, check if root vol and nfs vol is using the same export policy



